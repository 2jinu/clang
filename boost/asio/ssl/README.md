# clang

실행 환경

| Type              | Version                   |
| :---              | :---                      |
| OS                | Ubuntu 20.04.3 LTS        |
| Architecture      | x86-64                    |
| gcc               | gcc version 9.4.0         |
| libssl-dev        | 1.1.1f-1ubuntu2.13        |
| libboost-all-dev  | 1.71.0.0ubuntu2           |

# **INDEX**

**1. [패키지 설치](#패키지-설치)**

 - [ssl 설치](#ssl-설치)

 - [boost 설치](#boost-설치)

 - [소스 컴파일](#소스-컴파일)

**2. [SSL](#SSL)**

 - [Server](#Server)

 - [Client](#Client)

**3. [Full Code](#Full-Code)**

 - [Server Code](#Server-Code)

 - [Client Code](#Client-Code)


# **패키지 설치**

## **ssl 설치**

libssl-dev를 설치한다.

```sh
root@ubuntu:~# apt-get -y install libssl-dev
```

## **boost 설치**

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

## **소스 컴파일**

컴파일시 libssl, libcrypto, libpthread를 링크시켜준다.

```sh
root@ubuntu:~# g++ -g source.cpp -lssl -lcrypto -lpthread -o outputfile
```

# **SSL**

서버에서 SSL 암호화를 위한 인증서를 생성하자.

인증서를 생성하기 위해 비밀키와 인증요청서를 생성하자.

```sh
openssl req -nodes -newkey rsa:2048 -keyout private.key -out certificate.csr -subj "/C=KR/ST=Seoul/L=Seoul/O=2jinu/OU=2jinu"
```

인증요청서에 대해 자체 인증서를 만들자.

```sh
openssl req -x509 -key private.key -in certificate.csr -out certificate.crt -days 365
```

## **Server**

클라이언트 별 SSL 세션 클래스를 만들고 접속 시 SSL Handshake를 수행한다.

```cpp
class session : public std::enable_shared_from_this<session>
{
    public:
        session(boost::asio::ip::tcp::socket socket, boost::asio::ssl::context& context) : socket_(std::move(socket), context) {}
        void start() { do_handshake(); }
    private:
        void do_handshake()
        {
            auto self(shared_from_this());
            ClientAddress_ = socket_.lowest_layer().remote_endpoint().address().to_string() + ":" + std::to_string(socket_.lowest_layer().remote_endpoint().port());
            socket_.async_handshake(boost::asio::ssl::stream_base::server, [this, self](const boost::system::error_code& ec) {
                if (!ec) {
                    std::cout << "[" << ClientAddress_ << "] Connect" << std::endl;
                    do_read();
                }
                else close();
            });
        }
};
```

Handshake가 성공하면 클라이언트로부터 데이터를 읽는다.

```cpp
    private:
        ...
        void do_read()
        {
            auto self(shared_from_this());
            socket_.async_read_some(boost::asio::buffer(data_), [this, self](const boost::system::error_code& ec, std::size_t length) {
                if (!ec) {
                    std::cout << "[" << ClientAddress_ << "] " << (char *)data_ << std::endl;
                    do_write(length);
                }
                else close();
            });
        }
```

echo 형태로 읽은 데이터를 그대로 클라이언트에 전송해보자.

```cpp
    private:
        ...
        void do_write(std::size_t length)
        {
            auto self(shared_from_this());
            boost::asio::async_write(socket_, boost::asio::buffer(data_, length), [this, self](const boost::system::error_code& ec, std::size_t /*length*/) {
                if (!ec) do_read();
                else close();
            });
        }
```

통신 중 에러가 발생하면 소켓을 닫는다.

```cpp
    private:
        ...
        void close()
        {
            boost::mutex::scoped_lock lock(Mutex);
            if (socket_.lowest_layer().is_open()) {
                socket_.lowest_layer().close();
                std::cout << "[" << ClientAddress_ << "] Close" << std::endl;
            }
            std::cout << "[" << ClientAddress_ << "] Disconnect" << std::endl;
        }
        boost::asio::ssl::stream<boost::asio::ip::tcp::socket> socket_;
        std::string ClientAddress_;
        char data_[1024];
```

다음으로는 클라이언트의 연결 요청을 수락하는 클래스를 생성하자. SSL 2버전은 사용안하는 옵션을 주며 인증서를 로드하자.

```cpp
class SSLServer
{
    public:
        SSLServer(boost::asio::io_context& io_context, unsigned short port) : Acceptor_(io_context, boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(), port)), Context_(boost::asio::ssl::context::sslv23)
        {
            Context_.set_options( boost::asio::ssl::context::default_workarounds | boost::asio::ssl::context::no_sslv2);

            Context_.use_certificate_chain_file("certificate.crt", ErrorCode_);
            if (!ErrorCode_) {
                Context_.use_private_key_file("private.key", boost::asio::ssl::context::pem, ErrorCode_);
                if (!ErrorCode_) {
                    std::cout << "Waiting for client..." << std::endl;
                    do_accept();
                }
                else std::cerr << "use_private_key_file error : " << ErrorCode_.message() << std::endl;
            }
            else std::cerr << "use_certificate_chain_file error : " << ErrorCode_.message() << std::endl;
        }
};
```

클라이언트의 연결 요청이 있으면 수락하며 세션을 생선한다.

```cpp
class SSLServer
{
    ...
    private:
        void do_accept()
        {
            Acceptor_.async_accept([this](const boost::system::error_code& error, boost::asio::ip::tcp::socket socket) {
                if (!error) std::make_shared<session>(std::move(socket), Context_)->start();
                do_accept();
            });
        }
        boost::asio::ip::tcp::acceptor  Acceptor_;
        boost::asio::ssl::context       Context_;
        boost::system::error_code       ErrorCode_;
};
```

메인함수에서 SSLServer클래스를 실행하자.

SSL 서버는 12345번 포트에 바인딩된다.

```cpp
    try
    {
        boost::asio::io_context SSLIOContext;
        SSLServer s(SSLIOContext, 12345);
        SSLIOContext.run();
    }
    catch (std::exception& e) { std::cerr << "Exception: " << e.what() << "\n"; }
```

## **Client**

1개의 프로세스로 1개의 서버와 통신하는 클라이언트의 경우 세션이 필요없다.

클라이언트 클래스를 생성하며 서버의 인증서를 자체서명 인증서이기 때문에 verify mode는 none으로 해주자.

```cpp
class client
{
    public:
        client(boost::asio::io_context& io_context, boost::asio::ssl::context& context, const boost::asio::ip::tcp::resolver::results_type& endpoints) : socket_(io_context, context) {
        socket_.set_verify_mode(boost::asio::ssl::verify_none);
        connect(endpoints);
        }
};
```

endpoints로 접속 요청을 시도하자.

```cpp
class client
{
    ...
    private:
        void connect(const boost::asio::ip::tcp::resolver::results_type& endpoints)
        {
            boost::asio::async_connect(socket_.lowest_layer(), endpoints, [this](const boost::system::error_code& ec, const boost::asio::ip::tcp::endpoint& /*endpoint*/) {
                if (!ec) handshake();
                else std::cout << "Connect failed: " << ec.message() << "\n";
            });
        }
};
```

접속이 되면 SSL Handshake를 수행한다.

```cpp
class client
{
    ...
    void handshake()
        {
        socket_.async_handshake(boost::asio::ssl::stream_base::client, [this](const boost::system::error_code& ec) {
            if (!ec) send_request();
            else std::cout << "Handshake failed: " << ec.message() << "\n";
        });
        }
};
```

Handshake까지 수행이 완료되면 메세지를 보내보자.

```cpp
class client
{
    ...
    void send_request()
    {
        std::cout << "Enter message: ";
        std::cin.getline(request_, max_length);
        size_t request_length = std::strlen(request_);

        boost::asio::async_write(socket_, boost::asio::buffer(request_, request_length), [this](const boost::system::error_code& error, std::size_t length) {
            if (!error) receive_response(length);
            else std::cout << "Write failed: " << error.message() << "\n";
        });
    }
};
```

메세지를 보낸 후 서버의 메세지를 받는다.

```cpp
class client
{
    ...
    void receive_response(std::size_t length)
    {
        boost::asio::async_read(socket_, boost::asio::buffer(reply_, length), [this](const boost::system::error_code& error, std::size_t length) {
            if (!error)
            {
            std::cout << "Reply: ";
            std::cout.write(reply_, length);
            std::cout << "\n";
            }
            else std::cout << "Read failed: " << error.message() << "\n";
        });
    }
    boost::asio::ssl::stream<boost::asio::ip::tcp::socket> socket_;
    char request_[max_length];
    char reply_[max_length];
};
```

메인 함수에서 서버의 정보를 넣어 클래스를 실행시키자.

```cpp
    try
    {
        boost::asio::io_context io_context;
        boost::asio::ip::tcp::resolver resolver(io_context);
        auto endpoints = resolver.resolve("127.0.0.1", "12345");
        boost::asio::ssl::context ctx(boost::asio::ssl::context::sslv23);
        client c(io_context, ctx, endpoints);
        io_context.run();
    }
    catch (std::exception& e) { std::cerr << "Exception: " << e.what() << "\n"; }
```

# **Full Code**

## **Server Code**

```cpp
#include <boost/asio.hpp>
#include <boost/asio/ssl.hpp>
#include <boost/thread/mutex.hpp>
#include <iostream>

static boost::mutex Mutex;

class session : public std::enable_shared_from_this<session>
{
    public:
        session(boost::asio::ip::tcp::socket socket, boost::asio::ssl::context& context) : socket_(std::move(socket), context) {}
        void start() { do_handshake(); }

    private:
        void do_handshake()
        {
            auto self(shared_from_this());
            ClientAddress_ = socket_.lowest_layer().remote_endpoint().address().to_string() + ":" + std::to_string(socket_.lowest_layer().remote_endpoint().port());
            socket_.async_handshake(boost::asio::ssl::stream_base::server, [this, self](const boost::system::error_code& ec) {
                if (!ec) {
                    std::cout << "[" << ClientAddress_ << "] Connect" << std::endl;
                    do_read();
                }
                else close();
            });
        }

        void do_read()
        {
            auto self(shared_from_this());
            socket_.async_read_some(boost::asio::buffer(data_), [this, self](const boost::system::error_code& ec, std::size_t length) {
                if (!ec) {
                    std::cout << "[" << ClientAddress_ << "] " << (char *)data_ << std::endl;
                    do_write(length);
                }
                else close();
            });
        }

        void do_write(std::size_t length)
        {
            auto self(shared_from_this());
            boost::asio::async_write(socket_, boost::asio::buffer(data_, length), [this, self](const boost::system::error_code& ec, std::size_t /*length*/) {
                if (!ec) do_read();
                else close();
            });
        }

        void close()
        {
            boost::mutex::scoped_lock lock(Mutex);
            if (socket_.lowest_layer().is_open()) {
                socket_.lowest_layer().close();
                std::cout << "[" << ClientAddress_ << "] Close" << std::endl;
            }
            std::cout << "[" << ClientAddress_ << "] Disconnect" << std::endl;
        }
        boost::asio::ssl::stream<boost::asio::ip::tcp::socket> socket_;
        std::string ClientAddress_;
        char data_[1024];
};

class SSLServer
{
    public:
        SSLServer(boost::asio::io_context& io_context, unsigned short port) : Acceptor_(io_context, boost::asio::ip::tcp::endpoint(boost::asio::ip::tcp::v4(), port)), Context_(boost::asio::ssl::context::sslv23)
        {
            Context_.set_options( boost::asio::ssl::context::default_workarounds | boost::asio::ssl::context::no_sslv2);

            Context_.use_certificate_chain_file("certificate.crt", ErrorCode_);
            if (!ErrorCode_) {
                Context_.use_private_key_file("private.key", boost::asio::ssl::context::pem, ErrorCode_);
                if (!ErrorCode_) {
                    std::cout << "Waiting for client..." << std::endl;
                    do_accept();
                }
                else std::cerr << "use_private_key_file error : " << ErrorCode_.message() << std::endl;
            }
            else std::cerr << "use_certificate_chain_file error : " << ErrorCode_.message() << std::endl;
        }

    private:
        void do_accept()
        {
            Acceptor_.async_accept([this](const boost::system::error_code& error, boost::asio::ip::tcp::socket socket) {
                if (!error) std::make_shared<session>(std::move(socket), Context_)->start();
                do_accept();
            });
        }
        boost::asio::ip::tcp::acceptor  Acceptor_;
        boost::asio::ssl::context       Context_;
        boost::system::error_code       ErrorCode_;
};

int main(int argc, char* argv[]) {
    try
    {
        boost::asio::io_context SSLIOContext;
        SSLServer s(SSLIOContext, 12345);
        SSLIOContext.run();
    }
    catch (std::exception& e) { std::cerr << "Exception: " << e.what() << "\n"; }

    return 0;
}
```

## **Client Code**

```cpp
#include <cstdlib>
#include <cstring>
#include <functional>
#include <iostream>
#include <boost/asio.hpp>
#include <boost/asio/ssl.hpp>

enum { max_length = 1024 };

class client
{
    public:
        client(boost::asio::io_context& io_context, boost::asio::ssl::context& context, const boost::asio::ip::tcp::resolver::results_type& endpoints) : socket_(io_context, context) {
            socket_.set_verify_mode(boost::asio::ssl::verify_none);
            connect(endpoints);
        }
    private:
        void connect(const boost::asio::ip::tcp::resolver::results_type& endpoints)
        {
            boost::asio::async_connect(socket_.lowest_layer(), endpoints, [this](const boost::system::error_code& ec, const boost::asio::ip::tcp::endpoint& /*endpoint*/) {
            if (!ec) handshake();
            else std::cout << "Connect failed: " << ec.message() << "\n";
            });
        }
        void handshake()
        {
            socket_.async_handshake(boost::asio::ssl::stream_base::client, [this](const boost::system::error_code& ec) {
            if (!ec) send_request();
            else std::cout << "Handshake failed: " << ec.message() << "\n";
            });
        }
        void send_request()
        {
            std::cout << "Enter message: ";
            std::cin.getline(request_, max_length);
            size_t request_length = std::strlen(request_);

            boost::asio::async_write(socket_, boost::asio::buffer(request_, request_length), [this](const boost::system::error_code& error, std::size_t length) {
                if (!error) receive_response(length);
                else std::cout << "Write failed: " << error.message() << "\n";
            });
        }
        void receive_response(std::size_t length)
        {
            boost::asio::async_read(socket_, boost::asio::buffer(reply_, length), [this](const boost::system::error_code& error, std::size_t length) {
            if (!error)
            {
                std::cout << "Reply: ";
                std::cout.write(reply_, length);
                std::cout << "\n";
            }
            else std::cout << "Read failed: " << error.message() << "\n";
            });
        }
        boost::asio::ssl::stream<boost::asio::ip::tcp::socket> socket_;
        char request_[max_length];
        char reply_[max_length];
};

int main(int argc, char* argv[])
{
    try
    {
        boost::asio::io_context io_context;
        boost::asio::ip::tcp::resolver resolver(io_context);
        auto endpoints = resolver.resolve("127.0.0.1", "12345");
        boost::asio::ssl::context ctx(boost::asio::ssl::context::sslv23);
        client c(io_context, ctx, endpoints);
        io_context.run();
    }
    catch (std::exception& e) { std::cerr << "Exception: " << e.what() << "\n"; }

    return 0;
}
```