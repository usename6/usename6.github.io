---
title: Wannafly挑战赛27
date: 2018-10-28 22:39:52
tags: [Wannafly,ACM]
categories: [ACM,Wannafly]
---
https://www.nowcoder.com/acm/contest/215#question
A.灰魔法师
<!--more-->
给出长度为n的序列a, 求有多少对数对 (i, j) (1 <= i < j <= n) 满足 $a_{i} + a_{j}$ 为完全平方数。
因为这玩意$n<=100000$，于是我去暴力打了一个表，发现这个完全平方数不是很多，直接二分check就行了，跟沈阳的G比起来差远了
{% codeblock lang:C %}
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
vector<int> v;
int n;
int a[200005];
int cnt[200005];
int main()
{
    for(int i=1;i<=200000;i++)
    {
        int tp=sqrt(i);
        if(tp*tp==i)
            v.push_back(i);
    }
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
    {
        scanf("%d",&a[i]);
        cnt[a[i]]++;
    }
    LL ans=0;
    for(int i=1;i<=n;i++)
    {
        int id=upper_bound(v.begin(),v.end(),a[i])-v.begin();
        //printf("yes\n");
        for(int j=id;j<(int)v.size();j++)
        {
            ans+=cnt[v[j]-a[i]];
            if(v[j]==2*a[i])ans--;
        }
    }
    printf("%lld\n",ans/2);
    return 0;
}
{% endcodeblock %}

B.紫魔法师
给出一棵仙人掌(每条边最多被包含于一个环，无自环，无重边，保证连通)，要求用最少的颜色对其顶点染色，满足每条边两个端点的颜色不同，输出最小颜色数即可
签到题二，这个题，很明显发现答案的取值范围在1，2，3之间，为什么呢，因为题目保证每条边最多被包含一个环，也就是这个图好吧，就是类似联通快一个东西，
可以证明奇数环下，至少需要3个颜色，其余偶环和长度为2的颜色只需要2个颜色，然后特殊考虑其中一个边界，那就是都是单点的时候，这个时候一个颜色就行，偶环
就用二分图染色法搞一搞。
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN 100005
using namespace std;
vector<int> G[MAXN];
int n,m;
int color[MAXN];
bool BFS(int s)
{
    queue<int> q;
    //printf("s=%d\n",s);
    color[s]=1;
    q.push(s);
    while(!q.empty())
    {
        int u=q.front();
        q.pop();
        for(int i=0;i<(int)G[u].size();i++)
        {
            int v=G[u][i];
            if(color[v]!=-1&&color[v]==color[u])return true;
            if(color[v]==-1)q.push(v);
            color[v]=color[u]^1;
        }
    }
    return false;
}
int main()
{
    scanf("%d%d",&n,&m);
    if(!m)
    {
        printf("1\n");
        return 0;
    }
    for(int i=1;i<=m;i++)
    {
        int u,v;
        scanf("%d%d",&u,&v);
        G[u].push_back(v);
        G[v].push_back(u);
    }
    memset(color,-1,sizeof(color));
    bool flag=false;
    for(int i=1;i<=n;i++)
    {
        if(color[i]==-1)
        {
            if(BFS(i))
            {
                flag=true;
                break;
            }
        }
    }
    flag?puts("3"):puts("2");
    return 0;
}
{% endcodeblock %}

C.蓝魔法师

“你，你认错人了。我真的，真的不是食人魔。”--蓝魔法师

给出一棵树，求有多少种删边方案，使得删后的图每个连通块大小小于等于k，两种方案不同当且仅当存在一条边在一个方案中被删除，而在另一个方案中未被删除，答案对998244353取模
考试的时候，想出是树形dp，怎么合并啊，想了一个小时，想不出来，遂弃疗，好吧其实是再不复习图形学就GG了。
dp[u][num]表示根节点为u的子树下大小为num节点合并的方案数
首先去枚举儿子节点，然后去枚举每个儿子中的多少个节点和父亲合并后的方法数，很明显是个乘法原理，把所有的方案数加起来
这操作很秀吧，滚两个数组，c和dp，c表示合并当前子树的方案数，然后滚动一下，怎么老是滚动优化，真的是。。。，DP这么喜欢用滚动吗
然后注意考虑下DP边界，就是儿子选0个合并的时，需要计算子树v 1~k的方案数的和，因为不选的话,v也是一样1~k的选择权（选k以上肯定不符合题意）
于是这题就做完了，不得不说usename6 DP 真垃圾
{% codeblock lang:C %}
#include <cstdio>
#include <cstring>
#include <string>
#include <cmath>
#include <algorithm>
#include <iostream>
#define MAXN 2010
using namespace std;
typedef long long LL;
const LL mod=998244353;
struct Edge{int v,next;};
int head[MAXN],num[MAXN],tot,n,k;
LL dp[MAXN][MAXN],c[MAXN];
Edge e[MAXN<<1];
void add(int u,int v)
{
    e[tot].v=v;
    e[tot].next=head[u];
    head[u]=tot++;
}
void dfs(int u,int pre)
{
    //printf("u=%d\n",u);
    dp[u][1]=1;         //初始化当u这个子树为1的方案数
    num[u]=1;          //表示u的子树个数
    for(int i=head[u];i!=-1;i=e[i].next)
    {
        int v=e[i].v;
        if(v!=pre)
        {
            dfs(v,u);
            //printf("%d --- > %d\n",u,v);
            for(int x=0;x<=num[v];x++)      //枚举v子树的大小，从0开始是因为他可不选
            {
                for(int y=1;y<=num[u];y++) //枚举u子树的大小，从1开始是因为他必须选
                {
                    c[x+y]+=dp[u][y]*dp[v][x]%mod;
                    c[x+y]%=mod;
                    //printf("%d %lld\n",x+y,c[x+y]);
                }
            }
            num[u]+=num[v];
            for(int x=1;x<=num[u];x++)
            {
                dp[u][x]=c[x];
                c[x]=0;
            }
        }
    }
    for(int i=1;i<=min(k,num[u]);i++)
        dp[u][0]=(dp[u][0]+dp[u][i])%mod;
}
int main()
{
    memset(head,-1,sizeof(head));
    tot=0;
    scanf("%d%d",&n,&k);
    for(int i=1;i<n;i++)
    {
        int u,v;
        scanf("%d%d",&u,&v);
        add(u,v);
        add(v,u);
    }
    dfs(1,1);
    printf("%lld\n",dp[1][0]);
    return 0;
}
{% endcodeblock %}

D.绿魔法师
听说是容斥？数论选手决定留坑