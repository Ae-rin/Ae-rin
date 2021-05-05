> mod\_ssl 을 이용한 SSL 설치 가이드 ( \* 경로는 Apache 자동설치 기준 )

## 1\. mod\_ssl 설치

```
$ yum -y install mod_ssl
```

## 2\. vi /etc/httpd/conf.d/ssl.conf 알맞게 수정

```
## 
## SSL Virtual Host Context 
## 

DocumentRoot "/var/www/html"


<VirtualHost *:443> 

... 

ServerName 페이지 주소(=서버 주소):443 

JkMount /* loadbalancer 


# SSL Engine Switch: 
# Enable/Disable SSL for this virtual host. 

ErrorLog logs/ssl_error_log
TransferLog logs/ssl_access_log
LogLevel warn


SSLEngine on 

SSLProtocol all -SSLv2  -SSLv3


... 
SSLCertificateFile ssl파일 경로/파일명.crt


... 
SSLCertificateKeyFile ssl파일 경로/KeyFile명.key 


... 
SSLCertificateChainFile ssl파일 경로/ChainFile명.crt 


... 

<Directory "ssl을 적용할 데이터 상위 경로(해당 경로 하위 파일을 로드하는 모든 페이지에 ssl적용)"> 
		Options Indexes FollowSymLinks 
        AllowOverride None 
        Order allow,deny 
        Allow from all 
        Require all granted 
</Directory>


```

## 3\. 서비스 재시작 및 포트 확인

```
$ systemctl restart httpd 
$ netstat -lnp | grep 443
```
