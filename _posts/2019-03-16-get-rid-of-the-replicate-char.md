---
layout: post
title: '王大锤的人生巅峰之路-算法'
tags: [read]
---

> 题目描述是这样的，在王大锤的眼里，给字符串纠错是个大事情，一般碰到三个连续的字母在一块儿的这种直接去掉一个就行，有AABB这种的，去掉一个B就好了，碰到AABBCCDD这样的依照最左匹配的原则进行修改。王大锤梦想着靠着这走上巅峰之路，但很快，，，就被炒了。

我的写法，递归，其中有很多时间花销是重复的，因此并不是最优写法，但还算直观

```java
package com.leyou.test;

import java.util.LinkedList;
import java.util.List;
import java.util.Scanner;
public class Main {
    private static List<String> res=new LinkedList<>();
    public static void main(String[] args) {
        Scanner scanner=new Scanner(System.in);
        //接收字符串数目
        int n=scanner.nextInt();
        List<String> strings=new LinkedList<>();
        for(int i=0;i<n;i++){
            strings.add(scanner.next());
        }
        scanner.close();
        solve(strings);
        for(int i=0;i<res.size();i++){
            System.out.println(res.get(i));
        }

    }
    public static void solve(List<String> strings){
        for(int i=0;i<strings.size();i++){
            String string=removeThree(strings.get(i));
            String string1=removeAABB(string);
            res.add(string1);
        }
    }
    public static String removeThree(String string){
        String newString="";
        if(string.length()<3){
            return string;
        }
        for(int i=1;i<string.length();i++){
            if((i+1)<=string.length()-1){
                char pre=string.charAt(i-1);
                char cur=string.charAt(i);
                char next=string.charAt(i+1);
                if(cur==pre&&cur==next){
                    newString=string.substring(0,i)+string.substring(i+1);
                    break;
                }
            }else{
                return string;
            }
        }
        newString=removeThree(newString);
        return newString;
    }
    private static String removeAABB(String string){
        if(string.length()<4){
            return string;
        }
        char cur = 0,next1,next2,next3;
        String newString="";
        for(int i=0;i<string.length();i++){
            if((string.length()-i)>=4){
                cur=string.charAt(i);
                next1=string.charAt(i+1);
                next2=string.charAt(i+2);
                next3=string.charAt(i+3);
                if(cur==next1&&cur!=next2&&next2==next3){
                    newString=string.substring(0,i+2)+string.substring(i+3);
                    break;
                }
            }else {return string;}

        }
        newString=removeAABB(newString);
        return newString;
    }

}

```

测试用例

```xml
4
helloo
helllo
helllllllooooo
helllaabbccddnnn
hello
hello
hello
hellabbcddn
```

大佬的写法，注重逻辑推演，很简洁，运行时间也很短

```java
package com.leyou.test;

import java.util.Scanner;

public class Dachui {

    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
        String[] words=new String[n+1];
        int i=0;
        while(sc.hasNextLine()) {
            words[i++]=sc.nextLine();
            if(i==(n+1))
                break;
        }
        sc.close();
        for(int j=1;j<words.length;j++) {
            checkedWord(words[j]);
        }
    }
    private static void checkedWord(String word) {
        if(word.isEmpty())
            System.out.println("");
        char[] wc=word.toCharArray();
        int cur_count=0;
        int pre_count=0;
        for(int i=0;i<wc.length;i++) {
            if(i==0) {
                cur_count=1;
                continue;
            }
            if(wc[i]==wc[i-1]) {
                cur_count++;
                if(cur_count==3) {
                    wc[i-2]=',';
                    cur_count=2;
                }
                if(cur_count==2 && pre_count==2) {
                    wc[i-1]=',';
                    cur_count=1;
                }
            }else {
                pre_count=cur_count;
                cur_count=1;
            }
        }
        String result=new String(wc);

        System.out.println(result.replaceAll(",", ""));
    }

}

```

测试用例

```xml
4
helloo
helllo
helllllllooooo
helllaabbccddnnn
hello
hello
hello
hellabbcddn
```

