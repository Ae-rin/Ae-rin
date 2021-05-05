## 1\. Apache 와 Tomcat 을 연동하는 이유?

> Apache 는 외부에서 HTTP프로토콜의 호출이 오면 설정에 따라 웹 페이지를 전송. 단순 이미지 파일이나 정적 데이터를 처리 하기에 최적화 그러나 JSP/PHP 같은 동적페이지 처리에는 TOMCAT 이 적합

**TIP**  [Tomcat 설치 가이드 바로가기](https://github.com/Ae-rin/Ae-rin/blob/main/Server/%EB%A6%AC%EB%88%85%EC%8A%A4%20%EC%84%9C%EB%B2%84%EC%97%90%20tomcat%EC%84%A4%EC%B9%98.md)

<br>

## 2\. Apache 설치

> 환경에 따라 Apache Web Server 가 이미 설치 되어있을 수 있음 자동설치의 경우 root 계정으로 진행

<br>

### 2.1 설치 확인

```
$ yum list installed | grep httpd
```

**TIP**

yum 명령어가 동작하지 않을 경우

```
$ ping 8.8.8.8
```

응답이 정상적으로 올경우

```
$ vi /etc/resolv.conf
```

```
#resolv.conf
...
nameserver 8.8.8.8
```

설정 저장 후 다시 시도

<br>

### 2.2 설치

```
$ yum install httpd 
```

-   설치 경로는 /etc/httpd 이다.

<br>

### 2.3 방화벽 설정 ( CentOs 7 기준 )

> 설치 OS 종류에 따라 방화벽 설정이 다를 수 있습니다. OS 정보를 확인하세요.

```
$ firewall-cmd --permanent --add-service=http
$ firewall-cmd --permanent --add-service=https
$ firewall-cmd --reload
```

-   http://아이피 로 접속해서 확인

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpO4Tm%2FbtqReQrRJgq%2FH1KUvXK2g9vXCd1KobkUjk%2Fimg.png)

<br><br>

## 3\. Apache + Tomcat 연동

### 3.1 Tomcat Connector 설치

**TIP**

[Tomcat-Connector 최선버전 다운로드](http://apache.tt.co.kr/tomcat/tomcat-connectors/jk/)

로컬에 다운로드 받고 FTP로 서버에 업로드 하거나, 다운로드 링크를 복사해서 서버에서 바로 다운로드 받아도된다.

해당 가이드는 서버에서 직접 다운로드 받는 방법을 사용.

```
$ yum install gcc gcc-c++     

$ wget http://apache.tt.co.kr/tomcat/tomcat-connectors/jk/tomcat-connectors-1.2.48-src.tar.gz
$ tar zxvf tomcat-connectors-1.2.46-src.tar.gz

$ cd tomcat-connectors-1.2.46-src/native/          → native안으로 이동
```

Makefile을 생성하기 위해 아래 명령어를 실행.

-   옵션 내용은 Apache확장기능 설치를 도와주는 유틸리티의 경로.
-   다른 블로그나 사이트에서의 설명에는 /usr/sbin/아래 또는 /usr/bin/아래의 경로로 지정으로 설명되어져 있는데
-   내가 설치 한 버전은 /bin/아래의 경로에 있었음.
-   경로에 찾을수 없다는 에러가 발생하신다면 /usr/sbin/아래 또는 /usr/bin/아래의  경로도 한번 살펴봐야함.

```
$ ./configure --with-apxs=/bin/apxs
```

**WARNING**

/bin 또는 /sbin 하위에 'apxs'파일이 존재하지 않을 경우 apache-devel 패키지 설치가 안된것으로 설치 후 다시 실행

```
$ yum install httpd-devel 
$ which apxs              <-- 파일 존재 확인
```

native 안에서

make 명령어로 컴파일 실행

```
$ make
```

컴파일 완료 후 install 실행

```
$ make install
```

install 후 /etc/httpd/modules/ 경로의 파일 안에 mod\_jk.so파일이 생성되어있음을 확인할 수 있음.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbnJNAW%2FbtqReQFsaUm%2F19BQTquQA1d4QeRQSE2ac1%2Fimg.png)
**TIP**

Redhat 의 경우 오류날 경우

```
$ yum install make
$ dnf install redhat-rpm-config
```

<br>

### 3.2 Apache 설정파일 수정

```
$ vi /etc/httpd/conf/httpd.conf

/LoadModule      <-- 해당 단어 검색후 뒤에 아래 내용 추가

## -----------------------------------------

LoadModule jk_module modules/mod_jk.so

<IfModule jk_module>
   JkWorkersFile /etc/httpd/conf/workers.properties
   JkLogFile /etc/httpd/mod_jk.log
   JkLogLevel info
   JkLogStampFormat "[%a %b %d %H:%M:%S %Y]"
   
   # 모든 요청을 woker loadbalancer 로 연결
   JkMount /* loadbalancer          
</IfModule>

## -----------------------------------------
```

**WARNING**

설정내용은 WEB1-WAS1 / WEB2-WAS2 구성의 예로 작성되어 있으며 각 프로젝트 환경에 맞는 설정으로 수정 후 적용

3.2.1 이중화 구성시

```
$ vi /etc/httpd/conf/workers.properties 

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker1,worker2
#worker.loadbalancer.balanced_workers=worker2
worker.loadbalancer.sticky_session=1

worker.worker1.type=ajp13
worker.worker1.host= 
worker.worker1.port=8010
worker.worker1.lbfactor=1

worker.worker2.type=ajp13
worker.worker2.host= 
worker.worker2.port=8010
worker.worker2.lbfactor=1
```

**TIP**

type과 port 속성은 Tomcat의 server.xml 설정을 참조합니다. 3.4 Tomcat 설정 참조!

<br>

3.2.2 WEB1-WAS1 설정 참조(단일구성/단일테스트시)

```
$ vi /etc/httpd/conf/workers.properties  

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker1
worker.loadbalancer.sticky_session=1

worker.worker1.type=ajp13
worker.worker1.host=
worker.worker1.port=8010
worker.worker1.lbfactor=1
```

<br>

3.2.3 WEB2-WAS2 설정 참조(단일구성/단일테스트시)

```
$ vi /etc/httpd/conf/workers.properties  

worker.list=loadbalancer

worker.loadbalancer.type=lb
worker.loadbalancer.balanced_workers=worker2
worker.loadbalancer.sticky_session=1

worker.worker2.type=ajp13
worker.worker2.host=
worker.worker2.port=8010
worker.worker2.lbfactor=1
```

<br>

### 3.3 Apache 재시작

```
$systemctl restart httpd
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

..
               

<Context path="" docBase="/APP/jiniworks_cms" debug="0" reloadable="false" URIEncoding="utf-8" />               




```

Tomcat 재시작 후 웹서버 IP로 확인
