# IO Monad

## 前置知识

Monad

## 前言

我们在编程的时候经常遇到这么个问题:

1. 我的程序该如何面对复杂多样的世界而保持稳定？
2. 现实世界出了问题我们的问题定位速度该如何加快？

IO Monad 作为 Monad 的一种，专门用来解决对外部资源的访问操作，即与现实世界打交道。

它的设计初衷就是将计划与具体工作分开，计划为纯函数，具体工作为脏函数。

## 举例

### 抽象举例

> 我们有个函数，里面包含了三个步骤:
>
> - 从屏幕读入字符串A
> - 从数据库读入字符串B
> - 在屏幕上输出A+"-"+B
>
> 可以看到，这三个步骤都是脏的。为了保证函数的纯度，我们原则上应该给它们套上盒子。可是，最后一个步骤的执行依赖之前步骤的结果，应该怎么做呢？答案是:
> ```javascript
> IO(readScreen()).flatMap(
>   A=> IO((A, readDB()))
> ).flatMap(
>   (A,B)=> IO(print(A+"-"+B))
> )
> ```
> 可以看到，我们先把第一个步骤的答案装进了IO的盒子。然后，我们使用flatMap，把结果以假如的形式拿了出来（表示假如我们已经读到了A），然后装进了新的盒子里。注意到，新的盒子里面有A和readDB()两个数。此时，我们再次调用flatMap，表示假如我们已经读到了A和B，那么我们把它们连起来打印一下。注意到，这个操作又被装进了一个新的盒子里。
>
> 因此，这一通操作下来，这个函数是一个纯函数。我们没有和现实世界进行交互，没有施加影响，一切都还没有发生。**只有我们真正打开最后一个盒子的时候，这三个步骤才会真正被执行。**

### 比喻举例

> 考虑你和你的好朋友在上机器学习课的时候讨论某个网络游戏应该怎么通关。你的好朋友告诉你，其实很简单，只要按照如下的步骤去做：
>
> 1. 你出门右转去市场上找一个绿衣服的玩家，他会给你一个字条。
> 2. 这个字条上面有三种可能，根据不同情形，你要去A，B，C三个地方中的某一个，找到红衣服的NPC，跟她买一个锤子。
> 3. 拿着这个锤子和字条，你可以去D，找到怪兽，用锤子打死他。
>
> 注意到：
>
> - 游戏过程不是纯函数的，因为它依赖绿衣服玩家和红衣服NPC给你的字条和锤子。如果绿衣服玩家给你不同的字条，你就会走不一样的游戏路径。
> - 游戏攻略没有对"现实世界"产生影响（这里的现实世界不是指教室，而是指网络游戏的那个世界），因为你们在上机器学习课，你还没有来得及登录你的账号，实践你朋友的攻略。
>
> 所以游戏攻略是一个你准备之后（回宿舍）执行，但是暂时还没有执行的计划。**这个计划的产生过程是一个纯函数，即不管你听你好朋友说多少遍，你最后得到的计划是一样的**。但是计划执行的过程可能会充满变数，因为你要和充满脏东西的游戏世界打交道，你不知道绿衣服玩家会给你一个什么样的字条。
>
> 在这个例子里面，每个步骤都产生了一个新的Monad盒子，里面包含的内容则是你当前步骤的游戏状态（拿了什么字条，或者买了一个锤子）。
>
> - 我们需要用盒子把游戏状态包裹起来，因为在机器学习课上谈论的字条和锤子，不是真正的字条和锤子（在宿舍里玩游戏的时候拿到的才是真的）。换句话说，盒子装的字条，是写在纸面上的抽象字条，没有真正实例化；游戏里拿到的字条，是真正的字条，有真正的取值。
> - 盒子的双面胶flatMap在这里就是把几个步骤粘在了一起，形成了一个完整的游戏攻略，让你回去一步一步操作把游戏打通关。
>
> 那我们也可以清晰地看到，为什么flatMap一定要把盒子打开，取出其中的值，然后进行下一步。因为这里的值是指游戏里拿到的真正的字条，只有那个玩意儿才有真实的取值，可以帮助你判断下一步要做什么；但是如果没有把盒子打开，只看纸面上的抽象字条，那你永远不知道到底要去A，B，C的哪个地方。

## IO Monad

**注意，本段均来自作者: 前端夜点心，为了方便阅读，不使用引用框。**

IO 就是 Input/Output，副作用无非是对外部世界的 Input（读）和 Output（写），所以我们用 IO Monad 来命名这种包裹着对外部世界读写行为的 Monad 函子。

那如何把一段原本有副作用的逻辑变得没有副作用呢？这其实用不上什么骚操作，IO Monad 的核心思想就是：“无论遇到什么副作用都不用怕，把它包在一个函数里晚点再管它"

例如下面是一个典型的会产生副作用的函数 `showReview`，它首先访问并读取了外部存储 `localStorage`，然后对读到的数据做了一些处理并在控台打印出来，正好把 I/O 两项副作用全占了：

```javascript
function showReview() {
    const dataStr = localStorage.getItem('前端夜点心的数据');
    const data = JSON.parse(dataStr);
    const reviewData = data.review;
    const reviewOutput = reviewData.map(
        (count, index) => `第${index}篇文章的阅读量是${count}`
    ).join(',');
    console.log(reviewOutput);
}
```

上面的代码中 2、8 两行是具有副作用的，一旦代码报错发生我们就无法拍着胸脯担保 3-7 行的纯逻辑没有错误——他们被混合在了一起，难以自证清白。

所以首先我们要做的是把包含副作用的两行代码通过函数包装起来：

```javascript
const readFromStorage = () => localStorage.getItem('前端夜点心的阅读量');
const writeToConsole = console.log;
```

然后把 `readFromStorage` 用一个名为 `IO` 的 Monad 包裹起来：

```javascript
const storageIO = new IO(readFromStorage);
```

上面的 storageIO 包裹的 `value` 不是从 `localStorage` 读取的不可预测的值，而仅仅是 `readFromStorage` 这个函数，该函数本身作为一个值是可控且确定的。
换句话说，我们包裹了一个确定的「动作」，比如“挥拳”，而不是它的不可预测的「结果」，比如打赢了别人，比如被对方暴击，比如进局子。
至此，“挥拳”这个动作并没有被执行，只是被我们暂存了起来。

接下来我们把后续的一系列数据处理都声明成函数，并通过 `map` 方法调用组织起来

```javascript
// 解析 JSON
const parseJSON = string => JSON.parse(string);

// 读取 review 字段
const getReviewProp = data => data.review;

// 把 review 字段拼装成字符串
const mapReview = reviewData => reviewData.map(
    (count, index) => `第${index}篇文章的阅读量是${count}`
).join(',');

// 组合上面的这些函数，得到新的 Monad
const task = storageIO
    .map(parseJSON)
    .map(getReviewProp)
    .map(mapReview)
```

上面我们得到的 `task` 同样是一个包含了「动作」而不是「结果」的 Monad，它把一系列动作组合起来：`parse` -> `getReviewProps` -> `mapReview`。
就好比想好了“挥拳”然后“观察”然后“逃跑”然后“躲起来”，但并没有执行任何一个上述动作。这些动作仍然停留在「动机」阶段，没有对潜在的对手产生任何实质影响。

最后终于到了执行的时刻了！我们通过 IO Monad 特有的 fork 方法订阅了 writeToStorage 函数，同时执行了包裹在 Monad 中的组装好的函数：从 localStorage 读取了值，一通操作以后把它输出到了控台：

```javascript
task.fork(writeToConsole);
```

`fork` 方法就好比扣动了板机，把我们早已在脑海里想好的一系列动作一股脑打了出去。与最初的 `showReview` 函数不同的是，在这之前我们已经推敲了每一个动作（中间步骤的纯函数），保证它们准确无误。所以如果出现问题，就可以断定是在 `readFromStorage` 或者 `writeToConsole` 中出现的了。

这就是 IO Monad 用法的一个简单的例子。当然之所以被称为 Monad，肯定少不了 `chain` 方法：

### chain

我们还是以从 `localStorage` 读取数据为例，先定义一个可以根据输入的 key 返回包裹了读取这个 key 的存储值的 `IO` 的函数：

```javascript
const readByKey = key => new IO(() => localStorage.getItem(key));
```

以此为基础，就可以通过第一个 key 读取数据，根据读到的数据获得第二个 key，然后借助 `chain` 方法读取第二个值并返回：

```javascript
const task = readByKey('firstKey') // 通过第一个 key 读取存储
    .map(parseJSON)
    .map(v => v.key) // 获取第二个 key
    .chain(readByKey) // 通过第二个 key 读取存储
    .map(parseJSON)
```

一切的副作用都被控制在 `readByKey` 这个函数中，使得错误易于定位。

### IO Monad 函子实现

```typescript
function compose<T, U>(f: (x: T) => U, g: () => T) {
    return () => f(g());
}

function join<T>(io: IO<T>): () => T {
    return io.effect();
}

export class IO<T> {
    static of<T>(x: T) {
        return new IO(() => x);
    }

    effect: () => T;

    constructor(effect: () => T) {
        this.effect = effect;
    }

    map<U>(f: (x: T) => U) {
        return new IO(compose(f, this.effect));
    }

    chain<U>(f: (x: T) => IO<U>) {
        const g = compose(f, this.effect);
        return new IO(compose(join, g));
    }

    fork(callback: (x: T) => void) {
        callback(this.effect());
    }
}
```

## 为什么要分离纯函数与脏函数

在正常的生产环境中，纯函数是不可能出错的，所以定位的时候不需要管纯函数，直接看脏函数里面哪里出问题了就好，大大加快定位速度。

为什么纯函数在生产环境中不可能出错:

1. 纯函数在接收确定的输入后一定会产生恒定不变的输出。
2. 所以纯函数部分一定已经适配了业务了。
3. 如果是纯函数有问题那就意味着测试不行。

## 如何理解 IO Monad 代码?

### map 方法的本质

IO Monad 自己本身是函子，其工作原理来自一个理论，**纯函数可以合并**。所以 `map` 方法的本质就是函数合并，将一个函子内的纯函数取出来与新的纯函数合并，构造出一个新的函子，直到最终有某个外部调用将这个函子打开了，也就是执行了。

### chain 方法的本质

与 `map` 方法不同， `chain` 方法传入的函数在执行后会返回一个函子，合并后的函数最终会响应一个函子，函子一般是不作为参数传入到下一个纯函数中的，所以 `chain` 函数中多了一个组合纯函数 `join` 步骤，即增加一个打开这个函子的步骤。

## 参考资料

1. [什么是 Monad (Functional Programming)？](https://www.zhihu.com/question/19635359/answer/420267395)
2. [对普通程序员来说，Monad有什么用？](https://zhuanlan.zhihu.com/p/575642401)
3. [函数式夜点心：IO Monad 与副作用处理](https://juejin.cn/post/6844904082390384653)
