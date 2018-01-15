### Rxjava源码阅读笔记
这里只记录源码阅读，不说明rxjava库的使用

先上一段基本使用代码

```java
Observable.create(new Observable.OnSubscribe<Boolean>()
        {
            @Override
            public void call(Subscriber<? super Boolean> subscriber)
            {

            }
        }).subscribe(new Action1<Boolean>()
        {
            @Override
            public void call(Boolean aBoolean)
            {

            }
        });

```

这里调用Observable的create方法，传入参数是new一个OnSubscribe对象。这里我们看下create方法

```java
public static <T> Observable<T> create(OnSubscribe<T> f) {
    return new Observable<T>(RxJavaHooks.onCreate(f));
}
```

这里是new一个Observable对象，到这里被观察者创建好了。至于RxJavaHooks.onCreate(f)我们只要知道它返回的就是f，也就是Observable.create(f)方法里面的f。然后在看下Observable构造方法。

```java
protected Observable(OnSubscribe<T> f) {
    this.onSubscribe = f;
}
```

到这里我们先总结一下，Observable.create方法实际就是创建一个OnSubscribe对象作为自己的一个变量，且有方法call。

![image](/pics/observable.png)

第一步再简单不过，然后就是subscribe方法
```java
public final Subscription subscribe(final Action1<? super T> onNext) {
        if (onNext == null) {
            throw new IllegalArgumentException("onNext can not be null");
        }

        Action1<Throwable> onError = InternalObservableUtils.ERROR_NOT_IMPLEMENTED;
        Action0 onCompleted = Actions.empty();
        //1
        return subscribe(new ActionSubscriber<T>(onNext, onError, onCompleted));
    }
```
最后会封装成一个ActionSubscriber，从1继续往下看

```java
public final Subscription subscribe(Subscriber<? super T> subscriber) {
        return Observable.subscribe(subscriber, this);
    }
```
```java
static <T> Subscription subscribe(Subscriber<? super T> subscriber, Observable<T> observable) {
        if (subscriber == null) {
            throw new IllegalArgumentException("subscriber can not be null");
        }
        if (observable.onSubscribe == null) {
            throw new IllegalStateException("onSubscribe function can not be null.");
        }

        //这里调用观察者的onstart方法，用于数据初始化。
        subscriber.onStart();

        // 这里我们先忽略
        if (!(subscriber instanceof SafeSubscriber)) {
            // assign to `observer` so we return the protected version
            subscriber = new SafeSubscriber<T>(subscriber);
        }

        try {
            // 2. 这句是重点了
            RxJavaHooks.onObservableStart(observable, observable.onSubscribe).call(subscriber);
            
            return RxJavaHooks.onObservableReturn(subscriber);
        } catch (Throwable e) {
            //异常部分先忽略,只看正常流程
            ...
            return Subscriptions.unsubscribed();
        }
    }
```
可以看到，在订阅后，会先调用onStart方法，用于数据的初始化准备等。有效代码很少，看一下RxJavaHooks.onObservableStart方法
```java
public static <T> Observable.OnSubscribe<T> onObservableStart(Observable<T> instance, Observable.OnSubscribe<T> onSubscribe) {
    Func2<Observable, Observable.OnSubscribe, Observable.OnSubscribe> f = onObservableStart;
    if (f != null) {
        return f.call(instance, onSubscribe);
    }
    return onSubscribe;
}
```
最后只是返回，带入的第二个参数，也就是当前Observable的onSubscribe参数。回到上面2中，最后执行了call方法，且参数调用了我们的观察者。整个过程也就明白了。
