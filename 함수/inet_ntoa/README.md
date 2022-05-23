# inet_ntoa

ipv4 네트워크 주소를 ASCII 문자열로 변환합니다.

# **Syntax**

```c++
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

char *inet_ntoa(struct in_addr in);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| in        | in_addr 구조체 |

# **Return**

성공 시 "x.x.x.x"형식의 char형 포인터를 반환하고, 실패 시 NULL을 반환합니다.