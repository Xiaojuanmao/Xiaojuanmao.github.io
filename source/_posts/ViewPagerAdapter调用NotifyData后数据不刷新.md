---
title: ViewPagerAdapter调用NotifyData后数据不刷新
date: 2018-04-12 10:04:02
tags: Android

---

线上存在一个关于PagerAdapter的少量崩溃，描述如下：
> The application's PagerAdapter changed the adapter's contents without calling PagerAdapter#notifyDataSetChanged!

由于在使用`PagerAdapter`的过程中，使用的数据源是从外界传入的，数据源被改变之后未调用`notifyDataSetChanged`函数来刷新并通知`Adapter`，此时滑动`ViewPager`时候会产生上述崩溃。

在`ViewPager`内部的`PagerObserver`中有如下代码，检测数据源的数量是否处于同步状态，并会抛出异常:

```
     if (N != mExpectedAdapterCount) {
            String resName;
            try {
                resName = getResources().getResourceName(getId());
            } catch (Resources.NotFoundException e) {
                resName = Integer.toHexString(getId());
            }
            throw new IllegalStateException("The application's PagerAdapter changed the adapter's"
                    + " contents without calling PagerAdapter#notifyDataSetChanged!"
                    + " Expected adapter item count: " + mExpectedAdapterCount + ", found: " + N
                    + " Pager id: " + resName
                    + " Pager class: " + getClass()
                    + " Problematic adapter: " + mAdapter.getClass());
        }
```

最初的解决办法是按照异常的提示，直接在修改了数据源的地方调用`notifyDataSetChanged()`函数，崩溃消失，出现了数据错位的现象。

之后了解到，`ViewPager`的数据绑定和`RecyclerView`存在不同，`RecyclerView`在数据源变动之后会调用`onBindData`将数据源重新进行绑定，而`ViewPager`会根据`PagerAdapter`中的`getItemPosition`函数来决定是否刷新，默认的实现都是不刷新。

```
 /**
     * Called when the host view is attempting to determine if an item's position
     * has changed. Returns {@link #POSITION_UNCHANGED} if the position of the given
     * item has not changed or {@link #POSITION_NONE} if the item is no longer present
     * in the adapter.
     *
     * <p>The default implementation assumes that items will never
     * change position and always returns {@link #POSITION_UNCHANGED}.
     *
     * @param object Object representing an item, previously returned by a call to
     *               {@link #instantiateItem(View, int)}.
     * @return object's new position index from [0, {@link #getCount()}),
     *         {@link #POSITION_UNCHANGED} if the object's position has not changed,
     *         or {@link #POSITION_NONE} if the item is no longer present.
     */
    public int getItemPosition(@NonNull Object object) {
        return POSITION_UNCHANGED;
    }
```

此函数返回`POSITION_UNCHANGED`则不会进行数据刷新，而`POSITION_NONE`则会重新生成并绑定数据。对于数据源产生混乱的情况，通过控制该函数的返回值即可解决。

最终的解决办法为通过设置`TAG`来判断当前数据源是否和Index匹配，不匹配则重新刷新，匹配则不用处理


##### 参考文章

> https://www.jianshu.com/p/266861496508