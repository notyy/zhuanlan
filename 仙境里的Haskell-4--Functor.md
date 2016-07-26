## 仙境里的Haskell（之四）—— Functor类型类

昨天又有Scala新手在问关于类型参数的问题，Haskell也一样有不少人一时转不管那个弯来。 因为类型参数非常重要，所以在今天的内容前再稍微做一些解释。

所谓类型参数，就是说有个参数，它的类型，也是个参数。。。。
```Haskell
myfst :: (Int, Char) -> Int
myfst (x,y) = x

----run-----
myfst (1,'c')
1 ---Int
```
上面这个myfst函数，能够取出(Int,Char)二元组的第一个元素。其实从我们的定义来看，函数实现并不依赖于第一个元素的具体类型，只要是第一个元素，就取出来便是。所以我这个函数应该可以用于(Int,Char),(Char,Int),(String,Int)等，只要是二元组即可。但是同时我又不想丢失它的实际类型，也就是说我希望后续的代码能够知道myfst (1,'c')的结果是Int类型，myfst('c',1)的结果是Char类型。

所以我们可以用类型参数代替实际类型，要求用户给出这个类型。
```Haskell
myfst :: (a, b) -> a
myfst (x,y) = x

----run----
myfst ('c',1)
'c'    ---- Char
myfst (1,'c')
1 ---Int
```
在编译过程中，编译器推断出'c'的类型是Char，也就是说a在这里的具体类型是Char，因此函数的返回结果也必须是Char类型。其实完整的写法应该是
```Haskell
myfst ('c' :: Char,1)
```
但因为Haskell有自动类型推断能力，这里就省略了类型声明。


在上一篇里，我们用代数数据类型定义了一个二叉树(这个定义有点缺陷，但是比较方便，在下文中暂时沿用)：
```Haskell
data MyTree a = Leaf a
            | Node a (MyTree a) (MyTree a)
            deriving (Show)
```
并且利用递归和模式匹配很方便的定义了获取最左侧叶子节点数据的函数：
```Haskell
leftMost :: MyTree a -> a
leftMost (Leaf x) = x
leftMost (Node _ l r) = leftMost l
```
那么假如我们要对MyTree a的每个节点的数据做一个运算，得到另一个MyTree，要怎么写呢？也就是：
```Haskell
myMap :: (a -> b) -> MyTree a -> MyTree b
```
用Haskell写代码的第一件事就是把要设计的函数的类型签名写下来，很多时候签名写下来，实现的思路也就很清楚了。
```Haskell
myMap :: (a -> b) -> MyTree a -> MyTree b
myMap f (Leaf x) = Leaf (f x)
myMap f (Node x l r) = Node (f x) (myMap f l) (myMap f r)
```
可以试着运行一下：
```
myMap (+1) (Leaf 2)
Leaf 3
it :: Num b => MyTree b

myMap (+2) (Node 1 (Leaf 2) (Leaf 3))
Node 3 (Leaf 4) (Leaf 5)
it :: Num b => MyTree b
```
具有MyTree a这种性质的数据类型很多，比如
```Haskell
data Option a = None
              | Some a
              deriving (Show)

myMap :: (a -> b) -> Option a -> Option b
myMap f None = None
myMap f (Some x) = Some (f x)
```
用用看：
```Haskell
myMap (+1) (Some 1)
Some 2
it :: Num b => Option b

myMap (*2) None
None
it :: Num b => Option b
```
Option在标准库里的原型叫Maybe,是一种非常有用的类型，比如我们有个根据email查找User的函数，如果数据库里并不一定存在给定email所对应的用户，那么我们可以把函数的返回类型定义为Maybe User
```Haskell
findUser :: Email -> Maybe User
```
再比如最常用的列表类型，[1,2,3]，我们希望对里面的数据做一个运算得到一个新列表
```Haskell
map :: (a -> b) -> [a] -> [b]

Prelude> map (*2) [1,2,3]
[2,4,6]
```
Haskell是一种能通过类型表达丰富业务含义的强类型编程语言
```Haskell
f1 :: Int -> Int -- 这个运算一定会返回**一个**Int型的结果
f2 :: Int -> Maybe Int -- 这个运算**可能**会返回一个Int或者None
f3 :: Int -> [Int]     -- 这个运算会返回**不确定个数**个Int
```
Maybe代表可能。<br/>
[]代表不确定个数。<br/>
对Maybe和[]列表类型来说，他们都支持map运算，这种运算会对它**里面的值**做一个运算，结果的性质不变，Maybe仍然是Maybe，[]仍然是[]。<br/>
（注意我虽然用**里面的值**来表述，但是不要简单的理解为容器类型，因为容器太狭隘，你很快就会看到不是容器，但也具有相同类型的类型。所以我一般用**上下文**来表达。）

显然有**一类**类型都具有相同的性质，即可以对其所代表的上下文里的数据做个运算，结果仍然处于相同的上下文。如果我们对这类类型做个提炼和抽象，以后看到这种类型就知道怎么使用了。 前面代码里出现的三种ADT具有形似的类型定义：
```Haskell
MyTree a
Option a
[] a  -- [a]只是语法糖
```
我们可以提炼出类型类Functor:
```Haskell
class MyFunctor f where
  fmap :: (a -> b) -> f a -> f b
```
然后我们可以在各自的数据类型里去实现它：
```Haskell
data Option a = None
              | Some a
              deriving (Show)

myMap :: (a -> b) -> Option a -> Option b
myMap f None = None
myMap f (Some x) = Some (f x)

instance MyFunctor Option where
  fmap = myMap
```
相应的，也可以为MyTree提供MyFunctor类型实现
```Haskell
instance MyFunctor MyTree where
  fmap = myMap
```
MyFunctor在标准库的原型就叫Functor，如果看到一个类型是Functor的实例，我们就知道可以对它使用fmap函数。比如说：
```Haskell
Prelude> fmap (*2) (Just 2)
Just 4

Prelude> fmap (*2) [1,2,3]
[2,4,6]

Prelude> let f = fmap (*2) (+1)
Prelude> f 2
6
```
且慢！最后一个是什么鬼呢？
