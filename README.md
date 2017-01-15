#Let's Encrypt Certbot
Get a signed SSL certificate from [Let's Encrypt](https://letsencrypt.org) with one command.

#Usage

##Issuing a new certificate:

```shell
docker run -it --rm \
	-p 80:80 -p 443:443 \
	-e DOMAINS="example.com www.example.com" \
	-v $(pwd)/out:/etc/letsencrypt \
	mujz/lets-encrypt
```

You'll be asked to enter your email address and agree to the terms of service next. The cert files will then be generated for you inside the `out` directory. Make sure you set your own domains instead of the "example" ones. 

##Renewing an existing certificate:

```shell
docker run --rm -v $(pwd)/out:/etc/letsencrypt mujz/lets-encrypt certbot renew
```

This will renew your certificate for you if it is due for renewal.
You can also set up a cron job so you don't have to do it manually every 3 months. To do this in Ubuntu, for example, you can run:

```
30 2 * * 1 /usr/bin/docker run --rm -v <local_lets_encrypt_dir>:/etc/letsencrypt mujz/lets-encrypt certbot renew >> /var/log/le_renew.log
35 2 * * 1 /usr/bin/docker restart <server_container>
```

#How it works

Let's start by disecting the command above:

- `docker run --rm -it` runs an interactive docker container that will be deleted once it is stopped.
- `-p 80:80 -p 443:443` maps the host ports 80 and 443 to those of the container.
- `-e DOMAINS="example.com www.example.com"` sets the container's environment variable `DOMAINS` to the domains you want to get the cert for. The certbot looks at this variable to generate the certs.
- `-v $(pwd)/out:/etc/letsencrypt/live` mounts the local directory "out" onto the container's "/etc/letsencrypt/live", which is where the cert files will be generated.
- `mujz/lets-encrypt` tells docker to run this image.

The image is built by installing certbot on `debian:jessie`. When the run command is executed, we first check if the `DOMAINS` env variable was set and then run `certbot certonly --standalone`, which starts a local server on the ports 80 and 443 and generates the certificate.

