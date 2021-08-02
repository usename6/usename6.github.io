---
title: DP优化专题一
date: 2018-10-28 23:10:17
tags: [DP,ACM]
categories: [ACM,DP]
---
DP优化专题
虽然不是DP的选手，但是可见DP的重要性，DP这个东西高深的很，打算写一写关于DP的事情，usename6 DP 水平有限？dalao求轻喷
<!--more-->
DP优化一 枚举优化
曾经在一次暑假集训的讲课中，选择了讲DP，本来就不是很懂DP这个东西，于是随便选了几个题讲讲，没想到就讲出了DP的其中一个优化，枚举优化，若不是雨神说，我可能都不知道这是什么。。。
枚举优化是什么？
DP的优化有两种方向，第一种是选择优化时间，第二种是优化空间，如果从时间的角度去考虑优化，那么就要从转移上考虑，如何优化转移，是优化时间重要的思想
首先思考这样一个问题：
给一个nxm的矩阵，要求你求出一个子矩阵，使得这个矩阵的和最大这个题的传统做法就是枚举起点(x1,y1)、终点(x2,y2)，然后套两重循环，复杂度O($n^6$)
一个优化就是加上二维前缀和，这样省去了两个循环，复杂度O($n^4$)
固定行的上下界，然后将一列压成一个元素，这题就变成了最大子段和的问题，DP即可O($n^3$)
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define maxn 600
using namespace std;
typedef long long LL;
int n,m;
LL sum[maxn][maxn],a[maxn],dp[maxn];
int main()
{
    scanf("%d%d",&m,&n);
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            int x;
            scanf("%d",&x);
            sum[i][j]=sum[i-1][j]+x;
        }
    }
    LL mx=0;
    for(int i=1;i<=n;i++)
    {
        for(int j=i;j<=n;j++)
        {
            dp[0]=0;
            for(int k=1;k<=m;k++)
            {
                a[k]=sum[j][k]-sum[i-1][k];
                dp[k]=max(a[k],dp[k-1]+a[k]);
                mx=max(mx,dp[k]);
            }
        }
    }
    printf("%lld\n",mx);
    return 0;
}
{% endcodeblock %}
但是这个题还有一种优化，就是可以枚举优化掉这个东西转移
假设你换一种角度去思考这道题
我都把矩阵的上下界固定了，左右的界能不能固定，对应每个矩阵右端列，我能不能快速找到左端列，使得以该右端列结尾的最大连续子矩阵和
单调队列去优化转移，每次存该列的前缀子矩阵和（类似扫描线推进），然后在1~当前枚举右端列i-1中从单调队列快速提出来
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define maxn 600
using namespace std;
typedef long long LL;
int n,m;
LL sum[maxn][maxn],q[maxn];
int main()
{
    scanf("%d%d",&m,&n);
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            int x;
            scanf("%d",&x);
            sum[i][j]=sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1]+x;
        }
    }
    LL mx=0;
    int qs,qe;
    for(int i=1;i<=n;i++)
    {
        for(int j=i;j<=n;j++)
        {
            qs=0;qe=1;
            q[0]=0;
            //printf("%d %d\n",i,j);
            for(int k=1;k<=m;k++)
            {
                //printf("k = %d\n",k);
                LL data=sum[j][k]-sum[i-1][k];
                //printf("data = %I64d\n",data);
                if(qs<qe)mx=max(mx,data-q[qs]);
                while(qs<qe)
                {
                    if(data>=q[qe-1])break;
                    qe--;
                }
                q[qe]=data;
                qe++;
            }
        }
    }
    printf("%lld\n",mx);
    return 0;
}
{% endcodeblock %}
由于是前缀，于是可以做枚举优化？
由于每次转移需要都是1~i-1的前缀子矩阵和，于是只需要开个数mx维护1~i-1的最小值即可
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define maxn 600
#define INF 0x3f3f3f3f
using namespace std;
typedef long long LL;
int n,m;
LL sum[maxn][maxn],q[maxn];
int main()
{
    scanf("%d%d",&m,&n);
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=m;j++)
        {
            int x;
            scanf("%d",&x);
            sum[i][j]=sum[i-1][j]+sum[i][j-1]-sum[i-1][j-1]+x;
        }
    }
    LL mx=0,mm;
    int qs,qe;
    for(int i=1;i<=n;i++)
    {
        for(int j=i;j<=n;j++)
        {
            mm=0;
            for(int k=1;k<=m;k++)
            {
                LL data=sum[j][k]-sum[i-1][k];
                if(k!=1)mx=max(mx,data-mm);
                if(k==1)mx=max(mx,data);
                mm=min(mm,data);
            }
        }
    }
    printf("%lld\n",mx);
    return 0;
}
{% endcodeblock %}