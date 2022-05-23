# 패킷 생성(Libnet)

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpnet-dev   | 1.1.6+dfsg-3.1build1      |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [패킷 생성](#패킷-생성)**

 - [ICMP](#ICMP)

 - [TCP](#TCP)

 - [UDP](#UDP)

**3. [Full Code](#Full-Code)**


# **패키지 설치**

libnet-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libnet-dev
```

컴파일시 libnet을 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lnet -o outputfile
```

# **패킷 생성**

libnet의 libnet_t 구조체 포인터를 [libnet_init](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_init) 함수를 이용하여 초기화해주자.

```cpp
char ErrorBuffer[LIBNET_ERRBUF_SIZE]    = { '\0' };
libnet_t *Libnet    = NULL;
Libnet              = libnet_init(LIBNET_RAW4, "ens33", ErrorBuffer);
if (Libnet == NULL) {
    fprintf(stderr, "libnet_init() failed: %s", ErrorBuffer);
    return -1; 
}
```

## **ICMP**

소스 IP 주소를 난수로 생성하여 ECHO Request를 전송해보자.

icmp에 데이터(a~z)를 얹기 위해서 데이터를 생성한다.

```cpp
libnet_ptag_t ip    = LIBNET_PTAG_INITIALIZER;
u_char ICMPData[27] = { '\0' };
for (int i = 0; i < 26; i++) ICMPData[i] = i + 0x61;
```

[libnet_name2addr4](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_name2addr4)를 사용하여 문자열의 IP주소를 unsigned long형태로 변환하여 목적지 IP 주소를 설정하자.

```cpp
u_long DestinationIPAddress;
if ((DestinationIPAddress = libnet_name2addr4(Libnet, "192.168.0.30", LIBNET_DONT_RESOLVE)) == -1) {
    fprintf(stderr, "libnet_name2addr4 error");
    return -1;
}
```

[libnet_seed_prand](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_seed_prand)를 이용하여 난수를 생성하기 위해 seed값을 랜덤하게 설정한다.

```cpp
if (libnet_seed_prand(Libnet) == -1) {
    fprintf(stderr, "libnet_seed_prand error");
    return -1;
}
```

[libnet_build_icmpv4_echo](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_icmpv4_echo)를 이용하여 ICMP를 생성한다.

```cpp
libnet_ptag_t icmp  = LIBNET_PTAG_INITIALIZER;
icmp = libnet_build_icmpv4_echo(ICMP_ECHO, 0, 0, 666, 666, ICMPData, sizeof(ICMPData) - 1, Libnet, 0);
if (icmp == -1) {
    fprintf(stderr, "Can't build ICMP header: %s\n", libnet_geterror(Libnet));
    return -1;
}
```

[libnet_build_ipv4](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_ipv4)를 이용하여 IP를 생성한다.

소스 IP 주소는 난수이기 때문에 [libnet_get_prand](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_get_prand)를 이용한다.

```cpp
icmp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_ICMPV4_ECHO_H + sizeof(ICMPData) - 1, 0, 666, 0, 64, IPPROTO_ICMP, 0, libnet_get_prand(LIBNET_PRu32), DestinationIPAddress, NULL, 0, Libnet, 0);
if (icmp == -1) {
    fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
    return -1;
}
```

[libnet_write](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_write)를 이용하여 패킷을 전송한다.

```cpp
if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));
```

## **TCP**

[libnet_build_tcp](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_tcp)를 이용하여 TCP를 생성한다.

```cpp
const char *TCPPayload      = "Hello TCP";
u_short TCPPayloadLength    = strlen(TCPPayload);
libnet_ptag_t tcp           = LIBNET_PTAG_INITIALIZER;
tcp = libnet_build_tcp(12345, 80, 0, 0, TH_SYN, 64240, 0, 0, LIBNET_TCP_H + 20 + TCPPayloadLength, (uint8_t*)TCPPayload, TCPPayloadLength, Libnet, 0);
if (tcp == -1) {
    fprintf(stderr, "Can't build TCP header: %s\n", libnet_geterror(Libnet));
    return -1;
}
```

[libnet_get_ipaddr4](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_get_ipaddr4)를 이용하여 현재 시스템의 IP를 구하고 [libnet_build_ipv4](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_ipv4)를 이용하여 IP를 생성한다.

```cpp
struct in_addr ip_addr;
u_long MyIPAddress  = libnet_get_ipaddr4(Libnet);
ip_addr.s_addr      = MyIPAddress;
printf("myip : %s\n", inet_ntoa(ip_addr));

tcp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_TCP_H + TCPPayloadLength, 0, 666, 0, 64, IPPROTO_TCP, 0, MyIPAddress, DestinationIPAddress, NULL, 0, Libnet, 0);
if (tcp == -1) {
    fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
    return -1;
}
```

[libnet_write](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_write)를 이용하여 패킷을 전송한다.

```cpp
if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));
```

## **UDP**

[libnet_build_udp](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_udp)를 이용하여 UDP를 생성한다.

```cpp
const char *UDPPayload      = "Hello UDP";
u_short UDPPayloadLength    = strlen(UDPPayload);
libnet_ptag_t udp           = LIBNET_PTAG_INITIALIZER;
udp = libnet_build_udp(12345, 80, LIBNET_UDP_H + UDPPayloadLength, 0, (uint8_t*)UDPPayload, UDPPayloadLength, Libnet, 0); 
if (udp == -1) {
    fprintf(stderr, "Can't build UDP header: %s\n", libnet_geterror(Libnet));
    return -1;
}
```

[libnet_build_ipv4](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_build_ipv4)를 이용하여 IP를 생성한다.

```cpp
udp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_UDP_H + UDPPayloadLength, 0, 666, 0, 64, IPPROTO_UDP, 0, MyIPAddress, DestinationIPAddress, NULL, 0, Libnet, 0);
    if (udp == -1) {
        fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
        return -1;
    }
```

[libnet_write](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_write)를 이용하여 패킷을 전송한다.

```cpp
if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));
```

# **Full Code**

[libnet_write](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_write)이후 [libnet_destroy](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/libnet_destroy)를 이용하여 Libnet을 해제해야 한다.

```cpp
#include <libnet.h>

int main(void)
{
    char ErrorBuffer[LIBNET_ERRBUF_SIZE]    = { '\0' };
    libnet_t *Libnet    = NULL;
    Libnet              = libnet_init(LIBNET_RAW4, "ens33", ErrorBuffer);
    if (Libnet == NULL) {
        fprintf(stderr, "libnet_init() failed: %s", ErrorBuffer);
        return -1; 
    }

    u_long DestinationIPAddress;
    if ((DestinationIPAddress = libnet_name2addr4(Libnet, "192.168.0.30", LIBNET_DONT_RESOLVE)) == -1) {
        fprintf(stderr, "libnet_name2addr4 error");
        return -1;
    }

    u_char ICMPData[27] = { '\0' };
    for (int i = 0; i < 26; i++) ICMPData[i] = i + 0x61;

    if (libnet_seed_prand(Libnet) == -1) {
        fprintf(stderr, "libnet_seed_prand error");
        return -1;
    }

    libnet_ptag_t icmp  = LIBNET_PTAG_INITIALIZER;

    for (int i = 0; i < 3; i ++)
    {
        icmp = libnet_build_icmpv4_echo(ICMP_ECHO, 0, 0, 666, 666, ICMPData, sizeof(ICMPData) - 1, Libnet, 0);
        if (icmp == -1) {
            fprintf(stderr, "Can't build ICMP header: %s\n", libnet_geterror(Libnet));
            return -1;
        }
        icmp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_ICMPV4_ECHO_H + sizeof(ICMPData) - 1, 0, 666, 0, 64, IPPROTO_ICMP, 0, libnet_get_prand(LIBNET_PRu32), DestinationIPAddress, NULL, 0, Libnet, 0);
        if (icmp == -1) {
            fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
            return -1;
        }
        if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));

        libnet_destroy(Libnet);
        Libnet  = NULL;
        Libnet  = libnet_init(LIBNET_RAW4, "ens33", ErrorBuffer);
        if (Libnet == NULL) {
            fprintf(stderr, "libnet_init() failed: %s", ErrorBuffer);
            return -1; 
        }
    }

    struct in_addr ip_addr;
    u_long MyIPAddress  = libnet_get_ipaddr4(Libnet);
    ip_addr.s_addr      = MyIPAddress;
    printf("myip : %s\n", inet_ntoa(ip_addr));

    const char *TCPPayload      = "Hello TCP";
    u_short TCPPayloadLength    = strlen(TCPPayload);
    libnet_ptag_t tcp           = LIBNET_PTAG_INITIALIZER;
    tcp = libnet_build_tcp(12345, 80, 0, 0, TH_SYN, 64240, 0, 0, LIBNET_TCP_H + 20 + TCPPayloadLength, (uint8_t*)TCPPayload, TCPPayloadLength, Libnet, 0);
    if (tcp == -1) {
        fprintf(stderr, "Can't build TCP header: %s\n", libnet_geterror(Libnet));
        return -1;
    }
    tcp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_TCP_H + TCPPayloadLength, 0, 666, 0, 64, IPPROTO_TCP, 0, MyIPAddress, DestinationIPAddress, NULL, 0, Libnet, 0);
    if (tcp == -1) {
        fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
        return -1;
    }
    if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));

    libnet_destroy(Libnet);
    Libnet  = NULL;
    Libnet  = libnet_init(LIBNET_RAW4, "ens33", ErrorBuffer);
    if (Libnet == NULL) {
        fprintf(stderr, "libnet_init() failed: %s", ErrorBuffer);
        return -1; 
    }


    const char *UDPPayload      = "Hello UDP";
    u_short UDPPayloadLength    = strlen(UDPPayload);
    libnet_ptag_t udp           = LIBNET_PTAG_INITIALIZER;
    udp = libnet_build_udp(12345, 80, LIBNET_UDP_H + UDPPayloadLength, 0, (uint8_t*)UDPPayload, UDPPayloadLength, Libnet, 0); 
    if (udp == -1) {
        fprintf(stderr, "Can't build UDP header: %s\n", libnet_geterror(Libnet));
        return -1;
    }
    udp = libnet_build_ipv4(LIBNET_IPV4_H + LIBNET_UDP_H + UDPPayloadLength, 0, 666, 0, 64, IPPROTO_UDP, 0, MyIPAddress, DestinationIPAddress, NULL, 0, Libnet, 0);
    if (udp == -1) {
        fprintf(stderr, "Can't build IP header: %s\n", libnet_geterror(Libnet));
        return -1;
    }
    if (libnet_write(Libnet) == -1) fprintf(stderr, "Write error: %s\n", libnet_geterror(Libnet));

    libnet_destroy(Libnet);
    Libnet  = NULL;
    return 0;
}
```