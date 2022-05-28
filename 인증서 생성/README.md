# 인증서 생성

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libssl-dev    | 1.1.1f-1ubuntu2.13        |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [개인키 생성](#개인키-생성)**

 - [개인키 암호화](#개인키-암호화)**

**3. [인증서 생성](#인증서-생성)**

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

# **개인키 생성**

[BN_new](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/BN_new)로 BIGNUM 구조체 초기화하자.

```cpp
BIGNUM *bignum = NULL;
ERR_clear_error();
if ((bignum  = BN_new()) == NULL) {
    printf("BN_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    return EXIT_FAILURE;
}
```

[BN_set_word](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/BN_set_word)로 BIGNUM에 데이터를 쓰자.

```cpp
if(BN_set_word(bignum, RSA_F4) != 1) {
    printf("BN_set_word error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    BN_clear_free(bignum);
    return EXIT_FAILURE;
}
```

[RSA_new](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/RSA_new)로 RSA를 초기화하자.

```cpp
RSA *rsa = NULL;
if ((rsa = RSA_new()) == NULL) {
    printf("RSA_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    BN_clear_free(bignum);
    return EXIT_FAILURE;
}
```

[RSA_generate_key_ex](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/RSA_generate_key_ex)로 RSA 키 쌍을 생성하자.

```cpp
if (RSA_generate_key_ex(rsa, 2048, bignum, NULL) != 1) {
    printf("RSA_generate_key_ex error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    BN_clear_free(bignum);
    return EXIT_FAILURE;
}
BN_clear_free(bignum);
```

[fopen](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fopen)으로 파일을 생성하자.

```cpp
FILE *PrivateFilePointer = fopen("./private.key", "wb");
if (PrivateFilePointer == NULL) {
    perror("private.key open fail");
    RSA_free(rsa);
    rsa     = NULL;
    return EXIT_FAILURE;
}
```

[PEM_write_RSAPrivateKey](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/PEM_write_RSAPrivateKey)으로 파일에 개인키를 쓰자.

```cpp
if(PEM_write_RSAPrivateKey(PrivateFilePointer, rsa, NULL, NULL, 0, NULL, NULL) != 1)
{
    printf("PEM_write_RSAPrivateKey error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    fclose(PrivateFilePointer);
    PrivateFilePointer = NULL;
    RSA_free(rsa);
    rsa = NULL;
    return EXIT_FAILURE;
}
fclose(PrivateFilePointer);
PrivateFilePointer = NULL;
RSA_free(rsa);
rsa = NULL;
```

## **개인키 암호화**

키 값을 별도로 설정하여 개인키를 암호화하여 저장해보자.

```cpp
unsigned char KEY[32] = { 0x01, 0x02, 0x02, 0x03, 0x03, 0x03, 0x04, 0x04, 0x04, 0x04, 0x05, 0x05, 0x05, 0x05, 0x05, 0x06, 0x06, 0x06, 0x06, 0x06, 0x06, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x08, 0x08, 0x08, 0x08 };
if(PEM_write_RSAPrivateKey(PrivateFilePointer, rsa, EVP_aria_256_cbc(), KEY, sizeof(KEY), NULL, NULL) != 1)
{
    printf("PEM_write_RSAPrivateKey error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    fclose(PrivateFilePointer);
    PrivateFilePointer = NULL;
    RSA_free(rsa);
    rsa = NULL;
    return EXIT_FAILURE;
}
```

# **인증서 생성**

[EVP_PKEY_new](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_PKEY_new)로 EVP_PKEY를 초기화하자.

```cpp
EVP_PKEY *pkey = NULL;
if ((pkey = EVP_PKEY_new()) == NULL) {
    printf("EVP_PKEY_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    RSA_free(rsa);
    rsa = NULL;
    return EXIT_FAILURE;
}
```

[EVP_PKEY_set1_RSA](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/EVP_PKEY_set1_RSA)로 EVP_PKEY의 키를 RSA로 할당하자.

```cpp
if(EVP_PKEY_set1_RSA(pkey, rsa) != 1) {
    printf("EVP_PKEY_set1_RSA error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    RSA_free(rsa);
    rsa     = NULL;
    EVP_PKEY_free(pkey);
    pkey    = NULL;
    return EXIT_FAILURE;
}
RSA_free(rsa);
rsa = NULL;
```

[X509_new](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/X509_new)로 X509 구조를 초기화하자.

```cpp
X509 *x509 = NULL;
if ((x509 = X509_new()) == NULL) {
    printf("X509_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    EVP_PKEY_free(pkey);
    pkey  = NULL;
    return EXIT_FAILURE;
}
```

버전, 일련 번호(Serial Number), 유효 기간(시작)(Not Before), 유효 기간(끝)(Not After)을 설정하고 공개키를 작성하자.

```cpp
X509_set_version(x509, 2);
ASN1_INTEGER_set(X509_get_serialNumber(x509), (long)time(NULL));
X509_gmtime_adj(X509_get_notBefore(x509), 0);
X509_gmtime_adj(X509_get_notAfter(x509), (long)(60 * 60 * 24 * 365));
X509_set_pubkey(x509, pkey);
```

주체 및 발급 대상(CN) 등을 설정하자.

```cpp
X509_NAME_add_entry_by_txt(name, "C", 	MBSTRING_ASC, (unsigned char *)"KR",	-1, -1, 0);
X509_NAME_add_entry_by_txt(name, "O", 	MBSTRING_ASC, (unsigned char *)"World",	-1, -1, 0);
X509_NAME_add_entry_by_txt(name, "OU",	MBSTRING_ASC, (unsigned char *)"X509",	-1, -1, 0);
X509_NAME_add_entry_by_txt(name, "CN",	MBSTRING_ASC, (unsigned char *)"2jinu", -1, -1, 0);
X509_set_issuer_name(x509, name);
```

자체 서명(자신의 개인키)을 통해 발급하자.

```cpp
if(X509_sign(x509, pkey, EVP_sha256()) == 0) {
    printf("X509_sign error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    EVP_PKEY_free(pkey);
    pkey  = NULL;
    X509_free(x509);
    x509 = NULL;
    return EXIT_FAILURE;
}
EVP_PKEY_free(pkey);
pkey  = NULL;
```

[fopen](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/fopen)으로 파일을 생성하자.

```cpp
FILE *CertificateFilePointer = fopen("./certificate.crt", "wb");
if (CertificateFilePointer == NULL) {
    perror("certificate.crt open fail");
    X509_free(x509);
    x509 = NULL;
    return EXIT_FAILURE;
}
```

[PEM_write_X509](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/PEM_write_X509)로 파일에 인증서를 쓰자.

```cpp
if(PEM_write_X509(CertificateFilePointer, x509) != 1) {
    printf("PEM_write_X509 error : %s\n", ERR_error_string(ERR_get_error(), NULL));
    X509_free(x509);
    x509 = NULL;
    fclose(CertificateFilePointer);
    CertificateFilePointer = NULL;
    return EXIT_FAILURE;
}
CRYPTO_cleanup_all_ex_data();
ERR_free_strings();
EVP_cleanup();
```

# **Full Code**

```cpp
#include <openssl/rsa.h>
#include <openssl/err.h>
#include <openssl/evp.h>
#include <openssl/bio.h>
#include <openssl/pem.h>
#include <openssl/x509v3.h>
#include <stdio.h>

int main(void) {
    BIGNUM *bignum = NULL;
    ERR_clear_error();
    if ((bignum  = BN_new()) == NULL) {
        printf("BN_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        return EXIT_FAILURE;
    }
    if(BN_set_word(bignum, RSA_F4) != 1) {
        printf("BN_set_word error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        BN_clear_free(bignum);
        return EXIT_FAILURE;
    }

    RSA *rsa = NULL;
    if ((rsa = RSA_new()) == NULL) {
        printf("RSA_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        BN_clear_free(bignum);
        return EXIT_FAILURE;
    }
    if (RSA_generate_key_ex(rsa, 2048, bignum, NULL) != 1) {
        printf("RSA_generate_key_ex error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        BN_clear_free(bignum);
        return EXIT_FAILURE;
    }
    BN_clear_free(bignum);

    FILE *PrivateFilePointer = fopen("./private.key", "wb");
    if (PrivateFilePointer == NULL) {
        perror("private.key open fail");
        RSA_free(rsa);
        rsa     = NULL;
        return EXIT_FAILURE;
    }
    
    unsigned char KEY[32] = { 0x01, 0x02, 0x02, 0x03, 0x03, 0x03, 0x04, 0x04, 0x04, 0x04, 0x05, 0x05, 0x05, 0x05, 0x05, 0x06, 0x06, 0x06, 0x06, 0x06, 0x06, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x07, 0x08, 0x08, 0x08, 0x08 };
    if(PEM_write_RSAPrivateKey(PrivateFilePointer, rsa, EVP_aria_256_cbc(), KEY, sizeof(KEY), NULL, NULL) != 1)
    {
        printf("PEM_write_RSAPrivateKey error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        fclose(PrivateFilePointer);
        PrivateFilePointer = NULL;
        RSA_free(rsa);
        rsa = NULL;
        return EXIT_FAILURE;
    }
    fclose(PrivateFilePointer);
    PrivateFilePointer = NULL;


    EVP_PKEY *pkey = NULL;
    if ((pkey = EVP_PKEY_new()) == NULL) {
        printf("EVP_PKEY_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        RSA_free(rsa);
        rsa = NULL;
        return EXIT_FAILURE;
    }
    if(EVP_PKEY_set1_RSA(pkey, rsa) != 1) {
        printf("EVP_PKEY_set1_RSA error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        RSA_free(rsa);
        rsa     = NULL;
        EVP_PKEY_free(pkey);
        pkey    = NULL;
        return EXIT_FAILURE;
    }
    RSA_free(rsa);
    rsa = NULL;

    X509 *x509 = NULL;
    if ((x509 = X509_new()) == NULL) {
        printf("X509_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        EVP_PKEY_free(pkey);
        pkey  = NULL;
        return EXIT_FAILURE;
    }
    X509_set_version(x509, 2);
    ASN1_INTEGER_set(X509_get_serialNumber(x509), (long)time(NULL));
    X509_gmtime_adj(X509_get_notBefore(x509), 0);
    X509_gmtime_adj(X509_get_notAfter(x509), (long)(60 * 60 * 24 * 365));
    X509_set_pubkey(x509, pkey);

    X509_NAME *name = NULL;
    if ((name = X509_NAME_new()) == NULL) {
        printf("X509_NAME_new error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        EVP_PKEY_free(pkey);
        pkey  = NULL;
        X509_free(x509);
        x509 = NULL;
        return EXIT_FAILURE;
    }
    X509_NAME_add_entry_by_txt(name, "C", 	MBSTRING_ASC, (unsigned char *)"KR",	-1, -1, 0);
    X509_NAME_add_entry_by_txt(name, "O", 	MBSTRING_ASC, (unsigned char *)"World",	-1, -1, 0);
	X509_NAME_add_entry_by_txt(name, "OU",	MBSTRING_ASC, (unsigned char *)"X509",	-1, -1, 0);
	X509_NAME_add_entry_by_txt(name, "CN",	MBSTRING_ASC, (unsigned char *)"2jinu", -1, -1, 0);
    X509_set_issuer_name(x509, name);

    if(X509_sign(x509, pkey, EVP_sha256()) == 0) {
        printf("X509_sign error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        EVP_PKEY_free(pkey);
        pkey  = NULL;
        X509_free(x509);
        x509 = NULL;
        return EXIT_FAILURE;
    }
    EVP_PKEY_free(pkey);
    pkey  = NULL;

    FILE *CertificateFilePointer = fopen("./certificate.crt", "wb");
    if (CertificateFilePointer == NULL) {
        perror("certificate.crt open fail");
        X509_free(x509);
        x509 = NULL;
        return EXIT_FAILURE;
    }
    if(PEM_write_X509(CertificateFilePointer, x509) != 1) {
        printf("PEM_write_X509 error : %s\n", ERR_error_string(ERR_get_error(), NULL));
        X509_free(x509);
        x509 = NULL;
        fclose(CertificateFilePointer);
        CertificateFilePointer = NULL;
        return EXIT_FAILURE;
    }
    CRYPTO_cleanup_all_ex_data();
    ERR_free_strings();
    EVP_cleanup();

    return 0;
}
```