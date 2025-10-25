KMP算法可以将暴力字符串匹配算法的时间复杂度由O（nm）压缩到O（n+m)
# 原理
text string : a b c a a b c a b b c a b c
pattern string : a b b c a b
首先先引入next数组的概念
## next数组：
next[i]表示子串中i位置之前  不包含自身且不包含空字符串 的 前后缀相等的最大字符数
其中next[0]由于第0个字符前面已经没有字符，规定next[i] = -1
例如 "a    b    a    b    c   a    b " 
next  -1   0    1    2    0   1    2
稍后我们讲解next数组是如何递推的
## KMP算法实现部分      
现在我们已经有了next数组，开始用KMP算法进行匹配
假设某一时刻从主串的start位置开始匹配
当pattern string的第j个字符 与  text string的第i的字符不匹配时，
暴力求解：
   将匹配 ***重置*** 为patter string的第0个字符与text string的start+1位置的字符开始重新匹配
KMP算法：
   将匹配 ***跳转*** 为pattern string的第next [ j ]字符与text string的i位置继续匹配

---为什么这样做---？
主串
       - - - |- - - - - - - |- - - - - - - - - - - -
            
          s        i
字串
           |- - - - - - -| - - - ...       
          `0        j 
当主串的i位置与字串的j位置不相等时，说明匹配已经失败，那么就要向前跳转，那为什么
可以i不变，让字串的next[j]位置继续与主串的i位置匹配呢？
我们来看字串
      
    ` - - - - - - - - - - - - - - - - - - -`
                                       `j`
我们已知next[j],也就是从0到j-1的这个串中，最长的前后缀相等的值为next[j]，我们这里记为k，
那么从0-j-1的这个串中前K个字符组成的串与后K个字符组成的串是相同的
    ` |- - - - - -| - - - - - - |- - - - - -|      -`
                                               `j 
    我们把相同的地方标记为1   
    ` |1 1 1 1 1 1| - - - - - - |1 1 1 1 1 1|      -`
                                               `j
接下来把主串一起放进来
 `- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
         `|1 1 1 1 1 1|- - - - - -|1 1 1 1 1 1|-`
                                           `j
注意：主串与字串是在主串的第i个位置，字串的第j个位置，才出现不匹配，这说明在这之前主串与字串都是相同的
`- - - - -|1 1 1 1 1 1|- - - - - -|1 1 1 1 1 1|- - - - - - - - - - -
         `|1 1 1 1 1 1|- - - - - -|1 1 1 1 1 1|- - - - - -...` 
                                           `j
因此现在就能看到四个相等的部分：
字串的前k个与后k个，主串与字串匹配部分的前k个与后k个
由于主串与字串前面部分是匹配的，
因此字串右移再去匹配主串的时候，实际上是拿字串的前缀（字串）去匹配字串的后缀（主串）
例如偏移2位：
`- - - - -|1 1 **1 1 1 1|- - - - - -|1 1 1 1 1 1**|- - - - - - - - - - -
                                               `i`
            `|**1 1 1 1 1 1|- - - - - -|1 1 1 1** 1 1|- - - - - -...` 
                                                  `j
实际上是要求`**`之间的相同，也就是字串的前缀（字串`**`之间的部分）与字串的后缀（主串`**`之间的部分）相等
而这能不能做到呢？
由于我们已知了next[j]的值，所以前后缀最长的相等的部分是K，而不是图中的 j-2，正确的的偏移是下图：
` - - - -|1 1 1 1 1 1|- - - - - -|1 1 1 1 1 1|- - - - - - - - - - -
                               `|1 1 1 1 1 1|- - - - - -|1 1 1 1 1 1| - - - ...` 
                                                                `j`
这样就可以实现最高效的错位
因为：如果错位比图中的少，也就要求前后缀相同的最大数大于k，与next[j]是矛盾的
       如果错位比图中多,有可能匹配失败，并且还会遗漏情况
即：图中的错位就是最近的 可能出现匹配的情况
## next数组实现部分


字串
           |- - - - - - -| - - - ...       
          `0        m n 
         `-1        k`
  假如我已知：next[m] = k,现在要求next[n]
  首先：next[n]是与pattern[n]无关的1，因为是求0到n-1的前后缀
  而0到m-1前后缀最大的相等为k，用1来表示前后缀相等的部分，中间串省略
          |1 1 -  1 1 -| - - - ...       
          `0  ⬆    m n 
         `-1        k`
 那么就有两种情况
  1. pattern[m] == pattern[⬆],那么next[n] = next[m] + 1
  2. pattern[m] != pattern[⬆],那么j就需要跳转
    如何跳转呢？
我们把字串扩大
  |1 1 1 1 1 1 1 1 1  -  - 1 1 1 1 1 1 1 1 -| - - - ...       
` 0               ⬆                m n 
 首先第一次判断时 由于next[m] = k,所以⬆是字串第k+1个数，也就是pattern[k]
 pattern[m] != pattern[⬆]，那么我就需要重新再找前后缀相等的最大数量
 注意到，左右两部分的 `1`,是两个相同的串
 那么next[⬆] 就表示1所组成的串，最大的前后缀相等数量，这里用2表示
 |2 2 1 1 1 1 1 2 2  -  - 2 2 1 1 1 1 2 2 -| - - - ...       
`0   ⬆                             m n 
同理，又会出现四处相同的部分，重点是这样的情况是整个字符串不包含m的情况下所能达到的最大的前后缀相等的数量
因此再判断判断pattern[m] == pattern[⬆] ，其中⬆ = next[上一个⬆]
这样一直匹配下去，如果⬆ == -1,说明已经到头了，无法继续跳转，自然next[n] = 0;
## 代码部分
### 求next数组
  ```
  vector<int> buildNext(const string& pattern) {
    int n = pattern.size();
    vector<int> next(n, 0);
    if (n == 0) return next;//特殊判断空串
    next[0] = -1;
    if (n == 1) return next;//特殊判断大小为1的串
    next[1] = 0;
    int j = 0;
    //next[0],next[1]都赋初始值，从2开始遍历
    for (int i = 2; i < n; i++) {
        while (j != -1 && pattern[j] != pattern[i - 1])
          {
           j = next[j];
	       /*不相等时，字串往前跳转，跳转到j跳转next[j]的位置，因为next[j]
             已经是不含pattern[i-1]的最大的前后缀相等的数了
           */  
          } 
        next[i] = j + 1;
        j++;
    }
    return next;
}
  ```
### KMP实现
```
vector<int> kmp(const string& text, const string& pattern) {
    vector<int> res;
    int n = text.size(), m = pattern.size();
    if (n == 0 || m == 0 || m > n) return res;

    vector<int> next = buildNext(pattern);
    int x = 0, y = 0;
    while (x < n) {
        if (y == -1 || text[x] == pattern[y]) {
            x++;
            y++;
            if (y == m) {
                res.push_back(x - y); // 匹配位置
                y = next[y - 1];      // 继续匹配后续部分
            }
        }
        else {
            y = next[y];
        }
    }
    return res;
}
```
如果只想匹配第一个串
```
int kmp(const string& text,const string& pattern)
{
int n = text.size(),m = pattern.size();
vector<int> next = buildNext(pattern);
int x= 0 , y = 0;
while( x < n && y < m)
{
  if(text[x] == pattern[y])
  {
    x++;
    y++;
  }
  else if(y > 0)
  {
    y = next[y];
  }
  else
  {
    x++;
  }
}
return y == m ? x-y : -1;
}
```
