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
* nginx.port = *port to expose throught traefik*
* nginx.protocol = *http | https | ws | wss | tcp* will set your backend protocol.
* nginx.fqdn= *Fully Qualified Domain Name to route rules. Multiple domains separated by ",". Be careful, collisions are possible!*
    * **OPTIONAL:** only required if setting nginx.enable to manual. This label sets the fqdn
* nginx.proxyprotocol= *true | false*
    * **OPTIONAL:** only required if the backend service is expecting incoming traffic with the PROXY_PROTOCOL header



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


