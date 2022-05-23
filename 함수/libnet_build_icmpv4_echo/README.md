# libnet_build_icmpv4_echo

RFC 792 IPv4 ICMP echo 요청/응답 헤더를 생성합니다.

# **Syntax**

```c++
#include <libnet.h>

libnet_ptag_t libnet_build_icmpv4_echo(uint8_t type, uint8_t code, uint16_t sum, uint16_t id, uint16_t seq, const uint8_t *payload, uint32_t payload_s, libnet_t *l, libnet_ptag_t ptag);
```

# **parameter**

| parameter | description   |
| :---      | :---          |
| type      | Type (ICMP_ECHO 혹은 ICMP_ECHOREPLY)  |
| code      | Code |
| sum       | Checksum (0이면 자동 계산) |
| id        | Identifier |
| seq       | Sequence Number |
| payload   | Payload |
| payload_s | Payload 길이 |
| l         | libnet 컨텍스트 |
| ptag      | 프로토콜 태그 |

# **Return**

성공 시 프로토콜 태그을 반환하고, 실패 시 -1을 반환합니다.