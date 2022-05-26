# EVP_CIPHER_CTX_free

암호화 컨텍스트의 모든 정보를 지우고 컨텍스트의 메모리를 해제합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

void EVP_CIPHER_CTX_free(EVP_CIPHER_CTX *ctx);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| ctx | 암호화 컨텍스트 |

# **Return**

없음