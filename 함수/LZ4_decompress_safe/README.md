# LZ4_decompress_safe

데이터를 압축해제합니다.

# **Syntax**

```c++
#include <lz4.h>

int LZ4_decompress_safe(const char* src, char* dst, intcompressedSize, int dstCapacity);
```

# **parameter**

| parameter     | description |
| :---          | :--- |
| src           | 압축된 데이터 버퍼 |
| dst           | 압축해제 후 저장될 데이터 버퍼 |
| srcSize       | src의 크기 |
| dstCapacity   | dst의 최대 크기 |

# **Return**

성공 시 dst의 크기를 반환하고, 실패 시 음수를 반환합니다.