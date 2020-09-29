+++
title= "RxJava基本语法源码分析"
date= "2019-05-05"
categories= [ "Android" ]
+++  

最近看了下网上的RxJava源码分析，发现所基于的源码版本和最新的略有不同，于是自己动手翻阅了一下最新的源码版本（rxjava:2.2.8，rxandroid:2.1.1），并写分析博客作分享。
 
```java
//示例代码
private static void rxJavaTest() {
    Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(ObservableEmitter<String> emitter) {
              //1
            emitter.onNext("onNext");
            emitter.onComplete();
        }
    }).subscribe(new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
                //2
               Log.d(TAG, "onSubscribe");
        }
        @Override
        public void onNext(String s) {
              //3
              Log.d(TAG, s);
        }
        @Override
        public void onError(Throwable e) {
              //4
              Log.d(TAG, "onError");
        }
        @Override
        public void onComplete() {
              //5
              Log.d(TAG, "onComplete");
        }
    });
}
```  
上面RxJava最简单的使用，主要涉及被观察者Observable、观察者Observer和事件订阅subscribe()三个角色。  
首先分析Observable的创建过程，即Observable的create()方法：

```java
public static <T> Observable<T> create(ObservableOnSubscribe<T> source) {
    ObjectHelper.requireNonNull(source, "source is null");
    //这里传入的source对象是我们传入的匿名内部类
    return RxJavaPlugins.onAssembly(new ObservableCreate<T>(source));
}
```  
这里先看一下我们传入的匿名内部类类型ObservableOnSubscribe源码：

```java
public interface ObservableOnSubscribe<T> {
    void subscribe(@NonNull ObservableEmitter<T> emitter) throws Exception;
}
```  
可见ObservableOnSubscribe是一个只含有一个抽象方法subscribe()的接口。


接着调用RxJavaPlugins的onAssembly()方法并传入一个新建的ObservableCreate对象，而ObservableCreate的构造函数如下：  

```java
final ObservableOnSubscribe<T> source;
public ObservableCreate(ObservableOnSubscribe<T> source) {
   this.source = source;
}
```
内部操作很简单，只是把ObservableCreate的成员变量source赋值为传入的ObservableOnSubscribe对象，即最开始我们创建的匿名内部类。  
RxJavaPlugins的onAssembly()方法调用如下：

```java
public static <T> Observable<T> onAssembly(@NonNull Observable<T> source) {
    Function<? super Observable, ? extends Observable> f = onObservableAssembly;
    if (f != null) {
        return apply(f, source);
    }
    return source;
}
```

首先会创建一个Function f，赋值为onObservableAssembly，而onObservableAssembly默认为null（当使用转换操作符时会进行赋值，在后面的文章中会进一步分析），所以会直接返回source。至此Observable创建完毕。  
然后看Observer的内部实现，是一个包含4个抽象方法的接口：

```java
public interface Observer<T> {
    void onSubscribe(@NonNull Disposable d);
    void onNext(@NonNull T t);
    void onError(@NonNull Throwable e);
    void onComplete();
}
```
最后来看重点subscribe()方法：

```java
@Override
public final void subscribe(Observer<? super T> observer) {
    ObjectHelper.requireNonNull(observer, "observer is null");
    try {
       
        observer = RxJavaPlugins.onSubscribe(this, observer);//1
        ObjectHelper.requireNonNull(observer, "The RxJavaPlugins.onSubscribe hook returned a null Observer...");
        subscribeActual(observer);//2
    } catch (NullPointerException e) { 
        throw e;
    } catch (Throwable e) {
        Exceptions.throwIfFatal(e);
        RxJavaPlugins.onError(e);
        NullPointerException npe = new NullPointerException("Actually not, but can't throw other exceptions due to RS");
        npe.initCause(e);
        throw npe;
    }
}
```

在1处调用RxJavaPlugins.onSubscribe()，将我们传入的observer进行包装：


```java
public static <T> Observer<? super T> onSubscribe(@NonNull Observable<T> source, @NonNull Observer<? super T> observer) {
    BiFunction<? super Observable, ? super Observer, ? extends Observer> f = onObservableSubscribe;
    if (f != null) {
        return apply(f, source, observer);
    }
    return observer;
}
```

和上文的onAssembly()方法一样，这里的onObservableSubscribe也默认为null，所以返回值还是我们传入的observer本身。  
重中之重在于2处的subscribeActual(observer)：

```java
//subscribeActual()是Observable中的抽象方法，本文示例的具体实现是在ObservableCreate类中
protected void subscribeActual(Observer<? super T> observer) {
    //1、创建事件发射器
    CreateEmitter<T> parent = new CreateEmitter<T>(observer);
    //2、调用observer的onSubscribe()方法
    observer.onSubscribe(parent);
    try {
        //3、调用source（即包装过的observer）的onSubscribe()方法，传入发射器
        source.subscribe(parent);
    } catch (Throwable ex) {
        Exceptions.throwIfFatal(ex);
        parent.onError(ex);
    }
}
```

第1步，创建CreateEmitter，CreateEmitter类的主要代码如下：

```java
static final class CreateEmitter<T> extends AtomicReference<Disposable>
    implements ObservableEmitter<T>, Disposable {
        ...
        final Observer<? super T> observer;
        CreateEmitter(Observer<? super T> observer) {
            //将observer赋值给成员变量
            this.observer = observer;
        }
        @Override
        public void onNext(T t) {
            if (t == null) {
                onError(new NullPointerException("onNext called with null. Null values are generally not allowed in 2.x operators and sources."));
                return;
            }
            if (!isDisposed()) {
                observer.onNext(t);
            }
        }
        @Override
        public void onError(Throwable t) {
            if (!tryOnError(t)) {
                RxJavaPlugins.onError(t);
            }
        }
        ...
        @Override
        public void onComplete() {
            if (!isDisposed()) {
                try {
                    observer.onComplete();
                } finally {
                    dispose();
                }
            }
        }
        ...
        //取消发送事件
        @Override
        public void dispose() {
            DisposableHelper.dispose(this);
        }
        @Override
        public boolean isDisposed() {
            return DisposableHelper.isDisposed(get());
        }
        ...
    }
```  
第2步，调用observer的onSubscribe()方法，这样会走到示例代码的2处打印出"onSubscribe"。      
第3步，调用source.subscribe(parent)，source实际上就是示例代码中的传入的匿名内部类：  

```java
new ObservableOnSubscribe<String>() {
   @Override
   public void subscribe(ObservableEmitter<String> emitter) {
       //1
       emitter.onNext("onNext");
       emitter.onComplete();
   }
}
``` 

所以会走到示例代码的1处分别执行emitter的onNext()和onComplete()方法，而从CreateEmitter的内部实现可见emitter的onNext()和onComplete()方法的具体操作实际上就是调用观察者observer的onNext()和onComplete()方法，observer即示例代码中的  

```java
new Observer<String>() {
        @Override
        public void onSubscribe(Disposable d) {
                //2
               Log.d(TAG, "onSubscribe");
        }
        @Override
        public void onNext(String s) {
              //3
              Log.d(TAG, s);
        }
        @Override
        public void onError(Throwable e) {
              //4
              Log.d(TAG, "onError");
        }
        @Override
        public void onComplete() {
              //5
              Log.d(TAG, "onComplete");
        }
    }
```

于是就走到了示例代码3、4处，并在调用onNext()时传入1处emitter.onNext()方法传入的参数。  
以上只是RxJava最基础的用法的分析，主要是对观察者模式的不同角色进行封装，达到链式调用形式的目的，并且设计了发射器Emitter的概念，形成流式事件订阅的模式。









