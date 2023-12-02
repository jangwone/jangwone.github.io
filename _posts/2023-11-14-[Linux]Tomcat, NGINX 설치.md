---
layout: single
title: "[Linux]Tomcat, NGINX 설치"
tags: 
  - linux
  - nginx
  - tomcat
---
## Tomcat 설치하기

다음 한 줄을 실행해서 SDKMAN을 설치합니다.

```
$ curl -s "https://get.sdkman.io" | bash
$ source "/home/aosadm/.sdkman/bin/sdkman-init.sh"
```

Corretto에서 제공하는 Java 17 버전을 설치 하도록 하겠습니다.

```
$ sdk list java 
$ sdk install java 17.0.4-amzn
```

다음으로 maven을 설치하도록 하겠습니다.

```
$ sdk list maven
$ sdk install maven 3.9.5
```

tomcat을 설치한다.

```
$ wget https://dlcdn.apache.org/tomcat/tomcat-8/v8.5.95/bin/apache-tomcat-8.5.95.tar.gz
$ tar xvfz apache-tomcat-8.5.95.tar.gz
```

tomcat-users.xml 을 수정하여 tomcat manager를 local이 아닌 곳에서도 접근할 수 있게 합니다.

```
$ cd apache-tomcat-8.5.95/conf
$ ls -altr
$ vi tomcat-users.xml
-----------------------------------------------------------
# :$ 명령어를 이용해 맨 밑으로 이동 
# yy 복사 
# p 붙여넣기
<role rolename="manager-gui"/>
<user susername="tomcat" password="" roles="manager-gui">
# :wq
```

tomcat 디렉토리에서 tomcat을 실행한다.

```
$ bin/startup.sh
$ curl localhost:8080
$ bin/shutdown.sh
```

context.xml을 편집한다.<br>

 allow 에 ip를 추가한다. (. 은 \ escape 처리해주어야 한다.)<br>

 ip를 추가하고 톰캣을 재실행하면 manager 페이지에 접근할 수 있다.

```
$ cd webapps/manager/META-INF/
 $ vi context.xml
 $ :wq
```

## NginX 설치

```
sudo dnf install nginx
systemctl start nginx
```

Reverse Proxy 설정하기<br>

 conf 파일에 bodysize, gzip 옵션을 추가합니다.

```
$ su
$ vi /etc/nginx/nginx.conf
## 17dd (17line delete)
-----------------------------------------------------------
http {
	•••
	gzip  on;
	gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;
	•••
	client_max_body_size 200M;
	•••
	}
	
-----------------------------------------------------------
 $ cd /etc/nginx/conf.d
 $ vi service.conf
```
service.conf 파일에 다음 내용을 추가한다.
```
server {
    listen       80 default_server;
    listen       [::]:80 default_server;
    server_name  _;
#    root         /usr/share/nginx/html;

	location / {
	    sendfile off;
	    proxy_pass         http://127.0.0.1:8080;
	    proxy_redirect     default;
	    proxy_http_version 1.1;
	    proxy_set_header   Host              $host;
	    proxy_set_header   X-Real-IP         $remote_addr;
	    proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
	    proxy_set_header   X-Forwarded-Proto $scheme;
	    proxy_cache_bypass $http_upgrade;
	    proxy_max_temp_file_size 0;
	}
	
    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    error_page 404 /404.html;
        location = /40x.html {
    }

    error_page 500 502 503 504 /50x.html;
        location = /50x.html {
    }
}
```
conf 파일을 수정하면 항상 문법을 확인을 한다.
```
# nginx -t
# systemctl restart nginx
```

nginx 오류확인
proxy_pass 설정을 했는데 오류가 발생했다.
```
connect() to 127.0.0.1:8080 failed (13: Permission denied) while connecting to upstream, client: 127.0.0.1
```
위 로그는 SELinux 자체 보안 정책 때문에 발생했을 수 있다.
이 이슈를 해결해 보자.
```
$ sudo tail -f /var/log/nginx/error.log
$ setsebool -P httpd_can_network_connect 1
```
참고: [https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx](<https://stackoverflow.com/questions/23948527/13-permission-denied-while-connecting-to-upstreamnginx>)


