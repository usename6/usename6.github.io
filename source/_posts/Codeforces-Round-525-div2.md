---
title: 'Codeforces Round #525 div2'
date: 2018-12-06 20:31:52
tags: [ACM,codeforces]
categories: [ACM,codeforces]
---
题目链接：http://codeforces.com/contest/1088
codeforces 525 div2 题解 By usename6
<!--more-->
A. Ehab and another construction problem
http://codeforces.com/contest/1088/problem/A
题意：给你一个x，问你在[1,x]范围是否存在两个数a,b，使得$a*b>x$，且$ \frac{a}{b} < x $
暴力枚举即可
{% codeblock lang:C %}
#include <cstdio>
#include <cstring>
using namespace std;

int main()
{
    int x;
    scanf("%d",&x);
    bool flag=false;
    for(int b=1;b<=x;b++)
    {
        for(int a=b;a<=x;a+=b)
        {
            if(a*b>x&&a/b<x)
            {
                printf("%d %d\n",a,b);
                flag=true;
                break;
            }
        }
        if(flag)break;
    }
    if(!flag)printf("-1\n");
    return 0;
}
{% endcodeblock %}

B. Ehab and subtraction
http://codeforces.com/contest/1088/problem/B
题意：给你一个长度为n的数组，你可以对它做K次操作，每次选出里面最小非0数，然后用这个数减去所有数，如果最后这个数组只剩0，那么输出0
排序，利用前缀累加的性质，乱搞一发就行。
{% codeblock lang:C %}
#include <cstdio>
#include <cstring>
#include <algorithm>
#define MAXN 100005
using namespace std;
typedef long long LL;
LL a[MAXN];
int main()
{
    int n,k;
    scanf("%d%d",&n,&k);
    for(int i=1;i<=n;i++)scanf("%lld",&a[i]);
    sort(a+1,a+1+n);
    LL sum=0;
    printf("%lld\n",a[1]);
    int cnt=1;
    for(int i=1;i<=n;i++)
    {
        if(cnt==k)break;
        a[i]=max(a[i]-sum,0LL);
        if(a[i]==0)continue;
        sum=sum+a[i];
        int id=upper_bound(a+1,a+1+n,sum)-a;
        printf("%lld\n",max(0LL,a[id]-sum));
        cnt++;
        if(cnt==k)break;
    }
    for(int i=cnt+1;i<=k;i++)
        printf("0\n");
    return 0;
}
{% endcodeblock %}

C. Ehab and a 2-operation task
http://codeforces.com/contest/1088/problem/C
题意：给你长度为n的数组，然后你可以每次对这个元素进行两个操作，
1 x y,表示1~x这段都加y
2 x y,表示1~x这段都mody
然后你最多能操作n+1次，问你怎么做能把他变成一个递增序列

简单题，注意是对前缀操作，那我可以指定最后一次对全体数mod n,那么就简单了，那么我可以先对长度为n的做一次前缀操作，使得最后一个数mod n等于n-1，然后遗传对长度为n-1，操作，
直到第一个数为止，最后我在把所有的数mod n,这样的序列一定是递增的。
{% codeblock lang:C %}
#include <cstdio>
#include <cstring>
#include <cmath>
#include <algorithm>
#include <string>
#define MAXN 100005
using namespace std;
int a[MAXN];
int main()
{
    int n;
    scanf("%d",&n);
    for(int i=1;i<=n;i++)
    {
        scanf("%d",&a[i]);
        a[i]%=n;
    }
    printf("%d\n",n+1);
    for(int i=n;i>=1;i--)
    {
        int d=i-1-a[i];
        if(d<0)d+=n;
        printf("1 %d %d\n",i,d);
        for(int j=1;j<=i;j++)
            a[j]=(a[j]+d)%n;
    }
    printf("2 %d %d\n",n,n);
    return 0;
}
{% endcodeblock %}

D. Ehab and another another xor problem
http://codeforces.com/contest/1088/problem/D
题意：交互题，这个题是这样，让你猜数字，你可以输入a,b，要你猜c,d，系统会返回给你一系列的值
a ^ c _ b ^ d (_填写> 、 < 、=)
如果是a ^ c > b ^ d 那么返回值为1
如果是a ^ c < b ^ d 那么返回值为-1
相等返回值为0
还是一样，你最多只能进行62次操作，但c , d保证是小于$2^{30}$
这题看到62次询问，会想到的位操作，每次查询位，
假设你现在猜两个数字的最高位是什么？
1.如果这两个数字的最高位相同，那么你分别对最高位去异或1：
1）如果这两个数的都是1，那么第一个查询结果一定是-1
2）如果这两个数都是0，不需要做任何操作
2.如果两个数字都不相同，首先设定一个flag表示当前这两个数字大小比较
1）如果flag = 1,那么必然a是1
2）否则b是1

如何每次都使得查的是最高位，假设我已经猜出了a的29~i-1的位，那么我只需要那些这些数去异或原数，那么得到一定是i位是最高位，前面的位数都是0
flag的修改，因为flag表示当前除去29~i-1位，i~1位的c和d的比较，所以若第i位相同，我则不需要改变flag,如果第i位不相同，那么我就要改变flag,将c,d当前最高位去掉，使得他们最高位第i位不受任何影响，则异或用查询1的结果去更新即可。
{% codeblock lang:C %}
#include <bits/stdc++.h>
using namespace std;
inline int ask(int c,int d)
{
    printf("? %d %d\n",c,d);
    fflush(stdout);
    int res;
    scanf("%d",&res);
    return res;
}
int main()
{
    int a=0,b=0,flag;
    flag=ask(a,b);
    for(int i=29;i>=0;i--)
    {
        int f1=ask(a^(1<<i),b);
        int f2=ask(a,b^(1<<i));
        if(f1==f2)
        {
            if(flag==1)a=a^(1<<i);
            else if(flag==-1)b=b^(1<<i);
            flag=f1;
        }
        else if(f1==-1)a=a^(1<<i),b=b^(1<<i);
    }
    printf("! %d %d\n",a,b);
}
{% endcodeblock %}

E. Ehab and a component choosing problem
http://codeforces.com/contest/1088/problem/E
题意：给你一个n个节点的树，让你求选K个联通快，首先要最大化比例值，即联通块的权值和/k的值，其次最大化K的值
如果最大化比例，乍一看有点像01分数规划，但这里选的是联通快，而不是单独节点，这里我们放弃联通块的想法，考虑一种思路
没错，就是贪心。
设b是平均值
若选了k个联通块，权值分别为
$a_{1}，a_{2}，a_{3}，a_{4}…a_{k}$
那么$b=1/k*(a_{1}+a_{2}+…a_{k})$
即里面$a_{i}$可以用b替换
如果此时要增加平均值
则必须满足$a_{k+1}>=b$，那么就是平均值才会增加，于是不如贪心的选最大值，然后找到和最大值相同的有多少个
然后这个题就变成如何找最大联通块的问题，可以用树形DP解决，首先思考两个个问题：
1.如何求解树上最大联通快
2.如何求解树上最大联通快的个数
3.如何求解不相交最大联通块个数
可以定义状态dp[i]表示以i为根节点的子树的最大联通块的值大小，注意i必须选
那么dp[i]=max(dp[i],dp[i]+dp[j]) 这里j是i的儿子节点，往上递归就行，这样就解决了如何求解最大联通块的问题。
统计也是如此，当某个子树的联通块的大小等于最大值mx，则令其为0，这样保证了不相交。然后统计一下等于mx的联通快的个数有多少个，这题就做完了
{% codeblock lang:C %}
#include <bits/stdc++.h>
#define MAXN 300005
using namespace std;
typedef long long LL;
struct Edge{int v,next;};
int head[MAXN],tot,w[MAXN];
Edge e[MAXN<<1];
inline void add(int u,int v)
{
    e[tot].v=v;
    e[tot].next=head[u];
    head[u]=tot++;
}
int n,cnt;
LL mx;
LL DP(int u,int pre,bool flag)
{
    LL sum=w[u];
    for(int i=head[u];i!=-1;i=e[i].next)
    {
        int v=e[i].v;
        if(v!=pre)
            sum=max(sum,sum+DP(v,u,flag));
    }
    if(!flag)mx=max(mx,sum);
    if(sum==mx&&flag)
    {
        cnt++;
        return 0;
    }
    return sum;
}
int main()
{
    scanf("%d",&n);
    memset(head,-1,sizeof(head));
    tot=0;
    for(int i=1;i<=n;i++)scanf("%d",&w[i]);
    for(int i=1;i<n;i++)
    {
        int u,v;
        scanf("%d%d",&u,&v);
        add(u,v);
        add(v,u);
    }
    mx=-1e9;
    DP(1,1,false);
    DP(1,1,true);
    printf("%I64d %d\n",cnt*mx,cnt);
    return 0;
}
{% endcodeblock %}