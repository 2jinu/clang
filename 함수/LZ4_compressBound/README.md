# LZ4_compressBound

주로 메모리 할당을 위한 압축될 수 있는 최대 크기를 반환합니다.

# **Syntax**

```c++
#include <lz4.h>

int LZ4_compressBound(int inputSize);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| inputSize | 압출할 데이터의 크기 |

# **Return**

성공 시 최대 크기를 반환하고, 실패 시 0을 반환합니다.