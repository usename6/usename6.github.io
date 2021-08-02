---
title: C ++ 多线程编程1 ——— 创建线程
date: 2020-03-26 22:20:05
tags: [C++,多线程]
categories: [C++,多线程]
---
学习多线程，紧跟脚步，不做眼高手低，稳扎稳打
<!--more-->
1）HANDLE的定义
{% codeblock lang:C %}
typedef void *HANDLE;
{% endcodeblock %}
从中我们可以发现HANDLE的定义，它实际上表示一个void型的指针，使用这样的指针有个好处在于：**void型指针可以强制转化为任意类型的指针**，因此我们可以用他指向任意的数据结构。
那么我们就会思考HANDLE的作用是什么呢？这里参考了下stackoverflow的某位大佬（Lawrence Dol）的话：
来源：https://stackoverflow.com/questions/902967/what-is-a-windows-handle
![alt](/images/多线程/HANDLE.png)
a.句柄的重定向
    如下图所示，当我们想访问一个进程里面的对象的时候，我们先访问HANDLE得到一个这个进程的对象表(即指向这个进所有对象的内存空间的指针数组),内核通过处理这个对象表的元素，完成对进程的控制。比如说重定向，因为大多数操作系统采用了虚拟内存，那这样的话一个进程所有对象的内存地址并不是固定的。所以操作系统可以利用这样特效，更改这个对象表的指针指向，完成重定向，实现虚拟内存实现。
b.隐藏某些信息
    从(a)我们可以知道，句柄可以帮助操作系统实现虚拟内存，重定向的功能。但是仔细思考，要想实现虚拟内存，我们直接访问这个进程的对象表不行吗，为什么一定通过一个指针去访问一个对象表。这就涉及到操作系统的安全的内容。我们知道操作系统的调度是受到保护的，如果用户对其进行修改，会不会破坏原有操作系统的调度，可能会导致操作系统崩溃。于是我们需要设一个HANDLE，指向这段需要保护的空间，这样用户就没有办法对其进行直接修改。看似这是一个笨操作，事实上是为了保证操作系统的安全性。
    还有一个好处，从API开发者角度考虑这个问题，如果我们设计了HANDLE指针当做这个API的返回值，这样用户就不知道我们实际返回值的定义是什么，就没办法得知我们这个结构体或者类的定义是什么？就没办法对这个结构体进行操作，比如取数据（都不知道这个结构体里面的成员变量有哪些，怎么提取？）。这是一个很好的安全措施，给一个例子说明：
    下图是一个关于Widget这个结构体的一个函数
    {% codeblock lang:C %}
    Widget * GetWidget (std::string name)
    {
        Widget *w;
        w = findWidget(name);
        return w;
    }
    {% endcodeblock %}
    上述函数的返回值为Widget *,当我们在main函数调用这个GetWidget的时候，我们首先要定义Widget的指针这个变量，这样就相当于暴露了这个结构体名字，用户可以通过这个名字查询查询到Widget的成员变量。
    但是如果我们这样定义：
    {% codeblock lang:C %}
    typedef void *HANDLE
    HANDLE GetWidget (std::string name)
    {
        Widget *w;
        w = findWidget(name);
        return reinterpret_cast<HANDLE>(w);
    }
    {% endcodeblock %}
    那这样的话，我们在主函数调用GetWidget的时候，只需要定义HANDLE这个变量即可，这样用户只知道这个返回值返回的是HANDLE，即使暴露这个名字，也只能找到HANDLE的定义，无法确认这个HANDLE内部的实际意义是什么。当然我一直有个疑惑在于如果我知道了GetWidget函数不就可以知道实际返回的Widget，那这样不缺失了保护的意义？难道是它把这个GetWidget藏的很深？不让用户看见？
2）如何创建线程
当我们学会了HANDLE作用，了解了在Window操作系统中，是怎么样利用HANDLE实现对进程、资源等的控制（重定向，虚拟内存管理），怎么样保护Window的内核资源不被随意更改。那么我们就要了解在用户层面上是如何创建线程的。
{% codeblock lang:C %}
HANDLE CreateThread(
    LPSECURITY_ATTRIBUTES lpThreadAttributes,
    SIZE_T dwStackSize,
    LPTHREAD_START_ROUTINE lpStartAddress,
    LPVOID lpParameter,
    DWORD dwCreationFlags,
    LPDWORD lpThreadId
)
{% endcodeblock %}
下面简单介绍几个参数：
a.lpThreadAttributes
    LPSECURITY_ATTRIBUTES lpThreadAttributes,表示指向SECURITY_ATTRIBUTES型态的指针。设置为NULL表示默认安全性，此时不可被子线程继承。给出其定义如下：
    {% codeblock lang:C %}
    typedef struct _SECURITY_ATTRIBUTES {
        DWORD  nLength;         //nLength表示，这个结构所占的空间是多少
        LPVOID lpSecurityDescriptor; //安全描述符，可以控制用户对线程的访问，如果该值为NULL，则表示用户对这个进程访问权限是默认的。
        BOOL   bInheritHandle;      //该控制属性能否被子进程继承。
    } SECURITY_ATTRIBUTES, *PSECURITY_ATTRIBUTES, *LPSECURITY_ATTRIBUTES;
    {% endcodeblock %}
    因此，由上述定义可知，如果我们想创建子线程也继承父进程下用户访问权限，那么我们不能设置为NULL，而是设置一个新的LPSECURITY_ATTRIBUTES，并设置其中的成员变量bInheritHandle为True即可。
b.dwStackSize
    线程堆栈的初始化大小，由用户设定。一个线程所有函数都要依赖于这个栈，比如参数的存储，使用。如果设置为0，那么将线程堆栈设置为默认大小（1MB）。
c.lpStartAddress
    指向线程函数的指针。这里允许多个线程指向同一个函数。（PS:可以实验下）。
d.lpParameter
    传给多线程函数的参数。LPVOID是一个空指针类型，可以对在线程函数进行内部强制转化转化到你想要类型，然后再执行。最常见的可以为这个线程函数进行一个结构体，然后我们将这个结构体指针传进去。如果只有一个参数可以传递一个对象的指针即可。
e.dwCreationFlags
    控制线程的创建方式，这里给出了两种创建方式：
    1.CREATE_SUSPENDED，这个进程会以挂起（suspended）状态创建，并且进程函数不会被执行，除非ResumeThread这个函数被调用。
    2.STACK_SIZE_PARAM_IS_A_RESERVATION，为这个线程提供了设置预留栈，这个参数和前面dwStackSize有关，如果这个标志未指定的话，dwStackSize应该会设置为预留栈的大小，如果为指定，dwStackSize将指定为默认的值。
    3.0表示创建后立即激活。
    PS：可以分别实验下上面三种创建方式。
f.lpThreadId
    返回新线程的ID，如果设置为NULL，则不返回新线程的ID。

实践：
创建一个含参数的新线程：
{% codeblock lang:C %}
#include <windows.h>
#include <cstdio>
#include <cmath>
#include <iostream>
#define sq(x) (x)*(x)
struct Point
{ 
	int x, y; 
	Point(){}
	Point(int _x, int _y) :x(_x), y(_y){}
};
struct Input{ Point p1, p2; };
inline double get_dist(Point p, Point q)
{
	return sqrt(sq(p.x - q.x) + sq(p.y - q.y));
}
DWORD WINAPI Fun(LPVOID lpParamter)
{
	Input *cur = (Input *)lpParamter;
	Point p = cur->p1;
	Point q = cur->p2;
	std::cout << get_dist(p, q) << "\n";
	return 0;
}
int main()
{
	Input in;
	in.p1 = Point(1, 1);
	in.p2 = Point(2, 2);
	HANDLE hthread = CreateThread(NULL, 0, Fun, (LPVOID)(&in), 0, NULL);
	system("pause");
	return 0;
}
{% endcodeblock %}

栈空间这块，不知道为什么是随机的数值，这里不清楚变量是不是顺序存储在内存里面，好像跟我想的不一样，先给代码吧，以后再考虑。。。
{% codeblock lang:C %}
#define _CRT_SECURE_NO_WARNINGS
#include <windows.h>
#include <cstdio>
#include <cmath>
#include <iostream>
#define sq(x) (x)*(x)
DWORD WINAPI Test(PVOID lpParamter)
{
	DWORD dwRet = 0;
	printf("%-3d:0x%x\n", lpParamter, &dwRet);
	return dwRet;
}
int main()
{
	DWORD dwTid;
	printf("Main:0x%x\n", &dwTid);
	for (int i = 0; i < 2;i++)
		CreateThread(NULL, 4, Test, (PVOID)(i), STACK_SIZE_PARAM_IS_A_RESERVATION, NULL);
	Sleep(6000);
	return 0;
}
{% endcodeblock %}

3）CloseHandle函数
销毁线程用void CloseHandle(HANDLE thread);

4)GetCurrentThreadId函数
得到当前线程的PID，int GetCurrentThreadId().

以上是多线程的基本内容，下一节介绍window.h的临界区，并尝试基本的生产者消费者模型的编程。