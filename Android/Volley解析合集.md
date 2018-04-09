### 相关文章推荐
> - **郭霖的四篇解析**
> - [Android Volley完全解析(一)，初识Volley的基本用法](http://blog.csdn.net/guolin_blog/article/details/17482095)
> - [Android Volley完全解析(二)，使用Volley加载网络图片](http://blog.csdn.net/guolin_blog/article/details/17482165)
> -  [Android Volley完全解析(三)，定制自己的Request](http://blog.csdn.net/guolin_blog/article/details/17612763)
> [Android访问网络，使用HttpURLConnection还是HttpClient？](http://blog.csdn.net/guolin_blog/article/details/12452307)
> - [Android Volley完全解析(四)，带你从源码的角度理解Volley](http://blog.csdn.net/guolin_blog/article/details/17656437)
> - **三篇详细解析**
> - [聊下Volley源码（整体流程）](http://blog.csdn.net/sinat_23092639/article/details/54988070)
> - [聊聊Volley源码（网络请求过程）](http://blog.csdn.net/sinat_23092639/article/details/55673513)
> - [聊聊Volley源码（缓存流程）](http://blog.csdn.net/sinat_23092639/article/details/57599954)
> - **鸿洋的两篇解析**
> - [Android Volley 之自定义Request](http://blog.csdn.net/lmj623565791/article/details/24589837)
> - [Volley 图片加载相关源码解析](http://blog.csdn.net/lmj623565791/article/details/47721631)
> - **我们看volley主要是能从中学到知识，而不是单纯看实现**
> - [通过Volley我们能学到什么?(1) — 工作原理与设计模式](http://blog.csdn.net/u010825228/article/details/50457595)
> - [通过Volley我们能学到什么?(2) — 刨析网络请求框架](http://blog.csdn.net/u010825228/article/details/50608312)
> - [通过Volley我们能学到什么?(3) — 缓存原理](http://blog.csdn.net/u010825228/article/details/50608377)

以上文章推荐大家都看一下，Volley的流程都分析的很清楚，尤其是郭霖大神的第四篇文章，抓住脉络忽略细节才不会在源码中迷失方向。大神们写的都很完善了，我再对一些细节进行补充。

**细节一：如何中断一个正在运行的线程**
Dispatcher都是一些无限循环的线程，Volley如何保证其关闭的？
初始化了RequestQueue，调用了start()方法，其中start()的第一步是调用了stop()来确保CacheDispatcher和NetworkDispatcher已经退出了。我们来分析一下CacheDispatcher的退出过程。
```
 mCacheDispatcher.quit();
```

```
public void quit() {
        mQuit = true;
        interrupt();
    }
```
可以看到CacheDispatcher这个线程的quit()方法内调用了interrupt()方法中断线程。

如果线程在调用 Object 类的 wait() 或 sleep(long) 等方法方法过程中受阻，则其中断状态将被清除，它还将收到一个InterruptedException异常。

这时候会使得run()方法抛出一个InterruptedException异常，我们捕获这个异常来止线程的执行，具体可以通过return等退出或改变共享变量的值使其退出。
```
while (true) {
            // release previous request object to avoid leaking request object when mQueue is drained.
            request = null;
            try {
                // Take a request from the queue.
                request = mCacheQueue.take();
            } catch (InterruptedException e) {
                // We may have been interrupted because it was time to quit.
                if (mQuit) {
                    return;
                }
                continue;
            }
            ......
}
```


**细节二：图片压缩**
对于图片压缩的代码，可以在ImageRequest的parseNetworkResponse里面去看看，是如何压缩的。
```
private Response<Bitmap> doParse(NetworkResponse response) {
        byte[] data = response.data;
        BitmapFactory.Options decodeOptions = new BitmapFactory.Options();
        Bitmap bitmap = null;
        if (mMaxWidth == 0 && mMaxHeight == 0) {
            decodeOptions.inPreferredConfig = mDecodeConfig;
            bitmap = BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
        } else {
            // If we have to resize this image, first get the natural bounds.
            decodeOptions.inJustDecodeBounds = true;
            BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);
            int actualWidth = decodeOptions.outWidth;
            int actualHeight = decodeOptions.outHeight;

            // Then compute the dimensions we would ideally like to decode to.
            int desiredWidth = getResizedDimension(mMaxWidth, mMaxHeight,
                    actualWidth, actualHeight, mScaleType);
            int desiredHeight = getResizedDimension(mMaxHeight, mMaxWidth,
                    actualHeight, actualWidth, mScaleType);

            // Decode to the nearest power of two scaling factor.
            decodeOptions.inJustDecodeBounds = false;
            // TODO(ficus): Do we need this or is it okay since API 8 doesn't support it?
            // decodeOptions.inPreferQualityOverSpeed = PREFER_QUALITY_OVER_SPEED;
            decodeOptions.inSampleSize =
                findBestSampleSize(actualWidth, actualHeight, desiredWidth, desiredHeight);
            Bitmap tempBitmap =
                BitmapFactory.decodeByteArray(data, 0, data.length, decodeOptions);

            // If necessary, scale down to the maximal acceptable size.
            if (tempBitmap != null && (tempBitmap.getWidth() > desiredWidth ||
                    tempBitmap.getHeight() > desiredHeight)) {
                bitmap = Bitmap.createScaledBitmap(tempBitmap,
                        desiredWidth, desiredHeight, true);
                tempBitmap.recycle();
            } else {
                bitmap = tempBitmap;
            }
        }

        if (bitmap == null) {
            return Response.error(new ParseError(response));
        } else {
            return Response.success(bitmap, HttpHeaderParser.parseCacheHeaders(response));
        }
    }
```
可以看出来Volley也是通过设置BitmapFactory.Options中inSampleSize的值来实现处理图片压缩的。

如果不了解推荐看[Android高效加载大图、多图解决方案，有效避免程序OOM](http://blog.csdn.net/guolin_blog/article/details/9316683)这篇博客。

**Pass：自己实现了一个精简版本的Volley**
**[simple-volley](https://github.com/liuhuiAndroid/simple-volley)**
