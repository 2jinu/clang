# gzeof

압축파일의 끝에 도달했는지 검사합니다.

# **Syntax**

```c++
#include <zlib.h>

int gzeof(gzFile file);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| file      | gzFile 객체 |

# **Return**

파일의 끝에 도달 시 1을 반환하고 그렇지 않으면 0을 반환합니다.