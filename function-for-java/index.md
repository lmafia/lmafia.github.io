# 用 Java 来实现高阶函数


如何用 Java 使用高阶函数

<!--more-->

### Funciton 接口的定义

```Java
/**
 * Represents a function that accepts one argument and produces a result.
 *
 * <p>This is a <a href="package-summary.html">functional interface</a>
 * whose functional method is {@link #apply(Object)}.
 *
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {}
```

### 代码示例

实现功能:

- 2 次函数 apply
- 3 次函数 apply

```Java
public static void main(String[] args) {

        Function<Function<Integer, Integer>, Function<Integer, Integer>> applyTwice =
                    //f(f(x))
            f -> x -> f.apply(f.apply(x));

        Function<Function<Integer, Integer>, Function<Integer, Integer>> applyThrice =
            f -> {
                return x -> {
                    //f(f(f(x)))
                    return f.apply(f.apply(f.apply(x)));
                };
            };

        System.out.println(applyThrice.apply( x -> x + 1 ).apply(0));

    }
```

### 第一层：最外层 Function

```Java
Function<Function<Integer, Integer>, Function<Integer, Integer>>
```

这个类型的输入是 `Function<Integer, Integer>`
输出也是 `Function<Integer, Integer>`。

换句话说，这个高阶函数的作用是：

- 接收一个函数 `Funciotn<Interger, Integer>`
- 返回一个新的函数 `Function<Integer, Integer>`，

### 第二层：输入参数

`Function<Integer, Integer>`

这是 Java 函数接口, 输入输出参数都是 `Integer`

比如：

`Function<Integer, Integer> increment = x -> x + 1;`
`Function<Integer, Integer> square = x -> x * x;`

这些都是 `Function<Integer, Integer>` 类型的函数。

### 第三层 返回值 `Function<Integer, Integer>`

最后 `applyThrice` 应该返回的是一个`Function<Integer, Integer>`
拆解下 `applyThrice.apply( x -> x + 1 ).apply(0)`

1.  `applyThrice.apply( x -> x + 1 )` 返回了:
    `x -> ( ( ( x + 1 ) + 1 ) + 1 )`
2.  `applyThrice.apply( x -> x + 1 ).apply(0)` 就是把 0 作为输入 x
    `( ( ( 0 + 1 ) + 1 ) + 1 )` 结果为 `3

