# libnet_build_udp

RFC 768 UDP 헤더를 생성합니다.

# **Syntax**

```c++
#include <libnet.h>

libnet_ptag_t libnet_build_udp(uint16_t sp, uint16_t dp, uint16_t len, uint16_t sum, const uint8_t *payload, uint32_t payload_s, libnet_t *l, libnet_ptag_t ptag);
```

# **parameter**

| parameter | description   |
| :---      | :---          |
| sp        | Source Port |
| dp        | Destination Port |
| len       | Total Length of UDP |
| sum       | Checksum (0이면 자동 계산) |
| payload   | Payload |
| payload_s | Payload 길이 |
| l         | libnet 컨텍스트 |
| ptag      | 프로토콜 태그 |

# **Return**

성공 시 프로토콜 태그을 반환하고, 실패 시 -1을 반환합니다.