---
title: Open GL 基本函数一
date: 2018-10-27 20:51:38
tags: [OpenGL]
categories: [OpenGL]
---
Open-GL 基本函数一
转战图形学（其实为了应付考试。。。，再不复习就挂科了。。。）
<!--more-->
{% codeblock lang:C %}
void glutInit(int* argc,char** argv) 
{% endcodeblock %}
这个函数用来初始化GLUT，此时main函数也有对应的参数：
{% codeblock lang:C %}
int main(int argc,char* argv[])
{% endcodeblock %}
glutInit是从main函数获取参数

{% codeblock lang:C %}
void glutInitWindowSize(int width,int height)
{% endcodeblock %}
该函数用于初始化窗口的大小

{% codeblock lang:C %}
void glutInitWindowPosition(int x,int y)
{% endcodeblock %}
该函数用来初始化窗口左上角是什么，以像素为单位

{% codeblock lang:C %}
void glutInitDisplayMode(unsigned int mode)
{% endcodeblock %}
显示模式，可供选择的参数：
1.GLUT_RGBA：当未指明GLUT-RGBA或GLUT-INDEX时，是默认使用的模式。表明欲建立RGBA模式的窗口。
2.GLUT_RGB：与GLUT-RGBA作用相同。
3.GLUT_INDEX：指明为颜色索引模式。
4.GLUT_SINGLE：只使用单缓存
5.GLUT_DOUBLE：使用双缓存。以避免把计算机作图的过程都表现出来，或者为了平滑地实现动画。
6.GLUT_ACCUM：让窗口使用累加的缓存。
7.GLUT_ALPHA：让颜色缓冲区使用alpha组件。
8.GLUT_DEPTH：使用深度缓存。
9.GLUT_STENCIL：使用模板缓存。
10.GLUT_MULTISAMPLE：让窗口支持多例程。
11.GLUT_STEREO：使窗口支持立体。
12.GLUT_LUMINACE:luminance是亮度的意思。但是很遗憾，在多数OpenGL平台上，不被支持。

{% codeblock lang:C %}
glColor3f(float, float, float)
{% endcodeblock %}
好吧，他有三个浮点参数，但是不清楚是什么（其实是懒得翻他headfile了）
其中0.0表示不使用颜色成分，1.0表示使用颜色最大值
具体参数对应的颜色如下
{% codeblock lang:C %}
glColor3f(0.0, 0.0, 0.0);  --> 黑色  
glColor3f(1.0, 0.0, 0.0);  --> 红色  
glColor3f(0.0, 1.0, 0.0);  --> 绿色  
glColor3f(0.0, 0.0, 1.0);  --> 蓝色  
glColor3f(1.0, 1.0, 0.0);  --> 黄色  
glColor3f(1.0, 0.0, 1.0);  --> 品红色  
glColor3f(0.0, 1.0, 1.0);  --> 青色  
glColor3f(1.0, 1.0, 1.0);  --> 白色  
{% endcodeblock %}
这个函数有个需要注意的是，如果在glBegin()与glEnd()连续调用多个这个函数，只显示最后一个
