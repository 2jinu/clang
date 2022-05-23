# libnet_build_tcp

RFC 793 TCP 헤더를 생성합니다.

# **Syntax**

```c++
#include <libnet.h>

libnet_ptag_t libnet_build_tcp(uint16_t sp, uint16_t dp, uint32_t seq, uint32_t ack, uint8_t control, uint16_t win, uint16_t sum, uint16_t urg, uint16_t len, const uint8_t *payload, uint32_t payload_s, libnet_t *l, libnet_ptag_t ptag);
```

# **parameter**

| parameter | description   |
| :---      | :---          |
| sp        | Source Port |
| dp        | Destination Port |
| seq       | Sequence Number |
| ack       | Acknowledgement Number |
| control   | Flags |
| win       | Window |
| sum       | Checksum (0이면 자동 계산) |
| urg       | Urgent Pointer |
| len       | Total Length of TCP |
| payload   | Payload |
| payload_s | Payload 길이 |
| l         | libnet 컨텍스트 |
| ptag      | 프로토콜 태그 |

# **Return**

성공 시 프로토콜 태그을 반환하고, 실패 시 -1을 반환합니다.