---
title: leetcode-4
date: 2021-08-31 16:03:29
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode 练习第四周。
题目难度很大,因为有点事情拖到今天写blog。
<!--more-->
leetcode 1675
题意：
你有一个长度为n的数组nums。这个数组存在着任意**大于2**正整数。然后你可以做以下两个操作：
a.如果元素是偶数，可以除以2，
b.如果元素是奇数，可以乘以2。
我们定义数组的偏移量是数组里所有最大的元素和最小元素的差值。
问经过若干次操作后，我们可以拥有的最小偏移量是多少？
题解：
因为有两个操作，总是让我们很烦恼。于是我们可以做先将数组里的所有元素转化为偶数。
这样，我们只需要进行一个操作。于是决策就变得很简单了。因为我只能对这个数组的元素
进行缩小操作，于是我们只需要贪心地对最后一个元素做缩小操作即可。
考察点：Java优先队列编写（heatmap）
code:
{% codeblock lang:Java %}
class Solution {
    public int minimumDeviation(int[] nums) {
        //离散化
        int n=nums.length;
        PriorityQueue<Integer> maxheap = new PriorityQueue<Integer>(n, 
            new Comparator<Integer>() {
                @Override
                public int compare(Integer o1, Integer o2) {
                    return o2.compareTo(o1);
                }
        });
        int[] tots = new int[n];
        int mm=1000000007;
        for(int i=0;i<n;i++){
            if(nums[i]%2==1){
                maxheap.offer(nums[i]*2);
                mm=Math.min(mm,nums[i]*2);
            }
            else{
                maxheap.offer(nums[i]);
                mm=Math.min(mm,nums[i]);
            }
        }
        Arrays.sort(tots);
        int ans=1000000007;
        while(!maxheap.isEmpty()){
            int tp = maxheap.poll();
            //System.out.println("tp = "+tp);
            ans=Math.min(ans,tp-mm);
            //System.out.println("ans = "+ans);
            if(tp%2==0){
                maxheap.offer(tp/2);
               
                mm=Math.min(mm,tp/2);
            }
            else break;
        }
        return ans;
    }
}
{% endcodeblock %}

leetcode 1808
题意：
给一个正整数$n$，你要在因子个数为$[1,n]$范围内找到一个含有最多好因子的数，
并且返回最多好因子数。
好因子数是一个为$n$的数能够整除它所有的质因子。我们需要找到一个数
使得他的因子是好因子数的个数最多。
题解：
我这个题就打了个表找了规律。
然后快速幂解决掉了。

首先这个题可以转化为给出一个正整数m，拆分成n个正整数之和，
求他们的最大积
这个题可以认真用数学推导一下：
$$\frac{a_1+a_2+...+a_n}{n} >= \sqrt[n]{a_{1} * a_{2} * ... * a_{n}}$$
很显然，在$a_1 = a_2 = ... = a_n$的时候，等式成立。且此时取得最大值。
若取$a_1=x$，那么x和最大值关系为：
$$f(x) = x^{\frac{n}{x}}$$
求导后（取对数再求导即可。）
得到$x=e$的时候，此时f(x)的值最大。
故很显然分成3的时候最好。
快速幂即可。
code:
{% codeblock lang:Java %}
class Solution {
    long mod=1000000007;
    public long qpow_mod(long a,long b){
        long res=1;
        while(b>0){
            if(b%2==1)res=res*a%mod;
            a=a*a%mod;
            b>>=1;
        }
        return res;
    }
    public int maxNiceDivisors(int primeFactors) {
        if(primeFactors<3){
            return primeFactors;
        }
        int n=primeFactors;
        long res;
        if(n%3==1){
            int a=3,b=n/3-1;
            res=qpow_mod((long)a,(long)b)*4%mod;
        }
        else if(n%3==2){
            int a=3,b=n/3;
            res=qpow_mod((long)a,(long)b)*2%mod;
        }
        else{
            int a=3,b=n/3;
            res=qpow_mod((long)a,(long)b)%mod;
        }
        return (int)res;
    }
}
{% endcodeblock %}


leetcode 1879
题意：
给两个数组nums1和nums2。可以从中任意排列nums2.求最大异或和。
题解：
傻逼状压。
定义状态：dp[i]
code:
dp[i]表示选了nums2的子集。
状态转移方程：
$$dp[(1<<k)|j] = min(dp[1<<k|j],dp[j])$$
然后做一个滚动数组优化。
值得注意：如何调试状压DP。
{% codeblock lang:Java %}
class Solution {
    String DebugOutput(int now,int n){
        String ans="";
        for(int i=0;i<n;i++){
            if((now&(1<<i))!=0)ans=ans+"1";
            else ans=ans+"0";
        }
        return ans;
    }
    public int minimumXORSum(int[] nums1, int[] nums2) {
        int n = nums1.length;
        int[][] dp = new int[(1<<n)][2];
        dp[0][0]=1000000007;
        //init
        for(int i=0;i<n;i++)dp[1<<i][0]=nums1[0]^nums2[i];
        //DP
        for(int i=1;i<n;i++){
            for(int j=0;j<(1<<n);j++)dp[j][(i%2)]=1000000007;
            for(int j=0;j<(1<<n);j++){
                for(int k=0;k<n;k++){
                    if(((1<<k)&j)==0){
                        //System.out.println("k="+k);
                        //System.out.println("Pre: dp["+DebugOutput(j,n)+"]["+(i-1)+"] = " + dp[j][(i%2)^1]);
                        dp[(1<<k)|j][i%2] = Math.min(dp[(1<<k)|j][i%2], dp[j][(i%2)^1] + (nums1[i]^nums2[k]));
                        //System.out.println("cur = " + (nums1[i]^nums2[k]));
                        //System.out.println("cur2 = " + dp[j][(i%2)^1]);
                        //System.out.println("tot = " + (dp[j][(i%2)^1] + nums1[i]^nums2[k]));
                        //System.out.println("After: dp["+DebugOutput((1<<k)|j,n)+"]["+i+"] = " + dp[(1<<k)|j][i%2]);
                    }
                }
            }
        }
        return dp[(1<<n)-1][(n-1)%2];
    }
}
{% endcodeblock %}

leetcode 1977
题意：
你写下了若干 正整数 ，并将它们连接成了一个字符串 num 。但是你忘记给这些数字之间加逗号了。你只记得这一列数字是 非递减 的且 没有 任何数字有前导 0 。

请你返回有多少种可能的 正整数数组 可以得到字符串 num 。由于答案可能很大，将结果对 $10^9 + 7$ 取余 后返回。
题解：
有点困难的DP，首先考虑最简单的DP.
定义状态：dp[i][j]表示最后一个数字是从i开头，j结尾的合法方案数。
那么转移：
$$if num(k,i) <= num(i,j) then$$
$$dp[i][j] = (dp[i][j] + dp[k][i]) \mod MOD$$
这样看起来转移我们只需要枚举k，这样复杂度似乎是O(n^3)的。
首先我们假定if的条件是永远可以被满足的。由于我们转移依赖于k，而k是满足$k<=i$的。
那么我们可以做第一个优化，枚举优化。即我们维护一个前缀和：
$$pre[i][j] = \sum_{k=1}^i{pre[k][j]} $$
那么转移方程被转化为：
$$dp[i][j] = (dp[i][j] + pre[i-1][i-1] - pre[2*i-j-2][i-1]) \mod MOD$$
但是这并不是真实情况的。那么我们就需要比较$num(k,i)$和$num(i,j)$。那么怎么比较呢？
首先分类讨论一下：
如果$$len(k,i) < len(i,j)$$，显然满足条件。
而如果$$len(k,i) > len(i,j)$$，显然不满足条件，这也算我们DP的边界条件。
那么$$len(k,i) = len(i,j)$$我们似乎又得扫描一遍数位进行比较，这个复杂度又是O(n)。
为了能够O(1)判断，于是我们再去维护一个东西。因为我们发现，对于长度相同的数字，
我们只需要比较它们的非公共部分即可。所以我们只需要找到num(i,j)和num(k,i)第一次
出现非公共部分的地方然后O(1)比较就行。
这时候第一个从我们脑海中闪现的就是最大公共子串。
于是我们还需要再维护一个lcp，$lcp[i][j]$表示以i为开头的后缀和以j为开头的后缀最大公共子串长度。
其转移方程为: 
$$if num[i] == num[j] then$$
$$lcp[i][j] = lcp[i+1][j+1]$$
于是对于$len(i,j) == len(k,i)$的情况，我们只需要考虑num[i+lcp[k][i]]和num[k+lcp[k][i]]的情况即可。
转移方程变化为两段式：
$$dp[i][j] = (dp[i][j] + pre[i-1][i-1] - pre[2*i-j-1][i-1]) \mod MOD$$
$$ if num(i+lcp[2*i-j-1][i]) > num(2*i-j-1+lcp[2*i-j-1][i]) then$$
$$dp[i][j] = (dp[i][j] + pre[2*i-j-1][i-1]-pre[2*i-j-2][i-1]) \mod MOD$$
答案是pre[n][n]

code:
{% codeblock lang:Java %}
class Solution {
    int mod = 1000000007;
    public int numberOfCombinations(String num) {
        if(num.charAt(0)=='0')return 0;
        int n=num.length();
        //pre LCP
        int[][] lcp = new int[n+2][n+2];
        for(int i=n;i>=1;i--){
            for(int j=n;j>=1;j--){
                if(num.charAt(i-1)==num.charAt(j-1)){
                    lcp[i][j]=lcp[i+1][j+1]+1;
                }
            }
        }
        //pre init DP
        int[][] dp = new int[n+1][n+1];
        int[][] pre = new int[n+1][n+1];
        for(int i=1;i<=n;i++)dp[1][i]=pre[1][i]=1;
        //System.out.println("n="+n);
        for(int j=2;j<=n;j++){
            for(int i=2;i<=j;i++){
                if(num.charAt(i-1)=='0'){
                    pre[i][j]=(pre[i-1][j]+dp[i][j])%mod;
                    continue;
                }
                dp[i][j]=(dp[i][j]+pre[i-1][i-1]-pre[Math.max(2*i-j-1,0)][i-1])%mod;
                //System.out.println("pre1 = " + (Math.max(2*i-j-1,0)));
                if(2*i-j-1>0){
                    int tag=0;
                    //System.out.println("("+(2*i-j-1)+","+i+")");
                    int tmp = lcp[2*i-j-1][i];
                    if(tmp>=j-i+1)tag=1;
                    else if(tmp<j-i+1&&num.charAt(2*i-j-1+tmp-1)<num.charAt(i+tmp-1))tag=1;
                    dp[i][j]=(dp[i][j]+(pre[2*i-j-1][i-1]-pre[2*i-j-2][i-1])*tag)%mod;
                    //System.out.println("pre2 = " + (pre[2*i-j-1][j-1]-pre[2*i-j-2][j-1])*tag);
                }
                pre[i][j]=(pre[i-1][j]+dp[i][j])%mod;
                //System.out.println("dp["+i+"]"+"["+j+"] = "+dp[i][j]);
            }
        }
        return pre[n][n];
    }
}
{% endcodeblock %}


leetcode 1982
题意：
存在一个未知数组需要你进行还原，给你一个整数 n 表示该数组的长度。另给你一个数组 sums ，由未知数组中全部 $2^n$ 个 子集的和 组成（子集中的元素没有特定的顺序）。

返回一个长度为 n 的数组 ans 表示还原得到的未知数组。如果存在 多种 答案，只需返回其中 任意一个 。

如果可以由数组 arr 删除部分元素（也可能不删除或全删除）得到数组 sub ，那么数组 sub 就是数组 arr 的一个 子集 。sub 的元素之和就是 arr 的一个 子集的和 。一个空数组的元素之和为 0 。

题解：
这个题非常的难。我们首先思考一个最好搞的情况。
如果ans全是正整数，那么这里前缀和一定具有单调递增的特性。这里我们可以知道如果我们每次从arr取出最小两个数
分别记为$x$和$y$,其中$y>x$,那么我们可以知道:
$$d = y - x$$
很显然d是ans里一个最小的一个元素。然后我们可以把x删掉，然后再去重复上述操作。直到结束。

然后，我们将上述条件拓展到既有正整数也有负整数的情况。那么这时我们取出来的x和y，很有可能存在两个情况：
第一种情况：
$$ x + a_i = y , a_i >= 0 $$
第二种情况：
$$ y + a_i = x , a_i < 0 $$
那么此时我们就需要判断应该让$a_i$取什么值合法。这里为了方便讨论，我们定义:$d = y - x$。
于是我们用d将数组进行切分：分成两份arr1和arr2，其中arr1和arr2满足以下关系：
$$\forall i, \exists j, arr2[i] = arr[j] + d$$
经过arr1或arr2其中有个数组是除了$a_i$序列和子集。
首先arr是存在0,假设分开后的数组只存在这样的情况：0只在arr1或arr2中出现。
那么就有两种情况：
(1)0在arr1中出现。那么就说明$a_i=d$，那么接下来剩余的值我们只需要在arr1中找。
(2)0在arr2中出现。那么就说明$a_i=-d$，那么接下来剩余的值我需要在arr2中找。
我们通过这样的方式迭代，一定能够得到最后的结果。

但是如果arr1和arr2两边都存在0.那么该怎么选。这里可以给出结论，不论选择怎么样的情况都可以。
这可以证明这是等价的。
证明：
因为有两个0，必然存在两个情况：
情况1：
0是空集的序列的和。
情况2：
$$o_1 + o_2 + ... + o_k + o_{k+1} = 0$$
假设我们认为真正的0存在于arr1中，那么显然我们就先选arr1。如果真正的0不存在arr1中，那么我们
的选择还是否正确。
如果真正的0不存在arr1中，我们还选了arr1,就相当于把$$o_1,o_2,...,o_{k+1}$$全部变成了负数。
这里为了方便讨论我们假设:
$$o_{k+1} = a_{i}$$
那么我们需要假设这样情况形成的序列ans是否还满足arr。
我们这里定义一个A，其中A满足：
$$A = \sum_{i \in O}a_i$$
那么我这里$\forall i \in [1,n]$，必然满足$sum[i]-A \in sum$
故证明完毕。
code:
{% codeblock lang:Java %}
class Solution {
    int cc;
    int[] ans=null;
    void dfs(int n,int[] sums){
        if(n==1)return;
        int[] s = new int[n/2];
        int[] t = new int[n/2];
        int cnt=0,left=0,right=0;
        boolean[] vis = new boolean[n];
        int d=sums[1]-sums[0];
        boolean flag=false;
        while(cnt<n){
            while(left<n&&vis[left])left++;
            if(left>=n)break;
            vis[left]=true;
            s[cnt]=sums[left];
            if(sums[left]==0)flag=true;
            while(right<n&&(vis[right]||(sums[right]-sums[left]!=d)))right++;
            if(right>=n)break;
            vis[right]=true;
            t[cnt]=sums[right];
            cnt++;
        }
        if(flag){
            ans[cc++]=d;
            dfs(n/2,s);
        }
        else{
            ans[cc++]=-d;
            dfs(n/2,t);
        }
    }
    public int[] recoverArray(int n, int[] sums) {
        Arrays.sort(sums);
        ans = new int[n];
        dfs(sums.length,sums);
        return ans;
    }
}
{% endcodeblock %}
