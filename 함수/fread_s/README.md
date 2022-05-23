# fread_s

파일 스트림에서 데이터를 읽습니다.

# **Syntax**

```c++
#include <stdio.h>

size_t fread_s(void *buffer, size_t bufferSize, size_t elementSize, size_t count, FILE *stream);
```

# **parameter**

| parameter   | description |
| :---        | :--- |
| buffer      | 데이터를 담을 변수 포인터(위치) |
| bufferSize  | buffer의 크기 |
| elementSize | 읽을 크기 |
| count       | 읽을 수 |
| stream      | FILE 구조체에 대한 포인터 |

# **Return**

buffer로 읽은 count를 반환하며, 오류가 발생하거나 count만큼 읽기 전에 파일 끝(EOF)에 도달하면 count보다 작을 수 있습니다.