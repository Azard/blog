---
title: Scala的抽象语法树打印小工具-小拉达
date: 2015-12-02 01:39:00
categories: Scala
tags: Scala
updated:
comment: true
banner:
thumbnail: https://raw.githubusercontent.com/Azard/Rattata/master/Rattata.png
---
最近做的两个项目，一个是[VeriScala](https://github.com/VeriScala/VeriScala)，另一个是[Lickitung](https://github.com/Azard/Lickitung)，都涉及到了Scala的抽象语法树(AST)，前者是写macro的需要，后者是做AST的pattern match。

但是在网上竟没有发现一个很好的格式化打印AST的工具。唯一找到的是[ScalaAstPrinter](https://github.com/jugyo/ScalaAstPrinter)，然而用法和输出都不太符合我的期望，不知道是这个需求太小还是我走错方向了。于是自己写了一个。因为只有几十行代码并且是个很小的工具，于是取名叫**Rattata**，口袋妖怪中的小拉达。

项目地址：[https://github.com/Azard/Rattata](https://github.com/Azard/Rattata)

![](https://raw.githubusercontent.com/Azard/Rattata/master/Rattata.png)

<!-- more -->

大三的时候编译原理大作业也做过一个树状的AST输出，当时前端显示的部分是_guoyanchang_写的，他现在在阿里云搬砖。

当时_guoyanchang_实现的树状打印输入是一个我传给他的Java实现的多叉树的数据结构，现在的情况是Scala的抽象语法树经过`showRaw`处理后的一个字符串。

```Scala
val exp = reify {
  val x = 1
  val y = 2
  x + y
}
```

```Scala
scala> println(showRaw(exp))
Expr(Block(List(ValDef(Modifiers(), TermName("x"), TypeTree(), Literal(Constant(1))), ValDef(Modifiers(), TermName("y"), TypeTree(), Literal(Constant(2)))), Apply(Select(Ident(TermName("x")), TermName("$plus")), List(Ident(TermName("y"))))))
```

我最开始的想法是转成多叉树结构，再想办法打印，但觉得这样似乎小题大做而且不够优雅。

第二个想法是将每个token先提取出来，存在一个Array中，然后再读一遍这个字符串根据`(`和`)`的顺序判断入栈出栈，然后依次用不同的缩进打印token。这样的实现首先对字符串做了多次`replace`和`split`然后得到了一个token的Array，还用`map`什么的去除了`split`后的空格，然后再读取一遍得到入栈出栈顺序，感觉上又做了多余的事。

然后这个时候想到似乎可以读取一遍字符串，读到`(`就入栈，读到`)`就出栈，读到`,`只换行。然后就得到了如下代码：

```Scala
def pprintAST(input: Expr[Any]) = {
  var level = 0
  showRaw(input).foreach {
    case '(' =>
      level += 1
      println()
      if (showLine) {
        print(("|" + " "*(tabSize-1)) * (level-1))
        print("|" + "-"*(tabSize-1))
      } else {
        print(" " * tabSize * level)
      }
    case ')' =>
      level -= 1
    case ',' =>
      println()
      if (showLine) {
        print(("|" + " "*(tabSize-1)) * (level-1))
        print("|" + "-"*(tabSize-1))
      } else {
        print(" " * tabSize * level)
      }
    case ' ' =>
    case f => print(f)
  }
}
```

当然最开始的实现不包括`showLine`和`tabSize`相关的东西，调用`Rattata.pprintAST(exp)`得到了如下的输出：

```
Expr
|---Block
|   |---List
|   |   |---ValDef
|   |   |   |---Modifiers
|   |   |   |   |---
|   |   |   |---TermName
|   |   |   |   |---"x"
|   |   |   |---TypeTree
|   |   |   |   |---
|   |   |   |---Literal
|   |   |   |   |---Constant
|   |   |   |   |   |---1
|   |   |---ValDef
|   |   |   |---Modifiers
|   |   |   |   |---
|   |   |   |---TermName
|   |   |   |   |---"y"
|   |   |   |---TypeTree
|   |   |   |   |---
|   |   |   |---Literal
|   |   |   |   |---Constant
|   |   |   |   |   |---2
|   |---Apply
|   |   |---Select
|   |   |   |---Ident
|   |   |   |   |---TermName
|   |   |   |   |   |---"x"
|   |   |   |---TermName
|   |   |   |   |---"$plus"
|   |   |---List
|   |   |   |---Ident
|   |   |   |   |---TermName
|   |   |   |   |   |---"y"
```

然而和我期望的实现还是有点区别，这里有些多余的线，比如`Expr`下的直线只需要到`Block`为止。

加入`Expr`作为0级，`Block`作为1级，这里的主要问题是在边读边输出的时候不知道后面是否还要某一级的token，如果我输出完`Block`知道了后面的字符串没有第1级的token，我就可以不打印`Expr`下的直线。

于是我又想到了新的实现，先读第一遍根据`(`和`)`统计各个级的token数量，然后读第二遍再边读边输出，当输出完第n级的一个token时在第n级的总token数上减1，这样就可以去掉所有多余的线。

仔细想一想，似乎是没有办法做到只读一遍就不打印多余的线的，因为这需要知道整个抽象语法树的状态，必须先读一遍存好状态，第二遍根据状态输出。

然而这个多余的输出似乎更好看点，因为可以直接看到后面的token是第几级的，看起来更直观。我问了问_tcbbd_，他也觉得保留多余的线比较好，于是这个**Rattata**就阴差阳错的用上面那种方式打印Scala AST。

最终的实现`case '('`和`case ','`有7行重复的代码，可以用一个函数复用，但前几天看了王垠的[《编程的智慧》](http://www.jianshu.com/p/7645a5ea7f46)，感觉这个代码复用一个函数不太直观，有点操作过度，于是就保留上面这样了。
