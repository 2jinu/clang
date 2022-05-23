# clang

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpcap-dev   | 1.9.1-3                   |

# **INDEX**

**1. [Hello World](#Hello-World)**

 - [Hello](#Hello)

**2. [Full Code](#Full-Code)**


# **Hello World**

[Hello World](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/Hello)

## **Hello**

안녕하세요.

### Syntax

```c++
#include <stdio.h>

void Hello(const char *msg);
```

### parameter

| parameter | description |
| :--- | :--- |
| msg | 데이터가 담긴 변수 포인터(위치) |

### Return

성공 시 0을 반환하고, 실패 시 0이 아닌 값을 반환합니다.