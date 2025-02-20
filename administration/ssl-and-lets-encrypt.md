---
title: SSL & Let's Encrypt
description: Securing your AzuraCast installation with SSL / HTTPS
published: true
date: 2022-09-12T22:07:00.058Z
tags: administration, docker
editor: markdown
dateCreated: 2021-02-05T19:28:14.682Z
---

# Enabling HTTPS with LetsEncrypt

AzuraCast now includes built-in support for creating and managing SSL (HTTPS) certificates via LetsEncrypt from the System Settings panel.

LetsEncrypt is a free and simple way to allow safe and secure connections to your AzuraCast installation. With a valid SSL certificate, you can:

- Secure your connection to AzuraCast when administering your stations,

- Enforce security for all AzuraCast administrators via HTTP Strict Transport Security (HSTS), and

- Provide a secure listening endpoint to listeners, avoiding "Mixed Content" warnings when your radio signal is played from a secure web page.

## Important Considerations

Before setting up LetsEncrypt, you should make sure the following conditions are met:

- **AzuraCast must be on its own domain or subdomain.** You can't set up LetsEncrypt using only an IP address; you must have a domain (i.e. example.com) or a subdomain (radio.example.com) set up to point to your AzuraCast installation.

- **AzuraCast's web server must be served on the default ports, 80 for HTTP and 443 for HTTPS.** By default, AzuraCast is already set up this way, but if you've modified the ports to serve the site on a secondary port, you must switch the ports back to the defaults when setting up LetsEncrypt and when performing renewals.

# Enabling LetsEncrypt

To enable LetsEncrypt, follow these steps:

- Log in to your AzuraCast installation
- Click the dropdown on the top right, then "System Administration"
- Click "System Settings"
- Select the "Services" tab
- Complete the LetsEncrypt section fields
- Click "Save Changes" at the bottom

The HTTPS certificate will automatically be generated in the next few minutes, but you can do it manually by clicking the "Create/Renew Certificate" button under the LetsEncrypt fields.

> If users are still having issues with audio not playing, please ensure you have the "Use Web Proxy" option enabled in your System Settings. 
{.is-info}

## Renewing a Let's Encrypt Certificate

The web service will automatically renew your LetsEncrypt certificates. If you provide an e-mail in the initial setup process, that e-mail will be used to send you reminders of upcoming expiration in the event that automatic renewal fails.

# Using a Custom Certificate

If you have a custom SSL certificate on your host, you should create a `docker-compose.override.yml` file in your `/var/azuracast` directory on the host server with the contents below, modified to reflect your domain name and the path to your SSL certificate and key:


### By Version {.tabset}
#### Stable Release Version (0.16.0) and newer
```
services:
  web:
    volumes:
      - /path/on/host/to/ssl.crt:/var/azuracast/acme/ssl.crt:ro
      - /path/on/host/to/ssl.key:/var/azuracast/acme/ssl.key:ro
```

#### Stable Version 0.15.2 and Older
```
services:
  web:
    volumes:
      - /path/on/host/to/ssl.crt:/etc/nginx/certs/ssl.crt:ro
      - /path/on/host/to/ssl.key:/etc/nginx/certs/ssl.key:ro
      
  stations:
    volumes:
      - /path/on/host/to/ssl.crt:/etc/nginx/certs/ssl.crt:ro
      - /path/on/host/to/ssl.key:/etc/nginx/certs/ssl.key:ro
```

> Please note that Icecast expects an RSA private key as well as a certificate file with the complete certificate chain. For custom certificates in the `.pem` format generated by something like `Certbot` you will need to convert them like this:
> `openssl rsa -in privkey.pem -out example.com.key`
> `openssl crl2pkcs7 -nocrl -certfile fullchain.pem | openssl pkcs7 -print_certs -out example.com.crt`
> {.is-info}

Finally you need to restart AzuraCast in order to apply the changes:

```bash
docker-compose down
docker-compose up -d
```