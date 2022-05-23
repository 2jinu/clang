# libnet_init

libnet 환경을 초기화하고 컨텍스트를 반환합니다.

# **Syntax**

```c++
#include <libnet.h>

libnet_t* libnet_init(int injection_type, const char *device, char *err_buf);
```

# **parameter**

| parameter         | description |
| :---              | :--- |
| injection_type    | LIBNET_LINK<br>LIBNET_LINK_ADV<br>LIBNET_RAW4<br>LIBNET_RAW4_ADV<br>LIBNET_RAW6<br>LIBNET_RAW6_ADV |
| device            | 네트워크 인터페이스 이름 |
| err_buf           | 에러 메세지가 담길 버퍼 |

# **Return**

성공 시 libnet 컨텍스트를 반환하고, 실패 시 NULL을 반환합니다.