# CESelectThemes
今天我们就来实现一下一些常用资讯类App中常用的分类选择的控件的封装。本篇博客中没有使用到什么新的技术点，如果非得说用到了什么新的技术点的话，那么勉强的说，用到了一些iOS9以后UICollectionView添加的一些新的特性。本篇博客所涉及的技术点主要有UICollectionView的Cell移动，手势识别，控件封装，闭包回调，面向接口编程，Swift中的泛型等等。这些技术点在之前的博客中也多次使用到，只不过本篇博客使用这些技术点来完成我们的具体需求。


### 一、实例运行效果

先入为主，下方这个效果，就是本篇博客所涉及Demo的运行效果。当然下方的效果是一些资讯类App中选择分类时，常用的部分。主要还是对UICollectionView的使用。当然，下方效果的实现，网上也不乏相应的实例。虽然本篇博客的效果与其他类似的效果类似，但是代码设计以及结构实现时还是有所区别的。下方效果的实现使用了iOS9以后的UICollectionView才支持的更新Cell的方法，稍后会详细介绍到。当然，本篇博客我们依然使用Swift3.0来实现的。

在之前的博客中，我们系列的介绍了UICollectionView的各种回调，以及如何自定义CollectionView的布局，并给出了如何使用CollectionView自定义瀑布流。关于之前的博客请移步于[《UICollectionView详解系列》](http://www.cnblogs.com/ludashi/tag/CollectionView/)。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329102847842-911484840.gif)

我们可以使用Reveal来查看上述效果的层级关系。下方就是我们使用Reveal来查看的效果。从下方的效果中我们不难看出，该页面的实现并不复杂。主要还是对UICollectionView的使用。


上面这个效果就是我们今天博客中所实现的效果，而下方这两个效果是我们之前在聊UICollectionView以及自定义布局时所给出的相应的Demo, 下方的Demo所对应的源码也在Gitbub上进行了分享。还是那句话，今天博客的内容依然是对UICollectionView的应用。
![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170331111619055-56039178.png)

UICollectionView这个控件是非常强大的，之所以强大，源于其可定制性比较高，灵活多变。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329104046498-111603048.gif)

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329104041061-1568963571.gif)

### 二、控件的调用

我们将上述分类选择的控件进行了封装，接下来，我们将会给出其初始化和调用的方式。下方就是我们所封装控件的调用方式，下方的二维数组dataSource就是我们所封装控件中的CollectionView中的数据源，该数据源中的数据项要遵循我们指定的CEThemeDataSourceProtocal协议。稍后我们会给出该协议中所以对应的内容。

DataSourceTools类中的createDataSource()类方法就负责创建我们需要的测试数据。数据源创建好后，在实例化CESelectThemeController对象时，将相应的数据源传给我们的控件即可。然后给控件的对象设置更新数据源的闭包回调，也就是说，当我们使用该封装的控件对DataSource操作完毕后，会执行下方的闭包回调，将更新后的数据源传给调用者。如下所示：

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329105642295-1170783850.png)

CEThemeDataSourceProtocal协议就规范了数据源中的数据项必须要实现的方法，下方就是CEThemeDataSourceProtocal协议的实现代码。当然该协议的代码实现比较简单，就一个menuItemName()方法，该方法的返回值是一个字符串。该字符串就是我们要在Cell上显示的Menu的名字。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329110755201-1026497407.png)

下方就是创建我们的数据项的测试数据相关代码。下方的MeteData类就是我们要在上述控件测Cell中显示的数据。该类实现了CEThemeDataSourceProtocal协议，并给出了menuItemName()的方法实现。

在DataSourceTools中的createDataSource()方法中负责创建我们的测试数据，通过循环实例化MeteData并存入二维数组中，并将该二维数据组进行返回。该方法返回的二维数组就是我们需要的数据源。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329111158248-1947445703.png)


### 三、控件核心代码介绍

上面我们简单介绍了该控件的调用方式，接下来我们来看一下该控件的核心代码的实现。说吧了，就是长按手势识别以及CollectionView的Cell的移动。下方我们将详细的介绍一下该控件的核心代码的实现。

 

##### 1. UICollectionViewDataSource

下方就是该控件中使用UICollectionView的DataSource的代理方法。前面几个我们之前介绍过的代理方法就不做过多赘述了，下方两个画框的就是本篇博客的主角，一个是开启Cell移动的代理方法，另一个是移动后更新数据源的方法，具体如下所示。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329112254233-1825738041.png)
　　

 

###### 2、为CollectionView添加长按手势

接下来要做的就是给CollectionView添加LongPressGestureRecognize。addGestureRecognizer()方法负责为我们的CollectionView添加长按手势，longPress()方法就是该长按手势所触发的方法。手势开始时，我们调用longPressBegin()方法。手势改变时，我们调用longPressChange()方法。手势结束时，我们调用longPressEnd()。这三个方法是本篇博客的关键，下方会具体给出其实现方式。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329112907108-26143440.png)
　　

 

###### 3、longPressBegin()方法的实现

下方是长按手势开始时所触发的方法，首先根据触摸的点来获取该点所在cell的IndexPath。如果该Cell不是第一个Section中的Cell, 那么就不触发手势开始的事件，因为我们规定只有第一个Section中的Cell才有长按拖动手势。

如果Cell符合我们的要求，我们就调用UICollectionView的beginInteractiveMovementForItem()方法来启动移动Item功能。当然，该方法是iOS9以后才添加的。启动后我们将当前的Cell隐藏，然后将当前的Cell生成快照，让后让该快照跟着我们的手指移动即可。具体代码如下所示：

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329113337451-598167669.png)
　　

 

###### 4、longPressChange()方法的实现

下方方法就是手指移动时所触发的方法，该方法的代码比较简单，主要是改变我们快照的坐标，让Cell的快照随着手指的移动而移动。然后再调用updateInteractiveMovementTargetPosition()。调用该方法时，会执行DataSource代理中更新数据源的代理方法，也就是上面DataSource代理方法中最后一个更新数据源的方法。

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329142206014-1096764167.png)

　　

 

###### 5、longPressEnd()方法实现

该方法的主要功能是在手势结束后做一些善后工作，如结束移动，然后移除掉Cell的快照并显示隐藏掉的cell。具体如下所示：

![](http://images2015.cnblogs.com/blog/545446/201703/545446-20170329142607748-1872098961.png)
　　

  
