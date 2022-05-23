# 패킷 캡처(libpcap)

실행 환경

| Type | Version |
| :--- | :---    |
| OS   | Ubuntu 20.04.3 LTS |
| gcc  | gcc version 9.4.0 |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [네트워크 정보 확인](#네트워크-정보-확인)**

 - [pcap_findalldevs](#pcap_findalldevs)

 - [pcap_freealldevs](#pcap_freealldevs)

**3. [패킷 캡처](#패킷-캡처)**

 - [pcap_lookupnet](#pcap_lookupnet)

 - [pcap_open_live](#pcap_open_live)

 - [pcap_set_immediate_mode](#pcap_set_immediate_mode)

 - [pcap_compile](#pcap_compile)

 - [pcap_setfilter](#pcap_setfilter)

 - [pcap_loop](#pcap_loop)

**4. [패킷 분석](#패킷-분석)**

 - [ethernet](#ethernet)

 - [ip](#ip)

 - [icmp](#icmp)

 - [tcp](#tcp)

 - [udp](#udp)


# **Hello World**

Hello World

## **Hello**

안녕하세요.

### Syntax

```c++
#include <stdio.h>

void Hello(const char *msg);
```

### parameter

| parameter | description |
| :--- | :--- |
| msg | 데이터가 담긴 변수 포인터(위치) |

### Return

성공 시 0을 반환하고, 실패 시 0이 아닌 값을 반환합니다.