---
title: "[Server] Window 서버에 Nginx를 사용하여 phpmyadmin 구축하기"

categories:
  -  Server
  
tags:
  - [Window, Server, Nginx, phpmyadmin]

toc: true
toc_sticky: true

published: true

date: 2024-12-16
last_modified_at: 2024-12-16
---

이번 포스팅에서는 Window Os, Nginx 환경에서 phpmyadmin 을 실행시키는 방법에 대해서 소개합니다. 

## phpMyAdmin 이란?

![](/images/Pasted%20image%2020241216154516.png)

서버의 데이터베이스에 접근하려면, mysql client 를 열어 서버 데이터베이스에 원격 접속하는 방법이 있습니다. 하지만 이 방식에는 단점이 존재합니다. 바로 mysql client를 설치해야 한다는 것입니다.

phpMyAdmin은 이 단점을 보완하고자 웹 이라는 플랫폼에서 서버의 데이터베이스를 조작할 수 있도록 해주는 기술입니다. (웹은 브라우저만 있으면 됨)

이 phpMyAdmin을 돌리는 웹서버로는 Nginx를 설치하여 구성합니다. 우선, Nginx 부터 설치를 합니다.

## Nginx 설치하기

Linux나 Ubuntu 처럼 CLI(Command Line Interface) 로 구성되어 있는 운영체제에서는 명령어를 쳐서 Nginx를 다운받아야 합니다. 하지만, Window 는 GUI 가 강력한 OS 입니다. 그냥 웹사이트로 접속하여 다운로드를 받아줍니다. 링크는 다음과 같습니다. 

https://nginx.org/en/download.html

해당 링크로 접속하여 Window Stable 버전을 다운받아 줍니다.

![](/images/Pasted%20image%2020241216155320.png)

그 뒤에, 압축을 해제하고, nginx.exe 실행파일을 실행해 줍니다.

![](/images/Pasted%20image%2020241216155426.png)

Nginx 를 실행할 때, 사용자가 원하는 설정대로 돌리게 할 수 있습니다. conf - nginx.conf 파일을 열어서 설정을 조작할 수 있습니다. (일단 기본으로 두고, phpmyadmin이 설치되면 다시 만집니다.)

Nginx 가 잘 돌아가고 있는지 확인하기 위해서, 브라우저를 열고 localhost로 접근 해봅니다. 다음과 같은 화면이 나온다면, 정상적으로 작동하고 있는것 입니다.

![](/images/Pasted%20image%2020241216155735.png)


## PHP 설치하기

https://windows.php.net/download 위 사이트에 접속하여 php를 다운받아 줍니다. 저는 php 8.1 thread safe 버전을 다운받았습니다. (**여기서 주의해야할 점은 php 8.4 버전 (현재 최신 버전)을 다운받고 phpmyadmin을 실행했을 때, deprecated 가 도배되는 버그가 있었습니다. php 8.1 버전을 다운받으니 해결되었습니다.**)

VC15 x64 Non Thread Safe : 64bit IIS 서버
VC15 x64 Thread Safe : 64bit Apache / Nginx 서버
VC15 x64 Non Thread Safe : 32bit IIS 서버
VC15 x64 Non Thread Safe : 32bit Apache / Nginx 서버

php 파일을 다운받고 압축을 풀어줍니다. 그리고 원하는 경로에 위치시킵니다. 저는 `C:\php` 에 위치시켰습니다.

php 파일 안을 들여다보면, `php.ini-production` 또는 `php.ini-development` 라는 파일이 있습니다. 둘 중 하나의 파일을 열어서 php 의 설정을 해줍니다. 코드 앞에 달려있는 `;`는 주석입니다. 해당 주석을 없애줍니다. 주석을 없애줄 녀석들은 다음과 같습니다.

```bash
extension_dir = "C:\php\ext"
extension=curl
extension=gettext
extension=mbstring
extension=mysqli
extension=openssl
extension=pdo_sqlite
date.timezone = Asia/Seoul
```

설정을 마치고, 설정을 마친 파일의 이름을 `php.ini`로 바꿔줍니다. 여기서 주의해야할 점은 확장자까지 바뀌었는지 봐야 합니다. 

파일탐색기의 보기 - 확장 파일명을 체크하고 이름을 바꿔줍니다.

![](/images/Pasted%20image%2020241216161232.png)

위의 php.ini 처럼 아이콘이 바뀐다면 성공입니다.

이제, cmd 창을 켜서 `php -v`명령어를 쳐봅니다. 제대로 설정되었다면, php의 버전이 출력되어야 합니다. 하지만 다음과 같은 문구가 뜹니다.

![](/images/Pasted%20image%2020241216161348.png)

이는 환경변수가 지정되어 있지 않아서 발생하는 문제입니다.

시작화면 검색창에 `시스템 환경 변수 편집` 을 검색하여 다음 창을 열어줍니다.

![](/images/Pasted%20image%2020241216161459.png)

![](/images/Pasted%20image%2020241216161546.png)

여기서 오른쪽 아래의 환경변수 버튼을 클릭합니다.

![](/images/Pasted%20image%2020241216161806.png)

Path 변수를 선택하고 편집 버튼을 누릅니다.

![](/images/Pasted%20image%2020241216161847.png)

새로 만들기를 선택하여, php가 설치되어있는 경로를 넣습니다. 저의 경우는 `C:/php` 로 지정하였습니다.

여기까지 하고 cmd를 다시 켜서 `php -v` 를 입력하면 버전정보가 나타납니다.

![](/images/Pasted%20image%2020241216161957.png)

## phpMyAdmin 설치하기

이제 phpMyAdmin을 설치합니다. 다운로드 링크는 다음과 같습니다.

https://www.phpmyadmin.net/

![](/images/Pasted%20image%2020241216162204.png)

저는 최신버전인 5.2.1 버전을 설치하였습니다.

압축을 풀고, 해당 파일을 Nginx 폴더의 html 폴더 안에 위치시킵니다.

![](/images/Pasted%20image%2020241216162314.png)

이렇게 하면 설치 완료 입니다.

## Nginx Conf 파일 설정

그리고, nginx 의 conf 파일을 수정해줍니다. 

```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;

	location / {
	     root /usr/share/nginx/html;
	     index index.html index.htm; 
	}

        location /phpmyadmin {
            root C:/nginx/html;  # phpMyAdmin이 위치한 디렉토리
            index index.php;  # 기본 인덱스 파일 설정

            location ~ ^/phpmyadmin/.*\.php$ {
                fastcgi_pass   127.0.0.1:9000;  # PHP-FPM 서버 주소
                fastcgi_index  index.php;
                fastcgi_param  SCRIPT_FILENAME  C:/nginx/html$fastcgi_script_name;  # 실제 PHP 파일 경로
                include        fastcgi_params;
            }
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## php-cgi 실행

이제, 마지막으로 php-cgi를 열어서, php가 작동가능하게 만들어주면 됩니다. 

cmd를 열고, 다음 코드를 입력합니다.

```bash
php-cgi -b 127.0.0.1:9000
```

이렇게 하면 php-cgi 가 9000번 포트에서 열리게 됩니다. 

9000번 포트에서 잘 열렸는지 확인하기 위해 다음과 같은 명령어를 사용합니다.

```bash
netstat -ano | findstr :9000
```

아래의 코드가 출력되면 정상작동하고 있는것 입니다.
```bash
TCP    127.0.0.1:9000     0.0.0.0:0     LISTENING       1234
```

하지만, 위 코드가 출력되지 않는다면, 방화벽을 열어줘야 합니다.

![](/images/Pasted%20image%2020241216163548.png)

`방화벽 상태 확인`을 검색하여 들어가줍니다.

![](/images/Pasted%20image%2020241216163622.png)

고급 설정을 열어줍니다.

![](/images/Pasted%20image%2020241216163652.png)

![](/images/Pasted%20image%2020241216163705.png)

인바운드 규칙 - 새 규칙을 눌러줍니다.


![](/images/Pasted%20image%2020241216163727.png)

![](/images/Pasted%20image%2020241216163744.png)

![](/images/Pasted%20image%2020241216163759.png)

![](/images/Pasted%20image%2020241216163811.png)

![](/images/Pasted%20image%2020241216163829.png)

위와같이 설정을 마무리 하면, 방화벽이 열리게 됩니다. 그 다음 다시 아래 명령어를 수행하면, php-cgi가 정상작동하는것을 확인할 수 있습니다.

```bash
netstat -ano | findstr :9000
```

## phpmyadmin 실행

이제, 브라우저 주소란에 `localhost/phpmyadmin`을 치게 되면, 다음과 같은 화면이 열리게 되고, phpmyadmin 구축이 완료되었습니다.

![](/images/Pasted%20image%2020241216164034.png)

