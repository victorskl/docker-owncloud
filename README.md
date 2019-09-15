# docker-owncloud

### TL;DR

```
git clone https://github.com/victorskl/docker-owncloud.git owncloud
cd owncloud

cp env.sample .env
cp docker-compose.override.yml.sample docker-compose.override.yml

docker-compose up -d
docker-compose ps
docker-compose down
```

### Reverse Proxy

#### HAProxy
```
haproxy -v
HA-Proxy version 1.5.18 2016/05/10

vi /etc/haproxy/haproxy.cfg

    frontend http
        bind *:80
        default_backend redirect-https
    
    backend redirect-https
        reqadd X-Forwaded-Proto:\ http
        redirect scheme https code 301 if !{ ssl_fc }
    
    frontend https
        bind *:443 ssl crt /etc/haproxy/certs/example.com.pem
        http-request set-header X-Forwarded-Host %[req.hdr(Host)]
        reqadd X-Forwaded-Proto:\ https
        http-response set-header Strict-Transport-Security max-age=15552000;\ includeSubDomains;\ preload;
        default_backend owncloud
    
    backend owncloud
       server oc1 127.0.0.1:8080
```

#### Config

```
docker network inspect owncloud_default | grep Subnet
                    "Subnet": "172.26.0.0/16",

vi data/owncloud/data/config/config.php

    'trusted_domains' => array (0 => 'localhost', 1 => 'cloud.example.com'),
    'trusted_proxies' => array('172.26.0.0/16'),
    'forwarded_for_headers' => array('HTTP_X_FORWARDED', 'HTTP_FORWARDED_FOR'),
```

## REF

- https://hub.docker.com/r/owncloud/server
- https://github.com/owncloud-docker/server
- https://doc.owncloud.com/server/admin_manual/installation/docker/
- https://doc.owncloud.com/server/admin_manual/configuration/server/reverse_proxy_configuration.html
