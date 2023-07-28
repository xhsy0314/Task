实验过程
=

QEMU安装
-

```
sudo apt install ninja-build libc6-dev-i386
cd ~/Downloads/AFLplusplus/qemu_mode/
CPU_TARGET=i386 ./build_qemu_support.sh
make distrib
sudo make install
```

安装完后执行命令验证是否安装成功：

```
afl-qemu-trace --help
```

![image](https://github.com/xhsy0314/Task/assets/84487619/5dfc7f85-12bf-4a12-bc6e-57d5ec2f71c2)<br>

显示上图内容表示已安装QEMU。

Adobe Reader 安装
-

安装依赖项：

```
sudo apt-get install libxml2:i386
```

下载并解压缩：AdbeRdr9.5.1-1_i386linux_enu.deb

```
wget ftp://ftp.adobe.com/pub/adobe/reader/unix/9.x/9.5.1/enu/AdbeRdr9.5.1-1_i386linux_enu.deb
```

执行命令安装Adobe Reader：

```
sudo dpkg -i AdbeRdr9.5.1-1_i386linux_enu.deb
```

运行/opt/Adobe/Reader9/bin/目录下的acroread文件，会进入Adobe Reader 界面。<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/2d68478a-f2fd-4b54-804a-621bd705ca54)

模糊测试
--

**方法一**:使用 -Q 参数运行 afl-fuzz<br>

注意：二进制文件路径并非上面的 **/opt/Adobe/Reader9/bin/acroread**，而是 **/opt/Adobe/Reader9/Reader/intellinux/bin/acroread**。

```
ACRO_INSTALL_DIR=/opt/Adobe/Reader9/Reader ACRO_CONFIG=intellinux LD_LIBRARY_PATH=$LD_LIBRARY_PATH:'/opt/Adobe/Reader9/Reader/intellinux/lib' afl-fuzz -Q -i ./afl_in/ -o ./afl_out/ -t 2000 -- /opt/Adobe/Reader9/Reader/intellinux/bin/acroread -toPostScript @@
```

![30c5e48fcbce3b695463016ce292016](https://github.com/xhsy0314/Task/assets/84487619/d033b202-aaed-4f7b-b55c-fdd9086ea140)

<br>
在使用该方法进行fuzz时一直提示我上图的错误，我尝试设置 AFL_DEBUG=1，并在命令中加入 -m none,都没有解决，之后会记录第二种方法。<br>

**方法二：**
<br>
利用AFL_QEMU_PERSISTENT_ADDR来指定持久循环的开始，安装 valgrind 和 kcachegrind：

```
sudo apt-get install valgrind
sudo apt-get install kcachegrind
```

![image](https://github.com/xhsy0314/Task/assets/84487619/43a34684-c36d-4370-bb35-6b9295801e71)

<br>
另外，设置 AFL_QEMU_PERSISTENT_GPR=1，这将保存通用寄存器的原始值，并在每个持久循环中恢复它们。
<br>
运行模糊器:

```
AFL_QEMU_PERSISTENT_ADDR=0x085478AC AFL_QEMU_PERSISTENT_GPR=1 ACRO_INSTALL_DIR=/opt/Adobe/Reader9/Reader ACRO_CONFIG=intellinux LD_LIBRARY_PATH=$LD_LIBRARY_PATH:'/opt/Adobe/Reader9/Reader/intellinux/lib' afl-fuzz -Q -i ./afl_in/ -o ./afl_out/ -t 2000 -- /opt/Adobe/Reader9/Reader/intellinux/bin/acroread -toPostScript @@
```

发现仍然报错：**Fork server handshake failed**

<br>
于是我查看了一下二进制文件acroread，是32bit，下载qemu的时候已经将CPU_TARGET设置为 i386 <br>

![image](https://github.com/xhsy0314/Task/assets/84487619/7dfa10c1-a80e-4f69-bbb7-fad95c907466)<br>

尝试了很多方法仍然报错，卡在运行fuzz这一步

<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/e6a4c3fb-54e0-4238-951f-edb8ca535907)


