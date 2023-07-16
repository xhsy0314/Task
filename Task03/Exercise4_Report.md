下载并构建目标
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
