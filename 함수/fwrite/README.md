# Functions

파일 스트림에서 데이터를 작성합니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <stdio.h>

size_t fwrite(const void *buffer, size_t size, size_t count, FILE *stream);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| buffer    | 데이터가 담긴 변수 포인터(위치) |
| size      | 작성할 데이터의 크기 |
| count     | 작성 수 |
| stream    | FILE 구조체에 대한 포인터 |

# **Return**

파일에 작성한 count를 반환하며, 오류가 발생하면 count보다 작을 수 있습니다.