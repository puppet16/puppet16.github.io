---
title: Recyclerview加载图片错位
date: 2020-05-31 21:54:06
tags: [Android, RecyclerView]
summary: Recyclerview加载图片错位
---

<!-- toc -->

# 前言
使用recyclerView时遇到了图片错位的问题，这个问题网上已经讨论的很成熟，谨以此文章做个总结。
# 问题产生原因

**根本原因：** 因为有ViewHolder的重用机制，每一个item在移出屏幕后都会被重新使用以节省资源，避免滑动卡顿。  

**场景A:**  
1. 第一次进入页面，RecyclerView载入，不做任何触摸操作  
2. Adapter经过onCreateViewHolder()创建当前显示给用户的N个ViewHolder对象，并且在onBind时启动了N条线程加载图片  
3. N张图片全部加载完毕，并且显示到对应的ImageView上  
4. 控制屏幕向下滑动，前K个item离开屏幕可视区域，后K个item进入屏幕可视区域  
5. 前K个item被回收，重用到后K个item。后K个item显示的图片是前K个item的图片  
6. 开启了K条线程，加载后K张图片。等待几秒，后K个item显示的图片突然变成了正确的图片

经过细化分析可以看出：如果当前网络速度很快，第6个步骤的加载速度在1秒甚至0.5秒内，就会造成人眼看到的图片闪烁问题，后K个item的图片闪了一下变成了正确的图片。

**场景B:**  
1. 第一次进入页面，RecyclerView载入，不做任何触摸操作  
2. Adapter经过onCreateViewHolder()创建当前显示给用户的N个ViewHolder对象，并且在onBind时启动了N条线程加载图片  
3. 结果N张图片全部加载完毕，并且显示到对应的ImageView上，但还有1张未加载完(假设是第一张图片未加载完)  
4. 控制屏幕向下滑动动，前K个item离开屏幕可视区域，后K个item进入屏幕可视区域  
5. 前K个item被回收，重用到后K个item。**场景A**的问题不再说，后K张图片加载完毕(看上去一切正常)
6. 等待几秒，第一张图片终于加载完成，后K个item中的某一个突然从正确的图片(当前positon应
该显示的图片)变成不正确的图片(第一个item的图片)  

以上过程是场景B，问题出在加载第一张图片的线程T，持有了item1的ImageView对象引用，而这张图片加载速度非常慢，直到item1已经被重用到后面item后，过了一段时间，线程T才把图片一加载出来，并设置到item1的ImageView上，然而线程T并不知道item1已经不存在且已复用成其他item，于是，图片发生错乱了。

**场景C:**  
1. 第一次进入页面，RecyclerView载入，不做任何触摸操作  
2. Adapter经过onCreateViewHolder()创建当前显示给用户的N个ViewHolder对象，并且在onBind时启动了N条线程加载图片  
3. 忽略图片加载情况，直接向下滚动，再向上滚动，再向下滚动，来回操作  
4. 由于离开了屏幕的item是随机被回收并重用的，所以向下滚动时我们假设item1、item3被回收重用到item9、item10，item2、item4被回收重用到item11、item12  
5. 向上滚动时，item9、item12被回收重用到item1、item2，item10、item11被回收重用到item3、item4  
6. 多次上下滚动后，停下，最后发现某一个item的图片在不停变化，最后还不一定是正确的图片  

以上过程是场景C，问题出现在ViewHolder的回收重用顺序是随机的，回收时会从离开屏幕范围的item中随机回收，并分配给新的item，来回操作数次，就会造成有多条加载不同图片的线程，持有同一个item的ImageView对象，造成最后在同一个item上图片变来变去，错乱更加严重。

# 解决方案
## 一、设置占位图

**Glide有两种方法设置占位图**  
### 1、直接在链式请求中加`.placeholder()`：
```
Glide.with(this)
        .load(picUrl)
        .placeholder(R.drawable.ic_loading)
        .into(holder.ivThumb)
```
### 2、添加监听，在回调方法中设置：
```
Glide.with(mContext)
    .load(picUrl)
    .error(R.drawable.ic_loading)
    .into(new SimpleTarget<GlideDrawable>() {
        @Override
        public void onResourceReady(GlideDrawable glideDrawable, GlideAnimation<? super  GlideDrawable> glideAnimation) {
                    holder.ivThumb.setImageDrawable(glideDrawable);
        }

        @Override
        public void onStart() {
            super.onStart();
            holder.ivThumb.setImageResource(R.drawable.ic_loading);
        }
    });
```
## 二、设置TAG

使用setTag（）方式。但是，Glide图片加载也是使用这个方法，所以需要使用setTag（key，value）方式进行设置，这种方式是不错的一种解决方式，注意取值的时候应该是getTag（key）这个方法，当异步请求回来的时候对比下tag是否一样，再判断是否显示图片,我使用的是position设置tag.

**时间及事件梳理**  
开始->当前item1给ImageView打tag  
->当前item1发起网络请求  
->异步处理网络请求  
->当前item1滑出屏幕  
->划出屏幕的item重用并滑进屏幕命名为item2  
->item2重新给ImageView打上tag  
->item1的图片下载完成，因为重用的原因，图片将要加载给item2  
->用原先传入的item1设置的tag和新覆盖的tag比较，发现不相同  
->不给当前item2设置图片，避免了图片错位  


**代码**  
```
@Override
public void onBindViewHolder(final VideoViewHolder holder, final int position) {
    holder.thumbView.setTag(R.id.tag_dynamic_list_thumb, position);
    Glide.with(mContext)
        .load(picUrl)
        .error(R.drawable.video_thumb_loading)
        .into(new SimpleTarget<GlideDrawable>() {
            @Override
            public void onResourceReady(GlideDrawable glideDrawable, GlideAnimation<? super GlideDrawable> glideAnimation {
                if (position != (Integer) holder.thumbView.getTag(R.id.tag_dynamic_list_thumb))
                    return;
                    holder.thumbView.setImageDrawable(glideDrawable);
            }

            @Override
            public void onStart() {
                super.onStart();
                holder.thumbView.setImageResource(R.drawable.ic_loading);
            }
    });
}
```

## 三、在onViewRecycled方法中重置item的ImageView并取消网络请求
  
  ### 流程：在onBindViewHolder中发起加载请求，然后在view被回收时取消网络请求  

**代码**
```
@Override
public void onBindViewHolder(VideoViewHolder holder, int position) {
    String istrurl = mImgList.get(position).getImageUrl();
    if (null == holder || null == istrurl || istrurl.equals("")) {
        return;
    }
    Glide.with(mContext)
        .load(picUrl)
        .placeholder(R.drawable.ic_loading)
        .into(holder.thumbView);
}

@Override
public void onViewRecycled(VideoViewHolder holder) {
    if (holder != null) {
        Glide.clear(holder.thumbView);
        holder.thumbView.setImageResource(R.drawable.ic_loading);

    }
    super.onViewRecycled(holder);
}
```
>参考文章：
[https://blog.csdn.net/lililijunwhy/article/details/79869491](https://blog.csdn.net/lililijunwhy/article/details/79869491)
[https://blog.csdn.net/qq_33808060/article/details/59116624](https://blog.csdn.net/qq_33808060/article/details/59116624)
[https://blog.csdn.net/life90/article/details/78884618](https://blog.csdn.net/life90/article/details/78884618)