
### Rxjava源码阅读笔记2
1中理解了，rxjava的基本流程，接下来看一下map方法。
```java
Observable.create(new Observable.OnSubscribe<Boolean>()
        {
            @Override
            public void call(Subscriber<? super Boolean> subscriber)
            {

            }
        }).map(new Func1<Boolean, Boolean>()
        {
            @Override
            public Boolean call(Boolean aBoolean)
            {
                return null;
            }
        }).subscribe(new Action1<Boolean>()
        {
            @Override
            public void call(Boolean aBoolean)
            {

            }
        });
```
直接看下map方法：
```java
 public final <R> Observable<R> map(Func1<? super T, ? extends R> func) {
        //1. 创建一个新的Observable对象
        return create(new OnSubscribeMap<T, R>(this, func));
    }
```
哇，创建了一个新的被观察者，跟踪看一下OnSubscribeMap类。
```java
public OnSubscribeMap(Observable<T> source, Func1<? super T, ? extends R> transformer) {
    this.source = source;
    this.transformer = transformer;
}

//2
@Override
public void call(final Subscriber<? super R> o) {
    //3 创建一个MapSubscriber
    MapSubscriber<T, R> parent = new MapSubscriber<T, R>(o, transformer);
    o.add(parent);
    //4
    source.unsafeSubscribe(parent);
}
```
从基本流程的理解，我们知道，在订阅以后，会调用观察者的onStart方法，然后调用被观察者的onSubscribe变量的call方法。那么不就是执行2吗。那么就看下call方法，3中创建了一个MapSubscriber对象，4中原来最初的Observable被观察者又被我们创建的MapSubscriber订阅。
总结一下：

![image](/pics/rxjava_map.png)

