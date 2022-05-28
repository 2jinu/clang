# BN_set_word

BIGNUM에 데이터를 설정합니다.

# **Syntax**

```c++
#include <openssl/bn.h>

int BN_set_word(BIGNUM *a, BN_ULONG w);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| a         | BIGNUM에 대한 포인터 |
| w         | BIGNUM에 설정할 데이터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0이 아닌 값을 반환합니다.