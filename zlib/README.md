# zlib

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| zlib1g-dev    | 1:1.2.11.dfsg-2ubuntu1.3  |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [zlib](#zlib)**

 - [Compress](#Compress)

 - [Decompress](#Decompress)

 - [압축 파일 생성](#압축-파일-생성)

 - [압축 파일 해제](#압축-파일-해제)

**3. [Full Code](#Full-Code)**



# **패키지 설치**

zlib1g-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install zlib1g-dev
```

컴파일시 libz를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lz -o outputfile
```

# **zlib**

압축을 위한 데이터를 준비하자.

```cpp
const char *Data = "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.";
unsigned long DataLength  = strlen(Data);
```

## **Compress**

[compress](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/compress)로 데이터를 압축하자.

```cpp
Byte CompressedData[DataLength * 2 + 13] = { '\0' };
unsigned long CompressedDataLength = sizeof(CompressedData);
compress(CompressedData, &CompressedDataLength, (Byte *)Data, DataLength);
```

[compress2](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/compress2)를 이용하여 level별 압축을 진행할 수 도 있다.

```cpp
unsigned long Compressed2DataLength = DataLength + (DataLength * 0.1);
Byte Compressed2Data[Compressed2DataLength] = { '\0' };
if (0 != compress2(Compressed2Data, &Compressed2DataLength, (Byte *)Data, DataLength, 9)) printf("compress2 error\n");
```

## **Decompress**

[uncompress](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/uncompress)로 데이터를 압축 해제하자.

```cpp
Byte DecompressedData[CompressedDataLength * 2 + 13] = { '\0' };
unsigned long DecompressedDataLength = sizeof(DecompressedData);
uncompress(DecompressedData, &DecompressedDataLength, CompressedData, CompressedDataLength);
```

## **압축 파일 생성**

[gzopen](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/gzopen)로 압축파일을 열고 [gzwrite](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/gzwrite)로 데이터를 쓰자.

```cpp
gzFile ZipFilePointer;
int GzError = 0;
if ((ZipFilePointer = gzopen("TestGZip.gz", "wb")) == Z_NULL) {
    return -1;
}
if (gzwrite(ZipFilePointer, Data, DataLength) < 0) {
    printf("gzwrite error %s\n", gzerror(ZipFilePointer, &GzError));
}
if (gzclose(ZipFilePointer) != Z_OK) printf("gzclose error %s\n", gzerror(ZipFilePointer, &GzError));
```

## **압축 파일 해제**

[gzopen](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/gzopen)로 압축파일을 열고 [gzread](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/gzread)로 [gzeof](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/gzeof)까지 데이터를 읽자.

```cpp
char ZipReadData[256] = { '\0' };
if ((ZipFilePointer = gzopen("TestGZip.gz", "rb")) == Z_NULL) {
    return -1;
}
while ( !gzeof(ZipFilePointer) )
{
    int ReadSize = gzread(ZipFilePointer, ZipReadData, sizeof(ZipReadData));
    if (ReadSize < 0) printf("gzread error %s\n", gzerror(ZipFilePointer, &GzError));
    ZipReadData[ReadSize] = '\0';
}
if (gzclose(ZipFilePointer) != Z_OK) printf("gzclose error %s\n", gzerror(ZipFilePointer, &GzError));
```

# **Full Code**

```cpp
#include <zlib.h>
#include <string.h>
#include <stdio.h>

int main(void){
    const char *Data = "Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book. It has survived not only five centuries, but also the leap into electronic typesetting, remaining essentially unchanged. It was popularised in the 1960s with the release of Letraset sheets containing Lorem Ipsum passages, and more recently with desktop publishing software like Aldus PageMaker including versions of Lorem Ipsum.";
    unsigned long DataLength  = strlen(Data);
    printf("DataLength : %d\n", DataLength);

    unsigned long CompressedDataLength = DataLength + (DataLength * 0.1);
    Byte CompressedData[CompressedDataLength] = { '\0' };
    if (0 != compress(CompressedData, &CompressedDataLength, (Byte *)Data, DataLength)) printf("compress error\n");
    printf("CompressedDataLength : %d\n", CompressedDataLength);    

    unsigned long DecompressedDataLength = CompressedDataLength * 2;
    Byte DecompressedData[DecompressedDataLength] = { '\0' };
    if (0 != uncompress(DecompressedData, &DecompressedDataLength, CompressedData, CompressedDataLength)) printf("uncompress error\n");
    printf("DecompressedDataLength : %d\n", DecompressedDataLength);

    if (memcmp(Data, DecompressedData, DecompressedDataLength) != 0) printf("Validation Failed.\n");
    else printf("Validation Success\n");

    unsigned long Compressed2DataLength = DataLength + (DataLength * 0.1);
    Byte Compressed2Data[Compressed2DataLength] = { '\0' };
    if (0 != compress2(Compressed2Data, &Compressed2DataLength, (Byte *)Data, DataLength, 9)) printf("compress2 error\n");
    printf("Compressed2DataLength : %d\n", Compressed2DataLength);

    unsigned long Decompressed2DataLength = Compressed2DataLength * 2;
    Byte Decompressed2Data[Decompressed2DataLength] = { '\0' };
    if (0 != uncompress(Decompressed2Data, &Decompressed2DataLength, Compressed2Data, Compressed2DataLength)) printf("uncompress error\n");
    printf("Decompressed2DataLength : %d\n", Decompressed2DataLength);

    if (memcmp(Data, Decompressed2Data, Decompressed2DataLength) != 0) printf("Validation Failed.\n");
    else printf("Validation Success\n");

    gzFile ZipFilePointer;
    int GzError = 0;
    if ((ZipFilePointer = gzopen("TestGZip.gz", "wb")) == Z_NULL) {
        return -1;
    }
    if (gzwrite(ZipFilePointer, Data, DataLength) < 0) {
        printf("gzwrite error %s\n", gzerror(ZipFilePointer, &GzError));
    }
    if (gzclose(ZipFilePointer) != Z_OK) printf("gzclose error %s\n", gzerror(ZipFilePointer, &GzError));
    


    char ZipReadData[256] = { '\0' };
    if ((ZipFilePointer = gzopen("TestGZip.gz", "rb")) == Z_NULL) {
        return -1;
    }
    while ( !gzeof(ZipFilePointer) )
    {
        int ReadSize = gzread(ZipFilePointer, ZipReadData, sizeof(ZipReadData));
        if (ReadSize < 0) printf("gzread error %s\n", gzerror(ZipFilePointer, &GzError));
        ZipReadData[ReadSize] = '\0';
    }
    if (gzclose(ZipFilePointer) != Z_OK) printf("gzclose error %s\n", gzerror(ZipFilePointer, &GzError));
    
    return 0;
}
```