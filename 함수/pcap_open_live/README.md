# pcap_open_live

패킷 캡처를 위한 핸들을 얻습니다.

# **Syntax**

```c++
#include <pcap/pcap.h>

pcap_t *pcap_open_live(const char *device, int snaplen, int promisc, int to_ms, char *errbuf);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| device    | 네트워크 인터페이스 명<br>Linux Kernel 2.2 이상의 경우 "any" 혹은 NULL로 할 경우 모든 네트워크 인터페이스에 대하여 패킷 캡처를 수행 |
| snaplen   | 캡처할 최대 Byte 수 |
| promisc   | 패킷 캡처 모드<br>기본값은 1<br>1 (PROMISCUOUS): 동일 네트워크의 모든 패킷을 캡처<br>0 (NON-PROMISCUOUS): 자신에게 온 패킷만 캡처 |
| to_ms     | snaplen만큼 버퍼가 차거나, 시간만큼 기다린 후 패킷 전달|
| errbuf    | 에러 메세지가 담길 버퍼 |

# **Return**

성공 시 핸들을 반환하고, 실패 시 NULL을 반환합니다.