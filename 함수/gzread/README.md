# gzread

압축 파일의 데이터를 읽습니다.

# **Syntax**

```c++
#include <zlib.h>

int gzread(gzFile file, voidp buf, unsigned int len);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| file | gzFile 객체 |
| buf | 압축해제 후 저장될 데이터 버퍼 |
| len | 압축해제 할 최대 크기 |

# **Return**

성공 시 압축해제된 데이터 크기를 반환하고, 실패 시 음수를 반환합니다.