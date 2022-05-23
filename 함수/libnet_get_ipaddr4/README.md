# libnet_get_ipaddr4

초기화된 장치의 IP 주소를 반환합니다.

# **Syntax**

```c++
#include <libnet.h>

uint32_t libnet_get_ipaddr4(libnet_t *l);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| l         | libnet 컨텍스트 |

# **Return**

성공 시 big endian형식의 IP 주소를 반환하고, 실패 시 -1을 반환합니다.