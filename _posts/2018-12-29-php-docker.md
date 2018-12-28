---
layout: post
title: 'Docker를 사용하여 php를 운영해보자.' 
author: teamsmiley
date: 2018-12-29
tags: [devops]
image: /files/covers/blog.jpg
category: {devops}
---

# Docker를 사용하여 php를 운영해보자.

## nginx 기본 올리기 

vi docker-compose.yml
```yml
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
```

docker-compose up 

웹브라우저로 localhost를 확인하자.

## nginx 설정 
```
mkdir code
```

vi site.conf
```
server {
    index index.html;
    server_name AAA.com;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /code;
}
```

vi docker-compose.yml
```yml
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./code:/code
      - ./site.conf:/etc/nginx/conf.d/default.conf
```

vi code/index.html
```
hello
```
이제 다 되었다.
```
docker-compose up 
```

확인해보자.

## php-fpm을 추가해보자. 

php-fpm은 포트 9000번을 사용하여 대기한다. 

vi site.conf
```
server {
    index index.html;
    server_name tig.local;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /code;
    
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php:9000; 
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

vi docker-compose.yml
```yml
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./code:/code
      - ./site.conf:/etc/nginx/conf.d/default.conf
  php:
    image: php:fpm
    volumes:
        - ./code:/code
```

vi code/index.php
```php
<?php
echo phpinfo();
?>
```



