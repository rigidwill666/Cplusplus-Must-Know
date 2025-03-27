# 基础概念
Socket 编程是一种计算机网络通信技术，它允许不同设备或进程通过网络进行数据传输。Socket（套接字）是操作系统提供的**网络编程接口**，支持**TCP 和 UDP 协议**，常用于开发客户端-服务器（C/S）架构的应用。
> 解释Socket在网络通信中的作用？TCP和UDP协议的区别是什么？：
>
> Socket（套接字）是操作系统提供的网络编程接口，用于不同设备或进程之间的数据传输。它可以使用不同的协议（如 TCP、UDP）进行通信；网络层与应用层的桥梁，屏蔽底层实现细节，让开发者专注于数据处理；实现跨平台通信，支持 Windows、Linux、macOS。
>
> TCP 面向连接（三次握手四次挥手），UDP 不面向连接；TCP 传输可靠，UDP 不可靠，数据可能丢失或乱序；TCP 流式传输，UDP 以数据报发送；TCP 有拥塞控制，UDP 无；TCP 速度（需要确认和重传）较慢，UDP 较快；TCP 适用于数据完整性要求高的应用如文件传输、邮件、数据库查询，UDP 适用于对实时性要求高但可以容忍数据丢失的应用如视频会议、语音通话、直播。
>
> TCP 是如何确保可靠性？：
>
> 主要包括**三次握手、数据校验、超时重传、流量控制和拥塞控制**等。在数据传输前，TCP 需要建立连接，使用三次握手确保双方都准备好收发数据；TCP 在数据包头部使用校验和机制，检测数据在传输过程中是否发生错误或损坏；TCP 每个数据包都会分配一个序列号，确保数据按照正确的顺序到达接收端；如果 TCP 发送数据后未在规定时间内收到 ACK，它会重新发送数据，确保数据最终到达接收方；TCP 使用滑动窗口机制控制数据发送的速率，防止发送方发送太快、接收方处理不过来；TCP 使用拥塞控制算法，根据网络负载情况调整发送速率，防止网络拥堵。
>
> 三次握手？为什么需要三次握手？：
>
> TCP（三次握手）是 TCP 协议建立可靠连接的关键过程，三次握手是为了**防止已失效的连接请求影响新连接；保证客户端和服务器的收发能力**。
>
> Client → Server: [SYN] Seq = X
>
> Server → Client: [SYN, ACK] Seq = Y, Ack = X + 1
>
> Server → Client: [SYN, ACK] Seq = Y, Ack = X + 1
> 
> 四次挥手？：
>
> 四次挥手用于**断开连接**，确保双方都能优雅地释放资源。整个过程涉及两个方向的数据关闭，因此需要四次。
>
> A → B: FIN, SEQ = x
>
> B → A: ACK, SEQ = y, ACK = x + 1
>
> B → A: FIN, SEQ = y
>
> A → B: ACK, SEQ = x + 1, ACK = y + 1
>
> 为什么 TCP 关闭连接需要四次挥手，而建立连接只需三次握手？：
>
> 三次握手时，服务端收到 SYN 后可以立即 ACK 并 SYN，合并成一条报文，减少一次通信；四次挥手需要双方独立关闭数据流，不能合并 FIN 和 ACK，所以需要额外一次确认。
>
> TIME_WAIT 过后，A 为什么才能关闭？：
>
> 保证 B 收到 A 的 ACK，否则 B 重发 FIN，A 需要处理；防止旧连接的报文影响新连接，避免 TCP 端口复用时数据错乱。
>
> 什么是 TCP 粘包/拆包？如何解决？：
>
> TCP 是面向流的协议，没有消息边界，所以应用层数据可能会被合并（接收方读取数据不及时，一次性读取多个消息导致粘包）或拆分（数据长度超过 TCP MSS，导致拆包），这就是 TCP 粘包/拆包问题。
>
>  1.固定长度协议；2.特殊分隔符；3.消息头 + 消息体（长度前缀）；4.应用层协议，如 HTTP、WebSocket、MQTT 都有自己协议格式，明确 消息边界，避免粘包问题。
# 核心流程
## TCP 通信流程
- 服务器端

  创建套接字（socket()）
  
  绑定 IP 和端口（bind()）
  
  监听连接（listen()）
  
  接受客户端连接（accept()）
  
  发送和接收数据（send() / recv()）
  
  关闭连接（close()）

- 客户端

  创建套接字（socket()）
  
  连接服务器（connect()）
  
  发送和接收数据（send() / recv()）
  
  关闭连接（close()）
### C++ 实现 TCP 服务器示例
~~~
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <cstring>
#define PORT 8080
int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int addrlen = sizeof(address);
    char buffer[1024] = {0};
    // 1. 创建服务器 socket
    server_fd = socket(AF_INET, SOCK_STREAM, 0);
    if (server_fd == 0) {
        perror("Socket 创建失败");
        exit(EXIT_FAILURE);
    }
    // 2. 绑定 IP 和端口
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);
    if (bind(server_fd, (struct sockaddr*)&address, sizeof(address)) < 0) {
        perror("绑定失败");
        exit(EXIT_FAILURE);
    }
    // 3. 监听客户端连接
    if (listen(server_fd, 3) < 0) {
        perror("监听失败");
        exit(EXIT_FAILURE);
    }
    std::cout << "等待客户端连接..." << std::endl;
    // 4. 接受客户端连接
    new_socket = accept(server_fd, (struct sockaddr*)&address, (socklen_t*)&addrlen);
    if (new_socket < 0) {
        perror("连接失败");
        exit(EXIT_FAILURE);
    }
    // 5. 接收数据
    read(new_socket, buffer, 1024);
    std::cout << "收到客户端消息: " << buffer << std::endl;
    // 6. 发送数据
    char* hello = "Hello from server";
    send(new_socket, hello, strlen(hello), 0);
    // 7. 关闭 socket
    close(new_socket);
    close(server_fd);
    return 0;
}
~~~
### C++ 实现 TCP 客户端示例
~~~
#include <iostream>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <cstring>
#define PORT 8080
int main() {
    int sock = 0;
    struct sockaddr_in serv_addr;
    char buffer[1024] = {0};
    // 1. 创建 socket
    sock = socket(AF_INET, SOCK_STREAM, 0);
    if (sock < 0) {
        perror("Socket 创建失败");
        return -1;
    }
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);
    // 2. 连接服务器
    if (connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr)) < 0) {
        perror("连接失败");
        return -1;
    }
    // 3. 发送数据
    char* hello = "Hello from client";
    send(sock, hello, strlen(hello), 0);
    // 4. 接收数据
    read(sock, buffer, 1024);
    std::cout << "收到服务器消息: " << buffer << std::endl;
    // 5. 关闭 socket
    close(sock);
    return 0;
}
~~~
> bind()函数的作用是什么？如果服务端不调用bind()直接listen()会发生什么？:
> 
> bind() 是用于将一个本地地址（IP 地址和端口号）绑定到指定的 Socket 上的函数。它的作用是告诉操作系统，当前的 Socket 将使用指定的地址和端口进行通信。
>
> 在调用 listen() 之前如果没有调用 bind() 来绑定 IP 和端口，操作系统会自动为 Socket 分配一个**临时的端口**。即使没有明确绑定端口，listen() 仍然可以成功执行，服务端仍然会进入监听状态，并等待客户端连接。但由于未指定端口，客户端很难连接到服务器。
> 
> accept()返回的新Socket是做什么用的？它和监听Socket有何区别？:
> 
> 在 TCP 网络编程中，accept() 函数是用来接受客户端的连接请求的。当服务器端调用 accept() 时，它会从**监听队列**中取出一个客户端的连接，并返回一个新的 Socket，用于与该客户端进行数据交换。
>
> 监听 Socket 仅用于监听客户端的连接请求。它不会参与实际的数据传输，只是等待客户端的连接，通常只有一个;客户端 Socket 与客户端进行实际的通信，发送和接收数据。每当服务器接受一个连接请求时，accept() 会返回一个新的 Socket，每个客户端都会有一个对应的客户端 Socket。
>
> 调用recv()时返回0或-1分别代表什么情况？如何判断错误类型（如EAGAIN）？:
>
> 返回 0，表示连接已经被对方关闭，连接正常结束；返回 -1，表示发生了错误，接收数据失败。此时需要调用 errno 来获取具体的错误码——*EAGAIN/EWOULDBLOCK*表示非阻塞模式下当前没有数据可读，可以稍后重试；*EINTR*表示接收操作被信号中断，可以重新调用 recv()；*ECONNRESET*表示连接被远程主机重置，可以选择关闭当前的 Socket 连接；*EINVAL*表示无效的参数，检查调用 recv() 时的参数确保其正确。
>
> 为什么需要htonl()/ntohl()等函数？网络字节序是什么？：
>
> 数据需要在不同的机器之间传输。不同机器的处理器可能使用不同的字节序（即字节的存储顺序），因此，为了保证跨平台的兼容性和一致性，网络协议（如 TCP/IP）定义了网络字节序。网络字节序规定了数据在网络上传输时的字节顺序。*htonl()*将一个 32 位整数从主机字节序转换为网络字节序。*ntohl()*将一个 32 位整数从网络字节序转换为主机字节序（htons()和 ntohs()用于处理 16 位整数）。
>
> 网络字节序是指在网络通信中，数据的字节顺序是以**大端字节序**（高位字节存储在低地址处，低位字节存储在高地址处）的方式存储的。
## UDP 通信流程
- 服务器端

  创建套接字（socket()）
  
  绑定 IP 和端口（bind()）
  
  接收数据（recvfrom()）
  
  发送数据（sendto()）
  
  关闭套接字（close()）

- 客户端

  创建套接字（socket()）
  
  发送数据（sendto()）
  
  接收数据（recvfrom()）
  
  关闭套接字（close()）
### C++ 实现 UDP 服务器示例
~~~
#include <iostream>
#include <sys/socket.h>
#include <netinet/in.h>
#include <unistd.h>
#include <cstring>
#define PORT 8080
int main() {
    int sockfd;
    struct sockaddr_in serverAddr, clientAddr;
    char buffer[1024] = {0};
    socklen_t addrLen = sizeof(clientAddr);
    // 1. 创建 UDP Socket
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0) {
        perror("Socket 创建失败");
        exit(EXIT_FAILURE);
    }
    // 2. 绑定 IP 和端口
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    serverAddr.sin_port = htons(PORT);
    if (bind(sockfd, (struct sockaddr*)&serverAddr, sizeof(serverAddr)) < 0) {
        perror("绑定失败");
        exit(EXIT_FAILURE);
    }
    std::cout << "UDP 服务器已启动，等待消息..." << std::endl;
    // 3. 接收数据
    recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr*)&clientAddr, &addrLen);
    std::cout << "收到客户端消息: " << buffer << std::endl;
    // 4. 发送数据
    const char* response = "Hello from UDP server";
    sendto(sockfd, response, strlen(response), 0, (struct sockaddr*)&clientAddr, addrLen);
    // 5. 关闭 Socket
    close(sockfd);
    return 0;
}
~~~
### C++ 实现 UDP 客户端示例
~~~
#include <iostream>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <cstring>
#define PORT 8080
#define SERVER_IP "127.0.0.1"
int main() {
    int sockfd;
    struct sockaddr_in serverAddr;
    char buffer[1024] = {0};
    socklen_t addrLen = sizeof(serverAddr);
    // 1. 创建 UDP Socket
    sockfd = socket(AF_INET, SOCK_DGRAM, 0);
    if (sockfd < 0) {
        perror("Socket 创建失败");
        exit(EXIT_FAILURE);
    }
    // 2. 服务器地址配置
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(PORT);
    serverAddr.sin_addr.s_addr = inet_addr(SERVER_IP);
    // 3. 发送数据
    const char* message = "Hello from UDP client";
    sendto(sockfd, message, strlen(message), 0, (struct sockaddr*)&serverAddr, addrLen);
    // 4. 接收数据
    recvfrom(sockfd, buffer, sizeof(buffer), 0, (struct sockaddr*)&serverAddr, &addrLen);
    std::cout << "收到服务器消息: " << buffer << std::endl;
    // 5. 关闭 Socket
    close(sockfd);
    return 0;
}
~~~
> sendto()和recvfrom()与TCP的send()/recv()有何不同？:
>
> sendto() 和 recvfrom() 适用于 UDP，需要显式指定目标地址，数据传输是无连接、不可靠的;send() 和 recv()适用于 TCP，数据传输是可靠、有连接的，系统会自动管理连接和数据的可靠传输。
>
> UDP服务端是否需要调用listen()和accept()？为什么？:
>
> UDP 不需要调用 listen() 和 accept()，因为它是无连接的协议，没有明确的连接建立过程。UDP 服务端 通过 recvfrom() 接收来自客户端的数据，并通过 sendto() 发送响应数据，整个过程是无连接的，每个数据包独立处理。
> 
> 同步 Socket 和 异步 Socket 的核心区别？：
>
> 同步 和 异步 的核心区别主要体现在**数据发送与接收的方式**和**程序执行的阻塞性**。同步 Socket是阻塞式调用，网络 I/O 操作按顺序执行，必须等待当前操作完成才能继续；异步 Socket是非阻塞式调用，通常与 epoll、select、poll、IOCP（Windows） 等机制结合，实现高效的事件通知，适用于高并发、大规模连接的服务器，如 Web 服务器、IM、游戏服务器。
# 高级特性与性能优化
## I/O多路复用
I/O 多路复用 是一种技术，它允许程序在单个线程内同时处理多个 I/O 操作，而不需要为每个 I/O 操作创建一个独立的线程或进程。通过使用 I/O 多路复用，程序可以在一个线程中高效地等待多个 I/O 操作完成，然后在适当的时间处理这些操作。

I/O 多路复用的核心思想是通过使用操作系统提供的特定机制（如 select()、poll()、epoll() 等），使得一个线程能够“监视”多个文件描述符（如 Socket、文件等），并在其中的某一个或多个文件描述符准备好进行读写操作时进行处理。这避免了传统的做法中为每个 I/O 操作都创建一个线程，从而减少了线程切换的开销和资源消耗。
### select()——适用于小规模并发
早期的多路复用机制，用于监听多个文件描述符的变化。通过设置最大文件描述符数目和监听的文件描述符集合，采用轮询的方式检查它们的状态，select() 会阻塞直到有文件描述符准备好。存在**文件描述符数量限制**（通常是 1024）。
### poll()——适用于中等规模并发
与 select() 类似，但相较于 select()，poll() 不受文件描述符数量的限制。使用 pollfd 数组，可以动态地设置需要监视的文件描述符和事件。
### epoll()——Linux专有，适用于大规模高并发场景
epoll() 是 Linux 系统提供的高效 I/O 多路复用机制，主要用于处理大量并发连接。相比 select() 和 poll()，epoll() 采用**事件驱动（基于回调）** 机制，特别适用于大量并发连接的场景。通过**水平触发**和**边缘触发**两种模式，可以优化性能。
> 边缘触发（ET）和水平触发（LT）模式的区别？哪种模式需要非阻塞Socket？：
>
> LT 只要缓冲区仍有数据可读，就会持续触发，可以用阻塞或非阻塞，兼容性强、容易编程但性能稍低；ET 仅在状态发生变化（如新数据到达）时触发，必须使用非阻塞否则可能丢失事件，性能更高但容易丢失事件，必须合理使用非阻塞 IO。
## 高并发与性能
高并发服务器需要高效地管理大量客户端连接，避免线程和资源的过度消耗。为此，我们可以结合 **线程池、连接池、异步 I/O (如 IOCP, epoll, kqueue 等)** 来设计一个高性能的 Socket 服务器。
### 设计一个支持高并发的 Socket 服务器——基于 IOCP + 线程池
#### 主线程：
- 创建 监听 Socket，绑定端口。
- 使用 IOCP (Windows) / epoll (Linux) 进行高效事件监听。
- 接受新连接，并将其加入 IOCP 监听队列。
#### 工作线程池：
- 线程池中的线程监听 IO 事件（如 recv() 或 send()）。
- 当 I/O 事件发生时，IOCP/epoll 触发事件通知线程池，线程池执行相应的处理逻辑（如数据解析、数据库操作等）。
- 处理完成后，继续等待新的 I/O 事件。
#### 连接池：
- 维护已连接的客户端 Socket，避免重复创建/销毁。
- 采用**LRU** 或**定时检测** 关闭长期不活动的连接，节约资源。
~~~
#include <iostream>
#include <winsock2.h>
#include <windows.h>
#include <mswsock.h>
#include <vector>
#pragma comment(lib, "ws2_32.lib")
#define PORT 8080
#define MAX_THREADS 4  // 线程池大小
// 存储每个 Socket 关联的 IO 数据
struct PerIoData {
    OVERLAPPED overlapped;
    SOCKET clientSocket;
    char buffer[1024];
    WSABUF wsabuf;
    DWORD bytesReceived;
};
// 线程池工作函数
DWORD WINAPI WorkerThread(LPVOID lpParam) {
    HANDLE iocp = (HANDLE)lpParam;
    DWORD bytesTransferred;
    ULONG_PTR completionKey;
    PerIoData* ioData;
    while (true) {
        // 从 IOCP 获取完成的 I/O 事件
        BOOL result = GetQueuedCompletionStatus(iocp, &bytesTransferred, &completionKey, (LPOVERLAPPED*)&ioData, INFINITE);
        if (!result || bytesTransferred == 0) {
            // 连接关闭
            closesocket(ioData->clientSocket);
            delete ioData;
            continue;
        }
        // 处理接收到的数据
        std::cout << "Received: " << ioData->buffer << std::endl;

        // 继续监听下一个数据请求
        ZeroMemory(&ioData->overlapped, sizeof(OVERLAPPED));
        ioData->wsabuf.buf = ioData->buffer;
        ioData->wsabuf.len = sizeof(ioData->buffer);
        WSARecv(ioData->clientSocket, &ioData->wsabuf, 1, &ioData->bytesReceived, 0, &ioData->overlapped, NULL);
    }
}
int main() {
    // 初始化 Winsock
    WSADATA wsaData;
    WSAStartup(MAKEWORD(2, 2), &wsaData);
    // 创建监听 Socket
    SOCKET listenSocket = WSASocket(AF_INET, SOCK_STREAM, 0, NULL, 0, WSA_FLAG_OVERLAPPED);
    sockaddr_in serverAddr;
    serverAddr.sin_family = AF_INET;
    serverAddr.sin_port = htons(PORT);
    serverAddr.sin_addr.s_addr = INADDR_ANY;
    bind(listenSocket, (sockaddr*)&serverAddr, sizeof(serverAddr));
    listen(listenSocket, SOMAXCONN);
    // 创建 IOCP 端口
    HANDLE iocp = CreateIoCompletionPort(INVALID_HANDLE_VALUE, NULL, 0, 0);
    // 创建线程池
    std::vector<HANDLE> threads;
    for (int i = 0; i < MAX_THREADS; ++i) {
        HANDLE thread = CreateThread(NULL, 0, WorkerThread, iocp, 0, NULL);
        threads.push_back(thread);
    }
    std::cout << "Server listening on port " << PORT << "..." << std::endl;
    while (true) {
        // 接受新的客户端连接
        sockaddr_in clientAddr;
        int addrLen = sizeof(clientAddr);
        SOCKET clientSocket = accept(listenSocket, (sockaddr*)&clientAddr, &addrLen);
        CreateIoCompletionPort((HANDLE)clientSocket, iocp, (ULONG_PTR)clientSocket, 0);
        // 绑定 I/O 事件
        PerIoData* ioData = new PerIoData;
        ZeroMemory(&ioData->overlapped, sizeof(OVERLAPPED));
        ioData->clientSocket = clientSocket;
        ioData->wsabuf.buf = ioData->buffer;
        ioData->wsabuf.len = sizeof(ioData->buffer);
        DWORD flags = 0;
        WSARecv(clientSocket, &ioData->wsabuf, 1, &ioData->bytesReceived, &flags, &ioData->overlapped, NULL);
    }
    // 关闭服务器
    closesocket(listenSocket);
    WSACleanup();
    return 0;
}
~~~
## 超时与异常处理
在网络编程中，为了避免 send() 或 recv() 因为网络阻塞而无限等待，我们可以使用 setsockopt() 设置**发送超时** 和**接收超时**。
setsockopt() 用于配置 Socket 选项，其中包括超时设置：
~~~
int setsockopt(SOCKET s, int level, int optname, const char *optval, int optlen);
~~~
Windows 传递 int 毫秒值，Linux 需要 struct timeval（秒 + 微秒）。
> 如果客户端突然断开，服务端如何检测到？：
>
> 1.调用 recv() 检测返回值：只能在服务器尝试接收数据时检测，不会主动发现客户端断连。2.开启 SO_KEEPALIVE 选项（TCP KeepAlive 机制）：TCP 提供的一种**底层心跳机制**，可以让系统定期发送 KeepAlive 探测包，如果在一定时间内没有收到客户端的响应，则认为连接已断开，不需要应用层额外处理。3.主动心跳检测（心跳包机制）：服务器和客户端可以定期发送心跳包（应用层心跳），确保连接正常。4. 结合 poll() / select() / epoll() 监听断连：在 Linux 服务器上，可以使用 poll() / select() / epoll() 监听Socket 的可读性，如果 recv() 返回 0，则说明客户端断开。
# 扩展知识
## 协议序列化
协议序列化是指**将数据结构或对象转换为字节流**，以便在网络传输或存储后能够准确地还原成原始数据的过程。常见的序列化协议有：
- Protobuf（Protocol Buffers）：Google 开发的高效二进制序列化协议。
- JSON（JavaScript Object Notation）：可读性强的文本格式，但效率较低。
- XML（eXtensible Markup Language）：结构化数据格式，可读性高但冗余较多。
- MessagePack：类似 JSON，但更加紧凑的二进制格式。
  
在 Socket 通信中，服务器和客户端之间需要交换数据，Protobuf 主要用于数据的高效编码和解码。其作用包括：
-减少数据大小：Protobuf 采用二进制格式，比 JSON 或 XML 更紧凑，能够显著减少数据传输量。
- 提升解析效率：相比 JSON 解析字符串，Protobuf 解析二进制数据更快，减少 CPU 消耗。
- 跨平台兼容：Protobuf 生成的代码可以在不同语言（C++、Python、Java 等）中使用，保证数据格式一致。
- 结构化数据传输：Protobuf 提供了一种强类型的数据格式，避免了传统 char[] 或 std::string 传输带来的格式解析问题。
