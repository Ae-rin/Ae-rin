# Apache 수동설치 + Tomcat 연동(mod\_jk)

> **root 계정이 아닌 사용자계정으로 Apache 를 설치하고 서비스하는 가이드를 기술한다.**

<br>

## 1\. Apache 와 Tomcat 을 연동하는 이유?

> Apache 는 외부에서 HTTP프로토콜의 호출이 오면 설정에 따라 웹 페이지를 전송. 단순 이미지 파일이나 정적 데이터를 처리 하기에 최적화 그러나 JSP/PHP 같은 동적페이지 처리에는 TOMCAT 이 적합

**TIP**  [Tomcat 설치 가이드 바로가기](https://github.com/Ae-rin/Ae-rin/blob/main/TIL/Server/%EB%A6%AC%EB%88%85%EC%8A%A4%20%EC%84%9C%EB%B2%84%EC%97%90%20tomcat%EC%84%A4%EC%B9%98.md)

<br>

## 2\. Apache 설치

**WARNING**

추가 패키지 설치 및 권한 설정등의 작업이 있으므로 root 계정으로 설치 후 사용자 계정으로 권한 변경

<br>

### 2.1 설치파일 다운로드

-   [apr 최신버전 확인](https://apr.apache.org/download.cgi)
-   [apr-util 최신버전 확인](https://apr.apache.org/download.cgi)
-   [pcre 최신버전 확인](https://ftp.pcre.org/pub/pcre/)
-   [httpd 최신버전 확인](http://httpd.apache.org/download.cgi)

```
$ yum install gcc gcc-c++ httpd-devel
```

```
$pwd
/home/custom/
$ mkdir apache2    <-- 아파치 설치폴더(사용자 Home (ex./home/custom ))
$ mkdir APP        <-- 유틸설치 및 다운로드
$ cd APP
$ wget http://mirror.apache-kr.org/httpd/httpd-2.4.46.tar.gz
$ wget http://mirror.apache-kr.org/apr/apr-1.6.5.tar.gz
$ wget http://apache.mirror.cdnetworks.com/apr/apr-util-1.6.1.tar.gz
$ wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz

$ tar -xvf httpd-2.4.46.tar.gz
$ tar -xvf apr-1.6.5.tar.gz
$ tar -xvf apr-util-1.6.1.tar.gz
$ tar -xvf pcre-8.41.tar.gz
```

<br>

### 2.2 설치

**WARNING**

\--prefix 경로가 실제 설치경로와 일치하도록!!

```
$ cd /APP/apr-1.6.5
$ ./configure --prefix=/APP/apr-1.6.5
$ make && make install
```

```
$ cd /APP/apr-util-1.6.1
$ ./configure --prefix=/APP/apr-util-1.6.1 --with-apr=/APP/apr-1.6.5
$ make && make install
```

```
$ cd /APP/pcre-8.41
$ ./configure --prefix=/APP/apr-util-1.6.1 --with-apr=/APP/apr-1.6.5
$ make && make install
```

<br>

**WARNING**

\--prefix=/APP/apache2 이전에 생성한 Apache 설치 경로

```
$ cd /APP/httpd-2.4.46
$ ./configure --prefix=/APP/apache2 --enable-modules=most --enable-mods-shared=all --enable-so --with-apr=/APP/apr-1.6.5 --with-apr-util=/APP/apr-util-1.6.1
$ make && make install
```

<br>

**TIP**

Error : pcre-config for libpcre not found 관련 에러 발생시

```
$ yum install pcre-devel -y
```

<br>

### 2.3 실행

**TIP**

root 계정으로 다운로드 및 설치했을 경우 폴더 권한을 확인하여 사용자 권한으로 바꿔준다  
사용자 계정은 custom 라고 할때 ( root 계정)

```
$ cd /home/custom  <-- 사용자 홈으로 이동
$ ls -al
..
drwxr-xr-x.   6 root  root   128 Aug 28 13:23 APP
..
```

위와같이 root로 폴더 권한일 경우

```
$ chown -R custom:custom APP
$ ls -al 
..
drwxr-xr-x.   6 custom  custom   128 Aug 28 13:23 APP
..
```

-   사용자 계정으로 전환 후

```
$ cd /APP/apache2/bin
$ ./httpd -k start 

(13)Permission denied: AH00072: make_sock: could not bind to address [::]:80
(13)Permission denied: AH00072: make_sock: could not bind to address 0.0.0.0:80
no listening sockets available, shutting down
```

<br>

**WARNING**

1024 이하의 포트는 보안상의 이유로 root 권한을 가지고 있는 프로세스만이 사용할 수 있다.

```
$ su -  <-- root 계정 전환
$ cd /home/custom/APP/apache2/bin
$ chown root:custom httpd
$ chmod +s httpd


### 사용자 계정 전환 후 Apache 시작
$ su - custom 
$ cd /APP/apache2/bin
$ ./httpd -k start 

## localhost 로 접속해서 확인
```

<br>

### 2.4 서비스 등록

> 서버재기동시에 자동으로 서비스 올라오도록 서비스에 등록

```
#### root 계정
$ cp /APP/apache2/bin/apachectl /etc/init.d/httpd
$ chkconfig --add httpd


###### Error 발생시 ###########
$ vi /etc/init.d/httpd

######### $!/bin/sh 아래에 5줄 추가/자신의 설치경로로 수정

!/bin/sh    

 chkconfig: 2345 90 90
 description: init file for Apache server daemon
 processname: /APP/apache2/bin/apachectl
 config: /APP/apache2/conf/httpd.conf
 pidfile: /APP/apache2/logs/httpd.pid

:wq 


$ chkconfig --add httpd

#############################


$ chkconfig --list | grep httpd

Note: This output shows SysV services only and does not include native
      systemd services. SysV configuration data might be overridden by native
      systemd configuration.

      If you want to list systemd services use 'systemctl list-unit-files'.
      To see services enabled on particular target use
      'systemctl list-dependencies [target]'.

httpd           0:off   1:off   2:on    3:on    4:on    5:on    6:off



```

<br><br>

## 3\. Apache + Tomcat 연동

### 3.1 Tomcat Connector 설치

**TIP**

[Tomcat-Connector 최선버전 다운로드](http://apache.tt.co.kr/tomcat/tomcat-connectors/jk/)

로컬에 다운로드 받고 FTP로 서버에 업로드 하거나, 다운로드 링크를 복사해서 서버에서 바로 다운로드 받아도된다. 해당 가이드는 서버에서 직접 다운로드 받는 방법을 사용

```
#### root 계정으로 진행 ######### 

$ yum install gcc gcc-c++     

$ cd /home/custom/APP
$ wget http://apache.tt.co.kr/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz
$ tar -xvf tomcat-connectors-1.2.46-src.tar.gz

$ cd tomcat-connectors-1.2.46-src/native/
$ ./configure --with-apxs=/home/custom/APP/apache2/bin/apxs
$ make
$ make install
```

**TIP**

Redhat 의 경우 오류날 경우

```
$ yum install make
$ dnf install redhat-rpm-config
```

<br>

### 3.2 Apache 설정파일 수정

**WARNING**

mod\_jk.conf 파일 생성 ( \* 자동설치와 다름 )

```
$ vi /home/custom/APP/apache2/conf/mod_jk.conf

<IfModule jk_module>
   JkWorkersFile /home/custom/APP/apache2/conf/workers.properties
   JkLogFile /home/custom/LOG/apache2/mod_jk.log   
   JkLogLevel info
   JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
   JkMount /* loadbalancer
</IfModule>
```

```
$ vi /home/custom/APP/apache2/conf/httpd.conf

/LoadModule      <-- 해당 단어 검색후 뒤에 아래 내용 추가

## ------------------------
LoadModule jk_module modules/mod_jk.so

include conf/mod_jk.conf
## ------------------------
```
<br>

**WARNING**

설정내용은 WEB1-WAS1 / WEB2-WAS2 구성의 예로 작성되어 있으며 각 프로젝트 환경에 맞는 설정으로 수정 후 적용

3.2.1 이중화 구성시

```
$ vi /home/custom/APP/apache2/conf/workers.properties

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker1,worker2
#worker.loadbalancer.balanced_workers=worker2
worker.loadbalancer.sticky_session=1

worker.worker1.type=ajp13
worker.worker1.host=서버1의 주소
worker.worker1.port=8010
worker.worker1.lbfactor=1

worker.worker2.type=ajp13
worker.worker2.host=서버2의 주소
worker.worker2.port=8010
worker.worker2.lbfactor=1
```

<br>

**TIP**

type과 port 속성은 Tomcat의 server.xml 설정을 참조합니다. 3.4 Tomcat 설정 참조!

3.2.2 WEB-WAS 단독 서버 구성시

```
$ vi /home/custom/APP/apache2/conf/httpd.conf

/DocumentRoot       <-- 검색하고

## Tomcat 의 경로와 맞춰준다
DocumentRoot "/home/custom/apache-tomcat-8.5.57/webapps/ROOT"

```

```
$ vi /home/custom/APP/apache2/conf/workers.properties

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker1
worker.loadbalancer.sticky_session=1

worker.worker1.type=ajp13
worker.worker1.host=localhost
worker.worker1.port=8010
worker.worker1.lbfactor=1
```

3.2.3 WEB1-WAS1 설정 참조 (분리구성)

```
$ vi /home/custom/APP/apache2/conf/workers.properties

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker1
worker.loadbalancer.sticky_session=1

worker.worker1.type=ajp13
worker.worker1.host=서버1의 주소
worker.worker1.port=8010
worker.worker1.lbfactor=1
```

3.2.4 WEB2-WAS2 설정 참조 (분리구성)

```
$ vi /home/custom/APP/apache2/conf/workers.properties

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker2
worker.loadbalancer.sticky_session=1

worker.worker2.type=ajp13
worker.worker2.host=서버2의 주소
worker.worker2.port=8010
worker.worker2.lbfactor=1
```

<br>

### 3.3 Apache 재시작

```
$ cd /APP/apache2/bin
$ ./httpd -k restart 
```

**WARNING**

아래와 같은 에러발생시

```
SELinux policy enabled; httpd running as context system_u:system_r:httpd_t:s0
AH01232: suEXEC mechanism enabled (wrapper: /usr/sbin/suexec)
(13)Permission denied: mod_jk: could not open JkLog file /etc/httpd/mod_jk.log
AH00016: Configuration Failed

$ setenforce 0.    <--- Selinux 중지

$ vi /etc/sysconfig/selinux     <-- 서버 재시작시에도 사용안함 설정
..

SELINUX=disabled 
..
```

<br>


### 3.4 Tomcat-server.xml 설정

**TIP**

AJP 설정 부분 주석 해제하고 Tomcat8.5 이상 버전일 경우 secretRequired="false" 속성을 추가 workers.properties 에서 설정한 포트와 같은 포트값으로 적용

```
<!-- Define an AJP 1.3 Connector on port 8009 -->
<Connector protocol="AJP/1.3"
               address="0.0.0.0"
               port="8010"
               secretRequired="false"
               redirectPort="8443" />
..
               

<Context path="" docBase="/APP/?????" debug="0" reloadable="false" URIEncoding="utf-8" />
               
               
```

<br>

Tomcat 재시작 후 웹서버 IP로 확인
