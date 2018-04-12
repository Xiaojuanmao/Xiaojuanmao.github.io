---
title: SharedPreferece产生的Concurrent Modification Exception
date: 2018-04-12 10:04:03
tags: Android

---

在使用SharedPreference的过程中，Bugly上报出`Concurrent ModificationException`

```
java.util.HashMap$HashIterator.nextEntry(HashMap.java:806)
java.util.HashMap$KeyIterator.next(HashMap.java:833)
com.android.internal.util.XmlUtils.writeSetXml(XmlUtils.java:298)
com.android.internal.util.XmlUtils.writeValueXml(XmlUtils.java:447)
com.android.internal.util.XmlUtils.writeMapXml(XmlUtils.java:241)
com.android.internal.util.XmlUtils.writeMapXml(XmlUtils.java:181)
android.app.SharedPreferencesImpl.writeToFile(SharedPreferencesImpl.java:596)
android.app.SharedPreferencesImpl.access$800(SharedPreferencesImpl.java:52)
android.app.SharedPreferencesImpl$2.run(SharedPreferencesImpl.java:511)
java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1112)
java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:587)
java.lang.Thread.run(Thread.java:841)

```

最开始以为是由于使用了`RxJava`导致的多线程读写问题，但后来阅读`SharedPreferenceImpl`代码时发现，有使用`synchronized`来保证线程安全，排除多线程的可能性。

在`HashIterator`的源码中，抛出`ConcurrentModificationException`的片段如下
```
if(HashMap.this.modCount != this.expectedModCount) {
                throw new ConcurrentModificationException();
            } else if(var2 == null) {
                throw new NoSuchElementException();
            } else {
                if((this.next = (this.current = var2).next) == null) {
                    HashMap.Node[] var1 = HashMap.this.table;
                    if(HashMap.this.table != null) {
                        while(this.index < var1.length && (this.next = var1[this.index++]) == null) {
                            ;
                        }
                    }
                }

                return var2;
            }
```

基本情况就是同一`HashMap`或者`HashSet`被多个线程同时修改了，导致集合的大小和预期不一致，抛出了异常。在网上查找原因时，注意到了`SharedPreference`中有个函数：

```
  /**
     * Retrieve a set of String values from the preferences.
     * 
     * <p>Note that you <em>must not</em> modify the set instance returned
     * by this call.  The consistency of the stored data is not guaranteed
     * if you do, nor is your ability to modify the instance at all.
     *
     * @param key The name of the preference to retrieve.
     * @param defValues Values to return if this preference does not exist.
     * 
     * @return Returns the preference values if they exist, or defValues.
     * Throws ClassCastException if there is a preference with this name
     * that is not a Set.
     * 
     * @throws ClassCastException
     */
    @Nullable
    Set<String> getStringSet(String key, @Nullable Set<String> defValues);
    
```
明确提到了不能直接修改返回的`Set`，其实现类`SharedPreferenceImpl`中的代码,直接从`Map`中返回了集合，多个线程同时获取了同一个集合，同时修改肯定会出错:

```
  @Nullable
    public Set<String> getStringSet(String key, @Nullable Set<String> defValues) {
        synchronized (mLock) {
            awaitLoadedLocked();
            Set<String> v = (Set<String>) mMap.get(key);
            return v != null ? v : defValues;
        }
    }
```

#### 解决方案

在使用了`getStringSet`的地方，将返回的`Set`拷贝一份再使用，能够避免这个问题。

##### 参考文章
> https://janatechnology.wordpress.com/2016/01/29/concurrent-modification-exceptions-on-androids-shared-preferences/
