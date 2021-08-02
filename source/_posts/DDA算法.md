---
title: DDA算法
date: 2018-10-27 22:04:32
tags: [OpenGL,图形学]
categories: [OpenGL,图形学]
---
今日连更四篇，也是很肛（沉迷博客无法自拔）。。。
<!--more-->
原理：
假设直线为：$ y = kx + b $
分两种情况讨论
1.当$ | k | < = 1 $
则设$x_{i+1} = x_{i} + 1$
那么$y_{i+1} = y_{i} + k$
2.当$ | k | > = 1 $
则将直线方程转化为$ x = (1/k)y - b/k $
则设$y_{i+1} = y_{i} + 1$
那么$x_{i+1} = x_{i} + 1/k$
{% codeblock lang:C %}
#include <GL/glut.h>
#include <cmath>
#include <cstdio>
#include <algorithm>
void DDA_Line(int x1,int y1,int x2,int y2)  //描绘点在(x1,y1)到(x2,y2)之间的线段
{
    float k;   //表示斜率
    float t;   //在这之间像素点有多少个
    float x;   //起始点x坐标
    float y;   //起始点y坐标
    float dx=x2-x1; //x方向的增量
    float dy=y2-y1; //y方向的增量
    bool tag=false; //表示斜率的绝对值是大于还是小于1,大于1为true,小于1为false
    if(abs(dx)>=abs(dy))
    {
        k=dy/dx;
        t=dx;
        tag=true;
    }
    else
    {
        k=dx/dy;
        t=dy;
        tag=false;
    }
    x=x1;
    y=y1;
    for(int i=0;i<=t;i++)       //枚举这之间的像素点
    {
        glPointSize(1.0);
        glBegin(GL_POINTS);
        if(tag)
        {
            glVertex2f(x,int(y+0.5));
            glEnd();
            y=y+k;
            x++;
        }
        else
        {
            glVertex2f(int(x+0.5),y);
            glEnd();
            x=x+k;
            y++;
        }
    }
}
void display()
{
    glClearColor(1.0, 1.0, 1.0, 1.0);  //白色
    glClear(GL_COLOR_BUFFER_BIT);
    //设置背景为白色
    glViewport(0,0,500,500);

    DDA_Line(0,500,500,0);

    DDA_Line(0,0,500,600);

    DDA_Line(250,0,250,500);

    DDA_Line(0,250,500,250);


    glFlush();
}
int main(int argc,char **argv)
{
    glutInit(&argc,argv);
    glutInitDisplayMode(GLUT_SINGLE|GLUT_RED);
    glutInitWindowSize(500,500);
    glutInitWindowPosition(0,0);
    glutCreateWindow("DDA_Line");
    glutDisplayFunc(display);               //调用里面函数的绘图函数（当然是你定义）

    glColor3f(0.0, 1.0, 1.0);
    gluOrtho2D(0.0, 500.0, 0.0, 500.0);

    glutMainLoop();
    return 0;
}
{% endcodeblock %}