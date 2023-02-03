---
title: "[AWS] EC2 - Linux 환경에 Laravel 설치하기"
layout: single
categories: aws
tag: ['aws', 'ec2']
toc: true
toc_label: "목차"
toc_icon: "fas fa-list-ul"
author_profile: false
sidebar:
    nav: "docs"
---

AWS EC2 Linux환경에 라라벨 프로젝트를 배포하려 합니다. 제가 설치한 환경은 아래와 같습니다. <br>
AWS EC2       : Amazon Linux <br>
PHP                : 8.0

## Nginx, Php 설치하기

```php
sudo amazon-linux-extras install -y php8.0 nginx1
sudo yum install -y php-bcmath php-mbstring php-gd php-xml php-opcache php-zip
```

설치가 완료되면 nginx 버전과 php 버전을 확인합니다.

```php
nginx -v
php -v
```

AWS EC2의 퍼블릭ip를 입력해서 접속했을 때 nginx 홈페이지가 뜨면 정상입니다.

## Nginx 설정

설정파일 수정은 Laravel 공식 홈페이지를 참고했습니다.

[라라벨 8.x - 배포](https://laravel.kr/docs/8.x/deployment)

laravel 구동을 위해 nginx 설정 파일을 수정해야 합니다.
각각 자신에 환경에 맞게 server 부분을 수정합니다.

```php
sudo vim /etc/nginx/nginx.conf
```

```php
server {
    listen 80;
    listen [::]:80;
    server_name myProject.com;
    root /www/html/myProject/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

설정 후 nginx 설정파일을 수정하면 반드시 nginx를 재시작 해야합니다.

```php
sudo systemctl enable nginx
```

## Composer 설치

```php
curl -s https://getcomposer.org/installer | php
```

## Laravel 설치

nginx에서 설정한 경로에 Laravel을 설치해줍니다.

```php
composer global require laravel/installer
composer create-project laravel/laravel myProject
composer update
```

설치 후 `bootstrap` , `storage` 파일에 권한을 부여해야 합니다.

```php
chomd -R 777 bootstrap
chomd -R 777 storage
//# 권한은 상황에 맞게 설정해주세요
```

\
이제 EC2 퍼블릭ip로 접속해서 라라벨 웰컴페이지가 나오면 완료입니다!

![스크린샷 2023-02-03 오후 8.20.27.png](/assets/images/aws/ec2/aws_ec2_15.png)
