---
title: "Using nginx with self-signed certificates to encrypt local communications"
date: 2024-06-07T20:00:00+02:00
description: Learn how to use self-signed certs in an example with a self-hosted Gitlab server.
draft: false
tags: [tutorial, linux, raspberrypi, gitlab, nginx]
---

## Introduction

When you run local servers on your Raspberry Pi, you by default only get unencrypted http communications which makes it trivial for anyone in your LAN to snoop the traffic. This post is a brief tutorial how to get free encrypted https communications without having to purchase a real domain name. I use the example of a self-hosted Gitlab server as this was my actual use-case, but this tutorial should be useful even if you do not care about the Gitlab part.

## Assumptions

* You have a Raspberry Pi running their official OS
* You have Gitlab installed and running already (but it still uses http)

## Installing nginx
```bash
sudo apt install nginx
```
and then to create the directory where we will be putting our self-signed certificate
```bash
mkdir /etc/nginx/certs
```

## Getting a self-signed certificate
Just run the following command:
```bash
sudo openssl req -x509 -nodes -days 9999 -newkey rsa:4096 -keyout /etc/nginx/certs/selfSignedCert.key -out /etc/nginx/certs/selfSignedCert.crt
```
It doesn't matter which values you put in the fields you are asked to fill. Congrats, now you have an RSA certificate that is valid for 9999 days. 

## Configuring nginx
Let's create our custom config called "mylocalserver" with:
```bash
sudo nano /etc/nginx/sites-available/mylocalserver
```
Here, put the following content:
```text
server {
    listen 80;
    server_name _;

    location /gitlab { # if you want server to only be reachable via https://<localIP>/gitlab
        return 301 https://$host$request_uri;
    }

        # just add another location block if you have another service (must have new location path)
}

server {
    listen 443 ssl;
    server_name _;

    ssl_certificate /etc/nginx/certs/selfSignedCert.crt;
    ssl_certificate_key /etc/nginx/certs/selfSignedCert.key;

    location /gitlab { # if you want server to only be reachable via https://<localIP>/gitlab
        proxy_pass http://127.0.0.1:8084;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

        # just add another location block if you have another service (change location path and port to reflect actual)
}
```
Also create a symbolic link by running:
```bash
sudo ln -s /etc/nginx/sites-available/mylocalserver /etc/nginx/sites-enabled/
```
If you now test the current config using
```bash
sudo nginx -t
```
it should say success.

Ok, so now enable nginx at startup and also restart it now to apply new config:
```bash
sudo systemctl enable nginx

sudo systemctl restart nginx
```

In this config, we have specified that when we access
```text
https://<localIP>/gitlab
```
in the browser we should be encrypting all traffic with the self-signed cert we have created and be forwarded to Gitlab which is running at port 8084. For Gitlab specifically, you now also need to adjust your gitlab.rb by running 
```bash
sudo nano /etc/gitlab/gitlab.rb
```
Here you want to have the following three lines (external_url already exists and must be adjusted, the other two lines can just be added):
```text
external_url 'https://localhost/gitlab' # adding the /gitlab part is important as this is how it was specified in the nginx config
nginx['listen_port'] = 8084 			# must match port in nginx config
nginx['listen_https'] = false 			# avoid gitlab clashing with nginx, this does not mean that the traffic is not encrypted
```
Now apply the new Gitlab config by running
```bash
sudo gitlab-ctl reconfigure
```
Congrats, if you wait a minute or two you can now visit your Gitlab in the browser at the address:
```text
https://<localIP>/gitlab
```
You will get a warning (because the certificate is self-signed), so just once tell the browser to ignore the warning and continue.

## Conclusion
I wrote the tutorial because when searching online how to encrypt local traffic you get the craziest suggestions, ranging from "you should pay for an actual domain name" to "you should use this shady free service that does it for free". None of these things are necessary, the nginx approach is simple, and the only downside is that one time you have to tell your browser to ignore a warning. I am happy with my free setup, and it is nice that I do not have to rely on some third-party.