
# Let's Encrypt Certbot

Certbot handles LE certificates from the `certbot/certbot` image. There's also a systemd timer that checks daily for renewals.

Certificates are currently used for websites and the Murmur service.


## Installation

Copy the contents of `systemd` into `/etc/systemd` and start the auto-renewal timer service:

```
systemctl enable certbot.timer
systemctl start certbot.timer
```


## Adding a New Domain

The `nginx` service is preconfigured to respond to ACME challenges on http, as long as you mount the appropriate path in certbot.


### Issue Staging Certificates

Use a separate `/STAGING` path on the host machine to store all the Certbot outputs, and run the following (assuming the `nginx` service is running):

```
docker run -it --rm \
    -v /STAGING/letsencrypt/etc:/etc/letsencrypt \
    -v /STAGING/letsencrypt/lib:/var/lib/letsencrypt \
    -v /STAGING/letsencrypt/log:/var/log/letsencrypt \
    -v /srv/infrastructure/public/letsencrypt:/data/letsencrypt \
    certbot/certbot \
        certonly --webroot \
        --register-unsafely-without-email --agree-tos \
        --webroot-path=/data/letsencrypt \
        --staging \
        -d domain.com \
        -d sub.domain.com
```

And then verify them with:

```
docker run --rm -it \
    -v /STAGING/letsencrypt/etc:/etc/letsencrypt \
    -v /STAGING/letsencrypt/lib:/var/lib/letsencrypt \
    -v /srv/infrastructure/public/letsencrypt:/data/letsencrypt \
    certbot/certbot \
        --staging \
        certificates
```

If you want to do some browser testing, mount `/STAGING/letsencrypt/etc` to the Nginx service and reference the live certificates from within your server block.

>Know that the certificate authority for staging certs isn't (and shouldn't be) trusted in browsers.

When testing is done, just nuke `/STAGING`.


### Issue Production Certificates

Run the following to add certificates to our production path for Let's Encrypt (Matching `):

```
docker run -it --rm \
    -v /etc/letsencrypt:/etc/letsencrypt \
    -v /var/lib/letsencrypt:/var/lib/letsencrypt \
    -v /var/log/letsencrypt:/var/log/letsencrypt \
    -v /srv/infrastructure/public/letsencrypt:/data/letsencrypt \
    certbot/certbot \
        certonly --webroot \
        --email you@domain.com --agree-tos --no-eff-email \
        --webroot-path=/data/letsencrypt \
        -d domain.com \
        -d sub.domain.com
```


### Add to Nginx

Add your `server` block to `nginx/conf/http.conf`:

```
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name domain.com;

    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
}
```

Most of the SSL configuration rules are already in `nginx/conf/nginx.conf` to be universal across domains.


### Run SSLLabs Test

Run the [SSL Server Test](https://www.ssllabs.com/ssltest/) and fix any Nginx configuration issues.


### Testing Renewals

You can test the renewal process by running `systemctl start certbot.service`. Note that it will take a random amount of seconds before executing and will send nothing to stdout. You can check `/var/log/letsencrypt/letsencrypt.log` for the actual status updates.

## Current Issues

I don't have a method yet for automatically reloading certificates in Murmur. See https://github.com/mumble-voip/mumble/issues/1972 for discussion on the topic.

