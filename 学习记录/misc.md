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
