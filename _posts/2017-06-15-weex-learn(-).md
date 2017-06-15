---
layout: post
title:  "weex原理学习（-）"
---

# 前言
1. 如果没有weex 我们掌大是如何 做的？  
比如： IOS页面跳转 :  

 ```
 objcTest://schemaTest?json={"module":"xxxController","argArr":[{"message_type":"15","message": "20881703"}]}

 - (id)initWithJs:(NSDictionary)dic{};
 ```

通过 json字符串，解析 出对应的类及参数，调用  初始化方法。调用push完成。

* 优点：   
简单，能够完成我项目跳转的需求

* 缺点：  
 1. 扩展差           
  * 不能完成不同事件的调用。 比如：web 想调用 start方法，
  * 没有回调等
  * 不能完成界面的布局  

    2.通用性差

2.思考：如何没有weex， 我们如何用优化，扩展我们的方案。    

---          
  
     
###                 

# 带着思考，看看weex是如何做的。
 原理图  
 ![原理图]({{site.url}}/images/weex/flow.png "原理图")     



一些总结：
* JavaScript Runtime、Render Engine 简单的看就是WeexSDK,JS Framework就是那个main.js。
* 一句话概括 JS Framework 的编译过程就是： 将 JS Bundle 转换成 Virtual DOM 发送到原生模块渲染。



# 通过源码看看主线的流程：（看着源码跟着这个流程看一看）
主要参考[Weex 是如何在 iOS 客户端上跑起来的][ref1]

## 一. 初始化环境 >[WXSDKEngine initSDKEnvironment]完成两件事：     
1. 注册默认的component、module、handler   
 * 保存全局对应关系：类，暴露的方法和标签等。即一小部分Virtual DOM创建 （用OC创建的）  
 * 使用 dispatch_once 只会执行一次  
 * OC runtime 
 
2. 执行JSFramework 加载main.js 
 * js的环境和通用的方法 和创建一部分Virtual DOM（js 创建的）
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

## 二. 创建页面 WXSDKInstance  

1. 创建基本View结构  

 * [_instance renderWithURL:url options:@{@"bundleUrl":url.absoluteString} data:nil];  URL：是hello.js的地址。
  通过本地或者网络下载jsbundle之后调用  `[strongSelf _renderWithMainBundleString:jsBundleString];`
 * creat root view
 * WXJSCoreBridge的主要功能：封装JavaScriptCore的相关方法。完成js和oc的相互调用。 
 * WXBridgeManager 
 * WXBridgeContext
 
 >- (void)registerGlobalFunctions  
 注册全局的方法 ，注册了js的回调方法。是js方法执行后回调OC  registerCallNative、registerCallAddElement、registerCallNativeModule 

 ---    
 
  **分割线之间的方法是回调，可以先不关注，等下面call js 之后再回来看，这样更容易理解**
  * Call js 的一个回调方法： `- (NSInteger)invokeNative:(NSString *)instanceId tasks:(NSArray *)tasks callback:(NSString __unused*)callback `     
    
  这个方法根据参数（这里的参数是man.js 创建的Virtual DOM）区分出是component还是module。组装成WXComponentMethod或WXModuleMethod，全局找到oc的对应的类和方法执行。 
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
 ---

 * [[WXSDKManager bridgeMgr] createInstance:self.instanceId template:mainBundleString options:dictionary data:_jsData];
  dom的create body. - (void)createBody:(NSDictionary *)body 调用addsubview。就建立了基本的View的层级结构。

 * Call JS  `- (JSValue *)callJSMethod:(NSString *)method args:(NSArray *)args`   
js 执行一定的逻辑之后，就回调前面已经注册的回调方法,去建立Native的结构。初始化时的call js 也是调用者个方法。
  * 调用流程：native(OC) -- call(参数是native 组装的Virtual DOM) --> JS -- callback(参数是js 组装的Virtual DOM)  --> native(OC);
 
2. hellow weeX 是如何显示的    
* 前面hello.js 里面的text hellow weex文本等js代码，通过Call js `- (JSValue *)callJSMethod:(NSString *)method args:(NSArray *)args`  解析为       

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
（应该是有main.js来解析的），并执行相关逻辑之后
  回调到OC（回调的逻辑前面介绍了，前面注册的registerCallAddElement），具体的实现代码在- (void)registerGlobalFunctions的中 调用WXComponentManager来实现的。

* WXComponentManager 处理流程  
 主要调用下面两个方法    
 
 ```
  - (void)addComponent:(NSDictionary *)componentData toSupercomponent:(NSString *)superRef atIndex:(NSInteger)index appendingInTree:(BOOL)appendingInTree     

  - (void)_recursivelyAddComponent:(NSDictionary *)componentData toSupercomponent:(WXComponent *)supercomponent atIndex:(NSInteger)index appendingInTree:(BOOL)appendingInTree
  ```  
 1.这两方法主要通过 `WXComponent *component = [self _buildComponentForData:componentData];`  创建对应的component。（这里是类WXTextComponent）  
2.创建一个Task 放入队列。task的主要功能是addsubView 。
队列的执行由 [self _layoutAndSyncUI]触发 ; 根据逻辑有定时器DisplayLink，或其他代码直接触发。这个就是我们比较熟悉的界面的刷新渲染界面。当然，这里里面做了很多优化。这里不展开。
* WXTextComponent 如何渲染具体的文字  
 1.  WXTextComponent 创建     
 
	``` Objectiv-C
 	  1.  
 	  - (WXComponent *)_buildComponentForData:(NSDictionary *)data
{
    NSString *ref = data[@"ref"];
    NSString *type = data[@"type"];
    NSDictionary *styles = data[@"style"];
    NSDictionary *attributes = data[@"attr"];
    NSArray *events = data[@"event"];
    
    if (self.weexInstance.needValidate) {
        id<WXValidateProtocol> validateHandler = [WXHandlerFactory handlerForProtocol:@protocol(WXValidateProtocol)];
        if (validateHandler) {
            WXComponentValidateResult* validateResult =  [validateHandler validateWithWXSDKInstance:self.weexInstance component:type];
            if (validateResult && !validateResult.isSuccess) {
                type = validateResult.replacedComponent? validateResult.replacedComponent : @"div";
                WXLogError(@"%@",[validateResult.error.userInfo objectForKey:@"errorMsg"]);
            }
        }
    }

    Class clazz = [WXComponentFactory classWithComponentName:type];
    WXComponent *component = [[clazz alloc] initWithRef:ref type:type styles:styles attributes:attributes events:events weexInstance:self.weexInstance];
    WXAssert(component, @"Component build failed for data:%@", data);
    
    [_indexDict setObject:component forKey:component.ref];
    [component readyToRender];// notify redyToRender event when init
    return component;
}
        
      2.
      - (Class)classWithComponentName:(NSString *)name
{
    WXAssert(name, @"Can not find class for a nil component name");
    
    WXComponentConfig *config = nil;
    
    [_configLock lock];
    config = [_componentConfigs objectForKey:name];
    if (!config) {
        WXLogWarning(@"No component config for name:%@, use default config", name);
        config = [_componentConfigs objectForKey:@"div"];
    }
    [_configLock unlock];
    
    if(!config || !config.clazz) {
        return nil;
    }
    
    return NSClassFromString(config.clazz);
}
	```
从源码可知，是通过前注册的    `[self registerComponent:@"text" withClass:NSClassFromString(@"WXTextComponent") withProperties:nil];`  找到对应的类，通过js 解析的Virtual DOM 找到对应的标签和属性参数等。  
 2.  render  

 等等 都有WXComponent 及其 类别 内部处理。
 由前面WXComponentManager 调用的那两个方法中    
 ```
- (void)_recursivelyAddComponent:(NSDictionary *)componentData toSupercomponent:(WXComponent *)supercomponent atIndex:(NSInteger)index appendingInTree:(BOOL)appendingInTree
{
    WXComponent *component = [self _buildComponentForData:componentData];
    if (!supercomponent.subcomponents) {
        index = 0;
    } else {
        index = (index == -1 ? supercomponent->_subcomponents.count : index);
    }
    
    [supercomponent _insertSubcomponent:component atIndex:index];
    // use _lazyCreateView to forbid component like cell's view creating
    if(supercomponent && component && supercomponent->_lazyCreateView) {
        component->_lazyCreateView = YES;
    }
    
    [self _addUITask:^{
        [supercomponent insertSubview:component atIndex:index];
    }];
   ...
   ...
   ...
```
   可以看到，此时的所有处理都到WXComponent内部了。这是我们熟悉的OC的render 方法。主要是通过draw的方式完成的。具体的这里就不展开了。  
  3.Layout  
 待续。。。。

    
[ref1]:http://www.jianshu.com/p/41cde2c62b81 "Weex 是如何在 iOS 客户端上跑起来的"

