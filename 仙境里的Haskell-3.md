## 仙境里的Haskell（之三）

在[上一篇](https://zhuanlan.zhihu.com/p/21404404?refer=damotou)里，我们介绍了定义函数的四种，额，五种方法。 所谓函数，就是把一种类型的数据转换成另一种类型（也可以是同一种类型）的数据的运算。所以Haskell里的函数签名都是非常直观的体现了这种含义，比如Int -> Int, Int -> String等。 除了这种基本类型外，我们当然也可以定义自己的数据类型，以及在它们之间转换的函数。实际上Haskell程序的主要内容就是这些东西。

自定义数据类型的方法有n种，在仙境里就介绍一种大概就够了，也就是高大上的代数数据类型（Algebra Data Type，简称ADT）。

假设我们的系统允许用户用电话号码、Email、用户名三种方式注册，现在要写一个validate函数检查输入的数据是否合法，我们可以这样定义我们的数据类型：
```Haskell
data Credential = Mobile String
                | Email String
                | UserName String
```
然后用模式匹配(pattern matching)来对Credential类型的入参做处理：
```Haskell
validate :: Credential -> Bool
validate (Mobile num) = (length num) == 11
validate (Email address) = elem '@' address -- 判断address里是否有'@'
validate (UserName name) =
  let
    nameLength = length name
  in
    nameLength > 5 && nameLength <= 10
```
在上例中，Credential是类型名，Mobile是类型构造器，也就是个用来构造类型的函数，实际上如果你查看Mobile的类型的话，它的类型是String -> Credential。<br/>
模式匹配能够匹配构造器，然后获取其对应位置上的数据。<br/>
比如给定元组(1,"abc")，想分别取前后两个值出来，我们可以定义以下函数：
```Haskell
first :: (Int,String) -> Int
first (i, _) = i

second :: (Int, String) -> String
second (_, s) = s
```
等一下，不是应该匹配类型构造器吗？ 没错。()就是元组的类型构造器，类型名也是()，就是这么任性~
```
data ()

The unit datatype () has one non-undefined member, the nullary constructor ().

Constructors
()	 
```
还有二元组，三元组、四元组、以至于n元组：
```
data (,) a b
Constructors
(,) a b	 

data (,,) a b c
Constructors
(,,) a b c	 

data (,,,) a b c d 
Constructors
(,,,) a b c d
```
所以，实际上只是ADT加上一点点语法糖。

元组的定义使用类型参数a,b等等，这样就可以定义任何种类的元组，在实际数据给定时才确定元组类型。我们的first和second当然也应该使用类型参数，以便能够用于任何类型的二元组：
```Haskell
first :: (a,b) -> a
first (i, _) = i

second :: (a, b) -> b
second (_, s) = s
```
把函数签名改一改就好了，实现代码都不用改。

列表[]也是一样，只是类型定义中使用了递归定义。但是我不打算**解释**递归定义了，我们亲手来**做**一个简单的吧——二叉树！
```Haskell
data Tree a = Leaf a
            | Node a (Tree a) (Tree a)
            deriving (Show)
```
***（最后一行deriving (Show)是为了让我们构造出来的值能够以字符串形式呈现，具体细节在介绍类型类的时候再说。） *** <br/>

一个二叉树，要么是一个叶子节点，里面放了未知的a类型的值，要么是个树干节点，除了放a类型的值，还放了下级的两个**子树**， 子树究竟是叶子还是另一个树干呢？不知道。都可以。

定义好Tree类型后，我们就可以构造它的值了：
```
Leaf '1'
it :: Tree Char

Node '1' (Leaf '2') (Leaf '3')
it :: Tree Char

Node 1 (Node 2 (Leaf 3) (Leaf 4)) (Leaf 5)
it :: Num a => Tree a
```
如何获取里面的值呢？ 比如，取树的做左边叶子的值：
```Haskell
leftMost :: Tree a -> a
```
作为作业吧~~
