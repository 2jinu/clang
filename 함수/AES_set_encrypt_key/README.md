# AES_set_encrypt_key

암호화 키를 AES_KEY 구조체로 변환합니다.

# **Syntax**

```c++
#include <openssl/aes.h>

int AES_set_encrypt_key(const unsigned char *userKey, const int bits, AES_KEY *key);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| userKey   | 키 값 |
| bits      | 키 길이 (128, 192, 256) |
| key       | 키가 저장될 AES 키 구조체 포인터 |

# **Return**

성공 시 0을 반환하고, userKey 혹은 key가 NULL값일 때 -1을 반환하고, 지원되지 않는 bits값일 때 -2를 아닌 값을 반환합니다.