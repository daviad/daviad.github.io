---
layout: post
title:  "vpn learn"
---

# 前言
VPN开发，采坑、学习笔记。  
VPN开发分为客户端和服务器端。

# 服务器端
1. 安装并配置softether vpn [点击进入官网](https://www.softether.org),官网有很多介绍。
2. 参考郭芳华提供的视频资料。（她亲自录制的视频）
### 视频中注意事项
1. IPSec/L2TP Setting 中第三个选项：EtherIP / L2TPv3 over IPsec Server Function 需要选中。视频中没有选中，如果不选中，Mac，IOS是无法链接访问的。 [参考链接](https://www.softether.org/4-docs/2-howto/9.L2TPIPsec_Setup_Guide_for_SoftEther_VPN_Server/1.Setup_L2TP%2F%2F%2F%2FIPsec_VPN_Server_on_SoftEther_VPN_Server)


# 客户端（ios，mac）
客户端5种方案介绍
1. 直接利用系统自带设置功能手动配置。  
    1. Mac参考地址(https://www.softether.org/4-docs/2-howto/9.L2TPIPsec_Setup_Guide_for_SoftEther_VPN_Server/5.Mac_OS_X_L2TP_Client_Setup)
    2. IOS参考地址(https://www.softether.org/4-docs/2-howto/9.L2TPIPsec_Setup_Guide_for_SoftEther_VPN_Server/2.iPhone_iPad_L2TP_Client_Setting)

2. 编程实现，利用NetworkExtension framework 提供的NEVPNManager实现。代码很简单。网上有很多介绍，我自己实现的选择不同的targe 编译Mac和ios版本。 [SVN地址](https://192.168.0.102/svn/iphone/trunk/iOS-Team/XiuweiDing/IosVPN/) 
    1. 这个API的局限：只支持IPSec-specific portion of a VPN configuration和 IKEv2-specific portion of a VPN configuration. 这两个苹果自带的协议的VPN。

3. 编程实现，利用NetworkExtension framework 提供的NETunnelProviderManager实现。
   1. 这个API的实现比较复杂。 可以创建虚拟网卡，获取ip数据包，实现自己私有VPN协议。 需要拥有VPN相关知识才能完成。
   2. 苹果官方的例子SimpleTunnel，可以参考。 这个例子只是介绍了建虚拟网卡，获取ip数据包，关于VPN私有协议并没实现。 注意这个demo是swift写的版本比较老，请到github上下载这个好心人分享的 [地址](https://github.com/partyspy/SimpleTunnel)

4. 配置文件实现
 * IOS
   1. 利用iphone配置实用工具。配置vpn的相关配置，导出.mobileconfig文件。
   2. 将.mobileconfig文件放到服务器。
   3. iPhone通过Safari访问配置文件，Safari会自动引导安装配置文件。
 注意：iphone配置实用工具使用很简单，配置好通用和VPN两个选项即可。这是这个工具vpn选项没有用户密码这一选项，这样用户在安装时，会弹框让填写，打开导出的配置文件在 name 下面 添加一下代码
     ```
     <key>AuthPassword</key>
     <string>123456（替换为你的密码）</string>
     ```

 * Mac
   1. 通过方案一配置好VPN。导出配置文件。
   2. 将配置文件放到服务器。
   3. Mac通过网络下载配置文件，双击即可安装。

5. 命令行（此方案值适合Mac）
   1. [macosvpn](https://github.com/halo/macosvpn) 
   2. [macosx-vpn-config-sample](https://github.com/povilasb/macosx-vpn-config-sample)   
   这两个实现基本一样，我本想改成mac app 但是失败了。 因为这个命令行需要sudo 才可以。

# 目前未解决的问题
1.  利用NetworkExtension framework 提供的NEVPNManager实现。无法链接到我自己搭建的L2TP server。 有可能是苹果自带的这个协议和server的不同。
2.  NETunnelProviderManager的实现，比较复杂。如果公司给我时间，我也愿意去研究。
3.  利用配置文件，或命令行。客户操作比较繁琐。

# VPN 相关概念 
我的理解：VPN有很多类型，PPTP（苹果已经舍弃）、IKV2、L2TP等等。IPSec是一个总称，但在苹果默认肯能是指Cisco L2TP。 L2TP有新版本L2TPv3，苹果的L2TP可能是L2TPv3.


iOS开发之VPN协议（理论）   

http://blog.csdn.net/dengshuai_super/article/details/51872474  
http://www.51testing.com/html/48/n-3710748.html  
http://www.helusi.com/ios9-network-extension-vpn-api/  

OpenVPN的工作原理
http://blog.csdn.net/naipeng/article/details/54972404

###### 我这边有搭建好的服务器，有需要的找我即可





