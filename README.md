
# Default commands

- ifconfig
- mkdir /var/lib/consul
- mkdir /etc/consul.d
- consul agent -server -bootstrap-expect=<total_nodes> -node=<server_name> -bind=<ip_here> -data-dir=/var/lib/consul -config-dir=/etc/consul.d

# Servers nodes commands

## Server 01

- docker exec -it consulserver01 sh
- ifconfig
- mkdir /var/lib/consul
- mkdir /etc/consul.d
- consul agent -server -bootstrap-expect=3 -node=consulserver01 -bind=$(ifconfig eth0 | awk '/inet addr/{print substr($2,6)}') -data-dir=/var/lib/consul -config-dir=/etc/consul.d 

## Server 02

- docker exec -it consulserver02 sh
- ifconfig
- mkdir /var/lib/consul
- mkdir /etc/consul.d
- consul agent -server -bootstrap-expect=3 -node=consulserver02 -bind=$(ifconfig eth0 | awk '/inet addr/{print substr($2,6)}') -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=consulserver01

## Server 03

- docker exec -it consulserver03 sh
- ifconfig
- mkdir /var/lib/consul
- mkdir /etc/consul.d
- consul agent -server -bootstrap-expect=3 -node=consulserver03 -bind=$(ifconfig eth0 | awk '/inet addr/{print substr($2,6)}') -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=consulserver01

## Join Servers

- docker exec -it consulserver01 sh
- consul join consulserver02
- consul join consulserver03

# Clients nodes commands

## Client 01

- docker exec -it consulclient01 sh
- ifconfig
- mkdir /var/lib/consul
- consul agent -node=consulclient01 -bind=$(ifconfig eth0 | awk '/inet addr/{print substr($2,6)}') -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=consulserver01
- apk -U add bind-tools
- dig @localhost -p 8600 nginx.service.consul
- curl localhost:8500/v1/catalog/services
- curl localhost:8500/v1/catalog/nodes
- consul catalog nodes -service nginx
- consul catalog nodes -detailed
- dig @localhost -p 8600 web.nginx.service.consul
- apk add nginx
- mkdir /run/nginx
- nginx
- mkdir /usr/share/nginx/html -p # criar a pasta recursivamente
- vi /etc/nginx/conf.d/default.conf

```sh
# This is a default site configuration which will simply return 404, preventing
# chance access to any other virtualhost.

server {
    listen 80 default_server;
    listen [::]:80 default_server;

    root /usr/share/nginx/html;

    # You may need this to prevent return 404 recursion.
    location = /404.html {
        internal;
    }
}
```

- vi /usr/share/nginx/html/index.html

```html
Hello
```

- nginx -s reload
- curl localhost

## Client 02

- docker exec -it consulclient02 sh
- ifconfig
- mkdir /var/lib/consul
- consul agent -node=consulclient02 -bind=$(ifconfig eth0 | awk '/inet addr/{print substr($2,6)}') -data-dir=/var/lib/consul -config-dir=/etc/consul.d -retry-join=consulserver01
- apk -U add bind-tools
- dig @localhost -p 8600 nginx.service.consul
- curl localhost:8500/v1/catalog/services
- curl localhost:8500/v1/catalog/nodes
- consul catalog nodes -service nginx
- consul catalog nodes -detailed
- dig @localhost -p 8600 web.nginx.service.consul

# Utilizando o JSON para iniciar o agente

- consul agent -config-dir=/etc/consul.d

# Testando se os dados estao criptografados

- apk -U add bind-tools
- apk -U add tcpdump
- tcpdump -i eth0 -an port 8301 -A