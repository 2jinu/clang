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

[CIDR-Mask](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9)#%EC%A0%91%EB%91%90%EC%96%B4_%ED%95%A9%EC%B9%A8)를 확인하면 CIDR가 28일때, Mask는 255.255.255.240이다.

CIDR를 Mask로 코드로 변경해보자.

```cpp
char Result[16] = {'\0' };
unsigned long CIDR  = 28;
unsigned long Mask  = (0xFFFFFFFF << (32 - CIDR)) & 0xFFFFFFFF;
snprintf(Result, 16, "%lu.%lu.%lu.%lu", Mask >> 24, (Mask >> 16) & 0xFF, (Mask >> 8) & 0xFF, Mask & 0xFF);
printf("%s\n", Result);
```

# **netmask2prefix-cidr**

[CIDR-Mask](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9)#%EC%A0%91%EB%91%90%EC%96%B4_%ED%95%A9%EC%B9%A8)를 확인하면 Mask가 255.255.255.240일때, CIDR는 28이다.

Mask에서 CIDR로 코드로 변경해보자.

```cpp
unsigned int n, i = 0;
inet_pton(AF_INET, Result, &n);
while (n > 0)
{
    if (n & 1) i++;
    n = n >> 1;
}
printf("%d\n", i);
```

# **사용 가능한 IP 수**

[CIDR-Mask](https://ko.wikipedia.org/wiki/%EC%82%AC%EC%9D%B4%EB%8D%94_(%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%82%B9)#%EC%A0%91%EB%91%90%EC%96%B4_%ED%95%A9%EC%B9%A8)를 확인하면 CIDR가 28일때, Hosts는 16이다.

CIDR를 입력으로 사용 가능한 IP 수를 계산해보자.

```cpp
printf("%.0f\n", pow(2, 32 - i));
```

# **사용 가능한 IP 대역**

IP주소와 Mask를 가지고 사용 가능한 IP들을 출력해보자.

첫번째는 네트워크 주소이고, 마지막은 브로드캐스트 주소가 된다.

```cpp
struct in_addr IPAddress, NetMask;
int Count = 0;
inet_pton(AF_INET, "192.168.100.125", &IPAddress);
inet_pton(AF_INET, "255.255.255.240", &NetMask);

unsigned long FirstIPAddress    = ntohl(IPAddress.s_addr & NetMask.s_addr);
unsigned long LastIPAddress     = ntohl(IPAddress.s_addr | ~(NetMask.s_addr));

for (unsigned long IPAddress = FirstIPAddress; IPAddress <= LastIPAddress; ++IPAddress)
{
    struct in_addr ip_addr;
    ip_addr.s_addr = htonl(IPAddress);
    Count++;
    printf("The IP address is %s\n", inet_ntoa(ip_addr));
}
printf("Total Hosts : %d\n", Count);
```

# **Full Code**

```cpp
#include <stdio.h>
#include <arpa/inet.h>
#include <math.h>
//#include <arpa/inet.h>

int main(void)
{
    char Result[16] = {'\0' };
    unsigned long CIDR  = 28;
    unsigned long Mask  = (0xFFFFFFFF << (32 - CIDR)) & 0xFFFFFFFF;
    snprintf(Result, 16, "%lu.%lu.%lu.%lu", Mask >> 24, (Mask >> 16) & 0xFF, (Mask >> 8) & 0xFF, Mask & 0xFF);
    printf("%s\n", Result);

    unsigned int n, i = 0;
    inet_pton(AF_INET, Result, &n);
    while (n > 0)
    {
        if (n & 1) i++;
        n = n >> 1;
    }
    printf("%d\n", i);

    printf("%.0f\n", pow(2, 32 - i));


    struct in_addr IPAddress, NetMask;
    int Count = 0;
    inet_pton(AF_INET, "192.168.100.125", &IPAddress);
    inet_pton(AF_INET, "255.255.255.240", &NetMask);

    unsigned long FirstIPAddress    = ntohl(IPAddress.s_addr & NetMask.s_addr);
    unsigned long LastIPAddress     = ntohl(IPAddress.s_addr | ~(NetMask.s_addr));

    for (unsigned long IPAddress = FirstIPAddress; IPAddress <= LastIPAddress; ++IPAddress)
    {
        struct in_addr ip_addr;
        ip_addr.s_addr = htonl(IPAddress);
        Count++;
        printf("The IP address is %s\n", inet_ntoa(ip_addr));
    }
    printf("Total Hosts : %d\n", Count);
}
```