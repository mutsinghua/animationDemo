属性动画并不是一个新事物，早在Android3.0的时候，Google就推出了aiminator相关的动画类，这个知识点是非常成熟了，但为什么这里我要拿出来冷饭热炒，的确是因为理解它太重要了，它可以是说后面介绍的包括activity动画，场景动画的基础。如果说场景动画是一幢外表华丽的大楼，则属性动画就是修建这个大楼的基石。

###动画的基本属性
要实现一个动画，我们需要考虑以下几个维度。

 - 时间。这个动画执行多久。
 - 基于时间的位置或变化属性。听起来有点拗口，它其实是在动画执行过程中某个时间点所对应的变量。如果是一个位移动画，则某个时间点它移动的距离。如果是任意的属性，则就是某个点属性变化的变量。
 - 重复的次数以及重复的行为。前者好理解，就是一个动画播放多少次。后者是指，当动画被重复播的时候，它是顺着时间点正着播放，还是逆着时间倒着播放。
 - 帧时，每一帧播放的时间。
 - 复合动画，简单动画在不同变量上的叠加。

以上这几个维度，分别用程序员的语言来说，即是

 - durating
 - interpolator
 - repeat/reverse
 - frame
 - animatorSet

理解了以上的动画基础属性，我们先来实现一个简单的动画，一个View从上到下移动，然后一步步进行扩展。要实现这个动画，我们先定义一个animator.

animator.xml
```
<?xml version="1.0" encoding="utf-8"?>
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:valueFrom="0"
    android:valueTo="300"
    android:valueType="floatType"
    />
```

其中duration是时间，这里的单位是毫秒。valueFrom是值变动的开始，valueTo是值变动的结束，valueType是定义数据的类型。

现在我们需要把它加载进来

```
ValueAnimator move = (ValueAnimator) AnimatorInflater.
                    loadAnimator(this.getApplicationContext(), R.anim.animator);
```

为它增加一个listener，当有值有变动时进行回调。

```
move.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    textView.setY((Float) animation.getAnimatedValue());
                }
            });
```
属性动画的目标是改变属性，这个例子中，我们改变的是一个浮点型数值。当动画在进行中的时候，这个浮点型数据在不断变化，我们可以用animation.getAnimatedValue()来获取当前值，然后，我们这个值给了View的setY方法，于是就实现从0到200的Y方向上的一个变动。
最后再补上animation.start(); 整个动画就跑起来了。

下面我们继续优化。我们知道，Material Design讲究的是动画动起来符合自然原理，目前是匀速作动，看起来非常生硬。我们需要改变他，那么，我们引入了interpolator.


```
<?xml version="1.0" encoding="utf-8"?>
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:valueFrom="0"
    android:valueTo="300" android:valueType="floatType"
    android:interpolator="@android:interpolator/accelerate_cubic"
    /> 
```
interpolator主要是用于修改时间与变化值的函数，根据要实现的效果使用不同的interpolator可达到不同的效果。我们这里使用了一个三次方的加速效果，这样，由慢到快，感觉动画就自然了许多。android为我们内置了很多效果

AccelerateDecelerateInterpolator, 
AccelerateInterpolator, 
AnticipateInterpolator, 
AnticipateOvershootInterpolator, 
BounceInterpolator, 
CycleInterpolator, 
DecelerateInterpolator, 
FastOutLinearInInterpolator, 
FastOutSlowInInterpolator,
 LinearInterpolator, 
OvershootInterpolator, 
PathInterpolator

如果以上效果不满足你，那还有

![这里写图片描述](http://img.blog.csdn.net/20151012221848585)

如果你还得不到满足，那么有第三方的开源库为你服务。
比如：https://github.com/cimi-chen/EaseInterpolator

如果你的胃口比较奇特，就只有自己动手实现。

-----------------------------------番外篇自义定Interpolator 开始-----------------------------------（分隔线)

很简单，你只需要实现Interpolator这个接口就好，这个接口就一个方法

```
float getInterpolation(float input);
```

用数学公式表示就是y=f(x)

你可以是简单的二次方 y = x * x

```
float getInterpolation(float input) {
	return input * input;
}
```
也可以是复杂的阶段函数：


```
public float getInterpolation(float input) {
		if (input < (1 / 2.75))
			return (7.5625f * input * input);
		else if (input < (2 / 2.75))
			return (7.5625f * (input -= (1.5f / 2.75f)) * input + 0.75f);
		else if (input < (2.5 / 2.75))
			return (7.5625f * (input -= (2.25f / 2.75f)) * input + 0.9375f);
		else
			return (7.5625f * (input -= (2.625f / 2.75f)) * input + 0.984375f);
｝
```


-----------------------------------番外篇自义定Interpolator 结束-----------------------------------（分隔线)

我们继续扩展。
	

```
	android:repeatCount="2"
    android:repeatMode="reverse"
```
这两个扩展属性也很好理解。第一个是重复次数，第二个是重复时时否倒着播放，动画效果大家可以自己手动实验。
最终，animator是这个样子。
```
<animator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:valueFrom="0"
    android:valueTo="300"
    android:valueType="floatType"
    android:interpolator="@android:interpolator/accelerate_cubic"
    android:repeatCount="2"
    android:repeatMode="reverse"
    />
```

当然按照代码的灵活性，除了使用XML还可以使用代码来设置。
通常，我们使用静态工厂方法来创建，比如说：

```
	ValueAnimator valueAnimator = ValueAnimator.ofFloat(0.0f,300f);
            valueAnimator.setDuration(1000);
```
ofFloat可以带多个参数，一般说来，是动画的开始参数与结束参数，相当于valueFrom和valueTo。其他方法和XML属性一致，非常简单。


如果故事到这里就结束了，也就体会不出Android的强大。

可以看得出来，上述的动画在使用上不是很方面，于是，我们就引入了一个ValueAnimator的子类ObjectAnimator.

##对象动画

ObjectAnimator的优势是能直接对对象做操作。在前面的例子中，我们在UpdateListener中，不断的调用textview.setY方法进行动画，而使用ObjectAnimator，则连这一步都省了。

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:propertyName="Y"
    android:interpolator="@android:interpolator/accelerate_cubic"
    android:valueFrom="0"
    android:valueTo="300" >

</objectAnimator>
```

加载方式：

```
ObjectAnimator objectAnimator = (ObjectAnimator) AnimatorInflater.loadAnimator(this, R.anim.objanimator);
objectAnimator.setTarget(textView);//设置动画作用的目标对象
objectAnimator.start();
```

重点要说明的是
android:propertyName这个属性。与ValueAnimator不同，ObjectAnimator能直接对属性进行动画，因此这个android:propertyName就是指定的改变某个对象的属性，这里有个必要条件是，这个对象必须具有对应的get/set方法，比如android:propertyName = “Y”，则对象必须要有setY方法，同样，如果写成这样android:propertyName = “alpha”,则对象必定要用setAlpha方法。

使用这个ObjectAnimator能一步到位。

对于用纯代码来写，格式是这样的，同样是一个工厂方法，只不过参数多了。

```
ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(textView, "Y", 0f,300f);
```
第一个参数是目标，第二个参数是属性，接下来是开始值与结束值。


属性动画的入门就到这里，接下来，我们将在属性动画的基础上，做出与IPHONE媲美的特效。

相关的代码见：
https://github.com/mutsinghua/animationDemo

