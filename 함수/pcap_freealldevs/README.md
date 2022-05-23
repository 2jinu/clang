# pcap_freealldevs

[pcap_findalldevs](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_findalldevs)에 의해 할당된 목록을 해제합니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

void pcap_freealldevs(pcap_if_t *alldevs);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| alldevs   | pcap_if_t 구조체 포인터 |

# **Return**

없음