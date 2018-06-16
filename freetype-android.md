# Android使用freetype

> 在华为手机上，如果用户修改了系统字体，应用设置的字体就无效了，用freetype可以自己绘制字体，不受系统限制。后面有个例子工程，亲测在华为手机可行。

freetype可以解析字体文件提取字形，这里给出在安卓上使用freetype的方法。

## 编译freetype

这里提供的编译方式是用freetype源码编译出.a和.so文件，放到安卓工程使用。

### 编译准备

源码下载，[官网](https://www.freetype.org/)有源码下载地址，这里用到的是2.9.1版本。
准备安卓ndk，一般情况下，在Android Studio里面就可以下载到最新的ndk，ndk的目录在`/Users/<username>/Library/Android/sdk/ndk-bundle`，如果是需要其他版本的ndk需要自己到安卓官网下载解压。
解压了源码之后到源码根目录进行编译。
### 编译toolchain
使用下面的命令编译toolchain，`<path-to-ndk>`指ndk路径，`platform`使用的`android-17`，编译出来的toolchain放在`/tmp/ndk/17/`目录下，这个目录是可以随意指定的，下一步会用到
```
<path-to-ndk>/build/tools/make-standalone-toolchain.sh --platform=android-17 --install-dir=/tmp/ndk/17/ --arch=arm
```
### 设置编译环境
将下面`/tmp/ndk/17/`目录修改为上一步toolchain编译输出路径
```shell
export PATH=$PATH:/tmp/ndk/17/bin
export CC=arm-linux-androideabi-gcc
export CXX=arm-linux-androideabi-g++
```
### 配置freetype编译参数
```shell
./configure --host=arm-linux-androideabi --prefix=/freetype --without-zlib --with-png=no --with-harfbuzz=no
```
### 编译
编译出来的文件会在freetype/目录下
```shell
make -j4
make install DESTDIR=$(pwd)
```
### 编译输出
编译成功之后，在freetype目录应该可以看到
```shell
freetype
├── include
│   └── freetype2
├── lib
│   ├── libfreetype.a
│   ├── libfreetype.la
│   ├── libfreetype.so
│   └── ***
└── ***
```
其中include/包含一些头文件，可以直接整体复制到安卓工程对应的jni/目录下，lib/下一个.a和.so复制到安卓工程对应的jni/lib/目录下。如果编译出来的文件有问题，最好是从编译ndk toolchain重新开始。

## 安卓工程配置
这个给出的是使用ndk-build编译的配置方式。
### jni

在src/main目录新建jni目录，将上面编译出的include/目录复制到jni/include/，libfreetype.a和libfreetype.so复制到jni/lib/目录下。在jni目录新建Android.mk和Application.mk文件
```shell
jni
├── Android.mk
├── Application.mk
├── include
│   └── freetype2
└── lib
    ├── libfreetype.a
    └── libfreetype.so
```
### Android.mk

在Android.mk文件里面指定引用libfreetype.a：`LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libfreetype.a`
然后将include/下的文件夹都加到编译里面，大致内容如下：

```makefile
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)

LOCAL_ARM_MODE := arm

LOCAL_MODULE := libfontdecode
LOCAL_SRC_FILES := <your-c/cpp-files>

LOCAL_LDLIBS    := -llog -landroid

LOCAL_LDFLAGS := $(LOCAL_PATH)/lib/libfreetype.a

LOCAL_CFLAGS += -I$(LOCAL_PATH)/include \
                -I$(LOCAL_PATH)/include/freetype2 \
                -I$(LOCAL_PATH)/include/freetype2/freetype \
                -I$(LOCAL_PATH)/include/freetype2/freetype/config \
                -I$(LOCAL_PATH)/include/freetype2/freetype/internal

LOCAL_CPPFLAGS += -I$(LOCAL_PATH)/include \
                  -I$(LOCAL_PATH)/include/freetype2 \
                  -I$(LOCAL_PATH)/include/freetype2/freetype \
                  -I$(LOCAL_PATH)/include/freetype2/freetype/config \
                  -I$(LOCAL_PATH)/include/freetype2/freetype/internal

include $(BUILD_SHARED_LIBRARY)
```
Application.mk如下，没有什么需要做的
需要注意的是ndk17之后不支持armeabi编译了，需要改为armeabi-v7a。
```
APP_ABI := armeabi
APP_PLATFORM := android-17

APP_STL := stlport_static
```
### build.gradle

在build.gradle的android标签下增加ndk编译配置，在defaultConfig标签下指定编译的架构

```groovy
android {
    ...
    defaultConfig {
        ...
        ndk {
            abiFilters 'armeabi', 'armeabi-v7a'
        }
    }
    ...
    externalNativeBuild {
        ndkBuild {
            ndkBuild {
                path "src/main/jni/Android.mk"
            }
        }
    }
}
```

### 编译

在jni/src/目录添加自己的c或者cpp文件，替换Android.mk里面`<your-c/cpp-files>`。在jni目录下执行`<path-to-ndk>/ndk-build`就会在`../libs/`下生成需要的so文件。



[一个提取文字路径到Path例子](https://github.com/iamjinge/AndroidFreetypeSample)