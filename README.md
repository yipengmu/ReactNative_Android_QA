# ReactNative_Android_QA

##Android端10个最常见问题
这里逐条记录下最容易遇到的React native android 相关case：

* **1. app启动后，红色界面，unable load jsbundle ：**

1. 解决办法：一般来说就是，你是用dev-serve方式，且你的server没有正确匹配上，如果是用手机跑的话，需要pc和手机在同一个wifi下，且通过menu键设置menu-ip为pc的ip，如果是模拟器，则不需要手动设置ip,设置的话，反倒会出错

* **2. app启动后，红色界面，unRegisteredProject**

1. 提示提示什么，你的app没有在启动时候注册

2. 解决办法：这个后面也是一看就知道的错误，就是你的index.android.bundle中的最下面写的那个 

3. 'componetNameInYourLocalProject'在你的java代码中不是叫这个名字，自己check下，立刻就能修复
`AppRegistry.registerComponent('componetNameInYourLocalProject', () => JSObjAndroid);`

* **3. require（"xxx"）的组件失败**

1. js代码中有时候会出现require（"xxx"）的组件出错
解决办法：检测该node组件是否存在你的服务器上，如果是自己封装的NativeModule话可以直接使用

2. `var CustomMoudle = React.NativeModules.YourCustomModule
CustomMoudle.yourMethodDeclearInYourNative('someparms');`

* **4.调试**

1. 解决办法：可以利用pc端的chrome的 debug工具进行js端的调试，native的调试就只能用logcat跟踪了，目前看到大部分的错误都是自己代码的问题，ReactAndroid本身的Crash较少

* **5.so库的问题**

1. gradle的话，可以通到ndk filter来控制：

`	android {        	
        	defaultConfig {
	    	     ndk {
	 	     abiFilters "x86", "armeabi-v7a"
	            }
         }         `


2. maven的话，可以手动通过libs下的so拷贝来解决问题。

3. 这块有个比较大的坑就是，默认引入的jsc.aar中存在armabi文件夹，但是里面没有jsc.so 。导致在多个地方，去编码源码时ndk方面会报错。

* **6. 关于设备MinSdkVerison**

1. 默认Android要求4.1以上设备(4.0根据网络数据大概占比0.7比例，随着大部分app已经不支持4.0以下设备了，这块倒还可以接受)

2. 刚开始一直使用一个5.0的设备进行ReactAndorid的测试和开发，后来方向，其实搞上一个5.0+的Genymotion模拟器联调起来效率会更高。

* **7.UIExplorer demo问题**

1. 之前一直在看具体接入和代码实现方面的，当大头的工作回过头来看，其实当时应该先从这个UIExploror入手的话，效率和进度应该会有较大提高的。

2. 这块需要编译react源代码，如果遇到了https://github.com/facebook/react-native/issues/3976 的问题，可以使用我在下面回复的方法hook，但是本质原因还是那个armabi jsc.so的问题
![screenshot](http://img4.tbcdn.cn/L1/461/1/dfda3ab17e79df68f00b2ae2c18c24be062186c9)



* **8.能力覆盖范围**

1. 根据团队之前React iOS的经验，跟进主干代码，依赖RN本身提供的UI组件可以满足大部分业务场景。

2. 当然自己如果想复用之前团队沉淀下来的，配合着UIManager和UIModule这块本身工作量到也不算太大。

3. 但是应该尽可能的和团队以后的JS端和iOS端的协议接口保持一致，让React最大的意义发挥出来，“lean once run everywhere”

* **9.数据安全**

1. 0.14之前只支持dev-pc 和assert方式，从0.14.0 realease版本开始支持local file patch加载方式，最新版0.15.1。

2. 因为如果要动态能力，js必定是走网络端下发的，js本身是明文（即使JS做了混淆）,数据防劫持的保护还是必须要做的，这点可以配合https防篡改+sign校验来做

* **10.JNI消息轮训带来的影响**

1. 由于JNI的通信限制，Java层和Native通信是单向的，且为了保证RN的16ms的渲染频率，所有Java-Native-jscore层的通信都是异步的,这样可能对于JAVA层的UI渲染是个性能问题。

2. 当消息量非常大或ListView页面非常复杂时候，每1层Cell的渲染要以Css-ScrowllerView模型需要UI线程的连续绘制，对于瀑布流负责listview等可能会存在性能问题，但是该问题本身肯定是优于H5的体验的

> 后面继续追加相关踩坑

* **11.关于ReactInstanceManager 的创建及赋值**


1. 关于ReactInstanceManager的创建默认是使用，他的builder来创建，但是穿件后对象本身没有主要get接口的暴漏,我们暂且本次render一个view时都重新用builder创建一个实例

2. 此时问题来了，发现同一个context给ReactInstanceManager 设置的jsBunldeFile只有第一个才生效(后来发现，这个貌似是个bug，偶现的会加载不到，不是必先)

* **12.ReactInstanceManager 多实例的问题**

1. 目前的方案，是在重建ReactRootView,使用一份新的ReactInstanceManager实例，不会出现上面的情况。但是这样的话，在内存占用上可能会有问题

2. 多个ReactInstanceManager实例的创建方案，根本问题还是上面第一条那个builder的功能分离，不过这样也有单独实例的好处，这样能够保证在每个view的句柄是想回隔离和独立的。

3. 但是如果使用了多个ReactInstanceManager实例，对于back事件等其他action回调会存在问题，待解决


* **13.关于so包，太大的问题，可以对so进行裁剪**
1. so 打包裁剪方案，只保留armabi so包，部分so包通过v7进行hook，对v7、x86等高级cpu架构进行降级，整体so包比默认React包减小60%，保证包大小质量
2. 业务方可以动态注入业务PackageManager功能，核心模块被聚合在core中

* **14.如何实现 remoteUrl 方式**
1. 配合FaceBook原生的LocalFile加载方式，提供一组download模块，实现业务使用者，只需要传入RemoteUlr，整个React Native 容器依然可以直接获得一个渲染完成的ReactView实例的业务场景

* **15.项目的minsdk 低于16，而react android 默认的minsdk是16**
```
    <uses-sdk
        tools:overrideLibrary="com.facebook.react"
        android:minSdkVersion="14"
        android:targetSdkVersion="23" />
```
1. 解决办法：在manifest中配合overrideLibrary标签
2. 如果有多次集成关系，需要在每一个层级节点去overrideLibrary 直接的依赖package包名声明。
