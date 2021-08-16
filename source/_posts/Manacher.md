---
title: Manacher算法
date: 2021-08-16 16:14:10
tags: [Java leetcode]
categories: [Java leetcode]
---
一、回文串的朴素的解法
法一、双模hash+二分（常数很大）
法二、枚举中心点+暴力：
首先预处理字符串，在每个字符串的字符间隔插入'#'，使得所有回文串变成奇数的回文串。
定义$d1[i]$表示以i为中心的回文串的**个数**。
{% codeblock lang:Java %}
for(int i=0;i<n;i++){
    int k=1;
    while(0<=i-k&&i+k<n&&s.charAt(i-k)==s.charAt(i+k)){
        k++;
    }
    d1[i]=k;
}
{% endcodeblock %}
二、回文串的对称性
在以i为中心长度为$d1[i]$的回文串中，定义$\exists j, j < i$ ,那么定义 $ k = (2 \times i - j) $
分类讨论1：如果$ d1[j] < min(i-j+1, j) $
如图，显然对称性成立：
![alt](/images/Manacher/case1.png)
分类讨论2: 如果$ d1[j] > i - j + 1 && d1[j] < min(R-j+1,j) $
同样也可以证明对称性成立：
![alt](/images/Manacher/case2.png)
其中
我们可以把以j为中心的字符串分为两个部分，其中一部分不超过i，在图上是用褐色表示的线。
而另一部分是超过i的部分用绿色的线表示，由于回文的对称性，
那么我们必然能在以k为中心的字符串找到与它相同对应的位置。
由于回文串正反读都是相同的，故可以证明以对称性仍然成立。
分类讨论3： 如果$ d1[j] > min(R-j+1,j) $,那么对称性是部分存在的。
即在$[i-d1[i]+1, i+d1[i]+1]$以j为中心的回文串是存在对称性，
超出范围的回文串是不存在对称性的。

于是我们只需要传播对称性，然后再用朴素方法更新d1[i]即可，然后更新最长回文串的区间，
可以证明这个算法是线性。

实战Manacher算法：
leetcode 1960
题意：给出一串字符串，找到两个互不相交字符串，使其长度的乘积最大。
题解：
法二、马拉车+DP
{% codeblock lang:Java %}
class Manacher{
    String s;
    int[] d1;
    int n;
    /*Manacher初始化*/
    public Manacher(String s){
        this.s=s;
        n = s.length();
        d1 = new int[n+1];
    }
    /*Manacher 核心*/
    public void build(){
        for(int i=0,l=0,r=-1;i<n;i++){
            int k=(i>r)?1:Math.min(d1[l+r-i],r-i+1);
            while(0<=i-k&&i+k<n&&s.charAt(i-k)==s.charAt(i+k)){
                k++;
            }
            d1[i]=k--;
            if(i+k>r){
                l=i-k;
                r=i+k;
            }
        }
    }
}
class Solution {
    public long maxProduct(String s) {
        Manacher m = new Manacher(s);
        m.build();
        int n = s.length();
        int[] pre=new int[n+1];
        int[] suffix=new int[n+1];
        for(int i=0;i<n;i++){
            int l=i-m.d1[i]+1,r=i+m.d1[i]-1;
            pre[r]=Math.max((m.d1[i]-1)*2+1,pre[r]);
            suffix[l]=Math.max((m.d1[i]-1)*2+1,suffix[l]);
        }
        for(int i=1;i<n;i++){
            pre[i]=Math.max(pre[i],pre[i-1]);
        }
        for(int i=n-2;i>=0;i--){
            pre[i]=Math.max(pre[i],pre[i+1]-2);
        }
        for(int i=n-2;i>=0;i--){
            suffix[i]=Math.max(suffix[i],suffix[i+1]);
        }
        for(int i=1;i<n;i++){
            suffix[i]=Math.max(suffix[i],suffix[i-1]-2);
        }
        long ans=0;
        for(int i=0;i<n-1;i++){
            ans=Math.max((long)pre[i]*(long)suffix[i+1],ans);
        }
        return ans;
    }
}
{% endcodeblock %}