---
title: Android Knowledge&Interview
tags:
---


# Android 基础知识复习

## Activity

### Activity生命周期

![image](https://camo.githubusercontent.com/908f0c391fd50377aebbb3c0fd1a0c4419007c0eecb7651a1908be8e273a46ab/687474703a2f2f6769747975616e2e636f6d2f696d616765732f6c6966656379636c652f61637469766974792e706e67)


Activity A 启动另一个Activity B，回调如下:
Activity A 的onPause() → Activity B的onCreate() → onStart() → onResume() → Activity A的onStop()；如果B是透明主题又或则是个DialogActivity，则不会回调A的onStop；


### Activity的启动模式和Android的任务栈分析

####　四种启动模式：Standard，SingleTop，SingleTask,SingleInstance
1. Standard: 默认启动模式当目标Activity在AndroidManifest中为Standard类型时，不管当前的任务栈中是否已经存在该Activity的实例，都重新生成一个Activity的实例，并推入任务栈栈顶。
2. SingleTop: 当目标Activity被声明为SingleTop时，如目标不存在，则新建一个activity放在栈顶；如果目标已经存在，并且非处于任务栈顶，则重新生成一个目标Activity放在栈顶；如果目标已经存在，并处于栈顶，则调用该Activity的onNewIntent方法。
3. SingleTask: 当目标Activity被声明为SingleTask的时候，如果说目标不存在，则新建一个Activity放在栈顶，并且新建一个目标Activity放在栈中；如果目标存在于当前栈中，则将当前栈中位于目标之上的Activity全部都出栈并销毁（onDestory方法会调用），并调用目标Activity的onNewIntent方法。
4. SingleInstance： 当目标Activity被声明为SingleInstance的时候，如果说目标不存在，则新建一个Task，并把新建的目标Activity压入栈中；singleInstance的Activity始终单独使用一个任务栈；如果目标存在，则调用该Activity的onNewIntent方法。
注意：
因为 singleInstance 的属性是禁止与其他 Activities 共享任务栈，所以启动模式为 SingleInstance 的 Activity 启动其他 Activity 时会默认带有 FLAG_ACTIVITY_NEW_TASK 属性。所以从一个launchMode为singleInstance的activity中新启动一个启动模式为Standard的 Activity F 后， Activity F会被推到SingleInstance之前的那个标准任务栈的栈顶。


#### taskAffinity
什么是affinity？
　　affinity是指Activity的归属，Activity与Task的吸附关系，也就是该Activity属于哪个Task。一般情况下在同一个应用中，启动的Activity都在同一个Task中，它们在该Task中度过自己的生命。每个Activity都有taskAffinity属性，这个属性指出了它希望进入的Task。如果一个Activity没有显式的指明taskAffinity，那么它的这个属性就等于Application指明的taskAffinity，如果Application也没有指明，那么该taskAffinity的值就等于应用的包名。我们可以通过在元素中增加taskAffinity属性来为某一个Activity指定单独的affinity。这个属性的值是一个字符串，可以指定为任意字符串，但是必须至少包含一个”.”，否则会报错。

affinity在什么场合应用呢？
1.根据affinity重新为Activity选择宿主task（与allowTaskReparenting属性配合使用）
　　allowTaskReparenting用来标记Activity能否从启动的Task移动到taskAffinity指定的Task，当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在Task的能力。具体点来说，就是一个Activity现在是处于某个Task当中的，但是它与另外一个Task具有相同的affinity值，那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。allowTaskReparenting默认是继承至application中的allowTaskReparenting=false，如果为true，则表示可以更换；false表示不可以。
　　举一个形象点的例子，比如有一个天气预报程序，它有一个用于显示天气信息的Activity，allowTaskReparenting属性设置成true，这个Activity和天气预报程序的所有其它Activity具体相同的affinity值。这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，这个Activity会被转移到天气预报程序的任务当中，并显示出来。如果将你自己的应用切换到前台，发现你自己应用Task里的那个Activity消失了。

2.启动一个Activity过程中Intent使用了FLAG_ACTIVITY_NEW_TASK标记，根据affinity查找或创建一个新的具有对应affinity的task。
　　当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，如果在Intent中加入了FLAG_ACTIVITY_NEW_TASK flag的话，情况就会变的复杂起来。首先，系统会去检查这个Activity的affinity是否与当前Task的affinity相同。如果相同的话就会把它放入到当前Task当中，如果不同则会先去检查是否已经有一个名字与该Activity的affinity相同的Task,如果有，这个Task将被调到前台，同时这个Activity将显示在这个Task的顶端；如果没有的话，系统将会尝试为这个Activity创建一个新的Task。需要注意的是，如果一个Activity在manifest文件中声明的启动模式是”singleTask”，那么他被启动的时候，行为模式会和前面提到的指定FLAG_ACTIVITY_NEW_TASK一样。
　　那么，有了上面的知识，我们应该可以实现开头提到的功能了。


 ####　IntentFlags

 FLAG_ACTIVITY_BROUGHT_TO_FRONT

        比方说我现在有Ａ，在Ａ中启动Ｂ，在Ａ中Intent中加上这个标记。此时B就是以FLAG_ACTIVITY_BROUGHT_TO_FRONT 这个启动的，在B中再启动C，D（正常启动C，D），如果这个时候在D中再启动B，这个时候最后的栈的情况是 A,C,D,B。

FLAG_ACTIVITY_REORDER_TO_FRONT

        如果在Intent中设置，并传递给Context.startActivity()，这个标志将引发已经运行的Activity移动到历史stack的顶端。 例如，假设一个Task由四个Activity组成：A，B，C，D。如果D调用startActivity()来启动Activity B，那么，B会移动到历史stack的顶端，现在的次序变成A，C，D，B。如果FLAG_ACTIVITY_CLEAR_TOP标志也设置的话，那么这个标志将被覆盖。

FLAG_ACTIVITY_CLEAR_TASK

        如果在调用startActivity时传递这个标记，该task栈中的其他activity会先被清空，然后该activity在该task中启动，也就是说，这个新启动的activity变为了这个空task的根activity。所有老的activity都结束掉。该标志必须和FLAG_ACTIVITY_NEW_TASK一起使用。

FLAG_ACTIVITY_CLEAR_TOP

        如果该activity已经在task中存在，并且设置了该task，系统不会启动新的 Activity 实例，会将task栈里该Activity之上的所有Activity一律结束掉，然后将Intent发给这个已存在的Activity。Activity收到 Intent之后，或者在onNewIntent()里做下一步的处理，或者自行结束然后重新创建。如 Activity 在 AndroidMainifest.xml 里将启动模式设置成默认standard模式，且 Intent 里也没有设置 FLAG_ACTIVITY_SINGLE_TOP，那么Activity将会结束并且重启；否则则会传递到onNewIntent方法。

已经启动了四个Activity：A，B，C和D。在D Activity里，我们要跳到B Activity，同时希望C finish掉，可以在startActivity(intent)里的intent里添加flags标记，这样启动B Activity，就会把D，C都finished掉，如果你的B Activity的启动模式是默认的（multiple） ，则B Activity会finished掉，再启动一个新的Activity B。  如果不想重新再创建一个新的B Activity，则可在启动Intent添加flag FLAG_ACTIVITY_SINGLE_TOP。

        可以利用此特性来退出程序，假设A为程序入口，将A的Manifest.xml配置成android:launchMode="singleTop"，通过此flag来启动A，同时在A的onNewIntent中判断来结束自己，可到退出的效果。FLAG_ACTIVITY_CLEAR_TOP 还可以和 FLAG_ACTIVITY_NEW_TASK 配合使用，用来启动一个task栈的根activity，他将会把该栈清空为根状态，比如从notification manager启动activity。

FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

        设置完之后，新的activity将不会添加到当前activity列表中，当某些情况下我们不希望用户通过历史列表回到我们的Activity的时候这个标记比较有用。他等同于在XML中指定Activity的属性android:excludeFromRecents=”true”

FLAG_ACTIVITY_FORWARD_RESULT

        如果A需要onActivityResult中获取返回结果，startActivityForResult B，而B只是过渡页，启动C之后就finish掉了，需要在 C 中setResult返回给A就可以用到这个标志。

A -> B -> XXXXX(无论多少个过渡页) 设置 FLAG_ACTIVITY_FORWARD_RESULT 来启动 C ，之后该XXX过渡页finish - > C ，那么C的结果返回给A。

FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY

        如果设置，新的Activity将不再历史stack中保留。用户一离开它，这个Activity就关闭了。例如A启动B的时候，给B设置了FLAG_ACTIVITY_LAUNCHED_FROM_HISTORY，那么：

A -> B -> C ，启动C 就算 B没有自行finish ，也会变为 AC

FLAG_ACTIVITY_MULTIPLE_TASK

        这个标识用来创建一个新的task栈，并且在里面启动新的activity（所有情况，不管系统中存在不存在该activity实例），经常和FLAG_ACTIVITY_NEW_DOCUMENT或者FLAG_ACTIVITY_NEW_TASK一起使用。这上面两种使用场景下，如果没有带上FLAG_ACTIVITY_MULTIPLE_TASK标识，他们都会使系统搜索存在的task栈，去寻找匹配intent的一个activity，如果没有找到就会去新建一个task栈；但是当和FLAG_ACTIVITY_MULTIPLE_TASK一起使用的时候，这两种场景都会跳过搜索这步操作无条件的创建一个新的task。和FLAG_ACTIVITY_NEW_TASK一起使用需要注意，尽量不要使用该组合除非你完成了自己的顶部应用启动器，他们的组合使用会禁用已经存在的task栈回到前台的功能。

FLAG_ACTIVITY_NEW_DOCUMENT

        api 21之后加入的一个标识，用来在intent启动的activity的task栈中打开一个document，和documentLaunchMode效果相等，有着不同的documents的activity的多个实例，将会出现在最近的task列表中。

documentLaunchMode可以设置4个值

intoExisting： activity 会为该document请求一个已经存在的task，这与设置FLAG_ACTIVITY_NEW_DOCUMENT且不设置FLAG_ACTIVITY_MULTIPLE_TASK 有相同的效果。

always： activity 会为该document创建一个新的task，即使该document已经被打开了，这与设置 FLAG_ACTIVITY_NEW_DOCUMENT且设置 FLAG_ACTIVITY_MULTIPLE_TASK 有相同的效果。

none：activity 不会为 document 创建新的task，该app被设置为 single task 的模式，它会重新调用用户唤醒的所有activity中的最近的一个。

never：activity 不会为document创建一个新的task，设置这个值复写了 FLAG_ACTIVITY_NEW_DOCUMENT 和 FLAG_ACTIVITY_MULTIPLE_TASK 标签。如果其中一个标签被设置，并且overview screen 显示该app为 single task 模式。则该activity会重新调用用户最近唤醒的activity。

注意： none 或 nerver 使用时，activity必须设置为 launchMode=”standard” ，如果该属性没有设置，documentLaunchMode=”none” 属性就会被使用。有

FLAG_ACTIVITY_RETAIN_IN_RECENTS

        api21加入。默认情况下通过FLAG_ACTIVITY_NEW_DOCUMENT启动的activity在关闭之后，task中的记录会相对应的删除。如果为了能够重新启动这个activity你想保留它，就可以使用者个flag，最近的记录将会保留在接口中以便用户去重新启动。接受该flag的activity可以使用autoRemoveFromRecents去复写这个request或者调用Activity.finishAndRemoveTask()方法。

FLAG_ACTIVITY_NEW_TASK

        设置此状态，记住以下原则，首先会查找是否存在和被启动的Activity具有相同的亲和性的任务栈（即taskAffinity，注意同一个应用程序中的activity的亲和性在没有修改的情况下是一样的，所以下面的a情况会在同一个栈中），如果有，刚直接把这个栈整体移动到前台，并保持栈中的状态不变，即栈中的activity顺序不变，如果没有，则新建一个栈来存放被启动的activity。

a. 前提: Activity A和Activity B在同一个应用中。

操作: Activity A启动开僻Task堆栈(堆栈状态：A)，在Activity A中启动Activity B， 启动Activity B的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Activity B被压入Activity A所在堆栈(堆栈状态：AB)。

原因: 默认情况下同一个应用中的所有Activity拥有相同的关系(taskAffinity)。

b. 前提: Activity A在名称为”TaskOne应用”的应用中， Activity C和Activity D在名称为”TaskTwo应用”的应用中。

操作1:在Launcher中单击“TaskOne应用”图标，Activity A启动开僻Task堆栈，命名为TaskA(TaskA堆栈状态: A)，在Activity A中启动Activity C， 启动Activity C的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Android系统会为Activity C开僻一个新的Task，命名为TaskB(TaskB堆栈状态: C), 长按Home键，选择TaskA，Activity A回到前台, 再次启动Activity C（两种情况：1.从桌面启动；2.从Activity A启动，两种情况一样）， 这时TaskB回到前台, Activity C显示，供用户使用, 即：包含FLAG_ACTIVITY_NEW_TASK的Intent启动Activity的Task正在运行，则不会为该Activity创建新的Task，而是将原有的Task返回到前台显示。

操作2:在Launcher中单击”TaskOne应用”图标，Activity A启动开僻Task堆栈，命名为TaskA(TaskA堆栈状态: A)，在Activity A中启动Activity C，启动Activity C的Intent的Flag设为FLAG_ACTIVITY_NEW_TASK，Android系统会为Activity C开僻一个新的Task，命名为TaskB(TaskB堆栈状态: C)， 在Activity C中启动Activity D(TaskB的状态: CD) 长按Home键， 选择TaskA，Activity A回到前台， 再次启动Activity C(从桌面或者ActivityA启动，也是一样的)，这时TaskB回到前台, Activity D显示，供用户使用。说明了在此种情况下设置FLAG_ACTIVITY_NEW_TASK后，会先查找是不是有Activity C存在的栈，根据亲和性(taskAffinity)，如果有，刚直接把这个栈整体移动到前台，并保持栈中的状态不变，即栈中的顺序不变。

FLAG_ACTIVITY_NO_ANIMATION

        启动的时候不执行动画。

FLAG_ACTIVITY_NO_USER_ACTION

        禁止activity调用onUserLeaveHint()函。onUserLeaveHint()作为activity周期的一部分，它在activity因为用户要跳转到别的activity而退到background时使用。比如，在用户按下Home键（用户的操作），它将被调用。比如有电话进来（不属于用户的操作），它就不会被调用。注意：通过调用finish()时该activity销毁时不会调用该函数。

FLAG_ACTIVITY_PREVIOUS_IS_TOP

        如果给Intent对象设置了这个标记，这个Intent对象被用于从一个存在的Activity中启动一个新的Activity，那么新的这个Activity不能用于接受发送给顶层activity的intent，这个新的activity的前一个activity被作为顶部activity。

FLAG_ACTIVITY_TASK_ON_HOME

        api11加入。把当前新启动的任务置于Home任务之上，也就是按back键从这个任务返回的时候会回到home，即使这个不是他们最后看见的activity，注意这个标记必须和FLAG_ACTIVITY_NEW_TASK一起使用。

FLAG_EXCLUDE_STOPPED_PACKAGES和FLAG_INCLUDE_STOPPED_PACKAGES

        从Android 3.1开始，给Intent定义了两个新的Flag，分别为FLAG_INCLUDE_STOPPED_PACKAGES和FLAG_EXCLUDE_STOPPED_PACKAGES，用来控制Intent是否要对处于停止状态的App起作用，顾名思义：

FLAG_INCLUDE_STOPPED_PACKAGES：表示包含未启动的App

FLAG_EXCLUDE_STOPPED_PACKAGES：表示不包含未启动的App

值得注意的是，Android 3.1开始，系统向所有Intent的广播添加了FLAG_EXCLUDE_STOPPED_PACKAGES标志。这样做是为了防止广播无意或不必要地开启未启动App的后台服务。如果要强制调起未启动的App，后台服务或应用程序可以通过向广播Intent添加FLAG_INCLUDE_STOPPED_PACKAGES标志来唤醒。




### 常用Android View的使用

1. Views（TextView,ImageView,EditText,Button,Switch,CheckBox,RadioGroup,RadioButton,ToggleButton,WebView,ProgressBar,SeekBar）
2. ViewGroup(FrameLayout,RelativeLayout,ConstraintLayout,LinearLayout,GridLayout,ScrollView,ListView,TabHost)

ConstraintLayout 优势：
	1). 所见即所得。易于在AndroidStudio中进行配置
	2). 和RelativeLayout一样，可以有效减少布局层数，从而降低绘制复杂度
	3). 和RelativeLayout相比，其布局方式更灵活，更容易适配多种机型。

ConstraintLayout的使用分析：
各种属性的使用：
https://www.jianshu.com/p/17ec9bd6ca8a
https://www.jianshu.com/p/38ee0aa654a8

ConstraintSet的使用：
ConstraintSet主要是方便在代码中修改ConstraintLayout的各种约束属性。
使用：
http://hulkyang.blogspot.com/2018/12/android-constraintlayout-constraintset.html


###　RecyclerView 使用
https://www.jianshu.com/p/4f9591291365

RecyclerView的特点：
1. 默认实现的多级缓存机制
2. 灵活的布局规则，有很强的可配置、可定制性，可实现多行多列，可多向滚动的List
3. 通过LayoutManager，Adapter，ViewHolder，实现了布局和逻辑的解耦
4. 可以局部刷新，并且可以通过ItemAnimation实现更丰富的动画效果


RecyclerView的4级缓存机制：
RecylerView的多级缓存机制，是有RecyclerView中的内部类Recycler实现的。
其中最重要的方法：
getViewForPosition(int position)->getViewForPosition(int position, boolean dryRun) -> tryGetViewHolderForPositionByDeadline(int position,
                boolean dryRun, long deadlineNs) 

核心逻辑实现于tryGetViewHolderForPositionByDeadline中，其中从上个方法传递过来的参数中的dealineNs为Int.MaxValue.

第一步：判断是否处于isPreLayout状态，这个状态值只有在执行item动画的时候才会为true。如果是的话，则调用getChangedScrapViewForPosition（）方法获取viewHolder并返回

第二歩(核心步骤)：在isPreLayout=false的情况下，正常获取一个ViewHolder，需要先按照位置从scrap或者是hiden的或者是cachedView中找。如果说没有找到的话，就按照id和itemType在scrap和缓存中找；如果说没有找到合适的VH，那就判定是否有mViewCacheExtension的定义，有就从mViewCacheExtension中根据position和type找；如果还是没有找到合适的VH，下一个步骤就是在recyclerPool中根据type找

第三歩：如果都没有找到缓存的VH，则需要通过createViewHolder回调Adapter中的方法，进行创建ViewHolder。如果说缓存的VH的话判断是否需要绑定，需要的话就调用onBindViewHolder方法



### View 触摸事件传递机制


在应用中传递顺序：

>1. Activity  dispatchTouchEvent

				| 
>2. PhoneWindow的superDispatchTouchEvent()方法

				|
>3. DocerView的superDispatchTouchEvent()方法 调用super.dispatchTouchEvent

				|
>4. ViewGroup dispatchTouchEvent DocerView继承自ViewGroup，so调用ViewGroup的dispatch方法(划重点：该方法完成主要的传递逻辑)

				|
>5. ViewGroup中首先交由自己的onInterpretTouchEvent进行处理，如果说onInterpretTouchEvent返回true，那么事件被自身拦截，不会继续向下传递。如果不处理，则交由合适的子View进行处理，如果子view仍然是viewGroup则继续按照规则向下分发。如果没有子view则交由自身的touchEvent处理，如果都返回的false。touch事件将不再继续向下分发，而回到上一级的ViewGroup

 				|
>6. View 的dispatchTouchEvent中如果返回true，则交由该View进行处理，如果返回false，则继续由父View 进行处理。同理依次向上推导，直到回到Activity的dispatchTouchEvent方法。
https://blog.csdn.net/fengluoye2012/article/details/83782042

### View KeyEvent传递机制
和触摸事件的传递大同小异。相比较而言，KeyEvent事件会更加简单一些，因为触摸事件有ActionDown，ActionUp，中间还有若干次的ActionMove。

### View滚动事件分析，边际检测

### ViewPager使用和实现原理分析

### Android Jetpack
Room:官方数据库Orm框架
LifeCycle： 构建生命周期感知型组件，这些组件可以根据 Activity 或 Fragment 的当前生命周期状态调整行为
LiveData： 构建可被观察的数据结构，可在数据发生变动时，主线程通知观察者，不会发生内存泄露
WorkManager：满足您的后台调度需求
Navigation：管理应用流程
ViewModels： 帮助应用实现更好的MVVM架构
DataBinding： 主要是基于依赖注入实现配置化UI的框架，对比框架有ViewBindings，GreenOrm,Dagger2
ConstraintLayout: 主要是可以减少布局层数，实现更灵活的页面布局




### 线程间通信方式有几种
1. 线程间通讯，首先可以通过静态共享变量的方式，同一个静态变量，a线程也操作，b线程也操作，完全没有问题，如果不想出现同步问题，加锁就好了。
2. 使用Handler，A线程一个Handler，B线程一个Handler，两个线程通过各自的Handler给对方发消息，完全没有问题。但是注意，在使用Handler的时候要在Handler的run方法中，各自调用Looper.prepare,和Looper.run方法。
3. 通过各种事件总线，在线程中订阅对方的消息，也是可以的，但是感觉有点大材小用了。

扩展一下，Android中常常讨论的工作线程和主线程通信的方式：
在一个Android的应用的进程中，一共只有一个主线程，既MainThread。MainThread负责回调Activity的生命周期方法，以及绘制界面等操作，所以异常的重要。其他非重要的，或者是耗时较长，耗用内存较多的工作，尽量放在其他线程中去做，做完后再回到主线程上反馈结果。

和主线程通信的方式：
1. AsyncTask，一套完整的异步任务模板，适合完成一个单纯的可能需要和页面产生交互的任务，定制一个异步任务，只需要继承他，并且在对应的模板方法中写好UI逻辑和工作线程的逻辑就好。缺点是比较死板，还有就是和某个界面非常耦合，因为任务执行前和执行后的逻辑大多数都和具体的页面的元素有关，所以如果不是特别通用的逻辑，最好是直接写在Activity 中比较好，因此，比较难以通用化，流程化。
2. 通过Handler，在子线程中，传入主线程的Handler（注意引用应该为弱引用，这样才不会在一个Activity完成之后导致无法回收的风险），然后从工作线程中给主线程postMessage。
3. Looper.mainLooper(),获取到主线程的Looper，然后定义一个Handler（把主线程的Looper传入进去），然后在需要通信的地方，使用handler.postRunable()方法让Runable中的run方法执行在主线程之上。
4. 通过本地广播的方式，（非本地广播也是可以的，但是消耗较大，需要占用系统AMS中任务队列的资源），注意，本地广播的实现原理也是基于3的。


### 为什么在Looper中while true 取消息不会阻塞主线程，Looper，MessageQueue，Handler的关系

Looper中的loop方法中看似是死循环的方法里面，为什么不会主线程呢，原因有一，主线程和进程周期息息相关，如果进程被杀死了，这里的主线程必然就退出了，第二点是在loop方法中执行的方法，是从MessageQueue队列中取出消息，该方法调用了native方法 nativePollOnce（），这个方式是基于Linux的epoll机制，简而来说，就是通过文件阻塞住线程的执行，当有消息进来的时候，这个方法就会从阻塞状态恢复，取出消息，继续执行。所以，主线程随着进程一直执行，并不会阻塞。


Looper内部维护一个消息队列，他与当前声明他的线程绑定（ThreadLocal中保存着Looper本身，通过ThreadLocal锁，保证Looper和线程的绑定），Looper的作用，就是在调用loop方法之后，持续不断的从MessageQueue中取出消息。MessageQueue中的消息从哪儿来呢，从Handler中来，Handler中，必须要有一个Looper对象，如果没有，在使用Handler进行操作的时候，就会报错 the looper must call loop before you post message（类似）。。。所以通过Handler进行postMessage或者是postRunnalbe本质上都是通过looper向looper的消息队列中push Message对象。然后在loop方法中处理。


### 为什么不能在工作线程直接刷新UI
因为会报错吧！

刷新UI，会回调View的invalidate方法，而这个方法中，会检测线程是否为主线程，通过Looper吧，如果不是主线程，就报错。

但是，据说，有一种方法可以欺骗住invalidate方法，但是我觉得意义不大。



### Android 自定义View和ViewGroup需要实现的方法

View： onMeasure ，draw方法， onConfigChange方法

ViewGroup： onMeasure，onLayout onConfigChange方法需要按照需求去定制


###　Android draw， onDraw ，onLayout，方法的区别和联系
onLayout方法，主要是ViewGroup在绘制的时候，用来控制如何布局子view的位置和大小
draw方法： 主要是用来给View自定义的时候，去定义如何绘制。
而onDraw方法：我用到它主要是在做ViewGroup自定义的时候，去定制ViewGroup上的一些绘制操作。因为ViewGroup的draw方法其实可能会有很多继承的方法，尽量不要去动。


### Android 绘图相关 Paint 几种颜色混合模式
就是两种图形，第一次绘制的是dest，叠加上去的是src，两次绘制图形的交集、并集和补集，以及颜色叠加的时候用哪一种，透明度用谁的，或者是混合或是加强的巴拉巴拉这种。一共多达10来种变化。这种方式在绘制圆角的时候，可以避免使用clippath产生的毛边。

### Android Animation类型


1. Tween动画（补间动画）： 就是你提供几个值和动画类型，以及动画市场和动画插值器，然后根据时长和插值器，自动计算动画的幅度的动画。
   1. View动画：传统动画，都是你定义个动画（长度，类型，插值器）通过AnimationUtils加载xml定义好的动画，然后调用view的playAnimation方法
   2. Object动画：更灵活，然后可以直接定义到view的属性上去，非常的方便好用。
      1. Property动画：
      2. Value动画：

2. 帧动画：一堆图片叠起来，一张一张播

### 如何优化图片加载
1. 对图片进行裁剪以及压缩，加载合适大小的图片
2. 使用缓存，在图片加载数量较多的时候，重复利用对象。
3. 在从网络上下载图片的时候，使用holder图片。
4. 如果绘制图片，不要求太高质量，无透明度，使用rgb565，比使用rgb8888，节省一半左右的空间
5. 减少过度绘制，如果可以的话，只绘制变化的区域，不要大面积绘制。



### 实现一个简单的LruCache

LruCache中使用的数据结构为一个LinkedHashMap，它允许传入一个参数，就是在插入后调整链表的顺序，把刚操作的这个元素放在链表的头部，这样如果说队列满了，就开始优先移除掉最近最少使用的对象。
https://www.cnblogs.com/clnchanpin/p/6854999.html

```code=java
package com.knowledgeStudy.lrucache;
import java.util.ArrayList;
import java.util.Collection;
import java.util.LinkedHashMap;
import java.util.Map;
/**
 * 固定大小 的LRUCache<br>
 * 线程安全
 **/
public class LRUCache<K, V> {
    private static final float factor = 0.75f;//扩容因子
    private Map<K, V> map; //数据存储容器
    private int cacheSize;//缓存大小
    public LRUCache(int cacheSize) {
        this.cacheSize = cacheSize;
        int capacity = (int) Math.ceil(cacheSize / factor) + 1;
        map = new LinkedHashMap<K, V>(capacity, factor, true) {
            private static final long serialVersionUID = 1L;
            /**
             * 重写LinkedHashMap的removeEldestEntry()固定table中链表的长度
             **/
            @Override
            protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
                boolean todel = size() > LRUCache.this.cacheSize;
                return todel;
            }
        };
    }
    /**
     * 依据key获取value
     *
     * @param key
     * @return value
     **/
    public synchronized V get(K key) {
        return map.get(key);
    }
    /**
     * put一个key-value
     *
     * @param key
     *            value
     **/
    public synchronized void put(K key, V value) {
        map.put(key, value);
    }
    /**
     * 依据key来删除一个缓存
     *
     * @param key
     **/
    public synchronized void remove(K key) {
        map.remove(key);
    }
    /**
     * 清空缓存
     **/
    public synchronized void clear() {
        map.clear();
    }
    /**
     * 已经使用缓存的大小
     **/
    public synchronized int cacheSize() {
        return map.size();
    }
    /**
     * 获取缓存中全部的键值对
     **/
    public synchronized Collection<Map.Entry<K, V>> getAll() {
        return new ArrayList<Map.Entry<K, V>>(map.entrySet());
    }
}
```





### Android中新增的几种数据类型
SparseArray,ArrayMap,以及LongSparseArray等是Android为了更适合嵌入式设备优化的map。

1. HashMap：Key和Value都需要自动装箱存储，并且，为了支持java的Iterator接口，HashMap额外还存储了EntrySet对象。所以对于会增加内存消耗，在数据量比较多的适合比较明显。HashMap的存入和读取时间复杂度为O（1）。

2. ArrayMap： Key和Value都需要装箱，但是，在内部实现的时候，没有存储额外的对象，内部通过一个Object[]用来存储key和Value，key和value交叉存储，另外通过一个 int数组存储key的hashcode。这样ArrayMap的存入和读取的时间复杂度都是O（logN）

   * Key/Value会被自动装箱。
   * key会存储在mArray[]的下一个可用的位置。而value会存储在mArray[]中key的下一个位置。（key和value在mArray中交叉存储）
   * key的哈希值会被计算出来并存储在mHashed[]中。
   * 当查找一个key的时候：计算key的hashcode。在mHashes[]中对这个hashcode进行二分法查找。也就意味着时间复杂度增加到了O(logN)
   * 一旦我们得到了这个哈希值的位置index。我们就知道这个key是在mArray的2index的位置，而value则在2index+1的位置。
   这个ArrayMap还是没能解决自动装箱的问题。当put一对键值对进入的时候，它们只接受Object，但是我们相对于HashMap来说每一次put会少创建一个对象(HashMapEntry)。这是不是值得我们用O(1)的查找复杂度来换呢？对于大多数app应用来说是值得的。

SparseArray
SparseArray，存储的是key为int类型的map，其内部实现上使用两个数组，一个是int类型的key数组，一个是Object[]类型的Value数组。
当保存一对键值对的时候：

key（不是它的hashcode）保存在mKeys[]的下一个可用的位置上。所以不会再对key自动装箱了。
value保存在mValues[]的下一个位置上，value还是要自动装箱的，如果它是基本类型。
查找的时候：
   * 查找key还是用的二分法查找。也就是说它的时间复杂度还是O(logN) 知道了key的index，也就可以用key的index来从mValues中检索出value。
   * 相较于HashMap,我们舍弃了Entry和Object类型的key,放弃了HashCode并依赖于二分法查找。在添加和删除操作的时候有更好的性能开销。
其他类型的如LongSparseArray，StringSparseArray都和SparseArray类似，只不过key的数组类型换成了对应的基本数据类型。


引申：在数据结构和算法的设计上，常常要权衡时间复杂度和空间复杂度。需要根据具体情况具体分析，按照常理来说，HashMap的设计非常优秀，通过hash算法，把随机存取的复杂度降低到O（N）但是，他的自动装箱和数据冗余，在目前的移动设备上，常常都是内存较为金贵，而cpu的算力越来越高。所以SparseArray的这种以时间换空间的策略就更加适合移动端设备。

## service 
### Android Service相关知识
1. 生命周期。Service类型，两种方式 start&bind区别 。
   startService的情况：

   onCreate-onStartCommond-onStart-onStop-onDestory 

   而bindService的情况：
   onCreate-onBind-onUnBind-onDestory
   其中 onDestory是在所有binder都unbind之后自动调用

2. Service里面弹出Dialog的方式
需要设置Dialog的类型为SystemDialog，但是需要有系统权限类似于Alert_System_Dialog这种的才可以成功


## 广播
1.广播的类型
发送广播的类型：
	1. 普通广播： 
	2. 系统广播： Android定义好的，由系统应用以及系统服务所发出来的广播，如Wifi信号变化，联网，网络断开，时间日期变化等等的广播
	3. 	有序广播：OrderedBroadcast，这种广播，会按照接受者所定义的优先级，以及注册的顺序，依次发送，并且优先接收到广播的接受者，可以选择拦截广播，对广播进行修改，导致广播不再发送给下一个接受者。
	4. 粘滞广播： StickBroadcast，这种广播，在android3.1后被系统标记为deprecated，这种广播是如果定义了接收器，但是在发送广播的时候，接收器所在进程还没有被实例化，则在接受者实例化之后，接收到最后一次发出来的粘滞广播。
	5. 本地广播： LocalBroadcastManager。应用内广播。内部使用Handler实现。

广播接受者的类型：
按照使用方式，分为：
	1. 静态注册：

		```
<receiver android:enabled=["true" | "false"]
android:exported=["true" | "false"]
android:icon="drawable resource"
android:label="string resource"
android:name="string"
android:permission="string"
android:process="string" >
. . .
</receiver>
		```

其中，需要注意的属性
android:exported  ——此broadcastReceiver能否接收其他App的发出的广播，这个属性默认值有点意思，其默认值是由receiver中有无intent-filter决定的，如果有intent-filter，默认值为true，否则为false。（同样的，activity/service中的此属性默认值一样遵循此规则）同时，需要注意的是，这个值的设定是以application或者application user id为界的，而非进程为界（一个应用中可能含有多个进程）；
android:name  —— 此broadcastReceiver类名；
android:permission  ——如果设置，具有相应权限的广播发送方发送的广播才能被此broadcastReceiver所接收；
android:process  ——broadcastReceiver运行所处的进程。默认为app的进程。可以指定独立的进程（Android四大基本组件都可以通过此属性指定自己的独立进程）


4.不同注册方式的广播接收器回调onReceive(context, intent)中的context具体类型

1).对于静态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是ReceiverRestrictedContext；

2).对于全局广播的动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Activity Context；

3).对于通过LocalBroadcastManager动态注册的ContextReceiver，回调onReceive(context, intent)中的context具体指的是Application Context。

注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册的ContextReceiver才有可能接收到（静态注册或其他方式动态注册的ContextReceiver是接收不到的）。

参考：
https://www.cnblogs.com/lwbqqyumidi/p/4168017.html


	2. 动态注册:
2.Broadcast ANR
由于广播接收者的onReceive方法都是在主线程，所以如果在onReceive方法中有耗时操作，会导致应用ANR。尤其是`动态注册的广播`，属于是前台广播，使用的context对象是具体的ActivityContext，所以尤为要注意。

## ContentProvider
1. ContentProvider中使用的Context，

2. ContentProvider的生命周期
ContentProvider主要是为程序提供统一的数据访问接口，隐藏了数据存储的细节，数据可以是存储在内存，外部文件，数据库，以及网络中的数据，访问端只需要知道provider的uri以及访问权限，就可以以统一的方式访问本应用，以及跨应用访问数据。而不需要考虑其他所有问题。

参考：
https://www.jianshu.com/p/c70ae80cf64d

ContentProvider的线程：ContentProvider的onCreate方法是运行在主线程，使用的Context是Application的mainThread，所以要注意不要做耗时操作，如果使用数据库，也尽量不要在这个方法里面去调用getReadableDatabase或者是getWriteableDatabase方法（耗时），参考：

`
/**
     * Implement this to initialize your content provider on startup.
     * This method is called for all registered content providers on the
     * application main thread at application launch time.  It must not perform
     * lengthy operations, or application startup will be delayed.
     *
     * <p>You should defer nontrivial initialization (such as opening,
     * upgrading, and scanning databases) until the content provider is used
     * (via {@link #query}, {@link #insert}, etc).  Deferred initialization
     * keeps application startup fast, avoids unnecessary work if the provider
     * turns out not to be needed, and stops database errors (such as a full
     * disk) from halting application launch.
     *
     * <p>If you use SQLite, {@link android.database.sqlite.SQLiteOpenHelper}
     * is a helpful utility class that makes it easy to manage databases,
     * and will automatically defer opening until first use.  If you do use
     * SQLiteOpenHelper, make sure to avoid calling
     * {@link android.database.sqlite.SQLiteOpenHelper#getReadableDatabase} or
     * {@link android.database.sqlite.SQLiteOpenHelper#getWritableDatabase}
     * from this method.  (Instead, override
     * {@link android.database.sqlite.SQLiteOpenHelper#onOpen} to initialize the
     * database when it is first opened.)
     *
     * @return true if the provider was successfully loaded, false otherwise
	   public abstract boolean onCreate();
`

一个应用中，在使用Binder通信的时候，最多有16个Binder线程位于Binder的线程池中，顾最多有16个调用ContentResolver的进程（线程）。可通过实验验证。






## Android 常用的系统服务

### 1. AMS

### 2. PackageManagerService

### 3. WindowManagerService

### 4. InputManagerService

### 5. NotificationService

### 6. AccountManagerService

### 7. PowerManagerService


## Android 存储系统和存储优化相关内容

## AndroidStudio相关

###　AndroidStudio三方包管理工具maven

### AndroidStudio InstanceRun实现原理

## 简历上可能涉及到的内容

###　RN的原理

### Android应用 开发跨平台的几种流行方式（Native，Hybird，H5 +WebView单页 ）

### Kotlin相关知识

## Kotlin的一些优势分析

## Kotlin 跨线程方案分析

## Kotlin 一些语法特征

# Java基础知识复习

## Java5，Java6，Java7，Java8-11各版本feature

## Java 语法

### 数据类型，类接口和枚举

### java collection api相关 List,Map,Table,Tree,Queue,Stack

### Java HashMap，Arraylist分析，

### Java concorrent api 和多线程通信 线程池原理

### Java ThreadLocal原理



# Android Framework相关知识复习

## Android Binder 机制
### Binder 原理（上层，Linux层， Kernel层）

### Binder 的应用

## Android 跨进程通信的其他方式

## Android 系统启动流程分析
首先，系统系统是从system/core/init程序进入的，init程序，加载各种init.rc，对各种底层驱动服务进行初始化操作，系统启动的第一个应用是system_server是从加载init.zegote_xx.rc中定义的
```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc
    socket zygote stream 660 root system
    onrestart write /sys/android_power/request_state wake
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks

```

调用后的代码位于frameworks/base/cmds/app_process/app_main.cpp，编译成的shell脚本，执行的方法如下：

```
int main(int argc, char* const argv[])
{
    ...
    //AppRuntime定义于app_main.cpp中，继承自AndroidRuntime
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));
    // Process command line arguments
    // ignore argv[0]
    argc--;
    argv++;
    ...
    // Parse runtime arguments. Stop at first unrecognized option.
    // 通过app_main可以启动zygote、system-server及普通apk进程
    // 这个可以通过init.rc来配置
    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    // app_process的名称改为zygote
    String8 niceName;
    // 启动apk进程时，对应的类名
    String8 className;

    ++i;  // Skip unused "parent dir" argument.
    //开始解析输入参数
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            //init.zygote32.rc中定义了该字段，表示启动zygote进程
            zygote = true;
            //记录app_process进程名的nice name，即zygote32(平台相关)
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            //init.zygote.rc中定义了该字段， 启动zygote后会启动system-server
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            //表示启动制定进程
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            //可以自己指定进程名
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            //与--application配置，启动指定的类
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }
    //准备参数
    Vector<String8> args;
    if (!className.isEmpty()) {
        //启动普通进程
        // We're not in zygote mode, the only argument we need to pass
        // to RuntimeInit is the application argument.
        //
        // The Remainder of args get passed to startup class main(). Make
        // copies of them before we overwrite them with the process name.
        args.add(application ? String8("application") : String8("tool"));
        runtime.setClassNameAndArgs(className, argc - i, argv + i);

        if (!LOG_NDEBUG) {
          String8 restOfArgs;
          char* const* argv_new = argv + i;
          int argc_new = argc - i;
          for (int k = 0; k < argc_new; ++k) {
            restOfArgs.append("\"");
            restOfArgs.append(argv_new[k]);
            restOfArgs.append("\" ");
          }
          ALOGV("Class name = %s, args = %s", className.string(), restOfArgs.string());
        }
    } else {
        //创建dalvikCache所需的目录，并定义权限
        // We're in zygote mode.
        maybeCreateDalvikCache();

        if (startSystemServer) {
            //增加参数, 默认启动zygote后，就会启动system server
            args.add(String8("start-system-server"));
        }
        //获取平台对应的abi信息
        char prop[PROP_VALUE_MAX];
        if (property_get(ABI_LIST_PROPERTY, prop, NULL) == 0) {
            LOG_ALWAYS_FATAL("app_process: Unable to determine ABI list from property %s.",
                ABI_LIST_PROPERTY);
            return 11;
        }
        //参数需要制定abi
        String8 abiFlag("--abi-list=");
        abiFlag.append(prop);
        args.add(abiFlag);

        // In zygote mode, pass all remaining arguments to the zygote
        // main() method.
        for (; i < argc; ++i) {
            //将main函数未处理的参数都递交给zygote main处理
            args.add(String8(argv[i]));
        }
    }

    if (!niceName.isEmpty()) {
        //将app_process的进程名，替换为nice name
        runtime.setArgv0(niceName.string(), true /* setProcName */);
    }

    if (zygote) {
        //调用Runtime的start函数, 启动ZygoteInit
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        //启动zygote没有进入这个分支
        //但这个分支说明，通过配置init.rc文件，其实是可以不通过zygote来启动一个进程
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied.\n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}
```
system/core/init
```
init.cpp
init.rc
service.cpp
builtins.cpp
frameworks/base/cmds/app_process/app_main.cpp
frameworks/base/core/jni/AndroidRuntime.cpp
frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
ZygoteServer.java
Zygote.java
```

流程如下：

1. 解析init.zygote.rc中的参数，创建AppRuntime并调用AppRuntime.start()方法；
2. 调用AndroidRuntime的startVM()方法创建虚拟机，再调用startReg()注册JNI函数；
3. 通过JNI方式调用ZygoteInit.main()，第一次进入Java世界；
4. registerZygoteSocket()建立socket通道，zygote作为通信的服务端，用于响应客户端请求；
5. preload()预加载通用类、drawable和color资源、openGL以及共享库以及WebView，用于提高app启动效率；
6. zygote完毕大部分工作，接下来再通过forkSystemServer()，fork得力帮手system_server进程，也是上层framework的运行载体。
7. zygote功成身退，调用runSelectLoop()，随时待命，当接收到请求创建新进程请求时立即唤醒并执行相应工作。


SystemServer分析：
SystemServer是第一个运行的java程序
其中主要业务流程如下：
1. 各种准备工作，启动navtive系统服务
2. 声明一个SystemContext，其中会初始化ActivityThread，ActivityThread会初始化ApplicationThread，并实例化Application，调用Applicaiton的onCreate方法。
3. 如果是system_server服务，会实例化SystemServiceManager
4.启动各种系统服务，其中包括：
	4.1 启动binder线程池，这是SystemServer与其他进程通信的基础
	初始化Looper
	创建了SystemServiceManager对象，它会启动Android中的各种服务。包括AMS、PMS、WMS
	启动桌面进程，这样才能让用户见到手机的界面。
	开启loop循环，开启消息循环，SystemServer进程一直运行，保障其他应用程序的正常运行。

5. 各种准备活动结束后，回调ams的systemReady方法，ams的systemReady执行完后会自动调用startHomeActivityLocked（），启动Launcher。




## Android 应用启动分析
一般来说，冷启动包括了以下内容：

启动进程
点击图标发生在Launcher应用的进程，startActivity()函数最终是由Instrumentation通过Android的Binder跨进程通信机制 发送消息给 system_server 进程；
在 system_server 中，启动进程的操作由ActivityManagerService 通过 socket 通信告知 Zygote 进程 fork 子进程（app进程）
开启主线程
app 进程启动后，首先是实例化 ActivityThread，并执行其main()函数：创建 ApplicationThread，Looper，Handler 对象，并开启主线程消息循环Looper.loop()。
创建并初始化 Application和Activity
ActivityThread的main()调用 ActivityThread#attach(false)方法进行 Binder 通信，通知system_server进程执行 ActivityManagerService#attachApplication(mAppThread)方法，用于初始化Application和Activity。
在system_server进程中，ActivityManagerService#attachApplication(mAppThread)里依次初始化了Application和Activity，分别有2个关键函数：
- thread#bindApplication()方法通知主线程Handler 创建 Application 对象、绑定 Context 、执行 Application#onCreate() 生命周期
- mStackSupervisor#attachApplicationLocked()方法中调用 ActivityThread#ApplicationThread#scheduleLaunchActivity()方法，进而通过主线程Handler消息通知创建 Activity 对象，然后再调用 mInstrumentation#callActivityOnCreate()执行 Activity#onCreate() 生命周期
布局&绘制
源码流程可以参考Android View 的绘制流程分析及其源码调用追踪

至此，应用启动流程完成。


## Android （JVM虚拟机）内存泄露的原理

## Android ANR的定义

1、发生原因
一句话总结：没有在规定的时间内，干完要干的事情，就会发生ANR。

2、ANR分类
从发生的场景分类：

Input事件超过5s没有被处理完
Service处理超时，前台20s，后台200s
BroadcastReceiver处理超时，前台10S，后台60s
ContentProvider执行超时，比较少见

3、从发生的原因分：

主线程有耗时操作，如有复杂的layout布局，IO操作等。
被Binder对端block
被子线程同步锁block
Binder被占满导致主线程无法和SystemServer通信
得不到系统资源（CPU/RAM/IO）

4、从进程的角度分：

问题出在当前进程:
主线程本身耗时, 或则主线程的消息队列存在耗时操作;
主线程被本进程的其他子线程所blocked;
问题出在远端进程(一般是binder call或socket等通信方式)

##　Android apk打包流程分析和apk的签名机制
打包流程：
https://juejin.cn/post/6844903850453762055

## Android 中View的底层绘制原理Chaerographer和SurfaceFlinger
1. ActivityThread handleLaunchActivity() -> performLaunchActivity 创建Activity对象，执行Activity的attch ，初始化PhoneWindow，调用Activity的onCreate方法，初始化DecorView，并且添加布局到DecorView的content，执行Activity#onStart方法， 
2. handleResumeActivity 调用 Activity的onResume方法，并添加DecorView到WindowManager中去，这会调用
WindowManager的本地实现类WindowManagerImpl->addView,回调到WindowManagerGlobal->addView,最后会调动ViewRootImpl的setView方法。然后回调ViewRootImpl的requestLayout

3.ViewRootImpl  performTraversal， 对整个viewTree进行measure,layout,draw.通过Charographer。


https://www.jianshu.com/p/d3be5def8398

## Android 丢帧 冻帧 和帧率优化相关知识分析

## Android 各个主要版本的Feature

## Android 编译系统相关知识 make .mk,.bp ，Aosp库各个目录的作用。以及主要版本的演进Android5.1，Android 6.0,Android 8.0,Android 9.0

## Android selinux 相关知识 sepolicy的定义，生成。



## Android 内存优化相关内容
内存优化，常常和卡顿联系在一起，在用户看来，应用使用起来常常卡顿，UI不流畅，是内存不足的最直观的体现，因此，对内存的优化，常常和UI卡顿的优化放在一起讨论。
内存问题包括内存泄露和内存溢出，还有内存耗尽，内存泄露比较难从用户直观的看出来，有的泄露是缓慢的积小成多的；内存溢出了之后，作为一种RuntimeException，如果没有被捕获，会导致程序被杀死；还有内存耗尽的问题，内存泄漏积少成多，最终达到应用分配的内存的最大值，会出现内存耗尽的问题，内存耗尽之后，系统会杀死该进程。

内存泄漏的原因（对象上存在不正确的引用，导致垃圾回收时无法被释放）：
1.使用非静态内部类，并持有Activity的引用，没有在Activity生命周期周期结束之前释放；
2.使用静态变量，持有Activity的强引用，没有在Activity生命周期结束前释放；
3.存在未关闭的Cursor，文件流等；
4.使用Bitmap的时候，没有在不需要的时候，把Bitmap占用的内存Recycle掉
5.在多线程的场景中，工作线程持有对主线程的引用（非ApplicationContext），不释放


内存溢出的原因：
1.太多的内存泄露
2.如在for循环中声明很多资源占用较大的对象实例。如Bitmap，造成生成很多的内存碎片
3.加载大的图片不基于显示区域对图片进行优化，造成资源浪费。
4.在滑动或者是其他频繁调用的方法中声明图片等实例对象（不使用缓存）或者是频繁的io操作


如何内存优化：
1. 使用一些三方的内存检测的工具，如LeakCanary （研发状态），Bugly（线上）
2. 静态代码检测工具，如StrictMode，可以检测出一些未关闭的流，cursor，之类的问题。
3. 使用AS自带的cpu 监控和 内存监控 网络监控等对于怀疑的场景进行监控，如果说出现内存抖动（太多小内存频繁申请，回收）或者是内存持续占用缓慢升高，主动gc也回收不到内存的情况（内存泄露），就需要对代码中逻辑进行检查
4. 如果说是对于系统中其他已经上线的应用进行检测，可以使用systrace。systrace可以看到系统当时的状态信息。对于分析问题，更具有全面意义。


系统卡顿优化： 

帧率：一般来说普通应用，60帧/s基本上就算是比较流畅的状态，游戏类应用可能帧率到达30帧/s的状态是比较流畅的。
卡顿（丢帧）： 在60帧以下，一帧绘制超过16ms，就会出现丢帧。
冻帧：一帧绘制超过700ms 就表示冻帧

卡顿的原因：
1.频繁的GC，也就是内存抖动的情况，在一段时间内，有大量的对象的生成和回收，例如在for循环里声明对象，在滑动或者是绘制的方法中声明很多对象。
2.过度绘制： 一个对象被绘制很多遍，或者是不可见元素仍然被绘制了	
3.过度复杂的布局






## Android I/O优化相关内容

### Linux I/O 的基本组成
众所周知，Android 基于 Linux 系统，先介绍一些 Linux 上 I/O 的知识。

1. I/O 操作由应用程序、文件系统和磁盘共同完成，应用程序将 I/O 命令发送给文件系统，文件系统在合适的时间把 I/O 指令发送给磁盘。I/O 的流程如下图：

![io流程图](images/interview/image/io1.webp)

CPU 和内存的速度比磁盘快得多，I/O 操作的瓶颈在于磁盘的性能。为了降低磁盘对应用程序的影响，文件系统要进行各种各样的优化。

2. 文件系统
简单来说，文件系统就是存储和组织数据的方式。应用程序调用 read() 方法，系统会通过中断从用户空间进入内核空间，然后经过虚拟文件系统、具体文件系统、页缓存。
![文件系统](images/interview/image/io2.webp)

	* 虚拟文件系统（VFS）。主要用于屏蔽具体的文件系统，为应用程序的操作提供一个统一的接口。
	* 文件系统（File System）。ext4、F2FS 都是具体文件系统实现。每个文件系统都有适合自己的场景。
	* 页缓存（Page Cache）。文件系统对数据的缓存，读文件时先检查页缓存，如果命中就不去读磁盘。


3. 磁盘
磁盘指的是系统的存储设备，常见的有机械硬盘、固态硬盘等。如果发现应用程序要读的数据没有在页缓存中，这时候就需要真正向磁盘发起 I/O 请求。磁盘 I/O 的过程要先经过内核的通用块层、I/O 调度层、设备驱动层，最后才会交给具体的硬件设备处理。

![磁盘调度架构图](images/interview/image/io3.webp)

磁盘架构
* 通用块层。接收上层发出的磁盘请求，并最终发出 I/O 请求。它与 VPS 的作用类似。
* I/O 调度层。根据设置的调度算法对请求合并和排序。不能接收到磁盘请求就立刻交给驱动层处理。
* 块设备驱动层。根据具体的物理设备，选择对应的驱动程序，通过操控硬件设备完成最终的 I/O 请求。

### Android IO

1. 文件系统： Android普遍采用的是ext4的文件系统，以及f2fs（2012三星针对为闪存研发的文件系统，它针对闪存进行了大量优化，F2FS 文件系统在小文件的随机读写方面比 ext4 更快，**随机写性能提高**），目前还有只在华为上应用的，华为自研的EROFS（2018年6月，华为工程师在开源社区展示了基于Linux的全新只读文件系统EROFS（Extendable Read-Only File System），采用改进的压缩算法，致力于提高文件访问性能，特别是**随机读**性能。根据当时公布的测试数据，可以看到执行随机读取数据时，EROFS有着一边倒的优势，并且文件压缩率越小时优势越明显：当文件压缩率为4%（即100MB文件压缩为4MB）时，提升高达172%。在EROFS面世后的半年多里，华为工程师对其持续打磨，终于在P30上实现了规模商用）

2. 磁盘格式： Android 使用的闪存主流有2种类型，一种是emmc（Embedded Multi Media Card），另外一种是UFS（2.0,2.1），USF的读写速度更快，性能更好（最高速度在300MB/s以下的大几率都是eMMC，在500MB/s附近则可能是UFS 2.0，在700MB/s以上则较大可能是UFS 2.1了）

	* 查看磁盘类型方法：`ls /proc/fs/*` ；在ext4文件系统下如果看到很多个emmc-blkxxx，则是emmc的 ，如果说文件系统为f2fs的就继续查看 `ls /proc/fs/f2fs/`下，如果说以sd开头则是ufs居多。

	* 另外，如果说是ext4的文件系统，查看emmc-blk+数字对应的具体分区的方法：

	> `ls -l /dev/block/platform/ff0f0000.rksdmmc/by-name/`

3. 手机变卡
Android 手机用久了会变卡，除了系统升级、设备折旧等因素，还和 I/O 有密切关系。I/O 操作变慢的原因有下面几条：

	* 内存不足。系统回收 Page Cache 和 Buffer Cache 的内存，大部分的写操作会直接落盘，导致性能低下。
	* 写入放大
	闪存重复写入需要先进行擦除，一次写入会引起整个块数据的迁移，导致写入时间非常久。
	* 设备性能差 
	在高负载的情况下容易出现瓶颈。
	* 文件损坏
	文件损坏是令人头疼的问题，大多是由不正确的操作导致的。文件损坏的原因可以从应用程序、	文件系统和磁盘三个角度来分析：

		* 应用程序。
		大部分的 I/O 方法都不是原子操作，文件的跨进程或者多线程写入、使用一个已经关闭的文件描述符 fd 来操作文件，都有可能导致数据被覆盖或者删除。
		* 文件系统。虽说内核崩溃或者系统突然断电都有可能导致文件系统损坏，不过文件系统也做了很多的保护措施。例如 system 分区保证只读不可写，增加异常检查和恢复机制。
		* 磁盘。手机上使用的闪存是电子式的存储设备，所以在资料传输过程可能会发生电子遗失等现象导致数据错误。

4. I/O 操作的类型

总体分为，标准IO，mmap,直接IO三种类型。

![IO操作类型](images/interview/image/io4.webp)

	1. 标准 I/O
	应用程序平时用到 read/write 操作都属于标准 I/O，也就是缓存 I/O（Buffered I/O）。它的关键特性有：
		- 对于读操作，当应用程序读取某块数据时，如果这块数据已经在页缓存中，那么就不需要经过物理读盘操作。
		- 对于写操作，应用程序会先将数据写到页缓存中去，不需要等全部数据被写回磁盘，系统会定期将页缓存中的数据刷到磁盘上。
		- 缓存 I/O 可以很大程度减少真正读写磁盘的次数，从而提升性能。但是延迟写机制可能会导致数据丢失。在实际应用中，如果某些数据非常重要，我们应该采用同步写机制。
		- 读操作时，数据会先从磁盘拷贝到 Page Cache 中，然后再从 Page Cache 拷贝到应用程序的用户空间，这样就会多一次内存拷贝。内存相对磁盘是高速设备，即使多拷贝一次，也比真正读一次硬盘要快。

	2. mmap
	mmap 把文件映射到进程的地址空间，提高了 I/O 的性能。
	mmap 的优点有：
		- 减少系统调用。只需要一次 mmap() 系统调用，后续所有的调用像操作内存一样。
		- 减少数据拷贝。mmap 只需要从磁盘拷贝一次，由于做过内存映射，不需要再拷贝回用户空间。
		- 可靠性高。mmap 把数据写入页缓存后，跟缓存 I/O 的延迟写机制一样。
	存在的缺点：
		- 虚拟内存增大。Apk、Dex、so 都是通过 mmap 读取。mmap 会导致虚拟内存增大，mmap 大文件容易出现 OOM。
		- 磁盘延迟。mmap 通过缺页中断向磁盘发起真正的磁盘 I/O，不能通过 mmap 消除磁盘 I/O 的延迟。在 Android 中可以将文件通过 MemoryFile 或者 MappedByteBuffer 映射到内存，然后进行读写，使用这种方式对于小文件和频繁读写操作的文件还是有一定优势的。

	mmap 比较适合对同一块区域频繁读写的情况，推荐使用 I/O 线程来操作。用户日志、数据上报都满足这种场景，另外需要跨进程同步的时候，mmap 也是一个不错的选择。Android 跨进程通信有自己独有的 Binder 机制，它内部也是使用 mmap 实现。

	3. Direct I/O
	一些数据库自己实现了数据和索引的缓存管理，对页缓存的依赖没那么强烈。它们想绕开页缓存机制，减少一次数据拷贝，它的数据也不会污染页缓存。
	直接 I/O 访问文件方式减少了一次数据拷贝和一些系统调用的耗时，很大程度降低了 CPU 的使用率以及内存的占用。负面影响就是读写操作都是同步执行，导致应用程序等待。

	4. 同步与异步 I/O
	多线程阻塞式在 I/O 操作上的并没有优势，I/O 操作的主要瓶颈在于磁盘带宽。所以 I/O 操作不能开大量的线程。
	NIO 是非阻塞 I/O，将 I/O 以事件的方式通知，可以减少线程切换的开销。NIO 的最大作用不是减少读取文件的耗时，而是最大化提升应用整体的 CPU 利用率。
	另外，非常推荐 Square 的 Okio，它支持同步和异步 I/O，也做了比较多的优化。

I/O 优化对提升应用的体验非常有用，希望上面所讲的内容对你有帮助。

作者：落英坠露
链接：https://www.jianshu.com/p/43af9c156674
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

5. IO优化

	1. emmc 擦写次数和判断 emmc 使用寿命是否到期方法

	2. 查看系统整体的 io 状态

		2.1 iostat
		
		比如说我们想查看当前系统的 io 状态，以 kb/s　作为单位:	
		
		
    ```shell
	1|root@rk3368:/ # busybox iostat -k -d                                
	Linux 3.10.0 (localhost) 	03/03/21 	_aarch64_	(8 CPU)
	
	Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
	mmcblk0          70.26      1981.90       269.29  225512879   30641156
	mmcblk0p7         0.00         0.18         0.00      20288          0
	mmcblk0p9         0.00         0.00         1.10        305     124868
	mmcblk0p11        3.70       228.62         0.00   26013646         16
	mmcblk0p12        0.00         0.00         0.11        474      11992
	mmcblk0p15       59.42      1753.10       268.08  199478166   30504280
    ```


		比如说我们想查看当前系统的 io 状态，以 kb/s　作为单位，并且没两分钟统计一次，共5次:

		```
		busybox iostat -d -k  2 5
		```

		2.2 iotop

		iostat　只能查看系统的整体 io　状态，当 io 状态异常高时，　不能定位是哪个进程引起的问题，　这时我们可以使用 iotop 这个工具来查看每个进程的 io 占用比例。但是iotop只在Android9.0以上才原生集成，低版本系统，需要push Github开源脚本 [iotop.sh]('https://github.com/laufersteppenwolf/iotop'). 但是该脚本我在使用过程中发现会有不能用的情况，提示`Your kernel does not support I/O accounting,
		which is required for this tool to work :(`
	
		2.3 vmstat
		vmstat是一个非常有用的脚本，用来查看虚拟机当前的一些状态数据。

		```
		vmstat 2
		```

		2.4 http://qiushao.net/2020/02/11/Android/iostat/
		iotop　只可以查看某个时间点的 io 速率，　我们需要统计某个进程总共的 io 总量时就得使用　cat /proc/$pid/io 这个方法了。
## Android 性能优化相关内容

## Android 应用启动优化

1. 冷启动：系统中没有该应用的进程，这时系统会创建一个新的进程分配给该应用；
2. 热启动：在启动应用时，系统中已有该应用的进程（例：按back键、home键，应用虽然会退出，但是该应用的进程还是保留在后台）；
应用启动时间统计：
`adb shell am start -W componenetName/Action`
3. 温启动： 
温启动包含���在冷启动期间发生的部分操作；同时，它的开销要比热启动高。有许多潜在状态可视为温启动。例如：

用户在退出应用后又重新启动应用。进程可能已继续运行，但应用必须通过调用 onCreate() 从头开始重新创建 Activity。
系统将您的应用从内存中逐出，然后用户又重新启动它。进程和 Activity 需要重启，但传递到 onCreate() 的已保存的实例 state bundle 对于完成此任务有一定助益。

### 冷启动优化
冷启动是指应用从头开始启动：系统进程在`冷启动`后才`创建应用进程`。发生冷启动的情况包括`应用自设备启动后`或`系统终止应用后首次启动`。这种启动给最大限度地减少启动时间带来了最大的挑战，因为系统和应用要做的工作比在另外两种启动状态中更多。

在冷启动开始时，系统有三个任务，它们是：

1. 加载并启动应用。
2. 在启动后立即显示应用的空白启动窗口。
3. 创建应用进程。

系统一创建应用进程，应用进程就负责后续阶段：

1. 创建应用对象。
2. 启动主线程。
3. 创建主 Activity。
4. 扩充视图。
5. 布局屏幕。
6. 执行初始绘制。
一旦应用进程完成第一次绘制，系统进程就会换掉当前显示的后台窗口，替换为主 Activity。此时，用户可以开始使用应用。

参考：https://developer.android.com/topic/performance/vitals/launch-time?hl=zh-cn
https://developer.aliyun.com/article/687808（分析的比较好）

冷启动优化：
1. 减少在Application的onCreate方法，以及主Activity的生命周期（onCreate，onStart）中进行IO或者是其他耗时操作，可以延时加载的就延时或者放在工作线程
2. 	绘制优化，检查view层级，以及view绘制的复杂度，尽量优化层级，减少绘制耗时，在layout中，使用ViewStub，在适当的时候再inflate真正的布局
3. 注意尽量少在Application，Activity启动时，调用很多Service以及初始化很多非常复杂的工具类，对于某些必要的的工具类，尽量做成单例
4. 使用系统主题，来预防系统白屏的问题，由于启动的时候，在应用尚未初始化绘制完成之前，会临时展示一个预览窗口（白屏），可以通过给Activity加上theme主题，增加颜色，这样就可以不显示白屏，然后再在MainActivity的onCreate方法调用super.onCreate之前把主题换回真正的主题。

### 热启动
由于热启动的时候，应用走的生命周期方法是
onRestart-> onStart-> onRestoreInstanceState -> onResume,所以重点关注Activity在这几个生命周期中的工作流程和逻辑。进行相应优化，同时应该考虑冷启动相关的一些界面绘制方面的优化点。








## Android onSaveInstanceState和onRestoreInstanceState调用时机
https://blog.csdn.net/Liu_yunzhao/article/details/79382897
onSaveInstanceState()肯定被执行时机

1、当用户按下HOME键时 
2、从最近应用中选择运行其他的程序时 
3、按下电源按键（关闭屏幕显示）时 
4、从当前activity启动一个新的activity时 
5、屏幕方向切换时(竖屏切横屏或横屏切竖屏)

onSaveInstanceState()执行顺序是在onPause()之后，onStop()之前即： 
onPause() –> onSaveInstanceState() –> onStop()


# HTTP相关知识

## HTTP三次握手和四次挥手

## HTTP协议演进（HTTP1.0,1.1,2.0）

## HTTPS 加密方式（对称加密，非对称加密）

对称加密： Aes， des加密， 同一个秘钥，加密解密都用它

非对称加密： 公钥，私钥，客户端只持有公钥，加密数据后，服务端用私钥解密，

非对称加密：时间长，数据量小，泄露后历史数据都可以解密

对称加密： 容易破解，时间短，数据量大。



## HTTP 协议详解，Head Body分析



# 虚拟机相关
1. Java虚拟机加载类模型
http://qiushao.net/2020/02/18/Java/Java-%E7%B1%BB%E7%9A%84%E5%8A%A0%E8%BD%BD%E6%9C%BA%E5%88%B6%E4%BB%8B%E7%BB%8D/



## JVM虚拟机的内存模型


## Android虚拟机（Dalvik,art）虚拟机内存模型和特点

art虚拟机：
1.在堆上做了区分，将堆内存分成了 图像堆， 数组堆

## Android 虚拟机的垃圾回收机制(GC)

综述 jvm gc和delvik ，art虚拟机gc的区别：https://zhuanlan.zhihu.com/p/24835977
官方文档： https://source.android.google.cn/devices/tech/dalvik/gc-debug?hl=zh-cn

###　JVM 内存回收算法：

1. 标记-清除
2. 复制
3. 标记-压缩
4. 分代


### GC指标：

这里的几个概念，分别是Java堆的起始大小（Starting Size）、最大值（Maximum Size）和增长上限值（Growth Limit）。
在启动Dalvik虚拟机的时候，我们可以分别通过-Xms、-Xmx和-XX:HeapGrowthLimit三个选项来指定上述三个值，以上三个值分别表示表示如下.

1. Starting Size : Dalvik虚拟机启动的时候，会先分配一块初始的堆内存给虚拟机使用。

2. Growth Limit:是系统给每一个程序的最大堆上限,超过这个上限，程序就会OOM

3. Maximum Size：不受控情况下的最大堆内存大小，起始就是我们在用largeheap属性的时候，可以从系统获取的最大堆大小

同时除了上面的这个三个指标外，还有几个指标也是值得我们关注的，那就是堆最小空闲值（Min Free）、堆最大空闲值（Max Free）和堆目标利用率（Target Utilization）。假设在某一次GC之后，存活对象占用内存的大小为LiveSize，那么这时候堆的理想大小应该为(LiveSize / U)。但是(LiveSize / U)必须大于等于(LiveSize + MinFree)并且小于等于(LiveSize + MaxFree)，每次GC后垃圾回收器都会尽量让堆的利用率往目标利用率靠拢。所以当我们尝试手动去生成一些几百K的对象，试图去扩大可用堆大小的时候，反而会导致频繁的GC，因为这些对象的分配会导致GC，而GC后会让堆内存回到合适的比例，而我们使用的局部变量很快会被回收理论上存活对象还是那么多，我们的堆大小也会缩减回来无法达到扩充的目的。 与此同时这也是产生CONCURRENT GC的一个因素，后文我们会详细讲到。

### GC类型：

GC_FOR_MALLOC: 表示是在堆上分配对象时内存不足触发的GC。
GC_CONCURRENT: 当我们应用程序的堆内存达到一定量，或者可以理解为快要满的时候，系统会自动触发GC操作来释放内存。
GC_EXPLICIT: 表示是应用程序调用System.gc、VMRuntime.gc接口或者收到SIGUSR1信号时触发的GC。
GC_BEFORE_OOM: 表示是在准备抛OOM异常之前进行的最后努力而触发的GC。
实际上，GC_FOR_MALLOC、GC_CONCURRENT和GC_BEFORE_OOM三种类型的GC都是在分配对象的过程触发的。而并发和非并发GC的区别主要在于前者在GC过程中，有条件地挂起和唤醒非GC线程，而后者在执行GC的过程中，一直都是挂起非GC线程的。并行GC通过有条件地挂起和唤醒非GC线程，就可以使得应用程序获得更好的响应性。但是同时并行GC需要多执行一次标记根集对象以及递归标记那些在GC过程被访问了的对象的操作，所以也需要花费更多的CPU资源。后文在Art的并发和非并发GC中我们也会着重说明下这两者的区别。

### 回收算法和碎片
由于主流的虚拟机都是实现的（标记－清除）的算法，所以会有内存碎片产生，使用复制算法会避免内存碎片，但是复制算法由于用到两块内存，所以把可用内存缩减了一半。

### ART内存回收机制
ART在delvik虚拟机基础上做出了很多优化，包括
１．ART运行时内部使用的Java堆的主要组成包括Image Space、Zygote Space、Allocation Space和Large Object Space四个Space，Image Space用来存在一些预加载的类， Zygote Space和Allocation Space与Dalvik虚拟机垃圾收集机制中的Zygote堆和Active堆的作用是一样的。Large Object Space就是一些离散地址的集合，用来分配一些大对象从而提高了GC的管理效率和整体性能，类似如下图：

2 GC的类型
kGcCauseForAlloc ，当要分配内存的时候发现内存不够的情况下引起的GC，这种情况下的GC会stop world
kGcCauseBackground，当内存达到一定的阀值的时候会去出发GC，这个时候是一个后台gc，不会引起stop world
kGcCauseExplicit，显示调用的时候进行的gc，如果art打开了这个选项的情况下，在system.gc的时候会进行gc
其他更多

３．Art在GC上不像Dalvik仅有一种回收算法，Art在不同的情况下会选择不同的回收算法，比如Alloc内存不够的时候会采用非并发GC，而在Alloc后发现内存达到一定阀值的时候又会触发并发GC。同时在前后台的情况下GC策略也不尽相同，后面我们会一一给大家说明。


# 设计模式

## 单例模式
构造函数private， 懒汉式，饿汉式，懒汉需要注意多线程加锁。

方式：
1. 私有构造函数，加懒汉式
2. 通过静态内部类中的静态变量
3. 通过枚举函数（不需要加锁，只有方法没有变量的单例）
4. 通过Java的Singleton父类。

## 观察者模式
## 工厂模式，简单工厂
## 装饰器模式	
## 建造者模式
## 策略模式

## 适配器模式
## 代理模式
## 模板模式
## 桥接模式
## 状态模式
## 组合模式
## 命令模式
## 责任链模式

## 设计模式的几大原则



# 数据结构和算法


# 高频面试题

http://6a9396c1.wiz06.com/wapp/pages/view/share/s/1GAVr10WjQ002tX_1K3G9n4B3HlX-g2AmQ0_20ExS_27scZ8
https://github.com/BlackZhangJX/Android-Notes/blob/master/Docs/Android%E7%9F%A5%E8%AF%86%E7%82%B9%E6%B1%87%E6%80%BB.md#%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B-1

