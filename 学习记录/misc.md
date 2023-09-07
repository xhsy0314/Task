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
