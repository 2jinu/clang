# clang

실행 환경

| Type              | Version                   |
| :---              | :---                      |
| OS                | Ubuntu 20.04.3 LTS        |
| Architecture      | x86-64                    |
| gcc               | gcc version 9.4.0         |
| libssl-dev        | 1.1.1f-1ubuntu2.13        |
| libboost-all-dev  | 1.71.0.0ubuntu2           |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

 - [ssl 설치](#ssl-설치)

 - [boost 설치](#boost-설치)

 - [소스 컴파일](#소스-컴파일)

**2. [SSL](#SSL)**

 - [Server](#Server)

 - [Client](#Client)

**3. [Full Code](#Full-Code)**


# **패키지 설치**

## **ssl 설치**

libssl-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libssl-dev
```

## **boost 설치**

boost 관련 라이브러리들을 설치한다.

```sh
root@ubuntu:~# apt-get -y install libboost-all-dev
```

수동 설치 시 [boost](https://www.boost.org/users/history/)에서 소스를 다운받은 후 AMD의 경우 다음과 같이 bootstrap.sh을 실행킨다.

```sh
root@ubuntu:~# tar -zxf boost_1_76_0.tar.gz
root@ubuntu:~# cd boost_1_76_0
root@ubuntu:~/boost_1_76_0# ./bootstrap.sh
```

ARM의 경우 project-config.jam을 변경시켜줘야 한다.

```sh
root@ubuntu:~# tar -zxf boost_1_76_0.tar.gz
root@ubuntu:~# cd boost_1_76_0
root@ubuntu:~/boost_1_76_0# ./b2 install
root@ubuntu:~/boost_1_76_0# vi project-config.jam
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : arm : aarch64-linux-gnu-g++ ;
}
root@ubuntu:~/boost_1_76_0# ./b2 install toolset=gcc-arm
root@ubuntu:~/boost_1_76_0# cp -r boost /usr/local/include/
```

## **소스 컴파일**

컴파일시 libssl, libcrypto, libpthread를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lssl -lcrypto -lpthread -o outputfile
```

# **SSL**

서버에서 SSL 암호화를 위한 인증서를 생성하자.

인증서를 생성하기 위해 비밀키와 인증요청서를 생성하자.

```sh
openssl req -nodes -newkey rsa:2048 -keyout private.key -out certificate.csr -subj "/C=KR/ST=Seoul/L=Seoul/O=2jinu/OU=2jinu"
```

인증요청서에 대해 자체 인증서를 만들자.

```sh
openssl req -x509 -key private.key -in certificate.csr -out certificate.crt -days 365
```

## **Server**
## **Client**
# **Full Code**