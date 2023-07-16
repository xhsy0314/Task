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


在启用 ASAN 的情况下构建 tcpdump（和 libpcap） 进度
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


未完.


