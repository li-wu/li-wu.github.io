---
title: 判断字符串中是否所有字符唯一
date: 2014-08-19 19:40:29 +0800
categories: algorithm
---

Question
Implement an algorithm to determine if a string has all unique characters What *if you can not use additional data structures* ?

1.Assume the char set is ASCII(if not we need to increase the storage size):
```
public static boolean isUnique(String str){
    boolean[] char_set=new boolean[256];
    for(int i=0;i<str.length();i++{
        int val=str.charAt(i);
        if(char_set[val])return false;
        char_set[val]=true;
    }
    return true;
}
```
The time complexity is O(n) and space complexity is O(n).
2.Or you can use a bit vector to reduce the space usage.Assuming that the string is only lower case 'a' through 'z'.
```
public static boolean isUniqueChars(String str){
    int checker=0;
    for(int i=0;i<str.length();i++){
        int val=str.charAt(i)-'a';
        if((checker&(1<<val))>0)
            return false;
        checker|=(1<<val);
    }
    return true;
}
```
