# nginx-cache-proxy

This enables any HTTP service to employ a front cache to minimize unnecessary calls by employing a front cache with nginx

It also supports

* cors control
* local file serving
* request/response body contents logging
* request latency logging
* websockets support
* automatic localhost TLS certificate
* custom domain certificate configuration
* data throughput limit

See ENVs section for more configuration details.

## Usage

* Create a docker-compose.yml file

```yml
version: '3.7'
services:
  nginx-cache-proxy:
    image: flaviostutz/nginx-cache-proxy
    ports:
      - 8282:80
      - 8443:443
    environment:
      - PROXY_PASS_URL=http://map-demos/
      - SSL_DOMAIN=localhost

  map-demos:
    image: flaviostutz/map-demos
    ports: 
      - 8181:80
    restart: always
```

* Run 'docker-compose up'

* Open 'http://localhost:8282'

* Open network tab in your browser and check for the header "X-Cache-Status"
  * On first call it is meant to be 'MISS' and in the following calls 'HIT'

## ENV configurations

All ENV configurations have the same name as in NGINX documentation. Check http://nginx.org/en/docs/http/ngx_http_proxy_module.html for details

* CORE_SEND_FILE 'on'
* CORS_ALLOW_ORIGIN '*'

* REQUEST_LOG_LEVEL 'basic' - 'basic' logs to stdout basic request info (method, path, headers etc); 'body' logs to stdout the entire request and response body; 'none' silence request log

* PROXY_PASS_URL ''
* PROXY_READ_TIMEOUT '30s'
* PROXY_LIMIT_RATE_BYTES_PER_SECOND '0'
* PROXY_COOKIE_DOMAIN 'www.test.com localhost'

* SERVE_PATH '/_www' - custom http path for serving files under dir /www
* REDIR_FROM_PATH '/_test' - path from which to redirect to serve path so that the request will be handled by a file in this proxy

* CACHE_MAX_SIZE '1g'
* CACHE_KEY_SIZE '10m'
* CACHE_MIN_USES '1'
* CACHE_MAX_IDLE_HOURS '12'
* CACHE_INACTIVE_TIME '24h'
* CACHE_KEY '$scheme$proxy_host$uri$is_args$args'
* CACHE_LOCK 'on'
* CACHE_REVALIDATE 'off'
* CACHE_BYPASS '$http_pragma'
* CACHE_BACKGROUND_UPDATE 'on'
* CACHE_USE_STALE 'error timeout invalid_header updating http_500 http_502 http_503 http_504'
* CACHE_METHODS 'GET HEAD'
* CACHE_BYPASS '$cookie_nocache $arg_nocache'
* UPSTREAM_CACHE_STATUS '$upstream_cache_status'

* SSL_DOMAIN '' - If not empty, this will enable HTTPS on port 443
  * It will use any existing key/certificate at /etc/ssl/private/domain.key and /etc/ssl/certs/domain.crt
  * If you wish to provide valid public certificates, create a new container based on this one and add certificates using 'ADD /domain.key /etc/ssl/private/domain.key' and 'ADD /domain.crt /etc/ssl/certs/domain.crt'
  * If any of these files doesn't exist, it will generate a self signed certificate for this domain during startup

## Building for other architectures

* Install Docker Edge version
  * https://hub.docker.com/editions/community/docker-ce-desktop-mac

```shell
docker buildx create --name mybuilder
docker buildx create --use
docker buildx build --platform linux/amd64,linux/arm64,linux/arm/v7 -t flaviostutz/nginx-cache-proxy:1.7.1 --push .
```

* See more at https://engineering.docker.com/2019/04/multi-arch-images/

