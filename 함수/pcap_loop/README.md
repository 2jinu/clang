# Functions

패킷을 캡처 후 콜백 함수에 전달하는 함수입니다. cnt만큼 패킷을 캡처합니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <pcap/pcap.h>

int pcap_loop(pcap_t *p, int cnt, pcap_handler callback, u_char *user);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| p         | 패킷 캡처 핸들 |
| cnt       | 읽은 패킷의 수<br>-1 혹은 0이면 무한으로 캡처 |
| callback  | 패킷울 처리할 함수 |
| user      | 사용자 정의 데이터 |

# **Return**

성공 시 읽은 패킷의 수 혹은 0(EOF)을 반환하고, 실패 시 -1을 반환합니다.