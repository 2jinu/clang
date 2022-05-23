# Functions

파일 포인터의 위치를 파일의 시작 부분으로 변경합니다.

[fseek](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fseek)(stream, 0L, SEEK_SET)과 유사하지만, rewind를 포인터가 성공적으로 이동되었는지를 파악할 수 없습니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <stdio.h>

void rewind(FILE* stream);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| stream    | FILE 구조체에 대한 포인터 |

# **Return**

없음