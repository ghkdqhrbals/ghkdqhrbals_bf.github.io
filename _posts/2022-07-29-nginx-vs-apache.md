---
title: "Welcome to Jekyll!"
date: 2022-07-29 01:40:28 -0400
categories: jekyll update
published: true
---

현재까지 포트폴리오
얼마전에 AWS EC2 인스턴스에 React 프로젝트와 Nginx 컨테이너를 도커로 관리하는 중이었다.


in nginx/nginx.conf
```
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
Solution
https://stackoverflow.com/questions/70010475/cant-access-sub-paths-on-nginx-react-router-behind-a-proxy

If you want to run app in locally, you should set server_name as localhost or 127.0.0.1
When nginx proxy get request in http://localhost, nginx pass it to client(docker container). At this time, client get request with http://{server_name}:{client_port}. So client react-router cannot read



Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/