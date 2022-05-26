# EVP_EncryptInit_ex

암호 유형에 따른 암호화를 위한 암호화 컨텍스트를 설정합니다.

# **Syntax**

```c++
#include <openssl/evp.h>

int EVP_EncryptInit_ex(EVP_CIPHER_CTX *ctx, const EVP_CIPHER *type, ENGINE *impl, const unsigned char *key, const unsigned char *iv);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| ctx | 암호화 컨텍스트 |
| type | 암호화 모드 |
| impl | 암호 유형<br>NULL일 시 기본으로 설정 |
| key | 암호화 키 |
| iv | 암호화 초기 벡터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.