# EVP_EncryptFinal_ex

패딩 및 최종 블록 암호화 등 작업을 처리합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

int EVP_EncryptFinal_ex(EVP_CIPHER_CTX *ctx, unsigned char *out, int *outl);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| ctx | 암호화 컨텍스트 |
| out | 암호화의 결과가 저장될 버퍼 |
| outl | out의 크기가 저장 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.