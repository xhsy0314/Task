实验过程
==

下载并构建目标
--

下载并构建GEGL 0.2（通用图形库）：

```
wget https://download.gimp.org/pub/gegl/0.2/gegl-0.2.0.tar.bz2
tar xvf gegl-0.2.0.tar.bz2 && cd gegl-0.2.0
```

对源代码进行两个小的更改：

```
sed -i 's/CODEC_CAP_TRUNCATED/AV_CODEC_CAP_TRUNCATED/g' ./operations/external/ff-load.c
sed -i 's/CODEC_FLAG_TRUNCATED/AV_CODEC_FLAG_TRUNCATED/g' ./operations/external/ff-load.c
```

构建并安装 Gegl-0.2：

```
./configure --enable-debug --disable-glibtest  --without-vala --without-cairo --without-pango --without-pangocairo --without-gdk-pixbuf --without-lensfun --without-libjpeg --without-libpng --without-librsvg --without-openexr --without-sdl --without-libopenraw --without-jasper --without-graphviz --without-lua --without-libavformat --without-libv4l --without-libspiro --without-exiv2 --without-umfpack
make -j$(nproc)
sudo make install
```

![image](https://github.com/xhsy0314/Task/assets/84487619/9cc9849b-1d89-452a-9d39-44977308ee9f)

AFL persistent_mode：
--

AFL persistent_mode 基于进程内模糊器：使用单个进程的模糊程序，将代码注入目标进程并更改内存中的输入值。该模式将进程内模糊测试的优点与更传统的多进程工具的稳健性相结合：持久模式。
在持久模式下，AFL++ 在单个分叉进程中多次模糊目标，而不是为每个模糊执行分叉新进程。此模式可将模糊测试速度提高多达 20 倍。
<br>
具体方法：修改app.c文件并将AFL_LOOP宏包含在 for 循环中：<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/4be88acd-051b-4817-986a-8b1e75f9acb6)<br>

该方法可以针对不同的输入格式。

模糊测试
--

运行模糊程序：

```
ASAN_OPTIONS=detect_leaks=0,abort_on_error=1,symbolize=0 afl-fuzz -i './afl_in' -o './afl_out' -D -t 100 -- ./install/bin/gimp-console-2.8 --verbose -d -f @@
```

注意事项：

    gimp-console-2.8是 GIMP 的仅限控制台版本
    建议启用确定性突变 （-D）
    代码中还有一个无限循环错误，因此需要设置一个较低的超时限制（例如 -t 100）。

![image](https://github.com/xhsy0314/Task/assets/84487619/df2e5f66-e23c-40a6-8a09-64f80655aeda)<br>

如图运行了约40h也没跑出crash，需要检查是不是准备工作没做好导致配置等方面出现问题...
