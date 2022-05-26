# gzclose

gzFile 객체를 해제합니다.

# **Syntax**

```c++
#include <zlib.h>

int gzclose(gzFile file);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| file      | gzFile 객체 |

# **Return**

성공 시 Z_OK(0)을 반환하고, 실패 시 음수를 반환합니다.