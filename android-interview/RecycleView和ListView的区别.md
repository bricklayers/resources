# RecycleView和ListView的区别

## 使用上

**ListView**
继承重写BaseAdaoter类
自定义ViewHolder和convertView一起完成复用优化工作

**RecycleView**
继承重写RecycleView.Adapater和RecycleView.ViewHolder
设置LayoutManager 布局管理器控制布局效果

**RecycleView** 相比 **ListView**在基础上主要有几点：
ViewHolder的编写更规范了
RecycleView复用Item的工作已经内置了， 不需要像ListView那样设置setTag
RecycleView需要多出一步布局管理器LayoutManager的设置工作

## 布局效果

**ListView** 比较单一，只支持线性列表布局

**RecycleView** 布局丰富可通过设置LayoutManager实现线性布局、网格布局和瀑布流效果

## 空数据处理

ListView提供了setEmptyView这个api来处理Adapter中数据为空的情况

```java
listView.setEmptyView(findViewById(R.id.lv_empty_layout));//设置内容为空时显示的视图
```

RecycleView并没有提供此类API

## 数据刷新

ListView中提供了 notifyDataSetChanged()， 当需要更新数据源， 通过Adapter的notifyDataSetChanged来通知视图更新变化时，所有的Item都重新绘制了

RecycleView.Adapter则提供了notifyItemChanged用于简单更新单个ItemView的刷新

## 缓存机制

RecyclerView比ListView多两级缓存，支持多个离ItemView缓存，支持开发者自定义缓存处理逻辑，支持所有RecyclerView共用同一个RecyclerViewPool缓存池

ListView

```java
class RecycleBin
    View[] : mActiveViews
    ArrayList<View>[] : mScrapViews
```

|              | 是否需要回调createView | 是否需要回调bindView |                        生命周期                         |             描述             |
| ------------ | :--------------------: | :------------------: | :-----------------------------------------------------: | :--------------------------: |
| mActiveViews |           否           |          否          |                  onLayout函数周期调用                   | 用于屏幕内ItemView的快速复用 |
| mScrapViews  |           否           |          是          | 与mAdapter一致，当mAdapter被更换时，mScrapViews即被清空 |   用于屏幕外ItemView的缓存   |

RecycleView

```java
class Recycler
    ArrayList<ViewHolder> : mAttachedScrap
    ArrayList<ViewHolder> : mCachedViews
    ViewCacheExtension : mViewCacheExtension
    RecycledViewPool : mRecyclerPool
```

|                     | 是否需要回调createView | 是否需要回调bindView |                           生命周期                           |                            描述                             |
| ------------------- | :--------------------: | :------------------: | :----------------------------------------------------------: | :---------------------------------------------------------: |
| mAttachedScrap      |           否           |          否          |                     onLayout函数周期调用                     |                用于屏幕内ItemView的快速复用                 |
| mCachedViews        |           否           |          否          | 与mAdapter一致，当mAdapter被更换时，mScrapViews即被缓存至mRecyclerPool |            默认上限为2，即缓存屏幕外2个ItemView             |
| mViewCacheExtension |                        |                      |                                                              |            不直接使用，需要用户定制，默认不实现             |
| mRecyclerPool       |           否           |          是          |           与自身生命周期一致，不再被引用时即被释放           | 默认上限为5，支持所有RecyclerView共用同一个RecyclerViewPool |

ListView和RecyclerView缓存机制基本一致：

1). mActiveViews和mAttachedScrap功能相似，意义在于快速重用屏幕上可见的列表项ItemView，而不需要重新createView和bindView

2). mScrapView和mCachedViews + mReyclerViewPool功能相似，意义在于缓存离开屏幕的ItemView，目的是让即将进入屏幕的ItemView重用

3). RecyclerView的优势在于a.mCacheViews的使用，可以做到屏幕外的列表项ItemView进入屏幕内时也无须bindView快速重用；b.mRecyclerPool可以供多个RecyclerView共同使用，在特定场景下，如viewpaper+多个列表页下有优势.客观来说，RecyclerView在特定场景下对ListView的缓存机制做了补强和完善

## 使用场景

- 只是少量列表的展示滑动等，ListView和RecyclerView都能很好地工作，两者并没有很大的差异

- 数据源频繁更新的场景，如弹幕：Recyclerview实现的弹幕 （旧）等RecyclerView的优势会非常明显；
- 进一步来讲，结论是：
  **列表页展示界面，需要支持动画，或者频繁更新，局部刷新，建议使用RecyclerView，更加强大完善，易扩展；其它情况(如微信卡包列表页)两者都OK，但ListView在使用上会更加方便，快捷。**

## 知识点

- RecycleView支持多个不同类型的布局，他们是怎么缓存并且怎么查找的？

  ```java
  class RecycledViewPool {
  	static class ScrapData {
  		final ArrayList<ViewHolder> mScrapHeap = new ArrayList<>();
  	}
      SparseArray<ScrapData> mScrap = new SparseArray<>();
      
      // 通过 viewType 设置不同类型的集合
      mScrap.put(viewType, scrapData);
      
      // 根据 viewType 查找
      ScrapData scrapData = mScrap.get(viewType);
      ArrayList<ViewHolder> scrapHeap = scrapData.mScrapHeap;
  }
  ```

## 参考资料

[1]: https://blog.csdn.net/weixin_47285644/article/details/114588301	"ListView和RecycleView对比"
[2]: https://zhuanlan.zhihu.com/p/23339185	"Android ListView 与 RecyclerView 对比浅析—缓存机制"

