# libnet_name2addr4

"."으로 구분된 10진수 문자열을 IPv4로 변환합니다.

# **Syntax**

```c++
#include <libnet.h>

uint32_t libnet_name2addr4(libnet_t *l, char *host_name, uint8_t use_name);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| l         | libnet 컨텍스트 |
| host_name | 문자열의 IPv4 주소 |
| use_name  | LIBNET_RESOLVE : DNS 참조하는 경우<br>LIBNET_DONT_RESOLVE : DNS 참조를 실패하거나 참조하지 않는 경우 |

# **Return**

성공 시 IPv4 주소를 반환하고, 실패 시 -1을 반환합니다.