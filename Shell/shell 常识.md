# 数值运算

```shell
#!/bin/bash

val=`expr 2 + 2`
echo "两数之和为 : $val"
```

原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。

expr 是一款表达式计算工具，使用它能完成表达式的求值操作。

两点注意：

- 表达式和运算符之间要有空格，例如 2+2 是不对的，必须写成 2 + 2，这与我们熟悉的大多数编程语言不一样。
- 完整的表达式要被 **` `** 包含，注意这个字符不是常用的单引号，在 Esc 键下边。



如果想把shell的输出存进一个文件，可以 在命令行后边追加  >>  filename.txt

默认文件路径是当前工作路径

比如用 otool 分析 二进制文件

```powershell
otool -v -o /iOS/ipa_file_analyze/Payload_5.8.2_has_armv7/QQNews.app/QQNews  >> otool_result.txt
```

