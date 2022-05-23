# 파일 입출력

# **INDEX**

**1. [파일 열고 닫기](#파일-열고-닫기)**

 - [파일 열기](#파일-열기)

 - [파일 닫기](#파일-닫기)

**2. [파일 읽고 쓰기](#파일-읽고-쓰기)**

 - [파일 크기 구하기](#파일-크기-구하기)

 - [파일 읽기](#파일-읽기)

 - [파일 쓰기](#파일-쓰기)

**3. [Full Code](#Full-Code)**


# **파일 열고 닫기**

## **파일 열기**

[fopen_s](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fopen_s)를 이용하여 지정한 파일을 엽니다.

```c++
FILE *FilePointer;
errno_t err;
const char *FileName = ".\\Hello World.txt";

err = fopen_s(&FilePointer, FileName, "r");
if (err != 0 || FilePointer == NULL) {
    printf("cannot open file : %s\n", FileName);
    return 0;
}
```

## **파일 닫기**

[fclose](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fclose)를 이용하여 지정한 파일 스트림을 닫습니다.

```c++
fclose(FilePointer);
```

## **Full Code**

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


# **파일 읽고 쓰기**

파일을 읽기 위해서 Hello World.txt를 생성 후 미리 데이터를 넣어놓습니다.

* Hello World.txt

    abcdefghijklmnopqrstuvwxyz0123456789!

## **파일 크기 구하기**

파일을 읽기 위해서 [fseek](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fseek)과 [ftell](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/ftell)을 이용하여 파일의 크기를 구합니다.

```c++
int Result;
Result = fseek(FilePointer, 0L, SEEK_END);
if (Result != 0) {
    printf("fseek error : %d\n", Result);
    fclose(FilePointer);
    return 0;
}

long FileLength = ftell(FilePointer);
rewind(FilePointer);
```

## **파일 읽기**

구한 파일 크기를 이용하여 [malloc]을 통해 데이터를 담을 버퍼를 생성 후 [fread_s](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fread_s)를 통해 파일의 데이터를 읽습니다.

```c++
char *FileData  = (char*)malloc(sizeof(char) * FileLength);
if (FileData == NULL) {
    printf("malloc error\n");
    fclose(FilePointer);
    return 0;
}
int ReadCount       = fread_s(FileData, FileLength, FileLength, 1, FilePointer);
printf("read data  = %s\n", FileData);
free(FileData);
```

## **파일 쓰기**

[fwrite](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fwrite)를 이용하여 파일에 데이터를 씁니다.

```c++
const char* WriteData = "가나다라마바사";
fwrite(WriteData, strlen(WriteData), 1, FilePointer);
```

# **Full Code**

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
