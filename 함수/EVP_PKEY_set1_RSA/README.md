# EVP_PKEY_set1_RSA

EVP_PKEY가 참조하는 키를 RSA 키로 설정합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

int EVP_PKEY_set1_RSA(EVP_PKEY *pkey, RSA *key);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| pkey | EVP_PKEY에 대한 포인터 |
| key | RSA에 대한 포인터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.