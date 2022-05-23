# fclose

파일 스트림을 닫습니다.

# **INDEX**

**1. [Syntax](#Syntax)**

**1. [parameter](#parameter)**

**1. [Return](#Return)**


# **Syntax**

```c++
#include <stdio.h>

int fclose(FILE* stream);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| stream    | FILE 구조체에 대한 포인터 |

# **Return**

성공 시 0을 반환하고, 실패 시 EOF를 반환합니다.