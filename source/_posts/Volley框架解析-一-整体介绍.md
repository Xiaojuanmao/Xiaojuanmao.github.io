---
title: Volley框架解析(一)整体介绍
date: 2017-02-26 10:04:01
tags: volley

---

## Volley框架解析(一)整体介绍

感谢各位菊苣，[grumoon](https://github.com/grumoon "grumoon"),[huxian99](https://github.com/huxian99 "huxian99"),[trinea](https://github.com/trinea "trinea"),[郭霖juju](http://blog.csdn.net/guolin_blog "郭霖juju")的图片素材，以及详细的分析。

其他菊苣关于Volley解析的链接如下：

[codeKK—Volley源码解析](http://www.codekk.com/open-source-project-analysis/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)

[郭霖juju—Android Volley完全解析(一)，初识Volley的基本用法](http://blog.csdn.net/guolin_blog/article/details/17482095/ "郭霖juju")

<!--more-->

***

### [](#u9898_u5916_u8BDD_28_u53EF_u76F4_u63A5_u8DF3_u8FC7orz "题外话(可直接跳过orz")题外话(可直接跳过orz

在Android路上的第一个涉及到网络的项目中，就用到了Volley，当时也就照着网上的方法用了用，用到后面发现满足不了需求之后，尝试着去自定义了一些request，自己去结合Volley来处理服务器返回的cookie。第一个项目已经过去时间比较长了，突然想到想深入的了解下Volley,于是就开始了Volley源码之旅…..本人比较笨，需要比其他人花更多的时间来消化，没办法orz。看了比较长的一段时间后，把自己边看边写的笔记拿出来和大家分享。

### [](#1-_Volley_u7B80_u4ECB "1\. Volley简介")1\. Volley简介

#### [](#1-1_Volley_u662F_u4EC0_u4E48 "1.1 Volley是什么")1.1 Volley是什么

Volley是Google推出的Android异步网络请求框架和图片加载的框架。适合数据量小的,通信频繁的各种请求,官方已经封装好了各种API,而且还提供了很灵活的自定义请求接口,不仅使用起来方便,可扩展性也很强.
![volley](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/network/volley/image/volley.png)

可以通过下面的几种途径获取到volley的代码：

```
git clone https://android.googlesource.com/platform/frameworks/volley  

jar包下载地址： http://www.kwstu.com/ResourcesView/kwstu_201441183330928 

```

#### [](#1-2__u6574_u4F53_u6846_u67B6 "1.2 整体框架")1.2 整体框架

这是从上面提到的菊苣那里拿来的一张图，十分感谢Orz,这张图大致的分析出了Volley中Request从开始到结束需要经历的一个流程，在后面会详细的分析request每一步的动向，这里先简单的做个介绍。
![design](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/network/volley/image/design.png)

**最开始用到Volley发送请求的时候需要通过如下的几行代码(注意给应用添加网络访问的权限)**

```
//先新建一个请求队列(RequestQueue)
RequestQueue mQueue = Volley.newRequestQueue(context); 

//新建一个request
StringRequest stringRequest = new StringRequest("http://www.xxxxxxx.com",
                    new Response.Listener<String>() {
                        @Override
                        public void onResponse(String response) {
                            Log.d("TAG", response);
                        }
                    }, new Response.ErrorListener() {
                        @Override
                        public void onErrorResponse(VolleyError error) {
                            Log.e("TAG", error.getMessage(), error);
                        }
                    });

//将request加入到队列(RequestQueue)当中
mQueue.add(stringRequest); 

```

做完上面的这些工作,如果不出问题，等着request返回结果就可以了。结合上面的图片，mQueue(**RequestQueue**)被创建之后，会启动新的工作线程(**dispatcher**)开始工作，mQueue里面有专门用来存放request的容器，只要没被stop,这些工作线程会不停的从容器中取出request进行处理,工作线程大致分为两类：

1.  处理有缓存存在的request的dispatcher。该工作线程会涉及到从之前存储的有效缓存(**cache**)中读取数据并返回给调用者。
2.  处理网络请求的request的dispatcher。该工作线程会涉及到从网络(**network**)获取有效的数据，并返回合适数据给调用者，并会根据request的设置来决定是否将请求结果缓存到本地。

    在工作线程得到了请求响应结果response之后，会将response交给**ResponseDelivery**来处理并通过回调传递给调用者。

    通过上面的介绍，应该能大致的了解volley中，一个request创建并加入到RequestQueue之后大致的一个走向。

#### [](#1-3__u57FA_u7840_u7C7B_u7684_u7B80_u4ECB "1.3 基础类的简介")1.3 基础类的简介

在Volley中一共有43个类(不知道当前阅读的是否为最新版本的，不过核心类差不了很远）,主要介绍一下核心类以及其在Volley中起的作用，后面会对核心类的每行代码进行展开分析。

**Volley.java:** 从上面的用法`Volley.newRequestQueue`就能看出，Volley类是对外的接口，里面仅有4个重载了的`newRequestQueue()`函数，用来以各种不同的方式创建并启动一个RequestQueue。

**RequestQueue.java:** 外界通过Volley中的接口来创建其实例，RequestQueue的作用就是存放所有add进来的Request(所有的Request不仅会存放在`mCurrentRequests`里面，其原型是一个HashSet。而且Request还会被分类存放在`mCacheQueue`和`mNetworkQueue`中，分类的标准是是否涉及到网络数据的获取),并且里面会有两类调度器`mDispatchers`和`mCacheDispatcher`来负责处理Request。前者用来处理涉及到网络的Request，后者用来处理直接从缓存中获取数据的Request。它俩获得了数据之后都会交给`mDelivery`(ResponseDelivery.java的实例)来传递回caller。

**Request.java:** 请求类的基类，所有请求类都从该类继承。里面包含了请求方法(POST,GET等)，用户可自定义符合需求的Request，自由度很大。

**NetworkDispatcher.java:** 处理网络请求的调度器，继承自`Thread`类，其中包含了用于存储涉及网络请求的`mQueue`，以及用于网络请求的接口类`mNetwork`(Network.java实例)。在被停止之前进行死循环，调度器会不停的从`mQueue`中取出request来处理，将结果通过`mCache`(Cache.java实例)写入本地缓存中，通过`mDelivery`(ResponseDelivery.java实例)将结果回传给caller。

**CacheDispatcher.java:** 处理缓存请求的调度器，继承自`Thread`类，包含了用于存储涉及缓存请求的队列`mCacheQueue`，和上面的网络调度器工作原理类似。只是从缓存中取出数据再通过`mDelivery`返回给caller。

**ResponseDelivery.java:** 一个用于将Response传递给调用者的回调接口，包含了两类回调方法，`postResponse(Request<?> request, Response<?> response, Runnable runnable)`和`postError(Request<?> request, VolleyError error)`。

**Network.java:** 用于网络请求调用的接口，包含一个方法`performRequest()`。

**BasicNetwork.java:** 继承了Network类，是Volley中默认使用的网络请求处理工具类。在该类里面会处理Request发送前的一系列工作，以及发送工作和发送后返回NetworkResponse的解析工作。里面真正实现网络请求的发送工作是利用了其中的`mHttpStack`(HttpStack.java实例)。

**HttpStack.java:** 网络请求接口类，包含一个方法`performRequest(Request<?> request, Map<String, String> additionalHeaders)`。该方法和BasicNetwork类中实现的方法`performRequest(Request<?> request)`不同。前者在后者的方法中被调用，来实现真正的网络请求。

**HurlStack.java:** 实现了HttpStack接口，在android版本在2.3之上的系统中，通过HttpURLConnection类实现网络请求。

**HttpClientStack.java:** 实现了HttpStack接口，在android版本在2.3之下的系统中，通过HttpClient类实现网络请求。

**Cache.java:** 读写缓存类的接口类，抽象出了一系列有关缓存读写的方法。

**DiskBasedCache.java:** 继承并实现了Cache中的一系列方法，是Volley中默认使用的缓存读写工具类。

**Response.java:** Volley自定义的bean类，Request通过上面实现了HttpStack接口的两种实现方法发出之后，会返回相应的`NetworkResponse`类实例，这个类是`org.apache.http`包里面的类，`NetworkResponse`实例返回后，解析出有用的信息，并组成Response实例。

上面简单的介绍了Volley中的核心类，再盗用一张图orz，再次感谢上面的菊苣们。![design](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tool-lib/network/volley/image/Volley-run-flow-chart.png)

上图清晰的画出了，请求从加入到队列，怎么被分步骤处理，分缓存和网络两条路径，先查询是否存在请求对应的缓存，如存在有效缓存则直接取出缓存数据返回给调用者，如不存在有效缓存则从网络获取数据，写入缓存并返回将结果返回给调用者。

对Volley整体上的简单介绍就先到这里了，后面会将阅读源码时候的笔记整理之后再和大家分享。