# BN_new

BIGNUM 구조를 할당하고 초기화합니다.

# **Syntax**

```c++
#include <openssl/bn.h>

BIGNUM *BN_new(void);
```

# **parameter**

없음

# **Return**

성공 시 BIGNUM에 대한 포인터을 반환하고, 실패 시 NULL값을 반환합니다.