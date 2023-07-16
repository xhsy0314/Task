实验过程
==

构建和安装 libtiff
--

```
cd tiff-4.0.4/
./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```

使用示例图像：<br>

```
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff
```

![image](https://github.com/xhsy0314/Task/assets/84487619/d455c75a-fe47-4a7c-9961-8b983e5013d7)

    参数：-j -c -r -s -w
    作用：提高代码覆盖率并增加发现错误的机会。

<br>

**代码覆盖率**:显示每行代码被触发的次数。通过使用代码覆盖率，我们将了解模糊器到达了代码的哪些部分，并可视化模糊处理过程。

    安装 lcov;
    重建 libTIFF;
    收集代码覆盖率数据。

```
cd $HOME/fuzzing_tiff/tiff-4.0.4/   
lcov --zerocounters --directory ./         #重置以前的计数器
lcov --capture --initial --directory ./ --output-file app.info    #返回“基线”覆盖率数据文件，其中包含每个检测行的零覆盖率
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff    #运行要分析的应用程序。您可以使用不同的输入多次运行它
lcov --no-checksum --directory ./ --capture --output-file app2.info    #将当前覆盖状态保存到 app2.info 文件中
```

![image](https://github.com/xhsy0314/Task/assets/84487619/e3274508-2493-4af9-81a7-f44ba93bb574)

<br>

但目前存在一些问题，使当前覆盖状态无法保存到文件里，从而无法生成html，需要找出原因并改进。

模糊测试
--

在启用 ASAN 的情况下编译 libtiff，在调用make之前设置AFL_USE_ASAN=1：

```
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
AFL_USE_ASAN=1 make -j4
AFL_USE_ASAN=1 make install
```

运行模糊程序：

```
afl-fuzz -m none -i $HOME/fuzzing_tiff/tiff-4.0.4/test/images/ -o $HOME/fuzzing_tiff/out/ -s 123 -- $HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w @@
```
![image](https://github.com/xhsy0314/Task/assets/84487619/47d3fd3a-b611-4449-b6c9-fdab96e4a2b1)

<br>
几分钟后出现crash。<br>

ASan 跟踪crash:<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/d147944a-754f-4a97-a25a-7c101e0a9f7c)<br>

可以看到存在堆溢出。


修复
--

![image](https://github.com/xhsy0314/Task/assets/84487619/6f09d39f-727e-4d64-b585-696fa835c907)<br>

<br>
经过官方diff可以看到做出的改进。其中，在 TIFFFetchNormalTag（）中，确保
带有 TIFF_SETGET_C16_ASCII / TIFF_SETGET_C32_ASCII 的标签的值
访问以空终止，以避免在TIFFPrintField（） 中有潜在的外部缓冲区读取。

总结
--

通过实验四了解了使用LCOV测量代码覆盖率的基本流程，以及如何使用代码覆盖率数据提高模糊测试的有效性。对于启用ASAN的情况下进行编译的这一过程也更加熟悉了，之后应尝试对错误进行修正。

