---
title: Bresenham算法
date: 2018-10-29 19:46:36
tags: [图形学,Open GL]
categories: [图形学,Open GL]
---
Bresenham算法是第三种基于扫描线的算法，第一种是DDA算法，非常简单，直接强制转化y（四舍五入）然后绘点，而中点画线法，通过将函数隐式化，通过不等式关系，查看点与直线的关系，选择最近的像素点
<!--more-->
那么Bresenham算法又是怎么做的呢？
假设$dx = x2 - x1 $, $dy = y2 - y1$,那么直线方程可以表示为
$(\frac{dy}{dx})x + b = y$
取$k = \frac{dy}{dx}$
若假设点$p_{i}$坐标为$(x_{i},y_{i})$,则
分类对直线斜率进行分类讨论 
k > 0 时，若直线是按坐标序枚举，很容易发现相邻两个坐标之间的增量是正值
若$|dx| > |dy|$
剩下的过程其实和中点画线差不多，详细过程改日再补
{% codeblock lang:C %}
#include <GL/glut.h>
#include <cmath>
#include <cstdlib>
#include <algorithm>
#include <cstdio>
#include <cstring>
void Bresenham(int x1,int y1,int x2,int y2)
{
    int x,y,dx,dy,d,d1,d2,inc;
    dx=x2-x1;
    dy=y2-y1;
    if(dx*dy>=0)inc=1;
    else inc=-1;
    if(abs(dx)>abs(dy))
    {
        if(dx<0)
        {
            std::swap(x1,x2);
            std::swap(y1,y2);
            dx=-dx;dy=-dy;
        }
        d=2*dy-dx;
        d1=2*dy;
        d2=2*(dy-dx);
        x=x1;
        y=y1;
        while(x<x2)
        {
            glPointSize(5.0);
            glBegin(GL_POINTS);
            glVertex2i(x,y);
            glEnd();
            x++;
            if(d<0)d+=d1;
            else
            {
                y+=inc;
                d+=d2;
            }
        }
    }
    else
    {
        if(dy<0)
        {
            std::swap(x1,x2);
            std::swap(y1,y2);
            dx=-dx;dy=-dy;
        }
        d=2*dy-dx;
        d1=2*dy;
        d2=2*(dy-dx);
        x=x1;
        y=y1;
        while(y<y2)
        {
            glPointSize(5.0);
            glBegin(GL_POINTS);
            glVertex2i(x,y);
            glEnd();
            y++;
            if(d<0)d+=d1;
            else
            {
                x+=inc;
                d+=d2;
            }
        }
    }
}
void display()
{
    glClearColor(1.0, 1.0, 1.0, 1.0);
    glClear(GL_COLOR_BUFFER_BIT);

    glViewport(0,0,500,500);

    Bresenham(0,0,500,100);
    Bresenham(0,0,500,200);
    Bresenham(0,0,500,300);
    Bresenham(0,0,500,400);

    Bresenham(0,500,500,0);
    //printf("over\n");

    glFlush();
}
int main(int argc,char **argv)
{
    glutInit(&argc,argv);
    glutInitDisplayMode(GLUT_SINGLE|GLUT_RED);
    glutInitWindowSize(500,500);
    glutInitWindowPosition(0,0);
    glutCreateWindow("MPLine");
    glutDisplayFunc(display);               //调用里面函数的绘图函数（当然是你定义）

    glColor3f(0.0, 0.0, 0.0);
    gluOrtho2D(0.0, 500.0, 0.0, 500.0);

    glutMainLoop();
    return 0;
}
{% endcodeblock %}