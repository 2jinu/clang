# pcap_lookupnet

네트워크 장치의 네트워크 주소와 서브넷 마스크를 결정합니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

int pcap_lookupnet(const char *device, bpf_u_int32 *netp, bpf_u_int32 *maskp, char *errbuf);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| device    | 네트워크 인터페이스 명 |
| netp      | 네트워크 주소가 저장될 포인터 |
| maskp     | 서브넷 마스크 주소가 저장될 포인터 |
| errbuf    | 에러 메세지가 담길 버퍼 |

# **Return**

성공 시 0을 반환하고, 실패 시 PCAP_ERROR(-1)를 반환합니다.