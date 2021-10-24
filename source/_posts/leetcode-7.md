---
title: leetcode-7
date: 2021-10-17 17:08:59
tags: [Java leetcode]
categories: [Java leetcode]
---
leetcode 7次练习，养胃，养身体，冲刺高offer.
<!--more-->
leetcode
题意：
路径 被定义为一条从树中任意节点出发，沿父节点-子节点连接，达到任意节点的序列。同一个节点在一条路径序列中 至多出现一次 。该路径 至少包含一个 节点，且不一定经过根节点。
路径和 是路径中各节点值的总和。
给你一个二叉树的根节点 root ，返回其最大路径和。
题解:
树形DP
状态定义:dp[cur]表示以cur为根节点包含cur的子树最大的路径和
转移方程:
if(left!=null&&right!=null)ans=Math.max(ans, dp[left]+dp[right]+u.val);
if(left!=null)dp[cur]=Math.max(dp[cur], dp[left]+u.val);
if(right!=null)dp[cur]=Math.max(dp[cur], dp[right]+u.val);
code:
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
    int cnt=-1;
    int[] dp = new int[30004];
    int ans=-1000;
    public int dfs(TreeNode u){
        if(u==null){
            return -1;
        }
        int cur = ++cnt;
        dp[cur]=u.val;
        int left = dfs(u.left);
        int right = dfs(u.right);
        if(left!=-1&&right!=-1)ans=Math.max(ans, dp[left]+dp[right]+u.val);
        if(left!=-1)dp[cur]=Math.max(dp[cur], dp[left]+u.val);
        if(right!=-1)dp[cur]=Math.max(dp[cur], dp[right]+u.val);
        ans = Math.max(ans, dp[cur]);
        return cur;
    }
    public int maxPathSum(TreeNode root) {
        dfs(root);
        return ans;
    }
}
{% endcodeblock %}

leetcode
题意:
给定两个大小分别为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。请你找出并返回这两个正序数组的 中位数 。
题解:
复杂度O(log(n+m))的做法
首先明白中位数的定义，中位数就是中间的数，在长度为len的数组arr中:
1)如果len是偶数，那么中位数就是$(arr[mid]+arr[mid+1])/2$
2)如果len是奇数，那么中位数就是$arr[mid]$
我们以此作为切入点，解决这个题。
首先我们定义两个指针，l1和l2分别指向数组a和数组b。它表示在数组a和数组b中最有可能成为中位数的数的位置。
显而易见，我们将其初始化在mid1和mid2，即数组a和数组b的中间。
此时我们比较$a[l1+k/2-1]$和$b[l2+k/2-1]$的大小关系,如果$a[l1+k/2-1] \textless b[l2+k/2-1]$,在这里为了方便说明，我们令$mid1=l1+k/2-1$,而$mid2=l2+k/2-1$。那么显然a数组里$[0,mid1]$是没有可能成为中位数的。我们把他们丢进淘汰队列中，同时令$k=k-(mid1+1)$，那么此时将指针l1移动mid+1。反之亦然。然后就是你会发现这是一个递归的过程。在这个过程，我们需要特判三种情况：
a.一个数组全部进淘汰队列。那么显然就答案就是$a[l1+k-1]$或者$b[l2+k-1]$
b.mid1和mid2会越界。
c.k刚好为1，那答案显然是$a[l1]$或者$b[l2]$，此时是递归终止条件。
code:
{% codeblock lang:Java %}
class Solution {
    public int get_Kth(int l1, int r1,int l2,int r2,int[] nums1,int[] nums2,int k){
        int len1 = r1-l1+1;
        int len2 = r2-l2+1;
        //System.out.println("k="+k);
        //System.out.println("nums1:["+l1+","+r1+"]");
        //System.out.println("nums2:["+l2+","+r2+"]");
        if(len2<len1)return get_Kth(l2,r2,l1,r1,nums2,nums1,k);
        if(len1<=0)return nums2[l2+k-1];
        if(k==1)return Math.min(nums1[l1],nums2[l2]);
        int idx1 = Math.min(l1+k/2-1,r1);
        int idx2 = Math.min(l2+k/2-1,r2);
        //System.out.println("idx1="+idx1+", nums1 = "+nums1[idx1]);
        //System.out.println("idx2="+idx2+", nums2 = "+nums2[idx2]);
        if(nums1[idx1]<nums2[idx2]){
            //System.out.println("nums1 move!");
            return get_Kth(idx1+1,r1,l2,r2,nums1,nums2,k-(idx1-l1+1));
        }
        return get_Kth(l1,r1,idx2+1,r2,nums1,nums2,k-(idx2-l2+1));
    }
    public double findMedianSortedArrays(int[] nums1, int[] nums2) {
        int r1 = nums1.length, r2 = nums2.length;
        int n = r1+r2;
        return (double)(get_Kth(0,r1-1,0,r2-1,nums1,nums2,(n+1)/2)+
                get_Kth(0,r1-1,0,r2-1,nums1,nums2,(n+2)/2))/2;
    }
}
{% endcodeblock %}

leetcode 23
题意:
给你一个链表数组，每个链表都已经按升序排列。
请你将所有链表合并到一个升序链表中，返回合并后的链表。
题解:
傻逼题，直接做。
code:
{% codeblock lang:Java %}
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode mergeKLists(ListNode[] lists) {
        ListNode head=new ListNode(0);
        int n = lists.length;
        int tmp=n;
        ListNode ans=head;
        while(true){
            int k=-1,i=0;
            while(i<n){
                if(lists[i]!=null){
                    //System.out.println("i="+i+",cur lists:"+lists[i].val);
                    if(k==-1)k=i;
                    else if(lists[i].val<lists[k].val)k=i;
                    //System.out.println("k lists:"+lists[k].val);
                }
                i++;
            }
            //System.out.println("k="+k);
            if(k==-1)break;
            head.next=lists[k];
            head = head.next;
            lists[k]=lists[k].next;
            //if(lists[k]!=null)System.out.println("lists:"+k+":"+lists[k].val);
        }
        head.next = null;
        return ans.next;
    }
}
{% endcodeblock %}

leetcode 239
题意:
给你一个整数数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口内的 k 个数字。滑动窗口每次只向右移动一位。
返回滑动窗口中的最大值。
题解:
单调增的双端队列模板，傻逼题。
code:
{% codeblock lang:Java %}
class Elem{
    int id,data;
    Elem(int idt,int datat){
        id = idt;
        data=datat;
    }
}
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        int n=nums.length;
        Elem[] elems = new Elem[n*2];
        int top=0,cnt=0,tail=0;
        int[] ans = new int[n-k+1];
        for(int i=0;i<n;i++){
            //System.out.println("push "+nums[i]);
            //System.out.println("top = "+top+", tail = "+tail);
            while(tail-top!=0&&(i-elems[top].id+1)>k)top++;
            while(tail-top!=0&&elems[tail-1].data<nums[i])tail--;
            elems[tail++]=new Elem(i,nums[i]);
            /*for(int j=top;j<tail;j++){
                System.out.print(elems[j].data+" ");
            }*/
            //System.out.println("");
            if(i>=k-1){
                ans[cnt++]=elems[top].data;
                //System.out.print("ans:");
                //for(int j=0;j<cnt;j++)System.out.print(ans[j]+" ");
                //System.out.println("");
            }
        }
        return ans;
    }
}
{% endcodeblock %}
