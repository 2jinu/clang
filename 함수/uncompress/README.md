# uncompress

데이터를 압축 해제합니다.

# **Syntax**

```c++
#include <zlib.h>

int uncompress(Bytef *dest, uLongf *destLen, const Bytef *source, uLong sourceLen);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| dest | 압축해제 후 저장될 데이터 버퍼 |
| destLen | dest의 최대 크기 |
| source | 압축된 데이터 버퍼 |
| sourceLen | source의 크기 |

# **Return**

성공 시 Z_OK(0)을 반환하고, 메모리 부족 시 Z_MEM_ERROR(-4), 버퍼의 크기가 부족 시 Z_BUF_ERROR(-5), 압축된 데이터가 잘못됐을 경우 Z_DATA_ERROR(-3)을 반환합니다.