---
description: >-
  Statement: Unable to renew certbot SSL
  cert.                                                                                                   
  Date: 01/05/2024
layout:
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Let'sEncrypt certbot SSL

<mark style="color:red;">**Issue**</mark>: While trying to renew the Let'sEncrypt free SSL cert in the DEV server, we got a Timeout error and could not renew it.

<pre class="language-bash" data-title="Initial Error Output " data-overflow="wrap" data-full-width="true"><code class="lang-bash"><strong>> certbot renew --dry-run
</strong><strong>
</strong>Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/virtualhost.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Simulating renewal of an existing certificate for the domain

Certbot failed to authenticate some domains (authenticator: apache). The Certificate Authority reported these problems:
  Domain: XXXYZ.com
  Type:   connection
  Detail: 2.6.34.4: Fetching http://domain.com/.well-known/acme-challenge/dFQ7Ov0RIZ1HLcow-_YJ-qPN6DIfhiTX2Pc5hp8uDgA: Timeout during connect (likely firewall problem)

Hint: The Certificate Authority failed to verify the temporary Apache configuration changes made by Certbot. Ensure that the listed domains point to this Apache server and that it is accessible from the internet.

Failed to renew certificate with error: Some challenges have failed.

</code></pre>

<mark style="color:green;">**Steps Taken**</mark>**:**

* From the error, it becomes clear that the command was trying to connect using http which uses port 80 for communication. As per security standards, we have disabled port 80 once our domain is running on https. Because of this, it was getting time out.
* We opened port 80 from the security group and reran the command.

{% code title="Solution Output" overflow="wrap" fullWidth="true" %}
```bash
> certbot renew
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/domain.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Renewing an existing certificate for domain.com

Reloading apache server after certificate renewal
Congratulations, all renewals succeeded.

```
{% endcode %}

#### Reference Reading:

When you get a certificate from Let’s Encrypt, our servers validate that you control the domain names in that certificate using “challenges,” as defined by the ACME standard. Most of the time, this validation is handled automatically by your ACME client.

<mark style="color:yellow;">**ACME challenge:**</mark> As you obtained the SSL certificate and private key from Let’s Encrypt, you’re required to prove you own your domain name registered for this SSL certificate. This process is called the ACME (Automatic Certificate Management Environment) challenge. There are multiple ways you can prove your ownership, but mainly the DNS-01 challenge and HTTP-01 challenge.

If you're using either the Apache or standalone authenticator, you are using http-01 challenges that use files in webroot/.well-known/acme-challenge/ that are created by certbot. The standalone authenticator spins up its webserver so your basic authentication wouldn't affect it. The Apache authenticator creates a port 80 exception and doesn't utilize your site's content. You could have used the webroot authenticator instead of the Apache authenticator to avoid having certbot interfere with your Apache operation.

This is the most common challenge type today. Let’s Encrypt gives a token to your ACME client, and your ACME client puts a file on your web server at **http://\<YOUR\_DOMAIN>/.well-known/acme-challenge/**. That file contains the token, plus a thumbprint of your account key. Once your ACME client tells Let’s Encrypt that the file is ready, Let’s Encrypt tries retrieving it (potentially multiple times from multiple vantage points). If our validation checks get the right responses from your web server, the validation is considered successful and you can go on to issue your certificate. If the validation checks fail, you’ll have to try again with a new certificate.

The HTTP-01 challenge can only be done on port 80. Allowing clients to specify arbitrary ports would make the challenge less secure, and so it is not allowed by the ACME standard.

#### Reference Links:

* [https://eff-certbot.readthedocs.io/en/latest/using.html#getting-certificates-and-choosing-plugins](https://eff-certbot.readthedocs.io/en/latest/using.html#getting-certificates-and-choosing-plugins)
* [https://letsencrypt.org/docs/challenge-types/](https://letsencrypt.org/docs/challenge-types/)
* [https://medium.com/@samson\_sham/setup-lets-encrypt-https-server-fa54abff688#](https://medium.com/@samson_sham/setup-lets-encrypt-https-server-fa54abff688)
* [https://datatracker.ietf.org/doc/html/draft-ietf-acme-acme-16](https://datatracker.ietf.org/doc/html/draft-ietf-acme-acme-16)
