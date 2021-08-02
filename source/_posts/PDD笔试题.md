---
title: PDD笔试题
date: 2021-08-02 16:30:13
tags: [Java 笔试题]
categories: [Java 笔试题]
---
A. 
题意：
![alt](/images/PDD笔试题/1.jpg)
题解：
不难，排序后，一个个比较是否包含就行。
code:
{% codeblock lang:Java %}
import java.util.*;
//结构体的二维排序
class Line{
    int l,r;
    public Line(){}
    public Line(int ll,int rr){
        l=ll;
        r=rr;
    }
    @Override
    public String toString(){
        return "l = " + this.l + ", r = " + this.r;
    }
}
public class ProblemA {
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        int n = cin.nextInt();
        Line[] lines = new Line[n];
        for(int i=0;i<n;i++){
            lines[i] = new Line();
            lines[i].l=cin.nextInt();
            lines[i].r=cin.nextInt();
        }
        Arrays.sort(lines, new Comparator<Line>() {
            @Override
            public int compare(Line o1, Line o2) {
                if(o1.l == o2.l)
                    return o1.r - o2.r;
                else
                    return o1.l - o2.l;
            }
        });
        boolean flag=true;
        for(int i=1;i<n;i++){
            if(lines[i].r>lines[i-1].r){
                flag=false;
                break;
            }
        }
        if(flag) System.out.println("false");
        else System.out.println("true");
    }
}
{% endcodeblock %}


B.
题意：
![alt](/images/PDD笔试题/2.jpg)
题解：

code：
{% codeblock lang:Java %}
import java.util.*;
public class ProblemB {
    static int cnt=0;
    static int [] c = new int[100005];
    static Map<Integer, Integer> hashMap = null;
    static int setcards(int val){
        //O(1)的复杂度
        if(hashMap.containsKey(val)){
            int p = hashMap.get(val);
            if(p>cnt){
                c[++cnt]=val;  //set
                hashMap.put(val,cnt);
                return 0;
            }
            int num = cnt-p+2;
            cnt=p-1;   //只有前p-1有效
            return num;
        }
        c[++cnt]=val;
        hashMap.put(val,cnt);
        return 0;
    }
    public static void main(String[] args) {
        int [] a = new int[100005];
        int [] b = new int[100005];
        Scanner cin = new Scanner(System.in);
        int N = cin.nextInt();
        for(int i=1;i<=N;i++)a[i]=cin.nextInt();
        for(int i=1;i<=N;i++)b[i]=cin.nextInt();
        cnt = 0;
        int count1=0,count2=0,ans1=0,ans2=0;
        hashMap = new HashMap<Integer, Integer>();
        while(count1<N||count2<N){
            while(count1<N){
                count1++;
                int tag=setcards(a[count1]);
                if(tag<=0)break;
                ans1+=tag;
            }
            while(count2<N){
                count2++;
                int tag=setcards(b[count2]);
                if(tag<=0)break;
                ans2+=tag;
            }
        }
        for(int i=1;i<=cnt;i++){
            if(c[i]%2!=0)ans1++;
            else ans2++;
        }
        System.out.println(ans1+" "+ans2);
    }
}
{% endcodeblock %}
例子：
{% codeblock lang:Java %}
Input:
4
1 2 3 4
1 2 3 4
Output
4 4

Input:
4
2 3 1 2
1 4 3 4
Output:
7 1

Input:
4
2 3 1 2
1 4 4 4
Output:
6 2

Input:
3
1 2 3
4 5 1
Output:
0 6

Input:
3
1 2 3
4 5 6
Output:
3 3

Input:
5
1 2 5 1 3
4 3 3 4 3
Output:
4 6
{% endcodeblock %}


C.
题意：
![alt](/images/PDD笔试题/3.jpg)
题解:
数学题，需要推出通项公式：
若一个数在集合$S$中，那么有：
$$ S_{i} = S_{i-1} * C^{k_{i}} +  t_{i} * B, i=1,2,...N $$  (1)
$$ S_{i-1} = S_{i-2} * C^{k_{i-1}} +  t_{i-1} * B, i=1,2,...N $$ (2)
联合(1)和(2)，得：
$$ S_{i} = （S_{i-2} *  C^{k_{i-1}} +  t_{i-1} * B) * C^{k_{i}} + t{i} * B, i=1,2,...N $$
化简有：
$$ S_{i} = S_{i-2} * C^{k_{i-1}+k_{i}} +  (t_{i-1} * C^{k_{i}} + t{i}) * B, i=1,2,...N $$
我们令$ KK = k_{i-1} + k_{i} $,并且令$ TT =  (t_{i-1} * C^{k_{i}} + t{i}) $, 这里显然KK和TT可以取到N上的任意整数。
那么就会得到：
$$ S_{i} = S_{i-2} * C^{KK} + TT * B $$
化简得到通项：
$$ S_{i} = S_{0} * C^{K} + T * B $$

故只需要枚举$K$，然后判断$(Q - C^{k}*A) % B == 0$即可。特殊地若C=1，那么只需判断 $ (Q % B) == A $
code：
{% codeblock lang:Java %}
import java.util.*;
public class ProblemC {
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        int T=cin.nextInt();
        while(T>0){
            T--;
            int A = cin.nextInt();
            int B = cin.nextInt();
            int C = cin.nextInt();
            int Q = cin.nextInt();
            if(C==1){
                if(Q%B==A) System.out.println("1");
                else System.out.println("0");
            }
            else{
                int res=1;
                boolean tag=false;
                for(int i=0;i<=32;i++){
                    if(res>Q)break;
                    if((Q-res*A)%B==0){
                        tag=true;
                        break;
                    }
                    res=res*C;
                }
                if(tag) System.out.println("1");
                else System.out.println("0");
            }
        }
    }
}
{% endcodeblock %}

D.
题意:
![alt](/images/PDD笔试题/4.jpg)
题解:
贪心，首先数字被分的越多，乘积越小。数字越少，越大。
故这里肯定是将这些数分成两个数合理。
这题的结论是让两个数的差值尽可能小。（这个结论是猜的，证明暂时略）
code:
{% codeblock lang:Java %}
import java.math.BigInteger;
import java.util.Scanner;

public class ProblemD {
    public static void main(String[] args) {
        Scanner cin = new Scanner(System.in);
        int []cnts = new int[10];
        int []a = new int[100005];
        for(int i=0;i<10;i++)cnts[i]=cin.nextInt();
        BigInteger A = new BigInteger("0");
        BigInteger B = new BigInteger("0");
        int cnt=0;
        for(int i=9;i>=0;i--){
            for(int j=1;j<=cnts[i];j++){
                a[++cnt]=i;
            }
        }
        for(int i=1;i<=cnt;i+=2){
            if(A.compareTo(B)<=0){
                A = A.multiply(BigInteger.valueOf(10));
                A = A.add(BigInteger.valueOf(a[i]));
                B = B.multiply(BigInteger.valueOf(10));
                B = B.add(BigInteger.valueOf(a[i+1]));
            }
            else{
                A = A.multiply(BigInteger.valueOf(10));
                A = A.add(BigInteger.valueOf(a[i+1]));
                B = B.multiply(BigInteger.valueOf(10));
                B = B.add(BigInteger.valueOf(a[i]));
            }
        }
        System.out.println(A.multiply(B));
    }
}
{% endcodeblock %}