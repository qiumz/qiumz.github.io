 应用场景：当一个项目越来越庞大，假如有10个业务模块都需要使用AIDL来进行进程间通信，按之前的方式，每一个业务都创建一个service进行处理。这种弊端在于，大量的业务模块，要建立大量的service和AIDL文件进行通信，service作为4大组件之一，本身就是一种系统资源，大量的service会使得我们的应用看起来非常重量级。

 

解决方法：binder连接池机制。在这种模式下，整个工作机制是这样的，每个业务模块创建自己的AIDL接口并实现此接口，这个时候不同业务模块之间是不能有耦合的，所有细节需要我们分开单独实现，然后向服务端提供自己的唯一标识和其对应的Binder对象；对于服务端来说，只需要一个service就可以了，服务端提供一个querybinder接口，这个接口能根据业务模块的特征来返回相应的Binder对象给他们，不同的业务模块拿到所需的Binder对象就可以进行远程方法调用了。

工作原理：

![img](https://img-blog.csdn.net/20180110093112770?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzI0OTMxNTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Demo地址：[GIthub](https://github.com/ITQmz/IPCBinderPool)

注意点1：CountDownLatch的使用。由于binderService方法是异步调用，必须在onServiceConnect方法中，取到Ibinder对象后，才可以进行下一步，否则出现空指针。



```html
BinderPoolUtils.getInstance(MainActivity.this);
//如果没有使用countDown处理过，会造成空指针
IBinder serviceAdd=BinderPoolUtils.getInstance(MainActivity.this).queryBinder(0);     
```

CountDownLatch 可参考这篇文章 [ 跳转](http://www.importnew.com/15731.html)。简单来说，CountDownLatch 能使一个线程等待其他线程完成各自工作后再执行。





注意点2：binder链接池断开重连：

![img](https://img-blog.csdn.net/20180110095120422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzI0OTMxNTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdn.net/20180110095139792?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzI0OTMxNTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



在binder连接断开时候，会调binderDied方法，在里面进行重连。