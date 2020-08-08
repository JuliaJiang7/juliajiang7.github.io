---
title: KMP算法
date: 2020-07-17 16:17:33
tags:
  - String
  - Algorithm
  - KMP
categories: Algorithm
typora-copy-images-to: ..\pictures
---

本文主要介绍了 KMP算法的基本思想、代码、优化以及时间复杂度分析。

## 1. KMP算法思想

### 1.1 简介

**字符串匹配** 是计算机的基本任务之一。举例来说，即有一个字符串"BBC ABCDAB ABCDABCDABDE"，判断该字符串中是否包含另一个字符串"ABCDABD"？许多算法可以完成这个任务，[Knuth-Morris-Pratt算法](https://zh.wikipedia.org/wiki/克努斯-莫里斯-普拉特算法)（简称KMP）是最常用的之一。它以三个发明者命名，其中K代表著名科学家Donald Knuth。KMP算法可在一个字符串S内查找一个词P的出现位置，如果有返回P的起始索引，否则返回-1.

接下来，我会先举例对KMP算法的思路进行解释（不涉及任何代码）。

<!--more-->

### 1.2 基本思路

1. 首先，字符串"BBC ABCDAB ABCDABCDABDE"（称为文本串）的第一个字符与模式串"ABCDABD"的第一个字符，进行比较。因为B与A不匹配，所以搜索词后移一位。

   <img src="/pictures/bg2013050103.png" alt="img" style="zoom:67%;" />

2. 因为B与A不匹配，模式串再往后移。

   <img src="/pictures/bg2013050104.png" alt="img" style="zoom:67%;" />

3. 就这样，直到文本串有一个字符，与模式串的第一个字符相同为止。

   <img src="/pictures/bg2013050105.png" alt="img" style="zoom:67%;" />

4. 接着比较文本串和模式串的下一个字符，还是相同。

   <img src="/pictures/bg2013050106.png" alt="img" style="zoom:67%;" />

5. 直到字符串有一个字符，与搜索词对应的字符不相同为止。

   <img src="/pictures/bg2013050107.png" alt="img" style="zoom:67%;" />

6. 这时，最自然的反应是，将搜索词整个后移一位，再从头逐个比较。这样做固然可行，但是效率很差，因为你要把"搜索位置"移到已经比较过的位置，重比一遍。

   <img src="/pictures/bg2013050108.png" alt="img" style="zoom:67%;" />

7. 一个基本事实是，当空格与D不匹配时，你其实知道前面六个字符是"ABCDAB"。KMP算法的想法是，**设法利用这个已知信息，不要把"搜索位置"移回已经比较过的位置，继续把它向后移，这样就提高了效率**。

   <img src="/pictures/bg2013050107-1594975043794.png" alt="img" style="zoom:67%;" />

8. 怎么做到这一点呢？可以针对搜索词，算出一张《部分匹配表》（Partial Match Table）。这张表是如何产生的，后面再介绍，这里只要会用就可以了。

   <img src="/pictures/bg2013050109.png" alt="img" style="zoom:67%;" />

9. 已知空格与D不匹配时，前面六个字符"ABCDAB"是匹配的。查表可知，最后一个匹配字符B（不匹配字符的前一个字符）对应的"部分匹配值"为2，因此按照下面的公式算出向后移动的位数：

   ```java
   移动位数 = 已匹配的字符数 - 失配字符的前一位字符的部分匹配值
   ```

   因为 6 - 2 等于4，所以将搜索词向后移动4位。

10. 因为空格与Ｃ不匹配，搜索词还要继续往后移。这时，已匹配的字符数为2（"AB"），对应的"部分匹配值"为0。所以，移动位数 = 2 - 0，结果为 2，于是将搜索词向后移2位。

    <img src="/pictures/bg2013050110.png" alt="img" style="zoom:67%;" />

11. 因为空格与A不匹配，继续后移一位。

    <img src="/pictures/bg2013050111.png" alt="img" style="zoom:67%;" />

12. 逐位比较，直到发现C与D不匹配。于是，移动位数 = 6 - 2，继续将搜索词向后移动4位。

    <img src="/pictures/bg2013050112.png" alt="img" style="zoom:67%;" />

13. 逐位比较，直到搜索词的最后一位，发现完全匹配，于是搜索完成。如果还要继续搜索（即找出全部匹配），移动位数 = 7 - 0，再将搜索词向后移动7位，这里就不再重复了。

    <img src="/pictures/bg2013050113.png" alt="img" style="zoom: 67%;" />

14. 下面介绍《部分匹配表》是如何产生的。

    首先，要了解两个概念："前缀"和"后缀"。 "前缀"指除了最后一个字符以外，一个字符串的全部头部组合；"后缀"指除了第一个字符以外，一个字符串的全部尾部组合。

    <img src="/pictures/bg2013050114.png" alt="img" style="zoom:67%;" />

15. "部分匹配值"就是"前缀"和"后缀"的最大公共元素长度。以"ABCDABD"为例，计算过程如下：

    ![img](/pictures/20140725231726921)
    
16. "部分匹配"的实质是，有时候，字符串头部和尾部会有重复。比如，"ABCDAB"之中有两个"AB"，那么它的"部分匹配值"就是2（"AB"的长度）。搜索词移动的时候，第一个"AB"向后移动4位（字符串长度-部分匹配值），就可以来到第二个"AB"的位置。

    <img src="/pictures/bg2013050112-1594975805920.png" alt="img" style="zoom:67%;" />

到此，我们对于KMP的基本思路有了一个大致的了解，下一部分介绍KMP具体算法细节、代码及优化。

## 2. KMP算法代码及优化

假设现在我们面临这样一个问题：有一个文本串S，和一个模式串P，现在要查找P在S中的位置，怎么查找呢？

### 2.1 暴力匹配算法

如果用暴力匹配的思路，并假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置，则有：

- 如果当前字符匹配成功（即S[i] == P[j]），则i++，j++，继续匹配下一个字符；
- 如果失配（即S[i]! = P[j]），令i = i - (j - 1)，j = 0。相当于每次匹配失败时，i 回溯，j 被置为0。

暴力匹配代码如下：

```java
private int ViolentMatch(char[] s, char[] p){
    int sLen = s.length;
    int pLen = p.length;

    int i = 0, j = 0;
    while (i < sLen && j < pLen){
        if(s[i] == p[j]){
            i++;
            j++;
        }else{
            i = i - j + 1;
            j = 0;
        }
    }
    if(j == pLen){
        return i - j;
    }else{
        return -1;
    }
}
```

### 2.2 KMP算法

#### 2.2.1 求解 next 数组

##### 2.2.1.1 基本思路

前文已经计算过“部分匹配表”，即前缀和后缀的最大公共元素长度，如下图所示：

![img](/pictures/20140725231726921)

由前文可知，失配时，模式串向右移动的位数公式为：

```bash
移动位数 = 已匹配的字符数 - 失配字符的前一位字符的最大公共元素长度
```

由此我们发现，当匹配一个字符失配时，我们并不会考虑当前字符，而是看失配字符的前一个字符的最大公共元素长度，如此，便引出了next数组。**next 数组相当于最大长度值整体向右移动一位，然后初值赋为-1.** 因而，对于给定的模式串，它的最大长度及next数组分别如下：

![img](/pictures/20140728110939595)

求得next数组之后，失配时模式串向右移动的位数为：

```bash
移动位数 = 失配字符所在位置 - 失配字符对应的next值
```

##### 2.2.1.2 代码计算 next 数组

1. 如果**对于值 k, 已有 p0 p1, ..., pk-1 = pj-k pj-k+1, ..., pj-1，相当于next[j] = k**。究其本质，**next[j] = k 代表p[j] 之前的模式串子串中，有长度为k 的相同前缀和后缀**。有了这个next 数组，在KMP匹配中，当模式串中 j 处的字符失配时，下一步用next[j]处的字符继续跟文本串匹配，相当于模式串向右移动 j - next[j] 位。

2. 下面的问题是：已知next [0, ..., j]，如何求出next [j + 1]呢？

   - 若p[k] == p[j]，则next[j + 1] = next [j] + 1 = k + 1。

     如下图所示，假定给定模式串ABCDABCE，且已知next [j] = k（相当于“p0 pk-1” = “pj-k pj-1” = AB，可以看出k为2），现要求next [j + 1]等于多少？因为pk = pj = C，所以next[j + 1] = next[j] + 1 = k + 1（可以看出next[j + 1] = 3）。代表字符E前的模式串中，有长度k+1 的相同前缀后缀。

     ![img](/pictures/20140729182154066)

   - 若p[k ] ≠ p[j]，如果此时 p[next[k] ] == p[j]，则next[ j + 1 ] =  next[k] + 1，否则继续递归前缀索引k = next[k]，而后重复此过程。

     如下图所示，当pk != pj后，字符E前有多大长度的相同前缀后缀呢？很明显，因为C不同于D，所以ABC 跟 ABD不相同，即字符E前的模式串没有长度为k+1的相同前缀后缀，也就不能再简单的令：next[j + 1] = next[j] + 1 。所以，咱们只能去寻找长度更短一点的相同前缀后缀。  
     ![img](/pictures/20140729181940812)   

     结合上图来讲，若能在前缀“ p0 pk-1 pk ” 中不断的递归前缀索引k = next [k]，找到一个字符pk’ 也为D，代表pk’ = pj，且满足p0 pk'-1 pk' = pj-k' pj-1 pj，则最大相同的前缀后缀长度为k' + 1，从而next [j + 1] = k’ + 1 = next [k' ] + 1。否则前缀中没有D，则代表没有相同的前缀后缀，next [j + 1] = 0。

     那为何递归前缀索引k = next[k]，就能找到长度更短的相同前缀后缀呢？ 这又归根到next数组的含义。**我们拿前缀 p0 pk-1 pk 去跟后缀pj-k pj-1 pj匹配，如果pk 跟pj 失配，下一步就是用p[next[k]] 去跟pj 继续匹配，如果p[ next[k] ]跟pj还是不匹配，则需要寻找长度更短的相同前缀后缀，即下一步用p[ next[ next[k] ] ]去跟pj匹配。**

3. 综上，可以通过递推求得 next 数组，代码如下：

   ```java
   public int[] getNext(char[] p){
       int pLen = p.length;
       int[] next = new int[pLen];
       next[0] = -1;
       int k = -1;
       int j = 0;
       while (j < pLen - 1){
           // p[k] 表示前缀；p[j] 表示后缀
           if(k == -1 || p[j] == p[k]){
               ++k;
               next[++j] = k;
           }else{
               k = next[k];
           }
       }
       return next;
   }
   ```

##### 2.2.1.3 总结 next 数组含义

1. 代表失配字符之前的字符串中，有多大长度的相同前缀后缀。
2. 在某个字符失配后，next 值会告诉你下一步匹配中，模式串应该跳到哪个位置。如果next [j] 等于0或 -1，则跳到模式串的开头字符；若next [j] = k 且 k > 0，代表下次匹配跳到 j 之前的某个字符，而不是跳到开头，且具体跳过了k 个字符。

#### 2.2.2 KMP 算法

根据上文的分析，KMP算法的代码如下：

```java
public int kmp(char[] s, char[] p, int[] next){
    int i = 0, j = 0;
    int sLen = s.length;
    int pLen = p.length;
    while (i < sLen && j < pLen){
        if(j == -1 || s[i] == p[j]){
            i++;
            j++;
        }else{
            j = next[j];
        }
    }
    if(j == pLen){
        return i - j;
    }else{
        return -1;
    }
}
```

#### 2.2.3 next 数组的优化

行文至此，我们全面了解了KMP算法的基本思路、流程、代码以及next 数组的求解，但忽略了一个小问题。

比如，如果用之前的next 数组方法求模式串“abab”的 next 数组，可得其 next 数组为 -1 0 0 1，当它跟下图中的文本串去匹配的时候，发现 b 跟 c 失配，于是模式串右移 j - next[j] = 3 - 1 = 2位。

<img src="/pictures/8394323_1308075859Zfue.jpg" alt="8394323_1308075859Zfue" style="zoom:67%;" />

右移2位后，b又跟c失配。事实上，因为在上一步的匹配中，已经得知p[3] = b，与s[3] = c失配，而右移两位之后，让p[ next[3] ] = p[1] = b 再跟s[3]匹配时，必然失配。问题出在哪呢？

<img src="/pictures/8394323_13080758591kyV-1595069195969.jpg" alt="8394323_13080758591kyV" style="zoom:67%;" />

问题出在不该出现p[j] = p[ next[j] ]。为什么呢？理由是：当p[j] != s[i] 时，下次匹配必然是p[ next [j]] 跟s[i]匹配，如果p[j] = p[ next[j] ]，必然导致后一步匹配失败（因为p[j]已经跟s[i]失配，然后你还用跟p[j]等同的值p[next[j]]去跟s[i]匹配，很显然，必然失配），所以不能允许p[j] = p[ next[j ]]。如果出现了p[j] = p[ next[j] ]咋办呢？如果出现了，则需要再次递归，即令next[j] = next[ next[j] ]。

因此，求解 next 数组的代码优化如下：

```java
public int[] getNextval(char[] p){
    int pLen = p.length;
    int[] next = new int[pLen];
    next[0] = -1;
    int k = -1;
    int j = 0;
    while (j < pLen - 1){
        // p[k] 表示前缀；p[j] 表示后缀
        if(k == -1 || p[j] == p[k]){
            ++k;
            ++j;
            if(p[j] != p[k]){
                next[j] = k;
            }else{
                // 因为不能出现p[j] = p[next[j]]，所以当出现时需要继续递归，k = next[k] = next[next[k]]
                next[j] = next[k];
            }

        }else{
            k = next[k];
        }
    }
    return next;
}
```

## 3. KMP 算法时间复杂度分析

我们先来回顾一下KMP算法的流程，假设现在文本串S匹配到 i 位置，模式串P匹配到 j 位置：

1. 如果j = -1，或者当前字符匹配成功（即S[i] == P[j]），都令i++，j++，继续匹配下一个字符；
2. 如果j != -1，且当前字符匹配失败（即S[i] != P[j]），则令 i 不变，j = next[j]。此举意味着失配时，模式串P相对于文本串S向右移动了j - next [j] 位。

我们发现如果某个字符匹配成功，模式串首字符的位置保持不动，仅仅是i++、j++；如果匹配失配，i 不变（即 i 不回溯），模式串会跳过匹配过的next [j]个字符。整个算法最坏的情况是，当模式串首字符位于i - j的位置时才匹配成功，算法结束。

所以，如果文本串的长度为n，模式串的长度为m，那么匹配过程的时间复杂度为O(n)，算上计算next的O(m)时间，KMP的整体时间复杂度为O(m + n)。

## 4. 例题：实现 strStr() [28]

题目来源：[28. 实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)；另一种动态规划在我的另一篇博客：[这里]().

### 题目描述

实现 strStr() 函数。

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:

```bash
输入: haystack = "hello", needle = "ll"
输出: 2
```

示例 2:

```bash
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```


说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

### 代码

````java
public int strStr(String haystack, String needle) {
    if(needle.length() == 0){return 0;}
    char[] s = haystack.toCharArray();
    char[] p = needle.toCharArray();
    int[] next = getNextval(p);
    return kmp(s, p, next);
}

private int kmp(char[] s, char[] p, int[] next){
    int i = 0, j = 0;
    int sLen = s.length;
    int pLen = p.length;
    while (i < sLen && j < pLen){
        if(j == -1 || s[i] == p[j]){
            i++;
            j++;
        }else{
            j = next[j];
        }
    }
    if(j == pLen){
        return i - j;
    }else{
        return -1;
    }
}

private int[] getNextval(char[] p){
    int pLen = p.length;
    int[] next = new int[pLen];
    next[0] = -1;
    int k = -1;
    int j = 0;
    while (j < pLen - 1){
        // p[k] 表示前缀；p[j] 表示后缀
        if(k == -1 || p[j] == p[k]){
            ++k;
            ++j;
            if(p[j] != p[k]){
                next[j] = k;
            }else{
                // 因为不能出现p[j] = p[next[j]]，所以当出现时需要继续递归，k = next[k] = next[next[k]]
                next[j] = next[k];
            }

        }else{
            k = next[k];
        }
    }
    return next;
}
````

## 5. 参考文献

1. https://blog.csdn.net/v_july_v/article/details/7041827

2. [http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html](http://www.ruanyifeng.com/blog/2013/05/Knuth–Morris–Pratt_algorithm.html)



