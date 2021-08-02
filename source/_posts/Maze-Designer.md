---
title: Maze Designer
date: 2018-11-04 15:20:17
tags: [ACM,数据结构,LCA]
categories: [ACM,数据结构,LCA]
---
题目地址:https://nanti.jisuanke.com/t/31462
题目给你nxm的方阵图，用最小花费建墙，相当于是迷宫，且每个方格互相可达，并且最短路径唯一，相当于是nxm的生成树，给出q次查询，求出任意这两个点的最短距离
<!--more-->
首先去建一个最大生成树，因为生成树任意两个点均可达，最大是因为要让其花费最小，把所有花费大的墙拆掉，这样就留下一个唯一的路径，而且花费最小，且任意两点可达
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN 250010
#define MAXM 500020
using namespace std;
typedef long long LL;
int n,m;
struct Edge
{
    int u,v,next,w;
    bool operator <(const Edge &tp)
    {
        return w>tp.w;
    }
};
struct LCA
{
    Edge e[MAXM];
    int fa[MAXN][25];
    int head[MAXN];
    int dep[MAXN];
    int dis[MAXN];
    int tot;
    int height;
    void init()
    {
        memset(head,-1,sizeof(head));
        tot=0;
        height=20;
    }
    void add(int u,int v,int w)
    {
        e[tot].v=v;
        e[tot].w=w;
        e[tot].next=head[u];
        head[u]=tot++;
    }
    void dfs(int u)
    {
        for(int i=1;i<=height;i++)
        {
            if(fa[u][i-1]!=-1)fa[u][i]=fa[fa[u][i-1]][i-1];
            else break;
        }
        for(int i=head[u];i!=-1;i=e[i].next)
        {
            int v=e[i].v;
            if(fa[u][0]!=v)
            {
                dis[v]=dis[u]+e[i].w;
                dep[v]=dep[u]+1;
                fa[v][0]=u;
                dfs(v);
            }
        }
    }
    int lca(int u,int v)
    {
        if(dep[u]<dep[v])swap(u,v);
        int d=dep[u]-dep[v];
        for(int i=0;i<=height;i++)
        {
            if((1<<i)&d)
                u=fa[u][i];
        }
        if(u==v)return u;
        for(int h=height;h>=0;h--)
        {
            if(fa[u][h]!=fa[v][h])
            {
                u=fa[u][h];
                v=fa[v][h];
            }
        }
        return fa[u][0];
    }
    int query(int u,int v)
    {
        return dis[u]+dis[v]-2*dis[lca(u,v)];
    }
};
LCA L;
struct Krusal
{
    Edge e[MAXM];
    int f[MAXN];
    int tot;                        //多少条边
    int N;
    void init()
    {
        tot=0;
        for(int i=1;i<=N;i++)f[i]=i;
    }
    void add(int u,int v,int w)
    {
        e[tot].u=u;                 //记录起点
        e[tot].v=v;
        e[tot].w=w;
        tot++;
    }
    int fid(int u){return f[u]==u?u:f[u]=fid(f[u]);}    //并查集
    void krusal()
    {
        sort(e,e+tot);
        int cnt=0;
        for(int i=0;i<tot;i++)
        {
            int f1=fid(e[i].u);
            int f2=fid(e[i].v);
            if(f1!=f2)
            {
                f[f1]=f2;
                //记录最大生成树的边
                L.add(e[i].u,e[i].v,1);
                L.add(e[i].v,e[i].u,1);
                cnt++;
            }
            if(cnt>=N-1)return;
        }
    }
};
Krusal K;
int main()
{
    while(scanf("%d%d",&n,&m)!=EOF)
    {
        K.N=n*m;
        K.init();
        L.init();
        for(int i=1;i<=n;i++)
        {
            for(int j=1;j<=m;j++)
            {
                char s1[10],s2[10];
                int w1,w2;
                scanf("%s%d%s%d",s1,&w1,s2,&w2);
                if(s1[0]=='D')
                    K.add((i-1)*m+j,i*m+j,w1);
                if(s2[0]=='R')
                    K.add((i-1)*m+j,(i-1)*m+j+1,w2);
            }
        }
        K.krusal();
        int q;
        scanf("%d",&q);
        L.dfs(1);
        for(int i=1;i<=q;i++)
        {
            int x1,y1,x2,y2;
            scanf("%d%d%d%d",&x1,&y1,&x2,&y2);
            int u=(x1-1)*m+y1;
            int v=(x2-1)*m+y2;
            printf("%d\n",L.query(u,v));
        }
    }
    return 0;
}
{% endcodeblock %}