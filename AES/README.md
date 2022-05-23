# AES

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libssl-dev    | 1.1.1f-1ubuntu2.13        |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [AES CBC](#AES-CBC)**

 - [Encrypt](#Encrypt)

 - [Decrypt](#Decrypt)

**3. [AES GCM](#AES-GCM)**

 - [Encrypt](#Encrypt)

 - [Decrypt](#Decrypt)

**4. [Full Code](#Full-Code)**


# **패키지 설치**

libssl-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libssl-dev
```

컴파일시 libssl과 libcrypto을 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lssl -lcrypto -o outputfile
```

# **AES CBC**

AES 알고리즘에 CBC모드로 암/복호화하기 위해 데이터를 준비하자.

```cpp
unsigned char IV[AES_BLOCK_SIZE]    = { 0x99, 0xaa, 0x3e, 0x68, 0xed, 0x81, 0x73, 0xa0, 0xee, 0xd0, 0x66, 0x84, 0xee, 0xd0, 0x66, 0x84 };
unsigned char KEY[32]               = { 0xee, 0xbc, 0x1f, 0x57, 0x48, 0x7f, 0x51, 0x92, 0x1c, 0x04, 0x65, 0x66, 0x5f, 0x8a, 0xe6, 0xd1, 0x65, 0x8b, 0xb2, 0x6d, 0xe6, 0xf8, 0xa0, 0x69, 0xa3, 0x52, 0x02, 0x93, 0xa5, 0x72, 0x07, 0x8f };
unsigned char Data[]                =   "\x3c\x2f\x77\x73\x65\x3a\x52\x65\x6e\x65\x77\x52\x65\x73\x70\x6f" \
                                        "\x6e\x73\x65\x3e\x3c\x2f\x53\x4f\x41\x50\x2d\x45\x4e\x56\x3a\x42" \
                                        "\x6f\x64\x79\x3e\x3c\x2f\x53\x4f\x41\x50\x2d\x45\x4e\x56\x3a\x45" \
                                        "\x6e\x76\x65\x6c\x6f\x70\x65\x3e";
unsigned char AESIV[AES_BLOCK_SIZE];
AES_KEY EncKEY, DecKEY;
int DataLength          = sizeof(Data);
int PaddingLength       = 0;
```

AES는 블록 크기가 16 Byte이기 때문에 암/복호화하기 위한 데이터가 16 Byte로 나누어 떨어져야 한다.

따라서 데이터에 패딩을 더해주자.

```cpp
if (DataLength % AES_BLOCK_SIZE != 0) PaddingLength = (AES_BLOCK_SIZE - (DataLength % AES_BLOCK_SIZE));
int PaddedDataLength    = DataLength + PaddingLength;

unsigned char *PaddedData = (unsigned char *)malloc(sizeof(unsigned char *) * PaddedDataLength);
memset(PaddedData, 0x00, PaddedDataLength);
memcpy(PaddedData, Data, DataLength);
```

## **Encrypt**

[AES_set_encrypt_key](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/AES_set_encrypt_key)를 이용하여 키를 설정하고 [memcpy](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/memcpy)를 이용하여 IV를 설정하자.

```cpp
if (AES_set_encrypt_key(KEY, sizeof(KEY) * 8, &EncKEY) < 0 ) {
    printf("AES_set_encrypt_key error \n");
    free(PaddedData);
    return -1;
}
memcpy(AESIV, IV, AES_BLOCK_SIZE);
```

[AES_cbc_encrypt](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/AES_cbc_encrypt)를 이용하여 암호화를 수행한다.

```cpp
unsigned char EncryptedData[PaddedDataLength] = {0};
AES_cbc_encrypt(PaddedData, EncryptedData, PaddedDataLength, &EncKEY, AESIV, AES_ENCRYPT);

for(int i = 0; i < sizeof(EncryptedData); i++) std::cout << std::setw(2) << std::setfill('0') << std::hex << (int)EncryptedData[i] << std::dec << ' ';
std::cout << std::endl;
```

## **Decrypt**

[AES_set_encrypt_key](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/AES_set_encrypt_key)를 이용하여 키를 설정하고 [memcpy](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/memcpy)를 이용하여 IV를 설정하자.

```cpp
if (AES_set_decrypt_key(KEY, sizeof(KEY) * 8, &DecKEY) < 0 ) {
    printf("AES_set_decrypt_key error \n");
    free(PaddedData);
    return -1;
}
memcpy(AESIV, IV, AES_BLOCK_SIZE);
```

[AES_cbc_encrypt](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/AES_cbc_encrypt)를 이용하여 복호화를 수행한다.

```cpp
unsigned char DecryptedData[PaddedDataLength] = {0};
AES_cbc_encrypt(EncryptedData, DecryptedData, PaddedDataLength, &DecKEY, AESIV, AES_DECRYPT);

for(int i = 0; i < sizeof(DecryptedData); i++) std::cout << std::setw(2) << std::setfill('0') << std::hex << (int)DecryptedData[i] << std::dec << ' ';
std::cout << std::endl;
```

[memcmp](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/memcmp)를 이용하여 복호화된 데이터와 원본데이터를 비교해보자.

```cpp
if (memcmp(Data, DecryptedData, DataLength) != 0) printf("Validation Failed.\n");
else printf("Validation Success\n");
free(PaddedData);
```

# **AES GCM**
## **Encrypt**
## **Decrypt**
# **Full Code**