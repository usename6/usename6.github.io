---
title: C++ 网络编程（一） --- 初识socket
date: 2020-04-21 14:07:43
tags: [C++,网络编程]
categories: [C++,网络编程]
---
这周组会真的是灾难，还得要优化代码，真是难受。。。
<!--more-->
一、背景
1.1 OSI、TCP/IP网络模型
![alt](/images/winsock-1/figure1.png)
越高层服务对象越高，低级服务交给底层去完成，这样每一层只需要关注自己即可。
1.2 网络编程位置
![alt](/images/winsock-1/figure2.png)
可以看到Socket网络编程主要是为传输层和网络层服务。
传输层：端到端的传输方式。（即主机应用程序之间的传输方式）。协议主要有面向连接的TCP和面向无连接的UDP协议。
网络层：点到点的通信，通过路由选择算法，选择合适路径，经由多个路由器之后，传输到目的节点的网络上。再由数据链路层和物理层传递到对应的主机（MAC地址）上。
也就是说SOCKET编程能够实现进程和进程之间面向连接和面向无连接的功能。
![alt](/images/winsock-1/figure3.png)
二、什么是Socket
2.1 Socket的概念
首先先得说明一个故事，简单说明网络通信的过程。
假设有一个家庭A，成员有小明、小花、小聪。每个人都有一个自己发送和接收邮箱。每个人只能读取自己邮箱的文件。
家庭B，成员有小哥、小弟两人，同样每个人也有一个自己的发送和接收邮箱。
家庭A位于H区。家庭B位于S区。
如果小明想和小哥通信。那会经历以下几个过程。
a.首先将写好的信件，放入自己发送邮箱中。
b.然后通知快递员来拿，快递员先将邮件传到H区的传输中心。
c.接着再由汽车，经由Q区、W区、I区（H区与S区的最短路径）传输中心，达到S区传输中心。
d.接下来再通知S区快递员，让S区快递员将邮件送入到家庭B对应小哥的接收邮箱中。
这实际上类似于网络通信的过程。
家庭A，家庭B ----> 主机。
小明，小花，小聪，小哥，小弟 ----> 进程。
发送邮箱，接收邮箱 -----> 端口。
H区、Q区、W区、I区、S区传输中心 ---> 网关和路由器。
汽车 ---> 网络层的传输的过程。
快递小哥 ----> 数据链路层，物理层传输到网关的过程。
从网络通信的角度理解，这里socket指的是端口，即发送邮箱和接收邮箱，进程只需要把数据交付给邮箱，他可以选择如何交付，比如说加急交付等。这是他可以操作的过程。但是传输的过程并不是由他控制。而是由快递员和汽车（操作系统、线路等等控制）。
简单的说，socket通过将IP地址和端口号绑定起来，用来唯一表示主机中的进程通信的端口。是应用层和传输层之间的端口（API）。
从操作系统角度理解，socket指的是操作系统类似文件的一种资源。用句柄表示。
2.2 socket的好处
1）方便调试。（只需要关注进程，传输交由操作系统）
2）方便编写代码。（只需要关注进程，传输交由操作系统）
3）屏蔽了底层复杂的网络操作。
2.3 socket的种类
一共分为三种：字节流套接字（TCP）、数据报套接字（UDP）、原始套接字（IP）。
三、初识winsocket
3.1 认识头文件 ------ winsock.h
导入投文件
{% codeblock lang:C %}
#include <winsock.h>
{% endcodeblock %}
注意：在Visual studio中需要导入
{% codeblock lang:C %}
#define _WINSOCK_DEPRECATED_NO_WARNINGS 1
#pragma commet(lib, "Ws2_32.lib")
{% endcodeblock %}
第二个目的可以简单解释下：主要和程序的编译运行相关。
.h 文件指明了函数接口在哪？ --- 编译需要用
.lib 文件指明了链接器调用的函数在哪个dll中 --- 链接时需要用
.dll 可执行代码 --- 运行时用。
在程序中必须依次指定这三个文件的路径，否则无法正确执行。
3.2 初始化winsocket
WSAStartup (WORD wVersionRequested,LPWSADATA lpWSAData)
(1)wVersionRequested: 高阶字节指定小版本（修订本）号，低位字节指定主版本号。这里的版本号指的都是socket的版本号。
Eg.WORD sockVersion = MAKEWORD(minorVer, majorVer); //其中minorVer = majorVer = 2，表示采用winsock2的API。
(2)IpWSAData指定WSADATA数据结构的指针，用来接收Window sockets实现的细节（即dll库中的细节）。在WSAStartup调用后将对WSAdata进行一次初始化。
WSADATA的结构具体如下：
{% codeblock lang:C %}
typedef struct WSAData {
   WORD	wVersion;   	// 库文件建议应用程序使用的版本
   WORD	wHighVersion;   	 // 库文件支持的最高版本
   char	szDescription[WSADESCRIPTION_LEN+1];  // 库描述字符串
   char	szSystemStatus[WSASYS_STATUS_LEN+1];  // 系统状态字符串
   unsigned short	iMaxSockets;       // 同时支持的最大套接字的数量
   unsigned short	iMaxUdpDg;       // 2.0版中已废弃的参数
   char  *		lpVendorInfo;       // 2.0版中已废弃的参数
} WSADATA,  * LPWSADATA;
{% endcodeblock %}
(3)返回值：整型，成功返回0.不成功则返回以下错误码：
(这里注意：不能使用GetLastError()得到错误的信息。)
a.WSASYSNOTREADY: 当前网络还没有准备好进行通信
b.WSAVERNOTSUPPORTED：套接字无法支持此应用程序。
c.WSAEPROCLIM：任务数超过winsock的支持上限。（这块怀疑是和iMaxSockets有关系）。
d.WSAEFAULT：WSAdata是一个非法的指针。
(4)函数的功能：此函数被调用的时，可以通过所设置的环境变量找到winsock.dll文件;检查winsock.dll的版本号是否符合要求，符合则返回0（盲猜不符合返回WSAVERNOTSUPPORTED）;实现winsock和应用程序绑定（这块可以理解为编译链接运行的过程）。
3.3 终止Winsock
int WSACleanup(void)函数
(1)参数是void,这点和WSAStartup不同
(2)返回值：整型，0表示成功失败返回SOCKETERROR，注意这里可以GetLastError()得到错误信息（这是与WSAStartup的第二个不同点）。
(3)函数的功能：中止对WinSock.dll的调用，释放引用动态链接库时占用的系统资源。
3.4 创建套接字
{% codeblock lang:C %}
int socket(int domain, int type, int protocol); 
{% endcodeblock %}
domain：表示的地址族，WinSock可支持AF_INET（IPV4）。
type：用以指定套接字的类型，刚刚也提到过了，一般包括三种类型：
a.SOCK_STREAM
流套接字，为TCP提供有连接的可靠传输。
b.SOCK_DGRAM
数据报套接字，使用UDP提供无连接的不可靠传输。
c.SOCK_RAW
原始套接字，提供底层IP的接口。
protocol:指定使用的协议，如IPPROTO_TCP，值得注意的是当type指为SOCK_STREAM和SOCK_DGRAM时的时候，**实际上protocol也被指定了，此时我们可以将值设置为0**。
返回值：函数执行失败返回INVALID_SOCKET（即-1），可以通过调用WSAGetLastError取得错误代码。
**值得注意的是：SOCK类型是一个整型描述符（句柄），是一个指向内部数据结构的指针，我们在后期调用的时候导入这个整型描述符，相当于控制和操作这个socket**
3.5 关闭套接字
{% codeblock lang:C %}
int closesocket(SOCKET s); 	// 参数是要关闭的套接字的句柄 
{% endcodeblock %}
3.6 绑定套接字到指定IP地址
首先要明白Winsock的寻址方式是怎么样的，由于要兼容多个协议，所以我们需要设计一个通用的数据结构存储这个地址。这个通用的数据结构就是sockaddr。
{% codeblock lang:C %}
struct sockaddr {
	u_short sa_family;  //地址家族，AF_xx
	char	sa_data[14];  //14个字节的协议地址
};
{% endcodeblock %}
在介绍下sockaddr_in。
{% codeblock lang:C %}
struct sockaddr_in {
	short		sin_family;  //地址家族，eg.AF_INET,AF_INET6
	u_short 	sin_port;    //端口号
	struct in_addr 	sin_addr; //用于存储IP地址，这个等下详细介绍下
	char		sin_zero[8];  //填充字节使得sockaddr_in和sockaddr保持大小。
};
{% endcodeblock %}
注意到这里还有一个in_addr的结构体，主要是用于存储IP地址，其结构如下：
{% codeblock lang:C %}
struct in_addr {
	union {
		struct { u_char s_b1,s_b2,s_b3,s_b4; } S_un_b;
		struct { u_short s_w1,s_w2; } S_un_w;
		u_long S_addr;
	} S_un;
#define s_addr  S_un.S_addr     /* can be used for most tcp & ip code */
#define s_host  S_un.S_un_b.s_b2		/* host on imp */
#define s_net   S_un.S_un_b.s_b1		 /* network */
#define s_imp   S_un.S_un_w.s_w2		/* imp */
#define s_impno S_un.S_un_b.s_b4		/* imp # */
#define s_lh    S_un.S_un_b.s_b3		 /* logical host */
};
{% endcodeblock %}
难点： 1）区分sockaddr、sockaddr_in、in_addr
一般来说sockaddr和sockaddr_in的包含数据内容是一致，但是能够操作这两个结构体的对象不一样，sockaddr一般是在accept,getsockname之类函数中使用，而sockaddr_in则是用户进行操作和修改。一般来说我们需要先对sockaddr_in赋值，在使用函数的时候可能需要将sockaddr_in强制转化为sockaddr,以便于使用。**这里我的理解是，填充的时候要按照具体规则来，而调用的函数的时候，不同的协议的处理方式可能具有相似性，如果重写函数的话，似乎太浪费空间和笔墨，所以统一用sockaddr作为接口，处理的时候，我们需要将sockaddr_in转化为sockaddr进行处理。**
而in_addr则表示sockaddr中地址参数。三者关系如下图所示：
![alt](/images/winsock-1/figure4.png)
2）如何给in_addr赋值
首先要地址是以什么样的顺序存入数据结构中。TCP/IP统一规定使用大端传输数据，也称为网络字节顺序。**上述sockaddr和sockaddr_in除了sin_family（不是协议的一部分），其他所有值都要按网络字节顺序存储。**
注：大端存储是什么，就是将高位数据存储到低位空间中。
eg.
{% codeblock lang:C %}
servAddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1"); 
sin.sin_addr.S_un.S_addr = htonl(INADDR_ANY);//(ULONG)0x00000000
{% endcodeblock %}
简单介绍以上两种赋值方式：
inet_addr：可以将点分十进制的字符串转换为为一个长整型的数。
htonl: 可以将32位字节顺序转化为网络的字节顺序，刚刚也提到了除了sin_family，其他的值全部要按照网络的字节顺序来存储。
附：INADDR_ANY表示转换过来就是0.0.0.0，泛指本机的意思，也就是表示本机的所有IP，因为有些机子不止一块网卡，多网卡的情况下，这个就表示所有网卡ip地址的意思。一般服务器会使用INADDR_ANY。
其他函数还有htons、htonl、ntohs、ntohl等。不再赘述。（太懒了，逃
回到如何将套接字绑定到指定网络地址上，一般我们是用connect()或者listen()函数调用。值得注意的是在服务端，用于监听客户端连接请求套接字一定要绑定，而客户端不一定需要绑定。
{% codeblock lang:C %}
int bind (
    SOCKET s,        //socket的句柄
    const struct sockaddr *  name,  //网络地址，之前介绍那些，不过我们只能对sockaddr_in赋值，如果使用需要强制转化sockaddr_in变成sockaddr处理
    int namelen                     //sockaddr的长度，直接sizeof(name)就ojbk
);
{% endcodeblock %}
返回值：成功为0，否则是SOCKET_ERROR。
3.7 设置套接字进入监听状态
条件：字节流套接字，服务端，是一种被动连接模式。
{% codeblock lang:C %}
int listen（
	SOCKET s,     /*已绑定的套接字句柄*/
	int backlog  /*等待连接队列长度*/
)
{% endcodeblock %}
3.8 接收连接请求
从监听套接字的等待队列中抽取连接请求，建立新的套接字与该客户端的套接字建立连接，交换数据。
{% codeblock lang:C %}
SOCKET accept(
SOCKET s,                   //服务端处的监听套接字
struct sockaddr* addr,  //出口参数，客户端地址结构
int* addrlen                   //出口参数，客户端地址长度
);
{% endcodeblock %}
返回值：如果连接成功返回新的套接字描述符，否则返回INVALID_SOCKET。
3.9 请求连接函数
客户端请求与服务端建立连接。
{% codeblock lang:C %}
int connect (
    SOCKET s,   //客户端的套接字
    const struct sockaddr *  name, //客户端的地址结构
    int namelen    //客户端的sockaddr长度
);
{% endcodeblock %}
返回值：成功返回0，否则返回SOCKET_ERROR。
根据3.7 ~ 3.9节内容，我们可以对客户端的状态进行总结。
when 客户端发出connect（请求连接） ---> 服务端则会监听到该连接，将会采取三个动作： 1）TCP三次握手成功后，调用accept函数，然后讲该连接丢入已连接队列中。 2）将其丢入到等待连接队列中。 3）将其扔出去。
（具体三次握手下细化的状态转移问题，我想等学习了三次握手再说，先甩一个感觉不错链接：
 https://blog.csdn.net/kongxian2007/article/details/49153801
3.10 收发数据
{% codeblock lang:C %}
int send (
    SOCKET s,   //若是服务器，则该选项是accpet建立对应的套接字。客户端则为建立连接的套接字
    const char  * buf, //发送数据的缓冲区
    int len,         //缓冲区中的数据长度
    int flags        //调用执行方式，一般置为0
);
{% endcodeblock %}
返回值
成功：返回实际发送的字节数
失败：返回SOCKET_ERROR
**请注意send的成功调用并不代表数据发送成功，仅仅表示能成功送入缓冲区**
{% codeblock lang:C %}
int recv (
    SOCKET s,   //建立连接套接字
    char * buf, //接收缓冲区
    int len,    //缓冲区长度
    int flags   //选项，一般为0
);
{% endcodeblock %}
返回值：
成功：返回实际接收的字节数
连接已终止：0
失败：返回SOCKET_ERROR
下面的代码很有可能会出现一个问题，如果是循环接收服务端发送的数据，如果接收到的是这样数据：
aaaa
aaa
bbbb
实际上客户端接收的是：
aaaa
aaa
bbb
最后一个数据少了b，这是因为第三个参数在每一次调用前都在发生变化，从4到3，所以导致第三次接收缓冲区只能接收”bbb"。
{% codeblock lang:C %}
int nRecv = recv(s, buff, sizeof(buf), 0);
{% endcodeblock %}
正确的写法可以是这样：定义一个常数即可。
{% codeblock lang:C %}
int nRecv = recv(s, buff, 256, 0);
{% endcodeblock %}

实验：
{% codeblock lang:C %}
//////////////////////////////////////////////////
// TCPServer.cpp文件
#define _WINSOCK_DEPRECATED_NO_WARNINGS 1
#include <WinSock2.h>
#pragma commet(lib, "Ws2_32.lib")
#include "../common/InitSock.h"
#include <stdio.h>
#include <iostream>
CInitSock initSock;		// 初始化Winsock库
sockaddr_in server, client;
using namespace std;
int main()
{
	WORD ws_version = MAKEWORD(2, 2);
	WSADATA wsaData;
	WSAStartup(ws_version,&wsaData);
	// 创建套节字
	SOCKET sListen = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (sListen == INVALID_SOCKET)
	{
		printf("Failed socket() \n");
		return 0;
	}

	// 填充sockaddr_in结构
	sockaddr_in sin;
	sin.sin_family = AF_INET;
	sin.sin_port = htons(4567);
	sin.sin_addr.S_un.S_addr = htonl(INADDR_ANY);//(ULONG)0x00000000

	//output 1 Create
	int server_len = sizeof(server);
	getsockname(sListen, (sockaddr *)&server, &server_len);
	cout << "the Create Stage of Socket IP:" << inet_ntoa(server.sin_addr) << ":" << ntohs(server.sin_port) << endl;


	// 绑定这个套节字到一个本地地址
	if (bind(sListen, (LPSOCKADDR)&sin, sizeof(sin)) == SOCKET_ERROR)
	{
		printf("Failed bind() \n");
		return 0;
	}
	//output 2 Bind
	memset(&server, 0, sizeof(server));
	server_len = sizeof(server);
	getsockname(sListen, (sockaddr *)&server, &server_len);
	cout << "the Bind Stage of Socket IP:" << inet_ntoa(server.sin_addr) << ":" << ntohs(server.sin_port) << endl;


	// 进入监听模式
	if (listen(sListen, 2) == SOCKET_ERROR)
	{
		printf("Failed listen() \n");
		return 0;
	}
	//output 3 listen
	memset(&server, 0, sizeof(server));
	server_len = sizeof(server);
	getsockname(sListen, (sockaddr *)&server, &server_len);
	cout << "the Listen Stage of Socket IP:" << inet_ntoa(server.sin_addr) << ":" << ntohs(server.sin_port) << endl;


	// 循环接受客户的连接请求
	sockaddr_in remoteAddr;
	int nAddrLen = sizeof(remoteAddr);
	SOCKET sClient;
	char szText[] = " TCP Server Demo! \r\n";
	while (TRUE)
	{
		// 接受一个新连接
		sClient = accept(sListen, (SOCKADDR*)&remoteAddr, &nAddrLen);
		if (sClient == INVALID_SOCKET)
		{
			printf("Failed accept()");
			continue;
		}
		memset(&server, 0, sizeof(server));
		server_len = sizeof(server);
		getsockname(sListen, (sockaddr *)&server, &server_len);
		cout << "the accept Stage of Socket IP:" << endl;
		cout << "Server IP:" << inet_ntoa(server.sin_addr) << ":" << ntohs(server.sin_port) << endl;

		memset(&client, 0, sizeof(client));
		int client_len = sizeof(client);
		getsockname(sClient, (sockaddr *)&client, &client_len);
		cout << "Client IP:" << inet_ntoa(client.sin_addr) << ":" << ntohs(client.sin_port) << endl;

		printf(" 接受到一个连接：%s \r\n", inet_ntoa(remoteAddr.sin_addr));

		// 向客户端发送数据
		send(sClient, szText, strlen(szText), 0);
		// 关闭同客户端的连接
		closesocket(sClient);
	}

	// 关闭监听套节字
	closesocket(sListen);
	WSACleanup();
	return 0;
}
{% endcodeblock %}

{% codeblock lang:C %}
//////////////////////////////////////////////////////////
// TCPClient.cpp文件
#define _WINSOCK_DEPRECATED_NO_WARNINGS 1
#include <WinSock2.h>
#pragma commet(lib, "Ws2_32.lib")
#include "../common/InitSock.h"
#include <stdio.h>
#include <iostream>
CInitSock initSock;		// 初始化Winsock库
sockaddr_in client;
using namespace std;
int main()
{
	WORD ws_version = MAKEWORD(2, 2);
	WSADATA wsaData;
	WSAStartup(ws_version, &wsaData);
	// 创建套节字
	SOCKET s = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP);
	if (s == INVALID_SOCKET)
	{
		printf(" Failed socket() \n");
		return 0;
	}

	// 也可以在这里调用bind函数绑定一个本地地址
	// 否则系统将会自动安排

	// 填写远程地址信息
	sockaddr_in servAddr;
	servAddr.sin_family = AF_INET;
	servAddr.sin_port = htons(4567);
	// 注意，这里要填写服务器程序（TCPServer程序）所在机器的IP地址
	// 如果你的计算机没有联网，直接使用127.0.0.1即可
	servAddr.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	int client_len = sizeof(client);
	getsockname(s, (sockaddr *)&client, &client_len);
	cout << "the Create Stage of Socket IP:" << inet_ntoa(client.sin_addr) << ":" << ntohs(client.sin_port) << endl;

	if (connect(s, (sockaddr*)&servAddr, sizeof(servAddr)) == -1)
	{
		printf(" Failed connect() \n");
		return 0;
	}
	memset(&client, 0, sizeof(client));
	client_len = sizeof(client);
	getsockname(s, (sockaddr *)&client, &client_len);
	cout << "the Create Stage of Socket IP:" << inet_ntoa(client.sin_addr) << ":" << ntohs(client.sin_port) << endl;
	// 接收数据
	char buff[256];
	int nRecv = recv(s, buff, 256, 0);
	if (nRecv > 0)
	{
		buff[nRecv] = '\0';
		printf(" 接收到数据：%s", buff);
	}

	// 关闭套节字
	closesocket(s);
	WSACleanup();
	system("pause");
	return 0;
}
{% endcodeblock %}