react native布局
=====
#### CSS属性布局
* 视图边框

borderBottomWidth number 底部边框宽度

borderLeftWidth number 左边框宽度

borderRightWidth number 右边框宽度 

borderTopWidth number 顶部边框宽度 

borderWidth number 边框宽度 

border<Bottom|Left|Right|Top>Color 各方向边框的颜色,<>表示连着一起，例如borderBottomColor

borderColor 边框颜色


#### flex布局
Flex布局又叫弹性布局，会把组件看成一个容器，它的所有子组件都是它容器中的成员，通过Flex，就能迅速布局子成员。

* Flex 主轴和侧轴

> Flex中有两个重要的概念就是：主轴和侧轴

> 主轴和侧轴之间的关系是相互垂直的

> 主轴：决定子组件默认的布局方向：水平、竖直

> 侧轴：决定子组件与主轴垂直的方向

> 比如主轴水平，那么子组件默认水平布局排布，侧轴就是控制子组件在竖直方向上的布局

* flexDirection属性

flexDirection：决定子组件主轴的方向，水平或者竖直
flexDirection共有4个值，在RN中默认是column。

> row:主轴水平方向，从左往右。依次排列
 
> row-reverse:主轴水平方向，从右往左。依次排列

> column:主轴竖直方向，从上往下。依次排列

> column-reverse:主轴竖直方向，从下往上。依次排列

* flexWrap属性

flexWrap决定子控件在父视图类是否允许多行排列。flexWrap共有两个值，默认为nowrap

> nowrap 组件排列在一行，可能导致溢出
 
> wrap 组件在一行显示不下时，会换行

* justifyContent

justifyContent设置主轴子组件具体布局，是靠左，还是居中等。justifyContent共有5个值，默认为flex-start

> flex-start: 子组件向主轴起点对齐，如果主轴水平，从左开始，如果主轴竖直，从上开始。

> flex-end: 子组件向主轴终点对齐，如果主轴水平，从右开始，如果主轴竖直，从下开始。

> center: 居中显示，注意：并不是让某一个子组件居中，而是整体效果居中。

> space-between: 均匀分配，相邻元素间距相同。起点和终点靠边

> space-around: 均匀分配，相邻元素间距相同。起点和终点间距是组件间间距的一半。

* alignItems

alignItems决定子组件侧轴方向上的布局,alignItems共有4个值，默认为stretch

> flex-start: 侧轴方向上起点对齐

> flex-end: 侧轴方向上终点对齐

> center: 子组件侧轴居中

> stretch: 子组件在侧轴方向被拉伸到与容器相同的高度或宽度

注意点：如果指定了宽或者高，这stretch对应的地方不能拉伸，比如指定了高度，这stretch在高度上就是那个指定的值。


* alignSelf
* 
alignSelf：自己定义自己的侧轴布局，用于一个组件设置.当某个子组件不想参照默认的alignItems的时候，可以设置alignSelf，自己定义自己的侧轴布局.alignSelf共有5个值，默认为auto

> auto:继承它的父容器的alignItems属性。如果没有父容器则为 "stretch"

> flex-start:子组件向侧轴起点对齐

> flex-end:子组件向侧轴终点对齐

> center:子组件在侧轴居中

> stretch:子组件在侧轴方向被拉伸到与容器相同的高度或宽度

* flex

flex：决定子控件在主轴中占据几等分

flex：任意数字，所有子控件flex相加，自己flex占总共的多少，就有多少宽度

最后送一张图，更直观形象

[Flexbox 口诀](https://weibo.com/1712131295/CoRnElNkZ?ref=collection&type=comment#_loginLayer_1517633271976)