---
title: Android组件化思考
date: 2018-07-26 11:03:35
tags: [Android,组件化]
---
## 组件化的目标
- 代码解耦
- 组件单独运行
- 跨模块通信
- 集成调试
- 代码边界

## 组件化关键技术点
### 代码解耦
- 一种是基础库library，这些代码被其他组件直接引用。比如网络库module可以认为是一个library
- 另一种我们称之为Component，这种module是一个完整的功能模块
- 负责拼装这些组件以形成一个完成app的module，一般我们称之为主项目、主module或者Host

### 组件单独运行
- 只需要把apply plugin: 'com.android.library'切换成apply plugin: 'com.android.application'就可以，但是我们还需要修改下AndroidManifest文件，因为一个单独调试需要有一个入口的actiivity
```java
if(isRunAlone.toBoolean()){
    apply plugin: 'com.android.application'
}else{
    apply plugin: 'com.android.library'
}
.....
resourcePrefix "readerbook_"
sourceSets {
     main {
           if (isRunAlone.toBoolean()) {
                  manifest.srcFile 'src/main/runalone/AndroidManifest.xml'
                  java.srcDirs = ['src/main/java','src/main/runalone/java']
                  res.srcDirs = ['src/main/res','src/main/runalone/res']
           } else {
                  manifest.srcFile 'src/main/AndroidManifest.xml'
           }
     }
}
```
<!-- more -->
### 跨模块通信
- UI之间的跳转用的是一个中央路由的方式，这里使用阿里的ARouter
- 数据传输使用接口回调的方式或者eventbus

### 集成调试
- 主项目直接依赖各个组件或者通过生成aar的方式进行依赖，组成完整app进行调试

### 代码边界
- 组件之间不能直接调用，或者主项目不能直接使用组件中的具体类，要保持组件之间严格的独立
- 通过自定义gradle插件，将主项目依赖组件的代码放在运行时添加，这样在编译器就无法直接使用组件中的类

## 业务拆分
### core基础组件层
- 通用的开源库（网络，图片，工具）
- 网络库
- 自研工具库

### business基础业务组件层
- mvp,mvvm架构的基类
- 业务相关的开源UI控件
- 业务相关的拓展函数(kotlin相关)
- 统一的图片，字符串，主题样式

### commonbusiness通用业务组件层
这里会包含多个module，每个module只负责唯一一种功能
- 登录注册module业务组件
- IM消息module业务组件
- 钱包支付module业务组件
- 版本更新module业务组件
- 调试设置module业务组件

### 具体业务层
实现具体的业务代码，用到通用业务组件，就使用路由跳转，避免直接使用

## 组件化工具
集成调试和组件调试，需要配置不同的android插件，所以需要自定义gradle插件来自动化处理上面的过程
https://github.com/luojilab/DDComponentForAndroid
