# **\[CentOS7\] Tomcat8.5 설치**

> Tomcat은 지속적으로 버전업이 이루어 지고 있으며 가이드에서는 9버전을 기준으로 작성되었습니다.  
> [Tomcat 최신버전 다운로드](https://tomcat.apache.org/download-90.cgi)

<br>

## **Tomcat 설치**

-   FTP(FileZilla) 또는 wget 명령어를 사용하여 Tomcat 설치파일을 다운로드 받고 압축을 풀면 설치는 끝난다.
-   FTP(FileZilla)를 사용할시엔 다운로드 받은 파일을 압축을 풀고자 하는 위치에 옮긴다.
-   오리지날 폴더명 : apache-tomcat-9.0.37.tar.gz
-   변경하고자 하는 폴더명 : tomcat9.0

```
$ ls apache-tomcat-9.0.37.tar.gz

$ tar xzvf apache-tomcat-9.0.37.tar.gz
$ mv apache-tomcat-9.0.37.tar.gz tomcat9.0
```

<br>

## **Tomcat 메모리 설정**

> **TIP**  
> **setenv.sh 은 Tomcat 실행 환경 설정파일로 JAVA\_OPTS등 환경설정이 가능하다.**  
> **단! 기본적으로 제공되는 파일이 아니므로 직접 생성해야 한다.  
> **

```
$ cd /tomcat9.0/bin
$ vi setenv.sh   

#Heap 사이즈
JAVA_OPTS="$JAVA_OPTS -Xmx4096m -Xms1024m -XX:MaxMetaspaceSize=1024m"
```

<br>

## **시작 & 종료**

```
$ cd /tomcat9.0/bin 
$ ./startup.sh      --- start.sh이 될 수도 있다.
$ ./shutdown.sh
```

<br>

## **확인**

> http://서버아이피:8080

<br>

## **포트 번호 변경**

```
$ cd /tomcat9.0/conf
$ vi server.xml


<Connector port="8006" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
   
port번호를 변경 (ex.8006으로 변경)

확인
http://서버아이피:8006
```
