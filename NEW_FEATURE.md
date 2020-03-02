
### Custom Location Settings:

I've doing tiny work on how to change default `location /{}` settings in order to visit services inside docker with a more  flexible way.


### [Usage](https://github.com/southern-dust/nginx-proxy/blob/my_custom/README.md#usage)

Build it before run:
    
```shell
$ docker build -t jwilder/nginx-proxy:test .  # build the Debian variant image
```


To run it:

```shell
$ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy:test
```

Then start any containers you want proxied with an env var `VIRTUAL_HOST=subdomain.youdomain.com` `VIRTUAL_LOCATION=/app1`

```shell
$ docker run -e VIRTUAL_HOST=foo.bar.com  -e VIRTUAL_LOCATION=/app1...
```

note that you may set Env var `VIRTUAL_HOST` as empty, that's ok, if your `VIRTUAL_LOCATION` var isn't empty either same, you'll still doing the right way to visit your services. If


### What did I do?

1. nginx-proxy uses docker-gen to generate nginx config file located at `/etc/nginx/conf.d/default.conf`.

  docker-gen parse Golang text/template. Thus, we could change the template file located at `nginx.tmpl` here.

2. nginx allow multiple `server{}` block inside one `http{}` block in order to catch different requests and for multi thread processing.

there're two different but similler way for a nginx reverse proxied your traffic:

- visit `domain/app1/` directory path to proxy_pass to backend server. In this way, the first & default `server{}` will catch your traffic. Codes may like this:
```nginx
# The first server{} block inside http{}
server{
    location /{
    }

    # Below are reverse proxies
    location /app1/{
        proxy_pass http://app1.local/;
    }
    location /app1/{
        proxy_pass http://app1.local/;
    }
    ......
}
```
```shell
curl localhost/httpd -v
# nginx: 200 OK 
# GET /httpd/ Host: localhost
```

note that I uses two `location` block for a backend server to avoid path problems
for more information, read nginx official document here:
[proxy_pass: https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)

- another way is visit backend server domain directly, if you already set up your DNS resolve.
let all backends domain name bind on nginx-proxy's ip, then traffic will going through nginx-proxy to your backend server.
Codes may like this:

```nginx
# both way need to setup upstream 
upstream app1.local{
    server ip:port;
}

server{
    location /{
        proxy_pass http://app1.local/;
    }
}
```


```shell
curl localhost/httpd -H "host: httpd.local" -v
# httpd: 200 OK 
# GET /httpd/ Host: httpd.local
```

both way will work properly in the same time.
Of course it needs more configuration for special backend (Like Airsonic... :P

hope my tiny work will help you.



