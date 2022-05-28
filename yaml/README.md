# yaml

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpcap-dev   | 1.9.1-3                   |

# **INDEX**

**1. [소스 설치](#소스-설치)**

**2. [YAML 파일 파싱](#YAML-파일-파싱)**

**3. [YAML 파일 작성](#YAML-파일-작성)**

**4. [Full Code](#Full-Code)**


# **소스 설치**

YAML파일을 읽고 쓰기 편리한 라이브러리이다.

소스를 다운받자.

```sh
root@ubuntu:~# git clone https://github.com/jbeder/yaml-cpp.git
root@ubuntu:~# cd yaml-cpp/
root@ubuntu:~/yaml-cpp# mkdir build
root@ubuntu:~/yaml-cpp# cd build
```

소스를 컴파일하자.

```sh
root@ubuntu:~/yaml-cpp/build# apt -y install cmake
root@ubuntu:~/yaml-cpp/build# cmake -DYAML_BUILD_SHARED_LIBS=ON ..
root@ubuntu:~/yaml-cpp/build# make && make install
root@ubuntu:~/yaml-cpp/build# cd ../
root@ubuntu:~/yaml-cpp# cp -r include/yaml-cpp /usr/local/include
root@ubuntu:~/yaml-cpp# rm -rf build/
```

ARM으로 컴파일 하려면 다음과 같이 한다.

```sh
root@ubuntu:~/code/yaml-cpp/build# cmake ../ -DCMAKE_FIND_ROOT_PATH=/ -DCMAKE_CXX_COMPILER=/usr/bin/aarch64-linux-gnu-g++ -DYAML_BUILD_SHARED_LIBS=ON
```

컴파일시 libyaml-cpp를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lyaml-cpp -o outputfile
```

# **YAML 파일 파싱**

파싱을 위한 데이터를 작성하자.

```sh
root@ubuntu:~# vi example.yaml
```

```yaml
network:
  ethernets:
    enp1s0:
      addresses: [192.168.2.1/24]
      dhcp4: false
      gateway4: 192.168.2.1
      nameservers:
        addresses: [168.126.63.1, 168.126.63.2]
    enp2s0:
      addresses: [192.168.1.1/24]
      dhcp4: false
```

addresses와 같이 배열은 Sequence 타입, dhcp4나 gateway4와 같이 값만 있다면 Scalar, 그 외 network나 ethernets와 같이 하위 키가 또 있다면 Map으로 타입이 정의된다.

재귀 형식으로 데이터를 파싱해보자.

```cpp
void ParsingYAMLFile(YAML::Node Node) {
    for (YAML::iterator iter = Node.begin(); iter != Node.end(); ++iter)
    {
        std::cout << "KEY\t\t" << iter->first << std::endl;
        if (iter->second.IsMap()) ParsingYAMLFile(iter->second);
        else if (iter->second.IsScalar()) std::cout << "Value\t\t" << iter->second << std::endl;
        else if (iter->second.IsSequence())
        {
            for (std::size_t idx = 0; idx < iter->second.size(); idx++) std::cout << "Sequence [" << idx << "]\t" << iter->second[idx] << std::endl;
        }
    }
}

int main(void) {
    YAML::Node Node = YAML::LoadFile("example.yaml");
    ParsingYAMLFile(Node);
    return 0;
}
```

혹은 Map의 이름으로 접근할 수 있다.

```cpp
std::cout << std::endl;
std::cout << "[network]" << std::endl;
std::cout << Node["network"] << std::endl;
std::cout << "[ethernets]" << std::endl;
std::cout << Node["network"]["ethernets"] << std::endl;
std::cout << "[enp1s0]" << std::endl;
std::cout << Node["network"]["ethernets"]["enp1s0"] << std::endl;
std::cout << "[dhcp4]" << std::endl;
std::cout << Node["network"]["ethernets"]["enp1s0"]["dhcp4"] << std::endl;
```

as를 이용하여 데이터 타입을 변환할 수 있다.

```cpp
std::cout << std::endl;
std::cout << ((Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<bool>()) ? "dhcp is true" : "dhcp is false") << std::endl;
std::cout << typeid(Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<bool>()).name() << std::endl;
std::cout << typeid(Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<std::string>()).name() << std::endl;
```

# **YAML 파일 작성**

YAML파일을 작성해보자.

```cpp
std::ofstream writeFile("./example.yaml");
YAML::Emitter Emitter;
Emitter << YAML::BeginMap;
    Emitter << YAML::Key << "network" << YAML::Value;
    Emitter << YAML::BeginMap;
        Emitter << YAML::Key << "ethernets" << YAML::Value;
        Emitter << YAML::BeginMap;
            Emitter << YAML::Key << "enp1s0" << YAML::Value;
            Emitter << YAML::BeginMap;
                Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "192.168.2.1/24" << YAML::EndSeq;
                Emitter << YAML::Key << "gateway4" << YAML::Value << "192.168.2.1";
                Emitter << YAML::Key << "nameservers" << YAML::Value;
                    Emitter << YAML::BeginMap;
                    Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "168.126.63.1" << "168.126.63.2" << YAML::EndSeq;
                    Emitter << YAML::EndMap;
            Emitter << YAML::Key << "dhcp4" << YAML::Value << "false";
            Emitter << YAML::EndMap;

            Emitter << YAML::Key << "enp2s0" << YAML::Value;
            Emitter << YAML::BeginMap;
                Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "192.168.1.1/24" << YAML::EndSeq;
                Emitter << YAML::Key << "dhcp4" << YAML::Value << "false";
            Emitter << YAML::EndMap;
        Emitter << YAML::EndMap;
    Emitter << YAML::EndMap;
Emitter << YAML::EndMap;

writeFile << Emitter.c_str();
writeFile.close();
```

# **Full Code**

```cpp
#include <yaml-cpp/yaml.h>
#include <iostream>
#include <fstream>

void ParsingYAMLFile(YAML::Node Node) {
    for (YAML::iterator iter = Node.begin(); iter != Node.end(); ++iter)
    {
        std::cout << "KEY\t\t" << iter->first << std::endl;
        if (iter->second.IsMap()) ParsingYAMLFile(iter->second);
        else if (iter->second.IsScalar()) std::cout << "Value\t\t" << iter->second << std::endl;
        else if (iter->second.IsSequence())
        {
            for (std::size_t idx = 0; idx < iter->second.size(); idx++) std::cout << "Sequence [" << idx << "]\t" << iter->second[idx] << std::endl;
        }
    }
}

int main(void) {
    YAML::Node Node = YAML::LoadFile("example.yaml");
    ParsingYAMLFile(Node);

    std::cout << std::endl;
    std::cout << "[network]" << std::endl;
    std::cout << Node["network"] << std::endl;
    std::cout << "[ethernets]" << std::endl;
    std::cout << Node["network"]["ethernets"] << std::endl;
    std::cout << "[enp1s0]" << std::endl;
    std::cout << Node["network"]["ethernets"]["enp1s0"] << std::endl;
    std::cout << "[dhcp4]" << std::endl;
    std::cout << Node["network"]["ethernets"]["enp1s0"]["dhcp4"] << std::endl;

    std::cout << std::endl;
    std::cout << ((Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<bool>()) ? "dhcp is true" : "dhcp is false") << std::endl;
    std::cout << typeid(Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<bool>()).name() << std::endl;
    std::cout << typeid(Node["network"]["ethernets"]["enp1s0"]["dhcp4"].as<std::string>()).name() << std::endl;

    std::ofstream writeFile("./example.yaml");
    
    YAML::Emitter Emitter;
    Emitter << YAML::BeginMap;
        Emitter << YAML::Key << "network" << YAML::Value;
        Emitter << YAML::BeginMap;
            Emitter << YAML::Key << "ethernets" << YAML::Value;
            Emitter << YAML::BeginMap;
                Emitter << YAML::Key << "enp1s0" << YAML::Value;
                Emitter << YAML::BeginMap;
                    Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "192.168.2.1/24" << YAML::EndSeq;
                    Emitter << YAML::Key << "gateway4" << YAML::Value << "192.168.2.1";
                    Emitter << YAML::Key << "nameservers" << YAML::Value;
                        Emitter << YAML::BeginMap;
                        Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "168.126.63.1" << "168.126.63.2" << YAML::EndSeq;
                        Emitter << YAML::EndMap;
                Emitter << YAML::Key << "dhcp4" << YAML::Value << "false";
                Emitter << YAML::EndMap;

                Emitter << YAML::Key << "enp2s0" << YAML::Value;
                Emitter << YAML::BeginMap;
                    Emitter << YAML::Key << "addresses" << YAML::Value << YAML::Flow << YAML::BeginSeq << "192.168.1.1/24" << YAML::EndSeq;
                    Emitter << YAML::Key << "dhcp4" << YAML::Value << "false";
                Emitter << YAML::EndMap;
            Emitter << YAML::EndMap;
        Emitter << YAML::EndMap;
    Emitter << YAML::EndMap;

    writeFile << Emitter.c_str();
    writeFile.close();

    return 0;
}
```