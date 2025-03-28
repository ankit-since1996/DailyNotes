---
description: >-
  Error Statement: Mount Azure FileShare on client machine with Alma Linux
  9                                                                                                              
  Date:15/04/2024
---

# CIFS mount

CIFS

CIFS (Common Internet File System) is a protocol for sharing files, printers, and other resources over a network, mainly used by Windows systems. It operates over TCP/IP and historically supported NetBIOS/NetBEUI.

Main features include:

* **File and Printer Sharing**: Enables access and sharing of files and printers.
* **Network Browsing**: Allows users to see and navigate network resources.
* **Security**: Supports authentication and authorization processes.
* **Opportunistic Locking**: Boosts performance through local file caching.

CIFS is now largely replaced by SMB2 and SMB3 for better performance and security.

**Mount Azure FileShare on the client machine with Alma Linux:**

Check access from the server:

{% code overflow="wrap" fullWidth="false" %}
```bash
curl -X GET -H "x-ms-version: 2019-07-07" "https://<storage-account-name>.blob.core.windows.net/<container-name>?restype=container&comp=list&<sas-token>"

```
{% endcode %}



#### Reference:

* [https://chat.openai.com/share/77b8c4c0-381c-488c-8d77-2eda7c36379c](https://chat.openai.com/share/77b8c4c0-381c-488c-8d77-2eda7c36379c)
*

