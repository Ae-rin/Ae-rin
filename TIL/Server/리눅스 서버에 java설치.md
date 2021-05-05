

# openJDK 설치하기

> Oracle jdk 상용화에 따라 open-jdk 를 설치하는 두가지 방법을 진행

-   yum 명령어를 이용한 자동 설치 ( root 계정 )
-   open-jdk 를 다운받아 원하는 경로에 설치 ( 사용자 계정 )

<br>

## **1\. yum 명령어를 이용한 자동설치**

### 1.1 설치가능한 JDK 확인

```
$ yum list java*jdk-devel


Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.kakao.com
 * epel: ftp.iij.ad.jp
 * extras: mirror.kakao.com
 * updates: mirror.kakao.com
Available Packages
java-1.6.0-openjdk.x86_64            1:1.6.0.41-1.13.13.1.el7_3            base   
java-1.7.0-openjdk.x86_64            1:1.7.0.261-2.6.22.2.el7_8            updates
java-1.8.0-openjdk.i686              1:1.8.0.262.b10-0.el7_8               updates
java-1.8.0-openjdk.x86_64            1:1.8.0.262.b10-0.el7_8               updates
java-11-openjdk.i686                 1:11.0.8.10-0.el7_8                   updates
java-11-openjdk.x86_64               1:11.0.8.10-0.el7_8                   updates
java-latest-openjdk.x86_64           1:14.0.2.12-1.rolling.el7             epel
```

<br>

### 1.2 JDK 1.8 설치

```
$ yum install java-1.8.0-openjdk.x86_64 
....
Is this ok [y/d/N]: y
Is this ok [y/d/N]: y
....

Complete!
```

<br>

### 1.3 설치 확인

```
$ java -version
openjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)
```

> **TIP**  
> yum(macOS는 brew)으로 설치하면 리눅스 디폴트 경로에 설치되기 때문에 환경변수를 따로 설정해 주지 않아도 된다.  
> 하지만 수동으로 설치시엔 리눅스 디폴트 경로에 설치되는 것이 아니니 환경변수를 설정해주어야 한다.

<br><br>

## **2\. 수동설치**

## 2.1 openJDK 다운로드 링크

> 아래 두개 링크중 선택하여 다운로드 한 뒤, FTP를 이용해 서버로 업로드

-   [ojdkbuild-1.8-linux-x64.tar](http://gofile.me/48ukB/jQtEmYv4F)
    -   [참조링크](https://github.com/ojdkbuild/ojdkbuild)
-   [ZuluBuilds-openJDK1.8-linux-x64.tar.gz](http://gofile.me/48ukB/wOAxLxCt1)
    -   [참조링크](https://www.azul.com/downloads/zulu-community/?version=java-8-lts&os=centos&architecture=x86-64-bit&package=jdk)

<br>

### 2.2 압축풀기( ex : zuluBuilds-openJDK1.8 )

압축을 풀고자 하는 디렉토리 위치로 이동하여, 그 안에서 압축을 푼다.

```
$ tar xzvf zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz
```

<br>

### 2.3 폴더명 변경

압축을 푼 폴더명을 보기 쉽게 변경한다.

-   오리지널 폴더명 : zulu8.46.0.19-ca-jdk8.0.252-linux\_x64.tar.gz
-   변경할 폴더명 : open-jdk-1.8

```
$ mv zulu8.46.0.19-ca-jdk8.0.252-linux_x64.tar.gz open-jdk-1.8
```

<br>

### 2.4 환경변수 등록

vi 명령어로 편집공간에 진입하여 i를 입력하면 insert모드로 전환되어 입력할 수 있다.

편집을 마친 뒤 esc → shift + : → wq입력 을 하게 되면 입력 내용을 **저장하고** 편집 공간에서 나오게 된다.

```
$ cd        ----- 사용자 홈으로 이동
$ ls -al 
drwx------  3 cms  cms       4096 Sep 14 20:23 .
drwxr-xr-x. 5 root root      4096 Sep 10 14:28 ..
-rw-------  1 cms  cms       1850 Sep 14 20:05 .bash_history
-rw-r--r--  1 cms  cms         18 Oct 31  2018 .bash_logout
-rw-r--r--  1 cms  cms        332 Sep 14 20:03 .bash_profile      <-- 변수적용할 파일
-rw-r--r--  1 cms  cms        231 Oct 31  2018 .bashrc

$ vi .bash_profile


# User specific environment and startup programs

JAVA_HOME=/home/cms/open-jdk-1.8       <-- 경로는 환경에 따라 다름
JRE_HOME=$JAVA_HOME/jre

PATH=$PATH:$HOME/.local/bin:$HOME/bin:$JAVA_HOME/bin:$JRE_HOME/bin

export JAVA_HOME
export JRE_HOME

export PATH


또는



export JAVA_HOME=/home/cms/open-jdk-1.8       <-- 경로는 환경에 따라 다름

export PATH=${PATH}:$JAVA_HOME/bin
```

> **TIP  
> **보통은 기본적으로 /Library/Java/JavaVirtualMachines/jdk-15.0.1.jdk/Contents/Home 위치에 설치가 된다.  
> 확인해보자.

<br>

### 2.5 환경변수 설정 적용 후 확인

편집 공간에서 나와 source 명령어를 입력해줘야 편집 내용이 적용된다.

```
$ source .bash_profile
$ java -version 
openjdk version "1.8.0_265"
OpenJDK Runtime Environment (build 1.8.0_265-01)
OpenJDK 64-Bit Server VM (build 25.265-b01, mixed mode)
```
