---
title: leetcode-5
date: 2021-10-01 20:55:44
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode 第五次练习，补上25天的笔记。
<!--more-->
leetcode 493
题意：
给定一个数组 nums ，如果 i < j 且 nums[i] > 2*nums[j] 我们就将 (i, j) 称作一个重要翻转对。

你需要返回给定数组中的重要翻转对的数量。

题解：
权值树状数组的裸题，离线+离散化预处理。
然后维护一下$nums[i]$的权值树状数组即可。
code:
{% codeblock lang:Java %}
class BIT{
    int[] tree;
    int n;
    public BIT(int nn){
        n = nn;
        tree = new int[n];
    }
    private int lowbit(int x){
        return x&(-x);
    }
    public void update(int x,int d){
        while(x<=n){
            tree[x] += d;
            x += lowbit(x);
            //System.out.println("update: x="+x);
        }
    }
    public int query(int x){
        int res=0;
        while(x>0){
            res += tree[x];
            x -= lowbit(x);
            //System.out.println("query:x="+x);
        }
        return res;
    }
}
class Solution {
    public int lower_bound(long[] arr,int l,int r,long x){
        int ans=-1;
        while(l<r){
            int mid=(l+r)>>1;
            //System.out.println("["+l+","+r+"]");
            //System.out.println("arr["+mid+"] = " + arr[mid] + ", x = "+x);
            if(arr[mid]>=x){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(arr[l]>=x)ans=l;
        return ans;
    }
    public int reversePairs(int[] nums) {
        int n=nums.length;
        BIT b = new BIT(n*2+1);
        long []a = new long[n*2];
        long []arr = new long[n*2];
        for(int i=0,j=0;i<n;i++,j+=2){
            a[j]=(long)nums[i];
            a[j+1]=(long)nums[i]*2;
        }
        Arrays.sort(a);
        //for(int i=0;i<n*2;i++)System.out.print(a[i]+" ");
        //System.out.println("");
        int cnt=0;
        for(int i=0;i<2*n;i++){
            if(i>0&&a[i]==a[i-1])continue;
            arr[cnt++]=a[i];
        }
        //for(int i=0;i<cnt;i++)System.out.print(arr[i]+" ");
        //System.out.println("");
        int res=0;
        for(int i=0;i<n;i++){
            //System.out.println("i = "+i);
            int id = lower_bound(arr,0,cnt-1,(long)nums[i]*2)+1;
            int tmp = 0;
            //System.out.println("1:arr = "+nums[i]*2 + ", id = "+id);
            tmp = b.query(cnt) - b.query(id);
            //System.out.println("[1,id] = " + b.query(id));
            //System.out.println("cnt = "+cnt+", [1,cnt] = " + b.query(cnt));
            //System.out.println("tmp = "+tmp);
            res = res + tmp;
            id = lower_bound(arr,0,cnt-1,(long)nums[i])+1;
            //System.out.println("2:arr = "+nums[i] + ", id = "+id);
            b.update(id,1);
        }
        return res;
    }
}
{% endcodeblock %}

leetcode 1994
题意：给你一个整数数组 nums 。如果 nums 的一个子集中，所有元素的乘积可以用若干个 互不相同的质数 相乘得到，那么我们称它为 好子集 。
题解：由于nums[i]<=30,而小于30的质数只含有{2,3,5,7,11,13,17,19,23,29}这10个。
于是只用状压DP即可。
dp[i][cur] 表示以i为结尾，当前乘积组成的质因数对应的二进制状态为cur方案数。
状态转移：
$$ dp[i][cur] = dp[i][cur] + dp[i-1][pre] \mod mod$$
但是这样的复杂度是n x 1024，复杂度太大会超时。
所以需要我们把转移进行优化，我们注意到这n个数范围是$$[1,30]$$，是有很大的冗余的。所以我们可以先预先处理这30个数的质因数分解，过滤掉两个相同因子乘积的数，用cnt存下这些数的数量。然后新的转移方程如下：
**这里用滚动数组优化了空间**
$$ dp[cur][j|status] = (dp[cur][j|status] + (dp[cur^1][j] * cnt[i])\mod mod) \mod mod;$$
然后注意边界条件的处理：
很容易推得，选与不选都不会收到影响。故i=1的时候可以用二项式定理推得。
dp[cur][0] = qpow_mod((long)2,(long)cnt[i]);

code:
{% codeblock lang:Java %}
class Solution {
    long mod = 1000000007;
    int[] tt={2,3,5,7,11,13,17,19,23,29};
    int qpow_mod(long a,long nn){
        long res=1;
        while(nn>0){
            if(nn%2==1)res=res*a%mod;
            nn>>=1;
            a=(a*a)%mod;
        }
        return (int)(res%mod);
    }
    String DebugOutput(int now,int n){
        String ans="";
        for(int i=0;i<n;i++){
            if((now&(1<<i))!=0)ans=ans+"1";
            else ans=ans+"0";
        }
        return ans;
    }
    public int numberOfGoodSubsets(int[] nums) {
        long[][] dp = new long[2][1<<10];
        int[] pstatus = new int[31];
        dp[0][0] = 1;
        int n = nums.length;
        for(int i=1;i<=30;i++){
            boolean flag=false;
            int status = 0;
            for(int j=0;j<10&&tt[j]<=i;j++){
                if(i%tt[j]==0){
                    status = status|(1<<j);
                }
                if(i%(tt[j]*tt[j])==0){
                    flag=true;
                    break;
                }
            }
            if(flag)pstatus[i]=-1;
            else pstatus[i]=status;
        }
        long[] cnt = new long[31];
        for(int num:nums)cnt[num]++;
        for(int i=1;i<=30;i++){
            int status = pstatus[i];
            int cur=i%2;
            if(i==1){
                dp[cur][0] = qpow_mod((long)2,(long)cnt[i]);
                continue;
            }
            for(int j=0;j<(1<<10);j++)dp[cur][j] = dp[cur^1][j];
            for(int j=0;j<(1<<10);j++){
                if(status!=-1&&(j&status)==0&&dp[cur^1][j]!=0&&cnt[i]!=0){
                    dp[cur][j|status] = (dp[cur][j|status] + (dp[cur^1][j] * cnt[i])%mod) % mod;
                }
            }
        }
        long ans = 0;
        for(int i=1;i<(1<<10);i++){
            ans = (ans + dp[30%2][i])%mod;
        }
        return (int)ans;
    }
}
{% endcodeblock %}

leetcode 1044
题意：给出一个字符串 S，考虑其所有重复子串（S 的连续子串，出现两次或多次，可能会有重叠）。

返回任何具有最长可能长度的重复子串。（如果 S 不含重复子串，那么答案为 ""。）

题解：hash裸题，这个答案很明显具有单调性，于是二分答案，hash check即可。这里双modhash套合并hash

code:
{% codeblock lang:Java %}
class Hash{
    int[] h1,h2;
    int[] b1,b2;
    int n;
    static long mod1 = 1000000007;
    static long mod2 = 1000000009;
    static long base1 = 23;
    static long base2 = 37;
    public Hash(String str){
        int n = str.length();
        h1 = new int[n+1];
        h2 = new int[n+1];
        b1 = new int[n+1];
        b2 = new int[n+1];
        b1[0]=b2[0]=1;
        long bb1 = 1, bb2 = 1;
        for(int i=1;i<=n;i++){
             bb1 = (bb1 * base1) % mod1;
             bb2 = (bb2 * base2) % mod2;
             b1[i] = (int)bb1;
             b2[i] = (int)bb2;
        }
        h1[0] = h2[0] = 0;
        long hh1 = 0, hh2 = 0;
        for(int i=1;i<=n;i++){
            hh1 = (hh1 * base1 % mod1 + str.charAt(i-1) - 'a')%mod1;
            hh2 = (hh2 * base2 % mod2 + str.charAt(i-1) - 'a')%mod2;
            h1[i] = (int)hh1;
            h2[i] = (int)hh2;
        }
    }
    public long gethash1(int l,int r){
        return ((long)h1[r] - 
            (long)h1[l-1] * b1[r-l+1] % mod1 + mod1)%mod1;
    }
    public long gethash2(int l,int r){
        return ((long)h2[r] - 
            (long)h2[l-1] * b2[r-l+1] % mod2 + mod2)%mod2;
    }
}
class Solution {
    public String check(int mid,String str,Hash hash){
        int n=str.length();
        HashSet<Long> hash1 = new HashSet<Long>();
        for(int i=0;i+mid-1<n;i++){
            Long hh1 = hash.gethash1(i+1, i+mid);
            Long hh2 = hash.gethash2(i+1, i+mid);
            Long hh = (hh1 << 32) + hh2;
            if(hash1.contains(hh)){
                return str.substring(i, i+mid);
            }
            else hash1.add(hh);
        }
        return "";
    }
    public String longestDupSubstring(String s) {
        Hash hash = new Hash(s);
        int l=1,r=s.length();
        String ans="";
        while(l<r){
            int mid = (l+r)>>1;
            String tmp = check(mid, s, hash);
            //System.out.println("[" + l + ", " + r +"]");
            //System.out.println("mid = " + mid);
            if(tmp.length() > 0){
                //System.out.println("ans = " + tmp);
                ans = tmp;
                l = mid + 1;
            }
            else r = mid - 1;
        }
        String tmp = check(l, s, hash);
        //System.out.println("ans = " + tmp);
        if(tmp.length() > 0)ans=tmp;
        return ans;
    }
}
{% endcodeblock %}

leetcoe 1453
题意：
墙壁上挂着一个圆形的飞镖靶。现在请你蒙着眼睛向靶上投掷飞镖。
投掷到墙上的飞镖用二维平面上的点坐标数组表示。飞镖靶的半径为 r 。
请返回能够落在 任意 半径为 r 的圆形靶内或靶上的最大飞镖数。
题解：
贪心，这样半径的圆至少刚好三个点在圆上。
朴素做法：
$O(n^4)$，枚举三个点，然后暴力check。
注意到我们其实是知道半径了，相当于问题多了一个已知条件。所以我们可以减少一个自由度，即省去枚举最后一个点。枚举两个点，利用圆的性质确定圆心。然后暴力check。
复杂度$O(n^3)$
code:
{% codeblock lang:Java %}
class Solution {
    public int check(int[][] points, double x, double y,double r){
        int ans=0;
        for(int[] point:points){
            double dist = Math.sqrt((double)(point[0]-x)*(point[0]-x) + (double)(point[1]-y)*(point[1]-y));
            if(dist <= r)ans++;
        }
        //System.out.println("");
        return ans;
    }
    public int numPoints(int[][] points, int r) {
        int n = points.length;
        int ans = 0;
        for(int i=0;i<n;i++){
            for(int j=i+1;j<n;j++){
                double xx = (double)points[j][0] - (double)points[i][0];
                double yy = (double)points[j][1] - (double)points[i][1];
                double midx = ((double)points[j][0] + (double)points[i][0]) / 2;
                double midy = ((double)points[j][1] + (double)points[i][1]) / 2;
                double tt = Math.sqrt(xx*xx + yy*yy);
                if(tt < 1e-3)continue;
                double xx1 = -yy/tt, yy1 = xx/tt;
                double dist = Math.sqrt((double)(r*r) - (tt/2)*(tt/2));
                double Rx = midx + dist * xx1;
                double Ry = midy + dist * yy1;
                int tmp = check(points, Rx, Ry, (double)r);
                ans = Math.max(ans,tmp);
                tmp = check(points, midx - dist * xx1, midy - dist * yy1, (double)r);
                ans = Math.max(ans,tmp);
            }
        }
        return ans==0?1:ans;
    }
}
{% endcodeblock %}
