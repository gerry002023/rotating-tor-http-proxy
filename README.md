[![Docker Pulls](https://img.shields.io/docker/pulls/zhaowde/rotating-tor-http-proxy.svg)](https://hub.docker.com/r/zhaowde/rotating-tor-http-proxy/)

# rotating-tor-http-proxy

This Docker image provides one HTTP proxy endpoint with many IP addresses for use scenarios like crawling.

Technically, it has an HAProxy sitting in front of multiple pairs of Privoxy and Tor client. The HAProxy dispatches the incoming requests
to the Privoxy with a round-robin strategy. 

## Usage

### Simple case
```shell
docker run --rm -p 3128:3128 zhaowde/rotating-tor-http-proxy
```
At the host, `127.0.0.1:3128` is the HTTP/HTTPS proxy address.

### Moreover

```shell
docker run --rm -p 3128:3128 -p 4444:4444 -e "TOR_INSTANCES=5" -e "TOR_REBUILD_INTERVAL=3600" zhaowde/rotating-tor-http-proxy
```

Port `4444/TCP` can be mapped to the host if HAProxy stats information is needed. With `docker run -p 4444:4444`, the HAProxy statistics
report is available at http://127.0.0.1:4444.  An [article](https://www.haproxy.com/blog/exploring-the-haproxy-stats-page/) from the
HAProxy official blog explains in detail how to understand this report.

Environment variable `TOR_INSTANCES` can be used to config the number of concurrent Tor clients (as well as the associated Privoxy 
instances). The default is 10, and the valid value is purposely limited to the range between 1 and 40. 

Each Tor client attempts to build a new circuit (results in a new outbound IP address) every 30 seconds. Every 30 minutes, this image
rebuild all the circuits. This interval can be changed with environment variable `TOR_REBUILD_INTERVAL`, the default value is 1800
seconds, while it can be set up any number greater than 600 seconds.

### Test the proxy

```shell
while :; do ; curl -sx localhost:3128 ifconfig.me; echo ""; sleep 2; done
```

## Credit

At Github there are many repos build Docker image to provide HTTP proxy connects to the Tor network. The project is reinventing the wheel
based on many of them.
Remarkably:
- [y4ns0l0/docker-multi-tor](https://github.com/y4ns0l0/docker-multi-tor) creates a setup with multiple pairs of Privoxy-Tor. Having no
  HAProxy-like dispatcher, each Privoxy expose itself to the host as a different TCP port.
- [mattes/rotating-proxy](https://github.com/mattes/rotating-proxy) does exactly the same job as this project. However,
    1. it utilizes [Polipo](https://www.irif.fr/~jch/software/polipo/) as the HTTP-SOCKS proxy adapter. Polipo ceased to be maintained on
       6 November 2016
    2. the base image is Ubuntu 14.04, which it too heavy for this case, and out-of-maintenance as well
    3. the main control logic is written in Ruby

## Bill-of-Material

- alpine 3.12.0
- curl-7.69.1
- haproxy-2.1.11
- privoxy-3.0.28
- sed-4.8
- tor-0.4.3.7