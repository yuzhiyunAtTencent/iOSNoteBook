# 打印三角形

## 源码



```shell
#!/bin/bash
#This program print triangle!

read -p "Please enter the line number:" line
read -p "Please enter the char:" char

currentLine=1

while [ $currentLine -le $line ]
do
	column=1
	while [ $column -le $currentLine ]
	do
		echo -n "$char"
		column=`expr $column + 1`
	done
	echo
	currentLine=`expr $currentLine + 1`
done
```

## 运行结果

```shell
➜  ~ ./triangle.sh
Please enter the line number:10
Please enter thie char:*
*
**
***
****
*****
******
*******
********
*********
**********
➜  ~ ./triangle.sh
Please enter the line number:10
Please enter thie char:&
&
&&
&&&
&&&&
&&&&&
&&&&&&
&&&&&&&
&&&&&&&&
&&&&&&&&&
&&&&&&&&&&
```

# 对 QQNews 项目 bundle 下图片按size 排序

实现方法是先递归遍历指定目录，然后把图片全部copy 到 另一个目录下/iOS/tempImage/，在/iOS/tempImage/目录下执行 la -LS 即可倒序排列

##源码

```shell
#!/bin/bash

tempImageDirectory=/iOS/tempImage/

mkdir $tempImageDirectory

function getDirection() {
    for file in $1/*
    do
    	if test -f $file
    	then
        	echo $file

		# 把文件移动到临时文件夹下
		cp $file $tempImageDirectory
    	else
    			# 递归
        	getDirection $file
    	fi
    done
}

# 调用函数 getDirection 递归遍历目录
getDirection  /iOS/tencent2/QQNews/QQNews/asset/bundles

echo "_____________end___________"
```

## 腾讯新闻已有脚本 sort_images.sh

新闻自带一个脚本，就分析出项目中所有大文件（主要是图片），还可以参数指定具体子目录，最终结果生成一个 html文件，可以用safari 打开查看，非常方便，过程中也没有跟我一样复制文件，实现的更加简洁、扩展性更强。

### sort_images.sh 源码

```shell
#!/bin/bash

searchpath=$1
if [[ $searchpath == "" ]];
then
    searchpath="."
    echo "Please input search path. Now search path is current path '.'."
fi

temp="temp"
rm -rf $temp

files="";
let count=0;
let totalsize=0;

find "$searchpath" -type f -name '*.gif' -o -name '*.jpg' -o -name '*.JPG' -o -name '*.png' -o -name '*.PNG' -o -name '*.jpeg' -o -name '*.JPEG' -o -name '*.pdf' | while read filepath;
do
  filepath=`echo $filepath | sed  's/ /\\\ /g'`
  command="stat -f \"%z\" ""$filepath"
  filesize=`eval "$command"`

  if [[ -z "$filesize" ]];
  then
    echo "[error]: ""$command"
  else
    filesizekb=`echo "$filesize 1024.0" | awk '{printf "%.3f", $1 / $2}'`
    echo $filepath,$filesizekb
    line="<a href=\"$filepath\">$filepath</a> ( $filesizekb kb ) <br>"
    echo $line >> "temp"
  fi
done	

files=`cat $temp | sort -t " " -nr -k 4,4`
count=`echo $(cat $temp | wc -l)`
totalsizekb=`awk -F ' ' '{sum+=$4} END {print sum}' $temp`

html="./safari.html"
rm -rf "$html"

echo "<html>\n<h2> Sorted images </h2>" >> "$html"
echo "<body>" >> "$html"
echo "<h3>There are $count images (total size: $totalsizekb kb)</h3>" >> "$html" >> "$html"
echo "<pre>\n$files\n</pre>" >> "$html"
echo "</body>\n</html>" >> "$html"

#clean
rm -rf $temp
```

## 我修改后的代码

```shell
#!/bin/bash

#定义第一个参数为搜索目录，如果没输入参数，默认搜索当前目录
SEARCH_PATH=$1
if [[ -z $SEARCH_PATH ]]
then
    SEARCH_PATH="."
    echo "Please input search path. Now search path is current path '.'"
fi

#临时文件存储未排序结果
TEMP="unsorted_image_temp"
rm -rf $TEMP

# 搜索目标文件 （-type f 普通文件 -name '*.gif' 名字后缀为gif的文件 -o 或者 or）
find $SEARCH_PATH -type f -name '*.gif' -o -name '*.jpg' -o -name '*.JPG' -o -name '*.png' -o -name '*.PNG' -o -name '*.jpeg' -o -name '*.JPEG' | while read file;
do

  # 开始处理每一张图片

  # 获取图片size (stat 查看状态 -f 按指定format打印  %后边跟z表示size)
  size=`stat -f %z $file`
  # 把图片大小转换成kb,保留三位小数点
  sizekb=`echo "$size 1024.0" | awk '{printf "%.3f", $1 / $2}'`
  echo $file,$sizekb
  line="<a href=\"$file\">$file</a> ( $sizekb kb ) <br>"

  # 把每张图片信息写进临时文件
  echo $line >> $TEMP

done

# 根据图片大小进行排序 （-t " " 以空格作为分隔符 -n 以数字大小排序 -r 逆序排列 -k 4 以第四栏作为sort_key）
files=`cat $TEMP | sort -t " " -nr -k 4`
# 计算图片数量
count=`echo $(cat $TEMP | wc -l)`
# 计算图片总大小 (awk 命令，以空格分割，第4栏的累计之和)
total_size_kb=`awk -F ' ' '{sum += $4} END {print sum}' $TEMP`

rm -rf $TEMP

# 把结果打印到html文件，方便使用浏览器查看，且可以点击a标签直接查看图片
HTML="sorted_images_list.html"
rm -rf $HTML

echo "<html>\n<h1> Sorted images by Zhiyunyu</h1>" >> $HTML
echo "<body>" >> $HTML
echo "<h2>There are $count images (total size: $total_size_kb kb)</h2>" >> $HTML
echo "<pre>\n$files\n</pre>" >> $HTML
echo "</body>\n</html>" >> $HTML
```

# commit 之前检测大图片

在工程目录下找到 .git/hook文件夹

新建文件 pre-commit ( 或者直接rename  git 自带的   pre-commit.sample )

并执行 chmod 777  pre-commit

输入以下内容即可

```powershell
#!/bin/sh

# 脚本用于commit之前检查大图片，如果新增单张图片超过50k,则commit失败

# 新增图片后，使用 git status可以看到类似结果： 	new file:   QQNews/asset/bundles/listenNews.bundle/image/WechatIMG49.png

file_max_size=50
git status | grep "new file" | while read line;
do

	# 截取文件名
	file_name=`echo $line | awk '{print $3 }' `

	file_size=`stat -f %z $file_name`
	# 把图片大小转换成kb,保留2位小数点
	sizekb=`echo "$file_size 1024" | awk '{printf "%i", $1 / $2}'`

	if [ $sizekb -gt  $file_max_size ]
	then
		echo "Sorry ! File more than $file_max_size K is forbidden :"
		echo
		echo "$file_name is $sizekb K"
		# 中止 commit 
		exit 1
	fi

done
```

但是由于git 不支持把 .git/hook 下脚本push到仓库进去，且 clone 的时候不会把这个目录下的脚本 checkout 下来，

所以我们可以把 pre-commit 文件放在仓库中别的地方，然后我们需要单独追加一个脚本，这个脚本的功能是把 pre-commit 文件 拷贝到 .git/hooks目录下。

现在我们追加一张超过50k的图片到项目中，执行 git commit 之后看看效果：

```shell
➜  QQNews git:(feature/zhiyunyu_pre_commit_file_size) ✗ g commit
Sorry ! File more than  is forbidden :

QQNews/asset/bundles/listenNews.bundle/image/recommend_mini_beauty.png is 70 K
```

的确达到目的了。

# 判断字符串包含关系

```powershell
# 忽略debug目录
    if [[ $file_name =~ 'QQNews/asset/debug' ]] ; then
    	continue
    fi
```



