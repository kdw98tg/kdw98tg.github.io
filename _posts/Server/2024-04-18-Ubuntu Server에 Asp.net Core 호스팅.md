---
title: "[Ubuntu] Ubuntu Server에 Asp.net 호스팅 설정하기"

categories:
  -  Ubuntu
  
tags:
  - [Ubuntu, Server, nginx, phpmyadmin]

toc: true
toc_sticky: true

published: true

date: 2024-04-20
last_modified_at: 2024-05-14
---


Ubuntu 22.0.4 LTS 버전을 활용하여 서버를 구성하였습니다.

웹서버는 `nginx`를 사용하였습니다.

이제 asp.net core의 애플리케이션을 배포하는 작업만이 남았었습니다.

처음 배포를 하다보니까 nginx의 디폴트 설정도 헷갈렸습니다.

다음 코드는 제 asp.net 호스팅을 위한 세팅입니다. 본 세팅은 Microsoft 공식문서를 참고하였습니다.

또한 `phpmyadmin`을 설치하여 외부에서도 접근가능하도록 만들어두었습니다.

다음은 제 설정파일 입니다

사용하실때는 `[your-http-port]`처럼 생긴것들을 통쨰로 삭제하고 포트번호를 넣으시면 됩니다.

```shell
server {
        listen [your-http-port] default_server;
#        listen [::]:[your-http-port] default_server;

        server_name _;

        location / {

#               try_files $uri $uri/ =404;
                proxy_pass         http://127.0.0.1:[your-proxy-path]]/;
                proxy_http_version 1.1;
                proxy_set_header   Upgrade $http_upgrade;
                proxy_set_header   Host $host;
                proxy_cache_bypass $http_upgrade;
                proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header   X-Forwarded-Proto $scheme;
        }

        location /phpmyadmin {

                root /usr/share/;       
                index index.php index.html index.htm;
                location ~ ^/phpmyadmin/(.+\.php)$ {
                    try_files $uri =404;
                    root /usr/share/;
                    fastcgi_pass unix:/run/php/php8.1-fpm.sock; # PHP 버전에 맞춰 경로 수정 필요
                    fastcgi_index index.php;
                    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
                    include /etc/nginx/fastcgi_params;
        }
}

}
```