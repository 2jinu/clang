# ioctl

특수 파일의 기본 장치 매개변수를 조작합니다.

# **Syntax**

```c++
#include <sys/ioctl.h>

int ioctl(int fd, unsigned long request, ...);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| fd        | 파일 디스크립터 |
| request   | 장치에서 요청할 코드 |

# **Return**

성공 시 0을 반환하고, 실패 시 -1을 반환합니다.