---
title: Github搜索技巧
tags: Github
category: Github
abbrlink: 6321
date: 2018-03-16 16:44:36
---
![image](http://ovi3ob9p4.bkt.clouddn.com/TIETU/CT0154.jpg)

 github搜索技巧
<!--more-->

### 1. Search

- 如何查看一门语言的 Repository 排行榜（按 stars 数量排）？
  如图所示，以 Objective-C 为例，直接在输入框中输入  language:Objective-C stars:>0， 然后再在右侧排名选项中选择 Most stars。

按 stars 数量排名(以 Objective-C 为例).png

- 为什么有些数据模糊搜索不到？
  比如，输入搜索关键字 “collectionView”，然后在左侧边栏 Languages 中选择 Objective-C ，发现搜索结果中没有 “PSTCollectionView” 这个Repository，实际上，如果搜索的是 “PSTCollectionView” 的话，确实是能搜索到的。
  从搜索结果中来看，“collectionView” 是被作为一个单词整体来进行搜索的，所以搜到的结果都是 Repository name 或者 description 中出现以 “collectionView” 开头或者包含 “-collectionView” 的单词的 Repository。
  所以为了能搜索到更多想要的结果，我们最好以单词为单位，用 OR 将各个关键字拼接起来进行搜索，例如，搜 “CollectionView OR UICollectionView OR collection” 而不是 “collectionView”。
  下面是两种搜索词的结果对比。


- Github 有高级搜索吗？
  在上图中，我们可以看到左侧边栏的下方有两个可点击的选项 [Advanced search](https://link.jianshu.com?t=https://github.com/search/advanced) 和 [Cheat sheet](https://link.jianshu.com?t=https://github.com/search?utf8=%E2%9C%93&q=&type=Repositories&ref=advsearch&l=&l=)，点击 [Advanced search](https://link.jianshu.com?t=https://github.com/search/advanced)  即可进行自定义条件的高级搜索了，点击 [Cheat sheet](https://link.jianshu.com?t=https://github.com/search?utf8=%E2%9C%93&q=&type=Repositories&ref=advsearch&l=&l=) 则可以查看一些有关搜索的帮助信息。哪里不会点哪里，妈妈再也不用担心我的学习了！

### 2.Trending

作为一枚程序猿，除了有目的的搜索之外，我们有时也需要去“瞎逛逛”，开阔一下眼界。如果你有空，不妨去 Github 的 [Trending](https://link.jianshu.com?t=https://github.com/trending) 看看最近发生了什么。*See what the GitHub community is most excited about today! * 在这里你可以看到各种不同开发语言的每天/周/月的最热门的 Repositories 和 Developers。比如前一段时间走红的 [YYKit](https://link.jianshu.com?t=https://github.com/ibireme/YYKit)，苹果最近开源的 [CareKit](https://link.jianshu.com?t=https://github.com/carekit-apple/CareKit)，等等。

### 3.制定搜索方式：

我默认检索的关键词都是android

按照文件搜索

```
android in:file
```

按照路径检索

```
andrioid in:path
```

按照语言检索

```
android language:java
```

按照文件大小

```
android size:>100
```

按照后缀名检索

```
android extention:css
```

按照是否被fork过

```
android fork:true
```

按照地域检索（这个猎头和hr应该用得着）

```
android location:beijing
```
