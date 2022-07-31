---
title: "nginx vs apache"
categories:
  - package
date: 2022-07-30 16:00:25 +0900
tags:
  - nginx
  - apache
  - reverse proxy
---

Golang으로 Micro Service Architecture을 위해 banking service를 담당하는 backend 서버를 생성하였다.

```
Client - Nginx - frontend Server
                    |--- bankdend Server - DB
                    |--- etc. 
```

<img src="/assets/images/nginx/original-auth.png">

original 시나리오는 다음과 같다.
1. 사용자가 https://ghkdqhrbals.com 에 접속하면, ISP에서 해당 IP로 라우팅한다.
2. nginx는 TCP/TLS handshake 수행 및 세션을 설정한 후, proxy_pass http:{your_domain}:3000/
   * React
3. react에서 특정 권한 필요한 곳 접근
   * axios http request to backend Server with TOKEN()

원래는 gin의 middleware으로 token validation 및 생성을 관리하였지만, 더 앞단인 nginx 에서 처리를 하려 한다.

이러한 이유는 backend에서 authentication을 관리하려면 다음과 같은 error handling이 필요하다.

* Missing access token
* Extremely large access token
* Invalid or unexpected characters in access token
* Multiple access tokens presented
* Clock skew across backend services

따라서 token관리 및 error handling, backend server의 리소스 소모 reduce를 위해 더 앞단에서의 관리를 하기 위하여 nginx로 authentication을 옮길 것이다.

최종적인 MSA 형태는 다음과 같다.

```
Client - Nginx - frontend Server
          |         |--- bankdend Server - DB2
          |         |--- etc.
          |
          |--- Oauth Server - DB1               
```
<img src="/assets/images/nginx/nginx-auth.png">

시나리오는 다음과 같다.
1. 사용자가 https://ghkdqhrbals.com 에 접속하면, ISP에서 해당 IP로 라우팅한다.
2. nginx는 TCP/TLS handshake 수행 및 세션을 설정한 후, timeout = 10 second
   * nginx의 token based authentication 활용하여 서버의 session 유지에 관한 부담을 줄일 것이다.

https://www.nginx.com/blog/validating-oauth-2-0-access-tokens-nginx/

<p style="margin:0 0 0 0; background-color:#f2f3f3; border-radius:0.2em; text-align:center;">/nginx/auth_request.conf</p>

```conf
server {
  listen 80;
  location = / {
    auth_request /_oauth2_token_introspection;                              
    proxy_pass http://{your.domain}/backendapi;
    error_page 500 http://{your.domain}/login;
  }
  location = /_oauth2_token_introspection {
    internal;
    proxy_method      POST;
    proxy_set_header  Authorization "bearer SecretForOAuthServer";
    proxy_set_header  Content-Type "application/x-www-form-urlencoded";
    proxy_set_body    "token=$http_apikey&token_hint=access_token";
    proxy_pass        https://{your.domain}/oauth/;
  }
}
```

```conf
location / {
    auth_request /_oauth2_token_introspection;
    proxy_pass http://ghkdqhrbals.com/backendapi;
    error_page 500 http://ghkdqhrbals.com/login;
}

location /auth_test {
    internal;
    // 각종 설정들
    proxy_set_header  accept "application/json";
    proxy_set_header  authorization "bearer SecretForOAuthServer";
    proxy_set_body    "token=$http_apikey&token_hint=access_token";
    proxy_pass        https://ghkdqhrbals.com/oauth/;
}
```




