实验过程
==

下载并构建目标
--

构建并安装 libexif：
```
cd libexif-libexif-0_6_14-release/
sudo apt-get install autopoint libtool gettext libpopt-dev
autoreconf -fvi
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/"
make
make install
```

构建并安装 exif 命令行实用程序
--

```
cd exif-exif-0_6_15-release/
autoreconf -fvi
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/" PKG_CONFIG_PATH=$HOME/fuzzing_libexif/install/lib/pkgconfig
make
make install
```

通过该命令看到如下图片中信息则正常。<br>

```
$HOME/fuzzing_libexif/install/bin/exif
```
![image](https://github.com/xhsy0314/Task/assets/84487619/bdd4590f-e9b3-4c57-99e7-2d88992d229c)

获取exif样本
--

执行操作(举例)：

```
$HOME/fuzzing_libexif/install/bin/exif $HOME/fuzzing_libexif/exif-samples-master/jpg/Canon_40D_photoshop_import.jpg
```

![image](https://github.com/xhsy0314/Task/assets/84487619/fa6d7f05-1ede-4c0c-9d40-bd3063e32bcf)


 afl-clang-lto 作为编译器构建 libexif
 --

```
 rm -r $HOME/fuzzing_libexif/install
cd $HOME/fuzzing_libexif/libexif-libexif-0_6_14-release/
make clean
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/"
make
make install
```

```
cd $HOME/fuzzing_libexif/exif-exif-0_6_15-release
make clean
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/" PKG_CONFIG_PATH=$HOME/fuzzing_libexif/install/lib/pkgconfig
make
make install
```

模糊测试
--

运行模糊程序：<br>

```
afl-fuzz -i $HOME/fuzzing_libexif/exif-samples-master/jpg/ -o $HOME/fuzzing_libexif/out/ -s 123 -- $HOME/fuzzing_libexif/install/bin/exif @@
```

![image](https://github.com/xhsy0314/Task/assets/84487619/f71d33d0-9542-493b-892c-0f260c3d6342)

出现多次崩溃。


Eclipse工具的应用
--

下载eclipse并解压，进行以下配置<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/ee3966be-2662-48a4-aa64-776b390dbd62)

点击"Debug"启动调试功能，程序就会在主函数的开头停止。点击Run -> Resume，当检测到分段错误时，执行就会停止。<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/2c4ff96c-4709-4f11-9998-9b51312d6b6a)

总结
--

在本实验中，学习了使用 afl-clang-lto 作为编译器来构建 libexif，并且明确了使用afl-clang-lto而不是afl-clang-fast的原因。另外，对模糊测试的步骤更加熟悉了。在上个实验中学习了用gdb调试工具来对crash进行追踪、回溯过程，对于本实验的crash会尝试用gdb进行分析。
