1.zip伪加密
-

题目给出一个zip，里面有一个flag.txt，但是需要密码。<br>
拖到winhex，<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/0cc48105-7a64-4944-98bd-608b10908f53)

    **未加密：**

    文件头中的全局方式位标记为00 00
    
     目录中源文件的全局方式位标记为00 00
    
    **伪加密：**
    
    文件头中的全局方式位标记为00 00
    
    目录中源文件的全局方式位标记为09 00
    
    **真加密：**
    
    文件头中的全局方式位标记为09 00
    
    目录中源文件的全局方式位标记为09 00

按伪加密处理，把两个地方的全局方式位标记都改为00 00，接着保存，这样处理后文件就属于未加密了<br>

![image](https://github.com/xhsy0314/Task/assets/84487619/4a320ef8-af4c-45a6-abc7-b25745fd3d6f)


2.流量嗅探
-
```
http.request.method==POST
```
追踪http流量

3.取证
-

工具：firepwd<br>
将logins.json放到firepwd.py目录下，
```
python .\firepwd.py .\logins.json
```
得到flag<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/86aa6251-93cd-4449-90a8-787f6cdac1fa)

4.excel加密、汉信码（例题：[GKCTF 2021]excel 骚操作）
-

打开题目给出的excel文件，点击不同单元格发现有的值为1有的不显示<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/54f6ad39-1f26-481f-ad37-950a79ca9833)
<br>
将值为1的单元格全部显示出来，
      选中单元格-> 右键设置单元格格式
![image](https://github.com/xhsy0314/Task/assets/84487619/4347be67-5045-4d13-b3b3-c418e5fefe6c)
<br>
看样子像是二维码一样的排列，将值为1的单元格涂黑。
      选中单元格，点击条件格式，突出显示单元格规则，等于，自定义格式为黑色。

最后搜索“列宽”功能，调整宽度使二维码呈正方形<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/2907e75a-1bf8-4639-8f08-59c74b3782d5)

直接扫好像扫不出来，属于**汉信码**，打开中国编码网下载对应app，扫出flag

5.vmdk解压+brainfuck+ook编码（例题 BUUCTF 面具下的flag）
--

题目给出一个图片，用binwalk分解得到一个flag.vmdk文件，用kali下的7z命令解压该文件
```
7z x flag.vmdk -o./
```

解压得到四个文件夹<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/8ef6e61d-0c84-4e4d-a1c9-0c05f99abb83)
<br>

key_part_one打开是这样一段编码<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/05888a92-bf33-4837-983e-40feb1bc4a7b)<br>
搜索后知道是brainfuck，解码得到第一段flag。附上网址：
<https://www.splitbrain.org/services/ook><br>
![image](https://github.com/xhsy0314/Task/assets/84487619/324b3f12-4131-415e-b6ce-77cac0ae7f01)
<br>

第二部分是一段ook编码<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/1e257d8d-2794-425a-b7ad-412bbb91cba2)
<br>
同样通过上述网址解码，得到flag

6.（PatroitCTF 2023 ——Binary）
--
题目给出一个txt，内容：
```
1111111111001000101010111010111111101011111111110110110110110000011010010010111111111111110010100100000000000010101111111111001010000000000000000000001010011011110010100100101111111111111111111100101000000000000000000000000000000000101011111100101000000000000101001101100101001001010010100000000000010100000010100010100000010100110110010100100101111111110010100000000000000000000000000000000000000000000010101111111111100101000000000000000000000000000000000000000101001101100101001001011111111111111111001010000000000000000000000000000000001010111111001010000000000001010100101000000000000000000000000000000000000000000000101011111110010100000000000000000000010100110111111111111111100101000000000000000000000000000000000000000000000101001100101001001000000000000000101000000000000000000000000000000000000000000101011111111111111111001010000000000000000000000000000000000000000000001010010111111111111111111111001010011111111111111100101001000000000000000000000000000000000000101001101111111111111111110010100010100100000000000000000010101111111111111111100101001111111111111111111111110010100000000000000000000000000000000000000000000000000000000000010100010100101111100101000000000000000000101000000000000000000000000000000000000000000000000000000000101001100101001001000000000000000000000000000000000000000000000000000000101001101100000000101000101000101011111111001010000000000000000000000000001010010010111111111111111111111111111111001010
```
再没有其他提示。尝试了将数据导入到excel中，将01转换成二维码，但转换以后明显不是二维码。。。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/89ab0681-4721-4f33-8352-0d0ec58c8a7f)
<br>
再没招了。赛后查看wp，给了一个网址：<https://tio.run/#spoon><br>
将二进制数字输入到code中运行得到了flag。。。这种题只能靠积累了吧
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/03e607ea-ff74-4556-a0e0-6ed0533f128c)
<br>

7.正则表达式练习（PatriotCTF 2023）
-

![image](https://github.com/xhsy0314/Task/assets/84487619/e2ddf369-b972-48a6-b453-d1a2265dba52)
<br>
从题目中下载文件，解压后是一个txt文件，按题目要求需要找到 (NPA) XXX-XXXX 这个格式的电话号码并输出行号。其中根据北美区号标准，须是0-9的数字，利用notepad搜索正则表达式
```
^[(][0-9]{3}[)][[:space:]][0-9]{3}[-][0-9]{4}$
```
![image](https://github.com/xhsy0314/Task/assets/84487619/d5017fdb-36c2-400a-9442-93aa21069adc)
<br>
如果用grep命令也可以：
```
grep -E "^[(][0-9]{3}[)][[:space:]][0-9]{3}[-][0-9]{4,6}$" -n rockyou.txt
```

该正则表达式的含义是：

      ^[(]：以左括号开头
      [0-9]{3}：接着是三个数字
      )：然后是右括号
      [[:space:]]：接着是一个空格字符
      [0-9]{3}：然后是三个数字
      [-]：接着是一个短横线
      [0-9]{4,6}：最后是4到6个数字
      -n选项告诉grep命令输出匹配行的行号。

8.outguess加密
--

**加密：**
```
outguess -k "my secret key" -d hidden.txt demo.jpg out.jpg
```
加密之后，demo.jpg会覆盖out.jpg,

hidden.txt中的内容是要隐藏的东西

**解密：**
```
outguess -k "my secret key" -r out.jpg hidden.txt
```
解密之后，解密内容放在hidden.txt中

9.盲文解密
--

附上链接：<https://www.qqxiuzi.cn/bianma/wenbenjiami.php?s=mangwen>

10.Base64隐写
--

```
import base64
b64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
with open('D:/bluewhale-oj/attachment/tmp/ComeOn!.txt', 'rb') as f:
    flag = ''
    bin_str = ''
    for line in f.readlines():
        stegb64 = str(line, "utf-8").strip("\n")
        rowb64 = str(base64.b64encode(base64.b64decode(stegb64)), "utf-8").strip("\n")
        offset = abs(b64chars.index(stegb64.replace('=', '')[-1]) - b64chars.index(rowb64.replace('=', '')[-1]))
        equalnum = stegb64.count('=')  # no equalnum no offset
        if equalnum:
            bin_str += bin(offset)[2:].zfill(equalnum * 2)
            # flag += chr(int(bin(offset)[2:].zfill(equalnum * 2), 2))
            # print(flag) 这样写得不出正确结果
        print([chr(int(bin_str[i:i + 8], 2)) for i in range(0, len(bin_str), 8)])

```

11.字频统计
-

```
# -*- coding:utf-8 -*-
#Author: mochu7
alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()_+- =\\{\\}[]"
strings = open('./flag.txt').read()

result = {}
for i in alphabet:
	counts = strings.count(i)
	i = '{0}'.format(i)
	result[i] = counts

res = sorted(result.items(),key=lambda item:item[1],reverse=True)
for data in res:
	print(data)

for i in res:
	flag = str(i[0])
	print(flag[0],end="")

```
12.十六进制转换成rar
--

```
import binascii

hex_data='txt文件中16进制数据'
out=open('flag.rar','wb')
out.write(binascii.unhexlify(hex_data))
out.close()


```
