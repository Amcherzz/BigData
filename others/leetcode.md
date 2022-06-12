### 



#### [3. 无重复字符的最长子串](https://leetcode.cn/problems/longest-substring-without-repeating-characters/)

子串和子序列

方法 ：以某个字符结尾来计算

往前推： 

1） 和当前字符一样

2）i-1 往前推多远距离

以上两个更近的位置

map查询字符位置

```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        if not s or s == '':
            return 0
        # 记录映射关系
        map = {}
        # 前一个位置
        pre = -1
        # 收集结果
        ans = 0
        for i in range(len(s)):
            # 往左推 要么碰见一样的字符，要么碰见上一个位置的边界
            pre = max(pre, map.get(s[i], -1))
            map.update({s[i]:i})
            cur = i - pre
            ans = max(ans, cur)
        return ans
```

#### [4. 寻找两个正序数组的中位数](https://leetcode.cn/problems/median-of-two-sorted-arrays/)   **

下标换函数 两个有序数组  第k个小的数     to 49

#### [5. 最长回文子串](https://leetcode.cn/problems/longest-palindromic-substring/)

manacher算法 求解  需要掌握



#### [7. 整数反转](https://leetcode.cn/problems/reverse-integer/)

#### [10. 正则表达式匹配](https://leetcode.cn/problems/regular-expression-matching/)

```python
# dp[-1] 没算过
# dp[0] false
# dp[1] true


def isMath(s, p):
    return is_valid(s, p) and process(s, p, 0, 0)

# 参数检查  
def is_valid(s, p):
    if '.' in s or '*' in s:
        return False
    if '**' in p or p.startwith('*'):
        return False
    return True


# p的pi位置不是为*
def process(s, p, si, pi):
    if si == len(s):
        if pi == len(p):
            return True
        if pi + 1< len(p) and p[pi+1] == '*':
            return process(s, p, si, pi+1)
        return False
    if pi == len(p):
        return si == len(s)
    # si没有终止， pi没有终止 pi+1 不是*
    if pi+1 >= len(p) and p[pi+1] != '*':
        return s[si] == p[pi] or p[pi] == '.' and process(s, p, si+1, pi+1)
    
    # pi+1 不是*
    if p[pi] != '.' and s[si] != p[pi]:
        return process(s, p, si, pi+2)
    
    # * 0份
    if process(s, p, si, pi+2):
        return True
    
    while si < len(s) and (s[si] == p[pi] or p[pi] == '.'):
        if process(s, p, si+2, pi+2):
            return True
        si += 1
    return False
    
```

