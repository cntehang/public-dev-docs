# Issues of migrating from ionic3 to ionic4

## 1. 在package删除了插件后再plugins文件中并没有同步删除

> 需要手动删除，否则会一直拷贝到iOS或Android项目中，导致编译出错

 删除某个插件后，再build，如果失败了，报错某个插件未安装或未找到，直接使用 
 ```cordova platform rm ios/android``` 移除掉platform然后再重新添加就好了，目前还没有更好的处理方式

## 2. 本地跑Android机器build一直失败

> 执行cordova build android 出现输出如下,编译不成功。
>
> ANDROID_HOME=/Users/huangenai/Library/Android/sdk
> JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_191.jdk/Contents/Home
>
> /Applications/Android Studio.app/Contents/gradle/gradle-4.6/bin/gradle: Command failed with exit code EACCES

 解决办法: 是因为项目文件目录的权限问题，给够权限就行

## 3. header ,title,segment,searchbar 在安卓和ios设备上样式差异较大，且兼容较差

> 统一使用iOS样式，设置mode=”ios",这样可以统一头部展示，title显示在正中间，返回按钮也统一了。

## 4. iOS10.3  footer无法显示

在全局样式 ***variables.scss***  文件中更改 ion-content 的 ***display***,代码如下：

``` css
  ion-content {
    display: flex;
  }
```

参考资料：
<https://github.com/ionic-team/ionic/issues/15799>
<https://github.com/ionic-team/ionic/issues/16328>

## 5. header无法透明

> 在有些页面需要header透明，content可在header下滚动，但是返回按钮可以显示。 官方文档中给出的方案是设置***ion-header*** 的 ***translucent*** 为true，同事设置 ***ion-content*** 的 ***fullscreen*** 为true

事实证明此方案目前不起作用，虽然此时header处已经透明了，ion-content也是屏幕高度了，只是ion-content的position是relative, 仍会根据header来设置y轴位置，因此需要将ion-header的position改为 fixed

具体步骤如下:

- 设置ion-content 的 fullscreen为true
- 设置ion-toolbar 的css样式：

  ```css
  --background: transparent;
  --ion-color-base: transparent !important;
  --border-style: none;
  ```

参考资料:
<https://medium.com/@paulstelzer/ionic-4-how-to-make-a-transparent-header-toolbar-38405f2e4a74>

## 6. 系统兼容问题

> ionic4 目前支持：
>
> | Platform    | Supported Versions |
> | ----------- | ------------------ |
> | **Android** | 4.4+               |
> | **iOS**     | 10+                |

参考资料:
<https://developer.apple.com/support/app-store/>

## 7. header中多个toolbar，页面切换时不跟随页面滑动

> 这是ionic的bug，需要升级ionic/core 和ionic/angular

升级明星如下：

``` ts
npm i @ionic/core@latest @ionic/angular@latest
```

参考资料:

<https://github.com/ionic-team/ionic/issues/16540>

<https://github.com/ionic-team/ionic/issues/16262>

## 8. 界面切换时列表卡顿

> 如果下一个界面是一个列表，切换期间加载数据渲染列表会导致卡顿问题
此时可以在生命周期钩子 ngOnInit中拉取数据，但是当 生命周期钩子 ionViewDidEnter 被调用时再显示列表

如果有loading，甚至可以在 ionViewDidEnter 拉取数据
