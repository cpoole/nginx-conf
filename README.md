nginx-conf
==============

This image is the dynamic confd and rancher based sidekick for nginx.

## Build

```
docker build -t connorpoole/nginx-conf:<version> .
```

## Usage

This repo contains the docker-compose and rancher-compose files needed to deploy this as a stack to rancher. This repo also contains a dockerfile to build the nginx-conf sidekick. This image has to be run as a sidekick of traefik, and makes the /etc/traefik directory as a volume. It scans from rancher-metadata, for services that have traefik labels, and generates traefik frontends and backends to expose the services.

## Configuration labels

The following labels can be be added to your service.

* nginx.enable = *service | stack | manual | tcp*
    * true: the service will be published as *service_name.stack_name.nginx_tld*
    * stack: the service will be published as *stack_name.nginx_tld*. WARNING: You can have collisions inside services within your stack
    * manual: the service frontend will be published as the labels *nginx_fqdn*
    * tcp: the service will be published as a layer 4 stream on a specific port
* nginx.tld= *Top level domain names to route rules. Multiple domains separated by ","*
    * **OPTIONAL:** not required if you are in tcp or manual mode
* nginx.layer4 = a list of all layer4 upstream servers
    * uses the format `protocol @ proxy protocol true or false: port nginx and your service listen on` 
    * the only valid protocols are `tcp` or `udp`
    * For example: I have a container with a tcp service listening on port 5000 whose listener is not expecting the PROXYPROTOCOL headers. 
        * the `nginx.layer4` label has the value `tcp@ppfalse:5000`
        * this will add a TCP listener to your nignx containers on port 5000 that forwards TCP traffic to your container(s) on port 5000
    * For example: I have a container with a tcp service listening on port 5000 whose listener is expecting the PROXYPROTOCOL headers. And the container has a tcp processes listening on port 5001 that is not expecting the PROXYPROTOCOL headers:
        * the `nginx.layer4` label has the value `tcp@pptrue:5000,tcp@ppfalse:5001`
        * this will add TCP listeners to your nignx containers on port 5000 and 5001 that forward TCP traffic to your container(s) on port 5000 and 5001 with traffic to port 5000 having the PROXYPROTOCOL header injected while traffic to 5001 is left plain.
* nginx.layer7 = the configuration of your layer7 listener
    * uses the format `protocol @ proxy protocol true or false: port nginx and your service listen on` 
    * currently the container only supports one hostname per backend container
    * the only valid protocols are `http`, `https`, `ws`, or `wss`
    * For example: I have a container that should recieve http traffic on port 5000 without the PROXYPROTOCOL headers.
        * the `nginx.layer7` label has the value `http@ppfalse:5000`
        * this will forward all traffic that reaches nginx on layer 7 with the hostname given by `nginx.fqdn` to your backend container via http protocol on port 5000.
* nginx.fqdn= *Fully Qualified Domain Name to route rules. Multiple domains separated by ",". Be careful, collisions are possible!*
    * **OPTIONAL:** only required if setting nginx.enable to manual. This label sets the fqdn



WARNING: Only services with healthy state are added to nginx, so health checks are mandatory.

The primary config file is also included in this image and volume mounted into nginx.
The following environment variables are required:
* CONFD_INTERVAL | the number of seconds between confd polling the rancher metadata. int only, no characters
* CONFD_PREFIX | the rancher metadata version string (ie. latest, 2015-12-19)

The following environment variables can be used to override the defaults:
* NGINX_LOG_LEVEL | default=debug
* NGINX_HTTP_PORT | default=80
* NGINX_HTTPS_PORT | default=443

## Deployment
```rancher-compose up --upgrade -d```

DISCLAIMER: This stack was inspired by @rawmind0's traefik proxy stack in the rancher catalog
