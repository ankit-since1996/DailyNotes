---
description: 'Source : AWS Skill Builder'
---

# Amazon ECS - Knowledge Badge

[Reference URL](https://skillbuilder.aws/learning-plan/BDJU9EAPJK/amazon-ecs--knowledge-badge-readiness-path)



#### ECS Deployment using GitHub Actions (+ OIDC + GitHub Secrets)

```yaml
name: ECS Multi-Branch & Multi-Environment CI/CD

on:
  push:
    branches:
      - develop
      - main
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment to deploy'
        required: true
        type: choice
        options:
          - stage
          - prod
      image_tag:
        description: 'Optional existing image tag in ECR to deploy (e.g., main-a1b2c3d-12). Leave blank to build from commit.'
        required: false
        default: ''

permissions:
  id-token: write
  contents: read

env:
  AWS_REGION: eu-west-1
  # Configure via GitHub Actions Repository/Environment Variables or Secrets
  ECR_REGISTRY: ${{ vars.ECR_REGISTRY }}
  ECR_REPOSITORY: ${{ vars.ECR_REPOSITORY }}
  CONTAINER_NAME: ${{ vars.CONTAINER_NAME }}
  STAGE_CLUSTER_NAME: ${{ vars.STAGE_CLUSTER_NAME }}
  STAGE_ECS_SERVICE: ${{ vars.STAGE_ECS_SERVICE }}
  STAGE_TASK_DEFINITION: ${{ vars.STAGE_TASK_DEFINITION }}
  PROD_CLUSTER_NAME: ${{ vars.PROD_CLUSTER_NAME }}
  PROD_ECS_SERVICE: ${{ vars.PROD_ECS_SERVICE }}
  PROD_TASK_DEFINITION: ${{ vars.PROD_TASK_DEFINITION }}
  # Optional: fail pipeline if CRITICAL or HIGH vulnerabilities are found (set to "true" to enforce)
  FAIL_ON_VULNERABILITIES: ${{ vars.FAIL_ON_VULNERABILITIES || 'false' }}

concurrency:
  group: ecs-deploy-${{ github.ref }}
  cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}

jobs:
  resolve-context:
    name: Resolve Target Environment
    runs-on: ubuntu-latest
    outputs:
      target_env: ${{ steps.ctx.outputs.target_env }}
      iam_role_arn: ${{ steps.ctx.outputs.iam_role_arn }}
      sns_topic_arn: ${{ steps.ctx.outputs.sns_topic_arn }}
      cluster_name: ${{ steps.ctx.outputs.cluster_name }}
      ecs_service: ${{ steps.ctx.outputs.ecs_service }}
      task_definition: ${{ steps.ctx.outputs.task_definition }}
      tag_prefix: ${{ steps.ctx.outputs.tag_prefix }}
      should_build: ${{ steps.ctx.outputs.should_build }}
      explicit_tag: ${{ steps.ctx.outputs.explicit_tag }}
      image_tag_only: ${{ steps.ctx.outputs.image_tag_only }}
    steps:
      - id: ctx
        shell: bash
        run: |
          SHORT_SHA=$(echo "${{ github.sha }}" | cut -c 1-7)
          INPUT_ENV="${{ github.event.inputs.environment }}"
          INPUT_TAG="${{ github.event.inputs.image_tag }}"

          if [ "${{ github.event_name }}" = "workflow_dispatch" ]; then
            TARGET_ENV="$INPUT_ENV"
          elif [ "${{ github.ref }}" = "refs/heads/main" ]; then
            TARGET_ENV="prod"
          else
            TARGET_ENV="stage"
          fi

          if [ "$TARGET_ENV" = "prod" ]; then
            ROLE_ARN="${{ secrets.AWS_ROLE_ARN_PROD }}"
            SNS_ARN="${{ secrets.SNS_TOPIC_ARN_PROD }}"
            CLUSTER="${{ env.PROD_CLUSTER_NAME }}"
            SERVICE="${{ env.PROD_ECS_SERVICE }}"
            TASK_DEF="${{ env.PROD_TASK_DEFINITION }}"
            PREFIX="prod"
          else
            ROLE_ARN="${{ secrets.AWS_ROLE_ARN_STAGE }}"
            SNS_ARN="${{ secrets.SNS_TOPIC_ARN_STAGE }}"
            CLUSTER="${{ env.STAGE_CLUSTER_NAME }}"
            SERVICE="${{ env.STAGE_ECS_SERVICE }}"
            TASK_DEF="${{ env.STAGE_TASK_DEFINITION }}"
            PREFIX="stage"
          fi

          if [ -n "$INPUT_TAG" ]; then
            SHOULD_BUILD="false"
            RESOLVED_TAG="$INPUT_TAG"
          else
            SHOULD_BUILD="true"
            RESOLVED_TAG="${PREFIX}-${SHORT_SHA}-${{ github.run_number }}"
          fi

          {
            echo "target_env=$TARGET_ENV"
            echo "iam_role_arn=$ROLE_ARN"
            echo "sns_topic_arn=$SNS_ARN"
            echo "cluster_name=$CLUSTER"
            echo "ecs_service=$SERVICE"
            echo "task_definition=$TASK_DEF"
            echo "tag_prefix=$PREFIX"
            echo "should_build=$SHOULD_BUILD"
            echo "explicit_tag=$RESOLVED_TAG"
            echo "image_tag_only=$RESOLVED_TAG"
          } >> "$GITHUB_OUTPUT"

  build-scan-push:
    name: Build, Scan & Push Image (${{ needs.resolve-context.outputs.target_env }})
    needs: resolve-context
    if: needs.resolve-context.outputs.should_build == 'true'
    runs-on: ubuntu-latest
    environment: ${{ needs.resolve-context.outputs.target_env }}
    outputs:
      image_uri: ${{ steps.build-tag.outputs.image_uri }}
      critical: ${{ steps.ecr-scan.outputs.critical }}
      high: ${{ steps.ecr-scan.outputs.high }}
      medium: ${{ steps.ecr-scan.outputs.medium }}
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ needs.resolve-context.outputs.iam_role_arn }}
          role-session-name: gha-ecs-build-${{ github.run_id }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build and tag Docker image
        id: build-tag
        env:
          IMAGE_TAG: ${{ needs.resolve-context.outputs.image_tag_only }}
        run: |
          IMAGE_URI="${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${IMAGE_TAG}"
          docker build \
            --build-arg BUILD_ENV="${{ needs.resolve-context.outputs.target_env }}" \
            -t "$IMAGE_URI" \
            .
          echo "image_uri=$IMAGE_URI" >> "$GITHUB_OUTPUT"

      - name: Push image to Amazon ECR
        run: |
          docker push "${{ steps.build-tag.outputs.image_uri }}"

      - name: Run ECR vulnerability scan and evaluate findings
        id: ecr-scan
        shell: bash
        env:
          IMAGE_TAG: ${{ needs.resolve-context.outputs.image_tag_only }}
          FAIL_ON_VULN: ${{ env.FAIL_ON_VULNERABILITIES }}
        run: |
          aws ecr start-image-scan \
            --repository-name "${{ env.ECR_REPOSITORY }}" \
            --image-id imageTag="$IMAGE_TAG" || true

          aws ecr wait image-scan-complete \
            --repository-name "${{ env.ECR_REPOSITORY }}" \
            --image-id imageTag="$IMAGE_TAG"

          FINDINGS_JSON=$(aws ecr describe-image-scan-findings \
            --repository-name "${{ env.ECR_REPOSITORY }}" \
            --image-id imageTag="$IMAGE_TAG" \
            --output json)

          CRITICAL_COUNT=$(echo "$FINDINGS_JSON" | jq '[.imageScanFindings.findings[]? | select(.severity == "CRITICAL")] | length')
          HIGH_COUNT=$(echo "$FINDINGS_JSON" | jq '[.imageScanFindings.findings[]? | select(.severity == "HIGH")] | length')
          MEDIUM_COUNT=$(echo "$FINDINGS_JSON" | jq '[.imageScanFindings.findings[]? | select(.severity == "MEDIUM")] | length')

          echo "ECR Scan Summary -> CRITICAL: $CRITICAL_COUNT, HIGH: $HIGH_COUNT, MEDIUM: $MEDIUM_COUNT"
          {
            echo "critical=$CRITICAL_COUNT"
            echo "high=$HIGH_COUNT"
            echo "medium=$MEDIUM_COUNT"
          } >> "$GITHUB_OUTPUT"

          if [ "$FAIL_ON_VULNERABILITIES" = "true" ] && { [ "$CRITICAL_COUNT" -gt 0 ] || [ "$HIGH_COUNT" -gt 0 ]; }; then
            echo "::error::Blocking deployment: Found $CRITICAL_COUNT CRITICAL and $HIGH_COUNT HIGH ECR vulnerabilities."
            exit 1
          fi

  deploy:
    name: Deploy to ECS (${{ needs.resolve-context.outputs.target_env }})
    needs: [resolve-context, build-scan-push]
    if: |
      always() &&
      needs.resolve-context.result == 'success' &&
      (needs.build-scan-push.result == 'success' || needs.build-scan-push.result == 'skipped')
    runs-on: ubuntu-latest
    environment: ${{ needs.resolve-context.outputs.target_env }}
    steps:
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ needs.resolve-context.outputs.iam_role_arn }}
          role-session-name: gha-ecs-deploy-${{ github.run_id }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Resolve image URI
        id: image
        run: |
          if [ -n "${{ needs.build-scan-push.outputs.image_uri }}" ]; then
            URI="${{ needs.build-scan-push.outputs.image_uri }}"
          else
            URI="${{ env.ECR_REGISTRY }}/${{ env.ECR_REPOSITORY }}:${{ needs.resolve-context.outputs.image_tag_only }}"
          fi
          echo "uri=$URI" >> "$GITHUB_OUTPUT"

      - name: Download current ECS task definition
        id: task-def-fetch
        shell: bash
        run: |
          aws ecs describe-task-definition \
            --task-definition "${{ needs.resolve-context.outputs.task_definition }}" \
            --query 'taskDefinition' > task-def.json

      - name: Render updated ECS task definition
        id: task-def-render
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: task-def.json
          container-name: ${{ env.CONTAINER_NAME }}
          image: ${{ steps.image.outputs.uri }}

      - name: Deploy to Amazon ECS
        id: ecs-deploy
        uses: aws-actions/amazon-ecs-deploy-task-definition@v2
        with:
          task-definition: ${{ steps.task-def-render.outputs.task-definition }}
          service: ${{ needs.resolve-context.outputs.ecs_service }}
          cluster: ${{ needs.resolve-context.outputs.cluster_name }}
          wait-for-service-stability: true

  notify-sns:
    name: Send Deployment Notification via AWS SNS
    needs: [resolve-context, build-scan-push, deploy]
    if: always() && needs.resolve-context.result == 'success'
    runs-on: ubuntu-latest
    environment: ${{ needs.resolve-context.outputs.target_env }}
    steps:
      - name: Configure AWS credentials via OIDC
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ needs.resolve-context.outputs.iam_role_arn }}
          role-session-name: gha-ecs-notify-${{ github.run_id }}
          aws-region: ${{ env.AWS_REGION }}

      - name: Publish deployment status to SNS
        shell: bash
        env:
          TARGET_ENV: ${{ needs.resolve-context.outputs.target_env }}
          IMAGE_TAG: ${{ needs.resolve-context.outputs.image_tag_only }}
          CLUSTER_NAME: ${{ needs.resolve-context.outputs.cluster_name }}
          ECS_SERVICE: ${{ needs.resolve-context.outputs.ecs_service }}
          SNS_TOPIC_ARN: ${{ needs.resolve-context.outputs.sns_topic_arn }}
          BUILD_RESULT: ${{ needs.build-scan-push.result }}
          DEPLOY_RESULT: ${{ needs.deploy.result }}
          CRIT_COUNT: ${{ needs.build-scan-push.outputs.critical || 'N/A' }}
          HIGH_COUNT: ${{ needs.build-scan-push.outputs.high || 'N/A' }}
        run: |
          if [ -z "$SNS_TOPIC_ARN" ]; then
            echo "SNS_TOPIC_ARN not set for $TARGET_ENV; skipping notification."
            exit 0
          fi

          if [ "$DEPLOY_RESULT" = "success" ]; then
            STATUS="SUCCEEDED"
          elif [ "$BUILD_RESULT" = "failure" ]; then
            STATUS="FAILED (Build/Scan)"
          elif [ "$DEPLOY_RESULT" = "failure" ]; then
            STATUS="FAILED (ECS Deploy)"
          else
            STATUS="SKIPPED/CANCELLED"
          fi

          SHORT_SHA=$(echo "${{ github.sha }}" | cut -c 1-7)
          RUN_URL="${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}"
          SUBJECT="[${TARGET_ENV^^}] ECS Deploy ${STATUS}: ${{ env.ECR_REPOSITORY }}:${IMAGE_TAG}"

          MESSAGE=$(cat <<EOF
ECS Deployment Notification
---------------------------
Environment : $TARGET_ENV
Status      : $STATUS
Service     : $ECS_SERVICE (Cluster: $CLUSTER_NAME)
Image Tag   : $IMAGE_TAG
Commit      : $SHORT_SHA (${{ github.actor }})
ECR Scan    : CRITICAL=$CRIT_COUNT, HIGH=$HIGH_COUNT
Workflow Run: $RUN_URL
EOF
)

          aws sns publish \
            --topic-arn "$SNS_TOPIC_ARN" \
            --subject "$SUBJECT" \
            --message "$MESSAGE"
```
