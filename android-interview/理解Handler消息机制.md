Handler一般用于解决线程与线程间的通信，如UI更新。因为Android规定访问UI只能在主线程中，当我们有一个子线程耗时处理，完成后更新UI的操作，这时Handler很容易帮我们把任务切回到它所在的线程。

想要了解Handler消息机制，首先要了解几个概念：
Java层消息机制：Message 消息主题；MessageQueue 消息队列；Looper 消息循环，用于循环取出消息；Handler 消息处理
Native层消息机制：包括Looper（native）；NaticeMessageQueue

Handler的运行机制，Handler的创建会采用当前线程的Looper来构建内部的消息循环系统，如果当前线程没有Looper就会报错，这时就需要我们通过Looper.prepare()来构建当前线程Looper。Looper构建时会对MessageQueue进行初始化，MessageQueue初始化时会通过 nativeInit()会构建一个Native层的NaticeMessageQueue对象，至此整个初始化过程完成。

消息循环机制是指实现一个循环监听，通过Linux中IO多路复用里的epoll机制来实现阻塞，不断通过MessageQueue中next()方法取出消息然后执行处理。具体运行流程为：
Looper.loop()开启消息循环，loop()是一个死循环，loop()中会调用MessageQueue的next()方法获取新消息，而next()是一个阻塞操作，当没有消息时，next()方法会一直阻塞在那里，所以loop()方法也一直阻塞在那里；如果返回了新消息，会在loop()中调用msg.target.dispatchMessage()方法来处理，这里的msg.target.是发送这条消息的Handler对象，这样就成功的把任务切换到指定线程中去执行了。

这里要注意的几点有：
loop()方法是一个阻塞方法，何时唤醒？这里是在Handler发送消息调用MessageQueue的equeueMessage(msg)时， 有新的Message添加的队列并调用nativeWake(mPtr)，wake会让Native层的Looper取消阻塞，同时调用Java层的loop()循环。

dispatchMessage()是用来分发处理消息的，首先判断msg.callback是不是null，msg.callback实际上就是我们调用handler的post方法传入的Runnable，而handleCallback方法也就是执行Runnable.run()，当msg.callback不为null时执行handleCallback，当msg.callback为null时会先判断mCallback是不是null，mCallback是我们在初始化Handler时传入的参数，如果外部mCallback.handleMessage(msg)返回true，表示不执行Handler中的handleMessage，否则就继续往下执行

https://blog.csdn.net/lyl278401555/article/details/51829381