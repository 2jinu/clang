# 패킷 캡처(libpcap)

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpcap-dev   | 1.9.1-3                   |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

**2. [네트워크 정보 확인](#네트워크-정보-확인)**

 - [인터페이스 이름 구하기](#인터페이스-이름-구하기)

 - [IP와 MAC 주소 구하기](#IP와-MAC-주소-구하기)

**3. [패킷 캡처](#패킷-캡처)**

 - [캡처 필터링 적용하기](#캡처-필터링-적용하기)

 - [캡처 콜백 함수](#캡처-콜백-함수)

**4. [패킷 분석](#패킷-분석)**

 - [ETHERNET](#ETHERNET)

 - [IP](#IP)

 - [ICMP](#ICMP)

 - [TCP](#TCP)

 - [UDP](#UDP)

**5. [Full Code](#Full-Code)**


# **패키지 설치**

libpcap-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libpcap-dev
```

컴파일시 libpcap을 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lpcap -o outputfile
```

# **네트워크 정보 확인**

## **인터페이스 이름 구하기**

[pcap_findalldevs](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_findalldevs)를 이용하여 시스템의 모든 인터페이스를 담는다.

```c++
int     Result;
char    ErrorBuffer[PCAP_ERRBUF_SIZE] = { '\0' };
pcap_if_t *alldevs, *d;

Result = pcap_findalldevs(&alldevs, ErrorBuffer);
if (Result == PCAP_ERROR) {
    printf("pcap_findalldevs error : %s\n", ErrorBuffer);
    return PCAP_ERROR;
}
```

모든 인터페이스를 담고 for문을 통해 하나씩 꺼내서 이름을 확인할 수 있다.

```c++
for (d = alldevs; d; d = d->next) printf("%-20s : %s\n", d->name, d->description);
```

하지만 패킷 캡처할 시 loopback이나 죽어있는 인터페이스는 필요 없으므로 살아있는 인터페이스만 선별해본다.

```c++
for (d = alldevs; d; d = d->next) {
    switch (d->flags)
    {
        case PCAP_IF_UP | PCAP_IF_RUNNING | PCAP_IF_CONNECTION_STATUS_CONNECTED:
        {
            printf("%s\t: %s\n", d->name, d->description);
            break;
        }
        default:
            break;
    }
}
```

이후 [pcap_freealldevs](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_freealldevs)를 통해 해제시켜준다.

```c++
pcap_freealldevs(alldevs);
```

## **IP와 MAC 주소 구하기**

인터페이스가 동작중이라면 해당 인터페이스의 IP주소와 MAC주소를 알아보자.

IP 주소는 [inet_ntoa](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/inet_ntoa) 함수를 사용하여 문자열로 바꾸어 출력한다.

```c++
for (struct pcap_addr *p = d->addresses; p; p = p->next) {
    if (p->addr->sa_family == AF_INET) printf("\t- IP Address  : %s\n", inet_ntoa(((struct sockaddr_in *)p->addr)->sin_addr));
}
```

MAC주소는 [ioctl](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/ioctl)를 이용하여 MAC주소를 HEX값으로 출력한다.

```c++
int sock = socket(AF_INET, SOCK_DGRAM, 0);
if (sock == INVALID_SOCKET) break;

struct ifreq ifr;
ifr.ifr_ifru.ifru_addr.sa_family = AF_INET;
strncpy(ifr.ifr_ifrn.ifrn_name, d->name, strlen(d->name));
if (ioctl(sock, SIOCGIFHWADDR, &ifr) == 0) printf("\t- MAC Address : %02x:%02x:%02x:%02x:%02x:%02x\n", (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[0], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[1], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[2], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[3], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[4], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[5]);
break;
```

# **패킷 캡처**

[pcap_lookupnet](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_lookupnet) 함수를 통해 네트워크 인터페이스의 정보를 구성한다.

```c++
bpf_u_int32 Network, Netmask;
const char *interface   = "ens33";

if (pcap_lookupnet(interface, &Network, &Netmask, ErrorBuffer) == PCAP_ERROR) {
    fprintf(stderr, "Couldn't get network/netmask address for device %s: %s\n", interface, ErrorBuffer);
    return -1;
}
struct in_addr ip_addr;
ip_addr.s_addr = Network;
printf("\t- Network     : %s\n", inet_ntoa(ip_addr));
ip_addr.s_addr = Netmask;
printf("\t- Subnet mask : %s\n", inet_ntoa(ip_addr));
```

[pcap_open_live](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_open_live) 함수를 통해 네트워크 인터페이스의 패킷 캡처에 대한 설정을 구성한다.

```c++
pcap_t *pcd = pcap_open_live(interface, BUFSIZ, 0, 1000, ErrorBuffer);
if (pcd == NULL) {
    fprintf(stderr, "Couldn't open device %s: %s\n", interface, ErrorBuffer);
    return -1;
}
```
## **캡처 필터링 적용하기**

[pcap_compile](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_compile) 함수를 통해 패킷 필터링을 컴파일하고 [pcap_setfilter](https://github.com/2jinu/clang/tree/main/%ED%95%A8%EC%88%98/pcap_setfilter) 함수를 통해 필터링을 적용한다.

```c++
struct bpf_program filter_code;
const char *filter      = "tcp port 80";

if (pcap_compile(pcd, &filter_code, filter, 0, Network) == PCAP_ERROR) {
    fprintf(stderr, "Couldn't parse filter %s: %s\n", filter, pcap_geterr(pcd));
    return -1;
}
if (pcap_setfilter(pcd, &filter_code) == PCAP_ERROR) {
    fprintf(stderr, "Couldn't install filter %s: %s\n", filter, pcap_geterr(pcd));
    return -1;
}
```

## **캡처 콜백 함수**

캡처가 준비되면 지정된 패킷이 캡처될 때, 함수를 호출하여 패킷을 다룰 수 있다.

[pcap_loop]()에 패킷 캡처 핸들과 함수를 지정한다.

```c++
void callback_function(u_char *args, const struct pcap_pkthdr *pkthdr, const u_char *packet);

int main(void) {
    ...
    pcap_loop(pcd, 0, callback_function, NULL);

    return 0;
}

void callback_function(u_char *args, const struct pcap_pkthdr *pkthdr, const u_char *packet)
{
    if ((unsigned short)ethernet_header->h_proto == 0x8)
    {
        switch (((struct iphdr*)(packet + sizeof(struct ethhdr)))->protocol) {
            case IPPROTO_ICMP: //1
                printf("icmp packet captured!!\n");
                break;
            case IPPROTO_TCP: //6
                printf("tcp packet captured!!\n");
                break;
            case IPPROTO_UDP: //17
                printf("udp packet captured!!\n");
                break;
            default:
                break;
        }
    }
}
```

# **패킷 분석**

## **ETHERNET**

ETHERNET 헤더는 14byte로 구성되어 있다.

| Type                      | Byte  |
| :---                      | :---  |
| Destination MAC Address   | 6     |
| Source MAC Address        | 6     |
| Type                      | 2     |

```c++
struct ethhdr *ethernet_header = (struct ethhdr *)packet;
printf("Ethernet Header (%d)\n", 14);
printf("   |-Destination Address\t: %.2X-%.2X-%.2X-%.2X-%.2X-%.2X \n", ethernet_header->h_dest[0] , ethernet_header->h_dest[1] , ethernet_header->h_dest[2] , ethernet_header->h_dest[3] , ethernet_header->h_dest[4] , ethernet_header->h_dest[5] );
printf("   |-Source Address\t\t: %.2X-%.2X-%.2X-%.2X-%.2X-%.2X \n", ethernet_header->h_source[0] , ethernet_header->h_source[1] , ethernet_header->h_source[2] , ethernet_header->h_source[3] , ethernet_header->h_source[4] , ethernet_header->h_source[5] );
printf("   |-Protocol\t\t\t: %u \n",(unsigned short)ethernet_header->h_proto);
```

## **IP**

IP 헤더는 20byte로 구성되어 있다.

| Type                      | Byte  |
| :---                      | :---  |
| Version                   | 0.5   |
| Header Length             | 0.5   |
| Type Of Service           | 1     |
| Total Length              | 2     |
| Identification            | 2     |
| Fragmentation Offset      | 2 (Flags + Offset) |
| TTL                       | 1     |
| Protocol                  | 1     |
| Checksum                  | 2     |
| Source IP Address         | 4     |
| Destination IP Address    | 4     |


```c++
struct iphdr *ip_header = (struct iphdr *)(packet + ETHERNET_HEADER_LEN);
struct sockaddr_in source,dest;
memset(&source, 0, sizeof(source));
memset(&dest, 0, sizeof(dest));
source.sin_addr.s_addr  = ip_header->saddr;
dest.sin_addr.s_addr    = ip_header->daddr;

printf("IP Header (%d)\n", IP_HEADER_LEN);
printf("   |-IP Version\t\t\t: %d\n",(unsigned int)ip_header->version);
printf("   |-IP Header Length\t\t: %d bytes (%d)\n",((unsigned int)(ip_header->ihl))*4, (unsigned int)ip_header->ihl);
printf("   |-Type Of Service\t\t: %d\n",(unsigned int)ip_header->tos);
printf("   |-IP Total Length\t\t: %d  Bytes(Size of Packet)\n",ntohs(ip_header->tot_len));
printf("   |-Identification\t\t: %d\n",ntohs(ip_header->id));
printf("   |-TTL\t\t\t: %d\n",(unsigned int)ip_header->ttl);
printf("   |-Protocol\t\t\t: %d\n",ip_header->protocol);
printf("   |-Checksum\t\t\t: %d\n",ntohs(ip_header->check));
printf("   |-Source IP\t\t\t: %s\n" , inet_ntoa(source.sin_addr) );
printf("   |-Destination IP\t\t: %s\n" , inet_ntoa(dest.sin_addr) );
```

## **ICMP**

ICMP 헤더는 8byte로 구성되어 있다.

| Type              | Byte  |
| :---              | :---  |
| Type              | 1     |
| Code              | 1     |
| Checksum          | 2     |
| Identifier        | 2     |
| Sequence number   | 2     |

```c++
struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
struct icmphdr *icmp_header = (struct icmphdr *)(packet + IP_HEADER_LEN  + ETHERNET_HEADER_LEN);
int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
int icmp_header_size        =  ETHERNET_HEADER_LEN + IP_HEADER_LEN + ICMP_HEADER_LEN;
int icmp_data_size          = total_len - icmp_header_size;
printf("\n\n***********************ICMP Packet (%d)*************************\n", total_len);
ethernet_parse(packet);
ip_parse(packet);
printf("ICMP Header (%d)\n", ICMP_HEADER_LEN);
printf("   |-Type\t\t\t: %d",(unsigned int)(icmp_header->type));
if((unsigned int)(icmp_header->type) == 11){printf("  (TTL Expired)\n");}
else if((unsigned int)(icmp_header->type) == ICMP_ECHOREPLY){printf("  (ICMP Echo Reply)\n");}
else printf("\n");
printf("   |-Code\t\t\t: %d\n",(unsigned int)(icmp_header->code));
printf("   |-Checksum\t\t\t: %d (%d)\n",ntohs(icmp_header->checksum), icmp_header->checksum);
printf("   |-Identifier (BE)\t\t: 0x%02X%02X\n", (packet + icmp_header_size)[-4], (packet + icmp_header_size)[-3]);
printf("   |-Identifier (LE)\t\t: 0x%02X%02X\n", (packet + icmp_header_size)[-3], (packet + icmp_header_size)[-4]);
printf("   |-Sequence number (BE)\t: 0x%02X%02X\n", (packet + icmp_header_size)[-2], (packet + icmp_header_size)[-1]);
printf("   |-Sequence number (LE)\t: 0x%02X%02X\n", (packet + icmp_header_size)[-1], (packet + icmp_header_size)[-2]);
printf("Data Payload (%d)\n", icmp_data_size);   
PrintData(packet + icmp_header_size , icmp_data_size );
```

## **TCP**

TCP 헤더는 20byte와 Options로 구성되어 있다.

| Type              | Byte  |
| :---              | :---  |
| Source Port       | 2     |
| Destination Port  | 2     |
| Sequence Number   | 4     |
| Acknowledge Number| 4     |
| Flag              | 2 (Header Length + Flag) |
| Window            | 2     |
| Checksum          | 2     |
| Urgent Pointer    | 2     |
| Options           | n     |

```c++
struct tcphdr *tcp_header   =(struct tcphdr*)(packet + IP_HEADER_LEN + ETHERNET_HEADER_LEN);
struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
int tcp_header_size         =  ETHERNET_HEADER_LEN + IP_HEADER_LEN + (unsigned int)tcp_header->doff*4;
int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
int tcp_data_size           = total_len - tcp_header_size;
printf("\n\n***********************TCP Packet (%d)*************************\n", total_len); 
printf("TCP Header (%d)\n", (unsigned int)tcp_header->doff*4);
printf("   |-Source Port\t\t: %u\n",ntohs(tcp_header->source));
printf("   |-Destination Port\t\t: %u\n",ntohs(tcp_header->dest));
printf("   |-Sequence Number\t\t: %u\n",ntohl(tcp_header->seq));
printf("   |-Acknowledge Number\t\t: %u\n",ntohl(tcp_header->ack_seq));
printf("   |-Header Length\t\t: %d DWORDS or %d BYTES\n" ,(unsigned int)tcp_header->doff,(unsigned int)tcp_header->doff*4);
printf("   |-Urgent Flag\t\t: %d\n",(unsigned int)tcp_header->urg);
printf("   |-Acknowledgement Flag\t: %d\n",(unsigned int)tcp_header->ack);
printf("   |-Push Flag\t\t\t: %d\n",(unsigned int)tcp_header->psh);
printf("   |-Reset Flag\t\t\t: %d\n",(unsigned int)tcp_header->rst);
printf("   |-Synchronise Flag\t\t: %d\n",(unsigned int)tcp_header->syn);
printf("   |-Finish Flag\t\t: %d\n",(unsigned int)tcp_header->fin);
printf("   |-Window\t\t\t: %d\n",ntohs(tcp_header->window));
printf("   |-Checksum\t\t\t: %d\n",ntohs(tcp_header->check));
printf("   |-Urgent Pointer\t\t: %d\n",tcp_header->urg_ptr);
printf("Data Payload (%d)\n", tcp_data_size);   
PrintData(packet + tcp_header_size , tcp_data_size );
```

## **UDP**

UDP 헤더는 8byte로 구성되어 있다.

| Type              | Byte  |
| :---              | :---  |
| Source Port       | 2     |
| Destination Port  | 2     |
| UDP Length        | 2     |
| UDP Checksum      | 2     |

```c++
struct udphdr *udp_header   = (struct udphdr*)(packet + IP_HEADER_LEN  + ETHERNET_HEADER_LEN);
struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
int udp_header_size         =  ETHERNET_HEADER_LEN + IP_HEADER_LEN + UDP_HEADER_LEN;
int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
int udp_data_size           = total_len - udp_header_size;
printf("\n\n***********************UDP Packet (%d)*************************\n", UDP_HEADER_LEN);
printf("UDP Header (%d)\n", UDP_HEADER_LEN);
printf("   |-Source Port\t\t: %d\n" , ntohs(udp_header->source));
printf("   |-Destination Port\t\t: %d\n" , ntohs(udp_header->dest));
printf("   |-UDP Length\t\t\t: %d\n" , ntohs(udp_header->len));
printf("   |-UDP Checksum\t\t: %d\n" , ntohs(udp_header->check));
printf("Data Payload (%d)\n", udp_data_size);   
PrintData(packet + udp_header_size , udp_data_size );
```

# **Full Code**

```c++
#include <pcap.h>
#include <string.h>
#include <sys/ioctl.h>
#include <net/if.h>
#include <net/ethernet.h>
#include <netinet/ip.h>
#include <netinet/ip_icmp.h>
#include <netinet/tcp.h>
#include <netinet/udp.h>

void callback_function(u_char *args, const struct pcap_pkthdr *pkthdr, const u_char *packet);
void PrintData (const u_char* data , int Size);

int main(void) {
    int     Result;
    char    ErrorBuffer[PCAP_ERRBUF_SIZE] = { '\0' };
    pcap_if_t *alldevs, *d;

    Result = pcap_findalldevs(&alldevs, ErrorBuffer);
    if (Result == PCAP_ERROR) {
        printf("pcap_findalldevs error : %s\n", ErrorBuffer);
        return PCAP_ERROR;
    }

    for (d = alldevs; d; d = d->next) {
        switch (d->flags)
        {
            case PCAP_IF_UP | PCAP_IF_RUNNING | PCAP_IF_CONNECTION_STATUS_CONNECTED:
            {
                printf("%s\t: %s\n", d->name, d->description);

                for (struct pcap_addr *p = d->addresses; p; p = p->next) {
                    if (p->addr->sa_family == AF_INET) printf("\t- IP Address  : %s\n", inet_ntoa(((struct sockaddr_in *)p->addr)->sin_addr));
                }

                int sock = socket(AF_INET, SOCK_DGRAM, 0);
                if (sock == INVALID_SOCKET) break;

                struct ifreq ifr;
                ifr.ifr_ifru.ifru_addr.sa_family = AF_INET;
                strncpy(ifr.ifr_ifrn.ifrn_name, d->name, strlen(d->name));
                if (ioctl(sock, SIOCGIFHWADDR, &ifr) == 0) printf("\t- MAC Address : %02x:%02x:%02x:%02x:%02x:%02x\n", (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[0], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[1], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[2], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[3], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[4], (unsigned char)ifr.ifr_ifru.ifru_hwaddr.sa_data[5]);
                break;
            }
            default:
                break;
        }
    }
    pcap_freealldevs(alldevs);

    bpf_u_int32 Network, Netmask;
    const char *interface   = "ens33";

    if (pcap_lookupnet(interface, &Network, &Netmask, ErrorBuffer) == PCAP_ERROR) {
        fprintf(stderr, "Couldn't get Network for device %s: %s\n", interface, ErrorBuffer);
        return -1;
    }
    struct in_addr ip_addr;
    ip_addr.s_addr = Network;
    printf("\t- Network     : %s\n", inet_ntoa(ip_addr));
    ip_addr.s_addr = Netmask;
    printf("\t- Subnet mask : %s\n", inet_ntoa(ip_addr));

    pcap_t *pcd = pcap_open_live(interface, BUFSIZ, 0, 1000, ErrorBuffer);
    if (pcd == NULL) {
        fprintf(stderr, "Couldn't open device %s: %s\n", interface, ErrorBuffer);
        return -1;
    }

    struct bpf_program filter_code;
    const char *filter      = "tcp port 80 or icmp";

    if (pcap_compile(pcd, &filter_code, filter, 0, Netmask) == PCAP_ERROR) {
        fprintf(stderr, "Couldn't parse filter %s: %s\n", filter, pcap_geterr(pcd));
        return -1;
    }
    if (pcap_setfilter(pcd, &filter_code) == PCAP_ERROR) {
        fprintf(stderr, "Couldn't install filter %s: %s\n", filter, pcap_geterr(pcd));
        return -1;
    }

    pcap_loop(pcd, 0, callback_function, NULL);

    return 0;
}

void callback_function(u_char *args, const struct pcap_pkthdr *pkthdr, const u_char *packet)
{
    int ETHERNET_HEADER_LEN = 14;
    int IP_HEADER_LEN       = 20;
    int ICMP_HEADER_LEN     = 8;
    int UDP_HEADER_LEN      = 8;
    struct ethhdr *ethernet_header = (struct ethhdr *)packet;
    printf("Ethernet Header (%d)\n", ETHERNET_HEADER_LEN);
    printf("   |-Destination Address\t: %.2X-%.2X-%.2X-%.2X-%.2X-%.2X \n", ethernet_header->h_dest[0] , ethernet_header->h_dest[1] , ethernet_header->h_dest[2] , ethernet_header->h_dest[3] , ethernet_header->h_dest[4] , ethernet_header->h_dest[5] );
    printf("   |-Source Address\t\t: %.2X-%.2X-%.2X-%.2X-%.2X-%.2X \n", ethernet_header->h_source[0] , ethernet_header->h_source[1] , ethernet_header->h_source[2] , ethernet_header->h_source[3] , ethernet_header->h_source[4] , ethernet_header->h_source[5] );
    printf("   |-Protocol\t\t\t: %u \n",(unsigned short)ethernet_header->h_proto);

    if ((unsigned short)ethernet_header->h_proto == 0x8)
    {
        struct iphdr *ip_header = (struct iphdr *)(packet + ETHERNET_HEADER_LEN);
        struct sockaddr_in source,dest;
        memset(&source, 0, sizeof(source));
        memset(&dest, 0, sizeof(dest));
        source.sin_addr.s_addr = ip_header->saddr;
        dest.sin_addr.s_addr = ip_header->daddr;

        printf("IP Header (%d)\n", IP_HEADER_LEN);
        printf("   |-IP Version\t\t\t: %d\n",(unsigned int)ip_header->version);
        printf("   |-IP Header Length\t\t: %d bytes (%d)\n",((unsigned int)(ip_header->ihl))*4, (unsigned int)ip_header->ihl);
        printf("   |-Type Of Service\t\t: %d\n",(unsigned int)ip_header->tos);
        printf("   |-IP Total Length\t\t: %d  Bytes(Size of Packet)\n",ntohs(ip_header->tot_len));
        printf("   |-Identification\t\t: %d\n",ntohs(ip_header->id));
        printf("   |-TTL\t\t\t: %d\n",(unsigned int)ip_header->ttl);
        printf("   |-Protocol\t\t\t: %d\n",ip_header->protocol);
        printf("   |-Checksum\t\t\t: %d\n",ntohs(ip_header->check));
        printf("   |-Source IP\t\t\t: %s\n" , inet_ntoa(source.sin_addr) );
        printf("   |-Destination IP\t\t: %s\n" , inet_ntoa(dest.sin_addr) );

        switch (((struct iphdr*)(packet + sizeof(struct ethhdr)))->protocol) {
            case IPPROTO_ICMP: //1
            {
                struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
                struct icmphdr *icmp_header = (struct icmphdr *)(packet + IP_HEADER_LEN  + ETHERNET_HEADER_LEN);
                int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
                int icmp_header_size        = ETHERNET_HEADER_LEN + IP_HEADER_LEN + ICMP_HEADER_LEN;
                int icmp_data_size          = total_len - icmp_header_size;
                printf("\n\n***********************ICMP Packet (%d)*************************\n", total_len);
                printf("ICMP Header (%d)\n", ICMP_HEADER_LEN);
                printf("   |-Type\t\t\t: %d",(unsigned int)(icmp_header->type));
                if((unsigned int)(icmp_header->type) == 11){printf("  (TTL Expired)\n");}
                else if((unsigned int)(icmp_header->type) == ICMP_ECHOREPLY){printf("  (ICMP Echo Reply)\n");}
                else printf("\n");
                printf("   |-Code\t\t\t: %d\n",(unsigned int)(icmp_header->code));
                printf("   |-Checksum\t\t\t: %d (%d)\n",ntohs(icmp_header->checksum), icmp_header->checksum);
                printf("   |-Identifier (BE)\t\t: 0x%02X%02X\n", (packet + icmp_header_size)[-4], (packet + icmp_header_size)[-3]);
                printf("   |-Identifier (LE)\t\t: 0x%02X%02X\n", (packet + icmp_header_size)[-3], (packet + icmp_header_size)[-4]);
                printf("   |-Sequence number (BE)\t: 0x%02X%02X\n", (packet + icmp_header_size)[-2], (packet + icmp_header_size)[-1]);
                printf("   |-Sequence number (LE)\t: 0x%02X%02X\n", (packet + icmp_header_size)[-1], (packet + icmp_header_size)[-2]);
                printf("Data Payload (%d)\n", icmp_data_size);   
                PrintData(packet + icmp_header_size , icmp_data_size );
                break;
            }
            case IPPROTO_TCP: //6
            {
                struct tcphdr *tcp_header   =(struct tcphdr*)(packet + IP_HEADER_LEN + ETHERNET_HEADER_LEN);
                struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
                int tcp_header_size         =  ETHERNET_HEADER_LEN + IP_HEADER_LEN + (unsigned int)tcp_header->doff*4;
                int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
                int tcp_data_size           = total_len - tcp_header_size;
                printf("\n\n***********************TCP Packet (%d)*************************\n", total_len); 
                printf("TCP Header (%d)\n", (unsigned int)tcp_header->doff*4);
                printf("   |-Source Port\t\t: %u\n",ntohs(tcp_header->source));
                printf("   |-Destination Port\t\t: %u\n",ntohs(tcp_header->dest));
                printf("   |-Sequence Number\t\t: %u\n",ntohl(tcp_header->seq));
                printf("   |-Acknowledge Number\t\t: %u\n",ntohl(tcp_header->ack_seq));
                printf("   |-Header Length\t\t: %d DWORDS or %d BYTES\n" ,(unsigned int)tcp_header->doff,(unsigned int)tcp_header->doff*4);
                printf("   |-Urgent Flag\t\t: %d\n",(unsigned int)tcp_header->urg);
                printf("   |-Acknowledgement Flag\t: %d\n",(unsigned int)tcp_header->ack);
                printf("   |-Push Flag\t\t\t: %d\n",(unsigned int)tcp_header->psh);
                printf("   |-Reset Flag\t\t\t: %d\n",(unsigned int)tcp_header->rst);
                printf("   |-Synchronise Flag\t\t: %d\n",(unsigned int)tcp_header->syn);
                printf("   |-Finish Flag\t\t: %d\n",(unsigned int)tcp_header->fin);
                printf("   |-Window\t\t\t: %d\n",ntohs(tcp_header->window));
                printf("   |-Checksum\t\t\t: %d\n",ntohs(tcp_header->check));
                printf("   |-Urgent Pointer\t\t: %d\n",tcp_header->urg_ptr);
                printf("Data Payload (%d)\n", tcp_data_size);   
                PrintData(packet + tcp_header_size , tcp_data_size );
                break;
            }
            case IPPROTO_UDP: //17
            {
                struct udphdr *udp_header   = (struct udphdr*)(packet + IP_HEADER_LEN  + ETHERNET_HEADER_LEN);
                struct iphdr *ip_header     = (struct iphdr *)(packet  + ETHERNET_HEADER_LEN);
                int udp_header_size         =  ETHERNET_HEADER_LEN + IP_HEADER_LEN + UDP_HEADER_LEN;
                int total_len               = (ntohs(ip_header->tot_len) + ETHERNET_HEADER_LEN);
                int udp_data_size           = total_len - udp_header_size;
                printf("\n\n***********************UDP Packet (%d)*************************\n", UDP_HEADER_LEN);
                printf("UDP Header (%d)\n", UDP_HEADER_LEN);
                printf("   |-Source Port\t\t: %d\n" , ntohs(udp_header->source));
                printf("   |-Destination Port\t\t: %d\n" , ntohs(udp_header->dest));
                printf("   |-UDP Length\t\t\t: %d\n" , ntohs(udp_header->len));
                printf("   |-UDP Checksum\t\t: %d\n" , ntohs(udp_header->check));
                printf("Data Payload (%d)\n", udp_data_size);   
                PrintData(packet + udp_header_size , udp_data_size );
                break;
            }
            default:
                break;
        }
    }
}

void PrintData (const u_char* data , int Size)
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