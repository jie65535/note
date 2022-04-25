---
tags: #server #game #anime
---

# 【教程】 Grasscutter 一个动画游戏的服务端模拟器
原文地址：https://xtaolink.cn/360.html
作者：深海小涛
# 简介
**[Grasscutter](https://github.com/Grasscutters/Grasscutter)**  一个动画游戏的服务器端模拟器，它能让人更快乐地游戏。

功能
 - 登录
 - 战斗
 - 命令行生成怪物
 - 库存管理（背包、角色等等）
 - 抽卡系统
 - 好友系统
## 指北
本指北基于 `win 11` x64 位系统，game `v2.6`，且已经在 D 盘创建了名为 `Grasscutter` 的文件夹，我们称此为 `工作目录` 。

### 依赖项
#### JDK-8u202
从 [华为镜像源](https://mirrors.huaweicloud.com/java/jdk/8u202-b08/) 下载 `jdk-8u202-windows-x64.exe` 并安装，安装完成后将 `xxx\Java\jdk1.8.0_202\bin` 加入系统环境变量 PATH 中。

你可以通过打开 `cmd` 输入 `java -version` ，如果输出为
```
java version "1.8.0_202"
```
那么此步则已配置正确。

#### Mongodb
从 官网 下载 `Mongodb 5.0.7` 压缩包格式文件，或者 点击这里 直接开始下载，下载后请解压到 `D:\Grasscutter\mongodb` ，此时 `D:\Grasscutter\mongodb\bin` 中应该有 `mongod.exe` 文件。

在 `D:\Grasscutter\mongodb` 中新建一个 `db` 文件夹用于保存数据库文件。打开 `cmd` 进入到 `D:\Grasscutter\mongodb\bin` ，输入下列命令启动 `mongodb`
```
mongod --dbpath "D:\Grasscutter\mongodb\db"
```
如果有数据输出，且没有结束运行，则此步已配置正确，请不要关闭此 `cmd` 窗口，保持后台运行。

### 服务端
#### 懒人包
请 [下载此文件](https://drive.google.com/file/d/1AAHg-BqMfew0BRcGtkhn65Uv94lhi0wj/view?usp=sharing) ，直接解压到 `D:\Grasscutter` 中

#### 手动操作
##### 构建服务端
###### 拉取项目
首先你需要安装 `git` ，然后运行 `cmd` 进入 `D:\Grasscutter` 中输入并且运行 `git clone https://github.com/Grasscutters/Grasscutter.git`

###### 开始构建
拉取完成后，进入 `D:\Grasscutter\Grasscutter` ，进行如下操作：

1. 运行 `gradlew.bat`
2. 打开 `cmd` 窗口，输入并且运行 `gradlew jar`
构建将会自动完成，并且生成 `D:\Grasscutter\Grasscutter\grasscutter.jar` ，此为服务端文件，将它放到 `D:\Grasscutter\grasscutter.jar` 即可

###### 下载资源文件
首先创建 `D:\Grasscutter\resources` 文件夹，我们称此为 `资源文件夹` 。

将 [GenshinData](https://github.com/Dimbreath/GenshinData) 中的 `TextMap`、`Subtitle`、`Readable`、`ExcelBinOutput` 移动到 `资源文件夹` 中
将 [gi-bin-output](https://github.com/radioegor146/gi-bin-output.git) 中的 `2.5.52/Data/_BinOutput` 重命名为 `BinOutput` 然后移动到 `资源文件夹` 中
将 [Grasscutter-Protos](https://github.com/Grasscutters/Grasscutter-Protos.git) 中的 `proto` 移动到 `资源文件夹` 中
将 [Grasscutter](https://github.com/Grasscutters/Grasscutter) 中的 `keys`、`data`、`keystore.p12` 移动到 `工作目录` 中

### 运行
此时的目录树结构为：
```
Grasscutter
├── mongodb
│   ├── bin
│   │   └── mongod.exe
│   └── db
├── data
│   ├── Banners.json
│   └── ...
├── keys
│   ├── dispatchKey.bin
│   └── ...
├── resources
│   ├── BinOutput
│   ├── ExcelBinOutput
│   ├── proto
│   ├── Readable
│   ├── Subtitle
│   └── TextMap
├── grasscutter.jar
└── keystore.p12
```
运行 `cmd` 进入 `工作目录` 输入并且运行 `java -jar grasscutter.jar -handbook` ，再次输入并且运行 `java -jar grasscutter.jar`

此时如果没有端口冲突并且 `mongodb` 也在后台同时运行，那么服务器端将正常运行。

你需要运行成功后输入 `account create 用户名 用户id（可选）` 来创建你的游戏账号。

## 客户端
请确保你的客户端为国际版，您可以通过 [Snap.Genshin](https://github.com/DGP-Studio/Snap.Genshin) 切换你的客户端版本为国际版

请下载 [此 Releases](https://github.com/Grasscutters/GrassClipper/releases) 下的 `GrassClipper.zip` 文件，解压到任意目录后，运行 `install.cmd` ，等待进程结束，然后打开 `GrassCutter.exe` 在下方选择游戏的工作目录，例如：`D:\Program Files\Genshin Impact` ，在上方输入 `127.0.0.1` ，然后点击运行，即可打开游戏，在登录时你需要输入创建的用户名和任意密码即可加入测试服务器。

# 总结
过程过于繁琐，而且现在仅仅处于能用的状态（bushi

# 交流
[TG 非官方频道 Discord 官方频道](https://discord.gg/T5vZU6UyeG)