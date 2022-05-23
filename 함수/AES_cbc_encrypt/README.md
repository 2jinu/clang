# AES_cbc_encrypt

암호화 키를 AES_KEY 구조체로 변환합니다.

# **Syntax**

```c++
#include <openssl/aes.h>

void AES_cbc_encrypt(const unsigned char *in, unsigned char *out, size_t length, const AES_KEY *key, unsigned char *ivec, const int enc);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| in        | 원본 데이터 버퍼 |
| out       | 암호화된 데이터가 저장될 데이터 버퍼 |
| length    | 암호화 크기 |
| key       | 키가 저장된 AES 키 구조체 포인터 |
| ivec      | 초기 벡터 |
| enc       | 암/복호화 구분<br>1 (AES_ENCRYPT) : 암호화<br>0 (AES_DECRYPT) : 복호화 |

# **Return**

없음