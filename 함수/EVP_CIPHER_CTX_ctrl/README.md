# EVP_CIPHER_CTX_ctrl

다양한 암호에 대해 특정 매개변수를 설정할 수 있습니다.

# **Syntax**

```c++
#include <openssl/evp.h>

int EVP_CIPHER_CTX_ctrl(EVP_CIPHER_CTX *ctx, int type, int p1, void *p2);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| ctx | 암호화 컨텍스트 |
| type | 작업 유형 |
| p1 | 작업에 대한 매개 변수 |
| p2 | 작업에 대한 추가 데이터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.