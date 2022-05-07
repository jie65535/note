---
tags: #android #逆向
---

# Android App 逆向

## 前置工具
 - Java
 - [Apk Tool](https://github.com/iBotPeaches/Apktool) 反编译与构建APK
 - [Jadx](https://github.com/skylot/jadx) 反编译APK
 - [ADB](https://developer.android.google.cn/studio/releases/platform-tools) 安卓调试器
 - [APK Signer](https://github.com/patrickfav/uber-apk-signer) APK签名工具
 - 可选 [APK Studio](https://vaibhavpandey.com/apkstudio/) 可视化交互

## 流程概要
 1. 通过adb把目标apk导出到电脑
 2. 用Jadx反编译apk，定位到要修改的位置
 3. 用ApkTool反编译apk，找到并修改目标
 4. 修改并保存后，用ApkTool构建新的apk
 5. 用ApkSigner对新的apk进行签名
 6. 卸载旧的APK，安装新的

## 有用的链接
 - [smali基本语法](https://www.jianshu.com/p/2766b76da201)

## 具体流程
### 导出目标App的Apk文件
 1. `adb shell pm list package <Filter>` 列出所有包
 2. `adb shell pm path 包名` 查看安装路径
 3. 根据输出路径，用 `adb pull` 拉到电脑上
示例：
```shell
> adb shell pm list package
package:com.andr...
...
package:com.DeviceTest
...
> adb shell pm path com.DeviceTest
package:/system/priv-app/DeviceTest/DeviceTest.apk
> adb pull /system/priv-app/DeviceTest/DeviceTest.apk D:\DeviceTest\
```

### 用 jadx-gui 反编译 apk
这个很简单，安装好以后直接选择打开apk，就可以看到反编译代码了。

可以选择导出项目，然后用自己喜欢的开发环境打开查看。

这一步的目的是查看apk代码，定位要修改的位置，因为接下来修改要用另一个方式进行。

### 用 apk tool 反编译 apk
 1. 根据[安装教程](https://ibotpeaches.github.io/Apktool/install)安装apk tool
 2. 根据[基础教程](https://ibotpeaches.github.io/Apktool/install)使用
其实非常简单，`apktool d DeviceTest.apk` 就可以反编译了，改完代码以后用 `apktool b DeviceTest` 就可以重新编译了，编译后的结果在该目录下 `DeviceTest/dist/DeviceTest.apk`

用VSCode打开反编译的文件夹，可以选择装`smali`插件来提供语法提示，因为要修改的代码是`smali`语言的。

根据jadx-gui中定位文件和方法，然后在smali中修改，达到自己预期目的。这部分网上有一些博客，就不具体描述了。

修改完成后，用参数`b`来构建apk即可。

### 用 uber-apk-signer 签名apk
apk tool 构建的apk是没有签名的，无法直接安装，因此需要使用签名工具对apk进行签名。
直接 `java -jar uber-apk-signer.jar -a D:\DeviceTest\dist\DeviceTest.apk --allowResign --overwrite` 签名

### 用 adb install 安装apk
 1. `adb uninstall com.DeviceTest` 卸载旧的APK
 2. `adb install D:\DeviceTest\dist\DeviceTest.apk` 安装新的APK
后续只需要加上`-r`参数覆盖就可以了，不用每次卸载。`adb install -r D:...`

