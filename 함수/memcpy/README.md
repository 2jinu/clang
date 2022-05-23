# memcpy

source에서 바이트 만큼을 destination에 복사한다.

# **Syntax**

```c++
#include <string.h>

void* memcpy(void* destination, const void* source, size_t num);
```

# **parameter**

| parameter     | description |
| :---          | :--- |
| destination   | 데이터가 복사될 곳의 주소 |
| source        | 복사할 데이터의 주소 |
| num           | 복사할 데이터의 바이트 수 |

# **Return**

destination에 대한 포인터가 반환됩니다.