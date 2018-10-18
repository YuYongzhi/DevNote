#编译IJKPlayer so
基于 [Bilibili] (https://github.com/Bilibili/ijkplayer) 的开源`ijkplayer`替换MediaPlayer。

# 以下以编译　`Android`为例

### 1.  直接使用
    allprojects {
        repositories {
            jcenter()
        }
    }
    
    dependencies {
        # required, enough for most devices.
        compile 'tv.danmaku.ijk.media:ijkplayer-java:0.8.8'
        compile 'tv.danmaku.ijk.media:ijkplayer-armv7a:0.8.8'

        # Other ABIs: optional
        compile 'tv.danmaku.ijk.media:ijkplayer-armv5:0.8.8'
        compile 'tv.danmaku.ijk.media:ijkplayer-arm64:0.8.8'
        compile 'tv.danmaku.ijk.media:ijkplayer-x86:0.8.8'
        compile 'tv.danmaku.ijk.media:ijkplayer-x86_64:0.8.8'

        # ExoPlayer as IMediaPlayer: optional, experimental
        compile 'tv.danmaku.ijk.media:ijkplayer-exo:0.8.8'
    }
>　
> **ijkplayer默认不支持HTTPS** 
> 　

是的，**不支持**，如果你尝试使用ijkplayer播放Https开头的音频，会报这样的错误：

```
E/IJKMEDIA: ****** : Protocol not found
E/tv.danmaku.ijk.media.player.IjkMediaPlayer: Error (-10000,0)
```
解决方案只有重新编译ijkplayer 的源码，没有其他选择。

### 2. 编译环境
* macOS - `Mojava 10.14`
* NDK - `r10e` [下载链接](https://dl.google.com/android/repository/android-ndk-r10e-darwin-x86_64.zip
)
* git - `2.19.0`

### 3. 环境配置

```
ANDROID_HOME="/data/Android/Sdk"
export PATH=$PATH:$ANDROID_HOME/build-tools:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools
ANDROID_NDK=/data/Android/Ndk
export PATH=$PATH:$ANDROID_NDK
```

测试： `ndk-build -v`，输出如下内容则配置成功

```
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.

This program built for x86_64-apple-darwin
```
### 4. 拉取 ijkplayer 源码

```
git clone https://github.com/Bilibili/ijkplayer.git
```

### 5. 初始化 android

```
cd /data/ijkplayer
```
* 修改配置
     
    ```
    vim init-android.sh
    ```
    把 `IJK_FFMPEG_COMMIT=ff3.4--ijk0.8.7--20180103--001`修改成`IJK_FFMPEG_COMMIT=ff4.0--ijk0.8.25--20180926--001`
    
    最新分支可根据如下命令查看：
    
    ```
    git ls-remote --tag https://github.com/Bilibili/FFmpeg.git
    ```
    从结果中查找类似 `ff4.0--ijk*`的tag，找出最新的分支，目前最新的为`ff4.0--ijk0.8.25--20180926--001`。
* 初始 android

```
./init-android.sh
./init-android-openssl.sh   # 支持https
```

### 6. 编辑脚本配置
* 官方提供了三个配置文件模板
     
    ```
    module-default.sh # 默认配置
    module-lite-hevc.sh # 较小的二进制大小的编解码器/格式（包括hevc功能）
    module-lite.sh  # 较小的二进制大小的编解码器/格式（默认）
    ``` 
* 修改配置文件，默认的配置文件为`config/module-lite.sh`
    * 删除 `--disable-ffserver`，如下：
        
        把
        
        ```
        export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-ffserver"
        ```
        修改成
        
        ```
        # export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-ffserver"
        ```
    * 删除 `--disable-vda`，如下：
        
        把
        
        ```
        export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-vda"
        ```
        
        修改成
        
        ```
        # export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-vda"
        ``` 
    * 增加 `--disable-x86asm`，如下：
        
        ```
        export COMMON_FF_CFG_FLAGS="$COMMON_FF_CFG_FLAGS --disable-x86asm"
        ```

* 注释所有 `eac3_core_bsf.c` 文件中的相关代码，
    > 目前在代码标签为 `ff4.0--ijk0.8.25--20180926--001`时，会编译报错，注释后会编译成功，但不清楚是否会有什么影响
    
    * 查找 `eac3_core_bsf.c` 文件并编辑
        
        ```
        cd android/contrib
         find . -name "eac3_core_bsf.c"
        ``` 
        会找到5个文件，分别对应不同 CPU 的文件，如下
        
        ```
        ./ffmpeg-x86_64/libavcodec/eac3_core_bsf.c
        ./ffmpeg-armv7a/libavcodec/eac3_core_bsf.c
        ./ffmpeg-x86/libavcodec/eac3_core_bsf.c
        ./ffmpeg-armv5/libavcodec/eac3_core_bsf.c
        ./ffmpeg-arm64/libavcodec/eac3_core_bsf.c
        ```
        
        依次编辑每个文件（以 `armv7a` 为例）：
        
        ```
        vim ./ffmpeg-armv7a/libavcodec/eac3_core_bsf.c
        ```
        分别注释文件中的两段代码：
        * 第一段代码（大概为`L:39-43`）：
        
        ```
        ret = ff_ac3_parse_header(&gbc, &hdr);
        if (ret < 0) {
            ret = AVERROR_INVALIDDATA;
            goto fail;
        }
        ```
        * 第二段代码（大概为 `L:55-59`）：
        
        ```
        ret = ff_ac3_parse_header(&gbc, &hdr2);
        if (ret < 0) {
            ret = AVERROR_INVALIDDATA;
            goto fail;
        }
        ```  


### 7. 清除一波

```
cd android/contrib
./compile-openssl.sh clean
./compile-ffmpeg.sh clean
```
### 8. 编译 openssl

```
./compile-openssl.sh all
```
### 9. 编译 ffmpeg

```
./compile-ffmpeg.sh all
```
### 10. 编译 ijkplayer

```
./compile-ijk.sh all
```



