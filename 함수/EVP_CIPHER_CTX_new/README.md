# EVP_CIPHER_CTX_new

암호화 컨텍스트를 할당하고 반환합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

EVP_CIPHER_CTX *EVP_CIPHER_CTX_new(void);
```

# **parameter**

없음

# **Return**

성공 시 암호화 컨텍스트를 반환하고, 실패 시 NULL값을 반환합니다.