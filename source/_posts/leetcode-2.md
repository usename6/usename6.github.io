---
title: leetcode-2
date: 2021-08-16 15:42:40
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode hard练习2。
心得：学会了java的upper_bound的写法，第一次用暴力过leetcode hard。
<!--more-->
leetcode 1960
题意：给出一串字符串，找到两个互不相交字符串，使其长度的乘积最大。
题解：
法一：暴力 双模hash + DP
正向预处理hash(双模hash,防止hash冲突)，反向预处理hash(同样也是双模hash)。
定义两个DP数组：
其中$dp1[i]$表示以i为结尾的前缀中有最长的回文子串的长度。
而$dp2[i]$表示以i为开头的后缀中的最长回文子串的长度。
枚举中点，二分回文串的长度，更新dp1和dp2的初始值。
然后更新：
$$ dp1[i] = max(dp1[i], dp1[i-1]) $$
$$ dp1[i] = max(dp1[i], dp[i+1] - 2) $$
$$ dp2[i] = max(dp2[i], dp2[i+1]) $$
$$ dp2[i] = max(dp2[i], dp2[i-1]-2) $$
code:
{% codeblock lang:Java %}
class Hash{
    ArrayList<Integer> h1P, h1S;
    ArrayList<Integer> h2P, h2S;
    ArrayList<Integer> b1, b2;
    static long mod1 = 1000000007;
    static long mod2 = 1000000009;
    static long base1 = 23;
    static long base2 = 37;
    public Hash(){
        h1P = new ArrayList<>();
        h1S = new ArrayList<>();
        h2P = new ArrayList<>();
        h2S = new ArrayList<>();
        b1 = new ArrayList<>();
        b2 = new ArrayList<>();
    }
    public void init_b(int m){
        long r1 = 1, r2 = 1;
        b1.add((int)r1);
        b2.add((int)r2);
        for(int i=0;i<m+1;i++){
            r1 = (r1*base1)%mod1;
            b1.add((int)r1);
            r2 = (r2*base2)%mod2;
            b2.add((int)r2);
        }
    }
    public void cal_hash(String s){
        int len = s.length();
        //prefix hash
        h1P.add(0);
        h2P.add(0);
        for(int i=0;i<len;i++){
            long data1 = (h1P.get(i)*base1+s.charAt(i)-'a')%mod1;
            h1P.add((int)data1);
            long data2 = (h2P.get(i)*base2+s.charAt(i)-'a')%mod2;
            h2P.add((int)data2);
        }
        //suffix hash
        h1S.add(0);
        h2S.add(0);
        int cnt = 0;
        for(int i=len-1;i>=0;i--){
            long data1 = (h1S.get(cnt)*base1+s.charAt(i)-'a')%mod1;
            h1S.add((int)data1);
            long data2 = (h2S.get(cnt)*base2+s.charAt(i)-'a')%mod2;
            h2S.add((int)data2);
            cnt++;
        }
        //System.out.println("h1.size = " + h1.size());
        //System.out.println("h2.size = " + h2.size());
    }
    public long gethash1P(int l,int r){
        return ((long)h1P.get(r)-
                    (long)h1P.get(l-1)*b1.get(r-l+1)%mod1+mod1)%mod1;
    }
    public long gethash1S(int l,int r){
        return ((long)h1S.get(r)-
                    (long)h1S.get(l-1)*b1.get(r-l+1)%mod1+mod1)%mod1;
    }
    public long gethash2P(int l,int r){
        return ((long)h2P.get(r)-
                    (long)h2P.get(l-1)*b2.get(r-l+1)%mod2+mod2)%mod2;   
    }
    public long gethash2S(int l,int r){
        return ((long)h2S.get(r)-
                    (long)h2S.get(l-1)*b2.get(r-l+1)%mod2+mod2)%mod2;   
    }
}
class Solution {
    public boolean check(int s, int mid, Hash hash, int len){
        //System.out.println("["+(s-mid)+","+(s+mid)+"]");
        if(hash.gethash1P(s-mid,s-1)==hash.gethash1S(len-(s+mid)+1,(len-s)) &&
            hash.gethash2P(s-mid,s-1)==hash.gethash2S(len-(s+mid)+1,(len-s)))
            return true;
        return false;
    }
    public long maxProduct(String s) {
        if(s.length()==2)return 1;
        //step 1. init hash
        Hash hash = new Hash();
        hash.init_b(s.length());
        hash.cal_hash(s);
        int[] dp1 = new int[100005];
        int[] dp2 = new int[100005];
        dp1[0] = dp1[s.length()-1] = 1;
        dp2[0] = dp2[s.length()-1] = 1;
        //step 2. find longest 
        int n = s.length();
        for(int i=1;i<s.length()-1;i++){
            dp1[i] = Math.max(1,dp1[i]);
            dp2[i] = Math.max(1,dp2[i]);
            int l=1,r=Math.min(s.length()-i-1, i);
            int ans=0;
            while(l<r){
                int mid=(l+r)>>1;
                if(this.check(i+1, mid, hash, n)){
                    ans=mid;
                    l=mid+1;
                }
                else r=mid-1;
            }
            if(l==r&&this.check(i+1, l, hash, n))ans=l;
            //if(ans>0)System.out.println("mid = "+ i);
            //if(ans>0)System.out.println("ans = " + ans);
            dp1[i+ans]=Math.max(2*ans+1,dp1[i+ans]);
            //if(ans>0)System.out.println("dp1["+(i+ans)+"] = " + dp1[i+ans]);
            dp2[i-ans]=Math.max(2*ans+1,dp2[i-ans]);
            //if(ans>0)System.out.println("dp2["+(i-ans)+"] = " + dp2[i-ans]);
        }
        //step 3. DP
        for(int i=s.length()-2;i>0;i--){
            dp1[i]=Math.max(dp1[i+1]-2,dp1[i]);
        }
        for(int i=1;i<s.length();i++){
            dp2[i]=Math.max(dp2[i-1]-2,dp2[i]);
        }
        for(int i=1;i<s.length();i++){
            //System.out.println("pre:dp1["+i+"] = " + dp1[i]);
            dp1[i]=Math.max(dp1[i],dp1[i-1]);
            dp1[i]=Math.max(dp1[i],1);
            //System.out.println("dp1["+i+"] = " + dp1[i]);
        }
        for(int i=s.length()-1;i>=0;i--){
            dp2[i]=Math.max(dp2[i],dp2[i+1]);
            dp2[i]=Math.max(dp2[i],1);
            //System.out.println("dp2["+i+"] = " + dp2[i]);
        }
        long res=0;
        for(int i=0;i<s.length()-1;i++){
            long tmp=(long)dp1[i]*(long)dp2[i+1];
            res=Math.max(res,tmp);
        }
        return res;
    }
}
{% endcodeblock %}

leetcode 1964
题意：不想说了，阅读理解，就是nlogn的最长上升子序列。
题解：最长上升子序列。其中arr的数组维护长度为i的最小元素的大小。
code:
{% codeblock lang:Java %}
class Solution {
    public int upper_bound(int []arr,int l,int r,int k){
        int ans=r+1;
        //if(r==3)System.out.println("[" + l + "," + r + "]");
        //if(r==3)System.out.println("k = " + k);
        while(l<r){
            int mid=(l+r)>>1;
            //if(r==3)System.out.println("In: [" + l + "," + r + "]");
            //if(r==3)System.out.println("arr[" + mid +"] = " +arr[mid]);
            if(arr[mid]>k){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(l==r&&arr[l]>k)
            ans=l;
        return ans;
    }
    public int upper_bound2(int[] arr,int l,int r,int k){
        int ans=r+1;
        while(l<r){
            int mid=(l+r)>>1;
            if(arr[mid]<k){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(l==r&&arr[l]<k)ans=l;
        return ans;
    }
    public int[] longestObstacleCourseAtEachPosition(int[] obstacles) {
        int n = obstacles.length;
        int[] dp = new int[obstacles.length];
        int[] arr = new int[obstacles.length+1];
        int len=1;
        arr[1] = obstacles[0];
        dp[0] = 1;
        for(int i=1;i<n;i++){
            int id = upper_bound(arr,1,len,obstacles[i]);
            len = Math.max(len,id);
            arr[id] = obstacles[i];
            dp[i] = id;
            //System.out.println("arr["+id+"] = " + arr[id]);
            //System.out.println("dp["+i+"] = " +dp[i]);
        }
        //System.out.println("len = " + len);
        int[] dp2 = new int[n+1];
        int[] arr2 = new int[n+1];
        arr2[1] = obstacles[n-1];
        len=1;
        for(int i=n-2;i>=0;i--){
            int id = upper_bound2(arr2, 1, len, obstacles[i]);
            if(len!=id)len=id;
            arr2[id]=obstacles[i];
            dp2[i]=id;
            //System.out.println("dp2["+i+"] = " +dp2[i]);
        }
        int[] ans = new int[n];
        for(int i=0;i<n;i++){
            ans[i] = dp[i] + dp2[i] -1;
        }
        return dp;
    }
}
{% endcodeblock %}
这里值得一提，讲一道好题。是上题的plus版本。就是必须包含数组的第i个元素的最长子序列。
这里返回ans就行。
和上一题一样，正着处理和反着处理。
这里值得一提的是upper_bound的写法可以作为模板。

leetcode 1955
题意：0...0  1..1 2...2这样序列是正规序列，给一个数组，问正规序列的个数。
题解：傻逼题，见代码。
code:
{% codeblock lang:Java %}
class Solution {
    public int countSpecialSubsequences(int[] nums) {
        int n = nums.length;
        int[][] dp = new int[n+1][3];
        for(int i=0;i<n;i++){
            dp[i+1][0] = dp[i][0] + (nums[i]==0?0:1) * (dp[i][0] + 1);
            dp[i+1][1] = dp[i][1] + (nums[i]==1?0:1) * (dp[i][1] + 1 + dp[i][0]);
            dp[i+1][2] = dp[i][2] + (nums[i]==2?0:1) * (dp[i][2] + 1 + dp[i][1]);
            //System.out.println("dp["+(i+1)+"][0]"+)
        }
        return dp[n][2];
    }
}
{% endcodeblock %}