# EVP_DecryptUpdate

데이터를 복호화합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

int EVP_DecryptUpdate(EVP_CIPHER_CTX *ctx, unsigned char *out, int *outl, unsigned char *in, int inl); 
```

# **parameter**

| parameter | description |
| :---      | :--- |
| ctx | 암호화 컨텍스트 |
| out | 복호화의 결과가 저장될 버퍼 |
| outl | out의 크기가 저장 |
| in | 암호문 버퍼 |
| inl | in의 크기 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.