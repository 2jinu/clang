# libnet_build_ipv4

RFC 791 IPv4 헤더를 생성합니다.

# **Syntax**

```c++
#include <libnet.h>

libnet_ptag_t libnet_build_ipv4(uint16_t ip_len, uint8_t tos, uint16_t id, uint16_t frag, uint8_t ttl, uint8_t prot, uint16_t sum, uint32_t src, uint32_t dst, const uint8_t * payload, uint32_t payload_s, libnet_t *l, libnet_ptag_t ptag);
```

# **parameter**

| parameter | description   |
| :---      | :---          |
| ip_len    | 하위 계층을 포함한 IP 패킷의 총 길이 |
| tos       | Type of Service |
| id        | Identification |
| frag      | Fragmentation bits and offset |
| ttl       | Time to Live |
| prot      | 상위 계층의 Protocol |
| sum       | Checksum (0이면 자동 계산) |
| src       | Source IPv4 Address |
| dst       | Destination IPv4 Address |
| payload   | Payload |
| payload_s | Payload 길이 |
| l         | libnet 컨텍스트 |
| ptag      | 프로토콜 태그 |

# **Return**

성공 시 프로토콜 태그을 반환하고, 실패 시 -1을 반환합니다.