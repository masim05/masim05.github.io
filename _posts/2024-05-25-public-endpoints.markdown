---
layout: post
title:  "Public endpoints setup"
date:   2024-05-25 07:11:46 +0700
categories: crypto
---

## Context
After reviewing my previous posts (eg, [this one](/crypto/2024/05/19/why-tls.html))
I caught myself on thinking that I tend to reveil issues in nodes configuration,
which might be kind of impolite. So, in order to avoid being "an assh*le who is
always highlighting the problems", I decided to write a guide about public
endpoints setup.

## Plan
I assume you have already running node, it is hosted on a server with public IP.

The roadmap is the following:
 - Enable APIs
 - Setup DNS
 - Setup Nginx: upstreams
 - Setup Nginx: TLS

### Enable APIs
Cosmos based validating nodes usually provide several APIs:
 - REST
 - RPC
 - gRPC

#### Enable REST API
This endpoint is controlled by `config/app.toml` configuration
file, `API Configuration` section:
```
$ grep -B 1 -A 12 "API Configuration"  ./config/app.toml
###############################################################################
###                           API Configuration                             ###
###############################################################################

[api]

# Enable defines if the API server should be enabled.
enable = true

# Swagger defines if swagger documentation should automatically be registered.
swagger = true

# Address defines the API server to listen on.
address = "tcp://localhost:1317"

```
Make sure the following parameter is set to true: `enable`. You can also
enable swagger, it provides documentation for the API and ability to perform
requests from the browser UI. If you host several nodes on the same server,
you need to use different ports in order to avoid conflicts - just replace
1317 in `address` parameter to whatever you want.

After configuration change, restart you node and check API is working:
```
$ sudo systemctl restart <your node service>
$ curl -I localhost:1317 # from the node terminal
HTTP/1.1 200 OK
X-Server-Time: 1716600371
Date: Sat, 25 May 2024 01:26:11 GMT
Content-Length: 860
Content-Type: text/html; charset=utf-8
```

HTTP 200 OK indicates everything is fine.

#### RPC
RPC is managed by `./config/config.toml` configuration file,
`RPC Server Configuration Options` section: 
```
$ grep -B 1 -A 6 "RPC Server Configuration Options" ./config/config.toml
#######################################################
###       RPC Server Configuration Options          ###
#######################################################
[rpc]

# TCP or UNIX socket address for the RPC server to listen on
laddr = "tcp://127.0.0.1:26657"
```

Again, if you want to use custom port, just replace 26657 with proper value.

After any configuration chanhe you should restart your node:
```
$ sudo systemctl restart <your node service>
```
To check RPC endpoint run:
```
$ curl 127.0.0.1:26657 # from the node terminal
<html><body><br>Available endpoints:<br><br>Endpoints that require arguments:
...
```

#### gRPC
Need to work with `./config/app.toml` again:
```
$ grep -B 1 -A 10 "gRPC Configuration" ./config/app.toml 
###############################################################################
###                           gRPC Configuration                            ###
###############################################################################

[grpc]

# Enable defines if the gRPC server should be enabled.
enable = true

# Address defines the gRPC server address to bind to.
address = "localhost:9090"
```

Make sure it is enabled and change the port if needed.

Now we have REST, RPC and gRPC endpoints enabled.

*You might notice that they
all are bound to localhost, so it is not possible to access them via
public node ip. We will fix it later, during Nginx setup.*

### Setup DNS
In order you node was accessible via cool human readable address, DNS should
be setup. It is a very generic topic, so you can learn more about the
service at [wiki](https://en.wikipedia.org/wiki/Domain_Name_System). There are
also many articles about DNS setup:
 - GoDaddy: [buy domain](https://www.godaddy.com/en-sg/domains),
 [add A record](https://sg.godaddy.com/help/add-an-a-record-19238)
 - Cloudflare: [register domain](https://www.cloudflare.com/products/registrar/),
 [manage DNS records](https://developers.cloudflare.com/dns/manage-dns-records/how-to/create-dns-records/)

It is convenient to use different subdomains for different networks and APIs:
 - [https://rpc.band.crptmax.com](https://rpc.band.crptmax.com/)
 - [https://galactica-t-api.noders.services](https://galactica-t-api.noders.services/)
 - [https://testnet.zero-g.rpc.service.nodebrand.xyz](https://testnet.zero-g.rpc.service.nodebrand.xyz/)
 - gRPC: haqq-mainnet-grpc.itrocket.net:443

### Setup Nginx: upstreams
Nginx is a very powerfull web server, I prefer to proxy all incoming traffic
with it. First, need to install it:
```
$ sudo apt install -y nginx
```
The main configuration file is `/etc/nginx/nginx.conf`. In the `http` section
one can see:
```
	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```
It means one can have several configuration files inside `/etc/nginx/conf.d/`
and `/etc/nginx/sites-enabled/`, they will be included. We will use it to
create config files per domain. Assuming the following subdomains are
pointing to your node's ip: `api.galactica-t.example.com`,
`rpc.galactica-t.example.com` and `grpc.galactica-t.example.com`:
```
sudo tee /etc/nginx/conf.d/galactica-t.example.com > /dev/null << EOF

upstream api-galactica {
    server localhost:1317;
}

server {
    listen              80;
    server_name         api.galactica-t.example.com;
    keepalive_timeout   70;

    location / {
        proxy_pass           http://api-galactica$request_uri;
        proxy_set_header     Host $host;
    }
}

upstream rpc-galactica {
    server localhost:26657;
}

server {
    listen              80;
    server_name         rpc.galactica-t.example.com;
    keepalive_timeout   70;

    location / {
        proxy_pass           http://rpc-galactica$request_uri;
        proxy_set_header     Host $host;
    }

    location /websocket {
       proxy_pass http://rpc-galactica;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
    }
}

upstream grpc-galactica {
    server localhost:9090;
}

server {
    listen              8080;

    location / {
    	grpc_pass grpc://grpc-galactica;
    }
}
EOF
```
Since now public endpoints should be available without TLS:
`http://api.galactica-t.example.com`, `http://rpc.galactica-t.example.com`
and `rpc.galactica-t.example.com:8080`:
```
curl -I http://api.galactica-t.example.com # should return 200 OK
curl http://rpc.galactica-t.example.com/status | jq . # should show json
grpcurl -plaintext grpc.galactica-t.example.com:8080 list # should show list of methods
wscat --connect ws://rpc.galactica-t.example.com/websocket # should show json
> { "jsonrpc": "2.0", "method": "status", "id": 1 }
```

### Setup TLS
The last step of the setup is TLS. In order to enable it we need a valid
certificate. There is a [letsencrypt](https://letsencrypt.org/) project
which can issue a free certificate. This
[article](https://www.f5.com/company/blog/nginx/using-free-ssltls-certificates-from-lets-encrypt-with-nginx)
is amazing guide how to setup Nginx with letsencrypt certificate. Certbot
will also update config files, one should make sure they are fine
(assuming certificate was issued for all domains and located in
`/etc/letsencrypt/live/api.galactica-t.example.com/` folder):
```
$ cat /etc/nginx/conf.d/galactica-t.example.com

upstream api-galactica {
    server localhost:1317;
}

server {
    listen              443 ssl http2;
    server_name         api.galactica-t.example.com;
    keepalive_timeout   70;

    ssl_certificate     /etc/letsencrypt/live/api.galactica-t.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.galactica-t.example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

#    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
#    ssl_ciphers         HIGH:!aNULL:!MD5;
    #...
    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    location / {
        proxy_pass           http://api-galactica$request_uri;
        proxy_set_header     Host $host;
    }
}

upstream rpc-galactica {
    server localhost:26657;
}

server {
    listen              443 ssl http2;
    server_name         rpc.galactica-t.example.com;
    keepalive_timeout   70;

    ssl_certificate     /etc/letsencrypt/live/api.galactica-t.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.galactica-t.example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

#    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
#    ssl_ciphers         HIGH:!aNULL:!MD5;
    #...
    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    location / {
        proxy_pass           http://rpc-galactica$request_uri;
        proxy_set_header     Host $host;
    }

    location /websocket {
       proxy_pass http://rpc-galactica;
       proxy_http_version 1.1;
       proxy_set_header Upgrade $http_upgrade;
       proxy_set_header Connection "upgrade";
    }
}

upstream grpc-galactica {
    server localhost:9090;
}

server {
    listen              9443 ssl http2;

    ssl_certificate     /etc/letsencrypt/live/api.galactica-t.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.galactica-t.example.com/privkey.pem;

    location / {
        grpc_pass grpc://grpc-galactica;
    }
}
```

Since now endpoints should be available with TLS:
```
curl -I https://api.galactica-t.example.com # should return 200 OK
curl https://rpc.galactica-t.example.com/status | jq . # should show json
grpcurl grpc.galactica-t.example.com:9443 list # should show list of methods
wscat --connect wss://rpc.galactica-t.example.com/websocket # should show json
> { "jsonrpc": "2.0", "method": "status", "id": 1 }
```

That's it. REST, RPC (including websocket) and gRPC endpoints are working
over TLS.