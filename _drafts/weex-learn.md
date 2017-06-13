---
layout: post
title:  "weex learn"
---
1.如果没有weex 我们掌大是如何 做的？  
话题 IOS跳转 :
```
objc://loocha?json={"module":"PostBrowserController","argArr":[{"message_type":"15","message": "20881703"}]}

 - (id)initWithJs:(NSDictionary)dic{};
```
通过 json字符串，解析 出对应的类及参数，调用  初始化方法。调用push完成。

优点：   简单，能够完成我项目跳转的需求

缺点：  
 扩展差           
- 不能完成不同事件的调用。 比如：web 想调用 start方法，
- 没有回调等
- 不能完成界面的布局

通用性差

思考：如何没有weex， 我们如何用优化，扩展我们的方案。    

---          
  
     
###                 

## 带着思考，看看weex是如何做的。
 原理图  
 ![原理图]({{site.url}}/images/weex/flow.png "原理图")     



一些总结：
* JavaScript Runtime、Render Engine 等合称JS Framework 简单的看就是WeexSDK。 一句话概括 JS Framework 的编译过程就是： 将 JS Bundle 转换成 Virtual DOM 发送到原生模块渲染。



通过源码来看看具体主线的流程：主要参考[Weex 是如何在 iOS 客户端上跑起来的][1]

一. 初始化环境 >[WXSDKEngine initSDKEnvironment]完成两件事：     

1.注册默认的component、module、handler   
 * 保存全局对应关系：类，暴露的方法和标签等。即一部分Virtual DOM （用OC创建的）  
 * 使用 dispatch_once 只会执行一次  
 * OC runtime 
 
2.执行JSFramework 加载main.js 
 * js的环境和通用的方法 也有一部分即一部分Virtual DOM（js 创建的）
 * javascriptCore framework
 >这里的最后，调用了一些基础的固定的方法（call js），下面创建页面会解析方法的调用。先不关心，以免，我们的思考主线混乱。

Virtual DOM 的结构类似这样
```Objective-C
 <Weex>[debug]WXJSCoreBridge.m:155, Calling JS... method:registerComponents, args:(
        (
                {
            methods =             (
            );
            type = container;
        }
    )
) 

<Weex>[debug]WXJSCoreBridge.m:155, Calling JS... method:registerComponents, args:(
        (
                {
            methods =             (
            );
            type = div;
        }
    )
)

<Weex>[debug]WXJSCoreBridge.m:155, Calling JS... method:registerModules, args:(
        {
        dom =         (
            addEventListener,
            removeAllEventListeners,
            addEvent,
            removeElement,
            updateFinish,
            getComponentRect,
            scrollToElement,
            addRule,
            updateAttrs,
            addElement,
            createFinish,
            createBody,
            updateStyle,
            removeEvent,
            refreshFinish,
            moveElement
        );
    }
)

<Weex>[debug]WXJSCoreBridge.m:155, Calling JS... method:createInstance, args:(
    0,
    "// { \"framework\": \"Vue\" }\n ........
    ...
    ....
    ...
    those is content of hello5.js );",
        {
        bundleUrl = "file:///Users/dxw/Library/Developer/CoreSimulator/Devices/EAC1F5DC-7CF1-4351-8AD5-9EE9A982C1B2/data/Containers/Bundle/Application/7F5D8BF4-2422-47CC-9B4D-7F82C95D99EA/AppTest.app/hello5.js";
        debug = 1;
    }
)

```
二. 创建页面 WXSDKInstance  
1. 创建基本View结构  
 * [_instance renderWithURL:url options:@{@"bundleUrl":url.absoluteString} data:nil];  URL：是hello.js的地址。
  通过本地或者网络下载jsbundle之后调用 >[strongSelf _renderWithMainBundleString:jsBundleString];
 * creat root view
 * WXJSCoreBridge的主要功能：封装JavaScriptCore的相关方法。完成js和oc的相互调用。 
 * WXBridgeManager 
 * WXBridgeContext
 >- (void)registerGlobalFunctions  
 注册全局的方法 注册了，js的回调方法。是js方法执行后回调OC  registerCallNative、registerCallAddElement、registerCallNativeModule

 * `- (NSInteger)invokeNative:(NSString *)instanceId tasks:(NSArray *)tasks callback:(NSString __unused*)callback `     

 这个方法根据参数区分出是component还是module。组装成WXComponentMethod或WXModuleMethod，全局找到oc的对应的类和方法执行。  
比如全局注册的dom `[self registerModule:@"dom" withClass:NSClassFromString(@"WXDomModule")];`  WXDomModule 类里面有creatBody方法。WX_EXPORT_METHOD(@selector(createBody:))。 调用WXComponentManager具体实现。addsubview等方法。
WXComponentManager 管理WXComponent。 WXcomponent封装了View，及相关操作。用多类别组织代码。如：
```
WXComponent+Layout
WXComponent_internal
WXComponent+BoxShadow
WXComponent+Events
WXComponent+Display
WXTextComponent
WXDivComponent
...
...
```   
 * [[WXSDKManager bridgeMgr] createInstance:self.instanceId template:mainBundleString options:dictionary data:_jsData];
 * Call JS  `- (JSValue *)callJSMethod:(NSString *)method args:(NSArray *)args`   
初始化时的call js 也是调用者个方法。

 dom的create body. - (void)createBody:(NSDictionary *)body 调用addsubview。就建立了基本的View的层级结构。
 
2.hellow weeX 是如何显示的

前面hello.js 里面的text hellow weex文本 通过Call js 处理解析为（应该是有main.js来解析的）
```
<Weex>[debug]WXJSCoreBridge.m:188, callAddElement...2, 14, {
    attr =     {
        "data-v-549ffcfb" = "";
    };
    ref = 15;
    style =     {
        color = "#ff0000";
        fontSize = 50px;
    };
    type = text;
}, -1 [;
```  
回调到OC。（前面注册的registerCallAddElement）oc，通过block  具体的代码在- (void)registerGlobalFunctions的中 调用WXComponentManager来实现的。
    
[1]:http://www.jianshu.com/p/41cde2c62b81 "Weex 是如何在 iOS 客户端上跑起来的"

