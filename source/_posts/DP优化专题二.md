---
title: DP优化专题二
date: 2018-10-29 12:49:52
tags: [DP,ACM]
categories: [ACM,DP]
---
垃圾usename6又来更新DP专题了。。。
<!--more-->
今天要讲的是DP的第二个专题优化，倍增优化，之前在专题一中，我们已经说过了，DP优化无非从时间，即DP转移入手，或是从空间入手。
下面我来看一道这样题目（来源HDU 2157）
给定一个有向图，问从A点恰好走k步（允许重复经过边）到达B点的方案数mod p的值
乍一看是图论，实际上可以从DP角度去考虑这个题，定义状态dp[u][k]表示走k步走到u这个节点的方案数是多少，很明显根据图的性质和加法原理，转移为dp[v][k]+=dp[u][k-1] （存在u---> v的 有向边）
但是若这个k很大的时候，难免就要进行很大计算，我们发现不管走多少步，只要你从k节点出发，转移是固定的，转移的方程也是固定的（！这个很重要）于是，我们可以采用倍增去加速一阶线性递推式，即矩阵快速幂去倍增优化转移
两个状态到两个状态之间转移应该是邻接矩阵，于是对邻接矩阵进行快速幂优化转移即可
{% codeblock lang:C %}
#include <iostream>
#include <cstdio>
#include <cstring>
#define maxn 200
using namespace std;
typedef struct
{
    int a[maxn][maxn];
}Martix;
const int mod=1000;
Martix unit;
int n,m;
Martix operator*(Martix a,Martix b)
{
    Martix res;
    memset(res.a,0,sizeof(res.a));
    for(int i=0;i<n;i++)
    {
        for(int j=0;j<n;j++)
        {
            for(int k=0;k<n;k++)
            {
                res.a[i][j]=(res.a[i][j]+a.a[i][k]*b.a[k][j])%mod;
            }
        }
    }
    return res;
}
void init()
{
    memset(unit.a,0,sizeof(unit.a));
    for(int i=0;i<n;i++)
        unit.a[i][i]=1;
}
Martix qpow_mod(Martix a,int n)
{
    Martix res=unit;
    while(n)
    {
        if(n&1)
            res=res*a;
        n>>=1;
        a=a*a;
    }
    return res;
}
int main()
{
    int s,e,k,cnt,t,A,B;
    Martix res,tmp;
    while(scanf("%d%d",&n,&m)!=EOF)
    {
        if(!n&&!m)break;
        memset(tmp.a,0,sizeof(tmp.a));
        for(int i=1;i<=m;i++)
        {
            scanf("%d%d",&s,&e);
            tmp.a[s][e]=1;
            //printf("%d %d\n",s,e);
        }
        scanf("%d",&t);
        while(t--)
        {
            scanf("%d%d%d",&A,&B,&k);
            init();
            res=qpow_mod(tmp,k);
            cnt=res.a[A][B];
            printf("%d\n",cnt);
        }
    }
    return 0;
}
{% endcodeblock %}

一开始我一直错误的认为矩阵的倍增优化转移只能在一阶线性递推式解决，知道发现了这道题：
有n天，m件衣服，如果某一天穿了第i件衣服，第二天穿了第j件衣服，那么就会获得f[i][j]的权值。然后给你矩阵f，每件衣服可以穿无限多次，问第n天能获得的最大权值是多少。
同样的我们发现，若是定义状态dp[u][k]表示第k天穿了u这件服装，那么转移式子只需把上面的式子进行小小的改动：dp[v][k] = max(dp[u][k-1] + f[u][v])   (u--->v相连)
同样的不管k是多少，对于每个节点的u的转移是不变的，每次都会用到这个矩阵去转移，于是我们采用了模仿矩阵快速幂的方向，做倍增优化，其实是取max和+运算的结合律导致的，这个不做证明，需要用到离散数学的群论和代数系统的知识，有兴趣自行证明，于是导致我们可以先算后面在和前面合并
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN 105
using namespace std;
typedef long long LL;
typedef struct
{
    LL a[MAXN][MAXN];
}Martix;
Martix unit;
int n,m;
Martix operator*(Martix a,Martix b)
{
    Martix ans;
    memset(ans.a,0,sizeof(ans.a));
    for(int i=1;i<=n;i++)
    {
        for(int j=1;j<=n;j++)
        {
            for(int k=1;k<=n;k++)
            {
                ans.a[i][j]=max(ans.a[i][j],a.a[i][k]+b.a[k][j]);
            }
        }
    }
    return ans;
}
void init()
{
    memset(unit.a,0,sizeof(unit.a));
    for(int i=1;i<=n;i++)
        for(int j=1;j<=n;j++)
            unit.a[i][i]=1;
}
Martix qpow_mod(Martix a,LL n)
{
    Martix res;
    memset(res.a,0,sizeof(res.a));
    while(n)
    {
        if(n&1)res=res*a;
        n>>=1;
        a=a*a;
    }
    return res;
}
Martix tmp,ret;
int main()
{
    while(scanf("%d%d",&m,&n)!=EOF)
    {
        init();
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++)
                scanf("%lld",&tmp.a[i][j]);
        ret=qpow_mod(tmp,(LL)m-1);
//        for(int i=1;i<=n;i++)
//        {
//            for(int j=1;j<=n;j++)
//            {
//                printf("%lld ",ret.a[i][j]);
//            }
//            puts("");
//        }
        LL ans=0;
        for(int i=1;i<=n;i++)
            for(int j=1;j<=n;j++)
                ans=max(ans,ret.a[i][j]);
        printf("%lld\n",ans);
    }
    return 0;
}
{% endcodeblock %}

上述两个都是类似分层图的DP，因此没有后效性，可以自己思考一下，状态有二维，尽管图上有环也不会导致后效性