[linuxserverurl]: https://linuxserver.io
[forumurl]: https://forum.linuxserver.io
[ircurl]: https://www.linuxserver.io/irc/
[podcasturl]: https://www.linuxserver.io/podcast/
[appurl]: https://letsencrypt.org/
[hub]: https://hub.docker.com/r/lsioarmhf/letsencrypt-aarch64/

[![linuxserver.io](https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/linuxserver_medium.png)][linuxserverurl]

The [LinuxServer.io][linuxserverurl] team brings you another container release featuring easy user mapping and community support. Find us for support at:
* [forum.linuxserver.io][forumurl]
* [IRC][ircurl] on freenode at `#linuxserver.io`
* [Podcast][podcasturl] covers everything to do with getting the most from your Linux Server plus a focus on all things Docker and containerisation!

# lsioarmhf/letsencrypt-aarch64
[![](https://images.microbadger.com/badges/version/lsioarmhf/letsencrypt-aarch64.svg)](https://microbadger.com/images/lsioarmhf/letsencrypt-aarch64 "Get your own version badge on microbadger.com")[![](https://images.microbadger.com/badges/image/lsioarmhf/letsencrypt-aarch64.svg)](http://microbadger.com/images/lsioarmhf/letsencrypt-aarch64 "Get your own image badge on microbadger.com")[![Docker Pulls](https://img.shields.io/docker/pulls/lsioarmhf/letsencrypt-aarch64.svg)][hub][![Docker Stars](https://img.shields.io/docker/stars/lsioarmhf/letsencrypt-aarch64.svg)][hub][![Build Status](https://ci.linuxserver.io/buildStatus/icon?job=Docker-Builders/arm64/arm64-letsencrypt)](https://ci.linuxserver.io/job/Docker-Builders/job/arm64/job/arm64-letsencrypt/)

This container sets up an Nginx webserver and reverse proxy with php support and a built-in letsencrypt client that automates free SSL server certificate generation and renewal processes. It also contains fail2ban for intrusion prevention.

[![letsencrypt](https://raw.githubusercontent.com/linuxserver/docker-templates/master/linuxserver.io/img/le-logo-wide.png)][appurl]

## Usage

```
docker create \
  --cap-add=NET_ADMIN \
  --name=letsencrypt \
  -v <path to data>:/config \
  -e PGID=<gid> -e PUID=<uid>  \
  -e EMAIL=<email> \
  -e URL=<url> \
  -e SUBDOMAINS=<subdomains> \
  -e VALIDATION=<method> \
  -p 80:80 -p 443:443 \
  -e TZ=<timezone> \
  lsioarmhf/letsencrypt-aarch64
```

## Parameters

`The parameters are split into two halves, separated by a colon, the left hand side representing the host and the right the container side. 
For example with a port -p external:internal - what this shows is the port mapping from internal to external of the container.
So -p 8080:80 would expose port 80 from inside the container to be accessible from the host's IP on port 8080
http://192.168.x.x:8080 would show you what's running INSIDE the container on port 80.`


* `-p 80 -p 443` - the port(s)
* `-v /config` - all the config files including the webroot reside here
* `-e URL` - the top url you have control over ("customdomain.com" if you own it, or "customsubdomain.ddnsprovider.com" if dynamic dns)
* `-e SUBDOMAINS` - subdomains you'd like the cert to cover (comma separated, no spaces) ie. `www,ftp,cloud`. For a wildcard cert, set this _exactly_ to `wildcard` (wildcard cert is available via dns validation only)
* `-e VALIDATION` - letsencrypt validation method to use, options are `http`, `tls-sni` or `dns` (dns method also requires `DNSPLUGIN` variable set)
* `-e PGID` for GroupID - see below for explanation
* `-e PUID` for UserID - see below for explanation
* `-e TZ` - timezone ie. `America/New_York`  
  
_Optional settings:_
* `-e DNSPLUGIN` - required if `VALIDATION` is set to `dns`. Options are `cloudflare`, `cloudxns`, `digitalocean`, `dnsimple`, `dnsmadeeasy`, `google`, `luadns`, `nsone`, `rfc2136` and `route53`. Also need to enter the credentials into the corresponding ini file under `/config/dns-conf` 
* `-e EMAIL` - your e-mail address for cert registration and notifications
* `-e DHLEVEL` - dhparams bit value (default=2048, can be set to `1024` or `4096`)
* `-p 80` - Port 80 forwarding is required when `VALIDATION` is set to `http`, but not `dns` or `tls-sni`
* `-e ONLY_SUBDOMAINS` - if you wish to get certs only for certain subdomains, but not the main domain (main domain may be hosted on another machine and cannot be validated), set this to `true`
* `-e EXTRA_DOMAINS` - additional fully qualified domain names (comma separated, no spaces) ie. `extradomain.com,subdomain.anotherdomain.org`
* `-e STAGING` - set to `true` to retrieve certs in staging mode. Rate limits will be much higher, but the resulting cert will not pass the browser's security test. Only to be used for testing purposes.
* `-e HTTPVAL` - Deprecated, please use the `VALIDATION` parameter instead

_Important notice:_
* This image previously used tls-sni validation over port 443. However, due to a security vulnerability, letsencrypt disabled tls-sni validation: https://community.letsencrypt.org/t/2018-01-11-update-regarding-acme-tls-sni-and-shared-hosting-infrastructure/50188 If you are getting the following error in the log, that means you are attempting tls-sni authentication, which is disabled by the servers: `Client with the currently selected authenticator does not support any combination of challenges that will satisfy the CA.` Please set the `VALIDATION` parameter to either `http` or `dns` and follow the above directions to revalidate. 

It is based on alpine linux with s6 overlay, for shell access whilst the container is running do `docker exec -it letsencrypt /bin/bash`.

### User / Group Identifiers

Sometimes when using data volumes (`-v` flags) permissions issues can arise between the host OS and the container. We avoid this issue by allowing you to specify the user `PUID` and group `PGID`. Ensure the data volume directory on the host is owned by the same user you specify and it will "just work" ™.

In this instance `PUID=1001` and `PGID=1001`. To find yours use `id user` as below:

```
  $ id <dockeruser>
    uid=1001(dockeruser) gid=1001(dockergroup) groups=1001(dockergroup)
```

## Setting up the application
`IMPORTANT... THIS IS THE ARM64 VERSION`

* Before running this container, make sure that the url and subdomains are properly forwarded to this container's host, and that port 443 (and/or 80) is not being used by another service on the host (NAS gui, another webserver, etc.).
* For `http` validation, port 80 on the internet side of the router should be forwarded to this container's port 80
* For `tls-sni` validation, port 443 on the internet side of the router should be forwarded to this container's port 443
* `--cap-add=NET_ADMIN` is required for fail2ban to modify iptables
* If you need a dynamic dns provider, you can use the free provider duckdns.org where the url will be `yoursubdomain.duckdns.org` and the subdomains can be `www,ftp,cloud`
* The container detects changes to url and subdomains, revokes existing certs and generates new ones during start. It also detects changes to the DHLEVEL parameter and replaces the dhparams file.
* If you'd like to password protect your sites, you can use htpasswd. Run the following command on your host to generate the htpasswd file `docker exec -it letsencrypt htpasswd -c /config/nginx/.htpasswd <username>`


## Info

* Shell access whilst the container is running: `docker exec -it letsencrypt /bin/bash`
* To monitor the logs of the container in realtime: `docker logs -f letsencrypt`

* container version number 

`docker inspect -f '{{ index .Config.Labels "build_version" }}' letsencrypt`

* image version number

`docker inspect -f '{{ index .Config.Labels "build_version" }}' lsioarmhf/letsencrypt-aarch64`

## Versions

+ **13.03.18:** Support for wildcard cert with dns validation added. Switched to v2 api for ACME.
+ **21.02.18:** Reduce shellcheck directives by renaming secondary variables
+ **20.02.18:** Sanitize variables, increase log verbosity
+ **01.21.18:** Fix repos in info, bring up to date with x86-64 and armhf
+ **31.01.18:** Initial release
