---
layout: post
title: Linux 动态库 undefined symbol 原因定位与解决方法
date: 2020-05-24
categories:
- linux
tags: [linux]
---

在使用动态库开发部署时，遇到最多的问题可能就是 undefined symbol 了，导致这个出现这个问题的原因有多种多样，快速找到原因，采用对应的方法解决是本文写作的目的。

### 可能的原因

#### 1. 依赖库未找到  
  这是最常见的原因，一般是没有指定查找目录，或者没有安装到系统查找目录里

#### 2. 链接的依赖库不一致  
  编译的时候使用了高版本，然后不同机器使用时链接的却是低版本，低版本可能缺失某些 api 

#### 3. 符号被隐藏  
  如果动态库编译时被默认隐藏，外部代码使用了某个被隐藏的符号。

#### 4. c++ abi 版本不一致  
  最典型的例子就是 gcc 4.x 到 gcc 5.x 版本之间的问题，在  4.x 编辑的动态库，不能在 5.x 中链接使用。

### 解决方法

#### 1. 依赖库未找到
  - 使用 ldd -r <lib-file-name>, 确定系统库中是否存在所依赖的库
  - 执行  ldconfig 命令更新 ld 缓存
  - 执行 ldconfig -p | grep {SO_NAME} 查看是否能找到对应的库
  - 检查 LD_LIBRATY_PATH 是否设置了有效的路径

#### 2. 链接的库版本不一致

  如果系统中之前有安装过相同的库，或者存在多个库，就需要确定链接的具体是哪个库

  有一个特殊场景需要注意下，.so 文件中有个默认 rpath 路径，用于搜索被依赖的库，这个路径优先于系统目录和LD_LIBRARY_PATH。假如 rpath 存在相同名字的 .so 文件，会优先加载这个路径的文件。

在遇到 undefined symbol 问题时，使用 `readelf -d <lib-file> | grep - i rpath` 查看:

```
$ readelf -d libSXVideoEngineJni.so | grep rpath
 0x000000000000000f (RPATH)              Library rpath: [/home/slayer/workspace/SXVideoEngine-Core/Render/cmake-build-debug:/home/slayer/workspace/SXVideoEngine-Core/Render/../../SXVideoEngine-Core-Lib/blend2d/linux/lib]
```

如果存在的路径中有相应的库，可以先重命名文件再测试确认。

关于连接时的顺序可以查看文档： http://man7.org/linux/man-pages/man8/ld.so.8.html

```
   If a shared object dependency does not contain a slash, then it is
   searched for in the following order:

   o  Using the directories specified in the DT_RPATH dynamic section
      attribute of the binary if present and DT_RUNPATH attribute does
      not exist.  Use of DT_RPATH is deprecated.

   o  Using the environment variable LD_LIBRARY_PATH, unless the
      executable is being run in secure-execution mode (see below), in
      which case this variable is ignored.

   o  Using the directories specified in the DT_RUNPATH dynamic section
      attribute of the binary if present.  Such directories are searched
      only to find those objects required by DT_NEEDED (direct
      dependencies) entries and do not apply to those objects' children,
      which must themselves have their own DT_RUNPATH entries.  This is
      unlike DT_RPATH, which is applied to searches for all children in
      the dependency tree.

   o  From the cache file /etc/ld.so.cache, which contains a compiled
      list of candidate shared objects previously found in the augmented
      library path.  If, however, the binary was linked with the -z
      nodeflib linker option, shared objects in the default paths are
      skipped.  Shared objects installed in hardware capability
      directories (see below) are preferred to other shared objects.

   o  In the default path /lib, and then /usr/lib.  (On some 64-bit
      architectures, the default paths for 64-bit shared objects are
      /lib64, and then /usr/lib64.)  If the binary was linked with the
      -z nodeflib linker option, this step is skipped.
```

#### 3. 符号被隐藏

第三方已经编译好的库，在引入了对应的头文件，使用了其中的某个方法，最终链接的时候出现 undefined symbol，这种情况有可能是库的开发者并没有导出这个方法的符号。

```
# 使用 nm 命令查看导出的函数符号， 这里查看 License 相关的函数
$ nm -gDC libSXVideoEngineJni.so | grep -i license
0000000000008110 T __ZN13SXVideoEngine6Public7License10SetLicenseEPKc
0000000000008130 T __ZN13SXVideoEngine6Public7License13LicenseStatusEv
0000000000008190 T __ZN13SXVideoEngine6Public7License19IsVideoCutSupportedEv
0000000000008170 T __ZN13SXVideoEngine6Public7License26IsDynamicTemplateSupportedEv
0000000000008150 T __ZN13SXVideoEngine6Public7License26IsStadardTemplateSupportedEv

# nm 返回的并不是原始函数名，通过 c++filt 获取原始名称
$ c++filt __ZN13SXVideoEngine6Public7License10SetLicenseEPKc
SXVideoEngine::Public::License::SetLicense(char const*)

```

#### 4. c++ Abi 版本不一致   

Gcc 对 c++ 的新特性是一步一步的增加的，如果实现了新的特性，就可能会修改 c++ 的 abi，并且会升级 glibc 的版本。

Abi 链接最常见的错误是 std::string 和 std::list 的在gcc 4.x 和 gcc 5.x 的不同实现引起的。在gcc 4.x 时，gcc 对标准 string 的实现就放在 std 命名空间下，编译时展开为 std::basic_string 。但是 gcc 5.x 开始，对 string 的实现就放在了 std::__cxx11空间里，编译后展开为 std::__cxx11::basic_string 。这就会导致在 gcc 4.x 编译的动态库，假如有的函数使用了 string 作为参数或者返回值，这时导出的函数参数为 std::basic_string 类型。 无法在 gcc 5.x 下编译连接使用。  
错误类似：

```
undefined symbol:  "std::__cxx11 ***"
```

这种情况有一个折中办法就是在gcc 5.x 或以上 编译时，增加 `-D_GLIBCXX_USE_CXX11_ABI=0` 禁用 c++11 abi。

当然最好的做法就是保证编译器大版本基本一致。在新开发的程序如果用到了 c++ 的新特性，升级 gcc 版本和 glibc 是十分必要的。


### 实用命令总结

#### ldd 命令，用于查找某个动态库所依赖的库是否存在

```
# ldd -r <lib/excutable file> 
# 找不到的库会出现 not found
$ ldd -r libSXVideoEngine.so
        linux-vdso.so.1 =>  (0x00007ffc337d2000)
        libz.so.1 => /lib64/libz.so.1 (0x00007f061cf41000)
        libX11.so.6 => /lib64/libX11.so.6 (0x00007f061cc03000)
        libEGL.so.1 => /lib64/libEGL.so.1 (0x00007f061c9ef000)
        libGLESv2.so.2 => /lib64/libGLESv2.so.2 (0x00007f061c7dd000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x00007f061c5c1000)
        libblend2d.so => /home/seeshion/workspace/SXVideoEngine-Core/Render/../../SXVideoEngine-Core-Lib/blend2d/linux/lib/libblend2d.so (0x00007f061c187000)
        libfreeimage.so.3 => /lib/libfreeimage.so.3 (0x00007f061b8ac000)
        libavcodec.so.58 => /lib/libavcodec.so.58 (0x00007f06198b6000)
        libavformat.so.58 => /lib/libavformat.so.58 (0x00007f06193e1000)
        libavutil.so.56 => /lib/libavutil.so.56 (0x00007f06190bd000)
        ...
```

#### nm 命令，用于读取库被导出的符号

```
$ nm -gDC libSXVideoEngineJni.so | grep -i license
0000000000008110 T __ZN13SXVideoEngine6Public7License10SetLicenseEPKc
0000000000008130 T __ZN13SXVideoEngine6Public7License13LicenseStatusEv
0000000000008190 T __ZN13SXVideoEngine6Public7License19IsVideoCutSupportedEv
0000000000008170 T __ZN13SXVideoEngine6Public7License26IsDynamicTemplateSupportedEv
0000000000008150 T __ZN13SXVideoEngine6Public7License26IsStadardTemplateSupportedEv
```


#### readelf 用于读取 elf 文件的相关信息

```
$ readelf -d libSXVideoEngineJni.so | grep rpath
0x000000000000000f (RPATH)              Library rpath: [/home/slayer/workspace/SXVideoEngine-Core/Render/cmake-build-debug:/home/slayer/workspace/SXVideoEngine-Core/Render/../../SXVideoEngine-Core-Lib/blend2d/linux/lib]
```

#### c++filt 用于获取符号的原始名

```
$ c++filt __ZN13SXVideoEngine6Public7License10SetLicenseEPKc
SXVideoEngine::Public::License::SetLicense(char const*)
```
