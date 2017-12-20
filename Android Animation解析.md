                                 
                                                  ##
                                                  Android动画
                                                  
在app中增加一些动画效果能使我们的app更加绚丽，本文主要总结android中的Animation的使用

Android中的Animation可以分为三种，View Animation,Drawable Animation ,Property Animation,其中第一种比较简单，只能实现一些简单的平移，缩放，旋转，透明度渐变的基本动画效果，第二种可以实现一种逐帧动画效果，而第三种属性动画比较复杂，Android 3.0之后出现，view动画能实现的它都能实现，除此之外还能实现很多其他的动画效果。 
 
####1.View动画，它的实现方式有两种，XML方式和javacode方式
XML方式：
* 新建项目，在res目录中新建anim文件夹
* 在anim目录中新建动画代码
* 载入XML动画效果

直接上代码
透明度渐变动画

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
      <!--alpha渐变透明度动画
       fromAlpha 代表起始时透明度
       toAlpha 代表终止时透明度
       duration 动画持续事件 毫秒为单位-->
    <alpha
        android:fromAlpha="0.0"
        android:toAlpha="1.0"
        android:duration="1000"/>
    </set>

旋转动画

      <?xml version="1.0" encoding="utf-8"?>
      <set xmlns:android="http://schemas.android.com/apk/res/android">
        <!--旋转动画
       interpolator 动画插值器，跟伸缩动画一样
       fromDegress 动画起始时物件的角度
       toDegress 动画结束时物件旋转的角度，当角度为负数表示逆时针旋转，当角度为正数表示顺时针旋转
                    (负数from——to正数:顺时针旋转)
                    (负数from——to负数:逆时针旋转)
                    (正数from——to正数:顺时针旋转)
                    (正数from——to负数:逆时针旋转)
       pivotX,pivotY表示旋转的x，y坐标，0%-100%取值。50%表示中点
       repeatMode代表重复类型：重复类型有两个值，reverse表示倒序回放，restart表示从头播放
       repeatCount代表重复次数，重复次数是int型
       -->
       <rotate
        android:duration="3000"
        android:fromDegrees="0"
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toDegrees="+350"
        android:repeatMode="restart"
        android:repeatCount="6"
        />
    </set>
尺寸伸缩动画

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
    <!--渐变尺寸伸缩动画
       duration 渐变时间
       interpolator 指定一个动画的插入器
       有三种动画的插入器
       accelerate_decelerate_interpolator 加速-减速动画插入器
       accelerate_interpolator 加速动画插入器
       decelerate_interpolator 减速动画插入器
        fromXScale 动画起始时x坐标的伸缩尺寸
        toXScale 动画结束时x坐标的伸缩尺寸

        fromYScale 动画起始时Y坐标的伸缩尺寸
        toYScale 动画结束时Y坐标的伸缩尺寸
        0.0表示没有，1.0表示正常无收缩，值小于1.0表示收缩，值大于1.0表示放大
        pivotX 属性为动画相对于物件的X坐标的开始位置
        pivotY 属性为动画相对于物件的Y坐标的开始位置
        从0%-100%开始取值， 50%表示物件的X或者Y方向坐标上的中点位置
        fillAfter 当设置为true，该动画转化在动画结束后被应用-->
      <scale
        android:duration="1000"
        android:fillAfter="false"
        android:fromXScale="0.0"
        android:fromYScale="0.0"
        android:toXScale="1.4"
        android:toYScale="1.4"
        android:pivotX="50%"
        android:pivotY="50%"
        android:interpolator="@android:anim/accelerate_decelerate_interpolator"/>
    </set>

位移动画

    <?xml version="1.0" encoding="utf-8"?>
    <set xmlns:android="http://schemas.android.com/apk/res/android">
     <!--位置转移动画
       fromXDelta 动画开始时X坐标位置
       fromYDelta 动画开始时Y坐标位置
       toXDelta 动画结束时X坐标位置
       toYDelta 动画结束时Y坐标位置
       -->
    <translate
        android:duration="2000"
        android:fromXDelta="30"
        android:fromYDelta="30"
        android:toXDelta="-80"
        android:toYDelta="300"
        />
    </set>
载入XML动画效果

    switch (v.getId()) {
            case R.id.bt_alpha:
                //载入动画
                mAnimation = AnimationUtils.loadAnimation(this, R.anim.alpha_anim);
                mIv.startAnimation(mAnimation);
                break;
            case R.id.bt_scale:
                //载入动画
                mAnimation = AnimationUtils.loadAnimation(this, R.anim.scale_anim);
                mIv.startAnimation(mAnimation);
                break;
            case R.id.bt_translate:
                //载入动画
                mAnimation = AnimationUtils.loadAnimation(this, R.anim.translate_anim);
                mIv.startAnimation(mAnimation);
                break;
            case R.id.bt_rotate:
                //载入动画
                mAnimation = AnimationUtils.loadAnimation(this, R.anim.rotate_anim);
                mIv.startAnimation(mAnimation);
                break;
            default:
                break;
        }
和点击事件一样，动画也具有监听，只需要我们的activity实现AnimationListener接口就行，我们可以在动画开始或者结束或者重复做一些别的操作

    @Override
    public void onAnimationStart(Animation animation) {//动画开始
    }

    @Override
    public void onAnimationEnd(Animation animation) {//动画结束
    }

    @Override
    public void onAnimationRepeat(Animation animation) {//动画重复
    }
此外动画还有其他的属性，比如比较常用的android:repeatCount（XML中定义,表示重复次数），android:repeatMode（表示重复模式，重复类型有两个值，reverse表示倒序回放，restart表示从头播放），android:interpolator（设置差值器，这个在属性动画中会详细讲解）

上面讲了View动画的XML实现方式，下面介绍javacode实现方式，javacode的实现方式也很简单（不需要新建XML）

     switch (v.getId()) {
            case R.id.bt_alpha:
                mAnimation = new AlphaAnimation(0.1f, 0.1f);//新建一个透明度变化的动画
                mAnimation.setDuration(2000);//设置动画事件
                mIv.startAnimation(mAnimation);//开始动画
                break;
            case R.id.bt_scale:
                //渐变尺寸缩放动画效果
                mAnimation = new ScaleAnimation(0.0f, 2.0f, 1.5f, 1.5f, Animation.RELATIVE_TO_PARENT, 0.5f, Animation.RELATIVE_TO_PARENT, 0.0f);
                mAnimation.setDuration(2000);
                mIv.startAnimation(mAnimation);
                break;
            case R.id.bt_translate:
                //移动动画效果
                mAnimation = new TranslateAnimation(0, 100, 0, 100);
                mAnimation.setDuration(2000);
                mIv.startAnimation(mAnimation);
                break;
            case R.id.bt_rotate:
                //旋转动画效果，这里是旋转360°
                mAnimation = new RotateAnimation(0.0f, 360.0f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
                mAnimation.setDuration(2000);
                mIv.startAnimation(mAnimation);
                break;
            default:
                break;

看效果
![View动画2.gif](http://upload-images.jianshu.io/upload_images/4669002-80b0b1e788eae77d.gif?imageMogr2/auto-orient/strip)

####2.Drawable Animation动画，它是把一帧一帧拼起来组成动画
*  在res/drawable目录添加图片素材

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/4669002-33addca9a2f7e74c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 在drawable文件夹中添加动画帧布局文件 

        <?xml version="1.0" encoding="utf-8"?>
        <!--
            根标签为animation-list，其中oneshot代表着是否只展示一遍，设置为false会不停的循环播放动画  
            根标签下，通过item标签对动画中的每一个图片进行声明  
            android:duration 表示展示所用的该图片的时间长度  
        -->
        <animation-list xmlns:android="http://schemas.android.com/apk/res/android"
            android:oneshot="false" >
            <item
        android:drawable="@drawable/cmmusic_progress_1"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_2"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_3"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_4"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_5"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_6"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_7"
        android:duration="150">
            </item>
            <item
        android:drawable="@drawable/cmmusic_progress_8"
        android:duration="150">
            </item>
        </animation-list>
引入XML布局文件实现动画

       //给ImageView设置drawable
        imgPic.setImageResource(R.drawable.loading_anim);
        //给动画资源赋值
        animationDrawable = (AnimationDrawable) imgPic.getDrawable();
      animationDrawable.start();//开始动画
       animationDrawable.stop(); //停止动画
实现效果类似于下拉加载的菊花转

####3.属性动画
之所以引入属性动画有三点原因：
* 属性动画不仅可以对view操作，也能对非view的对象进行操作
* 属性动画不单单具有View动画的那些基本动画效果，也具有其他动画效果
* View动画只是改变了View的显示效果而已，而不会真正去改变View的属性。而属性动画不单单是一种视觉上的动画，它是真正的改变了view的位置

属性动画实际上是一种不断地对值进行操作的机制，并将值赋值到指定对象的指定属性上，可以是任意对象的任意属性。所以我们仍然可以将一个View进行移动或者缩放，但同时也可以对自定义View中的Point对象进行动画操作了。我们只需要告诉系统动画的运行时长，需要执行哪种类型的动画，以及动画的初始值和结束值，剩下的工作就可以全部交给系统去完成了。

看完上面这段话还是不理解属性动画，那我们用起来就可以了
属性动画有三个类特别重要，分别是ValueAnimator，ObjectAnimator，AnimatorSet，前两个都是动画的执行类，用来执行属性动画的，而且ObjectAnimator是集成ValueAnimator的，而AnimatorSet顾名思义是动画的集合，用它来实现组合动画，我们来分别认识这三个类
#####VlaueAnimator
这个是属性动画最核心的类了，前面已经提到，属性动画实际上是一种不断对值进行操作的机制，并将值赋值到指定对象的指定属性上，而初始值和结束值的动画过渡就是这个类来负责计算的，我们只需要将初始值和结束值提供给ValueAnimator，并且告诉它动画所需运行的时长，那么ValueAnimator就会自动帮我们完成从初始值平滑地过渡到结束值这样的效果。除此之外，ValueAnimator还负责管理动画的播放次数、播放模式、以及对动画设置监听器等。

使用ValueAnimator

    ValueAnimator anim = ValueAnimator.ofFloat(0f, 1f);  //值从0平滑到1，除此之外还可以用ofInt方法
    anim.setDuration(300);  //动画持续时间
    anim.start(); //动画开始
但是这段代码看不到任何界面效果，我们可以添加一个监听器方法，在动画执行过程中用不断进行回调

    anim.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {  
    @Override  
    public void onAnimationUpdate(ValueAnimator animation) {  
        float currentValue = (float) animation.getAnimatedValue();  
        Log.d("TAG", "cuurent value is " + currentValue);  
    }  
    }); 
可以通过日志看到动画确实在执行

#####ObjectAnimator
ValueAnimator只是实现值的平滑过渡，使用场景并不是很多，而ObjectAnimator可以对任意对象的任意属性进行操作，一般我们是对view进行操作，因为ObjectAnimator是继承ValueAnimator，所以在ValueAnimator使用的方法在ObjectAnimator中也可以正常使用，直接上代码

            float curTrans = mIv.getTranslationX();//得到
            ObjectAnimator animator = ObjectAnimator.ofFloat(mIv, "translationX", curTrans,-500 ,curTrans);
            animator.setDuration(5000);
            animator.start();
上面的代码实现了平移的操作，其他例如透明度变化，旋转和伸缩都类似，不同的是ofFloat里面的参数发生变化
* ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "alpha", 1f, 0f, 1f);  //透明度变化
* ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "rotation", 0f, 360f);  //旋转
* ObjectAnimator animator = ObjectAnimator.ofFloat(textview, "scaleY", 1f, 3f, 1f); 缩放

第一个参数是view，第二个参数是动画类型，后面的参数根据动画类型的不同参数也不一样，关于第二个类型，虽然view里面没有alpha或者rotation这些属性，但是有get和set方法，其实ObjectAnimator内部的工作机制并不是直接对我们传入的属性名进行操作的，而是会去寻找这个属性名对应的get和set方法。

#####AnimatorSet
AnimatorSet是用来实现组合动画的，比如我们要进行一组组合动画，先进行位移操作，再进行旋转操作，再进行淡入淡出的操作，代码如下：

    ObjectAnimator moveIn = ObjectAnimator.ofFloat(textview, "translationX", -500f, 0f);  //位移动画
    ObjectAnimator rotate = ObjectAnimator.ofFloat(textview, "rotation", 0f, 360f);  //旋转动画
    ObjectAnimator fadeInOut = ObjectAnimator.ofFloat(textview, "alpha", 1f, 0f, 1f);  //淡入淡出动画
    AnimatorSet animSet = new AnimatorSet();  //新建一个AnimatorSet对象
    animSet.play(rotate).with(fadeInOut).after(moveIn);  //AnimatorSet有一个play方法，动画执行顺序
    animSet.setDuration(5000);  //动画持续时间
    animSet.start();  //动画开始
