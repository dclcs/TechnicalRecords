## Swift 中的设计模式： Delegate

### 什么是委托模式

**to give a particular job, duty, right, etc. to someone else so that they do it for you**

简单来讲委托模式（delegate）就是委托其他人“帮你做事”。

比如下面这个例子，你想点一个pizza，可以自己吃，也可以点给你的朋友Mary。那么如何将其转换成代码呢？

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gufzkbl3m9j60my0bdmxm02.jpg)

如果Robot想买Pizza，那么这里需要一个pizza maker

```swift
class PizzaMaker() {
  func makePizza() {}
}
```

但是，pizza maker 需要有个人去送pizza，那这里就需要委托一个人去送pizza。

```swift
protocol PizzaMakerDelegate: AnyObject {
  func receivePizza(_ pizza: Pizza)
}

class PizzaMaker() {
  
  func makePizza(delegate: PizzaMakerDelegate) {
    ...
  }
}
```

那么，为了实现Robot可以给Mary点