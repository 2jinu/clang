# libnet_get_prand

mod에 의해 지정된 범위 내에서 부호 없는 의사 난수 값을 생성합니다.

# **Syntax**

```c++
#include <libnet.h>

uint32_t libnet_get_prand(int mod);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| mod       | LIBNET_PR* 상수 중 하나 (LIBNET_PR16, LIBNET_PR2, LIBNET_PR32, LIBNET_PR8, LIBNET_PRu16, LIBNET_PRu32) |

# **Return**

성공 시 1을 반환하고, 실패 시 -1을 반환합니다.