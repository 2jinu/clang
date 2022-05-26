# gzwrite

압축 파일에 데이터를 씁니다.

# **Syntax**

```c++
#include <zlib.h>

int gzwrite(gzFile file, voidpc buf, unsigned int len);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| file | gzFile 객체 |
| buf | 압축할 데이터 버퍼 |
| len | 압축할 데이터의 크기 |

# **Return**

성공 시 buf의 데이터 크기를 반환하고, 실패 시 음수를 반환합니다.