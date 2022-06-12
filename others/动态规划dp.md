## 1. 斐波那契数列

###  1.1 递归解法 只能通过部分用例

- Python版本

```python
def fib(n):
    # 递归的终止条件
    if n == 1 or n == 2:
        return 1
    else:
        # 递推关系
        return fib(n-1) + fib(n-2)

# 输入输出
n = int(input())
print(fib(n))
```

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int fib(int n){
        // 递归的终止条件
        if(n==2 || n ==1){
            return 1;
        }
        else{
            // 递推关系
            return fib(n - 1) + fib(n - 2);
        }
    }

    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int ans = fib(n);
        System.out.println(ans);
    } 
}
```



分析递归调用树，时间复杂度为2的n次方，存在着大量的重复计算，因此用递归版本大概率会超时，只能通过部分测试用例，但是递归调用的方式方便理解，可在暴力递归的版本下，再作缓存优化



### 1.2 动态规划版本一：用数组储存值

- Python版本

```python
def fib(n):
    if n == 1 or n == 2:
        return 1
    # 创建一个长度为n的数组, 并初始化为0
    dp = [0 for _ in range(n)]
    # 递归的边界条件即为数组的起始迭代条件
    dp[0] = 1
    dp[1] = 1
    # 填值  因为索引0和1位置已经填好了值 因此从2开始
    for i in range(2, n):
        # 递推关系，由迭代版本转换 
        dp[i] = dp[i-1] + dp[i-2]
    return dp[n-1]

# 输入输出
n = int(input())
print(fib(n))
```

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int fib(int n){
        // 递归的终止条件
        if(n==2 || n ==1){
            return 1;
        }
        // 创建一个长度为n的数组, 并初始化为0
        int[] dp = new int[n];
        dp[0] = 1;
        dp[1] = 1;
        // 循环填值
        for(int i = 2; i < n; i++){
            dp[i] = dp[i - 1] + dp[i - 2];
        }
        // 返回数组中最后一个值
        return dp[n-1];
    }

    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int ans = fib(n);
        System.out.println(ans);
    } 
}
```

### 1.3 动态规划版本二，空间优化

- Python版本

```python
def fib(n):
    if n == 1 or n == 2:
        return 1
    # 通过对缓存数组迭代计算分析，得到n的值仅依赖n-1和n-2的值，历史记录不会再使用，因此可以将数组优化为两个变量
    left, right = 1, 1
    # 填值  因为索引0和1位置已经填好了值 因此从2开始
    for i in range(2, n):
        left, right = right, left + right
    return right

# 输入输出
n = int(input())
print(fib(n))
```

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int fib(int n){
        // 边界条件
        if (n == 1 || n == 2){
            return 1;
        }
        // 通过对缓存数组迭代计算分析，得到n的值仅依赖n-1和n-2的值，历史记录不会再使用，因此可以将数组优化为两个变量
        // 赋初始值 也即递归版本的边界条件
        int left = 1;
        int right = 1;
        for(int i = 2; i < n; i++){
            // 临时变量储存上两个之和
            int temp = left + right;
            // 求和之后左边的值不再需要，因此把右边更大的值赋值给左边
            left = right;
            // 更新右边值
            right = temp;
        }
        return right;
    }
    
    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int n = sc.nextInt();
        int ans = fib(n);
        System.out.println(ans);
    } 
}
```

## 2. 矩阵的最小路径和

### 2.1 递归解法 只能通过部分用例

- Python 版本

```python
def get_min_dist(grid):
    # idx_x 和 idx_y 为二维数组索引
    idx_x = len(grid) - 1
    idx_y = len(grid[0]) - 1
    # 创建一个process函数，返回走到任意位置x，y的最小和
    return process(grid, idx_x, idx_y)

def process(grid, idx_x, idx_y):
    # 边界条件 当x和y都到达原点时候 直接返回当前值
    if idx_x == 0 and idx_y == 0:
        return grid[0][0]
    # 当x为0时候，到达二维数组左边界 即只能往上走 即idx_y - 1  结果还需加上当前位置的值
    elif idx_x == 0 and idx_y > 0:
        return process(grid, 0, idx_y - 1) + grid[0][idx_y]
    # 当y为0时候，到达二维数组上边界 即只能往左走 即idx_x - 1 结果还需加上当前位置的值
    elif idx_y == 0 and idx_x > 0:
        return process(grid, idx_x - 1, 0) + grid[idx_x][0]
    else:
        # 当既不在左边界也不在上边界，因此要选择左边一个值idx-1和上面一个值idx-y中的最小值 结果再加上当前位置的值
        return min(process(grid, idx_x - 1, idx_y), process(grid, idx_x, idx_y - 1)) + grid[idx_x][idx_y]
    
    
# 输入输出
x, y = map(int, input().split())
grid = []
for _ in range(x):
    rows = [int(x) for x in input().strip().split()]
    grid.append(rows)
print(get_min_dist(grid))
```

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int minPath(int[][] grid){
        // idx_x 和 idx_y 为二维数组索引
        int idx_x = grid.length - 1;
        int idx_y = grid[0].length - 1;
        // 创建一个process函数，返回走到任意位置x，y的最小和
        return process(grid, idx_x, idx_y);
    }
    
    public static int process(int[][] grid, int idx_x, int idx_y){
        //边界条件 当x和y都到达原点时候 直接返回当前值
        if (idx_x == 0 && idx_y == 0){
            return grid[0][0];
        }
        // 当x为0时候，到达二维数组左边界 即只能往上走 即idx_y - 1  同时还需加上当前位置的值
        if (idx_x == 0 && idx_y > 0){
            return process(grid, 0, idx_y - 1) + grid[0][idx_y];
        }
        // 当y为0时候，到达二维数组上边界 即只能往左走 即idx_x - 1 同时还需加上当前位置的值
        if (idx_y == 0 && idx_x > 0){
            return process(grid, idx_x - 1, 0) + grid[idx_x][0];
        }
        else{
            // 当既不在左边界也不在上边界时，要选择左边一个值idx-1和上面一个值idx-y中的最小值 同时再加上当前位置的值
            return Math.min(process(grid, idx_x - 1, idx_y), process(grid, idx_x, idx_y - 1)) + grid[idx_x][idx_y];
        }
    }
    
    
    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int x = sc.nextInt();
        int y = sc.nextInt();
        int[][] grid = new int[x][y];
        for(int i = 0; i < x; i++){
            for(int j = 0; j < y; j++){
                grid[i][j] = sc.nextInt();
            }
        }
        int ans = minPath(grid);
        System.out.println(ans);
    } 
}
```

### 2.2 动态规划 用二维数组储存值

- Python 版本

```python
def dp(grid):
    # 初始化二维数组 结果集
    x = len(grid)
    y = len(grid[0])
    dp = [[0] * y for _ in range(x)]
    # 赋值边界条件 根据递归边界条件转换
    dp[0][0] = grid[0][0]
    # 对二维数组第一列赋值 
    for i in range(1, x):
        dp[i][0] = grid[i][0] + dp[i-1][0]
    # 对二维数组第一行赋值 
    for j in range(1, y):
        dp[0][j] = grid[0][j] + dp[0][j-1]
    # 其他位置需要依据当前位置 左边和上面的最小值加上当前值
    for i in range(1, x):
        for j in range(1, y):
            dp[i][j] = min(dp[i-1][j], dp[i][j-1]) + grid[i][j]
    return dp[x-1][y-1]

# 输入输出
x, y = map(int, input().split())
grid = []
for _ in range(x):
    rows = [int(x) for x in input().strip().split()]
    grid.append(rows)
print(dp(grid))
```

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int dp(int[][] grid){
        // 初始化二维数组 结果集
        int x = grid.length;
        int y = grid[0].length;
        int[][] dp = new int[x][y];
        // 赋值边界条件 根据递归边界条件转换
        // 索引为0 0  时 直接赋值
        dp[0][0] = grid[0][0];
        // 对二维数组第一列赋值 
        for(int i = 1; i < x; i++){
            dp[i][0] = dp[i-1][0] + grid[i][0];
        }
        // 对二维数组第一行赋值 
        for(int j = 1; j < y; j++){
            dp[0][j] = dp[0][j-1] + grid[0][j];
        }
        
        // 其他位置需要依据当前位置 左边和上面的最小值加上当前值
        for(int i = 1; i < x; i++){
            for(int j = 1; j < y; j++){
                dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + grid[i][j];
            }
        }
        return dp[x-1][y-1];
    }
    
    
    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int x = sc.nextInt();
        int y = sc.nextInt();
        int[][] grid = new int[x][y];
        for(int i = 0; i < x; i++){
            for(int j = 0; j < y; j++){
                grid[i][j] = sc.nextInt();
            }
        }
        int ans = dp(grid);
        System.out.println(ans);
    } 
}
```

###  2.3 动态规划 二维数组降为一维数组

- Java版本

```java
import java.util.Scanner;

public class Main{
    public static int dp(int[][] grid){
        // 初始化一维数组 结果集
        int x = grid.length;
        int y = grid[0].length;
        int[] dp = new int[y];
        // 赋值边界条件 根据递归边界条件转换
        // 索引为0 0  时 直接赋值
        dp[0] = grid[0][0];
        // 对一维数组初始化  即第一行
        for(int j = 1; j < y; j++){
            dp[j] = dp[j-1] + grid[0][j];
        }
        
        for(int i = 1; i < x; i++){
            // 第一个值为数组边界值 直接累加
            dp[0] = dp[0] + grid[i][0];
            for(int j = 1; j < y; j++){
                // 其他位置需要依据当前位置 左边和上面的最小值加上当前值
                dp[j] = Math.min(dp[j-1], dp[j]) + grid[i][j];
            }
        }
        return dp[y-1];
    }
    
    
    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        int x = sc.nextInt();
        int y = sc.nextInt();
        int[][] grid = new int[x][y];
        for(int i = 0; i < x; i++){
            for(int j = 0; j < y; j++){
                grid[i][j] = sc.nextInt();
            }
        }
        int ans = dp(grid);
        System.out.println(ans);
    }
}
```

## 3 玩牌高手

### 3.1 暴力递归 可能只通过部分用例

- Java版本

```java
import java.util.Scanner;

public class Main {

    public static int getMaxScore(int[] nums){
        // 数组索引位置 即对应的轮数 0对应1轮，1对应2轮 .... n-1 对应n轮
        int idx = nums.length - 1;
        // 返回任意idx处可以获取的最大分数
        return process(nums, idx);
    }

    public static int process(int[] nums, int idx){
        // 在递归调用idx-3或idx-1的时候可能出现索引越界 因此先判断，索引越界直接返回0
        if(idx < 0){
            return 0; // 题意中如果1-3轮中如果值小于零则归零
        }
        // 如果只剩下一轮 即idx=0时，如果大于0直接返回，小于0则为零
        if(idx == 0){
            return  nums[0] < 0 ? 0 : nums[0];
        }else{
            // 其他轮数可获得最大分数为 上一轮最大分数和该轮拿到点数之和 与 上上上轮拿到最大分数 比较取最大值
            return Math.max(process(nums, idx-1) + nums[idx], process(nums, idx-3));
        }
    }


    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        String input = sc.next();
        String[] strArr = input.split(",");
        int[] nums = new int[strArr.length];
        for(int i = 0; i < nums.length; i++){
            nums[i] = Integer.parseInt(strArr[i]);
        }

        int ans = getMaxScore(nums);
        System.out.println(ans);
    }
}
```

### 3.2 动态规划 数组缓存

- Java版本

```java
import java.util.Scanner;
public class Main {

    public static int dp(int[] nums){
        // 创建一个长度为n的数组存放结果
        int n = nums.length;
        int[] dp = new int[n];
        // 边界条件赋值
        // 当为1轮的时候 大于零则 直接赋值  小于零则为0
        dp[0] = nums[0] < 0 ? 0 : nums[0];
        if(n == 1){
            return dp[0];
        }
        // 当为第二轮和第三轮 如果最大值大于零直接赋值 小于零则为0
        // 当为二轮时，用第一轮可获取的最大值加上第二轮拿到的点数 大于零直接赋值，小于零则为零
        dp[1] = Math.max(dp[0]+nums[1], 0);
        if(n == 2){
            return dp[1];
        }
        // 当为三轮时，用第二轮可获取的最大值加上第三轮拿到的点数 大于零直接赋值，小于零则为零
        dp[2] = Math.max(dp[1]+nums[2], 0);
        if(n == 3){
            return dp[2];
        }
		
        for(int i = 3; i < n; i++){
            // 从第四轮开始，上一轮最大分数和该轮拿到点数之和 与 上上上轮拿到最大分数 比较取最大值
            dp[i] = Math.max(dp[i-1]+nums[i], dp[i-3]);
        }

        return dp[n-1];
    }


    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        String input = sc.next();
        String[] strArr = input.split(",");
        int[] nums = new int[strArr.length];
        for(int i = 0; i < nums.length; i++){
            nums[i] = Integer.parseInt(strArr[i]);
        }

        int ans = dp(nums);
        System.out.println(ans);
    }
}
```

### 3.3 动态规划 数组压缩

- Java版本

```java
import java.util.Scanner;
public class Main {

    public static int dp(int[] nums){
        // 从动态规划版中发现计算n轮值仅依赖n-1轮和n-3轮值，又因计算n+1轮时需依赖n-2轮的值(n+1)-3=(n-2)
        // 因此可将缓存数组优化为三个变量 储存中间结果  优化与斐波那契数类似
        int n = nums.length;
        int left = 0;
        int mid = 0;
        int right = 0;
        // 边界条件赋值
        // 当为1轮的时候 大于零则直接赋值  小于零则为0
        left = nums[0] < 0 ? 0 : nums[0];
        if(n == 1){
            return left;
        }
        // 当为二轮时，用第一轮可获取的最大值加上第二轮拿到的点数 大于零直接赋值，小于零则为零
        mid = Math.max(left + nums[1], 0);
        if(n == 2){
            return mid;
        }
        // 当为三轮时，用第二轮可获取的最大值加上第三轮拿到的点数 大于零直接赋值，小于零则为零
        right = Math.max(mid + nums[2], 0);
        if(n == 3){
            return right;
        }

        for(int i = 3; i < n; i++){
            // 从第四轮开始，上一轮最大分数和该轮拿到点数之和 与 上上上轮拿到最大分数 比较取最大值
            // 临时变量储存当前 轮数的结果
            int temp = Math.max(right+nums[i], left);
            // 交换位置
            left = mid;
            mid = right;
            right = temp;
        }

        return right;
    }


    public static void main(String[] args){
        // 输入输出
        Scanner sc = new Scanner(System.in);
        String input = sc.next();
        String[] strArr = input.split(",");
        int[] nums = new int[strArr.length];
        for(int i = 0; i < nums.length; i++){
            nums[i] = Integer.parseInt(strArr[i]);
        }

        int ans = dp(nums);
        System.out.println(ans);
    }
}
```



## 总结要点

动态规划本质就是一种空间换时间优化思想，在暴力递归中，可能存在**重复计算**，因此考虑把计算的结果存储在一种数据结构中，通常为数组

难点：递推关系和边界条件的确定

递归的终止条件一般可以思考方向：1. 题目相关要求； 2. 数组边界

递推关系：难难点

考虑相邻之间关系，比如在斐波那契数中，如果要计算n的值，那么得先计算出 n-1的值和n-2的值，因此有

f(n) = f(n-1) + f(n-2)    其中f表示斐波那契数列中第n项的值



在矩阵最小路径和中，要计算x，y处的最小和距离，得先计算出它左边 (x-1，y)和它上面(x，y-1) 两者的最小和距离的最小值加上当前位置的值即为到(x，y)处的最小值，即有

f(x,y) = min ( f(x-1,y), f(x, y-1) )  +  arr\[x]\[y]   其中f定义为在(x,y)处的路径最小和



在玩牌高手中，要获取当前轮数的最大值，需要得到上一轮所能得到的最大值 和 上上上一轮所能获取的最大值 ，即有

f(x) = max(f(x-1) + arr\[x]   ,   f(x-3) )    其中f定义为在x轮处可获取的最大点数



**有了递推关系就可以写暴力递归，有了暴力递归就可以用数组作缓存，有了数组缓存可以根据暴力递归的边界条件，迭代填值，再根据往数组中填值的逻辑考虑是否可对数组作进一步空间优化**