实验过程记录
==


下载并构建目标
--

下载并解压缩 tcpdump-4.9.2.tar.gz和libpcap（TCPdump需要的跨平台库）。<br>
**构建并安装 libpcap：**
<br>
```
./configure --enable-shared=no
make
```

开始在执行./configure时会报错<br>
```
configure: error: Neither flex nor lex was found.
```
发现是由于缺少flex bison，于是进行安装：<br>
```
sudo apt-get install flex bison
```

之后成功构建<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/6eab0e0e-0ef6-46ae-a483-7b00c24a2057)
![image](https://github.com/xhsy0314/Task/assets/84487619/1d0725c2-e041-4594-9970-d1c9c8c547ec)

**构建和安装 tcpdump**


```
./configure --prefix="$HOME/fuzzing_tcpdump/install/"
make
make install
```


通过以下命令测试工作是正常的：<br>


```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -h
```
![image](https://github.com/xhsy0314/Task/assets/84487619/21f19143-1f70-4768-bbb7-6c78f37d4bda)


在启用 ASAN 的情况下构建 tcpdump（和 libpcap） 
--

```
cd $HOME/fuzzing_tcpdump/libpcap-1.8.0/
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --enable-shared=no --prefix="$HOME/fuzzing_tcpdump/install/"
AFL_USE_ASAN=1 make

cd $HOME/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/
AFL_USE_ASAN=1 CC=afl-clang-lto ./configure --prefix="$HOME/fuzzing_tcpdump/install/"
AFL_USE_ASAN=1 make
AFL_USE_ASAN=1 make install
```

模糊测试
--



```
afl-fuzz -m none -i /home/xhsy/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/tests/ -o /home/xhsy/fuzzing_tcpdump/out/ -- /home/xhsy/fuzzing_tcpdump/install/sbin/tcpdump @@
```

![image](https://github.com/xhsy0314/Task/assets/84487619/48b77f2a-9d04-4317-ab3d-59692b6a526d)

使用ASAN调试<br>

```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -vvvvXX -ee -nn -r '/home/fuzz/fuzzing_tcpdump/out/default/crashes/id:000000,sig:06,src:009837,time:21830778,execs:8381174,op:havoc,rep:5' 
```

![image](https://github.com/xhsy0314/Task/assets/84487619/ba8841e8-e70a-44c2-8862-cbb49957e57d)<br>


**==4123484==ERROR: AddressSanitizer: heap-buffer-overflow on address 0x60b000000154 at pc 0x0000005cee8f bp 0x7fffff49ec60 sp 0x7fffff49ec58**
    pc 0x0000005cee8f bp 0x7fffff49ec60 sp 0x7ffff f49ec58上的地址0x60b000000154上的堆缓冲区溢出

<br>


修复
--

```
https://github.com/the-tcpdump-group/tcpdump/commit/85078eeaf4bf8fcdc14a4e79b516f92b6ab520fc#diff-05f854a9033643de07f0d0059bc5b98f3b314eeb1e2499ea1057e925e6501ae8L381
```

官方已做出修复，对比可以查看改动。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/396b362b-13ee-4331-80e4-d24c9b342c8f)


