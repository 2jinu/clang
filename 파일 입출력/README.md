# 파일 입출력

# **INDEX**

**1. [파일 열고 닫기](#파일-열고-닫기)**

 - [fopen_s](#fopen_s)

 - [fclose](#fclose)

**2. [파일 읽고 쓰기](#파일-읽고-쓰기)**

 - [fseek](#fseek)

 - [ftell](#ftell)

 - [rewind](#rewind)

 - [fread_s](#fread_s)

 - [fwrite](#fwrite)


# **파일 열고 닫기**

파일을 다루기 위해서 파일을 열고 닫습니다.

```c++
#include <windows.h>
#include <stdio.h>

int main(void)
{
    FILE *FilePointer;
    errno_t err;
    const char *FileName = ".\\Hello World.txt";

    err = fopen_s(&FilePointer, FileName, "r");
    if (err != 0 || FilePointer == NULL) {
        printf("cannot open file : %s\n", FileName);
        return 0;
    }

    fclose(FilePointer);
	return 0;
}
```

## **fopen_s**

지정한 파일을 엽니다.

### Syntax

```c++
#include <stdio.h>

errno_t fopen_s(FILE** pFile, const char *filename, const char *mode);
```

### parameter

| parameter | description |
| :--- | :--- |
| pFile | 수신할 파일 포인터에 대한 포인터 |
| filename | 파일 이름 |
| mode | 접근 방법 |

mode는 다음과 같이 다양한 방법이 있습니다.

| mode | description |
| :---: | :--- |
| r | 읽기<br>파일이 없으면 에러 |
| w | 쓰기<br>파일이 없으면 새로 생성<br>파일이 있으면 모든 내용을 삭제 |
| a | 추가<br>파일이 없으면 새로 생성<br>파일이 있으면 내용 뒤에 추가로 작성 |
| r+ | 읽고 쓰기<br>파일이 없으면 에러 |
| w+ | 읽고 쓰기<br>파일이 없으면 새로 생성<br>파일이 있으면 모든 내용을 삭제 |
| a+ | 읽고 추가<br>파일이 없으면 새로 생성<br>파일이 있으면 내용 뒤에 추가로 작성 |
| t | 텍스트 모드<br>r,w,a와 같이 사용 |
| b | 바이너리 모드<br>r,w,a와 같이 사용 |

### Return

성공 시 0을 반환하고 실패 시 오류 코드를 반환합니다.

## **fclose**

파일 스트림을 닫습니다.

### Syntax

```c++
#include <stdio.h>

int fclose(FILE* stream);
```

### parameter

| parameter | description |
| :--- | :--- |
| stream | FILE 구조체에 대한 포인터 |

### Return

성공 시 0을 반환하고, 실패 시 EOF를 반환합니다.

# **파일 읽고 쓰기**

파일을 내용을 읽고 수정합니다.

파일을 읽기 위해서 Hello World.txt를 생성 후 미리 데이터를 넣어놓습니다.

* Hello World.txt

    abcdefghijklmnopqrstuvwxyz0123456789!

```c++
#include <windows.h>
#include <stdio.h>

int main(void)
{
    FILE *FilePointer;
    errno_t err;
    const char *FileName = ".\\Hello World.txt";

    err = fopen_s(&FilePointer, FileName, "r+t");
    if (err != 0 || FilePointer == NULL) {
        printf("cannot open file : %s\n", FileName);
        return 0;
    }

    int Result;
    Result = fseek(FilePointer, 0L, SEEK_END);
    if (Result != 0) {
        printf("fseek error : %d\n", Result);
        fclose(FilePointer);
        return 0;
    }

    long FileLength = ftell(FilePointer);
    rewind(FilePointer);

    char *FileData  = (char*)malloc(sizeof(char) * FileLength);
    if (FileData == NULL) {
        printf("malloc error\n");
        fclose(FilePointer);
        return 0;
    }
    int ReadCount       = fread_s(FileData, FileLength, FileLength, 1, FilePointer);
    printf("read data  = %s\n", FileData);
    free(FileData);

    const char* WriteData = "가나다라마바사";
    fwrite(WriteData, strlen(WriteData), 1, FilePointer);

    fclose(FilePointer);
	return 0;
}
```

파일에 데이터를 추가로 작성하였으니 결과를 확인해봅니다.

* Hello World.txt

    abcdefghijklmnopqrstuvwxyz0123456789!가나다라마바사

## **fseek**

파일 포인터를 지정된 위치로 이동합니다.

### Syntax

```c++
#include <stdio.h>

int fseek(FILE* stream, long offset, int origin);
```

### parameter

| parameter | description |
| :--- | :--- |
| stream | FILE 구조체에 대한 포인터 |
| offset | origin부터의 바이트 수 |
| origin | 초기 위치 |

origin은 stdio.h에 정의된 상수 중 하나여야 합니다.

| parameter | description |
| :--- | :--- |
| SEEK_CUR | 파일 포인터의 현재 위치 |
| SEEK_END | 파일 끝 |
| SEEK_SET | 파일 시작 |

### Return

반환 값을 성공 시 0을 반환하고 실패 시 0이 아닌 값을 반환합니다.

## **ftell**

파일 포인터의 현재 위치를 가져옵니다.

### Syntax

```c++
#include <stdio.h>

long ftell(FILE* stream);
```

### parameter

| parameter | description |
| :--- | :--- |
| stream | FILE 구조체에 대한 포인터 |

### Return

현재 파일 포인터의 위치를 반환하며 포인터의 위치가 SEEK_END일 시 파일의 크기와 같습니다.

## **rewind**

파일 포인터의 위치를 파일의 시작 부분으로 변경합니다.

fseek(stream, 0L, SEEK_SET)과 유사하지만, rewind를 포인터가 성공적으로 이동되었는지를 파악할 수 없습니다.

### Syntax

```c++
#include <stdio.h>

void rewind(FILE* stream);
```

### parameter

| parameter | description |
| :--- | :--- |
| stream | FILE 구조체에 대한 포인터 |

## **fread_s**

파일 스트림에서 데이터를 읽습니다.

### Syntax

```c++
#include <stdio.h>

size_t fread_s(void *buffer, size_t bufferSize, size_t elementSize, size_t count, FILE *stream);
```

### parameter

| parameter | description |
| :--- | :--- |
| buffer | 데이터를 담을 변수 포인터(위치) |
| bufferSize | buffer의 크기 |
| elementSize | 읽을 크기 |
| count | 읽을 수 |
| stream | FILE 구조체에 대한 포인터 |

### Return

buffer로 읽은 count를 반환하며, 오류가 발생하거나 count만큼 읽기 전에 파일 끝(EOF)에 도달하면 count보다 작을 수 있습니다.

## **fwrite**

파일 스트림에서 데이터를 작성합니다.

### Syntax

```c++
#include <stdio.h>

size_t fwrite(const void *buffer, size_t size, size_t count, FILE *stream);
```

### parameter

| parameter | description |
| :--- | :--- |
| buffer | 데이터가 담긴 변수 포인터(위치) |
| size | 작성할 데이터의 크기 |
| count | 작성 수 |
| stream | FILE 구조체에 대한 포인터 |

### Return

파일에 작성한 count를 반환하며, 오류가 발생하면 count보다 작을 수 있습니다.