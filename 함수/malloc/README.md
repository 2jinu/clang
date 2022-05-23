# malloc

메모리 블록을 할당합니다.

# **Syntax**

```c++
#include <stdio.h> // or <malloc.h>

void *malloc(size_t size);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| size      | 할당할 바이트 |

# **Return**

성공 시 void형 포인터를 반환하고, 실패 시 NULL을 반환합니다.