---
title: Open GL 基本函数二
date: 2018-10-27 21:31:31
tags: [OpenGL]
categories: [OpenGL]
---
Open GL 基本函数二
<!--more-->
{% codeblock lang:C %}
void glClearColor(GLclampf red,GLclampf green,GLclampf blue,GLclampf alpha);
{% endcodeblock %}
参数分别是红，绿，蓝，不透明度，取值范围在[0,1]，仅仅只是执行设定颜色，不执行清除工作，仅仅是为了glClear()函数做准备

{% codeblock lang:C %}
glClear(GL_COLOR_BUFFER_BIT|GL_DEPTH_BUFFER_BIT)
{% endcodeblock %}
参数可为一个或多个，若为多个，用'|'隔开
GL_COLOR_BUFFER_BIT
当前可写的颜色缓冲
GL_DEPTH_BUFFER_BIT
深度缓冲
GL_ACCUM_BUFFER_BIT 
累积缓冲
GL_STENCIL_BUFFER_BIT 
模板缓冲

如果两个函数结合一起用，即
{% codeblock lang:C %}
glClearColor(1.0, 1.0, 1.0, 1.0);  //白色
glClear(GL_COLOR_BUFFER_BIT);
{% endcodeblock %}
表示把背景设置成蓝色

{% codeblock lang:C %}
void glPointSize(GLfloat size)
{% endcodeblock %}
设置绘制时点的大小，大小单位为像素，默认值为1.0，很明显要大于0.0f

{% codeblock lang:C %}
void glBegin(GLenum mode)
{% endcodeblock %}
mode表示创建元素的类型，有如下几个类型
GL_POINTS 单个顶点集 
GL_LINES 多组双顶点线段 
GL_POLYGON 单个简单填充凸多边形 
GL_TRAINGLES 多组独立填充三角形 
GL_QUADS 多组独立填充四边形 
GL_LINE_STRIP 不闭合折线 
GL_LINE_LOOP 闭合折线 
GL_TRAINGLE_STRIP 线型连续填充三角形串 
GL_TRAINGLE_FAN 扇形连续填充三角形串 
GL_QUAD_STRIP 连续填充四边形串

{% codeblock lang:C %}
void glEnd(void)
{% endcodeblock %}
一个几何图形绘制的结束

{% codeblock lang:C %}
glVertex2f(float,float)
{% endcodeblock %}
设定点的坐标，f表示参数是浮点数
类似的还有：
glVertex2d
glVertex2f
glVertex3f
glVertex3fv
字母表示参数的类型
1.s表示16位整数（OpenGL中将这个类型定义为GLshort），
2.i表示32位整数（OpenGL中将这个类型定义为GLint和GLsizei），
3.f表示32位浮点数（OpenGL中将这个类型定义为GLfloat和GLclampf），
4.d表示64位浮点数（OpenGL中将这个类型定义为GLdouble和GLclampd）。
5.v表示传递的几个参数将使用指针的方式。

{% codeblock lang:C %}
glViewport(GLint x,GLint y,GLsizei width,GLsizei height)
{% endcodeblock %}
x，y 以像素为单位，指定了视口的左下角位置。
width，height 表示这个视口矩形的宽度和高度，根据窗口的实时变化重绘窗口。
