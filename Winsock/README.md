# Winsock

# **INDEX**

**1. [TCP Server](#TCP-Server)**

 - [Server Winsock 초기화](#Server-Winsock-초기화)

 - [Server 소켓 생성](#Server-소켓-생성)

 - [소켓 바인딩](#소켓-바인딩)

 - [소켓 리스닝](#소켓-리스닝)

 - [연결 수락](#연결-수락)

 - [Server 데이터 송수신](#Server-데이터-송수신)

 - [Server 연결 종료](#Server-연결-종료)

**2. [TCP Client](#TCP-Client)**

 - [Client Winsock 초기화](#Client-Winsock-초기화)

 - [Client 소켓 생성](#Client-소켓-생성)

 - [연결 요청](#연결-요청)

 - [Client 데이터 송수신](#Client-데이터-송수신)

 - [Client 연결 종료](#Client-연결-종료)


# **TCP Server**

## **Server Winsock 초기화**

[WSADATA](https://docs.microsoft.com/ko-KR/windows/win32/api/winsock/ns-winsock-wsadata) 개체를 생성 후 Winsock 2.2버전을 요청하여 초기화를 진행합니다.

```c++
#pragma comment(lib, "Ws2_32.lib")
#define _WINSOCK_DEPRECATED_NO_WARNINGS

#include <WinSock2.h>
#include <ws2tcpip.h>
#include <stdio.h>

int main(void)
{
	WSADATA wsaData;
	int iResult;

	iResult = WSAStartup(MAKEWORD(2, 2), &wsaData);
	if (iResult != 0) {
		printf("WSAStartup failed with error: %d\n", iResult);
		return 1;
	}

	WSACleanup();

	return 0;
}
```

## **Server 소켓 생성**

소켓을 사용하기 위해 서버가 사용할 소켓을 생성합니다.

바인딩할 포트는 33333으로 지정해봅니다.

```c++
    ...
    struct addrinfo ServerAddrInfo, *result = NULL;

	ZeroMemory(&ServerAddrInfo, sizeof(ServerAddrInfo));
	ServerAddrInfo.ai_family	= AF_INET;
	ServerAddrInfo.ai_socktype	= SOCK_STREAM;
	ServerAddrInfo.ai_protocol	= IPPROTO_TCP;
	ServerAddrInfo.ai_flags		= AI_PASSIVE;

	iResult = getaddrinfo(NULL, "33333", &ServerAddrInfo, &result);
	if (iResult != 0) {
		printf("getaddrinfo failed: %d\n", iResult);
		WSACleanup();
		return 1;
	}

    SOCKET ListenSocket = INVALID_SOCKET;
	ListenSocket = socket(result->ai_family, result->ai_socktype, result->ai_protocol);
	if (ListenSocket == INVALID_SOCKET) {
		printf("Error at socket(): %ld\n", WSAGetLastError());
		freeaddrinfo(result);
		WSACleanup();
		return 1;
	}

    closesocket(ListenSocket);
	WSACleanup();
    return 0;
}
```

## **소켓 바인딩**

클라이언트 연결을 허용하려면 네트워크 주소에 바인딩되어야 하고, 해당 절차를 수행합니다.

```c++
    ...
    iResult = bind(ListenSocket, result->ai_addr, (int)result->ai_addrlen);
	if (iResult == SOCKET_ERROR) {
		printf("bind failed with error: %d\n", WSAGetLastError());
		freeaddrinfo(result);
		closesocket(ListenSocket);
		WSACleanup();
		return 1;
	}
	freeaddrinfo(result);

    closesocket(ListenSocket);
	WSACleanup();
    return 0;
}
```

## **소켓 리스닝**

소켓이 IP와 포트에 바인딩된 후 클라이언트 연결 요청에 대해서 수신 대기합니다.

```c++
    ...
    if (listen(ListenSocket, SOMAXCONN) == SOCKET_ERROR) {
		printf("Listen failed with error: %ld\n", WSAGetLastError());
		closesocket(ListenSocket);
		WSACleanup();
		return 1;
	}

    closesocket(ListenSocket);
	WSACleanup();
    return 0;
}
```

다음의 코드를 진행하면 LISTENING 상태가 됩니다.

```
C:\Users\admin>netstat -ano | findstr 33333
  TCP    0.0.0.0:33333          0.0.0.0:0              LISTENING       36772
```

## **연결 수락**

클라이언트의 연결 요청에 대한 처리를 수행합니다.

```c++
    ...
    SOCKET ClientSocket = INVALID_SOCKET;
	ClientSocket = accept(ListenSocket, NULL, NULL);
	if (ClientSocket == INVALID_SOCKET) {
		printf("accept failed: %d\n", WSAGetLastError());
		closesocket(ListenSocket);
		WSACleanup();
		return 1;
	}
	closesocket(ListenSocket);

	closesocket(ClientSocket);
	WSACleanup();
	return 0;
}
```

## **Server 데이터 송수신**

연결 요청에 대한 수락이후 데이터를 송수신합니다.

```c++
    ...
    int recvbuflen = 512, iSendResult;
	const char* sendbuf = "World";
	char recvbuf[512];

	iResult = recv(ClientSocket, recvbuf, recvbuflen, 0);
	if (iResult > 0) {
		printf("Received: %s (%d)\n", recvbuf, iResult);

		iSendResult = send(ClientSocket, sendbuf, (int)strlen(sendbuf), 0);
		if (iSendResult == SOCKET_ERROR) {
			printf("send failed: %d\n", WSAGetLastError());
			closesocket(ClientSocket);
			WSACleanup();
			return 1;
		}
		printf("Bytes sent: %d\n", iSendResult);
	}
	else if (iResult == 0) printf("Connection closing...\n");
	else {
		printf("recv failed: %d\n", WSAGetLastError());
		closesocket(ClientSocket);
		WSACleanup();
		return 1;
	}
```

## **Server 연결 종료**

클라이언트와의 연결을 종료합니다.

```c++
    ...
    iResult = shutdown(ClientSocket, SD_SEND);
	if (iResult == SOCKET_ERROR) {
		printf("shutdown failed: %d\n", WSAGetLastError());
		closesocket(ClientSocket);
		WSACleanup();
		return 1;
	}

	closesocket(ClientSocket);
	WSACleanup();
	return 0;
}
```

# **TCP Client**

## **Client Winsock 초기화**

[WSADATA](https://docs.microsoft.com/ko-KR/windows/win32/api/winsock/ns-winsock-wsadata) 개체를 생성 후 Winsock 2.2버전을 요청하여 초기화를 진행합니다.

```c++
#pragma comment(lib, "Ws2_32.lib")
#define _WINSOCK_DEPRECATED_NO_WARNINGS

#include <WinSock2.h>
#include <ws2tcpip.h>
#include <stdio.h>

int main(void)
{
	WSADATA wsaData;
	int iResult;

	iResult = WSAStartup(MAKEWORD(2, 2), &wsaData);
	if (iResult != 0) {
		printf("WSAStartup failed with error: %d\n", iResult);
		return 1;
	}

	WSACleanup();

	return 0;
}
```

## **Client 소켓 생성**

소켓을 사용하기 위해 클라이언트가 사용할 소켓을 생성합니다.

접속할 주소는 127.0.01:33333으로 지정해봅니다.

```c++
    ...
    struct addrinfo ClientAddrInfo, * result = NULL;
	ZeroMemory(&ClientAddrInfo, sizeof(ClientAddrInfo));
	ClientAddrInfo.ai_family	= AF_UNSPEC;
	ClientAddrInfo.ai_socktype	= SOCK_STREAM;
	ClientAddrInfo.ai_protocol	= IPPROTO_TCP;

	iResult = getaddrinfo("127.0.0.1", "33333", &ClientAddrInfo, &result);
	if (iResult != 0) {
		printf("getaddrinfo failed: %d\n", iResult);
		WSACleanup();
		return 1;
	}

    SOCKET ClientSocket = INVALID_SOCKET;
	ClientSocket = socket(result->ai_family, result->ai_socktype, result->ai_protocol);
	if (ClientSocket == INVALID_SOCKET) {
		printf("socket failed with error: %ld\n", WSAGetLastError());
		WSACleanup();
		return 1;
	}

	closesocket(ClientSocket);
	WSACleanup();
    return 0;
}
```

## **연결 요청**

클라이언트가 통신을 하기위한 연결 요청을 수행합니다.

```c++
    ...
    iResult = connect(ClientSocket, result->ai_addr, (int)result->ai_addrlen);
	if (iResult == SOCKET_ERROR) {
		closesocket(ClientSocket);
		ClientSocket = INVALID_SOCKET;
	}
	freeaddrinfo(result);
	if (ClientSocket == INVALID_SOCKET) {
		printf("Unable to connect to server!\n");
		WSACleanup();
		return 1;
	}

	closesocket(ClientSocket);
	WSACleanup();
	return 0;
}
```

## **Client 데이터 송수신**

연결 요청에 대한 수락이후 데이터를 송수신합니다.

```c++
    ...
    int recvbuflen = 512;
	const char* sendbuf = "Hello";
	char recvbuf[512];
	iResult = send(ClientSocket, sendbuf, (int)strlen(sendbuf), 0);
	if (iResult == SOCKET_ERROR) {
		printf("send failed: %d\n", WSAGetLastError());
		closesocket(ClientSocket);
		WSACleanup();
		return 1;
	}
	printf("Bytes Sent: %ld\n", iResult);

	iResult = recv(ClientSocket, recvbuf, recvbuflen, 0);
	if (iResult > 0) {
		recvbuf[iResult] = '\0';
		printf("Received: %s (%d)\n", recvbuf, iResult);
	}
	else if (iResult == 0) printf("Connection closed\n");
	else printf("recv failed: %d\n", WSAGetLastError());

	closesocket(ClientSocket);
	WSACleanup();
	return 0;
}
```


## **Client 연결 종료**

서버와의 연결을 종료합니다.

```c++
    ...
    iResult = shutdown(ClientSocket, SD_SEND);
	if (iResult == SOCKET_ERROR) {
		printf("shutdown failed: %d\n", WSAGetLastError());
		closesocket(ClientSocket);
		WSACleanup();
		return 1;
	}

	closesocket(ClientSocket);
	WSACleanup();
	return 0;
}
```