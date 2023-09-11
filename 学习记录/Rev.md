1.JAVA反编译工具——jdGUI
-
反编译java代码。

2.运用gdb直接跳转到目标函数（PCTF2023 Patchwork）
-

![image](https://github.com/xhsy0314/Task/assets/84487619/06b78224-846f-443c-8078-99061c6908d3)
<br>
题目说程序执行流有问题，拖到ida里看到有give_flag函数，大概率就是目标函数。这里官方wp给出了用radar2解题的方法，但我没看懂😓，所以在这里借鉴大佬用gdb直接跳转到目标函数的解题方法。
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/f926ba59-cb8c-4eee-a6a4-19fe454862d9)
<br>

设置断点后运行程序。

<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/80722e55-521b-42b9-90d9-3433039e3d74)<br>

```
p &give_flag
#查看give_flag函数的地址

set $rip=0x55555555517d
#跳转到目标函数

c
#继续运行
```
![image](https://github.com/xhsy0314/Task/assets/84487619/9e5adcd4-cf19-47b2-b696-9d5cb0cb97d0)


