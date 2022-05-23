# pcap_findalldevs

네트워크 장치 목록을 구성합니다. 장치가 없을 경우 NULL로 설정되지만 실패로 간주되지 않습니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

int pcap_findalldevs(pcap_if_t **alldevsp, char *errbuf);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| alldevsp  | pcap_if_t 구조체 포인터에 대한 위치 |
| errbuf    | 에러 메세지가 담길 버퍼 |

# **Return**

성공 시 0을 반환하고, 실패 시 PCAP_ERROR(-1)를 반환합니다.