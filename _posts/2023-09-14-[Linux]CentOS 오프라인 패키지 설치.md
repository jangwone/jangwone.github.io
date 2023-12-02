---
layout: post
title: [Linux]CentOS 오프라인 패키지 설치
tags: linux, centos
---

> ## **작업환경**

- #### CentOS 7.9

  

> ## 작업방식



#### **1. 온라인 시스템에서 필요한 저장소 추가**

-  먼저 , 온라인 시스템에서 EPEL 및 PostgreSQL YUM 저장소를 추가 합니다.

```sh
yum install -y epel-release
rpm -Uvh https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
```



#### **2. 패키지 및 의존성 다운로드** 

- reposync 도구를 사용하여 필요한 패키지와 의존성을 다운로드 합니다.
- 먼저 yum-utils와 createrepo를 설치합니다.

```sh
yum install -y yum-utils createrepo
```

그런 다음, 리포지토리를 동기화하여 PostGIS와 그 의존성을 다운로드 합니다.

```
#리포지토리 메타데이터 생성
mkdir -p /path
reposync -p /path--repoid=pgdg -n --download-metadata
reposync -p /path/to/repo --repoid=epel -n --download-metadata
```



#### **3. Local Repository 생성**

- 다운로드한 패키지를 기반으로 로컬 Repository를 생성합니다.

```sh
createrepo /path/to/repo/pgdg
createrepo /path/to/repo/epel
```



#### **4. 오프라인 시스템에서 YUM Repository 설정**

- 이제 패키지와 메타데이터를 오프라인 시스템으로 전송합니다.
- 오프라인 시스템에서 로컬 Repository를 설정합니다.
- /etc/yum.repos.d/local.repo 파일을 생성하고 다음 내용을 추가합니다.

```sh
[pgdg]
name=Postgis CentOS 7 
baseurl=file:///path/to/repo/pgdg
enabled=1
gpgcheck=0

[epel]
name=epel CentOS 7
baseurl=file:///path/to/repo/epel
enabled=1
gpgcheck=0
```



#### **5. PostGIS 설치**

- 마지막으로, 로컬 Repository를 사용하여 PostGIS를 *설치합니다*.

```sh
# yum search postgis
# Ex>>  libpcre.0.().64bit
# Yym search pcre
# Yum install pcre-devel.x86_64

yum clean all
yum install postgis
```



