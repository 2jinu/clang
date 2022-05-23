# ftell

파일 포인터의 현재 위치를 가져옵니다.

# **Syntax**

```c++
#include <stdio.h>

long ftell(FILE* stream);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| stream    | FILE 구조체에 대한 포인터 |

# **Return**

현재 파일 포인터의 위치를 반환하며 포인터의 위치가 SEEK_END일 시 파일의 크기와 같습니다.