# fopen_s

지정한 파일을 엽니다.

# **Syntax**

```c++
#include <stdio.h>

errno_t fopen_s(FILE** pFile, const char *filename, const char *mode);
```

# **parameter**

| parameter | description |
| :---      | :--- |
| pFile     | 수신할 파일 포인터에 대한 포인터 |
| filename  | 파일 이름 |
| mode      | 접근 방법 |

mode는 다음과 같이 다양한 방법이 있습니다.

| mode  | description |
| :---: | :--- |
| r     | 읽기<br>파일이 없으면 에러 |
| w     | 쓰기<br>파일이 없으면 새로 생성<br>파일이 있으면 모든 내용을 삭제 |
| a     | 추가<br>파일이 없으면 새로 생성<br>파일이 있으면 내용 뒤에 추가로 작성 |
| r+    | 읽고 쓰기<br>파일이 없으면 에러 |
| w+    | 읽고 쓰기<br>파일이 없으면 새로 생성<br>파일이 있으면 모든 내용을 삭제 |
| a+    | 읽고 추가<br>파일이 없으면 새로 생성<br>파일이 있으면 내용 뒤에 추가로 작성 |
| t     | 텍스트 모드<br>r,w,a와 같이 사용 |
| b     | 바이너리 모드<br>r,w,a와 같이 사용 |

# **Return**

성공 시 0을 반환하고 실패 시 오류 코드를 반환합니다.