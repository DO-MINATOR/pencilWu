### [实现 strStr()](https://leetcode-cn.com/problems/implement-strstr/)

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

这是一道字符串匹配问题，实现类似于C++中的string.substring()函数，如图所示，假定haystack串长度为m，模式串长度为n，根据模式串在原串中是否匹配，返回第一次匹配成功时的位置。

![image-20200920142250217](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920142250217.png)

#### 滑动窗口

首先，这是最容易想到的方法，如图，窗口大小为n，窗口在原串中向前滑动，每次比较花费O(n)的时间复杂度，最坏情况下，即每次都是在窗口最后一个字符处匹配失败，则窗口向前移动一格，直至无剩余待匹配字符或者找到匹配位置。

![image-20200920142442990](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920142442990.png)

总体时间复杂度为O(n*(m-n))，空间复杂度O(1)。代码如下：

```c
class Solution {
public:
    int strStr(string haystack, string needle) {
        int n = needle.length();
        int m = haystack.length();
        if (n == 0)
            return 0;
        for (int i = 0; i < m - n + 1; ++i) {
            for (int j = i, cur = 0;; ++j) {
                if (cur == n)
                    return i;
                if (haystack[j] == needle[cur])
                    cur++;
                else
                    break;
            }
        }
        return -1;
    }
};
```

#### KMP

滑动窗口技术只是作为一个引子来引出今天的讨论对象KMP，KMP是三位发明人的名字首字母，我也是在leetcode上提交该题目时发现有这项技术时才去了解的，不得不说，KMP还真的不好理解，我也是花费了5个小时时间，先后在leetcode看kmp的题解，以及在知乎上，甚至是bilibili，不过都是在next数组的求解过程上卡住了。

好吧，首先来说一下KMP的大致思路，在原来的滑动窗口方法中，当某个字符匹配失败时，我们就把窗口向前移动了一位，但其实这是极为低效的做法，具体来看一个例子。

![image-20200920145156524](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920145156524.png)

上图中，模式串第一次匹配时，是在模式串中的第5号位('f')处，这时候应该如何向前滑动呢，可以看到 <u>aa</u>b<u>aa</u>f 中下划线部分字符串是重复的，这里产生了前后缀的概念，即从模式串中第0个位置起向后，和从模式串中第n-1个位置起向前，如果出现相同的字符子串，则称这两个为相同前后缀，有了前后缀的好处是，我可以直接从A位置直接跳转到D位置。（那你直接跳了那么多格，中间就一定不会有匹配的可能性了吗？答案是：一定不会，因为如果中间有可能匹配上的话，那么前后缀长度一定更长，换句话说，当前已经确定的前后缀一定是最长的前后缀，所以，放心大胆的向前迈进吧！）

知道向前跳转的原理后，相信也就明白了为何这种方法能够减小时间开销。

那么跳转的时机是什么，如果当前字符匹配失败后，那么查询模式串中前一位字符所对应的前后缀长度，并记为a，那么下一次匹配时，原串下标不变，模式串下标变为a。可以看到，由于匹配过程中，任意位置都有可能匹配失败，那么就需要对模式串中每一个位置都求解其前后缀长度，这步过程就是常说的求解next数组，也是最难的一步，如果处理不当，很可能就失配了。同样以aabaaf为例，求解其next数组：

![image-20200920151351549](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920151351549.png)

首先，`next[0]=0`，有的资料这一步初始化为-1，多少都无关紧要，这里0的意思是当前位置所对应的前后缀长度为0，只要记住定义并且前后保持一致即可，注意这里前后缀长度不能等于它自身长度，如果相等的话，那么将无法实现向前跳转，这一步读者自己试一试就知道。

![image-20200920151402811](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920151402811.png)

第二个next，可以看到aa字符串的前后缀长度为1

![image-20200920151411412](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920151411412.png)

第三个next，可以看到aab字符串的前后缀长度为0

![image-20200920151421634](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920151421634.png)

第四个next，可以看到aaba字符串的前后缀长度为1

![image-20200920151430594](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920151430594.png)

第五个next，可以看到aaba字符串的前后缀长度为2，以此类推...

通过以上例子可以实现针对原串为`aabaabaaf`，模式串为`aabaaf`的匹配算法了。

但是，next数组的求解过程远不及此，我初次接触KMP时，用的就是这一个例子，当时我的next求解过程是这样的：

```c
int next[n];
next[0] = 0;//初始化next[0]
int cmp = 0;//当前前后缀长度
for (int i = 1; i < n; i++) {
    if (needle[i] == needle[cmp]) {
        next[i] = ++cmp;
    } else {
        cmp=0;
        if(needle[i] == needle[cmp])
            next[i] = ++cmp;
        else
            next[i] = 0;
    }
}
```

但这其中有一个隐含的bug，即针对如下例子，将会求出错误的next数组。

![image-20200920152732146](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920152732146.png)

求解出该模式串的next为0101210，

![image-20200920152738022](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920152738022.png)

然后根据next进行位置跳转，此时在c处发生匹配失败，由于此时认为c前面的a对应的前后缀长度为1，所以下一次就会从模式串的第1位置开始比较，实际上正确的next数组值为0101220，即下图所示，根据此next数组才能找到正确的匹配结果。

![image-20200920152947375](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920152947375.png)

求解正确next数组的过程是十分艰难的，我也是冥思苦想了很久才想通...，初次看到正确的求解next过程长这样时，我也是百思不得其解。

```c
int next[n];
next[0] = 0;
for (int i = 1, j = 0; i < n; ++i) {//j为当前前后缀长度
    while (j > 0 && needle[i] != needle[j])//可匹配失败时，为什么要这样做，完全看不明白，为什么不是j--呢
        j = next[j - 1];
    if (needle[i] == needle[j])//匹配成功好说，next[i]直接等于++j
        next[i] = ++j;
    else
        next[i] = 0;
}
```

其实要想想清楚这个问题，需要画出一般情况：

![image-20200920154155629](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920154155629.png)

假如此时模式串正在求解第i个字符对应的前后缀长度，如果第i位置和第j位置字符相等，那么`next[i]=++j`，这一步没什么问题，如果不相等，那么前缀长度j就要先前回溯，试图寻找较短一点的相等前后缀。

![image-20200920154200513](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920154200513.png)

j回退到了j-1的位置，如果只因为j回退就认为可以判断i、j位置字符是否相等显然是错误的，因为不能保证此时j前面的字符串和i前面的字符串是相等的。

![image-20200920154205280](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920154205280.png)

接下来试图寻找next[j]，这一步的含义是看第j位置的前后缀长度，如果>0，则说明有前后缀子串，那么上图中下划线部分就可以两两交换了。

![image-20200920154209969](https://imagebag.oss-cn-chengdu.aliyuncs.com/img/image-20200920154209969.png)

我们只需要判断第j位置字符是否和i位置字符相等，如果等于，那么next[i]=j，如果不等，继续重复上述过程，直至j=0或者找到j位置字符等于i位置字符。这样，就写出了下述求解next数组的过程。

```c
int next[n];
next[0] = 0;
for (int i = 1, j = 0; i < n; ++i) {
    if (needle[i] == needle[j])
        next[i] = ++j;
    else {
        while (j > 0 && needle[i] != needle[j])
            j = next[j - 1];
        if (needle[i] == needle[j])
            next[i] = ++j;
        else
            next[i] = 0;
    }
}
```
进一步化简得：

```c
int next[n];
next[0] = 0;
for (int i = 1, j = 0; i < n; ++i) {
    while (j > 0 && needle[i] != needle[j])
        j = next[j - 1];
    if (needle[i] == needle[j])//注意跳出时，可能是因为j==0了，这是就要判断
        next[i] = ++j;
    else
        next[i] = 0;
}
```

完整KMP代码如下：

```c
class Solution {
public:
    int strStr(string haystack, string needle) {
        int m = haystack.length();
        int n = needle.length();
        if (n == 0)
            return 0;
        int next[n];
        next[0] = 0;
        for (int i = 1, j = 0; i < n; ++i) {
            while (j > 0 && needle[i] != needle[j])
                j = next[j - 1];
            if (needle[i] == needle[j])
                next[i] = ++j;
            else
                next[i] = 0;
        }
        for (int i = 0, j = 0; i < m;) {
            if (haystack[i] == needle[j]) {
                i++;
                j++;
            } else {
                if (j != 0)
                    j = next[j - 1];
                else
                    i++;
            }
            if (j == n)
                return i - n;
        }
        return -1;
    }
};
```

#### rehash

这种方法是leetcode官方给出的，看了一下，觉得挺巧妙的，针对滑动窗口技术，主要的时间花销在判断窗口内字符串和模式串是否相等上了，这一步时间复杂度是O(n)，能否化简到O(1)呢，思考hashcode()函数的作用，当两对象内容一致时，算出来的hash值相同（当然了，有可能发生hash碰撞，只需要比对equals()方法）。那么有没有可能求解hash的过程是O(1)复杂度呢。

由于该题目已经告知全是小写字母，因此将a-z映射到0-25的数字上，求解该字符串的hash值就可以看作是基数位权展开法的求解过程，除了第一次预处理是n的复杂度，以后每次重新计算即rehash的过程都是O(1)的时间复杂度（因为我们只需要将最高位减去，然后总数乘以权数25，再加上新进数字）。

该方法需要注意的点，一是溢出问题，通过%模运算可以解决。但这样又会导致hash碰撞，结合equals()或c++的`==`判断即可解决。

