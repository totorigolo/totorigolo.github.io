+++
title = "How to setup your own VPN on port 443"
date = 2021-05-23
description = ""

[extra]
extract_from = "content"
+++

A few weeks ago, there has been a major incident in one of OVH's datacenter.
Unfortunately, my two VPS burnt and I didn't have any backup. I didn't lose any
data because I mostly use them to deploy toy projects. However, I did lose years
of configurations. In this article, we will configure a VPN on Debian, with a
slight variation from standard setup to be able to use it even in networks where
non-Web ports are blocked. We will use HTTPS port (443) with a redirection of
non-VPN traffic for potential reverse proxies.

## Step 1: Read Digital Ocean's tutorial

<https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-debian-10>

## Step 2: Configure the VPN over 443

In `/etc/openvpn/server.conf`, change:

```ini
port 443
proto tcp
explicit-exit-notify 0

# This will redirect all non-VPN traffic to this fallback port.
port-share 127.0.0.1 4443
```

## Recap: How to create a new client

1. Generate a new client certificate/key pair.

```bash
# On the OpenVPN server
./easyrsa gen-req <client-name> nopass
cp pki/private/client1.key ~/client-configs/keys/
# transfer key to CA server

# On the CA server
./easyrsa import-req /path/to/<client-name>.req <client-name>
./easyrsa sign-req client <client-name>
# transfer the .crt to the OpenVPN server
```

2. Create the client configuration file.

```bash
./make_config.sh <client-name>
```
