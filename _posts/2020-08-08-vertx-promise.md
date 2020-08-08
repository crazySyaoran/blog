---
layout: post
title: AbstractVerticle.start()接受Promise类型的参数
date: 2020-08-08
Author: Syaoran
tags: [vertx, java]
comments: true
---

在vert.x中几乎所有Verticle都要继承AbstractVerticle这个类，而在这个类的定义中，start()方法的定义只有两种：
```
    // 第一种
    /** @deprecated */
    @Deprecated
    // 用Future<T>作为入参的方法已经被弃用了
    public void start(Future<Void> startFuture) throws Exception {
        this.start();
        startFuture.complete();
    }
    
    // 第二种
    // 这个是空的
    public void start() throws Exception {
    }
```
但很神奇的是有些sampleCode中传一个`Promise<T>`类型的参数也不会出问题，我试着单步了一下：

![1622c18f1239cc0d2a11e0cdd9357d01.png](..\post_images\1622c18f1239cc0d2a11e0cdd9357d01.png) 

可见由于某些原因这个`Promise<T>`类型的参数被自动转换成了`Future<T>`类型

哦。我找到了。

AbstractVerticle类中实现了Verticle接口：
```
public abstract class AbstractVerticle implements Verticle {...}
```
我忘记了在Java8之后接口类中也可以通过**default**关键字来对方法进行实现

![822e3d3fb24ea89bdec47d8078f93db8.png](..\post_images\822e3d3fb24ea89bdec47d8078f93db8.png)
> 这个位置夹在两个Deprecated中间是真的考验眼力。而且为什么IDE识别到了这个方法却不能进行跳转= =

点进去可以发现Verticle中通过default实现了`start(Promise<Void>)`方法，而且实际上就是把Promise~~强制~~类型转成Future = =
> 感谢[itgeekhome](https://itgeekhome.com/)的指正，这里应该不算强转

> 这样就顺便复习一下default关键字：default关键字由Java8引入，通过default就可以完成在接口类中进行方法的实现，实现了default方法的接口依然是接口，这意味着Java中可以通过这种方法完成“多重继承”（如果实现的两个接口中有同名方法会报错，需要手动override）。另外，default方法也是可以被Override的。













