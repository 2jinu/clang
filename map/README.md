# map

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |

# **INDEX**

**1. [요소 삽입](#요소-삽입)**

**2. [요소 삭제](#요소-삭제)**

**3. [요소 찾기](#요소-찾기)**

**4. [요소 읽기](#요소-읽기)**

**5. [Full Code](#Full-Code)**


# **요소 삽입**

map을 이용하여 키와 값으로 쌍을 이루는 데이터를 넣어보자.

```cpp
std::map<int, std::string> MyMap;
MyMap.insert(std::make_pair(1, "one"));
MyMap.insert(std::make_pair(2, "two"));
MyMap.insert(std::make_pair(9, "nine"));
MyMap.insert(std::make_pair(10, "ten"));
MyMap.insert(std::make_pair(-5, "negative"));
```

# **요소 삭제**

erase로 키와 값을 삭제할 수 있다.

```cpp
MyMap.erase(-5);
```

# **요소 찾기**

원하는 키값을 가지고 map에 있는지 찾을 수 있다.

```cpp
if(MyMap.find(9) != MyMap.end()) std::cout << "find : " << MyMap[9] << std::endl;
```

# **요소 읽기**

python의 dictionary처럼 키값을 주고 값을 읽을 수 있다.

```cpp
std::cout << MyMap[2] << std::endl;
std::cout << MyMap[9] << std::endl;
std::cout << MyMap[1] << std::endl;
```

반복문을 통해 map의 모든 데이터를 읽을 수도 있다.

```cpp
for(std::map<int, std::string>::iterator iter = MyMap.begin(); iter != MyMap.end(); iter ++)
{
    std::cout << iter->first << " : " << iter->second << std::endl;
}
```

# **Full Code**

```cpp
#include <map>
#include <iostream>

int main(void) {
    std::map<int, std::string> MyMap;
    MyMap.insert(std::make_pair(1, "one"));
    MyMap.insert(std::make_pair(2, "two"));
    MyMap.insert(std::make_pair(9, "nine"));
    MyMap.insert(std::make_pair(10, "ten"));
    MyMap.insert(std::make_pair(-5, "negative"));    

    MyMap.erase(-5);

    if(MyMap.find(9) != MyMap.end()) std::cout << "find : " << MyMap[9] << std::endl;

    std::cout << MyMap[2] << std::endl;
    std::cout << MyMap[9] << std::endl;
    std::cout << MyMap[1] << std::endl;

    for(std::map<int, std::string>::iterator iter = MyMap.begin(); iter != MyMap.end(); iter ++)
    {
        std::cout << iter->first << " : " << iter->second << std::endl;
    }

    return 0;
}
```