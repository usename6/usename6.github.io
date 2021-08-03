---
title: Shopee笔试题
date: 2021-08-03 17:45:43
tags: [Java 笔试题]
categories: [Java 笔试题]
---
2021 后端Shopee笔试题合集。只摘取有价值的题目。
<!--more-->
A.
题意：
![alt](/images/Shopee笔试题/A.jpg)
题解：
原型题：Leetcode 
简单DP
状态:$dp[i][j]$表示S1的前i个字符是否能够匹配S2的前j个字符。
状态转移：
如果s1[i] == '?',那么：
$$ dp[i][j] = dp[i-1][j-1] $$
如果s1[i] == '*',那么：
$$ dp[i][j] = dp[i-1][j] | dp[i][j-1] | dp[i-1][j-1] $$
如果s1[i] == '#',那么:
$$ dp[i][j] = dp[i-1][j-1] | dp[i-1][j] $$
如果s1[i] == s2[j],那么
$$ dp[i][j] = dp[i-1][j-1] $$
code:
{% codeblock lang:Java %}
import java.util.Scanner;

public class ProblemA {
    public static void main(String[] args) {
        int [][]dp = new int[1005][1005];
        Scanner cin = new Scanner(System.in);
        String s1 = cin.nextLine();
        String s2 = cin.nextLine();
        dp[0][0]=1;
        int len1 = s1.length(),len2 = s2.length();
        for(int i=1;i<=len1;i++){
            for(int j=1;j<=len2;j++){
                if(s1.charAt(i-1) == '?'){
                    dp[i][j]=dp[i-1][j-1];
                    //System.out.println("1:dp["+i+"]["+j+"] =" +dp[i][j]);
                }
                else if(s1.charAt(i-1) == '*'){
                    dp[i][j]=dp[i][j-1]|dp[i-1][j]|dp[i-1][j-1];
                    //System.out.println("2:dp["+i+"]["+j+"] =" +dp[i][j]);
                }
                else if(s1.charAt(i-1) == '#'){
                    dp[i][j]=dp[i-1][j-1]|dp[i-1][j];
                    //System.out.println("3:dp["+i+"]["+j+"] =" +dp[i][j]);
                }
                else if(s1.charAt(i-1) == s2.charAt(j-1)){
                    dp[i][j]=dp[i-1][j-1];
                    //System.out.println("4:dp["+i+"]["+j+"] =" +dp[i][j]);
                }
            }
        }
        System.out.println(dp[len1][len2]);
    }
}

{% endcodeblock %}

{% codeblock lang:Java %}
Input:
a##b
acb
Output:
1

Input:
#####
ab
Output:
1

Input:
*
abcd
Output:
1

Input:
*a
aaaab
Output:
0

Input:
a*?
acb
Output:
1

Input：
a?*
ab
Output:
1

Input:
**a
abca
Output:
1
{% endcodeblock %}

B. 
题意：
![alt](/images/Shopee笔试题/B.jpg)
题解：
贪心，我们去按照谁放在前面能够使得字典序最大方式去比较。
排序即可。
code:
{% codeblock lang:Java %}
import java.util.*;

public class ProblemB {
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        String tot = cin.nextLine();
        String []p = tot.split(" ");
        Comparator<String> compr = new Comparator<String>(){
            public int compare(String m,String n){
                String s1 = m+n;
                String s2 = n+m;
                return s2.compareTo(s1);
            }
        };
        Arrays.sort(p, compr);
        if(p[0].charAt(0)=='0'){
            System.out.println("0");
        }
        else {
            String res = "";
            for (String pp : p) {
                res = res + pp;
            }
            System.out.println(res);
        }
    }
}

{% endcodeblock %}
C.
题意：
![alt](/images/Shopee笔试题/C.jpg)
题解:
hashmap + dfs
code:
{% codeblock lang:Java %}
import java.util.HashMap;
import java.util.Map;
import java.util.Scanner;

public class ProblemC {
    static String xml,path;
    static int len;
    static Map<String,String> hashmap = null;
    static void dfs(int cur,String key){
        if(cur>=len)return;
        if(xml.charAt(cur)=='<') {
            cur++;
            String cc="";
            String ckey="";
            while(cur<len&&xml.charAt(cur)!='>') {
                cc = cc + xml.charAt(cur);
                cur++;
            }
            //System.out.println("cc="+cc);
            if(cc.charAt(0)=='/'){
                //System.out.println("key = " + key);
                String[] keys = key.split("\\.");
                //System.out.println(keys.length);
                //System.out.println("cc_1 = " + cc.substring(1));
                if(keys.length>1&&cc.substring(1).compareTo(keys[keys.length-1]) == 0){
                    int sufix_len = cc.length() - 1;
                    ckey = key.substring(0, key.length()-sufix_len);
                    ckey = ckey.substring(0, ckey.length()-1);
                    //System.out.println("ckey = " + ckey);
                }
                cur++;
            }
            else {
                if (key.length() == 0) ckey = cc;
                else ckey = key + "." + cc;
            }
            dfs(cur,ckey);
        }
        else if(xml.charAt(cur)=='>'){
            if(key.length()==0)return;
            String cc="";
            cur++;
            while(cur<len&&xml.charAt(cur)!='<'){
                cc = cc + xml.charAt(cur);
                cur++;
            }
            if(cur<len&&xml.charAt(cur)=='<') {
                //System.out.println("hashmap[" + key + "] = " + cc);
                hashmap.put(key, cc);
            }
            dfs(cur, key);
        }
    }
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        xml = cin.nextLine();
        path = cin.nextLine();
        len = xml.length();
        hashmap = new HashMap<>();
        dfs(0, "");
        System.out.println(hashmap.get(path));
    }
}
/*
<people><name>shopee</name></people>
people.name

<people><name>shopee</name><jobs>cs</jobs><q><t>s</t><x><f>g</f></x></q></people>
 */
{% endcodeblock %}

