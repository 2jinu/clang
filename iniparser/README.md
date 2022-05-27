# iniparser

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |

# **INDEX**

**1. [소스 설치](#소스-설치)**

**2. [INI 파일 파싱](#INI-파일-파싱)**

**3. [INI 파일 작성](#INI-파일-작성)**

**4. [Full Code](#Full-Code)**


# **소스 설치**

[INI 파일](https://ko.wikipedia.org/wiki/INI_%ED%8C%8C%EC%9D%BC)을 읽고 쓰기 편리한 라이브러리로 header only파일이라 include만 시켜주면 된다.

소스를 다운받자.

```sh
root@ubuntu:~# git clone https://github.com/Lek-sys/LeksysINI.git
```

컴파일시 헤더를 포함시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -I./LeksysINI/ -o outputfile
```

# **INI 파일 파싱**

파싱을 위한 데이터를 작성해놓자.

```sh
root@ubuntu:~# vi example.ini
[EXAMPLE1]
x = abc
y = 123
z = 1,2,3
t = true
[EXAMPLE2]
x = abc
y = 123
z = 1,2,3
t = true
```

작성한 INI 파일을 로드한다.

```cpp
INI::File ft;
if(ft.Load("example.ini")){
}
```

Section과 Value를 지정하여 타입으로 읽을 수 있다.

```cpp
std::string x       = ft.GetSection("EXAMPLE1")->GetValue("x","").AsString();
int y               = ft.GetSection("EXAMPLE1")->GetValue("y",0).AsInt();
std::vector<int> z  = ft.GetSection("EXAMPLE1")->GetValue("z").AsArray().ToVector<int>();
bool t              = ft.GetSection("EXAMPLE1")->GetValue("t",false).AsBool();

std::cout << "x is " << x << std::endl;
std::cout << "y is " << y << std::endl;
for(int idx = 0; idx < z.size(); idx++) std::cout << "z[" << idx << "] is " << z[idx] << std::endl;
std::cout << "t is " << t << std::endl;
```

반복문을 이용하여 모든 데이터를 읽을 수 있다.

```cpp
for (INI::File::sections_iter it = ft.SectionsBegin(); it != ft.SectionsEnd(); ++it)
{
    if((it->first).find("EXAMPLE") != std::string::npos)
    {
        INI::Section* sect = it->second;
        for (INI::Section::values_iter it2 = sect->ValuesBegin(); it2 != sect->ValuesEnd(); ++it2)
        {
            std::cout << it->first << " > " << it2->first << " : " << it2->second.AsString() << std::endl;
        }
    }
}
```

# **INI 파일 작성**

파일을 여는 함수인 fopen/fopen_s의 w모드처럼 기존 데이터가 있으면 모두 지우며 데이터를 작성한다.

```cpp
INI::File ft2;

ft2.SetValue("EXAMPLE1:x", "abc");
ft2.SetArrayValue("EXAMPLE1:y", 0, 1);
ft2.SetArrayValue("EXAMPLE1:y", 1, 2);
ft2.SetArrayValue("EXAMPLE1:y", 2, 3);
ft2.SetValue("EXAMPLE1:z", "xyz");
ft2.Save("example.ini");
```

INI파일을 확인해보자.

```sh
root@ubuntu:~# cat example.ini
[EXAMPLE1]
x = abc
y = 1,2,3
z = xyz

```

# **Full Code**

```cpp
#include "iniparser.hpp"
#include <iostream>

int main(void) {
    INI::File ft;
    if(ft.Load("example.ini")){
        std::string x       = ft.GetSection("EXAMPLE1")->GetValue("x","").AsString();
        int y               = ft.GetSection("EXAMPLE1")->GetValue("y",0).AsInt();
        std::vector<int> z  = ft.GetSection("EXAMPLE1")->GetValue("z").AsArray().ToVector<int>();
        bool t              = ft.GetSection("EXAMPLE1")->GetValue("t",false).AsBool();

        std::cout << "x is " << x << std::endl;
        std::cout << "y is " << y << std::endl;
        for(int idx = 0; idx < z.size(); idx++) std::cout << "z[" << idx << "] is " << z[idx] << std::endl;
        std::cout << "t is " << t << std::endl;

        for (INI::File::sections_iter it = ft.SectionsBegin(); it != ft.SectionsEnd(); ++it)
        {
            if((it->first).find("EXAMPLE") != std::string::npos)
            {
                INI::Section* sect = it->second;
                for (INI::Section::values_iter it2 = sect->ValuesBegin(); it2 != sect->ValuesEnd(); ++it2)
                {
                    std::cout << it->first << " > " << it2->first << " : " << it2->second.AsString() << std::endl;
                }
            }
        }
    }

    INI::File ft2;

    ft2.SetValue("EXAMPLE1:x", "abc");
    ft2.SetArrayValue("EXAMPLE1:y", 0, 1);
    ft2.SetArrayValue("EXAMPLE1:y", 1, 2);
    ft2.SetArrayValue("EXAMPLE1:y", 2, 3);
    ft2.SetValue("EXAMPLE1:z", "xyz");
    ft2.Save("example.ini");

    return 0;
}
```