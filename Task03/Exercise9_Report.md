实验过程
==

**环境：** 由于本机操作系统就为win10，因此没有使用镜像，直接用本机进行实验。

<br>

下载并构建 WinAFL
-

<ul>
  <li>
    安装Visual Studio编译器:
      <p>之前已安装过Visual Studio 2019，在这里直接使用。</p>
  </li>
  <li>
    安装DynamoRIO 8.0.0:下载DynamoRIO Windows二进制包并提取文件内容。<br>
    <img src="https://github.com/xhsy0314/Task/assets/84487619/45ea0ec9-d2ab-43c4-97ca-a13d142477d6" alt="some_text">
  </li>
  <li>
    下载 WinAFL：<https://github.com/googleprojectzero/winafl>
      <p>解压后进入winafl目录，并打开 "Developer Command Prompt for VS2019"控制台，切换到winafl目录中。输入以下命令：(注意要换成自己的DynamoRIO路径)
      
    mkdir build32
    cd build32
    cmake -G"Visual Studio 16 2019" -A Win32 .. -DDynamoRIO_DIR=D:\fuzz\DynamoRIO-Windows-8.0.0-1\cmake
    cmake --build . --config Release
      
  </p>

  发现使用CMake为Visual Studio生成工程项目时，会报如下错误：<br>
      <img src="https://github.com/xhsy0314/Task/assets/84487619/7f2cf675-2d02-48fb-a607-171456e666a1" alt="2">

<br>
<b>错误一：</b>本机的Cmake版本3.27.1，但是这里要求的版本太低了，因此修改了CMakeLists.txt文件：<br>
      <img src="https://github.com/xhsy0314/Task/assets/84487619/a36287f5-4566-482f-a1d3-93aa2efe7ee5" alt="3">
  <br>
<b>错误二：</b>
    <p>
      
          CMake Error at CMakeLists.txt:3 (project):
          Generator
          Visual Studio 15 2017
          could not find any instance of Visual Studio.
          Configuring incomplete, errors occurred!
  </p>

  这个提示很明确，就是找不到编译平台的问题。查找网上的方法没有头绪，还有人说卸载重新安装，更有甚者说重装系统。很久没有解决。
考虑可能是有些功能没有正确安装，所以利用Visual Studio Installer进行修复。<br>
首先打开<b>Visual Studio Installer</b>,发现是 <b>"Desktop development with C++"</b> package没有安装，因此勾选上这个包后进行安装，问题得以解决。<br>

  <img src="https://github.com/xhsy0314/Task/assets/84487619/749f30a9-1454-444c-9bcf-58cd80568797" alt="4">

排除所有错误之后，再执行上述命令，在文件夹 <b>winafl-master\build32\bin\Release</b> 中可以找到所有 WinAFL 二进制文件：<br>
  <img src="https://github.com/xhsy0314/Task/assets/84487619/69c8a2d2-1879-4f9c-81ff-7ffa1292c238" alt="5">

  </li>
</ul>

安装 7-Zip
---
下载安装包，安装路径为：**C:\Program Files (x86)\7-Zip\7z.exe**<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/0ed1e130-18da-4727-a95a-987cce4967d6)

<br>

创建HFS+图像
-

在afl_in文件夹中创建一些img文件：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/540df40a-64a1-497e-ad09-434c1921f0c8)

模糊测试
--

WinAFL 命令行与 AFL++ 略有不同。有一些新options：

    -coverage_module：用于记录覆盖范围的模块。支持多个模块标志
    -target_module：包含要模糊测试的目标函数的模块
    -target_offset：从模块开始模糊的方法的偏移量

这里选择0x02F3B3作为--target_offset。测试目标是否在 DynamoRIO 下正常运行：

```
D:\fuzz\DynamoRIO-Windows-8.0.0-1\bin32\drrun.exe -c winafl.dll -debug -target_module 7z.exe -target_offset 0x02F3B3 -fuzz_iterations 10 -nargs 2 -- "C:\Program Files (x86)\7-Zip\7z.exe" l D:\fuzz\example.img
```

运行该命令后看到与目标函数对应的输出正在运行 10 次，之后目标可执行文件将退出。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/47b98a11-5287-4bfa-98c3-e6be88700355)
<br>
运行模糊器：

```
afl-fuzz.exe -i D:\fuzz\afl_in -o D:\fuzz\afl_out -t 2000 -D D:\fuzz\DynamoRIO-Windows-8.0.0-1\bin32 -- -coverage_module 7z.exe -coverage_module 7z.dll -target_module 7z.exe -target_offset 0x02F3B3 -nargs 2 -- "C:\Program Files (x86)\7-Zip\7z.exe" e -y @@
```
![image](https://github.com/xhsy0314/Task/assets/84487619/0437442a-3cfc-4da3-b90d-10230e9fdfd9)<br>
可以看到winafl运行。

总结
-

在该实验中我了解了使用 WinAFL fuzz Windows 应用程序的方法，并在win10系统中下载并构建了 WinAFL（以及所需的依赖项），下载了 HFS+ ，并创建样本的种子语料库。运行winafl时了解了WinAFL 命令行与 AFL++ 的不同，明确了几个新options的含义。并且能够通过7-ZIP和DynamoRIO成功运行目标。下一步应该找到更好的target_offset来加快模糊测试过程。

