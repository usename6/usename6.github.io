---
title: leetcode-6
date: 2021-10-17 16:25:50
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode training 6.
To be a better person
<!--more-->
leetcode 2035
题意：
给你一个长度为 2 * n 的整数数组。你需要将 nums 分成 两个 长度为 n 的数组，分别求出两个数组的和，并最小化 两个数组和之 差的绝对值 。nums 中每个元素都需要放入两个数组之一。
题解：
看了这个题的数据范围，n才为15，两个的数组的范围为30左右。这个题艾教曾经讲过，如果数据范围为15，就用状压DP。数据范围为30，就用折半枚举。
这个题明显是用折半枚举，显然需要用到状压的技巧。
我们先预处理其中一个数组的所有的子集和，通过枚举二进制实现枚举所有的子集和。然后我们可以用arr[i]存储当前数组被选了i个元素的所有子集和。并对他们进行排序。
对于另外一个数组，我们去枚举这个数组的所有的子集和，并且用cnt表示当前我在这个数组中选了cnt个元素，那么我们只需要在前一个数组中选择n-cnt的个数即可，为了方便，我们可以将当前枚举的数组的子集和取负，此时只需要在用二分查找其最接近它的值即可。
code:
{% codeblock lang:Java %}
class Solution {
    public int lower_bound(ArrayList<Integer> a, int l,int r,int key){
        //System.out.println("key = "+key);
        int ans=r;
        //for(int i:a)System.out.print(i+" ");
        //System.out.println("");
        while(l<r){
            int mid=(l+r)>>1;
            if(a.get(mid)>=key){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(l==r&&a.get(l)>=key)ans=l;
        if(ans>0&&Math.abs(a.get(ans-1)-key)<Math.abs(a.get(ans)-key)){
            ans--;
        }
        return ans;
    }
    public int minimumDifference(int[] nums) {
        int n = nums.length/2;
        int[] num1 = new int[n];
        int[] num2 = new int[n];
        for(int i=0;i<2*n;i++){
            if(i<n)num1[i]=nums[i];
            else num2[i-n]=nums[i];
        }
        ArrayList<ArrayList<Integer>> arr = new ArrayList<ArrayList<Integer>>();
        for(int i=0;i<=n;i++){
            ArrayList<Integer> tmp = new ArrayList<Integer>();
            arr.add(tmp);
        }
        for(int i=0;i<(1<<n);i++){
            int tmp1=0,tmp2=0,cnt=0;
            for(int j=0;j<n;j++){
                if((i&(1<<j))!=0){
                    tmp1=tmp1+num1[j];
                    cnt++;
                }
                else tmp2=tmp2+num1[j];
            }
            arr.get(cnt).add(tmp1-tmp2);
        }
        for(int i=0;i<n;i++){
            Collections.sort(arr.get(i));
        }
        int ans=1000000000;
        for(int i=0;i<(1<<n);i++){
            int tmp1=0,tmp2=0,cnt=0;
            for(int j=0;j<n;j++){
                if((i&(1<<j))!=0){
                    tmp1=tmp1+num2[j];
                    cnt++;
                }
                else{
                    tmp2=tmp2+num2[j];
                }
            }
           // System.out.println("cnt = "+cnt);
            //System.out.println("r = " + arr.get(cnt).size());
            int id = lower_bound(arr.get(cnt),0,arr.get(cnt).size()-1,tmp1-tmp2);
            ans = Math.min(ans, Math.abs(arr.get(cnt).get(id)+tmp2-tmp1));
        }
        return ans;
    }
}
{% endcodeblock %}

leetcode 1998
题意:
给你一个整数数组 nums ，你可以在 nums 上执行下述操作 任意次 ：

如果 gcd(nums[i], nums[j]) > 1 ，交换 nums[i] 和 nums[j] 的位置。其中 gcd(nums[i], nums[j]) 是 nums[i] 和 nums[j] 的最大公因数。
如果能使用上述交换方式将 nums 按 非递减顺序 排列，返回 true ；否则，返回 false 。
题解：
先抛开gcd的壳子，我们显然可以知道含有交换运算的代数系统一定满足自反性、对称性、传递性。如果A和B可交换，B和C可交换，那么A和C可交换，如果A、B、C不再与其他元素可交换，那么ABC可以在图上建模为一个联通分量，即团。故我们可以用并查集求出所有的团，然后在判断a[i]和a[idx]是否在一个团中，这里idx指的是a数组排序过后a[i]应该处于的位置。可以证明在这样算法的中，我们可以解决元素相同的问题。
值得学习：Java的Euler筛的模板。
code:
{% codeblock lang:Java %}
class Solution {
    int[] f;
    boolean[] vis;
    ArrayList<Integer> p;
    int cnt,n;
    int maxn=100005;
    public int find(int x){
        return x==f[x]?f[x]:(f[x]=find(f[x]));
    }
    public void union_set(int s1,int s2){
        int f1=find(s1),f2=find(s2);
        if(f1!=f2)f[f2]=f1;
    }
    public void Prime(){
        vis[0]=vis[1]=true;
        for(int i=2;i<=maxn;i++){
            if(!vis[i]){
                cnt++;
                p.add(i);
            }
            for(int j=0;j<=cnt&&i*p.get(j)<=maxn;j++){
                vis[i*p.get(j)]=true;
                if(i%p.get(j)==0)break;
            }
        }
    }
    public int lower_bound(int[] arr,int l,int r,int k){
        int ans=r;
        while(l<r){
            int mid=(l+r)>>1;
            if(arr[mid]>=k){
                ans=mid;
                r=mid-1;
            }
            else l=mid+1;
        }
        if(l==r&&arr[l]>=k)ans=l;
        return ans;
    }
    public boolean gcdSort(int[] nums) {
        n=nums.length;
        f = new int[n+1];
        vis = new boolean[maxn+1];
        p = new ArrayList<Integer>();
        cnt = 0;
        Prime();
        int[] mark = new int[cnt+1];
        for(int i=0;i<=cnt;i++)mark[i]=-1;
        int []copy = new int[n];
        for(int i=0;i<n;i++)f[i]=i;
        for(int i=0;i<n;i++){
            int now=nums[i];
            copy[i]=nums[i];
            for(int j=0;j<cnt&&p.get(j)<=now;j++){
                while(now%p.get(j)==0){
                    //System.out.println("num = "+nums[i]+","+"prime = "+p.get(j));
                    now/=p.get(j);
                    if(mark[j]!=-1)union_set(i,mark[j]);
                    mark[j]=i;
                }
            }
        }
        Arrays.sort(copy);
        for(int i=0;i<n;i++){
            int id=lower_bound(copy,0,n-1,nums[i]);
            if(find(i)!=find(id))return false;
        }
        return true;       
    }
}
{% endcodeblock %}

leetcode 2003
题意：
有一棵根节点为 0 的 家族树 ，总共包含 n 个节点，节点编号为 0 到 n - 1 。给你一个下标从 0 开始的整数数组 parents ，其中 parents[i] 是节点 i 的父节点。由于节点 0 是 根 ，所以 parents[0] == -1 。
总共有 105 个基因值，每个基因值都用 闭区间 [1, 105] 中的一个整数表示。给你一个下标从 0 开始的整数数组 nums ，其中 nums[i] 是节点 i 的基因值，且基因值 互不相同 。
请你返回一个数组 ans ，长度为 n ，其中 ans[i] 是以节点 i 为根的子树内 缺失 的 最小 基因值。
节点 x 为根的 子树 包含节点 x 和它所有的 后代 节点。
题解：
启发式合并，每次使用启发式合并，将小的数组copy到大数组里，这样的复杂度是O(nlogn).然后我们再去检查这个集合不存在的元素是多少，事先可以做个优化，因为子树的不存在的值，一定比它的子树不存在的值大，所以可以取出它的子树所有不存在的值的最大值，从这个值开始找起即可。
注意点：树上启发式合并的复杂度证明，以及set集合的使用方法。
code:
{% codeblock lang:Java %}
class Edge{
    int v,next;
}
class Solution {
    Edge[] e = null;
    int tot = 0, n;
    int[] head = null;
    int[] ans = null;
    Set<Integer> dfs(int u,int[] nums){
        if(head[u]==-1){
            if(nums[u]==1)ans[u]=2;
            else ans[u]=1;
            Set<Integer> tmp = new HashSet<>();
            tmp.add(nums[u]);
            return tmp;
        }
        ans[u] = 0;
        Set<Integer> curSet = new HashSet<>();
        //System.out.println("u = "+u);
        curSet.add(nums[u]);
        for(int i=head[u];i!=-1;i=e[i].next){
            int v=e[i].v;
            Set<Integer> tmp = dfs(v, nums);
            ans[u] = Math.max(ans[u], ans[v]);
            if(curSet.size() < tmp.size()){
                tmp.addAll(curSet);
                curSet = tmp;
            }
            else curSet.addAll(tmp);
            /*System.out.println("v = "+v);
            for(Integer s:curSet){
                System.out.print(s+" ");
            }
            System.out.println("");*/
        }
        while(curSet.contains(ans[u]))ans[u]++;
        return curSet;
    }
    void add(int u,int v){
        e[tot] = new Edge();
        e[tot].v = v;
        e[tot].next = head[u];
        head[u] = tot++;
    }
    public int[] smallestMissingValueSubtree(int[] parents, int[] nums) {
        n = nums.length;
        e = new Edge[n+1];
        head = new int[n+1];
        ans = new int[n];
        for(int i=0;i<n;i++)head[i]=-1;
        for(int i=0;i<n;i++){
            if(parents[i]!=-1)add(parents[i],i);
        }
        dfs(0,nums);
        return ans;
    }
}
{% endcodeblock %}

leetcode 810
题意：
黑板上写着一个非负整数数组 nums[i] 。Alice 和 Bob 轮流从黑板上擦掉一个数字，Alice 先手。如果擦除一个数字后，剩余的所有数字按位异或运算得出的结果等于 0 的话，当前玩家游戏失败。 (另外，如果只剩一个数字，按位异或运算得到它本身；如果无数字剩余，按位异或运算结果为 0。）
并且，轮到某个玩家时，如果当前黑板上所有数字按位异或运算结果等于 0，这个玩家获胜。
假设两个玩家每步都使用最优解，当且仅当 Alice 获胜时返回 true。
题解：
显然偶数的时候，根据博弈论的必胜必败态，我总能让他异或值不为0.
证明：如果此时nums有n个数字，n为偶数。
反证法：此时Alice必然输，不论擦掉哪个。
首先分类讨论：
先是$S=0$的情况：
$$ nums[0] \bigotimes nums[1] ... \bigotimes nums[n-1] = S$$
那么擦掉nums[i]的异或值$S_i$,其值可以通过nums[i]和S得到：
$$ S_i = nums[i] \bigotimes S$$
如果把这些$S_i$都异或将得到
$$ S = 0 $
显然矛盾
而$S!=0$显然易得，因为nums只含有非负的元素，因此，它被擦掉肯定不会是0.
而奇数的时候
而如果擦掉之前S=0,那么根据上面的推论很显然还是Alice赢，否则Bob赢。
code:
{% codeblock lang:Java %}
class Solution {
    public boolean xorGame(int[] nums) {
        if(nums.length % 2 == 0)return true;
        int xor = 0;
        for(int num:nums){
            xor = xor ^ num;
        }
        return xor == 0;
    }
}
{% endcodeblock %}