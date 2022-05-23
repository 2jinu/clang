# libnet_write

미리 작성된 패킷을 네트워크를 통해 전송합니다.

# **Syntax**

```c++
#include <libnet.h>

int libnet_get_prand(libnet_t *l);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| l         | libnet 컨텍스트 |

# **Return**

성공 시 전송한 Byte 수를 반환하고, 실패 시 -1을 반환합니다.