# C语言之实战

## 一、流程控制

### 1. 寻找水仙花数

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

### 2. 九九乘法表

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

