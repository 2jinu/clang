# subnetting

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |

# **INDEX**

**1. [prefix-cidr2netmask](#prefix-cidr2netmask)**

**2. [netmask2prefix-cidr](#netmask2prefix-cidr)**

**3. [사용 가능한 IP 수](#사용-가능한-IP-수)**

**4. [사용 가능한 IP 대역](#사용-가능한-IP-대역)**

**5. [Full Code](#Full-Code)**


# **prefix-cidr2netmask**

[CIDR-Mask](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9)#%EC%A0%91%EB%91%90%EC%96%B4_%ED%95%A9%EC%B9%A8)를 확인하면 28은 255.255.255.240이다.

cidr를 mask로 코드로 변경해보자.

```cpp
char Result[16] = {'\0' };
unsigned long CIDR  = 28;
unsigned long Mask  = (0xFFFFFFFF << (32 - CIDR)) & 0xFFFFFFFF;
snprintf(Result, 16, "%lu.%lu.%lu.%lu", Mask >> 24, (Mask >> 16) & 0xFF, (Mask >> 8) & 0xFF, Mask & 0xFF);
printf("%s\n", Result);
```

# **netmask2prefix-cidr**
# **사용 가능한 IP 수**
# **사용 가능한 IP 대역**
# **Full Code**