实验过程
==

下载并构建目标
--

构建并安装 libxml2：
```
sudo apt-get install python-dev
CC=afl-clang-lto CXX=afl-clang-lto++ CFLAGS="-fsanitize=address" CXXFLAGS="-fsanitize=address" LDFLAGS="-fsanitize=address" ./configure --prefix="$HOME/Fuzzing_libxml2/libxml2-2.9.4/install" --disable-shared --without-debug --without-ftp --without-http --without-legacy --without-python LIBS='-ldl'
make -j$(nproc)
make install
```

测试:


```
./xmllint --memory ./test/wml.xml
```


![image](https://github.com/xhsy0314/Task/assets/84487619/e4922d5b-b99a-41ac-a673-8b2d6ba0b45b)

<br>
看到图片中信息说明当前工作正常。<br>

使用 AFL++ 提供的 XML 字典:

```
mkdir dictionaries && cd dictionaries
wget https://raw.githubusercontent.com/AFLplusplus/AFLplusplus/stable/dictionaries/xml.dict
cd ..
```

**字典：**

    当我们想要模糊复杂的基于文本的文件格式（如 XML）时，为模糊器提供一个包含基本语法标记列表的字典很有用。
    在 AFL 的情况下，这样的字典只是一组单词或值，AFL 使用它来将更改应用于当前内存中的文件。具体而言，AFL 使用字典中提供的值执行以下更改：

    覆盖：将特定位置替换为 n 个字节，其中 n 是字典条目的长度。
    插入：在当前文件位置插入字典条目，强制所有字符向下移动 n 个位置并增加文件大小。

模糊测试
-

**并行化模糊测试**

    独立实例：在这种模式下，运行完全独立的 afl-fuzz 实例。

    并行实例：使用共享实例是并行模糊测试的更好方法。在这种情况下，每个模糊器实例都会收集其他模糊器找到的任何测试用例。AFL 使用非确定性测试算法。因此，如果我们运行多个 AFL 实例，我们会增加成功的机会。
为此，只需要在多个终端窗口上运行多个“afl-fuzz”实例，为每个实例设置不同的“输出文件夹”。一种简单的方法是运行与系统上内核一样多的模糊测试作业。

通常一次只有一个主实例：

```
./afl-fuzz -i afl_in -o afl_out -M Master -- ./program @@
```

和 N-1 个从属设备数量：

```
./afl-fuzz -i afl_in -o afl_out -S Slave1 -- ./program @@
./afl-fuzz -i afl_in -o afl_out -S Slave2 -- ./program @@
...
./afl-fuzz -i afl_in -o afl_out -S SlaveN -- ./program @@
```


**主模糊器：**

```
afl-fuzz -m none -i ./afl_in -o afl_out -s 123 -x ./dictionaries/xml.dict -D -M master -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

     -x 标志设置字典路径
     -D 标志启用确定性突变（仅适用于主模糊器）

**从属模糊器：**

```
afl-fuzz -m none -i ./afl_in -o afl_out -s 234 -S slave1 -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

![image](https://github.com/xhsy0314/Task/assets/84487619/cb0bbcaa-6622-42a2-8a95-133310a551b1)

<br>
运行了大概29个小时slave出现crash...master未出现。

调试
-

提供crash文件，使用ASAN调试。

```
./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude './afl_out/slave1/crashes/id:000000,sig:11,src:009907,time:100983886,execs:70372968,op:havoc,rep:4'
```

![image](https://github.com/xhsy0314/Task/assets/84487619/b5f6e909-aa34-4fa7-ab32-523dd46a2dab)<br>

跟预期收到的结果好像不太一样，需要进一步调整。

修复
--

![image](https://github.com/xhsy0314/Task/assets/84487619/4f66060a-7a12-4891-86dc-3ef6c33e3ecd)<br>

xmlSnprintfElementContent未能正确检查可用的两个位置的缓冲空间，修复了xmlSnprintfElementContent中的缓冲区大小检查。

总结
--

在实验五中主要完成了以下工作：

    1.查找使用 LibXML2 库的接口应用程序；
    2.将 SampleInput.xml 文件复制到 AFL 输入文件夹；
    3.为模糊测试XML创建自定义词典;
    4.模糊 LibXML2，直到出现崩溃。使用 并行AFL 实例；
    5.修复问题
    
主要学习了使用自定义词典帮助模糊器查找新的执行路径，以及并行化模糊测试的方法。但在调试过程中出现了问题，后续会尝试多种方法继续进行调试。
