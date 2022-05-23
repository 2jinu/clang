# pcap_compile

문자열 형태의 필터링 룰을 필터 프로그램으로 컴파일하는데 사용됩니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

int pcap_compile(pcap_t *p, struct bpf_program *fp, const char *str, int optimize, bpf_u_int32 netmask);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| p         | 패킷 캡처 핸들 |
| fp        | 패킷 필터링 룰이 저장될 프로그램 포인터 |
| str       | 문자열 형태의 필터링 룰 |
| optimize  | 수행 시 최적화 여부 |
| netmask   | 서브넷 마스크 주소가 저장된 포인터 |

# **Return**

성공 시 0을 반환하고, 실패 시 PCAP_ERROR(-1)를 반환합니다.