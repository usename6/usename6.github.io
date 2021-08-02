---
title: Educational Codeforces Round 53 (Rated for Div. 2)
date: 2018-10-27 10:41:24
tags: [codeforces]
categories: [codeforces]
---
传送门：http://codeforces.com/contest/1073
职业fst选手没有被fst，可喜可贺，成功的上了分
<!--more-->
A. Diverse Substring
水题，直接枚举所有子串判断一下就行，实际上有个更好的做法，直接看相邻的两个字母是不是相同，不相同的话，就取那两个就行了，复杂度O(n)
{% codeblock A.cpp lang:C %}
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
string s;
int cnt[30];
int main()
{
    int n;
    cin>>n;
    cin>>s;
    bool flag=false;
    for(int i=0;i<n;i++)
    {
        for(int j=1;i+j-1<n;j++)
        {
            string ss=s.substr(i,j);
            memset(cnt,0,sizeof(cnt));
            for(int k=0;k<ss.size();k++)
            {
                int id=ss[k]-'a';
                cnt[id]++;
            }
            bool tag=true;
            for(int k=0;k<26;k++)
            {
                if(cnt[k]>ss.size()/2)
                {
                    tag=false;
                    break;
                }
            }
            if(tag)
            {
                flag=true;
                cout<<"YES"<<endl;
                cout<<ss<<endl;
            }
            if(flag)break;
        }
        if(flag)break;
    }
    if(!flag)cout<<"NO"<<endl;
    return 0;
}
{% endcodeblock %}

B. Vasya and Books
没什么难度，按题目模拟就行
{% codeblock B.cpp lang:C %}
#include <bits/stdc++.h>
using namespace std;
int a[200005];
int b[200005];
bool used[200005];
int main()
{
    int n;
    scanf("%d",&n);
    for(int i=1;i<=n;i++)scanf("%d",&a[i]);
    for(int i=1;i<=n;i++)scanf("%d",&b[i]);
    int j=1;
    for(int i=1;i<=n;i++)
    {
        if(used[b[i]])
            printf("%d%c",0,i==n?'\n':' ');
        else
        {
            int cnt=0;
            for(;j<=n;j++)
            {
                cnt++;
                used[a[j]]=true;
                if(a[j]==b[i])
                {
                    j++;
                    break;
                }
            }
            //printf("%d %d\n",b[j],a[i]);
            //printf("j=%d cnt=%d\n",j,cnt);
            printf("%d%c",cnt,i==n?'\n':' ');
        }

    }
    return 0;
}
{% endcodeblock %}

C. Vasya and Robot
题目意思就是给你长度为n移动序列，让你通过修改给定移动序列，使得你能从(0,0)到(x,y)，并求出所有修改中，最大的位置-最小位置+1的最小值
刚开始看这题，感觉是双指针，不算很快速的敲完了双指针，后来发现双指针是个假算法，冷静分析了一下，发现区间具有单调性，而且，我只有固定好修改的区间在哪，直接看当前区间的长度和距离之间的关系，二分check一下就好了，最后三分钟绝杀这题
{% codeblock C.cpp lang:C %}
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int n;
char op[200005];
int x,y;
int dx[200005],dy[200005];
bool check(int mid)
{
    int cx=0,cy=0;
    //printf("mid=%d\n",mid);
    for(int i=1;i<=n;i++)
    {
        int j=i+mid-1;
        if(j>n)break;
        cx=cy=0;
        cx=dx[i-1]+(dx[n]-dx[j]);
        cy=dy[i-1]+(dy[n]-dy[j]);
        //printf("i=%d  cx=%d cy=%d\n",i,cx,cy);
        if(mid<abs(x-cx)+abs(y-cy))continue;
        if((mid-abs(x-cx)+abs(y-cy))%2==0)return true;
    }
    return false;
}
int main()
{
    scanf("%d",&n);
    scanf("%s",op+1);
    for(int i=1;i<=n;i++)
    {
        dx[i]+=dx[i-1];
        dy[i]+=dy[i-1];
        if(op[i]=='R')dx[i]++;
        else if(op[i]=='L')dx[i]--;
        else if(op[i]=='U')dy[i]++;
        else if(op[i]=='D')dy[i]--;
        //printf("%d %d %d\n",i,dx[i],dy[i]);
    }
    scanf("%d%d",&x,&y);
    if(dx[n]==x&&dy[n]==y)
    {
        printf("0\n");
        return 0;
    }
    if(abs(x)+abs(y)>n)
    {
        printf("-1\n");
        return 0;
    }
    int l=1,r=n;
    int ans=1e9;
    while(l<=r)
    {
        int mid=(l+r)>>1;
        if(check(mid))
        {
            ans=min(mid,ans);
            r=mid-1;
        }
        else l=mid+1;
    }
    printf("%d\n",ans==1e9?-1:ans);
    return 0;
}
{% endcodeblock %}

D. Berland Fair
好像是给你一个圆形序列，然后你手中有t元钱，你从1位置开始一直走圆形序列，如果能买当前物品，你就去买，但是你只能买一件，当你最后什么都不能买的时候，就退出，问可以买多少件物品
可以证明，假设你买的是序列{1,3,4,7},那么你下次一定也只能买{1，3，4，7}，那么根据贪心思想，我计算一下这个序列能被我买多少次，有点类似除法和取mod的思想
{% codeblock D.cpp lang:C %}
#include <bits/stdc++.h>
using namespace std;
typedef long long LL;
int n;
LL t;
int a[200005];
int main()
{
    scanf("%d%I64d",&n,&t);
    for(int i=1;i<=n;i++)scanf("%d",&a[i]);
    LL ans=0;
    while(1)
    {
        LL cur=0;
        LL cnt=0;
        for(int i=1;i<=n;i++)
        {
            if(cur+a[i]<=t)
            {
                cnt++;
                cur+=a[i];
            }
        }
        if(cur==0)break;
        ans+=cnt*(t/cur);
        t%=cur;
    }
    printf("%I64d\n",ans);
    return 0;
}
{% endcodeblock %}

