
给定一个数组 nums，有一个大小为 k 的滑动窗口从数组的最左侧移动到数组的最右侧。你只可以看到在滑动窗口 k 内的数字。滑动窗口每次只向右移动一位。

返回滑动窗口最大值。

示例:

输入: nums = [1,3,-1,-3,5,3,6,7], 和 k = 3

输出: [3,3,5,5,6,7] 

解释: 

      滑动窗口的位置                最大值
    ---------------               -----
    [1  3  -1] -3  5  3  6  7       3
     1 [3  -1  -3] 5  3  6  7       3
     1  3 [-1  -3  5] 3  6  7       5
     1  3  -1 [-3  5  3] 6  7       5
     1  3  -1  -3 [5  3  6] 7       6
     1  3  -1  -3  5 [3  6  7]      7
注意：

你可以假设 k 总是有效的，1 ≤ k ≤ 输入数组的大小，且输入数组不为空。

### 思路一：对大小为k的窗口切片取max，在res中保存结果。但是时间复杂度为O(N*k)

### 代码：
```py
class Solution():
    def maxSlidingWindow(self, nums, k):
        if not nums or not k:
            return []
        res = []
        for i in range(len(nums)-k+1):
            res.append(max(nums[i:i+k]))
        return res
```
### 思路二：使用双端队列deque，对新进入窗口的值判断是否可清理前值（前值不大于新值则用while+pop清理），每个元素只过window一次，所以总体上它没有把O(n)的复杂度嵌套在外层的循环里。

时间复杂度为O(N)

### 代码：
```py
class Solution():
    def maxSlidingWindow(self, nums, k):
        # 判空
        if not nums:return[]
        window, res = [], []
        # 递归nums
        for i, x in enumerate(nums):
            # 判定window中首位是否出窗口左界
            if i>=k and window[0] <= i-k:
                window.pop(0)
                
            # 判定nums中序号为window最右侧的值是否不大于新值x
            while window and nums[window[-1]] <= x:
                window.pop()
            window.append(i)
            
            # 满足第一个窗口大小，开始统计
            if i >= k-1:
                res.append(nums[window[0]])
        return res
```


https://thenewstack.io/linkedin-code-review/
