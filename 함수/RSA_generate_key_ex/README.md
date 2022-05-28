# RSA_generate_key_ex

RSA 키 쌍을 생성합니다.

# **Syntax**

```c++
#include <openssl/rsa.h>

int RSA_generate_key_ex(RSA *rsa, int bits, BIGNUM *e, BN_GENCB *cb);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| rsa | RSA에 대한 포인터 |
| bits | 키 길이 |
| e | BIGNUM에 대한 포인터 |
| cb | 생성 콜백 함수 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.