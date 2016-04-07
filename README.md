# Android-Notes

一些android 日常学习总结，用于查漏补缺



### 自定义View


##### 1.Custom ViewGroup Example/自定义ViewGroup

[google自定义ViewGroup例子](http://developer.android.com/intl/zh-tw/reference/android/view/ViewGroup.html)
（1）想要使用自己的layoutparams（比如ChildView中需要设置自定义属性）,需要override一些方法：generateLayoutParams generateDefaultLayoutParams generateLayoutParams checkLayoutParams
（2）自定义属性的声明一些技巧
使用android 自己的属性：
```xml
<attr name="android:layout_gravity" />
```
使用有取值范围的属性：
```xml
<attr name="layout_position">
    <enum name="middle" value="0" />
    <enum name="left" value="1" />
    <enum name="right" value="2" />
</attr>
```

##### 2.View/ViewGroup scrollTo,scrollBy

绝对不是自己想象的那样。现象如下：
View：比如一个按钮调用scrollTo时本身还是在原地，只是里面的内容scroll。
ViewGroup：一个含有多个按钮（所有内容）的view group调用scrollTo时，包含的所有内容会移动。
负值代表着向下，向右移动（在view group是这样的）这些都只是表象！！！

原因：这么说吧 ，[世界本是无边无界的，可是我们的眼睛我们的心约束了我们所看到的“世界”](http://blog.csdn.net/qinjuning/article/details/7247126) 。
可以这样看，就好比View/ViewGroup只不过是一个窗口（所以Button,LinearLayout等只不过是一个窗口），用来看自己下面压着的画布，
（每个窗口都有一个画布，所以Button,LinearLayout等都只能看到自己的画布，而ViewGroup（LinearLayout等）的画布上嵌入着自己包含的Button等（View/ViewGroup）窗口！！！）
而scrollTo只会改变每个画布与窗口的相对位置，而不会改变画布中内容的绝对位置！！！
注意：上面所说的画布、窗口的概念可能与android中的概念有出入，只是为了易于理解。

##### 3.Scroller的简单使用

第一次见到Scroller类是在看Gallery源码时看到的。当时为了实现换页效果，只是利用反射调用了它的movetoNext的方法（Android 6.0 更换了方法）。
今天为了完全消化[race604的下拉刷新控件FlyRefresh](https://github.com/race604/FlyRefresh)，
再次查找关于Scroller的资料。
最终找到了[Android中滑屏实现----手把手教你如何实现触摸滑屏以及Scroller类详解](http://blog.csdn.net/qinjuning/article/details/7419207)才理解
（1）原来Scroller只不过是一个工具类，一个计算器，用于计算滑动动画进行的进度。它本身不会更新View的状态。就像第一次使用Interaptor时一样就得好神奇啊。
（2）View的状态更新是在自己的一个方法computeScroll中调用scrollTo,scrollBy来完成的。
（3）View的computeScroll调用时机是在
```java
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
    ...
    child.computeScroll();
    ...
}
```
Scroller的简单使用：
> 第一、调用Scroller实例去产生一个偏移控制(对应于startScroll()方法) ，并手动调用invalid()方法去重新绘制
> 第二、剩下的就是在computeScroll()里根据当前已经逝去的时间，获取当前应该偏移的坐标(由Scroller实例对应的computeScrollOffset()计算而得)
> 第三、当前应该偏移的坐标，调用scrollBy()方法去缓慢移动至该坐标处。

Scroller建议用ScrollerCompat代替，而且还有Scroller Fling的用法


### RecycleView的简单使用


#####  1.LayoutManager

LayoutManager是RecycleView的布局管理类
（1）简单的竖直排列：`new LinearLayoutManager(this);`默认的布局为竖直排列。
（2）水平排列：仍使用使用上面的Manager，`mLayoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);`
（3）Grid布局：`mLayoutManager = new GridLayoutManager(context,columNum);`columNum为grid的列数
（4）瀑布流布局：使用StaggeredGridLayoutManager

#####  2.Adapter

Adapter是适配器类，讲数据源与视图结合在一起的类，同时也可作为数据源的容器。
使用Adapter时一般需要自己实现一个：`public class MyAdapter extends RecyclerView.Adapter<ViewHolder> {}`
实现Adapter时需要override一些方法：
（1）`@Override public ViewHolder onCreateViewHolder(ViewGroup viewGroup, int viewType)`创建新View，被LayoutManager所调用，此方法根据viewType,创建相应的ViewHolder.
（2）`@Override public void onBindViewHolder(ViewHolder viewHolder, int position)`将数据与界面进行绑定的操作
（3）`@Override public int getItemCount()`获取数据的数量

#####  3.ViewHolder

ViewHolder的作用避免多次findViewById,也是自己实现一个：`public class ViewHolder extends RecyclerView.ViewHolder{}`实现ViewHolder时需要override其构造方法，在构造方法中findViewById.

#####  4.点击事件

ListView有OnItemClickListener,然而RecycleView没有相应方法。网上有两种实现方法：
第一种方法为：http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2014/1118/2004.html
（1.1）使Adapter实现onClickListener方法：
`public class MyAdapter extends RecyclerView.Adapter<MyAdapter.ViewHolder> implements View.OnClickListener {}`
（1.2）在Adapter中onCreateViewHolder方法中setOnClickListener：
```java
public ViewHolder onCreateViewHolder(ViewGroup viewGroup, final int i) {
  View view=LayoutInflater.from(viewGroup.getContext()).inflate(R.layout.item, viewGroup, false);
  ...
  view.setOnClickListener(this);//将创建的View注册点击事件
  ...
}
```

（1.3）在Adapter中onBindViewHolder方法中还需要setTag将数据与视图绑定在一起：
```java
@Override
    public void onBindViewHolder(ViewHolder viewHolder, final int i) {
        viewHolder.mTextView.setText(datas.get(i).title);
        //将数据保存在itemView的Tag中，以便点击时进行获取
        viewHolder.itemView.setTag(datas.get(i));
    }
```

第二种方法是将ViewHolder实现OnClickListener:
```java
public static class ViewHolder extends RecyclerView.ViewHolder implements View.OnClickListener{
        public TextView mTextView;
        public ViewHolder(View itemView) {
            super(itemView);
            mTextView = ((TextView) itemView.findViewById(R.id.text));
            itemView.setOnClickListener(this);
        }

        @Override
        public void onClick(View v) {
            int adapter_position = getAdapterPosition();
            int layout_position = getLayoutPosition();
            Toast.makeText(v.getContext(),"adapter position :"+adapter_position+", layout postion :"+layout_position,Toast.LENGTH_SHORT).show();
        }
}
```
