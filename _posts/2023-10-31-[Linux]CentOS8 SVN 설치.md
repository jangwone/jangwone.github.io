CentOS 8에서 Subversion(SVN)을 설치하는 방법 및 설정을 기록해두려 한다.

## 시스템 업데이트

```
sudo dnf update
```

## Subversion 설치

```
sudo dnf install subversion
```

## SVN 서버 구성

저장소를 생성할 디렉토리를 선택한 다음, 다음 명령어를 사용하여 SVN 저장소를 생성합니다.

```
sudo svnadmin create /svn/repos
```

## SVN 사용자 및 그룹 설정

- `/svn/repos/conf` 디렉토리에 있는 `svnserve.conf` 파일을 편집하여 SVN 서버 구성을 설정할 수 있습니다.

<!-- -->

```
sudo nano /svn/repos/conf/svnserve.conf
```

파일을 열고 다음과 같이 설정합니다

```
[general]
anon-access = none
auth-access = write
password-db = passwd
authz-db = authz
```

- 사용자 및 그룹 설정: `passwd` 파일을 편집하여 SVN 사용자를 추가하고 비밀번호를 설정합니다.

<!-- -->

```
sudo nano /svn/repos/conf/passwd
```

파일을 열고 다음과 같이 사용자를 추가합니다.

```
[users] 
user1 = password1 
user2 = password2 
user3 = password3 
user4 = password4 
user5 = password5
```

- 권한 설정: `authz` 파일을 편집하여 사용자에 대한 권한을 설정합니다.

<!-- -->

그룹 별로 다른 권한을 지정하고 싶으면 다음과 같이 설정합니다.

```
sudo nano /svn/repos/conf/authz
```

```
[groups] 
admin = user1, user2 
developers = user3, user4, @teamA 
[/] 
* = @admin = rw 
[/projectA] 
@developers = rw
 user5 = r
```

## SVN 서버 시작

```
sudo svnserve -d -r /svn/repos
svnserve -d -r /mnt/data/svnrepos/ --listen-port 3690
```

## SVN 서버 포트 설정

- CentOS에서 방화벽을 사용하고 있다면, `firewall-cmd` 명령어를 사용하여 해당 포트를 열 수 있습니다.

<!-- -->

다음 명령어를 사용하여 3690 포트를 허용합니다.

```
sudo firewall-cmd --zone=public --add-port=3690/tcp --permanent

sudo firewall-cmd --reload
```

- 포트가 올바르게 열렸는지 확인하는 방법은 몇 가지가 있습니다. 주로 사용되는 방법 중 하나는 `firewall-cmd` 명령어를 통해 확인하는 것입니다.

- CentOS에서 `firewall-cmd` 명령어를 사용하여 열린 포트를 확인하려면 다음과 같이 실행할 수 있습니다:


<!-- -->

```
sudo firewall-cmd --list-ports
```

- 이 명령어는 방화벽에 설정된 모든 오픈 포트를 보여줍니다. 하지만 이 명령어는 특정 포트가 방화벽에 의해 허용되었는지만을 보여주는 것이며, 실제로 해당 포트로 서비스가 동작 중인지는 알 수 없습니다.

<!-- -->

특정 포트 번호에 대한 정보를 얻고자 한다면, 다음 명령어를 사용하여 해당 포트 번호의 상태를 확인할 수 있습니다:

```
sudo netstat -tuln | grep "포트번호"
```