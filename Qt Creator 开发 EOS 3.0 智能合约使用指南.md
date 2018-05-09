
# 下载和安装Qt Creator
下载地址： [http://download.qt.io/official_releases/qt/5.10/5.10.1/](http://download.qt.io/official_releases/qt/5.10/5.10.1/)

这里选择： qt-opensource-linux-x64-5.10.1.run

安装：
```python
sudo chmod +x ./qt-opensource-linux-x64-5.10.1.run 
```
然后就是要选择gcc

![](https://eosfans-static.strahe.com/photo/2018/a57dbabe-bf9e-4c63-b6a2-a26c15872c5c.png?x-oss-process=image/resize,w_1920)

# Qt Creator 调试工程

## 导入eos工程

文件 -> 新建文件或项目 -> Import Project -> 导入现有项目

![](https://eosfans-static.strahe.com/photo/2018/c8cee03e-6b62-4ea9-93db-4a794e07623e.png?x-oss-process=image/resize,w_1920)

选择eos工程目录

![](https://eosfans-static.strahe.com/photo/2018/e4bd4ac4-46bb-4da2-8ec3-708a6b94b0d9.png?x-oss-process=image/resize,w_1920)

下一步 -> 下一步 -> 完成
## 配置cmake11

CMake 下载地址：[https://cmake.org/download/](https://cmake.org/download/) 

下载到任意目录，执行解压命令：

```python
tar -zxvf cmake-3.11.0.tar.gz  
./bootstrap
make 
make install
```
工程指定cmake11

菜单栏：工具->选项->构建和运行

概要 -> 项目目录 -> 选择eos根目录

CMake ->Add

名字，然后选择cmake的目录，Apply

![](https://eosfans-static.strahe.com/photo/2018/0169d701-2579-4b13-b5f6-f5f54beca038.png?x-oss-process=image/resize,w_1920)

构建套件(kit)

添加，CMake Tool 选择刚才添加的cmake11，设置为默认，点击ok完成。

![](https://eosfans-static.strahe.com/photo/2018/dfc578f8-08f0-4d69-9b64-9742422cb422.png?x-oss-process=image/resize,w_1920)

## debug调试配置参数

这里要选择新创建的cmake11,默认是之前的，否则会构建失败。

为了编译省时间，可以在项目 -> Build & Run -> cmake11-> Build 

make 参数

运行多核编译： -j4

<b>要断点调试： CMAKE_BUILD_TYPE=Debug</b>

![](https://eosfans-static.strahe.com/photo/2018/9a847b52-c19d-4a4a-b028-b60761d2b1a6.png?x-oss-process=image/resize,w_1920)


构建目录：选择eos/build目录

在编辑或者debug中右键项目

编译完成后，应用程序输出中，没有报错

![](https://eosfans-static.strahe.com/photo/2018/5996076f-6f85-44ae-95bd-b31baa3b8228.png?x-oss-process=image/resize,w_1920)

Run ->  运行 -> Executable  选择你要运行调试的cmake,这里测试选择nodes

选中Run in terminal

最后，在programs/nodes/main.cpp的mian函数中打断点，测试一下

![](https://eosfans-static.strahe.com/photo/2018/6dff4005-228d-489a-a3f7-7d67340328a2.png?x-oss-process=image/resize,w_1920)



