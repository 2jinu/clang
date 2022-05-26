# LZ4_compress_default

데이터를 압축합니다.

# **Syntax**

```c++
#include <lz4.h>

int LZ4_compress_default(const char* src, char* dst, int srcSize, int dstCapacity);
```

# **parameter**

| parameter     | description |
| :---          | :--- |
| src           | 압축할 데이터 버퍼 |
| dst           | 압축 후 저장될 데이터 버퍼 |
| srcSize       | src의 크기 (최대 : LZ4_MAX_INPUT_SIZE = 2113929216) |
| dstCapacity   | dst의 최대 크기 |

# **Return**

성공 시 dst의 크기를 반환하고, 실패 시 0을 반환합니다.