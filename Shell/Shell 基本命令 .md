# find

鸟哥链接：<http://cn.linux.vbird.org/linux_basic/0220filemanager.php#find>

如何解读这条find命令呢？

```shell
find "$searchpath" -type f -name '*.gif' -o -name '*.jpg'
```



执行 man find 可以查看手册

## -type

```shell
-type t
             True if the file is of the specified type.  Possible file types are as follows:

             b       block special
             c       character special
             d       directory
             f       regular file
             l       symbolic link
             p       FIFO
             s       socket
```

显然，我们命令的 -type f 指的是 查找 普通文件

## -name

```shell
-name pattern
             True if the last component of the pathname being examined matches pattern.  Special shell pattern matching characters (``['',
             ``]'', ``*'', and ``?'') may be used as part of pattern.  These characters may be matched explicitly by escaping them with a
             backslash (``\'').
```

显然，-name '*.jpg' 用于匹配 jpg 图片

## -o

那么上述-name 前方的-o是啥呢？ 我们的手册也有说明，o 就是 或者 or , 而且 a 就是 and

```shell
STANDARDS
     ............
		 ............
     The operator -or was implemented as -o, and the operator -and was implemented as -a.
```

##总结



```shell
find "$searchpath" -type f -name '*.gif' -o -name '*.jpg'
```

这条命令指的是查找 变量 searchpath 目录下所有 gif jpg 图片，并打印出来。

同样的，下方命令查找当前目录下所有以 bundle 为后缀的文件夹

```shell
find . -type d -name '*.bundle'
```

这是打印结果：

./QQNews/asset/bundles/web.bundle
./QQNews/asset/bundles/channelaudio.bundle
./QQNews/asset/bundles/dedao.bundle
./QQNews/asset/bundles/mine.bundle
./QQNews/asset/bundles/weibo.bundle
./QQNews/asset/bundles/guest.bundle



# stat

查看文件状态

##-f

具体看一个例子，获取某个文件的大小信息

```shell
stat -f %z ./QQNews/asset/bundles/subscribe.bundle/image/om_header_back@3x.jpg
81418
```

 man stat 看一下，找一下 -f 

```shell
-f format
             Display information using the specified format.  See the FORMATS section for a description of valid formats.
```

意思是 -f 代表按照指定格式 format 展示信息, 根据指引( See the FORMATS section )，我们把目光移到 FORMATS 来

```shell
Formats
     Format strings are similar to printf(3) formats in that they start with %, are then followed by a sequence of formatting characters, and end in a
     character that selects the field of the struct stat which is to be formatted.  If the % is immediately followed by one of n, t, %, or @, then a
     newline character, a tab character, a percent character, or the current file number is printed, otherwise the string is examined for the following:

     Any of the following optional flags:
     ...
     ...
     ...

     datum   A required field specifier, being one of the following:

             ...
             ...

             z       The size of file in bytes.
```

根据Formats 中的描述，% 后边跟参数来代表文件不同的信息，比如 %z 代表文件的字节数 The size of file in bytes.



# test

用于检测条件是否成立，在条件控制语句中，test 也可以直接省略

## -e

判断文件是否存在

## -f 

判断是普通文件

## -z

判断 string 是否长度为0  ( z 代表 zero )



# wc

-- word, line, character, and byte count

## -l

统计多少行

## -c

统计字节数



# sort

排序命令

## -n

按照数字大小排序，即 numeric-sort

## -r

逆序排列

-nr 可以一起使用，表示逆序即从大到小排列

# awk

截取字符串

```shell
new_file="new file:   QQNews/asset/bundles/listenNews.bundle/image/WechatIMG49.png"

# 截取文件名
file_name=`echo $new_file | awk '{print $3 }' `
echo $file_name 
（最终$file_name 的值是 QQNews/asset/bundles/listenNews.bundle/image/WechatIMG49.png）

这里使用awk命令可以很方便的把文件名字从 
new file:   QQNews/asset/bundles/listenNews.bundle/image/WechatIMG49.png 
取出来

$3代表以空格分割的第三个子串 （这里数数就不是从0开始数的了）
```









