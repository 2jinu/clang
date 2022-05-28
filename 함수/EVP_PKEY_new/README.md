# EVP_PKEY_new

공개 키와 개인 키를 저장하기 위해 EVP_PKEY 구조를 할당합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

EVP_PKEY *EVP_PKEY_new(void);
```

# **parameter**

없음

# **Return**

성공 시 EVP_PKEY에 대한 포인터를 반환하고, 실패 시 NULL값을 반환합니다.