# memcmp

ptr1과 ptr2을 바이트만큼 비교합니다.

# **Syntax**

```c++
#include <string.h>

int memcmp(const void* ptr1, const void* ptr2, size_t num);
```

# **parameter**

| parameter     | description |
| :---          | :--- |
| ptr1          | 메모리 블록을 가리키는 포인터 |
| ptr2          | 메모리 블록을 가리키는 포인터 |
| num           | 비교할  바이트 수 |

# **Return**

두 메모리 블록이 같다면 0을 반환하고, ptr1이 더 크다면 0보다 큰 값을, 아니면 0보다 작은 값을 반환합니다.