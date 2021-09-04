---
title: MessageQueue中的nativePollOnce
date: 2018-04-12 10:04:01
tags: Android

---

之前了解过的Handler、MessageQueue以及Looper等机制，在MessageQueue的`nativePollOnce`就终止了，在没有队列中没有消息的状态下，不同Android版本从Linux层面使用管道等方式实现异步阻塞/唤醒。

在C++层的代码中，也存在一套Handler、MessageQueue以及Looper。

`android_os_MessageQueue.cpp`中`nativePollOnce`

```
static void android_os_MessageQueue_nativePollOnce(JNIEnv* env, jobject obj,
        jlong ptr, jint timeoutMillis) {
    NativeMessageQueue* nativeMessageQueue = reinterpret_cast<NativeMessageQueue*>(ptr);
    nativeMessageQueue->pollOnce(env, obj, timeoutMillis);
}
```

<!--more-->

调用了`nativeMessageQueue`的`pollOnce`方法:

```
void NativeMessageQueue::pollOnce(JNIEnv* env, jobject pollObj, int timeoutMillis) {
    mPollEnv = env;
    mPollObj = pollObj;
    mLooper->pollOnce(timeoutMillis);
    mPollObj = NULL;
    mPollEnv = NULL;
    if (mExceptionObj) {
        env->Throw(mExceptionObj);
        env->DeleteLocalRef(mExceptionObj);
        mExceptionObj = NULL;
    }
}
```

紧接着调用了`Looper.cpp`的`pollOnce`方法:

```
int Looper::pollOnce(int timeoutMillis, int* outFd, int* outEvents, void** outData) {
    int result = 0;
    for (;;) {
        while (mResponseIndex < mResponses.size()) {
            const Response& response = mResponses.itemAt(mResponseIndex++);
            int ident = response.request.ident;
            if (ident >= 0) {
                int fd = response.request.fd;
                int events = response.events;
                void* data = response.request.data;
#if DEBUG_POLL_AND_WAKE
                ALOGD("%p ~ pollOnce - returning signalled identifier %d: "
                        "fd=%d, events=0x%x, data=%p",
                        this, ident, fd, events, data);
#endif
                if (outFd != NULL) *outFd = fd;
                if (outEvents != NULL) *outEvents = events;
                if (outData != NULL) *outData = data;
                return ident;
            }
        }
        if (result != 0) {
#if DEBUG_POLL_AND_WAKE
            ALOGD("%p ~ pollOnce - returning result %d", this, result);
#endif
            if (outFd != NULL) *outFd = 0;
            if (outEvents != NULL) *outEvents = 0;
            if (outData != NULL) *outData = NULL;
            return result;
        }
        result = pollInner(timeoutMillis);
    }
}
```

最后会跟踪到了`Looper.cpp`的`pollInner`方法上，这个方法会对底层的消息进行处理，线程的阻塞也会在函数`epoll_wait()`这里产生。

```
int Looper::pollInner(int timeoutMillis) {
#if DEBUG_POLL_AND_WAKE
    ALOGD("%p ~ pollOnce - waiting: timeoutMillis=%d", this, timeoutMillis);
#endif
    // Adjust the timeout based on when the next message is due.
    if (timeoutMillis != 0 && mNextMessageUptime != LLONG_MAX) {
        nsecs_t now = systemTime(SYSTEM_TIME_MONOTONIC);
        int messageTimeoutMillis = toMillisecondTimeoutDelay(now, mNextMessageUptime);
        if (messageTimeoutMillis >= 0
                && (timeoutMillis < 0 || messageTimeoutMillis < timeoutMillis)) {
            timeoutMillis = messageTimeoutMillis;
        }
#if DEBUG_POLL_AND_WAKE
        ALOGD("%p ~ pollOnce - next message in %" PRId64 "ns, adjusted timeout: timeoutMillis=%d",
                this, mNextMessageUptime - now, timeoutMillis);
#endif
    }
    // Poll.
    int result = POLL_WAKE;
    mResponses.clear();
    mResponseIndex = 0;
    // We are about to idle.
    mPolling = true;
    struct epoll_event eventItems[EPOLL_MAX_EVENTS];
    int eventCount = epoll_wait(mEpollFd, eventItems, EPOLL_MAX_EVENTS, timeoutMillis);
    // No longer idling.
    mPolling = false;
    // Acquire lock.
    mLock.lock();
    
    ...
    
}
```

在C++层的`Looper`初始化的过程中，会初始化一个`管道`，管道会提供读、写两个端口，并创建一个`epoll`实例来监控管道的读端口。具体代码如下:

```
Looper::Looper(bool allowNonCallbacks) :
        mAllowNonCallbacks(allowNonCallbacks), mSendingMessage(false),
        mResponseIndex(0), mNextMessageUptime(LLONG_MAX) {
    int wakeFds[2];
    int result = pipe(wakeFds); // 创建管道
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not create wake pipe.  errno=%d", errno);
    mWakeReadPipeFd = wakeFds[0]; // 读端口
    mWakeWritePipeFd = wakeFds[1]; // 写端口
    result = fcntl(mWakeReadPipeFd, F_SETFL, O_NONBLOCK);
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not make wake read pipe non-blocking.  errno=%d",
            errno);
    result = fcntl(mWakeWritePipeFd, F_SETFL, O_NONBLOCK);
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not make wake write pipe non-blocking.  errno=%d",
            errno);
    // Allocate the epoll instance and register the wake pipe.
    mEpollFd = epoll_create(EPOLL_SIZE_HINT); // 创建一个epoll实例，用来监控mWakeReadPipeFd
    LOG_ALWAYS_FATAL_IF(mEpollFd < 0, "Could not create epoll instance.  errno=%d", errno);
    struct epoll_event eventItem;
    memset(& eventItem, 0, sizeof(epoll_event)); // zero out unused members of data field union
    eventItem.events = EPOLLIN;
    eventItem.data.fd = mWakeReadPipeFd;
    result = epoll_ctl(mEpollFd, EPOLL_CTL_ADD, mWakeReadPipeFd, & eventItem); // 注册监听
    LOG_ALWAYS_FATAL_IF(result != 0, "Could not add wake read pipe to epoll instance.  errno=%d",
            errno);
}
```

使用`epoll`注册监听`Looper`中的`mWakeReadPipeFd`之后，在上面提到过的`epoll_wait()`函数等待的线程，将会在`mWakeWritePipeFd`端口有数据写入的时候被唤醒，在Java层的`MessageQueue`中，有消息加入队列的时候，会检测是否需要唤醒当前在`epoll_wait()`等待的线程，如果需要唤醒，则会调用`MessageQueue.cpp`的`nativeWake()`方法，进而调用到`Looper.cpp`的`wake()`方法:

```
void Looper::wake() {
#if DEBUG_POLL_AND_WAKE
    ALOGD("%p ~ wake", this);
#endif
    ssize_t nWrite;
    do {
        nWrite = write(mWakeWritePipeFd, "W", 1);
    } while (nWrite == -1 && errno == EINTR);
    if (nWrite != 1) {
        if (errno != EAGAIN) {
            ALOGW("Could not write wake signal, errno=%d", errno);
        }
    }
}
```

唤醒的方式为，向之前创建的管道中写入一个`W`字符，管道的读端将会被唤醒，线程离开`epoll_wait()`方法，开始执行之后的代码，从消息队列中取出消息，开始干活.

##### 参考文章

> http://www.bijishequ.com/detail/214262
> http://shangjin615.iteye.com/blog/1778615

#### 源代码

[Looper.cpp](https://chromium.googlesource.com/aosp/platform/system/core/+/master/libutils/Looper.cpp)

[MessageQueue.cpp](https://android.googlesource.com/platform/frameworks/base/+/master/core/jni/android_os_MessageQueue.cpp)