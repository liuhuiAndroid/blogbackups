* 因为之前是看[可能是东半球最全的RxJava使用场景小结](http://blog.csdn.net/theone10211024/article/details/50435325)来学习RxJava的，由于升级2.0版本部分API已经替换。所以打算仿写一个。
个人所有的关于RxJava2的Sample都放在**[RxJava2-Android-Samples](https://github.com/liuhuiAndroid/RxJava2-Android-Samples)**

- ###一、Scheduler线程切换
这种场景经常会在“后台线程取数据，主线程展示”的模式中看见
subscribeOn指定Observable线程
observeOn指定Observer线程
**subscribeOn()改变调用它之前代码的线程，多次调用subscribeOn()却只有第一个起作用**

**subscribeOn 作用于该操作符之前的 Observable 的创建操符作以及 doOnSubscribe 操作符 ，换句话说就是 doOnSubscribe 以及 Observable 的创建操作符总是被其之后最近的 subscribeOn 控制**

**observeOn()改变调用它之后代码的线程，多次调用observeOn()却可以切换到不同线程**
[observeOn()与subscribeOn()的详解](http://blog.csdn.net/xx326664162/article/details/51967967)
```
Observable.just(1, 2)
                .subscribeOn(Schedulers.io()) 
                .observeOn(AndroidSchedulers.mainThread()) 
                .map(new Function<Integer, String>() {
                    @Override
                    public String apply(@NonNull Integer integer) throws Exception {
                        Logger.i("apply testChangeThread:" + Thread.currentThread().getName());
                        return integer.intValue()+" - ";
                    }
                })

                .observeOn(Schedulers.io()) 
                .map(new Function<String, Integer>() {
                    @Override
                    public Integer apply(@NonNull String s) throws Exception {
                        Logger.i("apply3 testChangeThread:" + Thread.currentThread().getName());
                        return 1;
                    }
                })

                .observeOn(AndroidSchedulers.mainThread()) 
                .subscribe(new Consumer<Integer>() {
                    @Override
                    public void accept(@NonNull Integer integer) throws Exception {
                        Logger.i("subscribe testChangeThread:" + Thread.currentThread().getName());
                    }
                });
```
- ###二、使用debounce做textSearch  **[RxBinding](https://github.com/JakeWharton/RxBinding)**
用简单的话讲就是当N个结点发生的时间太靠近（即发生的时间差小于设定的值T），debounce就会自动过滤掉前N-1个结点。
比如在做百度地址联想的时候，可以使用debounce减少频繁的网络请求。避免每输入（删除）一个字就做一次联想。

RxJava also implements the switchMap operator. It behaves much like flatMap, except that whenever a new item is emitted by the source Observable, it will unsubscribe to and stop mirroring the Observable that was generated from the previously-emitted item, and begin only mirroring the current one.
switch()和flatMap()很像，除了一点:当源Observable发射一个新的数据项时，如果旧数据项订阅还未完成，就取消旧订阅数据和停止监视那个数据项产生的Observable,开始监视新的数据项.
```
RxTextView.textChanges(mEditSearch)
                .debounce(500, TimeUnit.MILLISECONDS)
                .subscribeOn(AndroidSchedulers.mainThread())
                //过滤数据
                .filter(new Predicate<CharSequence>() {
                    @Override
                    public boolean test(@NonNull CharSequence charSequence) throws Exception {
                        Logger.i("filter = " + charSequence);
                        return charSequence.toString().trim().length() > 0;
                    }
                })
                .switchMap(new Function<CharSequence, ObservableSource<List<String>>>() {
                    @Override
                    public ObservableSource<List<String>> apply(@NonNull CharSequence charSequence) throws Exception {
                        Logger.i("flatMap = " + charSequence);
                        //search from net
                        List<String> stringList = new ArrayList<String>();
                        stringList.add("abc");
                        stringList.add("ada");
                        return Observable.just(stringList);
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<List<String>>() {
                    @Override
                    public void accept(@NonNull List<String> strings) throws Exception {
                        Logger.i("subscribe = " + strings.toString());
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(@NonNull Throwable throwable) throws Exception {
                        throwable.printStackTrace();
                    }
                });
```
- ###三、防止按钮重复点击
ThrottleFirst:允许设置一个时间长度，之后它会发送固定时间长度内的第一个事件，而屏蔽其它事件，在间隔达到设置的时间后，可以再发送下一个事件。用debounce也可以
```
RxView.clicks(mBtnThrottleFirst)
                .throttleFirst(1,TimeUnit.SECONDS)
                .subscribe(new Consumer<Object>() {
                    @Override
                    public void accept(@NonNull Object o) throws Exception {
                        Logger.i("按钮点击："+System.currentTimeMillis());
                    }
                });
```
- ###
```
```
- ###八、使用timer做定时操作。当有“x秒后执行y操作”类似的需求的时候，想到使用timer
例如：2秒后输出日志“hello world”，然后结束。
```
Observable.timer(2, TimeUnit.SECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable disposable) {
                        Log.i(TAG, " onSubscribe : " + disposable.isDisposed());
                    }
                    @Override
                    public void onNext(Long aLong) {
                        Log.i(TAG, " onNext : value : " + aLong);
                    }
                    @Override
                    public void onError(Throwable throwable) {
                        Log.i(TAG, " onError : " + throwable.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.i(TAG, " onComplete");
                    }
                });
```
- ###九、使用interval做周期性操作。当有“每隔xx秒后执行yy操作”类似的需求的时候，想到使用interval
例如：每隔2秒输出日志“helloworld”。
```
Observable.interval(0, 2, TimeUnit.SECONDS)
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable disposable) {
                        Log.i(TAG, " onSubscribe : " + disposable.isDisposed());
                    }
                    @Override
                    public void onNext(Long aLong) {
                        Log.i(TAG, " onNext : value : " + aLong);
                    }
                    @Override
                    public void onError(Throwable throwable) {
                        Log.i(TAG, " onError : " + throwable.getMessage());
                    }
                    @Override
                    public void onComplete() {
                        Log.i(TAG, " onComplete");
                    }
                });
```
- ###十、使用throttleFirst防止按钮重复点击
ps：debounce也能达到同样的效果
```
```
- ###十一、发送验证码倒计时案例
```
    @OnClick(R.id.btn_code)
    public void mBtnCode() {
        //倒计时十秒
        int time = 10;
        Observable.interval(0, 1, TimeUnit.SECONDS)
                .take(time + 1)
                .map(new Function<Long, Long>() {
                    @Override
                    public Long apply(@NonNull Long aLong) throws Exception {
                        return 10 - aLong;
                    }
                })
                // Run on a background thread
                .subscribeOn(Schedulers.io())
                // Be notified on the main thread
                .observeOn(AndroidSchedulers.mainThread())
                .doOnSubscribe(new Consumer<Disposable>() {
                    @Override
                    public void accept(@NonNull Disposable disposable) throws Exception {
                        Log.i(TAG, " accept : " + disposable.isDisposed());
                        mBtnCode.setEnabled(false);
                        mBtnCode.setTextColor(Color.GRAY);
                    }
                })
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable disposable) {
                    }

                    @Override
                    public void onNext(Long aLong) {
                        mBtnCode.setText("剩余" + aLong + "秒");
                    }

                    @Override
                    public void onError(Throwable throwable) {
                        throwable.printStackTrace();
                    }

                    @Override
                    public void onComplete() {
                        mBtnCode.setEnabled(true);
                        mBtnCode.setText("发送验证码");
                        mBtnCode.setTextColor(Color.BLACK);
                    }
                });
    }
```
- ###十二、购物车合并本地和网络数据的案例
```
    public void testMerge(){
        Observable.merge(getDataFromLocal(),getDataFromNetWork())
                .subscribe(new Observer<List<String>>() {
                    @Override
                    public void onSubscribe(@NonNull Disposable d) {
                    }
                    @Override
                    public void onNext(@NonNull List<String> strings) {
                        for (int i = 0; i < strings.size(); i++) {
                            Log.i(TAG,"onNext string = "+strings.get(i));
                        }
                    }
                    @Override
                    public void onError(@NonNull Throwable e) {
                        e.printStackTrace();
                    }
                    @Override
                    public void onComplete() {
                        Log.i(TAG,"onComplete. ");
                    }
                });
    }

    private Observable<List<String>> getDataFromLocal(){
        List<String> list = new ArrayList<>();
        list.add("Local");
        list.add("Local2");
        return Observable.just(list);
    }

    private Observable<List<String>> getDataFromNetWork(){
        List<String> list = new ArrayList<>();
        list.add("NetWork");
        list.add("NetWork2");
        return Observable.just(list).subscribeOn(Schedulers.io());
    }
```
- ### 十三、登录后获取用户信息
FlatMap是返回一个Observable，而Map是返回基础数据，这是他们的区别
```
 Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://")
                .addConverterFactory(GsonConverterFactory.create())
                .build();
        mDemoApi = retrofit.create(DemoApi.class);

 Observable.just(getUserParam())
                .flatMap(new Function<UserParam, ObservableSource<BaseResult>>() {
                    @Override
                    public ObservableSource<BaseResult> apply(@NonNull UserParam userParam) throws Exception {
                        BaseResult baseResult = mDemoApi.login(userParam).execute().body();
                        return Observable.just(baseResult);
                        //方法二
                        //return mDemoApi.login2(userParam);
                    }
                })
                .flatMap(new Function<BaseResult, ObservableSource<User>>() {
                    @Override
                    public ObservableSource<User> apply(@NonNull BaseResult baseResult) throws Exception {
                        User user = mDemoApi.getUserInfoWithPath(baseResult.getUser_id()).execute().body();
                        return Observable.just(user);
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<User>() {
                    @Override
                    public void accept(@NonNull User user) throws Exception {
                        
                    }
                });
```
```
public interface DemoApi {

    @GET("user/{id}")
    Call<User> getUserInfoWithPath(@Path("id") int user_id);

    @POST("login/json")
    Call<BaseResult> login(@Body UserParam userParam);

    @POST("login/json")
    Observable<BaseResult> login2(@Body UserParam userParam);

}
```
- ###十四、优先内存缓存、其次磁盘缓存，找不到在网络上查找
```
        final Observable<String> memoryObservable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("");
                e.onComplete();
            }
        });

        final Observable<String> diskObservable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("disk");
                e.onComplete();
            }
        });

        final Observable<String> networkObservable = Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
                e.onNext("network");
                e.onComplete();
            }
        });

        RxView.clicks(mButton).subscribe(new Consumer<Object>() {
            @Override
            public void accept(@NonNull Object o) throws Exception {
                Observable.concat(memoryObservable,diskObservable,networkObservable)
                        .filter(new Predicate<String>() {
                            @Override
                            public boolean test(@NonNull String s) throws Exception {
                                return s!=null || !TextUtils.isEmpty(s.toString());
                            }
                        })
                        .firstElement()
                        .subscribe(new Consumer<String>() {
                            @Override
                            public void accept(@NonNull String s) throws Exception {
                                Logger.i("accept = "+s);
                            }
                        });
            }
        });
```
- ###
```
```
