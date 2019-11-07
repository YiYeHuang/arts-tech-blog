---
layout: post
title:  "Secure your website"
date:   2019-11-07 15:59:53 -0700
categories: tips
tag: [security, https]
---

## Tips
For blogs that is hosted on github page or medium, https is taken for granted. With your own ec2 instance or on prime machines, here is the tips for setting up your https certificate with [Let's Encrypt](https://letsencrypt.org/). ~~if you are using google app engine, forget about this post.~~

### Prerequisites:
- having your site ready, of course.
- already have a domain

[reference && tips for prerequisites](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)

### Setup nginx
- install nginx
- setup /etc/nginx/sites-available/test.com
- enable network and ports

example nginx site config:
```
server {
    listen 80;
    listen [::]:80;
    server_name test.com www.test.com;
}
```
[reference && tips for setup nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-18-04)

### Setup certificate
- install certbot
- config firewall
- install certificate

[reference && trips for install certificate](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-18-04)