---
title: leetcode-8
date: 2021-10-24 18:51:24
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode Train 8
前行是孤独的。
<!--more-->
leetcode 753
题意:
有一个需要密码才能打开的保险箱。密码是 n 位数, 密码的每一位是 k 位序列 0, 1, ..., k-1 中的一个 。
你可以随意输入密码，保险箱会自动记住最后 n 位输入，如果匹配，则能够打开保险箱。
举个例子，假设密码是 "345"，你可以输入 "012345" 来打开它，只是你输入了 6 个字符.
请返回一个能打开保险箱的最短字符串。
题解:
题目可以等价于如何在一个最短的串内枚举所有的n位k进制数排列。而这种序列称之为de Bruijn序列。
为啥这个题，我们可以建模成欧拉回路呢？
首先得明白欧拉回路的定义:我可以在满足条件的有向图or无向图中找到一条这样的回路可以经过这样的图所有的边，且仅仅经过一次。
可能到这还是没有明白为什么？我们可以思考一下当n=2，k=2时，我们建出来的有向图是什么样的？
![alt](/images/leetcode-8/1.png)
我们可以发现如果我们用前一个节点的值加$0,...,k-1$的数字就可以唯一的表示上述的一条边。而且注意到通过这样的表示，我们将可以用边表示所有n+1位的K进制数。
于是问题等价于，给定n和k，找到n-1和k的任意一条欧拉回路即可。
于是我们将采用Hierholzer算法求解这样一条欧拉回路。
下面代码没搞懂，但是可以讲解一下Hierholzer算法的基本原理：
如果我们从u->...v...->u
此时是个回路，当dfs溯回时，此时溯回到最近的点v，从点v再次出发:
v->...t...->v
同样也回到了v。
如果我们把上述路径嵌入到u->...v...->u中，不断组合就会是答案。
可能问题就是如何嵌入到其中，这里利用了尾插法的思想。我们发现我们在不断嵌入的过程中，最后的路径一定是被确定，也就是v..->u这段路径一定是在最后出现的。同理t...->v一定是第二个出现的。
所以我们只需要用尾插法即可。当溯回的时候，我们将边压入栈中。
最后将栈中的字符串取出。重新组合即可。
（**到时候会用C++更新一个栈的写法**）
code:
{% codeblock lang:Java %}
class Edge{
    int v,next,w;
}
class Solution { 
    StringBuffer ans = new StringBuffer();
    Set<Integer> vis = new HashSet<Integer>();
    public void dfs(int cur,int nn,int k){
        for(int i=0;i<k;i++){
            int tmp=cur*10+i;
            if(!vis.contains(tmp)){
                vis.add(tmp);
                dfs(tmp%nn,nn,k);
                ans=ans.append(i);
            }
        }
    }
    public String crackSafe(int n, int k) {
        int nn = (int) Math.pow(10, n - 1);
        dfs(0,nn,k);
        for(int i=1;i<n;i++)ans.append("0");
        return ans.toString();
    }
}
{% endcodeblock %}
leetcode 327
题意:
给你一个整数数组 nums 以及两个整数 lower 和 upper 。求数组中，值位于范围 [lower, upper] （包含 lower 和 upper）之内的 区间和的个数 。
区间和 S(i, j) 表示在 nums 中，位置从 i 到 j 的元素之和，包含 i 和 j (i ≤ j)。
题解:
离散化权值树状数组傻逼题。
code：
{% codeblock lang:Java %}
class BIT{
    int[] tree;
    int n;
    public BIT(int nn){
        n=nn;
        tree=new int[n];
    }
    private int lowbit(int x){
        return x&(-x);
    }
    public void update(int x,int d){
        while(x<=n){
            tree[x]+=d;
            x+=lowbit(x);
        }
    }
    public int query(int x){
        int res=0;
        while(x>0){
            res+=tree[x];
            x-=lowbit(x);
        }
        return res;
    }
}
class Solution {
    long[] sum;
    public int lower_bound(long[] arr,int l,int r,long x){
        int ans=r;
        while(l<r){
            int mid=(l+r)>>1;
            if(arr[mid]>=x){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(arr[l]>=x)ans=l;
        return ans;
    }
    public int countRangeSum(int[] nums, int lower, int upper) {
        int n=nums.length;
        sum=new long[n+1];
        sum[0]=0;
        for(int i=1;i<=n;i++){
            sum[i]=sum[i-1]+(long)nums[i-1];
        }
        Arrays.sort(sum);
        for(int i=0;i<=n;i++)System.out.print(sum[i]+" ");
        System.out.println("");
        BIT bit = new BIT(n+3);
        long tot=0;
        int ans=0;
        for(int i=0;i<n;i++){
            tot=tot+(long)nums[i];
            int id=lower_bound(sum,0,n,tot)+1;
            System.out.println("tot="+tot);
            System.out.println("id="+id);
            bit.update(id,1);
            int right=lower_bound(sum,0,n,tot-lower);
            int left=lower_bound(sum,0,n,tot-upper);
            System.out.println("Cur: ["+(tot-lower)+","+(tot-upper)+"]");
            if(sum[left]>=tot-upper)left--;
            System.out.println("["+(left+1)+","+(right+1)+"]");
            int tmp=bit.query(right+1)-bit.query(left+1);
            System.out.println("tmp="+tmp);
            ans=ans+bit.query(right+1)-bit.query(left+1);
        }
        return ans;
    }
}
{% endcodeblock %}
leetcode 1889
题意:
给你 n 个包裹，你需要把它们装在箱子里，每个箱子装一个包裹。总共有 m 个供应商提供 不同尺寸 的箱子（每个规格都有无数个箱子）。如果一个包裹的尺寸 小于等于 一个箱子的尺寸，那么这个包裹就可以放入这个箱子之中。

包裹的尺寸用一个整数数组 packages 表示，其中 packages[i] 是第 i 个包裹的尺寸。供应商用二维数组 boxes 表示，其中 boxes[j] 是第 j 个供应商提供的所有箱子尺寸的数组。

你想要选择 一个供应商 并只使用该供应商提供的箱子，使得 总浪费空间最小 。对于每个装了包裹的箱子，我们定义 浪费的 空间等于 箱子的尺寸 - 包裹的尺寸 。总浪费空间 为 所有 箱子中浪费空间的总和。

比方说，如果你想要用尺寸数组为 [4,8] 的箱子装下尺寸为 [2,3,5] 的包裹，你可以将尺寸为 2 和 3 的两个包裹装入两个尺寸为 4 的箱子中，同时把尺寸为 5 的包裹装入尺寸为 8 的箱子中。总浪费空间为 (4-2) + (4-3) + (8-5) = 6 。
请你选择 最优 箱子供应商，使得 总浪费空间最小 。如果 无法 将所有包裹放入箱子中，请你返回 -1 。由于答案可能会 很大 ，请返回它对 1e9 + 7 取余 的结果。
题解:
贪心+二分+前缀和。
可以想一个非常暴力的解法：预处理排序每一个供应商的箱子，并排序包裹。首先枚举供应商，我们去维护两个指针，一个指针指向包裹，另一个指针指向箱子。对于每个箱子，我们检查当前的包裹是否小于这个箱子，小于则把丢进这个箱子（根据贪心，这肯定是最优的）。然后移动包裹的指针，否则我们移动箱子的指针，并加上这个箱子的贡献值。这样的复杂度是O(nm+l)。
但是注意到这其实还可以优化，因为我们可以找到每个箱子对应的包裹区间，这样就避免了指针一个一个的移动。那么就用二分即可。
code:
{% codeblock lang:Java %}
class Solution {
    public int find(int[] arr, int l, int r, int k){
        int ans=l;
        while(l<r){
            int mid=(l+r)>>1;
            if(arr[mid]<=k){
                ans=mid;
                l=mid+1;
            }
            else r=mid-1;
        }
        if(arr[l]<=k)ans=l;
        return ans;
    }
    public int minWastedSpace(int[] packages, int[][] boxes) {
        int n=packages.length;
        long[] sum=new long[n+1];
        sum[0]=0;
        Arrays.sort(packages);
        for(int i=0;i<n;i++)sum[i+1]=sum[i]+packages[i];
        int m=boxes.length;
        long ans=1000000000000L;
        for(int i=0;i<m;i++){
            Arrays.sort(boxes[i]);
            int l=0,r=n-1;
            long tmp=0;
            for(int j=0;j<boxes[i].length;j++){
                if(l>r)break;
                //System.out.print(boxes[i][j]+" ");
                int tt=find(packages,l,r,boxes[i][j]);
                //System.out.println("tt="+tt);
                if(packages[tt]>boxes[i][j])continue;
                tmp=tmp+(long)(tt-l+1)*boxes[i][j]-(sum[tt+1]-sum[l]);
                l=tt+1;
            }
            if(l>r)ans=Math.min(ans,tmp);
            //System.out.println("");
        }
        System.out.println("ans="+ans);
        long mod=1000000007;
        return ans==1000000000000L?-1:(int)(ans%mod);
    }
}
{% endcodeblock %}
leetcode 1851
题意:
给你一个二维整数数组 intervals ，其中 intervals[i] = [lefti, righti] 表示第 i 个区间开始于 lefti 、结束于 righti（包含两侧取值，闭区间）。区间的 长度 定义为区间中包含的整数数目，更正式地表达是 righti - lefti + 1 。
再给你一个整数数组 queries 。第 j 个查询的答案是满足 lefti <= queries[j] <= righti 的 长度最小区间 i 的长度 。如果不存在这样的区间，那么答案是 -1 。
以数组形式返回对应查询的所有答案。
题解:
傻逼线段树，lazy标记搞搞。真的很shit了，很不喜欢写这种长代码。离散化的技巧。
code:
{% codeblock lang:Java %}
class Node{
    int l,r;
    int val,len;
    boolean lazy;
}
class SegTree{
    Node[] tree;
    public SegTree(int len){
        tree = new Node[(len+1)<<3];
        build(1,1,len);
    }
    public void pushdown(int rt){
        if(tree[rt].lazy){
            if(tree[rt<<1].val==-1||tree[rt<<1].val>tree[rt].val){
                tree[rt<<1].len=tree[rt<<1].val=tree[rt].val;
                tree[rt<<1].lazy=true;
            }
            if(tree[rt<<1|1].val==-1||tree[rt<<1|1].val>tree[rt].val){
                tree[rt<<1|1].len=tree[rt<<1|1].val=tree[rt].val;
                tree[rt<<1|1].lazy=true;
            }
            tree[rt].lazy=false;
        }
    }
    public void pushup(int rt){
        if(tree[rt].len>tree[rt<<1].len){
            tree[rt].len=tree[rt<<1].len;
            tree[rt].val=tree[rt<<1].val;
        }
        if(tree[rt].len<tree[rt<<1|1].len){
            tree[rt].len=tree[rt<<1|1].len;
            tree[rt].val=tree[rt<<1|1].val;
        }
    }
    public void build(int rt,int l,int r){
        //System.out.println("tree["+rt+"]="+"("+l+","+r+")");
        tree[rt]=new Node();
        tree[rt].lazy=false;
        tree[rt].l=l;tree[rt].r=r;
        tree[rt].val=-1;tree[rt].len=0;
        if(l==r)return;
        int mid=(l+r)>>1;
        build(rt<<1,l,mid);
        build(rt<<1|1,mid+1,r);
        pushup(rt);
    }
    public void update(int rt,int l,int r,int len){
        //System.out.println("update:"+"("+l+","+r+")");
        if(tree[rt].l==l&&tree[rt].r==r){
            //System.out.println("cur val:"+tree[rt].val+", cur len="+tree[rt].len+", len = "+len);
            if((tree[rt].len==0)||(tree[rt].len>len)){
                tree[rt].val=tree[rt].len=len;
                //System.out.println("updated:("+tree[rt].l+","+tree[rt].r+"):"+tree[rt].val+","+tree[rt].len);
                tree[rt].lazy=true;
            }
            return;
        }
        int mid=(tree[rt].l+tree[rt].r)>>1;
        if(r<=mid)update(rt<<1,l,r,len);
        else if(l>mid)update(rt<<1|1,l,r,len);
        else{
            update(rt<<1,l,mid,len);
            update(rt<<1|1,mid+1,r,len);
        }
    }
    public int query(int rt,int l,int r){
        //System.out.println("query:("+tree[rt].l+","+tree[rt].r+") = "+tree[rt].val);
        if(tree[rt].l==l&&tree[rt].r==r)return tree[rt].val;
        pushdown(rt);
        int mid=(tree[rt].l+tree[rt].r)>>1;
        if(l<=mid)return query(rt<<1,l,r);
        return query(rt<<1|1,l,r);
    }
}
class Solution {
    public int[] minInterval(int[][] intervals, int[] queries) {
        int n = queries.length;
        Set<Integer> Numbers = new TreeSet<Integer>();
        for(int x:queries)Numbers.add(x);
        for(int[] interval:intervals){
            int l=interval[0],r=interval[1];
            Numbers.add(l);Numbers.add(r);
        }
        int len=Numbers.size();
        //System.out.println("len="+len);
        Map<Integer,Integer> values = new HashMap<Integer,Integer>();
        int idx=1;
        for(int x:Numbers){
            //System.out.println(x+"------>"+idx);
            values.put(x,idx);
            idx++;
        }
        SegTree seg = new SegTree(len);
        for(int[] interval:intervals){
            int l=values.get(interval[0]),r=values.get(interval[1]);
            seg.update(1,l,r,interval[1]-interval[0]+1);
        }
        int[] ans=new int[n];
        for(int i=0;i<n;i++){
            int id=values.get(queries[i]);
            ans[i]=seg.query(1,id,id);
            //System.out.println("");
        }
        return ans;
    }
}
{% endcodeblock %}