---
title: NAIPC-C.Greeting
date: 2018-11-06 17:08:14
tags: [ACM,DP,状压DP]
categories: [ACM,DP,状压DP]
---
题目链接：http://codeforces.com/gym/101002
国庆做的NAIPC，usename6来补题
<!--more-->
题目意思很简单，就是你给你N种信件，宽度为w，高度为h,数量为q，你可以自定义k种信封，让你求怎么制定信封，浪费的面积最小，信件不能旋转，不然可能还需要处理一下
数据范围很小，很容易往折半或者是状压，但是我觉得折半的话，得32这样吧，不然失去意义了，所以往状压DP想，若定义dp[i][s]表示i种信封可以覆盖状态为s的最小信件，预处理信件集合s的最小花费c[s],很明显我先去枚举s，再在s中枚举j，判断s需不需要拆分
转移方程为 $dp[i][s] = min(dp[i-1][s-j]+cost[j],dp[i][s])$ 其中j是s的子集，这样就做完了
这题有一个个坑点，结果会爆int,于是答案的上界应该处理为1e16左右
这题复杂度$O(3^n)$，还没证明出来。。。
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN (1<<15)+100
using namespace std;
typedef long long LL;
const LL INF=1e16;
LL dp[16][MAXN],c[MAXN];
LL w[16],h[16],q[16];
int n,k;
int main()
{
    scanf("%d%d",&n,&k);
    for(int i=0;i<n;i++)
        scanf("%lld%lld%lld",&w[i],&h[i],&q[i]);
    for(int s=1;s<(1<<n);s++)
    {
        LL mxw,mxh,need;
        int cnt=0;
        need=mxw=mxh=0LL;
        for(int i=0;i<n;i++)
        {
            if(s>>i&1)
            {
                cnt+=q[i];
                mxw=max(mxw,w[i]);
                mxh=max(mxh,h[i]);
                need+=w[i]*h[i]*q[i];
            }
        }
        c[s]=mxw*mxh*cnt-need;
    }
    for(int i=0;i<=k;i++)
        for(int j=0;j<(1<<n);j++)
            dp[i][j]=INF;
    dp[0][0]=0;                 //初始化边界条件
    for(int i=1;i<=k;i++)       //枚举能用多少个
        for(int s=0;s<(1<<n);s++)
            for(int j=s;j;j=(j-1)&s)
                if(dp[i-1][s-j]<INF)dp[i][s]=min(dp[i][s],dp[i-1][s-j]+c[j]);
    LL ans=INF;
    for(int i=0;i<=k;i++)ans=min(ans,dp[i][(1<<n)-1]);
    printf("%lld\n",ans);
    return 0;
}
{% endcodeblock %}