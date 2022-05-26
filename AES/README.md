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

CBC모드와 달리 GCM(Galois/Counter Mode)는 검증을 위한 AAD(Additional Authentication Data)와 Authentication Tag가 필요하다.

```cpp
unsigned char AAD[AES_BLOCK_SIZE] = { 0x4d, 0x23, 0xc3, 0xce, 0xc3, 0x34, 0xb4, 0x9b, 0xdb, 0x37, 0x0c, 0x43, 0x7f, 0xec, 0x78, 0xde };
unsigned char TAG[AES_BLOCK_SIZE] = { '\0' };
```

[EVP_CIPHER_CTX_new](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_CIPHER_CTX_new)을 이용하여 CTX를 초기화시켜주자.

```cpp
EVP_CIPHER_CTX *CTX = EVP_CIPHER_CTX_new();
```

## **Encrypt**

[EVP_EncryptInit_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptInit_ex)으로 GCM을 설정하고

[EVP_CIPHER_CTX_ctrl](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_CIPHER_CTX_ctrl), [EVP_EncryptInit_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptInit_ex), [EVP_EncryptUpdate](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptUpdate)를 이용하여 암호화를 위한 준비를 하자.

```cpp
int len = 0;
if(1 != EVP_EncryptInit_ex(CTX, EVP_aes_256_gcm(), NULL, NULL, NULL)) printf("EVP_EncryptInit_ex\n");
if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_IVLEN, sizeof(IV), NULL)) printf("EVP_CIPHER_CTX_ctrl\n");
if(1 != EVP_EncryptInit_ex(CTX, NULL, NULL, KEY, IV)) printf("EVP_EncryptInit_ex\n");
if(1 != EVP_EncryptUpdate(CTX, NULL, &len, AAD, sizeof(AAD))) printf("EVP_EncryptUpdate\n");
```

[EVP_EncryptUpdate](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptUpdate)를 이용하여 AES 블록 사이즈(16 Byte)만큼 암호화를 하며

마지막 블록은 AES 블록 사이즈 만큼 떨어지지 않을 수도 있으니 남겨두자.

```cpp
unsigned char *GCMEncryptedData = (unsigned char *)malloc(DataLength + EVP_MAX_BLOCK_LENGTH + 1);
int GCMEncryptedDataLength = 0;
while(GCMEncryptedDataLength <= (DataLength - AES_BLOCK_SIZE))
{
    if(1 != EVP_EncryptUpdate(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len, Data + GCMEncryptedDataLength, AES_BLOCK_SIZE)) printf("EVP_EncryptUpdate in while\n");
    GCMEncryptedDataLength += len;
}
```

나머지 부분을 암호화하고 [EVP_EncryptFinal_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptFinal_ex)을 통해 정리하자.

```cpp
if(1 != EVP_EncryptUpdate(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len, Data + GCMEncryptedDataLength, DataLength - GCMEncryptedDataLength)) printf("EVP_EncryptUpdate\n");
GCMEncryptedDataLength += len;
if(1 != EVP_EncryptFinal_ex(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len)) printf("EVP_EncryptFinal_ex\n");
GCMEncryptedDataLength += len;
```

복호화에서 검증용 TAG를 생성하고 [EVP_CIPHER_CTX_free](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_CIPHER_CTX_free)를 이용하여 컨텍스트를 해제합니다.

```cpp
if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_AEAD_GET_TAG, AES_BLOCK_SIZE, TAG)) printf("EVP_CIPHER_CTX_ctrl\n");
EVP_CIPHER_CTX_free(CTX);
```

## **Decrypt**

[EVP_DecryptInit_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_DecryptInit_ex)으로 GCM을 설정하고

[EVP_CIPHER_CTX_ctrl](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_CIPHER_CTX_ctrl), [EVP_DecryptInit_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_DecryptInit_ex), [EVP_DecryptUpdate](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_DecryptUpdate)를 이용하여 복호화를 위한 준비를 하자.

```cpp
int len = 0;
if(1 != EVP_DecryptInit_ex(CTX, EVP_aes_256_gcm(), NULL, NULL, NULL)) printf("EVP_DecryptInit_ex\n");
if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_IVLEN, sizeof(IV), NULL)) printf("EVP_CIPHER_CTX_ctrl\n");
if(1 != EVP_DecryptInit_ex(CTX, NULL, NULL, KEY, IV)) printf("EVP_DecryptInit_ex\n");
if(1 != EVP_DecryptUpdate(CTX, NULL, &len, AAD, sizeof(AAD))) printf("EVP_DecryptUpdate\n");
```

[EVP_DecryptUpdate](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_DecryptUpdate)를 이용하여 AES 블록 사이즈(16 Byte)만큼 복호화를 하며

마지막 블록은 AES 블록 사이즈 만큼 떨어지지 않을 수도 있으니 남겨두자.

```cpp
unsigned char *GCMDecryptedData = (unsigned char *)malloc(GCMEncryptedDataLength + EVP_MAX_BLOCK_LENGTH + 1);
int GCMDecryptedDataLength = 0;
while(GCMDecryptedDataLength <= (GCMEncryptedDataLength - AES_BLOCK_SIZE)) {
    if(1 != EVP_DecryptUpdate(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len, GCMEncryptedData + GCMDecryptedDataLength, AES_BLOCK_SIZE)) printf("EVP_DecryptUpdate in while\n");
    GCMDecryptedDataLength += len;
}
```

나머지 부분을 복호화하고 TAG 검증 후 [EVP_DecryptFinal_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_EncryptFinal_ex)을 통해 정리하자.

```cpp
if(1 != EVP_DecryptUpdate(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len, GCMEncryptedData + GCMDecryptedDataLength, GCMEncryptedDataLength - GCMDecryptedDataLength)) printf("EVP_DecryptUpdate2\n");
GCMDecryptedDataLength += len;
if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_TAG, AES_BLOCK_SIZE, TAG)) printf("EVP_CIPHER_CTX_ctrl\n");
if(1 != EVP_DecryptFinal_ex(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len)) printf("EVP_EncryptFinal_ex\n");
GCMEncryptedDataLength += len;
```

[memcmp](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/memcmp)를 이용하여 복호화된 데이터와 원본데이터를 비교해보자.

```cpp
if (memcmp(Data, GCMDecryptedData, GCMEncryptedDataLength) != 0) printf("Validation Failed.\n");
else printf("Validation Success\n");
EVP_CIPHER_CTX_free(CTX);
free(GCMEncryptedData);
free(GCMDecryptedData);
```

# **Full Code**

```cpp
#include <iostream>
#include <iomanip>
#include <vector>
#include <openssl/aes.h>
#include <string.h>
#include <openssl/evp.h>

int main(void) {
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

    if (DataLength % AES_BLOCK_SIZE != 0) PaddingLength = (AES_BLOCK_SIZE - (DataLength % AES_BLOCK_SIZE));
    int PaddedDataLength    = DataLength + PaddingLength;

    unsigned char *PaddedData = (unsigned char *)malloc(sizeof(unsigned char *) * PaddedDataLength);
    memset(PaddedData, 0x00, PaddedDataLength);
    memcpy(PaddedData, Data, DataLength);


    if (AES_set_encrypt_key(KEY, sizeof(KEY) * 8, &EncKEY) < 0 ) {
        printf("AES_set_encrypt_key error \n");
        free(PaddedData);
        return -1;
    }
    memcpy(AESIV, IV, AES_BLOCK_SIZE);

    unsigned char EncryptedData[PaddedDataLength] = {0};
    AES_cbc_encrypt(PaddedData, EncryptedData, PaddedDataLength, &EncKEY, AESIV, AES_ENCRYPT);


    if (AES_set_decrypt_key(KEY, sizeof(KEY) * 8, &DecKEY) < 0 ) {
        printf("AES_set_decrypt_key error \n");
        free(PaddedData);
        return -1;
    }
    memcpy(AESIV, IV, AES_BLOCK_SIZE);

    unsigned char DecryptedData[PaddedDataLength] = {0};
    AES_cbc_encrypt(EncryptedData, DecryptedData, PaddedDataLength, &DecKEY, AESIV, AES_DECRYPT);

    if (memcmp(Data, DecryptedData, DataLength) != 0) printf("Validation Failed.\n");
    else printf("Validation Success\n");
    free(PaddedData);


    unsigned char AAD[AES_BLOCK_SIZE] = { 0x4d, 0x23, 0xc3, 0xce, 0xc3, 0x34, 0xb4, 0x9b, 0xdb, 0x37, 0x0c, 0x43, 0x7f, 0xec, 0x78, 0xde };
    unsigned char TAG[AES_BLOCK_SIZE] = { '\0' };

    EVP_CIPHER_CTX *CTX = EVP_CIPHER_CTX_new();
    
    int len = 0;
    if(1 != EVP_EncryptInit_ex(CTX, EVP_aes_256_gcm(), NULL, NULL, NULL)) printf("EVP_EncryptInit_ex\n");
	if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_IVLEN, sizeof(IV), NULL)) printf("EVP_CIPHER_CTX_ctrl\n");
	if(1 != EVP_EncryptInit_ex(CTX, NULL, NULL, KEY, IV)) printf("EVP_EncryptInit_ex\n");
	if(1 != EVP_EncryptUpdate(CTX, NULL, &len, AAD, sizeof(AAD))) printf("EVP_EncryptUpdate\n");

    unsigned char *GCMEncryptedData = (unsigned char *)malloc(DataLength + EVP_MAX_BLOCK_LENGTH + 1);
    int GCMEncryptedDataLength = 0;
    while(GCMEncryptedDataLength <= (DataLength - AES_BLOCK_SIZE))
    {
        if(1 != EVP_EncryptUpdate(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len, Data + GCMEncryptedDataLength, AES_BLOCK_SIZE)) printf("EVP_EncryptUpdate in while\n");
        GCMEncryptedDataLength += len;
    }
    if(1 != EVP_EncryptUpdate(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len, Data + GCMEncryptedDataLength, DataLength - GCMEncryptedDataLength)) printf("EVP_EncryptUpdate\n");
	GCMEncryptedDataLength += len;
	if(1 != EVP_EncryptFinal_ex(CTX, GCMEncryptedData + GCMEncryptedDataLength, &len)) printf("EVP_EncryptFinal_ex\n");
	GCMEncryptedDataLength += len;
    if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_AEAD_GET_TAG, AES_BLOCK_SIZE, TAG)) printf("EVP_CIPHER_CTX_ctrl\n");
    EVP_CIPHER_CTX_free(CTX);


    CTX = EVP_CIPHER_CTX_new();
    len = 0;
    if(1 != EVP_DecryptInit_ex(CTX, EVP_aes_256_gcm(), NULL, NULL, NULL)) printf("EVP_DecryptInit_ex\n");
	if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_IVLEN, sizeof(IV), NULL)) printf("EVP_CIPHER_CTX_ctrl\n");
	if(1 != EVP_DecryptInit_ex(CTX, NULL, NULL, KEY, IV)) printf("EVP_DecryptInit_ex\n");
	if(1 != EVP_DecryptUpdate(CTX, NULL, &len, AAD, sizeof(AAD))) printf("EVP_DecryptUpdate\n");

    unsigned char *GCMDecryptedData = (unsigned char *)malloc(GCMEncryptedDataLength + EVP_MAX_BLOCK_LENGTH + 1);
    int GCMDecryptedDataLength = 0;
 	while(GCMDecryptedDataLength <= (GCMEncryptedDataLength - AES_BLOCK_SIZE)) {
		if(1 != EVP_DecryptUpdate(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len, GCMEncryptedData + GCMDecryptedDataLength, AES_BLOCK_SIZE)) printf("EVP_DecryptUpdate in while\n");
		GCMDecryptedDataLength += len;
 	}
	if(1 != EVP_DecryptUpdate(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len, GCMEncryptedData + GCMDecryptedDataLength, GCMEncryptedDataLength - GCMDecryptedDataLength)) printf("EVP_DecryptUpdate2\n");
	GCMDecryptedDataLength += len;
    if(1 != EVP_CIPHER_CTX_ctrl(CTX, EVP_CTRL_GCM_SET_TAG, AES_BLOCK_SIZE, TAG)) printf("EVP_CIPHER_CTX_ctrl\n");
    if(1 != EVP_DecryptFinal_ex(CTX, GCMDecryptedData + GCMDecryptedDataLength, &len)) printf("EVP_EncryptFinal_ex\n");
    GCMEncryptedDataLength += len;

    if (memcmp(Data, GCMDecryptedData, GCMEncryptedDataLength) != 0) printf("Validation Failed.\n");
    else printf("Validation Success\n");
    EVP_CIPHER_CTX_free(CTX);

    free(GCMEncryptedData);
    free(GCMDecryptedData);

    return 0;
}
```