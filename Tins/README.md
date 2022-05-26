# **Tins**

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libtins-dev   | 4.0-1build1               |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [네트워크 정보 확인](#네트워크-정보-확인)**

**3. [패킷 캡처](#패킷-캡처)**

 - [캡처 필터링 적용하기](#캡처-필터링-적용하기)

 - [캡처 콜백 함수](#캡처-콜백-함수)

**4. [패킷 분석](#패킷-분석)**

 - [ETHERNET 분석](#ETHERNET-분석)

 - [IP 분석](#IP-분석)

 - [ICMP 분석](#ICMP-분석)

 - [4TCP](#TCP-분석)

 - [UDP](#UDP-분석)

**5. [패킷 생성](#패킷-생성)**

 - [ETHERNET 생성](#ETHERNET-생성)

 - [IP 생성](#IP-생성)

 - [ICMP 생성](#ICMP-생성)

 - [TCP 생성](#TCP-생성)

 - [UDP 생성](#UDP-생성)

**6. [Full Code](#Full-Code)**


# **패키지 설치**

libtins-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libtins-dev
```

컴파일시 libtins를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -ltins -o outputfile
```


# **네트워크 정보 확인**

네트워크 인터페이스 명, IP 주소, MAC 주소를 알아보자.

```cpp
std::vector<Tins::NetworkInterface> NetworkInterfaceVector(Tins::NetworkInterface::all());
for(int i = 0; i < NetworkInterfaceVector.size(); ++i) std::cout << i << ' ' << NetworkInterfaceVector[i].name() << '\t' << NetworkInterfaceVector[i].ipv4_address() << '\t' << NetworkInterfaceVector[i].hw_address() << std::endl;
```

# **패킷 캡처**

나에게 오는 패킷만(Promiscuous false) 즉시(immediate true) 캡처해보자.

```cpp
Tins::SnifferConfiguration SnifferConfig;
SnifferConfig.set_immediate_mode(true);
SnifferConfig.set_promisc_mode(false);
```

## **캡처 필터링 적용하기**

포트가 80번인것만 필터링해보자.

```cpp
SnifferConfig.set_filter("port 80 and src 192.168.0.30");
```

## **캡처 콜백 함수**

패킷이 인터페이스로 들어왔을 때, 호출할 함수를 지정하자.

```cpp
Tins::Sniffer Sniffer("ens33", SnifferConfig);
Sniffer.sniff_loop(CallbackFunction);

...
bool CallbackFunction(Tins::PDU &Packet) {
    return true;
}
```

# **4. 패킷 분석**

## **ETHERNET 분석**

패킷의 출발지, 목적지의 MAC 주소를 확인하자.

```cpp
const Tins::EthernetII EthernetPacket  = Packet.rfind_pdu<Tins::EthernetII>();
std::cout << "\nEthernet Header (" << EthernetPacket.header_size() << ")" << std::endl;
std::cout << "   |-Source Address\t\t: " << EthernetPacket.src_addr() << std::endl;
std::cout << "   |-Destination Address\t: " << EthernetPacket.dst_addr() << std::endl;
```

## **IP 분석**

패킷의 출발지, 목적지의 IP 주소를 확인하자.

```cpp
const Tins::IP IP = Packet.rfind_pdu<Tins::IP>();
std::cout << "IP Header (" << IP.header_size() << ")" << std::endl;
std::cout << "   |-Source IP\t\t\t: " << IP.src_addr() << std::endl;
std::cout << "   |-Destination IP\t\t: " << IP.dst_addr() << std::endl;
```

IP계층이면 프로토콜을 확인하여 추가 데이터를 확인할 수 있다.

```cpp
switch ((unsigned)IP.protocol())
{
case IPPROTO_ICMP:
{
    break;
}
case IPPROTO_TCP:
{
    break;
}
case IPPROTO_UDP:
{
    break;
}
default:
    break;
}
```

## **ICMP 분석**

ICMP의 데이터를 확인해보자.

```cpp
const Tins::ICMP ICMP = Packet.rfind_pdu<Tins::ICMP>();
std::cout << "ICMP Header (" << ICMP.header_size() << ")" << std::endl;
std::cout << "   |-Type\t\t\t: " << (unsigned int)ICMP.type() << std::endl;

const Tins::RawPDU &RawPDU = ICMP.rfind_pdu<Tins::RawPDU>();
const Tins::RawPDU::payload_type &Data = RawPDU.payload();
std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
PrintData(&RawPDU.payload().front(), RawPDU.payload_size());
```

## **TCP 분석**

TCP의 데이터를 확인해보자.

```cpp
const Tins::TCP TCP = Packet.rfind_pdu<Tins::TCP>();
std::cout << "TCP Header (" << TCP.header_size() << ")" << std::endl;
std::cout << "   |-Source Port\t\t: " << TCP.sport() << std::endl;
std::cout << "   |-Destination Port\t\t: " << TCP.dport() << std::endl;

const Tins::RawPDU &RawPDU = TCP.rfind_pdu<Tins::RawPDU>();
const Tins::RawPDU::payload_type &Data = RawPDU.payload();
std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
PrintData(&RawPDU.payload().front(), RawPDU.payload_size());
```

## **UDP 분석**

UDP의 데이터를 확인해보자.

```cpp
const Tins::UDP UDP = Packet.rfind_pdu<Tins::UDP>();
std::cout << "UDP Header (" << UDP.header_size() << ")" << std::endl;
std::cout << "   |-Source Port\t\t: " << UDP.sport() << std::endl;
std::cout << "   |-Destination Port\t\t: " << UDP.dport() << std::endl;

const Tins::RawPDU &RawPDU = UDP.rfind_pdu<Tins::RawPDU>();
const Tins::RawPDU::payload_type &Data = RawPDU.payload();
std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
PrintData(&RawPDU.payload().front(), RawPDU.payload_size());
```

# **패킷 생성**

패킷을 생성해서 보내기 위하여 PacketSender를 선언하고 send를 호출해야한다.

```cpp
Tins::PacketSender PacketSender;
PacketSender.send(CraftedPacket, "ens33");
```

## **ETHERNET 생성**

PacketSender에 실을 Ethernet패킷을 생성해보자.

나한테 패킷을 전송한 곳으로 전송하되 출발지 MAC 주소를 변조해보자.

```cpp
Tins::EthernetII CraftedPacket(EthernetPacket.src_addr(), "00:00:00:00:00:00");
```

## **IP 생성**

PacketSender에 실을 IP패킷을 생성해보자.

나한테 패킷을 전송한 곳으로 전송하되 출발지 IP 주소를 변조해보자.

```cpp
CraftedPacket /= Tins::IP(IP.src_addr(), "8.8.8.8");
```

## **ICMP 생성**

PacketSender에 실을 ICMP패킷을 생성해보자.

ICMP의 타입을 Echo Reply로 변경해서 보내자.

```cpp
CraftedPacket /= ICMP;
CraftedPacket.rfind_pdu<Tins::ICMP>().type(Tins::ICMP::Flags::ECHO_REPLY);
```

## **TCP 생성**

PacketSender에 실을 TCP패킷을 생성해보자.

TCP의 페이로드에 데이터를 실어보자.

```cpp
CraftedPacket /= TCP;
CraftedPacket.rfind_pdu<Tins::TCP>().rfind_pdu<Tins::RawPDU>() = Tins::RawPDU("Hello World");
```

## **UDP 생성**

PacketSender에 실을 UDP패킷을 생성해보자.

UDP의 페이로드에 데이터를 실어보자.

```cpp
CraftedPacket /= UDP;
CraftedPacket.rfind_pdu<Tins::UDP>().rfind_pdu<Tins::RawPDU>() = Tins::RawPDU("Hello World");
```

# **Full Code**

```cpp
#include <tins/tins.h>
#include <vector>
#include <iostream>

bool CallbackFunction(Tins::PDU &Packet);
void PrintData(const u_char* data, int Size);

int main(void) {
    std::vector<Tins::NetworkInterface> NetworkInterfaceVector(Tins::NetworkInterface::all());
    for(int i = 0; i < NetworkInterfaceVector.size(); ++i) std::cout << i << ' ' << NetworkInterfaceVector[i].name() << '\t' << NetworkInterfaceVector[i].ipv4_address() << '\t' << NetworkInterfaceVector[i].hw_address() << std::endl;

    Tins::SnifferConfiguration SnifferConfig;
    SnifferConfig.set_immediate_mode(true);
    SnifferConfig.set_promisc_mode(false);
    SnifferConfig.set_filter("port 80 and src 192.168.0.30");

    Tins::Sniffer Sniffer("ens33", SnifferConfig);
    Sniffer.sniff_loop(CallbackFunction);

    return 0;
}

bool CallbackFunction(Tins::PDU &Packet) {
    const Tins::EthernetII EthernetPacket  = Packet.rfind_pdu<Tins::EthernetII>();
    std::cout << "\nEthernet Header (" << EthernetPacket.header_size() << ")" << std::endl;
    std::cout << "   |-Source Address\t\t: " << EthernetPacket.src_addr() << std::endl;
    std::cout << "   |-Destination Address\t: " << EthernetPacket.dst_addr() << std::endl;
    
    const Tins::IP IP = Packet.rfind_pdu<Tins::IP>();
    std::cout << "IP Header (" << IP.header_size() << ")" << std::endl;
    std::cout << "   |-Source IP\t\t\t: " << IP.src_addr() << std::endl;
    std::cout << "   |-Destination IP\t\t: " << IP.dst_addr() << std::endl;

    Tins::PacketSender PacketSender;
    Tins::EthernetII CraftedPacket(EthernetPacket.src_addr(), "00:00:00:00:00:00");
    CraftedPacket /= Tins::IP(IP.src_addr(), "8.8.8.8");

    switch ((unsigned)IP.protocol())
    {
    case IPPROTO_ICMP:
    {
        const Tins::ICMP ICMP = Packet.rfind_pdu<Tins::ICMP>();
        std::cout << "ICMP Header (" << ICMP.header_size() << ")" << std::endl;
        std::cout << "   |-Type\t\t\t: " << (unsigned int)ICMP.type() << std::endl;
        
        const Tins::RawPDU &RawPDU = ICMP.rfind_pdu<Tins::RawPDU>();
        const Tins::RawPDU::payload_type &Data = RawPDU.payload();
        std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
        PrintData(&RawPDU.payload().front(), RawPDU.payload_size());

        CraftedPacket /= ICMP;
        CraftedPacket.rfind_pdu<Tins::ICMP>().type(Tins::ICMP::Flags::ECHO_REPLY);
        break;
    }
    case IPPROTO_TCP:
    {
        const Tins::TCP TCP = Packet.rfind_pdu<Tins::TCP>();
        std::cout << "TCP Header (" << TCP.header_size() << ")" << std::endl;
        std::cout << "   |-Source Port\t\t: " << TCP.sport() << std::endl;
        std::cout << "   |-Destination Port\t\t: " << TCP.dport() << std::endl;
        
        const Tins::RawPDU &RawPDU = TCP.rfind_pdu<Tins::RawPDU>();
        const Tins::RawPDU::payload_type &Data = RawPDU.payload();
        std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
        PrintData(&RawPDU.payload().front(), RawPDU.payload_size());

        CraftedPacket /= TCP;
        CraftedPacket.rfind_pdu<Tins::TCP>().rfind_pdu<Tins::RawPDU>() = Tins::RawPDU("Hello World");
        break;
    }
    case IPPROTO_UDP:
    {
        const Tins::UDP UDP = Packet.rfind_pdu<Tins::UDP>();
        std::cout << "UDP Header (" << UDP.header_size() << ")" << std::endl;
        std::cout << "   |-Source Port\t\t: " << UDP.sport() << std::endl;
        std::cout << "   |-Destination Port\t\t: " << UDP.dport() << std::endl;
        
        const Tins::RawPDU &RawPDU = UDP.rfind_pdu<Tins::RawPDU>();
        const Tins::RawPDU::payload_type &Data = RawPDU.payload();
        std::cout << "Payload (" << RawPDU.payload_size() << ")" << std::endl;
        PrintData(&RawPDU.payload().front(), RawPDU.payload_size());

        CraftedPacket /= UDP;
        CraftedPacket.rfind_pdu<Tins::UDP>().rfind_pdu<Tins::RawPDU>() = Tins::RawPDU("Hello World");
        break;
    }
    default:
        break;
    }

    PacketSender.send(CraftedPacket, "ens33");
    return true;
}

void PrintData(const u_char* data , int Size)
{
    int i , j;
    for(i=0 ; i < Size ; i++){
        if( i!=0 && i%16==0){
            printf("         ");
            for(j=i-16 ; j<i ; j++){
                if(data[j]>=32 && data[j]<=128) printf("%c",(unsigned char)data[j]); //if its a number or alphabet
                else printf("."); //otherwise print a dot
            }
            printf("\n");
        }
        if(i%16==0) printf("   ");
            printf(" %02X",(unsigned int)data[i]);
        if( i==Size-1){
            for(j=0;j<15-i%16;j++){printf("   ");}
            printf("         ");
            for(j=i-i%16 ; j<=i ; j++){
                if(data[j]>=32 && data[j]<=128) {printf("%c",(unsigned char)data[j]);}
                else{printf(".");}
            }
            printf("\n" );
        }
    }
}
```