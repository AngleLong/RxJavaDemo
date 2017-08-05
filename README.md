[不怕跌倒，所以飞翔](http://www.jianshu.com/u/4a99c9554afc)

特别感谢：
[南尘2251的博客](http://www.jianshu.com/p/b39afa92807e)

Rxjava最近使用的时候有很多不记得的了，所以说一定要写一篇博客记录一下，好了废话不多说，快上车吧！

##关联类库，这个是最重要的！
```
   compile 'io.reactivex.rxjava2:rxjava:2.1.2'
   compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
```

##了解操作符：
关于RxJava的操作符，我们还要一个一个的说，哎挺头疼的...哈哈!

###Create操作符
这个操作符其实就是一个创建的操作符，这个也是最基本的一个操作符：最基本的写法，上面已经写了很多的注释，基本上都能看明白的！
```
        Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("a");
                e.onNext("b");
                e.onNext("c");
                e.onNext("d");
                e.onComplete();/*结束的操作符*/
            }
        });

        observable.subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                /*开始的时候执行*/
                Log.e(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(@NonNull String s) {
                /*onNext调用的时候执行*/
                Log.e(TAG, "onNext: " + s);
                mTvContent.setText(s);
            }

            @Override
            public void onError(@NonNull Throwable e) {
                /*错误的时候执行*/
                Log.e(TAG, "onError: " + e);
            }

            @Override
            public void onComplete() {
                /*complete的时候执行，也就是最后的时候执行*/
                Log.e(TAG, "onComplete: ");
            }
        });
```
![onCreate操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-7bb7a4705f5cb08f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里面有一点要说明的，如果你在b和c中间的时候调用e.onComplete();
那么c和d就不会执行的！这里记住！！！只要是onComplete()调用的话就会终止这次操作！

####just操作符
就是发送一个指定内容的操作符
```
  Observable.just(1, 2, 3, 4, 5)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![just操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-0595e63f1bc07057.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


###Map操作符
这个操作符你可以理解为每发送一个数据，我拦截一下，做出改变之后再发送出去。
```
  Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("1");
                e.onNext("2");
                e.onNext("3");
                e.onNext("4");
                e.onComplete();
            }
        }).map(new Function<String, Integer>() {
            @Override
            public Integer apply(@NonNull String s) throws Exception {
                return Integer.valueOf(s) * 10;
            }
        }).subscribe(new Observer<Integer>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.e(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(@NonNull Integer integer) {
                Log.e(TAG, "onNext: " + integer);
                mTvContent.setText(String.valueOf(integer));
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "onError: ");
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete: ");
            }
        });
```
这个**map**的操作符里面是一个Function这个类，传入的泛型这里说明一下，前一个是Observable发送类型的泛型，后面那个是你想改变之后类型的泛型，之后接收的时候Observer会把类型自动转换成后面那个泛型的类型！（说的有点白话，但是我觉得这样比较好理解，嘻嘻）

![Map操作符操作符的](http://upload-images.jianshu.io/upload_images/2546238-18cd157542fd092c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###Zip操作符
这个操作符就是和Windows的zip文件似的，就是把两个Observable进行合并！打成一个包！（但是这里有一点要注意，两个数量不同的Observable合并的时候是以最少的为准，很好理解，就是一个多的一个说的两两配对，没有配对的就舍去了呗！单身狗的悲哀）
```
        /*发送字符串的Observable*/
        Observable<String> mStringObservable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("1");
                e.onNext("2");
                e.onNext("3");
                e.onNext("4");
                e.onNext("5");
                e.onComplete();
            }
        });

        /*发送数字的Observable*/
        Observable<Integer> mIntObservable = Observable.create(new ObservableOnSubscribe<Integer>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<Integer> e) throws Exception {
                e.onNext(1);
                e.onNext(2);
                e.onNext(3);
                e.onNext(4);
                e.onNext(5);
                e.onNext(6);
                e.onComplete();
            }
        });

        Observable.zip(mStringObservable, mIntObservable, new BiFunction<String, Integer, String>() {
            @Override
            public String apply(@NonNull String s, @NonNull Integer integer) throws Exception {
                return String.valueOf(Integer.valueOf(s) + integer);
            }
        }).subscribe(new Observer<String>() {
            @Override
            public void onSubscribe(@NonNull Disposable d) {
                Log.e(TAG, "onSubscribe: ");
            }

            @Override
            public void onNext(@NonNull String s) {
                Log.e(TAG, "onNext: " + s);
            }

            @Override
            public void onError(@NonNull Throwable e) {
                Log.e(TAG, "onError: " + e);
            }

            @Override
            public void onComplete() {
                Log.e(TAG, "onComplete: ");
            }
        });
```
这里有几点说明的，那个BiFunction类中传入的泛型，第一个和第二个是两个要合并操作符的两个泛型，第三个是合并之后的泛型，zip的操作符中间的是合并的规则，最后那个Observer的泛型会和合并之后的泛型相同的！打印结果的话就能看出来是以最少的那个数量为标准！

![Zip操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-930b12df9f1e5b7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###Concat操作符
这个操作符是把两个Observable合成一个Observable的操作符
```
Observable.concat(Observable.just(1, 2, 3), Observable.just(4, 5, 6))
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
这里说明下就是要合并的两个操作符应该属于同一类型的，并且是按照顺序进行排列的。

![Concat操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-af9728479de976b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###FlatMap操作符的使用
就是将一个Observable转换成多个Observable进行发送，然后把多个Observable装进一个发射器中进行发送，**FlatMap**不能保证顺序，这点一定要切记！
```
Observable.just(1, 2, 3, 4)
                .flatMap(new Function<Integer, ObservableSource<Integer>>() {
                    @Override
                    public ObservableSource<Integer> apply(@NonNull Integer integer) throws Exception {
                        List<Integer> mList = new ArrayList<>();
                        for (int i = 0; i < 5; i++) {
                            mList.add(i);
                        }
                        return Observable.fromIterable(mList);
                    }
                })
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: ");
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
这里简单说下逻辑，就是发送1到4四个数据，之后每一个都转换成0到4四个数据打印出来。但是这里注意一点就是使用FlatMap的时候，传入第二个参数（Observable的时候一定要指定一个泛型，这个泛型一定是之后Observer的时候转换的泛型，切记切记！！！）

######这里面虽然看不出顺序问题，但是你一定要记得这中间有这么个问题！
![FlatMap操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-c9e441118d557ba0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###concatMap和FlatMap操作符唯一的区别就是能保证顺序，就不再这里写出代码来了！

###distinct操作符
去除重复的操作符
```
 Observable.just(1, 2, 3, 4, 3, 2, 1)
                .distinct()
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: ");
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
这个是从前面开始，如果之后有重复的话就不会发送了！
![distinc操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-d35608929612edf9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###Filter操作符
过滤操作符，你自己指定一个规定，自己过滤掉不想要的就可以了
```
  Observable.just(12, 53, 34, 21, 18, 20, 39)
                .filter(new AppendOnlyLinkedArrayList.NonThrowingPredicate<Integer>() {
                    @Override
                    public boolean test(Integer integer) {
                        return integer >30;
                    }
                })
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
这里面filter过滤的时候会传入一个对象，这个对象可以是**AppendOnlyLinkedArrayList.NonThrowingPredicate**或者是**Predicate**结果都是一样的，什么区别我还真的不知道，希望哪位大神帮忙讲解下，谢谢！接着说其实这里面就是返回一个过滤的方法，结果是true和false，从而进行过滤！

![filter操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-8b2897a2dc7827cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###buffef操作符
这个操作符是buffef(count,skip)这样的，作用就是把一个Observable按照你指定的数量count每次Observable从中取出count个元素，每一个Observable是里面包含count个项目（如果个数够的话，不够的话就取剩下的），这里要说明一下，count是代表每次取出的数量，skip是步长（也就是每次跳过的个数）。
```
 Observable.just(1, 2, 3, 4, 5, 6)
                .buffer(3, 2)
                .subscribe(new Observer<List<Integer>>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull List<Integer> integers) {
                        Log.e(TAG, "onNext: " + integers);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![buffer的打印结果](http://upload-images.jianshu.io/upload_images/2546238-a4eca39dddd01254.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
说明一下这个打印结果：首先从just中取出3个也就是count个个数，发送有一个Observable，然后跳过2个也就是skip继续取count个元素发送一个Observable，依次取，最后一个由于只有两个了，所以只取了2个

###timer操作符
首先说明一下在2.0当中timer已经变成一个定时任务了，就是发送一个指定（延迟）时间的请求，而不再是执行间隔逻辑了，间隔逻辑已经被interval取代了，下面会进行讲解的！
```
  Observable.timer(1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())/*发生的线程*/
                .observeOn(AndroidSchedulers.mainThread())/*回调的线程*/
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Long aLong) {
                        Log.e(TAG, "onNext: " + String.valueOf(aLong));
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![timer的打印结果](http://upload-images.jianshu.io/upload_images/2546238-59f99081bde5ac85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意一点问题：就是timer会创建一个新的线程去执行！

###interval操作符
这个操作符就相当于一个定时的请求了，会一直执行的
```
  Observable.interval(1, 1, TimeUnit.SECONDS)
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Long aLong) {
                        Log.e(TAG, "onNext: " + String.valueOf(aLong));
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![interval操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-5906b6bdc7bb58b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果想要停止的话可以使用Disposable 这个对象的dispose()方法

####doOnNext操作符
**其实它不算做操作符，只是比较常用**
它的作用是让订阅者在接收到数据之前干点有意思的事情。假如我们在获取到数据之前想先保存一下它，我们可以这样实现。
```
   Observable.just(1, 2, 3, 4, 5)
                .doOnNext(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "doOnNext: 这里做一些操作就可以了");
                    }
                })
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![doOnNext的打印结果](http://upload-images.jianshu.io/upload_images/2546238-a741cc1eb538f71c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实就是在每次发送之前先走doOnNext方法进行一些操作，它会在onNext之前调用。

####skip操作符
这个操作符其实就是跳过多少个条目之后执行，没有什么好说的，看一下打印结果就可以了！
```
Observable.just(1, 2, 3, 4, 5, 6, 7)
                .skip(3)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![skip操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-14377427ac8557be.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####take操作符
take操作符的作用就是最多能接受多少个参数，这个是从头开始来的。
```
 Observable.just(1, 2, 3, 4, 5, 6, 7)
                .take(4)
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![take操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-9ab4c13fa2422c0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####debounce操作符
这个操作符的作用就是去掉发送频率过快的项
```
 Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {

                e.onNext("1");// skip
                Thread.sleep(400);
                e.onNext("2"); // deliver
                Thread.sleep(505);
                e.onNext("3"); // skip
                Thread.sleep(100);
                e.onNext("4"); // deliver
                Thread.sleep(605);
                e.onNext("5"); // deliver
                Thread.sleep(510);
                e.onComplete();
            }
        })
                .debounce(500, TimeUnit.MILLISECONDS)
                .subscribe(new Observer<String>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull String s) {
                        Log.e(TAG, "onNext: " + s);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: " + e);
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![debounce操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-3d278c49212863a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个操作符还是要说一下的！他是去除发送较为频繁的项，这里指定的是500毫秒，所以就把发送在500毫秒以内的全部清除掉了。

####defer操作符
这个操作符的作用就是把之前的Observable转换成一个新的Observable发送出来，如果没有的话他就不发送了，我是这么理解的！
```
 Observable.defer(new Callable<ObservableSource<Integer>>() {
            @Override
            public ObservableSource<Integer> call() throws Exception {
                return Observable.just(1, 2, 3);
            }
        })
                .subscribe(new Observer<Integer>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                        Log.e(TAG, "onSubscribe: ");
                    }

                    @Override
                    public void onNext(@NonNull Integer integer) {
                        Log.e(TAG, "onNext: " + integer);
                    }

                    @Override
                    public void onError(@NonNull Throwable e) {
                        Log.e(TAG, "onError: ");
                    }

                    @Override
                    public void onComplete() {
                        Log.e(TAG, "onComplete: ");
                    }
                });
```
![defer操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-b11c1e7be3229ad7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####last操作符
这个操作符是获取最后一个发送项
```
 Observable.just(1,2,3,4,5)
                .last(1)
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "accept: " + integer);
                    }
                });
```
其实这里我测试过，这个last后面的参数如论传入多少获取的都是最后一项，写得是默认的条目，这里我还是不太理解，希望理解的大神说一下！

#### merge操作符
这个操作符的作用就是合并两个Observable
```
  Observable.merge(Observable.just(1, 2, 3), Observable.just(4, 5, 6))
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "accept: " + integer);
                    }
                });
```
![Merge操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-ddbb406495b9030d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### reduce操作符
每次用一个方法处理一个值，可以有一个 seed 作为初始值。
其实这个操作符的理解就是用一个方法把两个对象操作一下，然后合成了一个对象在操作一下，最后返回了一个对象.
```
 Observable.just(1, 2, 3, 4, 5, 6)
                .reduce(10, new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                })
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "accept: " + integer);
                    }
                });
```
![reduce操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-f4c9d90348f71a6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里还是说明一下，这个呢就是发送1到6，6个数字，但是中间经过了reduce方法之后，使得这一次的结果和上次的结果进行相加，前面那个10 是一个初始值。

####scan操作符
其实这个操作符和上面的reduce操作符类似，但是不同的是他会把每一个步骤都打印出来，而reduce只会打印结果
```
Observable.just(1, 2, 3, 4, 5, 6)
                .scan(10, new BiFunction<Integer, Integer, Integer>() {
                    @Override
                    public Integer apply(@NonNull Integer integer, @NonNull Integer integer2) throws Exception {
                        return integer + integer2;
                    }
                })
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(Integer integer) throws Exception {
                        Log.e(TAG, "accept: " + integer);
                    }
                });
```
代码都差不多，只是打印结果略有不同
![scan操作符的打印结果](http://upload-images.jianshu.io/upload_images/2546238-c794dc3336cbde6d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

基本上的操作符就讲到这里吧！我也是能力有限，写一半加深一下自己的印象，也希望对其他朋友有帮助！！！
下次就看看实际项目中怎么用吧！！！
