# spdlog

실행 환경

| Type          | Version                   |
| :---          | :---                      |
| OS            | Ubuntu 20.04.3 LTS        |
| Architecture  | x86-64                    |
| gcc           | gcc version 9.4.0         |
| libpcap-dev   | 1.9.1-3                   |

# **INDEX**

**1. [소스 설치](#소스-설치)**

**2. [로거 생성](#로거-생성)**

**3. [syslog 로깅](#syslog-로깅)**

**4. [로그 로테이팅](#로그-로테이팅)**

**5. [로그 레벨 설정](#로그-레벨-설정)**

**6. [로그 포맷](#로그-내용-포맷)**

 - [내용 포맷](#내용-포맷)

 - [시간 포맷](#시간-포맷)

 - [Hex 포맷](#Hex-포맷)

**7. [Full Code](#Full-Code)**


# **소스 설치**

로그를 읽고 쓰기 편리한 라이브러리로 header only파일이라 include만 시켜주면 된다.

소스를 다운받자.

```sh
root@ubuntu:~# git clone https://github.com/gabime/spdlog.git
```

컴파일시 헤더를 포함시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -I./spdlog/include/ -o outputfile
```

# **로거 생성**

spdlog를 이용하여 로깅을 해보자.

```cpp
std::shared_ptr<spdlog::logger> Logger;
Logger = spdlog::basic_logger_mt("LoggerName", "./LogFileName.log");
Logger->set_level(spdlog::level::debug);
spdlog::flush_on(spdlog::level::debug);
Logger->debug("Hello world");
```

생성된 파일을 확인해보자.

```sh
root@ubuntu:~# cat LogFileName.log
[2022-05-27 06:57:37.134] [LoggerName] [debug] Hello world
```

# **syslog 로깅**

spdlog를 이용하여 syslog에 로깅을 해보자.

```cpp
std::shared_ptr<spdlog::logger> SysLogger;
SysLogger = spdlog::syslog_logger_mt("syslog", "SysLoggerName", LOG_PID);
SysLogger->set_level(spdlog::level::debug);
spdlog::flush_on(spdlog::level::debug);
SysLogger->error("Hello World");
```

syslog를 확인해보자.

```cpp
root@ubuntu:~# cat /var/log/syslog | grep "SysLoggerName"
May 27 07:00:25 ubuntu SysLoggerName[7988]: Hello World
```

# **로그 로테이팅**

로그파일의 크기가 100byte를 넘으면 로그파일을 백업하고 최대 2개로 설정해보자.

```cpp
std::shared_ptr<spdlog::logger> RotatingLogger;
size_t LogFileMaxSize = 200;
size_t LogFileMaxCount = 2;
RotatingLogger = spdlog::rotating_logger_mt("RotatingLoggerName", "./LogFileName.log", LogFileMaxSize, LogFileMaxCount);
RotatingLogger->set_level(spdlog::level::debug);
spdlog::flush_on(spdlog::level::debug);

for(int idx = 0; idx < 10; idx ++) RotatingLogger->error("Hello World : {}", idx);
```

로그파일의 사이즈를 확인해보자.

```sh
root@ubuntu:~# ls -al | grep "LogFileName.*"
-rw-r--r--  1 root root     142 May 27 07:20 LogFileName.1.log
-rw-r--r--  1 root root     142 May 27 07:20 LogFileName.2.log
-rw-r--r--  1 root root     142 May 27 07:20 LogFileName.log
```

로그파일의 내용을 확인해보자.

```sh
root@ubuntu:~# grep -r "Hello World : " ./LogFileName.*
./LogFileName.1.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 6
./LogFileName.1.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 7
./LogFileName.2.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 4
./LogFileName.2.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 5
./LogFileName.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 8
./LogFileName.log:[2022-05-27 07:20:11.667] [RotatingLoggerName] [error] Hello World : 9
```

# **로그 레벨 설정**

최소 로그레벨을 설정하고 해당 레벨 이상만 로깅해보자.

* 로그 레벨

    trace -> debug -> info -> warn -> error -> critical

```cpp
std::shared_ptr<spdlog::logger> LevelLogger;
LevelLogger = spdlog::basic_logger_mt("LevelLoggerName", "./LevelLoggerName.log");
LevelLogger->set_level(spdlog::level::info);
spdlog::flush_on(spdlog::level::info);

LevelLogger->debug("hello world");
LevelLogger->info("hello world");
LevelLogger->warn("hello world");
LevelLogger->error("hello world");
LevelLogger->critical("hello world");
```

로그파일을 확인해보면 debug레벨은 기록이 안되어 있다.

```sh
root@ubuntu:~# cat LevelLoggerName.log
[2022-05-27 07:22:13.192] [LevelLoggerName] [info] hello world
[2022-05-27 07:22:13.192] [LevelLoggerName] [warning] hello world
[2022-05-27 07:22:13.192] [LevelLoggerName] [error] hello world
[2022-05-27 07:22:13.192] [LevelLoggerName] [critical] hello world
```

# **로그 포맷**

다양한 포맷팅을 활용해보자.

```cpp
std::shared_ptr<spdlog::logger> FormattingLogger;
FormattingLogger = spdlog::basic_logger_mt("FormattingLoggerName", "./FormattingLogger.log");
FormattingLogger->set_level(spdlog::level::debug);
spdlog::flush_on(spdlog::level::debug);
```

## **내용 포맷**

메세지에 대한 포맷팅을 적용해보자.

```cpp
FormattingLogger->debug("hello world");
FormattingLogger->info("{1} {0}", "hello", "world");
FormattingLogger->warn("float {:03.2f}", 3.989);
FormattingLogger->error("{:>7}", "hello world");
FormattingLogger->error("{:>7}", "hello");
FormattingLogger->critical("{:08d}", 10);
FormattingLogger->critical("{:03d}", 1011);
```

로그 파일의 내용을 확인해보자.

```sh
root@ubuntu:~# cat FormattingLogger.log
[2022-05-27 07:25:14.680] [FormattingLoggerName] [debug] hello world
[2022-05-27 07:25:14.680] [FormattingLoggerName] [info] world hello
[2022-05-27 07:25:14.680] [FormattingLoggerName] [warning] float 3.99
[2022-05-27 07:25:14.680] [FormattingLoggerName] [error] hello world
[2022-05-27 07:25:14.680] [FormattingLoggerName] [error]   hello
[2022-05-27 07:25:14.680] [FormattingLoggerName] [critical] 00000010
[2022-05-27 07:25:14.680] [FormattingLoggerName] [critical] 1011
```

## **시간 포맷**

시간에 대한 포맷팅을 적용해보자.

```cpp
spdlog::set_pattern("[%Y-%m-%d %X.%e][%n][%l] %v");
FormattingLogger->error("hello");
spdlog::set_pattern("[%Y-%B-%d (%A) %r %z][%n][%l] %v");
FormattingLogger->error("world");
```

로그 파일의 내용을 확인해보자.

```sh
root@ubuntu:~# cat FormattingLogger.log
[2022-05-27 07:27:35.807][FormattingLoggerName][error] hello
[2022-May-27 (Friday) 07:27:35 AM +00:00][FormattingLoggerName][error] world
```

포맷 별 결과는 다음과 같다.

| format    | result |
| :---:     | :---: |
| %a,%A     | Fri,Friday |
| %b,%B     | Oct,October |
| %c,%C     | Fri Oct 22 14:35:59 2021,21 |
| %d,%D     | 22,10/22/21 |
| %e,%E     | 963,1634880959 |
| %f,%F     | 963224,963224959 |
| %h,%H     | Oct,14 |
| %i,%I     | 17,02 |
| %l,%L     | error,E |
| %m,%M     | 10,35 |
| %n        | FormattingLoggerName |
| %o,%O     | 0,0 |
| %p,%P     | PM,63959 |
| %r,%R     | 02:35:59 PM,14:35 |
| %S        | 59 |
| %t,%T     | 63959,14:35:59 |
| %u        | 17106 |
| %v        | world |
| %x,%X     | 10/22/21,14:35:59 |
| %Y        | 2021 |
| %z        | +09:00 |

## **Hex 포맷**

byte값을 hex로 표현해보자.

```cpp
unsigned char data[3] = {0xa5, 0xde, 0x66};
FormattingLogger->error("bin to hex : {:xsn}", spdlog::to_hex(std::begin(data), std::begin(data) + sizeof(data)));
```

로그파일의 내용을 확인해보자.
```sh
root@ubuntu:~# cat FormattingLogger.log
[2022-May-27 (Friday) 07:38:36 AM +00:00][FormattingLoggerName][error] bin to hex : a5de66
```

포맷 별 결과는 다음과 같다.

| format    | result |
| :---:     | :---: |
| %x        | "\n0000: a5 de 66" |
| %s        | "\n0000: a5de66" |
| %p        | "\na5 de 66" |
| %n        | " a5 de 66" |
| %a        | "0000: a5 de 66  ..f" |
| %X        | "0000: A5 DE 66" |

예시에서 쓴 xsn은 소문자형태(x)와 구분자를 없애며(s) 위치(0000)과 개행(\n)을 제거(n)한 조합이다.

# **Full Code**

```cpp
#include "spdlog/spdlog.h"
#include "spdlog/sinks/basic_file_sink.h"
#include "spdlog/sinks/syslog_sink.h" // For syslog
#include "spdlog/sinks/rotating_file_sink.h" // For rotating
#include "spdlog/fmt/bin_to_hex.h" // To hex

int main(void)
{
    std::shared_ptr<spdlog::logger> Logger;
    Logger = spdlog::basic_logger_mt("LoggerName", "./LogFileName.log");
    Logger->set_level(spdlog::level::debug);
    spdlog::flush_on(spdlog::level::debug);
    Logger->debug("Hello world");

    std::shared_ptr<spdlog::logger> SysLogger;
    SysLogger = spdlog::syslog_logger_mt("syslog", "SysLoggerName", LOG_PID);
    SysLogger->set_level(spdlog::level::debug);
    spdlog::flush_on(spdlog::level::debug);
    SysLogger->error("Hello World");

    std::shared_ptr<spdlog::logger> RotatingLogger;
    size_t LogFileMaxSize = 200;
    size_t LogFileMaxCount = 2;
    RotatingLogger = spdlog::rotating_logger_mt("RotatingLoggerName", "./LogFileName.log", LogFileMaxSize, LogFileMaxCount);
    RotatingLogger->set_level(spdlog::level::debug);
    spdlog::flush_on(spdlog::level::debug);

    for(int idx = 0; idx < 10; idx ++) RotatingLogger->error("Hello World : {}", idx);

    std::shared_ptr<spdlog::logger> LevelLogger;
    LevelLogger = spdlog::basic_logger_mt("LevelLoggerName", "./LevelLoggerName.log");
    LevelLogger->set_level(spdlog::level::info);
    spdlog::flush_on(spdlog::level::info);

    LevelLogger->debug("hello world");
    LevelLogger->info("hello world");
    LevelLogger->warn("hello world");
    LevelLogger->error("hello world");
    LevelLogger->critical("hello world");

    std::shared_ptr<spdlog::logger> FormattingLogger;
    FormattingLogger = spdlog::basic_logger_mt("FormattingLoggerName", "./FormattingLogger.log");
    FormattingLogger->set_level(spdlog::level::debug);
    spdlog::flush_on(spdlog::level::debug);

    FormattingLogger->debug("hello world");
    FormattingLogger->info("{1} {0}", "hello", "world");
    FormattingLogger->warn("float {:03.2f}", 3.989);
    FormattingLogger->error("{:>7}", "hello world");
    FormattingLogger->error("{:>7}", "hello");
    FormattingLogger->critical("{:08d}", 10);
    FormattingLogger->critical("{:03d}", 1011);

    spdlog::set_pattern("[%Y-%m-%d %X.%e][%n][%l] %v");
    FormattingLogger->error("hello");
    spdlog::set_pattern("[%Y-%B-%d (%A) %r %z][%n][%l] %v");
    FormattingLogger->error("world");

    unsigned char data[3] = {0xa5, 0xde, 0x66};
    FormattingLogger->error("bin to hex : {:xsn}", spdlog::to_hex(std::begin(data), std::begin(data) + sizeof(data)));
}
```