---
layout: post
title:  weex learn
---

# 学习笔记
* Vue 中 export default 和 module.exports  
export default 服从 ES6 的规范,补充：default 其实是别名  
module.exports 服从CommonJS 规范  
一般导出一个属性或者对象用 export default    
一般导出模块或者说文件使用 module.exports  
以上来自网络，不一定准确，有待进一步确认。  
附带:  
import from 服从ES6规范,在编译器生效  
require 服从ES5 规范，在运行期生效  
目前 vue 编译都是依赖label 插件，最终都转化为ES5.  
[js export 命令](http://es6.ruanyifeng.com/?search=import&x=0&y=0#docs/module#export-default-命令)

宏的应用
css_node_t 
定义别名 通过加前缀 好处是。。。。


# define WX_ADD_EVENT(eventName, addSelector) \
if ([addEventName isEqualToString:@#eventName]) {\
    [self addSelector];\
}

隐藏了if语句


css_node 包含，style layout 两部分

 _addUITask 优点

     [[NSRunLoop currentRunLoop] addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];

        _indexDict = [NSMapTable strongToWeakObjectsMapTable];

        OSAtomicIncrement32(&_value);

   if (_isCompositingChild) {
        // compsiting children do not have own layers, so return here.
        return;

viewDidLayoutSubviews
setClipsToBounds

理解handler 和 module 的区别

 你真的了解UIResponder吗？

 程序load之后，代码比如某个类。就加载到，内存的代码区了吗？还是。。。。


从程序启动 一步一步的 再一次 理清楚。。。。。。

## WXBridgeManager
* 创建一个一直运行的线程 jsThread，name:WX_BRIDGE_THREAD_NAME
* 调用 `WXBridgeContext   *bridgeCtx`  执行具体的任务
* bridge 任务都放入 jsThread

## WXComponentManager
* 创建一个一直运行的线程 componentThread，name:WX_COMPONENT_THREAD_NAME 
* component 任务都放入 componentThread 
* 开启 `CADisplayLink *_displayLink` 不停的触发 `_layoutAndSyncUI` 当然，有优化，按需的触发。
* UITask 由_uiTaskQueue管理
* bridge 产生的 UI 任务，回调到这, 处理之后，转发到Component

## WXComponent (Display)

## WXComponent (Layout)
 











