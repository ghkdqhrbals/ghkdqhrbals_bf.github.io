---
layout: post
title:  "Nginx vs Apache"
date:   2022-07-25 02:00:25 +0900
categories: jekyll update
---


<Blockquote>

**`/react-portfolio-website/nginx/nginx.conf`**

```conf
user  nginx;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;
events {                     
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    upstream docker-client {
        server client:3000;
    }
    server {
        listen 80;
        server_name localhost;
        ...
        location / {
            proxy_pass         http://docker-client;
                proxy_redirect     off;
                proxy_set_header   Host $host;
                proxy_set_header   X-Real-IP $remote_addr;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        ...    
}
```

</Blockquote>

Solution
https://stackoverflow.com/questions/70010475/cant-access-sub-paths-on-nginx-react-router-behind-a-proxy

If you want to run app in locally, you should set server_name as localhost or 127.0.0.1
When nginx proxy get request in http://localhost, nginx pass it to client(docker container). At this time, client get request with http://{server_name}:{client_port}. So client react-router cannot read