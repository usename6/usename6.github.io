---
title: Red Black Tree
date: 2018-11-06 12:04:47
tags: [ACM,数据结构,LCA,二分]
categories: [ACM,数据结构,LCA,二分]
---
ZOJ 4048
题目意思是给你一颗树，有n个节点，有m个红节点，当然根也是给红节点，给出q次查询，每次查询k个节点，你可以修改树上的一个黑节点为红色，求出所有修改中距离红节点的最大值的最小
<!--more-->
最大值最小，一看就是二分，这题如果不带脑袋一想，应该就是求K个点的LCA，然后暴力二分check就行，check的方法很简单吧，就对于当前答案mid，找到所有大于mid，对这些节点求LCA，这样相当于是贪心吧，因为这些点距离最大值要最小，只能尽可能走LCA，于是就很简单了，如果这些点没有LCA，那肯定不行，如果有LCA，那就查看最大值是否小于mid就OK了。
据说倍增过不了，还没试，第二次写RMQ+LCA，卡的不行，没想到时间戳是两倍的空间，这个注意到了，然后上网查了一下题解，发现大家都很去掉一个log，于是去学习了下RMQ去掉log的姿势，然后做了排序剪枝二分check，当然是拉出来剪枝的，加了读入优化，硬是优化过去了，昨晚上一直卡，后来发现数组越界，然后发现答案会爆int，刷爆了ZOJ的测评，不知道是不是被ZJU给注意到了，早上起来改了改mid，然后交了一发，2010ms过吧，才发现ZOJ加时
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN 100015
#define MAXM 200030
using namespace std;
typedef long long LL;
struct Edge{int v,w,next;};
Edge e[MAXM];
int head[MAXN],tot;
int first[MAXN*2],R[2*MAXN],p[MAXN*2],cnt,dep[MAXN];
int node[MAXN],k,st[MAXN];
bool isred[MAXN];
int dp[MAXN*2][20];
int mm[MAXN*2];
LL dis[MAXN],cost[MAXN];
int n,m,q;
bool cmp(int a,int b)
{
    return cost[a]>cost[b];
}
int read(){
	char c=getchar();
	while (c>'9'||c<'0') c=getchar();
	int x=0;
	while ('0'<=c && c<='9'){
		x=x*10+c-'0';   c=getchar();
	}
	return x;
}
void add(int u,int v,int w)
{
    e[tot].v=v;
    e[tot].w=w;
    e[tot].next=head[u];
    head[u]=tot++;
}
void dfs(int u,int depth,int red,int pre)
{
    first[u]=++cnt;p[cnt]=u;R[cnt]=depth;
    dep[u]=depth;
    cost[u]=dis[u]-dis[red];                              //每个节点到红色节点的距离
    for(int i=head[u];i!=-1;i=e[i].next)
    {
        int v=e[i].v;
        if(v!=pre)
        {
            dis[v]=dis[u]+(LL)e[i].w;
            if(isred[v])dfs(v,depth+1,v,u);                   //下传最近的红色点
            else dfs(v,depth+1,red,u);
            p[++cnt]=u;
            R[cnt]=depth;
        }
    }
}
void ST(int n)
{
    mm[0]=-1;
    for(int i=1;i<=n;i++)
        mm[i] = ((i&(i-1)) == 0) ? mm[i-1]+1 : mm[i-1];
    for(int i=1;i<=n;i++)dp[i][0]=i;
    for(int j=1;j<=mm[n];j++)
    {
        for(int i=1;i+(1<<j)-1<=n;i++)
        {
            int a=dp[i][j-1],b=dp[i+(1<<(j-1))][j-1];
            dp[i][j]=R[a]<R[b]?a:b;
        }
    }
}
int RMQ(int l,int r)
{
    int k=mm[r-l+1];
    int a=dp[l][k],b=dp[r-(1<<k)+1][k];
    return R[a]<R[b]?a:b;
}
int LCA(int u,int v)
{
    int x=first[u],y=first[v];
    if(x>y)swap(x,y);
    int res=RMQ(x,y);
    return p[res];
}
bool ck(LL mid)
{
    int top=0;                      //top的值
    for(int i=1;i<=k;i++)
    {
        if(cost[node[i]]>mid)
            st[++top]=node[i];
    }
    if(top<=1)return true;          //小于或等于1个点改这个点即可
    int lca=st[1];
    for(int i=2;i<=top;i++)
        lca=LCA(lca,st[i]);
    for(int i=1;i<=top;i++)
        if(dep[lca]>dep[st[i]])return false;             //不存在LCA
    for(int i=1;i<=top;i++)
        if(dis[st[i]]-dis[lca]>mid)return false;       //大于答案的值
    return true;
}
int main()
{
    int t;
    t=read();
    while(t--)
    {
        n=read();m=read();q=read();
        memset(isred,false,sizeof(isred));
        cnt=tot=0;
        memset(head,-1,sizeof(head));
        for(int i=1;i<=m;i++)
        {
            int id;
            id=read();
            isred[id]=true;
        }
        for(int i=1;i<n;i++)
        {
            int u,v,w;
            u=read();v=read();w=read();
            add(u,v,w);
            add(v,u,w);
        }
        dis[1]=0;
        dfs(1,1,1,1);
        ST(2*n-1);              //dfs序是二倍
        while(q--)
        {
            k=read();
            LL mx=0;
            for(int i=1;i<=k;i++)
                node[i]=read();
            sort(node+1,node+1+k,cmp);
            mx=cost[node[1]];
            LL l=0,r=mx;
            LL ans;
            while(l<=r)
            {
                LL mid=(l+r)>>1;
                if(ck(mid))
                {
                    ans=mid;
                    r=mid-1;
                }
                else l=mid+1;
            }
            printf("%lld\n",ans);
        }
    }
    return 0;
}
{% endcodeblock %}