# clang

실행 환경

| Type              | Version                   |
| :---              | :---                      |
| OS                | Ubuntu 20.04.3 LTS        |
| Architecture      | x86-64                    |
| gcc               | gcc version 9.4.0         |
| libboost-all-dev  | 1.71.0.0ubuntu2           |

# **INDEX**

**1. [boost 설치](#boost-설치)**

**2. [JSON 파싱](#JSON-파싱)**

**3. [JSON 작성](#JSON-작성)**

**4. [Full Code](#Full-Code)**


# **boost 설치**

boost 관련 라이브러리들을 설치한다.

```sh
root@ubuntu:~# apt-get -y install libboost-all-dev
```

수동 설치 시 [boost](https://www.boost.org/users/history/)에서 소스를 다운받은 후 AMD의 경우 다음과 같이 bootstrap.sh을 실행킨다.

```sh
root@ubuntu:~# tar -zxf boost_1_76_0.tar.gz
root@ubuntu:~# cd boost_1_76_0
root@ubuntu:~/boost_1_76_0# ./bootstrap.sh
```

ARM의 경우 project-config.jam을 변경시켜줘야 한다.

```sh
root@ubuntu:~# tar -zxf boost_1_76_0.tar.gz
root@ubuntu:~# cd boost_1_76_0
root@ubuntu:~/boost_1_76_0# ./b2 install
root@ubuntu:~/boost_1_76_0# vi project-config.jam
if ! gcc in [ feature.values <toolset> ]
{
    using gcc : arm : aarch64-linux-gnu-g++ ;
}
root@ubuntu:~/boost_1_76_0# ./b2 install toolset=gcc-arm
root@ubuntu:~/boost_1_76_0# cp -r boost /usr/local/include/
```

# **JSON 파싱**

JSON 파싱을 위한 데이터를 준비하자.

```cpp
const char *JSONData = "{\"FIRST\":\"HELLO WORLD\",\"SECOND\":\"2jinu\",\"ARRAY\":[\"1\",\"2\"],\"X\":{\"Y\":\"Z\"}}";
```

데이터를 stream으로 변경한 뒤 read_json으로 ptree형태로 변경해주자.

```cpp
std::stringstream StringStream;
boost::property_tree::ptree JSONPtree;

StringStream << JSONData;
boost::property_tree::read_json(StringStream, JSONPtree);
```

파싱할 데이터의 타입과 키값으로 값을 읽을 수 있으며, 키 값이 없다면 반한될 데이터를 지정할 수 있다.

```cpp
std::cout << JSONPtree.get<std::string>("FIRST",   "") << std::endl;
std::cout << JSONPtree.get<std::string>("SECOND",  "") << std::endl;
std::cout << JSONPtree.get<std::string>("THIRD",   "NOT EXIST") << std::endl;
```

키 값을 다시 ptree형태로 변환하여 데이터를 읽을 수 있다.

```cpp
boost::property_tree::ptree& JSONPtreeArrayChild = JSONPtree.get_child("ARRAY");
for (boost::property_tree::ptree::value_type &element : JSONPtreeArrayChild) {
    std::cout << element.second.get<unsigned int>("") << std::endl;
}
boost::property_tree::ptree& JSONPtreeJSONChild = JSONPtree.get_child("X");
std::cout << JSONPtreeJSONChild.get<char>("Y",  "None") << std::endl;
std::cout << JSONPtreeJSONChild.get<char>("YY", "None") << std::endl;
```

반복문을 통해서 모든 데이터를 읽어보자.

```cpp
BOOST_FOREACH(boost::property_tree::ptree::value_type &v, JSONPtree) {
    std::cout << v.first << " : ";
    if (v.second.data().empty()) {
        std::cout << std::endl;
        BOOST_FOREACH(boost::property_tree::ptree::value_type &vv, v.second) {
            std::cout << "\t";
            if (!vv.first.empty()) std::cout << vv.first << " : ";
            std::cout << vv.second.data() << std::endl;
        }
    }
    else {
        std::cout << v.second.data() << std::endl;
    }
}
```

검색도 지원한다.

```cpp
boost::property_tree::ptree::assoc_iterator fit = JSONPtree.find("SECOND");
if (JSONPtree.not_found() != fit) std::cout << "Value : " << (*fit).second.data() << std::endl;
```

# **JSON 작성**

여러 ptree를 통해 JSON 데이터를 작성할 수 있다.

```cpp
StringStream.str(std::string());
boost::property_tree::ptree JSONPtreeParent, JSONPtreeChildren, JSONPtreeChild1, JSONPtreeChild2, JSONPtreeChild3;
JSONPtreeParent.put("FIRST", "HELLO WORLD");
JSONPtreeParent.put("SECOND", "2honrr");

JSONPtreeChild1.put("", 1);
JSONPtreeChild2.put("", 2);
JSONPtreeChildren.push_back(std::make_pair("", JSONPtreeChild1));
JSONPtreeChildren.push_back(std::make_pair("", JSONPtreeChild2));
JSONPtreeParent.add_child("ARRAY", JSONPtreeChildren);

JSONPtreeChild3.put("Y", "Z");
JSONPtreeParent.add_child("X", JSONPtreeChild3);
```

write_json으로 stream에 JSON 데이터를 넣자.

```cpp
boost::property_tree::write_json(StringStream, JSONPtreeParent, false);
std::cout << StringStream.str() << std::endl;

StringStream.str(std::string());
boost::property_tree::write_json(StringStream, JSONPtreeParent);
std::cout << StringStream.str() << std::endl;
```

# **Full Code**

```cpp
#include <boost/property_tree/json_parser.hpp>
#include <boost/foreach.hpp>
#include <iostream>

int main(void)
{
    const char *JSONData = "{\"FIRST\":\"HELLO WORLD\",\"SECOND\":\"2jinu\",\"ARRAY\":[\"1\",\"2\"],\"X\":{\"Y\":\"Z\"}}";
    std::stringstream StringStream;
    boost::property_tree::ptree JSONPtree;

    StringStream << JSONData;
    boost::property_tree::read_json(StringStream, JSONPtree);
    std::cout << JSONPtree.get<std::string>("FIRST",   "") << std::endl;
    std::cout << JSONPtree.get<std::string>("SECOND",  "") << std::endl;
    std::cout << JSONPtree.get<std::string>("THIRD",   "NOT EXIST") << std::endl;

    boost::property_tree::ptree& JSONPtreeArrayChild = JSONPtree.get_child("ARRAY");
    for (boost::property_tree::ptree::value_type &element : JSONPtreeArrayChild) {
        std::cout << element.second.get<unsigned int>("") << std::endl;
    }
    
    boost::property_tree::ptree& JSONPtreeJSONChild = JSONPtree.get_child("X");
    std::cout << JSONPtreeJSONChild.get<char>("Y",  "None") << std::endl;
    std::cout << JSONPtreeJSONChild.get<char>("YY", "None") << std::endl;
    std::cout << std::endl;

    BOOST_FOREACH(boost::property_tree::ptree::value_type &v, JSONPtree) {
        std::cout << v.first << " : ";
        if (v.second.data().empty()) {
            std::cout << std::endl;
            BOOST_FOREACH(boost::property_tree::ptree::value_type &vv, v.second) {
                std::cout << "\t";
                if (!vv.first.empty()) std::cout << vv.first << " : ";
                std::cout << vv.second.data() << std::endl;
            }
        }
        else {
            std::cout << v.second.data() << std::endl;
        }
    }
    std::cout << std::endl;

    boost::property_tree::ptree::assoc_iterator fit = JSONPtree.find("SECOND");
    if (JSONPtree.not_found() != fit) std::cout << "Value : " << (*fit).second.data() << std::endl;
    std::cout << std::endl;

    StringStream.str(std::string());
    boost::property_tree::ptree JSONPtreeParent, JSONPtreeChildren, JSONPtreeChild1, JSONPtreeChild2, JSONPtreeChild3;
    JSONPtreeParent.put("FIRST", "HELLO WORLD");
    JSONPtreeParent.put("SECOND", "2honrr");

    JSONPtreeChild1.put("", 1);
    JSONPtreeChild2.put("", 2);
    JSONPtreeChildren.push_back(std::make_pair("", JSONPtreeChild1));
    JSONPtreeChildren.push_back(std::make_pair("", JSONPtreeChild2));
    JSONPtreeParent.add_child("ARRAY", JSONPtreeChildren);

    JSONPtreeChild3.put("Y", "Z");
    JSONPtreeParent.add_child("X", JSONPtreeChild3);

    boost::property_tree::write_json(StringStream, JSONPtreeParent, false);
    std::cout << StringStream.str() << std::endl;

    StringStream.str(std::string());
    boost::property_tree::write_json(StringStream, JSONPtreeParent);
    std::cout << StringStream.str() << std::endl;
}
```