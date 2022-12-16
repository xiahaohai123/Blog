# xml 文件中的 xmlns 属性解释

## 前言

在 `MDN` 上学习 `svg` 的时候碰到了个一直困惑了我很久但我一直没有去找到答案的问题。

- xmlns 这个属性到底是干嘛的？
- 这个标签要连接到什么地址上么，难道要连接互联网？

刚好 `MDN` 的 `svg` 部分有篇文章解答了我的问题，在这里记录一下。

答案:

- xmlns 会标记当前标签以及子标签所属的命名空间。标记命名空间的原因是现今 xml 存在大量**方言**，不同的方言之间可能定义了同一种标签，使用命名空间是为了方便区分，以让程序或用户作出对应处理。
- xmlns 的属性值仅仅是一个字符串，并不会真的要连接到啥地方去，而看起来是 `uri` 的原因是这么做不会重复。

## 背景

W3C 的长期目标是使不同类型的 XML 基本内容可以混合在同一个 XML 文件中。例如，SVG 和 MathML 可以直接并入基于 XHTML 的科学文档中。能够混合这样的内容类型有很多优点，但也需要解决一个非常实际的问题。

合理来说，每个 XML 语言定义其规范中描述的标记标签名称的含义。在单个 XML 文档中混合来自不同 XML 方言的内容的问题是，由一个方言定义的标签可能与另一个方言定义的标签具有相同的名称。例如，XHTML 和 SVG 都有一个<title>标签。专业使用者（例如程序）如何区分两者？

问题的真正答案是：XML 内容通过给明确的属性表示这个标签属于哪个**命名空间**来告诉使用者这段表现属于哪个 XML 方言。

## 声明命名空间

所以这些命名空间声明是什么样的呢，并且在什么地方用他们，如下的例子所示：

```xml

<svg xmlns="http://www.w3.org/2000/svg">
    <!-- more tags here -->
</svg>
```

命名空间声明是通过 `xmlns` `attribute` 提供的。xmlns 属性意味着这个 `<svg>` 标签和它的子节点都属于 `http://www\.w3.org/2000/svg` 这个 SVG 命名空间。

注意，这个命名空间声明只需要在根节点上声明一次。这个声明定义了默认的命名空间，所以用户代理（例如程序）知道所有的 `<svg>` 标签的子标签也属于相同命名空间。这样用户代理可以通过识别这个命名空间来决定他们如何处理这个标记。

- 请注意，命名空间仅仅只是一些字符串，所以 SVG 上那些看起来像 URI 的命名空间并不重要。因为 URIs 的唯一性从而被广泛使用，它的本意并不是要“链接”到某个地址。

所以现在我们再看看属性名 `xmlns`，是不是一下子就能明白这个单词指的是什么了？这样我们就能轻易记住这个属性名了。以前笔者根本没办法记住这个属性名。

- xmlns: **xml name space**

## 重新定义默认命名空间

如果根节点的所有子节点也被定义为默认命名空间，那么你如何混合使用另一种命名空间呢？很简单，你仅仅只需重新定义默认命名空间即可。这里是一个简单的例子。

```xml

<html xmlns="http://www.w3.org/1999/xhtml">
    <body>
        <!-- 在这里放置一些 XHTML 标签 -->
        <svg xmlns="http://www.w3.org/2000/svg" width="300px" height="200px">
            <!-- 在这里放置一些 SVG 标签 -->
        </svg>
        <!-- 在这里放置一些 XHTML 标签 -->
    </body>
</html>
```

在这个例子中根节点 `<html>` 的 `xmlns` 属性定义了 XHTML 为默认命名空间。结果就是它和它所有的子节标签，除了 `<svg>` 标签，都被用户代理解释为属于 XHTML 命名空间。 `<svg>` 标签拥有它自己的xmlns属性，通过重新定义默认的命名空间，告诉用户代理 `<svg>` 标签以及他所包含的标签（除非他们也重新定义了默认名称空间）属于 SVG.

## 参考文献

1. [Namespaces Crash Course](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Namespaces_Crash_Course)