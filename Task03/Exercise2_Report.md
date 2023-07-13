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
![image](https://github.com/xhsy0314/Task/assets/84487619/a3588ed0-35ef-46a4-9ca1-10d2fd1d1a76)

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
