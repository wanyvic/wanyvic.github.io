---
title: MacOS M1 编译 Qv2ray + Trojan-go + QT6
date: 2023-01-19 06:35:42
updated: 2023-01-19 06:35:42
tags:
categories:
copyright:
---
本文撰写时间为2023/01/19 参考wicast C的文章（链接：<https://wicast.medium.com/-531cd9ed9ea>）在此基础上使用macdeployqt进行打包
# 不使用brew 安装qt6
brew 安装的qt6 在使用macdeployqt会有类似如下错误导致，qv2ray无法正常打包framework
```
ERROR: Cannot resolve rpath "@rpath/QtGui.framework/Versions/A/QtGui"
ERROR:  using QList("/Users/wany/Qv2ray/build3/lib")
ERROR: Cannot resolve rpath "@rpath/QtCore.framework/Versions/A/QtCore"
ERROR:  using QList("/Users/wany/Qv2ray/build3/lib")
ERROR: Cannot resolve rpath "@rpath/QtWidgets.framework/Versions/A/QtWidgets"
ERROR:  using QList("/Users/wany/Qv2ray/build3/lib")
ERROR: Cannot resolve rpath "@rpath/QtGui.framework/Versions/A/QtGui"
ERROR:  using QList("/Users/wany/Qv2ray/build3/lib")
ERROR: Cannot resolve rpath "@rpath/QtCore.framework/Versions/A/QtCore"
ERROR:  using QList("/Users/wany/Qv2ray/build3/lib")
```
## 卸载qt6
```
brew remove qt@6
```
## 使用qt.io官方安装器安装
<https://www.qt.io/download-qt-installer>

可能需要注册账号

参考：<https://blog.csdn.net/ilovejujube/article/details/128035916>
# 安装依赖
```
brew install go grpc cmake v2ray openssl
```
# 编译 Qv2ray
# 编译
注意替换`CMAKE_PREFIX_PATH`为自己的Qt路径
```
git clone https://github.com/Qv2ray/Qv2ray -b dev
cd Qv2ray
mkdir build
cd build
cmake -DQV2RAY_QT6=ON -DQV2RAY_USE_V5_CORE=OFF -DCMAKE_BUILD_TYPE=Release -DQV2RAY_AUTO_DEPLOY=ON -DCMAKE_PREFIX_PATH=/Users/wany/Qt/6.4.2/macos -DOPENSSL_ROOT_DIR=/opt/homebrew/Cellar/openssl@3/3.0.7 ..
make -j16
```
## 运行
```
open qv2ray.app
```

# 编译 QvPlugin-Trojan-Go
注意替换`CMAKE_PREFIX_PATH`为自己的Qt路径
```
git clone https://github.com/Qv2ray/QvPlugin-Trojan-Go -b dev
cd QvPlugin-Trojan-Go
mkdir build
cd build
# 要让插件用 QT6
cmake -DQVPLUGIN_USE_QT6=ON -DCMAKE_PREFIX_PATH=/Users/wany/Qt/6.4.2/macos -DCMAKE_BUILD_TYPE=Release ..
make -j16
```
