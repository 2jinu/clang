# pcap_setfilter

패킷 캡처 핸들에 필터를 적용합니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

int pcap_setfilter(pcap_t *p, struct bpf_program *fp);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| p         | 패킷 캡처 핸들 |
| fp        | 패킷 필터링 룰이 저장된 프로그램 포인터 |

# **Return**

성공 시 0을 반환하고, 실패 시 PCAP_ERROR(-1)를 반환합니다.