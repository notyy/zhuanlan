## 仙境里的Haskell（之五）—— Monad类型类

在[前一篇里](https://zhuanlan.zhihu.com/p/21551174?refer=damotou)，我们讲到有很多东西是Functor，可以应用fmap函数对其“里面”的内容做转换。
```Haskell
class Functor f where
  fmap :: (a -> b) -> f a -> f b
```
那么怎么理解这个例子呢？
```Haskell
Prelude> let f = fmap (*2) (+1)
Prelude> f 2
6
```
把函数a -> b 看做 (a ->) b，就像data Option b, data MyTree b一样， 我们可以把(a ->)看做data (a ->) b，(a ->) b 就构造了一个函数，函数也是数据。现在我们对(a ->) b这个***数据***fmap一个b -> c的函数，结果的**上下文**（我在前一篇里用了这个词）应该不变，如果原来是Option b，那么结果应该是Option c，仍然是Option。 同样的，原来是(a ->) b，结果仍然必须是(a ->)，只不过里面的值b变成了c，因此结果是a -> c。
```Haskell
-- 如果语法允许，可以写成这样
fmap :: (b -> c) -> (a ->) b -> (a ->) c
-- 然而语法并不支持
fmap :: (b -> c) -> (a -> b) -> (a -> c)
```
怎么实现呢？其实这种全是a、b、c的函数特别不容易出错，因为基本上只有一种能编译通过的解法。我们可以看到函数的返回值也是函数，所以用定义lambda函数的语法来定义=右边的函数体。
```Haskell
fmap :: (b -> c) -> (a -> b) -> (a -> c)
fmap f g = \x -> f(g x)
```
这是定义返回函数的函数的常用技巧，这样的函数有个术语叫做combinator。 返回的函数类型是a -> c，所以实现里面的x一定是a类型，在入参里只有函数g:: a -> b能够应用到a类型的x上，得到返回值b，又只有f: b -> c能够应用到b上，返回c，所以这是唯一一种能够编译通过的写法。 写的好的Haskell库往往具有这样的特点——能编译通过程序就是正确的。

从类型签名上就可以看出来fmap用在函数上的效果是**函数组合**。 在标准库里的实现叫做(.)
```Haskell
(.)    :: (b -> c) -> (a -> b) -> a -> c
(.) f g = \x -> f (g x)
```
而且它是中置的，所以：
```Haskell
Prelude> let f = (*2) . (+1)
Prelude> f 3
8
```

总结一下之前所学：<br/>
函数式编程的要素就是**函数**和**数据**。 <br/>

对于任意数据a，我们可以有函数a->b把它转换成结果数据b，然后继续用b -> c将它转换成c，以此类推。 函数式程序就是一系列的数据转换。

对于函数a -> b，我们可以把函数b -> c跟它组合，得到函数a -> c，然后可以接续组合c -> d，以此类推。

对于处于某种上下文里的数据F a, 我们可以fmap (a -> b) -> F a -> F b，将里面的数据转换掉而不脱离上下文。 然后可以继续fmap (b -> c),以此类推。 <br/>
这么做有什么好处呢？下面举个稍接地气的例子：
```Haskell
-- type定义类型别名，用来使后面的函数较有真实感
type Connection = String
type UrlStr = String
type User = String
type Email = String

getConn :: UrlStr -> Maybe Connection
getConn "damotou" = Just "good connection"
getConn _ = Nothing

getUser :: Connection -> User
getUser "good connection" = "good user"

getEmail :: User -> Email
getEmail "good user" = "good email"
```
已知Maybe是个Functor。 已经有个程序员写了上面三个基本函数，现在有另外一个程序员要调用这些函数组装一个业务：
```Haskell
fetchEmail :: UrlStr -> Maybe Email
```
因为获取数据库连接可能失败，所以当然最后的获取Email也可能失败，因此最后的类型是Maybe Email，很明智的设计。一种实现方式是用模式匹配把获取连接的结果分解出来分别处理：
```Haskell
fetchEmail :: UrlStr -> Maybe Email
fetchEmail urlStr = case getConn urlStr of
                      Just conn -> Just ((getEmail . getUser) conn)
                      Nothing -> Nothing
```
利用fmap，可以不把数据取出来，直接把函数塞进去，结果仍然是Maybe：
```Haskell
fetchEmail' :: UrlStr -> Maybe Email
fetchEmail' urlStr = fmap (getEmail . getUser) (getConn urlStr)
```
两边约掉个参数，可以改写成
```Haskell
fetchEmail'' :: UrlStr -> Maybe Email
fetchEmail'' = fmap (getEmail . getUser) . getConn
```
约掉参数是个玩笑话，这写法是什么意思大家可以用心体会下。

就这个例子来说，两种写法差别也不是特别大。 但假如说后面的步骤，getUser, getEmail也都有失败的可能，比如找不到用户也很正常啊，有用户没填email也很正常啊。 用类型来表示就是：
```Haskell
getConn :: UrlStr -> Maybe Connection
getConn "damotou" = Just "good connection"
getConn _ = Nothing

getUser :: Connection -> Maybe User
getUser "good connection" = Just "good user"
getUser _ = Nothing

getEmail :: User -> Maybe Email
getEmail "good user" = Just "good email"
getEmail _ = Nothing
```
这时用第一种方法来写就会非常冗长：
```Haskell
fetchEmail :: UrlStr -> Maybe Email
fetchEmail urlStr = case getConn urlStr of
                      Nothing -> Nothing
                      Just conn -> case getUser conn of
                                  Nothing -> Nothing
                                  Just user -> getEmail user
```
如果getEmail后面还有更多类似的逻辑，就会一直叠加下去。<br/>
而fmap也无法应用在这种场景下，因为fmap (a -> b) -> Maybe a -> Maybe b 要求函数是a -> b，而不是a -> Option b。为此我们需要定义新的函数：
```Haskell
bind :: (a -> Maybe b) -> Maybe a -> Maybe b
bind f Nothing = Nothing
bind f (Just v) = f v
```
然后就可以把这三个函数串起来。
```Haskell
fetchEmail' :: UrlStr -> Maybe Email
fetchEmail' urlStr = bind getEmail (bind getUser (getConn urlStr))
```
如果大家仔细分析fetchEmail和bind的函数，就会发现这个函数其实是把一个重复的逻辑给提炼了出来，即 Nothing -> Nothing。

当然在标准库里同样定义了类似bind的实现，叫做(>>=)
```Haskell
(>>=) :: forall a b. m a -> (a -> m b) -> m b  -- forall语法暂时忽略
```
和bind的区别是把ma放在函数前面，并且是中缀运算符。
```Haskell
fetchEmail'' :: UrlStr -> Maybe Email
fetchEmail'' urlStr = (getConn urlStr) >>= getUser >>= getEmail
```
具有>>=操作的数据类型就叫做**Monad**
