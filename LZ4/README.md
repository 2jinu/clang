# LZ4

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| liblz4-dev    | 1.9.2-2ubuntu0.20.04.1    |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [LZ4](#LZ4)**

 - [Compress](#Compress)

 - [Decompress](#Decompress)

**3. [Full Code](#Full-Code)**


# **패키지 설치**

liblz4-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install liblz4-dev
```

컴파일시 liblz4를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -llz4 -o outputfile
```

# **LZ4**

압축을 위한 데이터를 준비하자.

```cpp
char *Data      = "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.";
int DataLength  = strlen(Data);
```

## **Compress**

[LZ4_compressBound](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/LZ4_compressBound)와 [LZ4_compress_default](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/LZ4_compress_default)를 이용하여 데이터를 압축하자.

```cpp
int MaxCompressedLength     = LZ4_compressBound(DataLength);
char *CompressedData        = (char *)malloc(MaxCompressedLength);
int CompressedDataLength    = LZ4_compress_default(Data, CompressedData, DataLength, MaxCompressedLength);
```

## **Decompress**

[LZ4_decompress_safe](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/LZ4_decompress_safe)를 이용하여 압축을 해제하자.

```cpp
char* DeCompressedData      = (char*)malloc(DataLength);
int DeCompressedDataLength  = LZ4_decompress_safe(CompressedData, DeCompressedData, CompressedDataLength, DataLength);
```

# **Full Code**

```cpp
#include <lz4.h>
#include <string.h>
#include <stdlib.h>
#include <stdio.h>

int main(void) {
    char *Data      = "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.";
    int DataLength  = strlen(Data);

    int MaxCompressedLength     = LZ4_compressBound(DataLength);
	char *CompressedData        = (char *)malloc(MaxCompressedLength);
	int CompressedDataLength    = LZ4_compress_default(Data, CompressedData, DataLength, MaxCompressedLength);


    char* DeCompressedData      = (char*)malloc(DataLength);
	int DeCompressedDataLength  = LZ4_decompress_safe(CompressedData, DeCompressedData, CompressedDataLength, DataLength);

    if (memcmp(Data, DeCompressedData, DeCompressedDataLength) != 0) printf("Validation Failed.\n");
    else printf("Validation Success\n");
    
    free(CompressedData);
    free(DeCompressedData);

    return 0;
}
```