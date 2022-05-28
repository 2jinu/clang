# PEM_write_X509

파일로 인증서를 저장합니다.

# **Syntax**

```c++
#include <openssl/pem.h>

int PEM_write_X509(FILE *fp, X509 *x);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| fp | 파일 포인터 |
| x | X509에 대한 포인터 |

# **Return**

성공 시 1을 반환하고, 실패 시 0을 반환합니다.