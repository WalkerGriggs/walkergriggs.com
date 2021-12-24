+++
title = "ZNC, the right way"
author = ["Walker Griggs"]
date = 2021-10-13
tags = ["irc"]
draft = false
creator = "Emacs 27.2 (Org mode 9.4.4 + ox-hugo)"
weight = 2009
+++

I've setup [ZNC](https://wiki.znc.in/ZNC) one too many times.

Sometimes I forget it's [riding shotgun](https://en.wikipedia.org/wiki/Riding%5Fshotgun) on a spare droplet heading to the trash heap. Other times, my payment method expires and so too does the instance. Other times I'm too lazy to host it in the cloud at all, so I run it locally. In any case, today I wanted to set up ZNC the right way... for the last time.

I also want to document the process for posterity and stop scouring the web for the same articles time after time.

The TODO list for today:

-   Setup a dedicated domain
-   Provision a dedicated droplet, hosted on [DigitalOcean](https://www.digitalocean.com/)
-   Configure separate listeners for IRC and HTTP traffic
-   Generate an SSL cert with [LetsEncrypt](https://letsencrypt.org/)
-   Setup [Nginx](https://nginx.org/en/) to terminate SSL traffic and proxy to ZNC


## Dedicated domain and droplet {#dedicated-domain-and-droplet}

I'll gloss over the relatively simple steps like [provisioning a droplet](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04), [securing the firewall](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-20-04), [installing ZNC](https://wiki.znc.in/Installation), and [purchasing a domain](https://www.digitalocean.com/community/tutorials/how-to-point-to-digitalocean-nameservers-from-common-domain-registrars).

tldr; I...

1.  Provisioned a droplet.
2.  Purchased a new domain. I opted for a `.chat` TLD because I thought it was appropriate
3.  Directed the registrar to DigitalOcean's nameservers. Consolidating behind a single control panel makes life much easier.
4.  Created an A record with an `irc` subdomain pointing at the IP of my new droplet.

For the remainder of this post, I'll use `irc.example.chat` as my placeholder domain!


## Configuring ZNC {#configuring-znc}

How you configure ZNC is a matter of personal taste. I opt to load fairly standard modules like [chanserver](https://wiki.znc.in/Chansaver), [fail2ban](https://wiki.znc.in/Fail2ban), [log](https://wiki.znc.in/Log), and [identfile](https://wiki.znc.in/Identfile) but feel free to go crazy! One thing that is important to mention though, are the separate listeners.

I created one listener for SSL IRC traffic over 6697 and one listener for non-SSL HTTP traffic over 8080. The web listener has SSL disabled because 1) it's only a self signed cert 2) it's only hosting to `localhost`.

```xml
<Listener listener0>
    AllowIRC = true
    AllowWeb = false
    IPv4 = true
    IPv6 = false
    Port = 6697
    SSL = true
    URIPrefix = /
</Listener>

<Listener listener1>
    AllowIRC = false
    AllowWeb = true
    Host = localhost
    IPv4 = true
    IPv6 = false
    Port = 8080
    SSL = false
    URIPrefix = /
</Listener>
```


## Configuring Nginx {#configuring-nginx}

I'll first preface this section by saying: I'm not an Nginx wizard by any means. In fact, most of this configuration comes from the [Nginx blog](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/) and [Stack Overflow](https://stackoverflow.com/questions/34236949/znc-on-a-subdomain-with-nginx-reverse-proxy).

Before we can generate a certificate, we want to add a basic configuration. I dropped a file in `/etc/nginx/config.d` and create a softlink to `sites-available` and `sites-enabled`.

```bash
touch /etc/nging/config.d/irc.example.chat
ln -s /etc/nginx/config.d/irc.example.chat /etc/nginx/sites-available
ln -s /etc/nginx/config.d/irc.example.chat /etc/nginx/sites-enabled
```

I then edited the parent configuration. Fortunately, it's fairly readable; nginx will proxy all SSL traffic from `irc.example.chat` to our ZNC localhost listener. We can also set a few headers in the process.

```text
server {
    listen      443 ssl http2;
    server_name irc.example.chat;
    access_log  /var/log/nginx/irc.log combined;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header      Host             $host;
        proxy_set_header      X-Real-IP        $remote_addr;
        proxy_set_header      X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header      X-Client-Verify  SUCCESS;
        proxy_set_header      X-Client-DN      $ssl_client_s_dn;
        proxy_set_header      X-SSL-Subject    $ssl_client_s_dn;
        proxy_set_header      X-SSL-Issuer     $ssl_client_i_dn;
        proxy_read_timeout    1800;
        proxy_connect_timeout 1800;
    }
}
```

The `ssl_certificate` configs will be added by `certbot` in the next step. If they aren't added for whatever reason, they should look something like...

```text
ssl_certificate     /etc/letsencrypt/live/irc.example.chat/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/irc.example.chat/privkey.pem;
```


## Generating certs with LetsEncrypt {#generating-certs-with-letsencrypt}

Now the fun part, and the reason to setup the domain in the first place. I used the [EFF's](https://www.eff.org/) handy [certbot](https://certbot.eff.org/) with Nginx drivers to provision a cert with LetsEncrypt. Technically the Nginx drivers aren't necessary -- you could provision the certs directly -- but the added config editor is a nice feature.

`certbot` took care of just about everything!

```bash
sudo apt-get install certbot python3-certbot-nginx

certbot --nginx -d irc.example.chat
```

I say "just about" because these certs still expire every 90 days. I'm guaranteed to forget about the cert, so I set a cron job (`sudo crontab -e`) to renew the cert every week.

```text
0 0 * * 0 certbox renew --quiet
```


## Configuring Weechat {#configuring-weechat}

The last step of any ZNC install is to setup your client. I use [Weechat](https://weechat.org/), so the next steps may be different for you.

Weechat needs to validate ZNC's SSL cert to connect over `6697`, so grab the SSL certificate fingerprint from the droplet first.

```bash
cat ~/.znc/znc.pem \
    | openssl x509 -sha512 -fingerprint -noout \
    | tr -d ':' \
    | tr 'A-Z' 'a-z' \
    | cut -d = -f 2
```

On the weechat client, I added the `ZNC` server with a default network, set the fingerprint, connected, and saved my changes. One detail that I forget constantly: these creds aren't your network creds, they're your ZNC creds.

```text
/server add ZNC irc.example.chat/6697 -ssl -username=username/network -password=password
/set irc.server.ZNC.ssl_fingerprint <fingerprint>
/connect ZNC
/save
```

Most networks require you to authenticate with SASL these days, which I set through Weechat. Another option is to load the SASL module and set your credentials through the web console.

```text
/msg *Status LoadMod sasl
/msg *SASL Set nick pass
/msg *SASL RequireAuth true
```

And that's about it. We've setup the A record for our domain, configured separate HTTP and IRC listeners for ZNC, generated an SSL cert through LetsEncrypt, proxied web traffic to ZNC with Nginx, and connected securely with Weechat. A pretty productive afternoon!

If you'd like to chat, you can find me on [libera.chat](https://libera.chat/)!
