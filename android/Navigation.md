# Navigation

导航指的是允许用户在app中浏览，进入退出到不同内容页的交互。Navigation组件可以帮助我们快速，方便的实现简单或复杂的导航。

**有三个主要部分**

- Navigation graph：导航图，和layout一样是xml文件。包含所有导航相关的信息。
- NavHost: 一个空的容器，显示导航目的地
- NavController：协调内容的变化的一个对象

总结来说就是 navigation作为一个容器，存放多个fragment或activity。可以通过navController来控制这些fragment或activity的显示。

## Navigation Graph

res/目录下创建navigation目录，新建一个mobile_navigation.xml文件

- 根节点

  - 根节点<navigation>下可以包含一个或多个目的地<activity>或<fragment>

  - app:startDestination-- 声明该导航的首页

- 子节点<activity>或<fragment>

  - 导航子节点的 <argument>节点
    - app:argType
    - android:defaultValue
  - 导航子节点的 <action>节点
    - android:id
    - app:destination

```xml
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:app="http://schemas.android.com/apk/res-auto"
            xmlns:tools="http://schemas.android.com/tools"
            app:startDestination="@+id/flow_step_one_dest">
	<fragment
    	android:id="@+id/flow_step_one_dest"
    	android:name="com.example.android.codelabs.navigation.FlowStepFragment"
    	tools:layout="@layout/flow_step_one_fragment">
   		<argument
        	android:name="flowStepNumber"
        	app:argType="integer"
        	android:defaultValue="1"/>
    	<action
       		android:id="@+id/next_action"
       		app:destination="@+id/flow_step_two_dest">
   		</action>
	</fragment>
</navigation>    
```



## 使用navigation graph

在上一步中我们创建了一个导航文件mobile_navigation，那我们怎么引用到它呢？

- 使用NavHostFragment

  布局文件activity_main中 在需要使用导航进行切换内容的地方增加NavHostFragment节点，作为导航的根容器。如下

  ```xml
   <fragment
          android:layout_width="match_parent"
          android:layout_height="0dp"
          android:layout_weight="1"
          android:id="@+id/my_nav_host_fragment"
          android:name="androidx.navigation.fragment.NavHostFragment"
          app:navGraph="@navigation/mobile_navigation"
          app:defaultNavHost="true"
          />
  ```

  - android:name -- 指定fragment为NavHostFragment 节点
  - android:id -- 给定一个id,以便在class 中找到改对象
  - app:navGraph --指定要引用的navigation文件

- 切换导航文件的子内容（fragment/activity）

  - 获取NavController

    在导航的首页（导航文件<navigation>节点app:startDestination指向的页面）对应的Frament/Activity类中通过findNavController来找到所在导航的NavControllelr。

    

  - 导航跳转

    通过获取到的NavController的navigate方法进行导航跳转

    ```xml
    findNavController().navigate(R.id.flow_step_one_dest, null, options)
    ```

    第一个参数为需要跳转的fragment/activity的id

    第二个为携带的参数

    第三个参数可以配合跳转动画等

    - 配置动画示例：

      ```kotlin
      val options = navOptions {
          anim {
              enter = R.anim.slide_in_right
              exit = R.anim.slide_out_left
              popEnter = R.anim.slide_in_left
              popExit = R.anim.slide_out_right
          }
      }
      ```



## 使用action

​	在导航文件的视图中，两个子内容间连接的线代表的就是action

 - action有id属性，但id并不是唯一的
 - app:popUpTo  指host容器中的内容不断出栈知道该属性指向的内容
- app:destination 指定要跳转到navigation容器中的某一个



## 使用safe args

​	导航组件有个Gradle插件safe args, 可以生成简单的对象和构建类文件用以安全的访问导航跳转时的参数。

 - 不使用该插件，我们需要这样访问目标间的参数

   ```kotlin
   val username = arguments?.getString("usernameKey")
   ```

-  使用safe args插件，可以这样用

   ```kotlin
   val username = args.username
   ```

   **使用流程**

 - 添加依赖

   ```xml
   classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:$navigationVersion"
   ```

- 注册插件

  在app/build.gradle中

  ```xml
  apply plugin: 'com.android.application'
  apply plugin: 'kotlin-android'
  apply plugin: 'androidx.navigation.safeargs'
  ```

- 传参

  在导航文件的某个activity或fragment下的<argument>节点中传参,构建生成对应的类FlowStepFragmentArgs，<argument>节点定义的flowStepNumber对应与生成的类中会有flowStepNumber这个属性并有set 和 get 方法。

  ```xml
  <fragment
      android:id="@+id/flow_step_one_dest"
      android:name="com.example.android.codelabs.navigation.FlowStepFragment"
      tools:layout="@layout/flow_step_one_fragment">
      <argument
          android:name="flowStepNumber"
          app:argType="integer"
          android:defaultValue="1"/>
  
      <action...>
      </action>
  </fragment>
  ```

- 获取参数

  ```kotlin
          val safeArgs: FlowStepFragmentArgs by navArgs()
          val flowStepNumber = safeArgs.flowStepNumber
  ```

- 生成的Dictions类

  对于有明确destination的action的<fragment>或<activity>子节点都会生成一个对应的dictions类

  可以通过该类获取一个带参的action进行导航跳转。如下：

  ```kotlin
  val flowStepNumberArgs = 1
  var nextAction = HomeFragmentDirections.nextAction(flowStepNumberArgs)
  findNavController().navigate(nextAction)
  ```

   

## 导航结合menu、drawers、bottom navigation

navigation组件有一个类***NavigationUI*** 及 **navigation-ui-ktx**。

**NavigationUI** 有静态方法让menu items 与  navigation destination 关联。

- **NavigationUI 结合 menu**

  - 创建menu

    ```kotlin
    override fun onCreateOptionsMenu(menu: Menu): Boolean {
        menuInflater.inflate(R.menu.overflow_menu, menu)
        return true
    }
    ```

  - 按钮点击事件处理

    ```kotlin
     override fun onOptionsItemSelected(item: MenuItem): Boolean {
         //NavigationUI会遍历当前这个导航是否有destination与MenuItem的id匹配
         //如果找到就会进行导航调整
    	return item.onNavDestinationSelected
         (findNavController(R.id.my_nav_host_fragment))
                    || super.onOptionsItemSelected(item)
        }
    ```

- **NavigationUI 结合 Bottom Navigation**

  - 布局底部导航View

    ```xml
    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/bottom_nav_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:menu="@menu/bottom_nav_menu" />
    ```

  - setupWithNavController

    NavigationUI 让BottomNavigationView对象一句代码与navigation destination建立管理

    ```kotlin
    val bottomNav = findViewById<BottomNavigationView>(R.id.bottom_nav_view)      bottomNav?.setupWithNavController(findNavController(R.id.my_nav_host_fragment))
    ```

    setupWithNavController方法内部处理了BottomNavigationView item的点击事件



- **NavigationUI 结合  Navigation  Drawer**

  

## 外部widget跳转到destination

Navigation components支持deep link。deep link是一种跳转到app中非首页页面的一种能力，有两种方式可以实现，url和pending intent。

Navigation 提供 `NavDeepLinkBuilder` 类去构造PendingIntent来实现deep link。

- 创建小部件

  -  **AppWidgetProvider**子类实现

    新建一个类**DeepLinkAppWidgetProvider** 继承**AppWidgetProvider**并重写onUpdate()方法，

    - 小部件呈现view

      ```kotlin
      val remoteViews = RemoteViews(
                  context.packageName,
                  R.layout.deep_link_appwidget
              )
      ```

    - 设置点击事件(传入一个PendingIntent对象)

      ```kotlin
      remoteViews.setOnClickPendingIntent(R.id.deep_link_button, pendingIntent)
      ```

    - 添加到小部件集中

      ```kotlin
      appWidgetManager.updateAppWidget(appWidgetIds, remoteViews)
      ```

  - **PendingIntent**

    通过Navigation 提供 `NavDeepLinkBuilder` 类构造PendingIntent对象

    ```kotlin
     val args = Bundle()
            args.putString("myarg", "From Widget")
            val pendingIntent = NavDeepLinkBuilder(context)
                    .setGraph(R.navigation.mobile_navigation)//设置navigation文件，相当于														  存放多个内容页导航容器
                    .setDestination(R.id.deeplink_dest)  //设置展现内容
                    .setArguments(args)              //传递参数
                    .createPendingIntent()
    ```

    

- **后置堆栈**

  深度链接的后置堆栈是使用传入的导航图确定的。如果您选择的显式活动具有父activity，则还包括那些父activity。



## web link 与 destination关联

​	android中可以通过web链接来打开app的activity，通常是通过意图筛选来到达的。

​	navigation 组件提供了一个更简单易用的方式。使用<deepLink>

 - 增加<deepLink>节点

   在navigation graph文件中的某个节点添加<deepLink>

   ```xml
   <fragment
      ...
       <argument
           android:name="myarg"
           android:defaultValue="Android!"/>
       <deepLink app:uri="www.example.com/{myarg}" />
   </fragment>
   ```

- 配置 AndroidManifest.xml文件

  配置好<nav-graph>后，系统会自动生成适当的过滤器

  ```kotlin
  <activity android:name=".MainActivity">
      <intent-filter>
          <action android:name="android.intent.action.MAIN" />
          <category android:name="android.intent.category.DEFAULT" />
          <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
  
      <nav-graph android:value="@navigation/mobile_navigation" />
  </activity>
  ```

- 测试

  在其他地方点击www.example.com/test 时会弹出让你选择打开该链接的方式，选择我们的app就可以看到效果了。



## [更多](https://zjcqoo.github.io/-----http://d.android.com/arch/navigation)
