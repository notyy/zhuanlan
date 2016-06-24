## 仙境里的Haskell（之二）

Haskell是一种惰性求值(lazy evaluation)的、有自动类型推断能力的、纯（pure）函数式编程语言。

Haskell是一门很容易学的编程语言（相信我），就像数学一样(｀﹏′)。核心语法非常少，那些高深东西都可以从我今天要讲的基本的东西里自行推导出来。 有没有见过数学老师讲课的时候忘了一个定理，然后在黑板上直接推导出来？ 就这么简单 ╮(╯▽╰)╭。

[上一篇](https://zhuanlan.zhihu.com/p/21371274?refer=damotou)里，我们准备了学习环境，其实也就是ghci repl和一个基本的编辑器就够用了。

打开ghci
```Haskell
Prelude> 3 * 4
12
```
多好的计算器~

在ghci里设置一下显示类型信息：
```Haskell
Prelude> :set +t
Prelude> 3 * 4
12
it :: Num a => a
```
设置了 +t后，执行结果后面会跟一行显示结果的类型,每次退出ghci后要重新设置。Haskell里声明变量及其类型的语法就如这个例子里的 it :: Num a => a , it是变量名，=>箭头前面的Num a是个类型约束，表示编译器现在不确定it是个什么具体类型，可能是整数，也可能是浮点数等等，只知道它是Num类型类（typeclass）的实例，因为它能做乘法运算。就目前的使用场景来说，知道它是Num也就够了。类型约束不是必须的。比如
```Haskell
Prelude> 'c'
'c'
it :: Char
```
这里编译器明确的确定了'c'的类型是Char。
顺便说一句，这个it变量是实际可用的
```Haskell
Prelude> it
'c'
it :: Char
```
字符串的类型实际上就是字符列表
```Haskell
Prelude> "damotou"
"damotou"
it :: [Char]
```
你可以用字符构造一个列表，结果是一样的
```
Prelude> ['d','m','o','t','o','u']
"dmotou"
it :: [Char]
```
列表的所有元素必须是相同类型的。
```Haskell
Prelude> [1,'a']

<interactive>:15:2:
    No instance for (Num Char) arising from the literal ‘1’
    In the expression: 1
    In the expression: [1, 'a']
    In an equation for ‘it’: it = [1, 'a']
```
编译器根据第一个元素，1，推断这个列表应该是(Num a)类型的，但是'a'是Char类型，而Char类型不是Num类型的实例，所以编译不过。

我这么啰嗦的解释这个编译错误，是因为用Haskell写程序大部分时间都在跟编译错误做斗争，当程序编译过了的时候基本上功能就是好的了。所以需要细心的体味编译器给出的***贴心***的编译错误~~~

Haskell支持元组（tuple）类型
```
Prelude> (1,'a')
(1,'a')
it :: Num t => (t, Char)
Prelude> ("Haskell",'H',1)
("Haskell",'H',1)
it :: Num t => ([Char], Char, t)
```
元组的类型就是其每个成员的类型。当然你可以把元组放到列表里
```
Prelude> [(1,'a'), (2,'b'), (3,'c')]
[(1,'a'),(2,'b'),(3,'c')]
it :: Num t => [(t, Char)]
```
你就得到了这个元组类型的列表。

有以上这些基本类型基本上就够我们在仙境里的使用了，哦忘了还有个布尔型：
```
Prelude> True
True
it :: Bool
Prelude> False
False
it :: Bool
Prelude> not True
False
it :: Bool
Prelude> True && False
False
it :: Bool
Prelude> False || True
True
it :: Bool
```
和大多数语言都是一致的，就不多解释了。 这下齐活了。

以上都是在程序员不主动写明类型的情况下，编译器自动推断的结果。 实际上我们是可以主动写明类型的：
![Haskell类型声明](haskellfariyland.png)

ghci不支持这样的语法，所以你需要建个文件，叫做haskellfariyland.hs，然后在编辑器里输入这些内容。第一行的模块声明在仙境里不重要，所以不解释了，和文件名一致即可，照抄一下吧。

如果你和我一样用的是Haskellformac，那么在playground里输入x，就可以最右侧窗口看到x的执行结果，和在ghci里执行效果是一样的。你可以用你喜欢的编辑器，比如Atom写，然后在ghci里:l 文件名， 加载完成后，输入x回车，就能看到一样的结果了。

函数式编程，当然是以函数为核心的，所以我们要来学习**add函数的4种写法**,
** 第1种：**
```Haskell
add :: (Int,Int) -> Int
add (x,y) = x + y
```
虽然Haskell具有很强的类型推断能力（也许是现有编程语言中最强的），但是习惯上一个模块的顶层函数定义和值定义都会声明类型，方便阅读代码和生成文档，而且先写函数的类型签名再写实现也是驱动我们思考程序设计的非常好的方法，这种开发方式被称为类型驱动开发（Type Driven Development —— 简称**TDD**）  (｀﹏′)

函数式编程里的函数，就如同数学意义上的函数 y = f(x),函数就是把**一个**输入值转化成输出值的运算。
```
Prelude> add (1,2)
3
it :: Num a => a
```
这种写法的入参是一个二元组，(Int,Int)共同构成了一个值。返回值是Int，所以这个函数的类型就如写下来的那样：(Int,Int) -> Int

** 第二种写法：**
```
add' :: Int -> (Int -> Int)
add' x y = x + y
```
有了前面那个例子，现在这个函数签名应该也很容易理解吧？入参是一个Int,返回值是一个Int -> Int的函数。 返回的这个函数可以再接受第二个参数，最后返回一个Int结果。 所以我们可以这么调用它：
```
Prelude> add' 1 2
3
it :: Num a => a
```
可能有聪明的同学已经会想到，如果我只传一个参数会怎样？当然是返回给你一个函数咯，你还可以给返回的函数一个名字，比如：
```
Prelude> let add1 = add' 1
add1 :: Num a => a -> a
Prelude> add1 2
3
it :: Num a => a
```
***（再提醒一下，在ghci里定义一个名字需要用let关键字，在编辑器里不用。）***
这种行为称为**柯里化**， 柯里化是非常有用的特性，我们以后会看到更多使用场景。返回的函数叫做部分应用函数(partially applied functions)

**第三种写法**就是著名的lambda写法
```
add'' :: Int -> (Int -> Int)
add'' = \x y -> x + y
```
用法和add'一样，实际上他们应该是编译成一样的结果的。 lambda写法可以用来写匿名函数，比如说这个函数我就想用一次，不想给个名字，那么可以直接用lambda定义和使用：
```Haskell
Prelude> (\x y -> x + y) 1 2
3
it :: Num a => a
```
其实lambda更主要的作用是写一个函数作为参数传给另一个函数，后面讲高阶函数的时候大家就会知道。

**关于第四种写法。。。**
现在再回看一下第二种写法：
```
add' :: Int -> (Int -> Int)
add' x y = x + y
```
入参（形参x y）和一个Int对不上号~~~
所以在学习了lambda写法后，我们可以写个对的上号的：
```Haskell
add''' :: Int -> (Int -> Int)
add''' x = \y -> x + y

Prelude> add''' 1 2    --加载后在ghci里运行
3
it :: Int
```

以上总结了add函数的各种写法，有些很文艺，有些很二X，普通青年都这么写:
```Haskell
------------参数1---参数2---返回值
normalAdd :: Int -> Int -> Int
normalAdd x y = x + y
```
但是你要理解这是柯里化的效果。

我们的add函数有点挫，只能加整数，normalAdd 1.0 2.0就会出错，我们修改一下函数签名，让它能加天下可加之物。
```Haskell
add :: Num a => a -> a -> a
add x y = x + y
```
前面已经解释过Num a是类型约束，小写的字母a是类型参数，和函数参数一样，类型参数的名字也是你任意起的，叫b、c、d,x、y都无所谓，只是必须小写字母。大写字母开头的代表具体类型，像Int就是个具体类型。 不要把a理解为Any，也不要完全按照Java的接口来理解，以为a被变成了Num。 实际上a捕捉了你调用函数式实际给出的真实类型，并且做出了强类型的严格限定。a -> a -> a这个简单的签名表明第二个参数的类型必须和第一个参数相同，结果类型也一样。
```Haskell
Prelude> let x = 1 :: Int
x :: Int
Prelude> let y = 2 :: Int
y :: Int
Prelude> add x y
3
it :: Int

Prelude> let x = 1.0 :: Float
x :: Float
Prelude> let y = 2.0 :: Float
y :: Float
Prelude> add x y
3.0
it :: Float

Prelude> let x = 1 :: Int
x :: Int
Prelude> let y = 2.0 :: Float
y :: Float
Prelude> add x y

<interactive>:18:7:
    Couldn't match expected type ‘Int’ with actual type ‘Float’
    In the second argument of ‘add’, namely ‘y’
    In the expression: add x y
```
以上代码建议大家敲敲试试，并确保完全理解。

然后再介绍add的**第5种写法**~~~
Haskell是函数式编程语言，Haskell的世界里自然到处都是函数。比如 **+**
```Haskell
Prelude> :t (+)
(+) :: Num a => a -> a -> a
```
咦，这签名和我们的add一模一样啊？ 哈哈，你有没有发现我们的add实际上不就是调用+吗？只不过有一些规则让+可以作为中置运算符(infix)来使用.

既然我们的add函数就是系统提供的+函数，所以最简单的实现当然就是
```Haskell
Prelude> let add = (+)
add :: Num a => a -> a -> a
Prelude> add 1 2
3
it :: Num a => a
```

最后留一个彩蛋，我们的add函数也可以中置使用哦：
```
Prelude> 1 `add` 2
3
it :: Num a => a
```
