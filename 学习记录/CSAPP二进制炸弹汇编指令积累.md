phase_3:
-
    
    JG   ;有符号大于则跳转(前>后)
    JA   ;无符号大于则跳转( “无符号”一词表示只能包含正数的变量。)
    je   ;相等跳转 (左操作数=右操作数)
    jmpq *0x402390(,%rax,8)   ;用于直接跳转到存储在的绝对地址 8 * %rax + 0x402390

    cmpl source dest   ;source<dest则CF=1,否则CF=0
    0x8 (%rsp)   ;表示“获取栈上距离栈指针 %rsp 8 个字节的位置，然后获取该地址处的值。
    lea eax，[addr]   ;将表达式addr的值放入eax寄存器

```
cmpl   $0x7,0x8(%rsp)		# 第一个数必须小于7，否则爆炸
ja     400fad				# 引爆炸弹
```
以上两句汇编等同于 if (*rsp+8) > 7, 跳转到0x400fad， 因此第一个数必须不大于7。

```
jmpq *0x402470(,%rax,8)
```

是一个间接跳转指令, 可以看出这段代码是典型的switch语句，跳转表就存在于0x402470。%rax取值为[0, 7]，代表switch语句中8条不同的case。


phase_4:
-

        Test   ;普遍的用法是用来测试一方寄存器是否为空：
```
test ecx, ecx
jz somewhere
```
如果ecx为零，设置ZF零标志为1，jz跳转。

        JBE命令时，如果此时的CF标志位或者ZF标志位为1，跳转，其他不跳转
        SHR   ;（右移）指令使目的操作数逻辑右移一位，最高位用 0 填充。
        SAR   ;（算术右移）指令将目的操作数进行算术右移：
        lea (%rax,%rsi,1),%ecx   ;作用是ecx=rax+rsi*1

        cmp %edi,%ecx 
        jle 400ff2      ;若ecx≤edi，则跳转到400ff2

        cmp %edi,%ecx
        jge 401007     ；若ecx≥edi，则跳转到401007


Phase_5
-

		xor eax,eax
eax异或eax，相同为0，并把结果存放到eax，简单说该语句就是想eax寄存器清零。<br>

**windows x64寄存器命名规则**
前缀R
表示64位寄存器。例如RAX。
前缀E
表示32位寄存器。例如EAX
后缀L
表示寄存器的低8位
后缀H
表示寄存器的9~16位
