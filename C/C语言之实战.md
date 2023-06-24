# C语言之实战

## 一、流程控制

### 1.1 寻找水仙花数

> “水仙花数（Narcissistic number）也被称为超完全数字不变数（pluperfect digital invariant, PPDI）、自恋数、自幂数、阿姆斯壮数或阿姆斯特朗数（Armstrong number），水仙花数是指**一个 3 位数，它的每个位上的数字的 3次幂之和等于它本身。**例如：1^3 + 5^3+ 3^3 = 153。”
现在请你设计一个C语言程序，打印出所有1000以内的水仙花数。
```c
#include <stdio.h>

int main() {
    for (int i = 100; i < 1000; ++i) {
        int a = i % 10, b = i / 10 % 10, c = i / 10 / 10;
        if (a * a * a + b * b * b + c * c * c == i) {
            printf("%d is Narcissistic number\n",i);

        }
    }
    return 0;
}

```

### 1.2 九九乘法表

```c
#include <stdio.h>

int main() {
    for (int i = 1; i < 10; ++i) {
        for (int j = 1; j < 10; ++j) {
            if (j<=i){
                printf("%d X %d = %-2d ",j,i,i*j);
            }
        }
        printf("\n");
    }
    return 0;
}

```

### 1.3 斐波那契数列解法（枚举法）

> 斐波那契数列（Fibonacci sequence），又称[黄金分割](https://baike.baidu.com/item/黄金分割/115896)数列，因数学家莱昂纳多·斐波那契（Leonardo Fibonacci）以兔子繁殖为例子而引入，故又称为“兔子数列”，指的是这样一个数列：**1、1、2、3、5、8、13、21、34、……**在数学上，斐波那契数列以如下被以递推的方法定义：*F*(0)=0，*F*(1)=1, *F*(n)=*F*(n - 1)+*F*(n - 2)（*n* ≥ 2，*n* ∈ N*）在现代物理、准[晶体结构](https://baike.baidu.com/item/晶体结构/10401467)、化学等领域，斐波纳契数列都有直接的应用，为此，美国数学会从 1963 年起出版了以《斐波纳契数列季刊》为名的一份数学杂志，用于专门刊载这方面的研究成果。

斐波那契数列：1，1，2，3，5，8，13，21，34，55，89...，不难发现一个规律，实际上从第三个数开始，每个数字的值都是前两个数字的和，现在请你设计一个C语言程序，可以获取斐波那契数列上任意一位的数字，比如获取第5个数，那么就是5。

```c
#include <stdio.h>

int main() {
    int target = 7, result;  //target是要获取的数，result是结果
    int i = 1, j = 1;
    if (target < 3) {
        printf("1");
    } else {
        for (int k = 2; k < target; k++) {
            result = i + j;
            i = j;
            j = result;
        }
        printf("%d", result);
    }
}

```

---

## 二、数组

### 2.1 冒泡排序算法（枚举法）

现在有一个int数组```int arr[10] = {3, 5, 7, 2, 9, 0, 6, 1, 8, 4};```，但是数组内的数据是打乱的，现在请你通过C语言，实现将数组中的数据按**从小到大**的顺序进行排列：

```c
#include <stdio.h>

int main() {
    int arr[10] = {3, 5, 7, 2, 9, 0, 6, 1, 8, 4};

    for (int i=0;i<10;i++){
        for (int j=0;j<10-i;j++){
            if(arr[j]<arr[j-1]){
                int tmp = arr[j-1];
                arr[j-1]=arr[j];
                arr[j]=tmp;
            }
        }
    }
    for (int i = 0;i<10;i++){
        printf("%d ",arr[i]);
    }
}

```

### 2.2：斐波那契数列解法（动态规划法）

```c
#include <stdio.h>

int main() {
    int target = 7;

    int dp[target];
    dp[1] = dp[0] = 1;
    for (int i = 2; i < 7; i++) {
        dp[i] = dp[i-1]+dp[i-2];
    }
    printf("%d",dp[target-1]);
}

```



### 2.3：打家劫舍

>你是一个专业的小偷，计划偷窃沿街的房屋。每间房内都藏有一定的现金，影响你偷窃的唯一制约因素就是相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警。
>
>给定一个代表每个房屋存放金额的非负整数数组，计算你 **不触动警报装置的情况下** ，一夜之内能够偷窃到的最高金额。

**示例 1：**

> 输入：[1,2,3,1]
> 输出：4
> 解释：偷窃 1 号房屋 (金额 = 1) ，然后偷窃 3 号房屋 (金额 = 3)。
>   	  偷窃到的最高金额 = 1 + 3 = 4 。

**示例 2：**

> 输入：[2,7,9,3,1]
> 输出：12
> 解释：偷窃 1 号房屋 (金额 = 2), 偷窃 3 号房屋 (金额 = 9)，接着偷窃 5 号房屋 (金额 = 1)。
>   	  偷窃到的最高金额 = 2 + 9 + 1 = 12 。

```c
#include <stdio.h>

int main() {
    int arr[] = {2,7,9,3,1}, size = 5, result;

    int dp[size];
    dp[0]=arr[0];
    dp[1]=arr[1];
    for (int i=2;i<size;i++){
        dp[i]=dp[i-1]>dp[i-2]+arr[i]?dp[i-1]:dp[i-2]+arr[i];
    }
    result = dp[size-1];

    printf("%d", result);
}

```

---

## 三、字符串

