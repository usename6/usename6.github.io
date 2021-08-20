---
title: leetcode-3
date: 2021-08-20 09:48:44
tags: [Java leetcode]
categories: [Java leetcode]
---
树形DP和排列组合是弱项。
还有并查集删边的乱搞图论也是弱项。
<!--more-->
leetcode 1916 **好题**
题意：给出一根有根树（有向图）。让你找出拓扑排序总的方案数。
题解：
在这里需要知道一个前置的排列组合的知识点：
工厂有$n$个不同种类的零件，每个同种类零件外形相同。其中第$i$个零件有$a_{i}$个。
那么将这些零件摆成一排的方案数:
$$ \frac{(a_1 + a_2 + ... + a_n)!}{(a_{1}!+a_{2}!+...+a_{n}!)} $$
状态设置: $dp[i]$表示以i为根的子树的拓扑排序的方案数
状态转移：
先从一个最简单的情况考虑状态转移方程：
当前子树的子树间不存在交叉的情况：
$$dp[u]=\prod_{i=1}^m{dp[v_i]}$$
然后，我们可以把原问题拆分成两个问题：
(1)首先我们先选出$n$个不同种类的零件，然后计算他们的排列数（拓扑序）的数量。
然后将这些拓扑序组合起来，得到的方案数如下：
$$dp[u]=\prod_{i=1}^m{dp[v_i]}$$
(2)然后我们考虑将这些不同零件排成一排的方案数，首先预处理出零件个数，
这里需要通过开一个数组实现，即$cnt[i]$表示以i为根的子树的节点个数。
那么根据前置知识，这里方案数如下：
$$ \frac{(cnt_{v_1} + cnt_{v_2} + ... + cnt_{v_n})!}{(cnt_{v_1}!+cnt_{v_2}!+...+cnt_{v_n}!)} $$
又因为：
$$cnt[u]=\sum_{i=1}^n{cnt_{v_i}}$$
所以该式子可以化简为：
$$ \frac{(cnt_u-1)!}{(cnt_{v_1}!+cnt_{v_2}!+...+cnt_{v_n}!)} $$
整理得到最后的转移方程为：
$$dp[u]=\prod_{i=1}^m{dp[v_i]} \times \frac{(cnt_u-1)!}{(cnt_{v_1}!+cnt_{v_2}!+...+cnt_{v_n}!)} $$
code:
{% codeblock lang:Java %}
class Edge{
    int v,next;
}
class Solution {
    int[] dp; int[] cnt;
    int n,tot;Edge[] e;
    int[] head;
    long mod=1000000007;
    int[] JC;int[] inv;
    void init(){
        JC=new int[n+1];inv=new int[n+1];
        JC[0]=inv[0]=1;
        for(int i=1;i<n;i++){
            JC[i]=(int)((long)JC[i-1]*(long)i%mod);
            inv[i]=(int)qpow_mod((long)JC[i],mod-2);
        }
    }
    long qpow_mod(long a,long nn){
        long res=1;
        while(nn>0){
            if(nn%2==1)res=res*a%mod;
            nn>>=1;
            a=(a*a)%mod;
        }
        return res%mod;
    }
    void add(int u,int v){
        e[tot] = new Edge();
        e[tot].v = v;
        e[tot].next = head[u];
        head[u] = tot++;
    }
    public void dfs(int u,int pre){
        dp[u]=1;
        cnt[u]=0;
        for(int i=head[u];i!=-1;i=e[i].next){
            int v=e[i].v;
            if(pre!=v){
                dfs(v,u);
                cnt[u]+=cnt[v];
            }
        }
        if(cnt[u]>0)dp[u]=(int)((long)dp[u]*(long)JC[cnt[u]]%mod);
        for(int i=head[u];i!=-1;i=e[i].next){
            int v=e[i].v;
            if(pre!=v){
                dp[u]=(int)((long)dp[u]*(long)dp[v]%mod);
                dp[u]=(int)((long)dp[u]*(long)inv[cnt[v]]%mod);
            }
        }
        cnt[u]+=1;
    }
    public int waysToBuildRooms(int[] prevRoom) {
        n = prevRoom.length;
        init();
        int rt=0;
        tot = 0;
        dp = new int[n+1];
        e = new Edge[n+1];
        head = new int[n+1];
        cnt = new int[n+1];
        for(int i=0;i<n;i++)head[i]=-1;
        for(int i=0;i<n;i++){
            if(prevRoom[i]==-1)rt=i;
            else{
                add(prevRoom[i],i);
            }
        }
        dfs(rt,rt);
        return dp[rt];
    }
}
{% endcodeblock %}
其中：处理逆元（费马小定理可以做模板）。以及链式前向星的建立也可以做模板。

leetcode 1766
题意：给一个带点权的无根树（无向图）。这里指定根节点为n。对于该树的每个节点，
你需要找到一个和他互质的最近的祖先节点，如果没有则为-1。
题解：
傻逼题，这道题只需要注意到点权不会超过50，那这样只需要dfs一遍，
在这期间需要维护一下每个点权的最后位置在哪，然后扫一遍用gcd check一下即可。
code:
{% codeblock lang:Java %}
class Edge{
    int v,next;
}
class Solution {
    Edge[] e;
    int[] head;int tot,n,m,cnt;
    int[] ans;
    int[] dis;
    ArrayList<ArrayList<Integer>> pos;
    int gcd(int a,int b){
        return b==0?a:gcd(b,a%b);
    }
    void add(int u,int v){
        e[tot]=new Edge();
        e[tot].v=v;
        e[tot].next=head[u];
        head[u]=tot++;
    }
    void dfs(int u,int pre,int[] nums){
        if(u!=pre)dis[u]=dis[pre]+1;
        //if(u==2)System.out.println("u = "+nums[2]+", pre = "+nums[17]);
        int mm=100006,k=-1,tt=nums[u];
        for(int i=1;i<=50;i++){
            int sz=pos.get(i).size();
            if(gcd(tt,i)!=1)continue;
            if(sz!=0){
                int pp=pos.get(i).get(sz-1);
                //if(u==2)System.out.println("pos["+i+"] = "+pp);
                if(dis[u]-dis[pp]<mm){
                    mm=dis[u]-dis[pp];
                    k=pp;
                }
            }
        }
        ans[u]=k;
        pos.get(nums[u]).add(u);
        for(int i=head[u];i!=-1;i=e[i].next){
            int v=e[i].v;
            if(v!=pre){
                dfs(v,u,nums);
            }
        }
        pos.get(nums[u]).remove(pos.get(nums[u]).size()-1);
    }
    public int[] getCoprimes(int[] nums, int[][] edges) {
        n=nums.length;
        m=edges.length;
        head=new int[n];tot=0;
        e=new Edge[(m+1)*2];
        ans=new int[n]; dis=new int[n];
        for(int i=0;i<n;i++)head[i]=-1;
        pos=new ArrayList<ArrayList<Integer>>();
        for(int i=0;i<=50;i++){
            ArrayList<Integer> arr=new ArrayList<Integer>();
            pos.add(arr);
        }
        for(int[] edge:edges){
            int u=edge[0],v=edge[1];
            add(u,v);
            add(v,u);
        }
        dfs(0,0,nums);
        return ans;
    }
}
{% endcodeblock %}

leetcode 1627
题意：给出1~n的n个数，给出m个查询，每个查询包含两个点，需要判断这两个点是否相连。
相连的条件如下：
$$gcd(x,y) > threshold$$
题解：
暴力，对于每个点，暴力处理它所有的因数，对于每个因数我去维护它最后出现的数是什么。
然后把这个数和这个点连边，用并查集优化即可。
code:
{% codeblock lang:Java %}
class Solution {
    int[] f;
    int[] cc;
    public int find(int x){
        return x==f[x]?x:(f[x]=find(f[x]));
    }
    public void union_set(int s1,int s2){
        int f1=find(s1),f2=find(s2);
        if(f1!=f2)
            f[f2]=f1;
    }
    public List<Boolean> areConnected(int n, int threshold, int[][] queries) {
        f = new int[n+1];
        cc = new int[n+1];
        for(int i=1;i<=n;i++){
            f[i]=i;
            int now=i;
            for(int j=1;j*j<=i;j++){
                if(now%j==0){
                    if(cc[j]>0&&j>threshold){
                        union_set(i,cc[j]);
                    }
                    //System.out.println("pre:cc["+j+"]="+cc[j]);
                    cc[j]=i;
                    //System.out.println("after:cc["+j+"]="+cc[j]);
                    if(cc[i/j]>0&&(i/j)>threshold){
                        union_set(i,cc[i/j]);
                    }
                    //System.out.println("pre2:cc["+(i/j)+"]="+cc[i/j]);
                    cc[i/j]=i;
                    //System.out.println("after2:cc["+(i/j)+"]="+cc[i/j]);
                }
            }
        }
        List<Boolean> ans=new ArrayList<Boolean>();
        for(int[] query:queries){
            int x=query[0],y=query[1];
            if(find(x)!=find(y))
                ans.add(false);
            else ans.add(true);
        }
        return ans;
    }
}
{% endcodeblock %}

leetcode 1579  **好题**
题意：一个无向图，有三个类型的边，一个只有Alice能遍历，
一个只有Bob能遍历，另外一个是两个都能遍历。问你最少删掉
多少边可以保证Alice和Bob从任意节点出发，都可达任意其他节点。
题解：
贪心+并查集。
首先可以证明的是我们先得保留公共边，如果u和v在两个不同联通分量上，
保留公共边总比添加两个条非公共边。
于是我们用并查集判断我们当前要加入的边两个端点是否属于同一个联通分量，
不属于则添加，但优先插入公共边即可。
code:
{% codeblock lang:Java %}
class Solution {
    int[] f1,f2;
    public int find1(int x){
        return x==f1[x]?x:(f1[x]=find1(f1[x]));
    }
    public int find2(int x){
        return x==f2[x]?x:(f2[x]=find2(f2[x]));
    }
    public int maxNumEdgesToRemove(int n, int[][] edges) {
        Arrays.sort(edges, new Comparator<int[]>(){
            @Override
            public int compare(int[] o1,int[] o2){
                if(o1[0]==o2[0])return o1[1]-o2[1]; //第二个元素升序
                return o2[0]-o1[0]; //第一个元素升序
            }
        });
        f1=new int[n+1];
        f2=new int[n+1];
        for(int i=1;i<=n;i++)f1[i]=f2[i]=i;
        int ans=0,tot1=0,tot2=0;
        for(int[] edge:edges){
            int type=edge[0],u=edge[1],v=edge[2];
            if(type==3){
                int p1=find1(u),p2=find1(v);
                if(p1!=p2){
                    f1[p2]=p1;
                    tot1++;
                }
                else ans++;
                p1=find2(u);p2=find2(v);
                if(p1!=p2){
                    f2[p2]=p1;
                    tot2++;
                    //System.out.println(Arrays.toString(edge));
                }
            }
            else if(type==2){
                int p1=find2(u),p2=find2(v);
                if(p1!=p2){
                    f2[p2]=p1;
                    tot2++;
                    //System.out.println(Arrays.toString(edge));
                }
                else ans++;
            }
            else if(type==1){
                int p1=find1(u),p2=find1(v);
                if(p1!=p2){
                    f1[p2]=p1;
                    tot1++;
                    //System.out.println(Arrays.toString(edge));
                }
                else ans++;
            }
        }
        if(tot1!=n-1||tot2!=n-1)return -1;
        return ans;
    }
}
{% endcodeblock %}

leetcode 1970
题意：给出个$row \times col$的地图，让你判断最后通过陆地是天数
题解：
答案显然具有单调性，于是二分答案，check用BFScheck即可。
暴力即可。
code:
{% codeblock lang:Java %}
class Point{
    int x,y,step;
    public Point(){}
    public Point(int xx,int yy){
        x=xx;y=yy;
    }
}
class Solution {
    int[] dx={0,0,1,-1};
    int[] dy={1,-1,0,0};
    boolean check(int row,int col,int mid,int[][] cells){
        //System.out.println("mid="+mid);
        int[][] D=new int[row+1][col+1];
        int[][] step=new int[row+1][col+1];
        Queue<Point> q=new LinkedList<Point>();
        for(int i=0;i<mid;i++){
            int markX=cells[i][0],markY=cells[i][1];
            D[markX][markY]=1;
        }
        for(int i=1;i<=col;i++){
            if(D[1][i]!=1){
                q.offer(new Point(1,i));
                step[1][i]=1;
            }
        }
        while(!q.isEmpty()){
            Point tp=q.peek();
            q.poll();
            //System.out.println("pre:step["+tp.x+"]["+tp.y+"]="+step[tp.x][tp.y]);
            int day=step[tp.x][tp.y]+1;
            for(int i=0;i<4;i++){
                int xx=tp.x+dx[i];
                int yy=tp.y+dy[i];
                if(xx<1||xx>row||yy<1||yy>col)continue;
                if(D[xx][yy]==1)continue;
                if(step[xx][yy]==0){
                    if(xx==row)return true;
                    step[xx][yy]=step[tp.x][tp.y]+1;
                    //System.out.println("cur:step["+xx+"]["+yy+"]="+step[xx][yy]);
                    q.offer(new Point(xx,yy));
                }
            }
        }
        return false;
    }
    public int latestDayToCross(int row, int col, int[][] cells) {
        int l=1,r=cells.length,ans=1;
        while(l<r){
            int mid=(l+r)>>1;
            if(check(row,col,mid,cells)){
                ans=mid;
                l=mid+1;
            }
            else r=mid-1;
        }
        if(l==r&&check(row,col,l,cells))ans=l;
        return ans;
    }
}
{% endcodeblock %}