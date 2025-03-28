---
description: >-
  Access Gitlab repo on my network from a VM in another client
  network.                                                                       
  Date: 03/05/2024
---

# GitLab access issue

Initially tried to clone the Gitlab repo. Got error.

{% code overflow="wrap" fullWidth="true" %}
```bash
fatal: unable to access 'https://gitlab.xyx.com/root/abc.git/': Could not resolve host: gitlab.xyz.com

```
{% endcode %}

As it was a DNS resolve issue, tried to look at gitlab.xyz.com's IP address using the nslookup/dig tool in my local system.

```bash
> nslookup gitlab.xyz.com                                                                                INT root@HMECL003857 14:21:54
Server:		127.0.0.53
Address:	127.0.0.53#53

Non-authoritative answer:
Name:	gitlab.xyz.com
Address: 2.3.4.5

```

Once I got its address, I tried to ping and telnet to this address from the VM.

&#x20;  before that check VM's public IP

```bash
> curl ifconfig.me
> dig +short myip.opendns.com @resolver1.opendns.com
```

```bash
> ping 2.3.4.5
PING 2.3.4.5 (2.3.4.5) 56(84) bytes of data.
64 bytes from 2.3.4.5: icmp_seq=1 ttl=111 time=162 ms

> telnet 2.3.4.5 443 (443 port because GitLab is at https)
Trying 2.3.4.5...
Connected to 2.3.4.5.
Escape character is '^]'.

```

This means that the VM can connect to GitLab using an IP address but was having issues with name resolution. So decided to make this entry in the VM **/etc/hosts** file.

To resolve the name resolution issue, you can manually update the VM's **/etc/hosts** file to include the IP address and the corresponding hostname. This can be especially useful in cases where DNS lookups are failing or when you need to override DNS for testing purposes. To do this, use the following steps:

```
vim /etc/hosts

2.3.4.5  gitlab.xyz.com
```

After this entry, we started getting the "unable to access,403 Forbidden" error.

So I tried to wget/curl this link;

<pre class="language-bash"><code class="lang-bash"><strong>> wget https://gitlab.xyz.com/root/abc -vvvv
</strong><strong>
</strong>--2024-05-03 02:29:02--  https://gitlab.xyz.com/root/abc.git
Resolving gitlab.xyz.com (gitlab.xyz.com)... 2.3.4.5
Connecting to gitlab.xyz.com (gitlab.xyz.com)|2.3.4.5|:443... connected.
HTTP request sent, awaiting response... 403 Forbidden
2024-05-03 02:29:02 ERROR 403: Forbidden.

</code></pre>

Even though after adding the hosts file entry, it was resolved, but http request was denied. Most probably IPs are restricted on this GitLab.

So we have to whitelist this VM's public IP  address for this gitlab.
