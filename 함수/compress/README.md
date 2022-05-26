# compress

데이터를 압축합니다.

# **Syntax**

```c++
#include <zlib.h>

int compress(Bytef *dest, uLongf *destLen, const Bytef *source, uLong sourceLen);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| dest | 압축 후 저장될 데이터 버퍼 |
| destLen | dest의 최대 크기 |
| source | 압축할 데이터 버퍼 |
| sourceLen | source의 크기 |

# **Return**

성공 시 Z_OK(0)을 반환하고, 메모리 부족 시 Z_MEM_ERROR(-4), 버퍼의 크기가 부족 시 Z_BUF_ERROR(-5)를 반환합니다.