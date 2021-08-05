---
title: leetcode_1
date: 2021-08-04 08:21:53
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode hard 练习一。
<!--more-->
leetcode 1923
题意：
一个国家由 n 个编号为 0 到 n - 1 的城市组成。在这个国家里，每两个 城市之间都有一条道路连接。
总共有 m 个编号为 0 到 m - 1 的朋友想在这个国家旅游。他们每一个人的路径都会包含一些城市。每条路径都由一个整数数组表示，每个整数数组表示一个朋友按顺序访问过的城市序列。同一个城市在一条路径中可能 重复 出现，但同一个城市在一条路径中不会连续出现。
给你一个整数 n 和二维数组 paths ，其中 paths[i] 是一个整数数组，表示第 i 个朋友走过的路径，请你返回 每一个 朋友都走过的 最长公共子路径 的长度，如果不存在公共子路径，请你返回 0 。
一个 子路径 指的是一条路径中连续的城市序列。
题解：
双模hash模板+两个hash合并新hash
要点：Java容器的考查使用
code:
{% codeblock lang:Java %}
//双mod hash 模板
class Hash{
    ArrayList<ArrayList<Integer>> hash1;
    ArrayList<ArrayList<Integer>> hash2;
    ArrayList<Integer> b1,b2;
    static long mod1 = 1000000007;
    static long mod2 = 1000000009;
    static long base1 = 23, base2 = 37;
    //hash初始化
    public Hash(){
        hash1 = new ArrayList<ArrayList<Integer>>();
        hash2 = new ArrayList<ArrayList<Integer>>();
        b1 = new ArrayList<>();
        b2 = new ArrayList<>();
    }
    //传入数组
    public void get_hash(int id,int[] path){
        int len = path.length;
        ArrayList<Integer> h1 = new ArrayList<>(len);
        ArrayList<Integer> h2 = new ArrayList<>(len);
        h1.add(0);h2.add(0);
        for(int j=0;j<len;j++){
            long data1 = (h1.get(j)*base1+path[j])%mod1;
            long data2 = (h2.get(j)*base2+path[j])%mod2;
            h1.add((int)data1);
            h2.add((int)data2);
        }
        hash1.add(h1);
        hash2.add(h2);
    }
    //初始化B
    public void init_b(int m){
        long r1 = 1, r2 = 1;
        b1.add((int)r1);b2.add((int)r2);
        for(int i=0;i<m;i++){
            r1 = (r1 * base1)%mod1;
            r2 = (r2 * base2)%mod2;
            b1.add((int)r1);b2.add((int)r2);
        }
    }
    //得到hash1
    public long gethash1(int l,int r,int id){
        return ((long)hash1.get(id).get(r)-
                    (long)hash1.get(id).get(l-1)*(long)b1.get(r-l+1)%mod1+mod1)%mod1;
    }
    //得到hash2
    public long gethash2(int l,int r,int id){
        return ((long)hash2.get(id).get(r)-
                    (long)hash2.get(id).get(l-1)*(long)b2.get(r-l+1)%mod2+mod2)%mod2;
    }
}
class Solution {
    static boolean check(int mid, int[][] paths, Hash hash){
        Map<Long,Integer> mp = new HashMap<>();
        Map<Long,Integer> last = new HashMap<>();
        int sz1 = paths.length;
        for(int i=0;i<sz1;i++){
            int sz2 = paths[i].length;
            for(int j=0;j+mid-1<sz2;j++){
                long h1,h2;
                h1 = hash.gethash1(j+1, j+mid, i);
                h2 = hash.gethash2(j+1, j+mid, i);
                long idx = ((long)h1<<(long)32)+(long)h2;
                if(last.containsKey(idx) == true){
                    if(last.get(idx)!=(i+1)) {
                        int val = mp.get(idx);
                        mp.put(idx, val + 1);  //会自动覆盖？
                        last.put(idx, i+1);
                        if(val+1>=sz1)return true;
                    }
                }
                else{
                    last.put(idx, i+1);
                    mp.put(idx, 1);
                }
            }
        }
        return false;
    }
    public int longestCommonSubpath(int n, int[][] paths) {
        Hash hash = new Hash();
        int mm=10000000,mx=0,cnt=0;
        for(int[] path:paths){
            int len = path.length;
            hash.get_hash(cnt, path);
            mm = Math.min(mm,len);
            mx = Math.max(mx,len);
            cnt++;
        }
        hash.init_b(mx);
        int l=0,r=mm,ans=0;
        while(l<r){
            int mid=(l+r)>>1;
            if(check(mid, paths, hash)) {
                l = mid + 1;
                ans = mid;
            }
            else r=mid-1;
        }
        if(check(l,paths,hash))
            ans=l;
        return ans;
    }
}

{% endcodeblock %}

leetcode 1928
题意：
就是给一个图和一个规定的时间，要求在规定的时间内到达终点，求其最小的花费。
题解：
法一、
状态：$dp[u][t]$表示从0号节点出发到达u节点的最小花费是多少。
转移：
$$ dp[v][t] = min(dp[u][t-w[u][v]]+s[v],dp[v][t]) $$
其中u和v相连。
然后按照BFS的拓扑序更新即可。
优化1：超过时间的，不去做更新操作。
优化2：一条边第一次走过了，第二次就不再走了，肯定花费更大。
优化3：优先更新权重短的。（贪心）
code:
{% codeblock lang:C++ %}
#define MAXN 1010
#define INF 0x3f3f3f3f
typedef pair<int,int> py;
int dp[MAXN][MAXN];
int tmp[MAXN][MAXN];
int head[MAXN<<1],tot;
struct Edge{
    int v,w,next;
};
Edge e[MAXN<<1];
void add(int u,int v,int w){
    e[tot].v=v;
    e[tot].w=w;
    e[tot].next=head[u];
    head[u]=tot++;
}
struct Node{
    int p,v,w,t;
    Node(){}
    Node(int a,int b,int c,int d):p(a),v(b),t(c),w(d){}
    const bool operator <(const Node &tp)const{
        return w>tp.w;
    }
};
class Solution {
public:
    int DP(vector<int>& passingFees, int maxTime){
        int n=passingFees.size();
        for(int i=0;i<n;i++){
            for(int j=0;j<=1000;j++){
                dp[i][j]=INF;
            }
        }
        priority_queue<Node> q;
        q.push(Node(0,0,0,passingFees[0]));
        dp[0][0]=passingFees[0];
        while(!q.empty()){
            int u=q.top().v;
            int t=q.top().t;
            int p=q.top().p;
            q.pop();
            for(int i=head[u];i!=-1;i=e[i].next){
                int v=e[i].v;
                if(p==v)continue; //优化1
                if(t+e[i].w<=maxTime&&dp[v][t+e[i].w]>dp[u][t]+passingFees[v]){
                    dp[v][t+e[i].w]=dp[u][t]+passingFees[v];
                    //printf("dp[%d][%d]=dp[%d][%d]+%d=%d\n",v,t+e[i].w,u,t,passingFees[v],dp[v][t+e[i].w]);
                    q.push(Node(u,v,t+e[i].w,dp[v][t+e[i].w]));
                }
            }
        }
        int ans=INF;
        for(int i=0;i<=maxTime;i++){
            ans=min(ans,dp[n-1][i]);
        }
        return ans>=INF?-1:ans;
    }
    int minCost(int maxTime, vector<vector<int>>& edges, vector<int>& passingFees) {
        tot=0;
        memset(head,-1,sizeof(int)*(passingFees.size()+1));
        int n=passingFees.size();
        for(int i=0;i<n;i++)
            for(int j=0;j<n;j++)
                tmp[i][j]=INF;
        for(auto edge:edges){
            int u=edge[0];
            int v=edge[1];
            int w=edge[2];
            tmp[u][v]=min(tmp[u][v],w);
        }
        for(int i=0;i<n;i++){
            for(int j=0;j<n;j++){
                if(tmp[i][j]!=INF){
                    add(i,j,tmp[i][j]);
                    add(j,i,tmp[i][j]);
                }
            }
        }
        return DP(passingFees, maxTime);
    }
};
{% endcodeblock %}
法二、
状态：$dp[u][t]$表示从0号节点出发到达u节点的最小花费是多少。
转移：
这次不从u转移。从t转移即可。
(双向边要转移两次)
$$ dp[v][t] = min(dp[u][t-w[i][j]] + s[j], dp[v][t]) $$
code:
{% codeblock lang:Java %}
class Solution {
    public int minCost(int maxTime, int[][] edges, int[] passingFees) {
        int [][]dp = new int[maxTime+1][passingFees.length];
        int n = passingFees.length;
        for(int t=0;t<=maxTime;t++){
            for(int i=0;i<n;i++){
                dp[t][i] = 1000000009;
            }
        }
        dp[0][0] = passingFees[0];
        for(int t=0;t<=maxTime;t++){
            for(int[] edge:edges){
                int u=edge[0],v=edge[1],w=edge[2];
                if((t-w)>=0){
                    dp[t][v] = Math.min(dp[t-w][u] + passingFees[v], dp[t][v]);
                    dp[t][u] = Math.min(dp[t-w][v] + passingFees[u], dp[t][u]);
                }
            }
        }
        int ans = 1000000009;
        for(int t=0;t<=maxTime;t++){
            ans = Math.min(ans, dp[t][n-1]);
        }
        return ans==1000000009?-1:ans;
    }
}
{% endcodeblock %}

leetcode 1932
题意：
给一个含若干个的二叉搜索树（大小不超过3个节点）。请合并成一个新的二叉搜索树。
注意到合并的条件：
选择两个 不同的 下标 i 和 j ，要求满足在 trees[i] 中的某个 叶节点 的值等于 trees[j] 的 根节点的值 。
题解：
分成两步进行操作：
step 1. 先进行合并多颗二叉树的操作。
那么首先我们得确定二叉搜索树的整个根节点在哪？
入度为0的点，为什么呢，这是由**合并的策略**决定的。
step 2. 判断是否是二叉搜索树。
这个是比较难的点。
二叉树的前序、中序、后序遍历：
![alt](/images/leetcode-1/1.png)
这里主要是二叉树的中序遍历：
![alt](/images/leetcode-1/2.png)
这个易得二叉树的中序遍历是个递增的序列。
于是我们只需要检查二叉树的中序遍历是否是有序的即可。
code：
{% codeblock lang:C++ %}
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode() : val(0), left(nullptr), right(nullptr) {}
 *     TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
 *     TreeNode(int x, TreeNode *left, TreeNode *right) : val(x), left(left), right(right) {}
 * };
 */
 #define MAXN 50002
class Solution {
int cnt;
int idx[MAXN]; //以k为根的子树在vector中的标号
int in[MAXN];
TreeNode *prev;
public:
    void dfs(TreeNode *rt, TreeNode *pre, vector<TreeNode*>& trees){
        //printf("cur is NULL\n");
        if(rt == NULL)return;
        //printf("cur = %d\n",rt->val);
        int val = rt->val;
        int id=idx[val];
        if(id != -1 && rt->left==NULL && rt->right==NULL && pre!=rt){
            int pval = pre->val;
            TreeNode *cur=trees[id];
            bool flag = false;
            //左子树
            if(pval > val){
                if(cur->right==NULL)
                    flag=true;
                else if(cur->right!=NULL && pval > cur->right->val )
                    flag=true;
                else flag=false;
                if(flag){
                    pre->left = cur;
                    rt = cur;
                    cnt++;
                }
            }
            else{
                if(cur->left==NULL)
                    flag=true;
                else if(cur->left!=NULL && pval < cur->left->val)
                    flag=true;
                else flag=false;
                if(flag){
                    pre->right = cur;
                    rt = cur;
                    cnt++;
                }
            }
        }
        dfs(rt->left, rt, trees);
        dfs(rt->right, rt, trees);
    }
    //中序遍历的方法实现
    bool isBST(TreeNode *root){
        if(root != NULL)
        {
            if(!isBST(root->left))
                return false;
            if(prev != NULL && root->val < prev->val)
                return false;
            prev = root;
            if(!isBST(root->right))
                return false;
        }
        return true;
    }
    TreeNode* canMerge(vector<TreeNode*>& trees) {
        cnt = 0;
        int Tsz = trees.size();
        memset(idx, -1, sizeof(idx));
        memset(in, 0, sizeof(in));
        if(Tsz == 1)return trees[0];
        TreeNode *rt = NULL;
        int mx = 0;
        for(int i=0;i<Tsz;i++){
            int val = trees[i]->val;
            idx[val] = i;
            TreeNode *cur;
            if(trees[i]->left!=NULL){
                cur = trees[i]->left;
                in[cur->val]=1;
            }
            if(trees[i]->right!=NULL){
                cur = trees[i]->right;
                in[cur->val]=1;
            }
        }
        printf("%d %d\n",in[10],idx[10]);
        for(int i=0;i<MAXN;i++){
            if(in[i]==0 && idx[i]!=-1){
                //printf("i=%d\n",i);
                printf("idx[%d]=%d\n",i,idx[i]);
                rt = trees[idx[i]];
            }
        }
        if(trees[1]==NULL)printf("true\n");
        //printf("rt=%d\n",rt->val);
        dfs(rt, rt, trees);
        if(cnt!=Tsz-1)return NULL;
        prev=NULL;
        if(isBST(rt))return rt;
        printf("true\n");
        return NULL;
    }
};
/**
 * Your TreeAncestor object will be instantiated and called as such:
 * TreeAncestor* obj = new TreeAncestor(n, parent);
 * int param_1 = obj->getKthAncestor(node,k);
 */
{% endcodeblock %}

{% codeblock lang:Java %}
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    // 存储所有叶节点值的哈希集合
    HashSet<Integer> leaves = new HashSet<>();
    // Hash映射
    HashMap<Integer, TreeNode> candidates = new HashMap<>();
    int prev=0;
    boolean dfs(TreeNode tree){
        if(tree==null)return true;
        //叶子节点就合并
        if(leaves.contains(tree.val)&&candidates.containsKey(tree.val)){
            TreeNode ntree = candidates.get(tree.val);
            tree.left = ntree.left;
            tree.right = ntree.right;
            candidates.remove(tree.val);
        }
        if(!dfs(tree.left))
            return false;
        if(tree.left!=null&&prev>tree.val)
            return false;
        prev = tree.val;
        if(!dfs(tree.right))
            return false;
        return true;
    }
    public TreeNode canMerge(List<TreeNode> trees) {
        for(TreeNode tree:trees){
            if(tree.left!=null){
                leaves.add(tree.left.val);
            }
            if(tree.right!=null){
                leaves.add(tree.right.val);
            }
            candidates.put(tree.val, tree);
        }
        for(TreeNode tree:trees){
            //根节点
            if(!leaves.contains(tree.val)){
                candidates.remove(tree.val);
                return (dfs(tree)&&candidates.isEmpty()?tree:null);
            }
        }
        return null;
    }
}
{% endcodeblock %}