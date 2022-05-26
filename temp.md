# clang

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpcap-dev   | 1.9.1-3                   |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [World](#World)**

 - [Hello](#Hello)

**3. [Full Code](#Full-Code)**


# **패키지 설치**

libssl-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libssl-dev
```

컴파일시 libssl과 libcrypto을 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lssl -lcrypto -o outputfile
```

# **World**
## **Hello**
# **Full Code**