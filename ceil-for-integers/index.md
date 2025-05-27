# 整数向上取整公式证明


数学公式在编程里的应用: 整数向上取整公式，常用于分页，分片的场景

<!--more-->

## 背景

在用 AI 编程的时候，让它给我生成一个分页处理的工具类:

```java
 public static <T> List<T> paginate(List<T> list, int page, int pageSize) {
    if (list == null || list.isEmpty()) {
        return Collections.emptyList();
    }
    if (pageSize <= 0) {
        throw new IllegalArgumentException("pageSize must be greater than 0");
    }

    int total = list.size();
    int totalPages = (total + pageSize - 1) / pageSize;

    int safePage = Math.max(1, page);
    if (safePage > totalPages) {
        return Collections.emptyList();
    }

    int skip = (safePage - 1) * pageSize;

    return list.stream()
            .skip(skip)
            .limit(pageSize)
            .collect(Collectors.toList());
}
```

在 Review 生成代码的时候，发现求总页数的计算让我有点好奇，为什么它用这个公式
求总页数: 总数除于分页大小, 如果余数大 0，就加一。在编程里，整数相除一般是向下取整，我第一反应想到的逻辑是：先除，后取模，根据取模结果判断是否 +1，btw，看 Mybatis Plus 的代码，发现他也是这么设计的:

```java
    /**
     * 当前分页总页数
     */
    default long getPages() {
        if (getSize() == 0) {
            return 0L;
        }
        long pages = getTotal() / getSize();
        if (getTotal() % getSize() != 0) {
            pages++;
        }
        return pages;
    }
```

所以 AI 给我的这个计算方式着实让我好奇:

`(total + pageSize - 1) / pageSize;`

我很好奇，为什么是加`b-1` 和除以`b`

所以后来，我向 Chapt 老师请教了如何证明这个公式是有效的。

## 证明过程

“向上取整除法”的核心公式是：

$$ \left\lceil \frac{a}{b} \right\rceil = \frac{a + b - 1}{b} $$

前提：

- a ≥ 0, b > 0
- 除法采用整数除法（向下取整）
- 所有变量为整数

### 目标

我们要证明这个公式：

$$ \left\lceil \frac{a}{b} \right\rceil = \frac{a + b - 1}{b} $$

右边是整数除法（向下取整），但通过 +b-1 的技巧实现了向上取整效果。

### 整数除法基础（商和余）

设 a = bq + r，其中：

- q = a / b 是向下取整的商
- r = a % b 是余数，满足 0 ≤ r < b

因此：

- 如果 r = 0：说明 a 被 b 整除，结果无需进位。
- 如果 r > 0：则需进一位，即向上取整。

### 定义

定义：

$$
\left\lceil \frac{a}{b} \right\rceil =
\begin{cases}
q, & \text{if } r = 0 \\
q + 1, & \text{if } r > 0
\end{cases}
$$

### 分情况讨论

#### 当 r = 0 时

- a = bq
- a + b - 1 = bq + b - 1 = b(q + 1) - 1

代入

$$ \frac{a + b - 1}{b} $$

得到:

$$
\frac{a + b - 1}{b} = \frac{bq + b - 1}{b}
q + \left( \frac{b - 1}{b} \right)
$$

因为整数除法是向下取整的， (b - 1) < b , 所以 (b - 1)/b 为 0

所以证明得

$$
 \frac{a + b - 1}{b} = \frac{b(q + 1) - 1}{b} = q
$$

#### 当 r > 0 时

- a = bq + r, 0 < r < b
- a + b - 1 = bq + r + b - 1 = b(q + 1) + (r - 1)

代入

$$ \frac{a + b - 1}{b} $$

得到:

$$
\left( \frac{a + b - 1}{b} \right)
= \left( \frac{b(q + 1) + (r - 1)}{b} \right)
= (q + 1) + \left( \frac{r - 1}{b} \right)
$$

讨论 (r - 1) 和 b
因为 0 < r < b , 所以 -1 < r - 1 < b - 1 < b 得 r-1 < b
所以

$$ \left( \frac{r - 1}{b} \right) = 0 $$

所以证明得, 当 r > 0 时

$$ \frac{a + b - 1}{b} = (q + 1) + 0 = q + 1 $$

## 结语

因为只是涉及整数的运算，用不到浮点运算比 JDK `Math`提供的 `ceil` 更适合分页的场景
除此之外，还能用:

- 底层的 `内存块` / `缓冲区`申请
  > btw, 和公司的嵌入式同事强哥提到的时候，他已经使用过了，对于嵌入式开发来说比较常用
- 文件的`分块` ，以实现断点续传或分块并行下载
- 分布式任务调度的 `分片计算`
- 大数据任务计算的 `批处理`

