# fseek

파일 포인터를 지정된 위치로 이동합니다.

# **Syntax**

```c++
#include <stdio.h>

int fseek(FILE* stream, long offset, int origin);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| stream    | FILE 구조체에 대한 포인터 |
| offset    | origin부터의 바이트 수 |
| origin    | 초기 위치 |

origin은 stdio.h에 정의된 상수 중 하나여야 합니다.

| parameter | description |
| :---      | :--- |
| SEEK_CUR  | 파일 포인터의 현재 위치 |
| SEEK_END  | 파일 끝 |
| SEEK_SET  | 파일 시작 |

# **Return**

반환 값을 성공 시 0을 반환하고 실패 시 0이 아닌 값을 반환합니다.