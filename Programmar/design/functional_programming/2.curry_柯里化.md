# 柯里化 (curry)

## 前言

函数式编程中，了解并运用柯里化是必经之路，因为它带来的好处太多了，堪称是必备工具。

本文就柯里化本身进行讲解并提供实践引导。

由于笔者写本文的时候大部分时间在用 python 做运维脚本，所以在这里也使用 python 语言进行举例。

## 一个例子

假设我们有一个函数，这个函数可以根据输入的信息返回一个打招呼的句子。

```python
def greet(greeting, name: str) -> str:
    return f"{greeting} {name}"
```

那我们必然要这么调用它

```python
print(greet("Good morning", "Sam"))
print(greet("Good morning", "Lily"))
```

当然我们也可以优化一下，给早上好这个语句定义一个变量，以方便我们一次修改，到处生效。

```python
greeting_morning = "Good morning"
print(greet(greeting_morning, "Sam"))
print(greet(greeting_morning, "Lily"))
```

但这样的函数并不很好用，试想一下，如果你的应用程序里到处都有打招呼的功能：

1. 早上对用户说“一个美好的早晨”
2. 中午对用户说“吃饭了吗”
3. 晚上对用户说“还没休息吗”

那这个 `greet` 函数是不是会到处被使用，并且还带着它的变量到处跑。

由于这种代码实在是太频繁的被使用，为了不让变量到处跑导致我们的维护成本变高，而且有时候**我们的任务压根就不是关注怎么打招呼，而是跟谁打招呼**，所以我们不得不重新写一些常用函数：

```python
def greet_morning(name: str) -> str:
    return greet("Good morning,", name)


def greet_afternoon(name: str) -> str:
    return greet("Good afternoon,", name)
```

这样我们调用时就可以仅关注对谁打招呼了：

```python
print(greet_morning("Sam"))
print(greet_afternoon("Lily"))
```

但是现在，假设又有一个新场景，**用户希望自定义打招呼的方式**，当然我们可以采用传统方式，把原始 `greet` 函数拿出来用，这样确实符合场景要求，至少在 Java 中笔者一定会这么干，但在天然就支持函数式编程的语言中，我们有一个新的方案：

```python
def curry_greet(greeting: str) -> Callable:
    def curried_greet(name: str) -> str:
        return greet(greeting, name)

    return curried_greet
    
# 假设这是用户保存在咱们系统中的输出，不管他来自哪，数据库，redis，甚至是一级缓存都可以
greeting_user_define = "Have a nice day,"
greet_user = curry_greet(greeting_user_define)
```

于是我们在需要使用打招呼的地方只要这么用就行了：

```python
print(greet_user("Sam"))
```

现在我们仔细看看这个方案。我们写了一个函数，这个函数接收了一个参数，并返回了一个新的函数，我们只需要向新的函数输入另一个参数，就可以调用原始函数 `greet` 了。调用 `greet` 函数时所使用的参数就是 `curry_greet` 函数的参数和 `curried_greet` 函数的参数，最终把 `grret` 函数的结果返回给调用者。

函数的第一个参数实际上是被隐含在生成出来的函数的记忆空间内的，我们一般称之为**闭包记忆**。

**我们将原来可以一次性调用就完成的功能分离成了需要调用两次**。

这么做看起来似乎是多次一举？其实这么做为我们带来了巨大的好处，因为从此之后我们可以根据 `greet` 函数生成各种各样的函数，而且不再需要一个一个用 `def` 关键字定义了，所有的函数都是直接来自 `greet` 函数的派生。

我们使用的这个方案，在专业名词上被称之为**柯里化**，即 **curry**。

## 通用 curry

上一节的 curry 方案只能针对一个 `greet` 函数起作用，如果要让其他的函数也使用这个方案，我们就不得不再写个类似的 `curry_xxx`，且如果参数列表不止两个元素的时候，可就让人头大了。

所以我们需要将 curry 应用到函数上的方案本身抽象出来成为一个函数。

### 学习简单的 curry

为了方便讲解与学习，我们先从一个最简单的、有一堆缺陷的、但满足了基础功能的方案上开始：

```python
def curry(fn: Callable) -> Callable:
    len_in = len(inspect.signature(fn).parameters)
    cu_args = []

    def curried(*args):
        for arg in args:
            cu_args.append(arg)
        if len(cu_args) >= len_in:
            return fn(*cu_args)
        return curried

    return curried
```

先看看怎么使用，还是针对之前的打招呼例子:

```python
# 柯里化
curried_greet = curry(greet)
greeting_user_define = "Have a nice day,"
# 函数声明
greet_user = curried_greet(greeting_user_define)

# 函数调用
print(greet_user("Sam"))
```

看起来和之前的用法没什么区别，但是我们现在已经有能力用这个 `curry` 函数去处理其他函数了，且对函数的参数列表长度要求没有限制。

我们回过头来看看这个 `curry` 函数的实现：

1. 首先检查传入的函数的参数列表，确定参数列表长度，这是为了**让我们好知道什么时候才调用原始函数**。
2. 然后在闭包记忆空间定义一个现阶段已经接收到的参数列表，所有已经接收到的参数将被按顺序放在这里，供之后调用原始函数使用。
3. 定义一个新的函数，这个函数接收任意数量的参数，在这个函数中：
    1. 把接收到的参数加入到记忆空间内的参数列表。
    2. 检查记忆空间中已经收到的参数数量是否已经可以调用原始函数了，如果已经满足条件，则调用原始函数，并返回函数的输出。
    3. 如果没有满足条件，则返回函数自身，等待下一次调用继续补充参数。
4. 返回新函数。

很简单，对吧。

### 解决白色污染

前面说了，这个简单的函数有一堆缺陷，我们来看看如果我们重复调用生成出来的函数会怎么样：

```python
print(greet_user("Sam"))
print(greet_user("Linda"))
```

我们的预期是这样调用会向两个用户打招呼，但结果很不好，python 为我们报出了错误

```shell
TypeError: greet() takes 2 positional arguments but 3 were given
```

究其原因，是因为我们这么用本质上还是在用同一片记忆空间，以至于第二次调用时，上一次调用的记忆没有被清除，在记忆空间中有 `"Have a nice day,"`, `"Sam"`, `"Linda"` 三个参数，当三个参数都被扔进了 `greet` 函数中就出问题了。

这意味着我们生成的 `greet_user` 函数是一次性的，就跟一次性塑料袋一样，用完就得扔，这个问题不解决，我们的项目中很快就会遭到白色污染的冲击。

所以我们写了一个新的 curry 函数：

```python
def curry(fn: Callable) -> Callable:
    len_in = len(inspect.signature(fn).parameters)

    def curried(*args):
        if len(args) >= len_in:
            return fn(*args)
        return curry_v2(fn, *args)

    return curried


def curry_v2(fn: Callable, *args) -> Callable:
    len_in = len(inspect.signature(fn).parameters)

    def curried_v2(*_args):
        cu_args = []
        for arg in args:
            cu_args.append(arg)
        for arg in _args:
            cu_args.append(arg)
        if len(cu_args) >= len_in:
            return fn(*cu_args)
        return curry_v2(fn, *cu_args)

    return curried_v2
```

新的函数看起来似乎变复杂了，但是它也确实解决了白色污染的问题，让我们看看它是怎么做的：

1. `curry` 函数比起之前简单了不少，记忆空间中没有参数列表了，它只负责解析函数参数列表长度与返回一个柯里化函数 `curried`。
2. `curried` 函数也必之前简单了不少，只要判断传入的参数是否已经满足调用条件，满足则调用，否则将函数与收到的参数列表当作参数调用 `curry_v2` 函数并返回。
3. `curry_v2` 是解决白色污染的关键
    1. 首先它也要解析参数列表长度，以方便判断何时执行原始函数。
    2. 定义一个 `curried_v2` 函数并返回。
4. `curried_v2` 函数中
    1. 我们可以发现之前的参数列表被移动到这了，但这并不代表记忆空间中就没有现有参数列表了，现有参数列表被整合到了 `curry_v2` 函数的第二个参数上了。
    2. 先将记忆参数列表中的参数加入到 `cu_args` 中，再将新参数列表中的参数加入到 `cu_args` 中，合成为现有参数列表。
    3. 判断现有参数列表是否已经满足调用原函数的条件，满足则调用，**不满足则执行 `curry_v2` 函数并返回**。

该版本函数与上一个版本的函数上最显著的区别是

1. 每一次调用函数，不满足条件就会响应一个新的 `curried` 函数，而不是函数本身，这样每一个生成的柯里化函数都能更新记忆空间中的参数列表。
2. 现有的参数列表会在柯里化函数被调用期间生成，所以函数可以重复使用，杜绝了白色污染。

这个函数还有优化空间，读者可以自行优化。

## 一个实战例子

既然手上拿了锤子，那就得锤一下钉子，这里提供一个案例，咱们可以亲手将它好好地优化一下。

假设我们有一个需求，在控制台打印出有颜色字符，这里提供四个函数：

```python
def print_red(mess):
    print('\033[91m' + mess + '\033[0m')


def print_green(mess):
    print('\033[92m' + mess + '\033[0m')


def print_yellow(mess):
    print('\033[93m' + mess + '\033[0m')


def print_blue(mess):
    print('\033[94m' + mess + '\033[0m')
```

我们的任务就是把这四个函数压缩一下，用我们的新方法，压缩后可以长这样：

```python
@curry
def color_print(color_code, mess: str):
    print('\033[' + color_code + mess + '\033[0m')


print_red = color_print("91m")
print_green = color_print("92m")
print_yellow = color_print("93m")
print_blue = color_print("94m")
```

这里我们取了个巧，使用了 python 给予我们的权利，装饰器，如果其他语言没有这个能力，老老实实地用 `curry` 函数即可。

## 代码案例

[xiahaohai123/pythonStudy](https://github.com/xiahaohai123/pythonStudy)

## 参考文献

1. [第 4 章: 柯里化（curry）](https://llh911001.gitbooks.io/mostly-adequate-guide-chinese/content/ch4.html)