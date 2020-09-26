---
layout: post
tag: Developing&Reversing
author: aiQG_
---

# How to Compile Android Kernel on OSX

(在OSX上用clang编译arm64的安卓内核)


## First of All: 一个区分大小写的文件系统

(OSX文件系统不区分大小写, 导致下载/解压时不同文件的覆盖(文件缺少))

**打开磁盘工具, 创建一个区分大小写的APFS宗卷, 所有工作都在这个宗卷里进行.**



## 通过 `repo` 下载源码

[下载方式(Google官方提供)](https://source.android.com/setup/build/building-kernels#downloading)

执行 `build/build.sh` 构建内核 [官方链接](https://source.android.com/setup/build/building-kernels#building)

编译好的东西在 `./out/` 里

### 配置环境

1. 下载交叉编译器
   下载 clang, [不再支持gcc](https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86/+/master/GCC_4_9_DEPRECATION.md#:~:text=Android's%20GCC%204.9%20has%20been,as%20per%20the%20below%20timeline.).

   ```shell
   $ pwd 
   /Path/To/prebuilts-master/clang/host
   $ git clone https://android.googlesource.com/platform/prebuilts/clang/host/darwin-x86
   ```

2. `./build/_setup_env.sh`
替换 `readlink` 为 `greadlink`.
`export ROOT_DIR=$PWD` 改为 `export ROOT_DIR=$(greadlink -f $PWD)`

3. `./build/build.sh`
替换 `readlink` 为 `greadlink`. (可能需要`brew install coreutils`)
替换 `tar` 为 `gtar`.

4. `./private/msm-google/build.config.common`
   替换 `linux-x86` 为 `darwin-x86`. (同时确认所指向的几个路径正确)

5. `private/msm-google-modules/wlan/qcacld-3.0/Makefile`
   ``readlink` 替换为 `greadlink`.

6. `./private/msm-google/Makefile`
   - 给 `HOSTCFLAGS` 变量增加一个路径: `HOSTCFLAGS := -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk`
   - `brew install libelf`. 然后给 `HOSTCFLAGS` 变量增加一个路径: `HOSTCFLAGS := -I/usr/local/include` (解决`'elf.h' file not found`)
   - 将 `sed -e` 改为 `gsed -e`

7. `./private/msm-google/arch/arm64/kernel/vdso/gen_vdso_offsets.sh`
   将
   `'s/^\([0-9a-fA-F]*\) . VDSO_\([a-zA-Z0-9_]*\)$/\#define vdso_offset_\2\t0x\1/p'`
   改为
   `'s/^\([0-9a-fA-F]*\) . VDSO_\([a-zA-Z0-9_]*\)$/\#define vdso_offset_\2  0x\1/p'`
   (将 `\t` 改成空格或制表符)
   (解决 `use of undeclared identifier ‘vdso_offset_sigtramp'`)

8. `./private/msm-google/scripts/Makefile.modpost`
   去掉 `xargs` 后面的 `-r`

9.  `./private/msm-google/scripts/link-vmlinux.sh` 
    将 `stat` 改为 `gstat`

10. `./private/msm-google/scripts/headers_install.sh` 
    将 `sed -r` 改为 `gsed -r`


### 其他错误

1. `'asm/types.h' file not found`
   [mac-linux-headers](https://github.com/vgribov/mac-linux-headers) 按照README增加头文件

2. 缺少dtc/extract_dtb/lz4c/mkdtimg/ufdt_apply_overlay

   > 整理了一些: [AOSP_KernelCompileTools](https://github.com/aiQG/AOSP_KernelCompileTools) 
   

   确认以下文件存在

- `dtc` (`./prebuilts-master/misc/darwin-x86/dtc/dtc`)
  目前只能从ASOP中把编译好的拷贝过去...
  可能有用的链接🔗:
  [Github dgibson/dtc](https://github.com/dgibson/dtc)

- `mkdtimg` ()
  目前只能从ASOP中把编译好的拷贝过去...
  可能有用的链接🔗:
  [libufdt](https://source.android.google.cn/devices/architecture/dto/optimize#libufdt)
  [mkdtimg](https://source.android.com/devices/architecture/dto/partitions#mkdtimg)
  [mkdtimg (DEPRECATED, use mkdtboimg.py instead.)](https://android.googlesource.com/platform/system/libufdt/+/master/utils#mkdtimg-deprecated_use-mkdtboimg_py-instead)

- `extract_dtb` (`./prebuilts-master/misc/darwin-x86/libufdt/extract_dtb`)
  目前只能从ASOP中把编译好的拷贝过去...

- `ufdt_apply_overlay` (`./prebuilts-master/misc/darwin-x86/libufdt/ufdt_apply_overlay`)

  目前只能从ASOP中把编译好的拷贝过去...

- `lz4c` (`./prebuilts-master/misc/darwin-x86/lz4/lz4c`)
  似乎可以下载源代码进行编译`git clone https://android.googlesource.com/platform/external/lz4`; 
  似乎也可以 `brew install lz4`; 
  似乎osx系统会自带? `where lz4c`.


3. `Cannot use CONFIG_CC_STACKPROTECTOR_STRONG: -fstack-protector-strong not supported by compiler`
   (这个报错在我进行了和如下操作(解决 `'elf.h' file not found` 的操作)后就消失了, 没能成功复现)
   `brew install libelf`
   给 `./private/msm-google/Makefile` 中的 `HOSTCFLAGS` 变量增加一个路径:
   `HOSTCFLAGS := -I/usr/local/include`

