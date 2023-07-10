Exercise 1 - Xpdf
==
下载并构建目标
-
下载并构建Xpdf：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/04575ea4-2ca3-4de4-81fb-38013a3a8655)<br>

初步认识 AFL++
-


**安装AFL++**
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/254f9a09-2b03-4328-accc-1db8769fe168)<br>

使用 afl-clang-fast 编译器构建 xpdf时由于我的AFL++安装在afl-2.52b下，因此CC和CCX参数的设置也需要修改：<br>

```
export LLVM_CONFIG="llvm-config-11"
CC=$HOME/afl-2.52b/afl-clang CXX=$HOME/afl-2.52b/afl-clang++ ./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```
运行模糊程序：<br>
```
afl-fuzz -i $HOME/fuzzing_xpdf/pdf_examples/ -o $HOME/fuzzing_xpdf/out/ -s 123 -- $HOME/fuzzing_xpdf/install/bin/pdftotext @@ $HOME/fuzzing_xpdf/output
```
对于这条指令运行时会报错，提示不存在s这一参数，考虑到可能是AFL版本原因，我就将-s 123删去了。之后正常运行：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/c4b42436-5da3-45ef-9ae4-efff5b676147)

机器性能真的低，运行了快一个小时才找到第一个crash..<br>

**分流**<br>

运行gdb后输入run执行，<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/e74fa317-466d-43ed-8a8e-31cabde00ef5)<br>
输入bt获取回溯：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/735a4970-1df3-4c61-87df-a347ebf2ad4b)<br>

**修复**<br>
根据报错信息可知，可能存在内存泄漏，下载 Xpdf 4.02源码对比来看。观察获取的回溯，glibc报错，报错位置在malloc.c:3679。根据执行流信息，由分析可以看出调用过程是循环的，判断为无限循环漏洞；
根据xpdf/Parse.cc 94行的makeStream调用找到漏洞位置。

未完...
