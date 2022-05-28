# PEM_write_RSAPrivateKey

파일로 PEM 형식의 개인키를 저장합니다.

# **Syntax**

```c++
#include <openssl/pem.h>

int PEM_write_RSAPrivateKey(FILE *fp, RSA *x, const EVP_CIPHER *enc, unsigned char *kstr, int klen, pem_password_cb *cb, void *u);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| fp | 파일 포인터 |
| x | RSA 포인터 |
| enc | 암호화 알고리즘 |
| kstr | 암호화시 사용되는 암호 버퍼 |
| klen | kstr의 크기 |
| cb | 암호화 시 kstr이 NULL이 아닐 경우 호출 되는 콜백 함수 |
| u | 사용자 데이터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.