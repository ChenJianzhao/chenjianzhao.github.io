---
categories: Linux
date: 2017-02-12 20:05
status: public
title: 初识Shell脚本
---

```shell
#!/bin/sh

# -----------------------------------
#	使用变量
# -----------------------------------
# your_name="qinjx"
# echo $your_name
# echo ${your_name}

```
```shell
# -----------------------------------
#		只读变量
# -----------------------------------
# myUrl="http://www.w3cschool.cc"
# readonly myUrl
# myUrl="http://www.runoob.com"
## 运行脚本，结果如下：
## /bin/sh: NAME: This variable is read only.

```
```shell
# -----------------------------------
#		删除变量
# -----------------------------------
# unset variable_name

```
```shell
# -----------------------------------
#		 for 循环
# -----------------------------------
# for skill in Ada Coffe Action Java; 
# do
#    echo "I am good at ${skill}Script"
# done


```
```shell
# -----------------------------------
#		 拼接字符串
# -----------------------------------
# your_name="123"
# greeting="hello, "$your_name" !"
# greeting_1="hello, ${your_name} !"
# echo $greeting $greeting_1

# #字符串变量间有空格的情况
# my_name="abc"
# our_name=${your_name}" "${my_name}
# our_name1="${your_name} ${my_name}"
# echo ${our_name} ${our_name1}

```
```shell
# -----------------------------------
#		获取字符串长度
# -----------------------------------
# string="abcd"
# echo ${#string} #输出 4

```
```shell
# -----------------------------------
#		 提取子字符串
# -----------------------------------
# string="runoob is a great site"
# echo ${string:1:4} # 输出 unoo

```
```shell
# -----------------------------------
#		 查找字符 "i 或 s" 的位置：
# -----------------------------------
# string="runoob is a great company"
# echo `expr index "$string" is`

```
```shell
# -----------------------------------
# 		数组
# -----------------------------------
# array_name=(value0 value1 value2 value3)
# for value in ${array_name[@]}
# do
# 	echo ${value}
# done
```
```shell
# -----------------------------------
#		获取数组的长度
# -----------------------------------
## 取得数组元素的个数
# length=${#array_name[@]}
## 或者
# length=${#array_name[*]}
## 取得数组单个元素的长度
# lengthn=${#array_name[n]}
```
```shell
# -----------------------------------
#		传递参数（命令行传递给脚本的参数）
# -----------------------------------
# echo "Shell 传递参数实例！";
# echo "执行的文件名：$0";
# echo "第一个参数为：$1";
# echo "第二个参数为：$2";
# echo "第三个参数为：$3";

```
```shell
# -----------------------------------
#		参数访问和参数数量
# -----------------------------------
# echo "Shell 传递参数实例！";
# echo "第一个参数为：$1";
# echo "参数个数为：$#";
# echo "传递的参数作为一个字符串显示：$*";
# echo "传递的参数作为一个字符串显示：$@";


# -----------------------------------
# $* 与 $@ 区别：
# 相同点：都是引用所有参数。
# 不同点：只有在双引号中体现出来。假设在脚本运行时写了三个参数 1、2、3，，
# 则 " * " 等价于 "1 2 3"（传递了一个参数），而 "@" 等价于 "1" "2" "3"（传递了三个参数）。
# -----------------------------------
# echo "-- \$* 演示 ---"
# for i in "$*"; do
#     echo $i
# done

# echo "-- \$@ 演示 ---"
# for i in "$@"; do
#     echo $i
# done
```
```shell
# -----------------------------------
# 			Shell 基本运算符
# -----------------------------------
# 原生bash不支持简单的数学运算，但是可以通过其他命令来实现，例如 awk 和 expr，expr 最常用。
# expr 是一款表达式计算工具，使用它能完成表达式的求值操作。
# 例如，两个数相加(注意使用的是反引号 ` 而不是单引号 ')：
# -----------------------------------
# a=10
# b=20

# val=`expr $a + $b`
# echo "a + b : $val"

# val=`expr $a - $b`
# echo "a - b : $val"

# val=`expr $a \* $b`
# echo "a * b : $val"

# val=`expr $b / $a`
# echo "b / a : $val"

# val=`expr $b % $a`
# echo "b % a : $val"

# if [ $a == $b ]
# then
#    echo "a 等于 b"
# fi
# if [ $a != $b ]
# then
#    echo "a 不等于 b"
# fi

# if [ $a -eq $b ]
# then
#    echo "a 等于 b"
# fi
# if [ $a -ne $b ]
# then
#    echo "a 不等于 b"
# fi
```
```shell
# -----------------------------------
#		关系运算符
# -----------------------------------

# 关系运算符只支持数字，不支持字符串，除非字符串的值是数字。
# 下表列出了常用的关系运算符，
# 假定变量 a 为 10，变量 b 为 20：
# 运算符	说明	举例
# -eq	(等同于 == ) 检测两个数是否相等，相等返回 true。	[ $a -eq $b ] 返回 false。		
# -ne	(等同于 != ) 检测两个数是否相等，不相等返回 true。	[ $a -ne $b ] 返回 true。
# -gt	检测左边的数是否大于右边的，如果是，则返回 true。	[ $a -gt $b ] 返回 false。
# -lt	检测左边的数是否小于右边的，如果是，则返回 true。	[ $a -lt $b ] 返回 true。
# -ge	检测左边的数是否大于等于右边的，如果是，则返回 true。	[ $a -ge $b ] 返回 false。
# -le	检测左边的数是否小于等于右边的，如果是，则返回 true。	[ $a -le $b ] 返回 true。
```
```shell
# -----------------------------------
#		布尔运算符
# -----------------------------------
# 下表列出了常用的布尔运算符，
# 假定变量 a 为 10，变量 b 为 20：
# 运算符	说明	举例
# !	非运算，表达式为 true 则返回 false，否则返回 true。	[ ! false ] 返回 true。
# -o	或运算，有一个表达式为 true 则返回 true。	[ $a -lt 20 -o $b -gt 100 ] 返回 true。
# -a	与运算，两个表达式都为 true 才返回 true。	[ $a -lt 20 -a $b -gt 100 ] 返回 false。
```
```shell
# -----------------------------------
#		逻辑运算符
# -----------------------------------
# 以下介绍 Shell 的逻辑运算符，
# 假定变量 a 为 10，变量 b 为 20:
# 运算符	说明	举例
# &&	逻辑的 AND	[[ $a -lt 100 && $b -gt 100 ]] 返回 false
# ||	逻辑的 OR	[[ $a -lt 100 || $b -gt 100 ]] 返回 true
# -----------------------------------
# a=10
# b=20

# if [[ $a -lt 100 && $b -gt 100 ]]
# then
#    echo "返回 true"
# else
#    echo "返回 false"
# fi
```
```shell
# -----------------------------------
#		字符串运算符
# -----------------------------------
# 下表列出了常用的字符串运算符，
# 假定变量 a 为 "abc"，变量 b 为 "efg"：
# 运算符	说明	举例
# =	检测两个字符串是否相等，相等返回 true。	[ $a = $b ] 返回 false。
# !=	检测两个字符串是否相等，不相等返回 true。	[ $a != $b ] 返回 true。
# -z	检测字符串长度是否为0，为0返回 true。	[ -z $a ] 返回 false。
# -n	检测字符串长度是否为0，不为0返回 true。	[ -n $a ] 返回 true。
# str	检测字符串是否为空，不为空返回 true。	[ $a ] 返回 true。
# -----------------------------------

# a="abc"
# b="efg"

# if [ $a = $b ]
# then
#    echo "$a = $b : a 等于 b"
# else
#    echo "$a = $b: a 不等于 b"
# fi
# if [ $a != $b ]
# then
#    echo "$a != $b : a 不等于 b"
# else
#    echo "$a != $b: a 等于 b"
# fi
# if [ -z $a ]
# then
#    echo "-z $a : 字符串长度为 0"
# else
#    echo "-z $a : 字符串长度不为 0"
# fi
# if [ -n $a ]
# then
#    echo "-n $a : 字符串长度不为 0"
# else
#    echo "-n $a : 字符串长度为 0"
# fi
# if [ $a ]
# then
#    echo "$a : 字符串不为空"
# else
#    echo "$a : 字符串为空"
# fi
```
```shell
# -----------------------------------
#		文件测试运算符
# -----------------------------------

# 操作符	说明	举例
# -b file	检测文件是否是块设备文件，如果是，则返回 true。	[ -b $file ] 返回 false。
# -c file	检测文件是否是字符设备文件，如果是，则返回 true。	[ -c $file ] 返回 false。
# -d file	检测文件是否是目录，如果是，则返回 true。	[ -d $file ] 返回 false。
# -f file	检测文件是否是普通文件（既不是目录，也不是设备文件），如果是，则返回 true。	[ -f $file ] 返回 true。
# -g file	检测文件是否设置了 SGID 位，如果是，则返回 true。	[ -g $file ] 返回 false。
# -k file	检测文件是否设置了粘着位(Sticky Bit)，如果是，则返回 true。	[ -k $file ] 返回 false。
# -p file	检测文件是否是有名管道，如果是，则返回 true。	[ -p $file ] 返回 false。
# -u file	检测文件是否设置了 SUID 位，如果是，则返回 true。	[ -u $file ] 返回 false。
# -r file	检测文件是否可读，如果是，则返回 true。	[ -r $file ] 返回 true。
# -w file	检测文件是否可写，如果是，则返回 true。	[ -w $file ] 返回 true。
# -x file	检测文件是否可执行，如果是，则返回 true。	[ -x $file ] 返回 true。
# -s file	检测文件是否为空（文件大小是否大于0），不为空返回 true。	[ -s $file ] 返回 true。
# -e file	检测文件（包括目录）是否存在，如果是，则返回 true。	[ -e $file ] 返回 true。
# -----------------------------------
# file="D:/Bash/test.sh"
# if [ -r $file ]
# then
#    echo "文件可读"
# else
#    echo "文件不可读"
# fi
# if [ -w $file ]
# then
#    echo "文件可写"
# else
#    echo "文件不可写"
# fi
# if [ -x $file ]
# then
#    echo "文件可执行"
# else
#    echo "文件不可执行"
# fi
# if [ -f $file ]
# then
#    echo "文件为普通文件"
# else
#    echo "文件为特殊文件"
# fi
# if [ -d $file ]
# then
#    echo "文件是个目录"
# else
#    echo "文件不是个目录"
# fi
# if [ -s $file ]
# then
#    echo "文件不为空"
# else
#    echo "文件为空"
# fi
# if [ -e $file ]
# then
#    echo "文件存在"
# else
#    echo "文件不存在"
# fi

# read name 
# echo "$name It is a test"
```
```shell
# -----------------------------------
#		流程控制
# -----------------------------------
#		if 条件
# -----------------------------------
# a=10
# b=20
# if [ $a == $b ]
# then
#    echo "a 等于 b"
# elif [ $a -gt $b ]
# then
#    echo "a 大于 b"
# elif [ $a -lt $b ]
# then
#    echo "a 小于 b"
# else
#    echo "没有符合的条件"
# fi
```
```shell
# -----------------------------------
#		for 循环
# -----------------------------------
# for str in 'This is a string'
# do
#     echo $str
# done
```
```shell
# -----------------------------------
#		while 循环
# -----------------------------------
# int=1
# while(( $int<=5 ))
# do
#         echo $int
#         let "int++"
# done

```
```shell
# -----------------------------------
#		until 循环
# -----------------------------------
# until condition
# do
#     command
# done
```
```shell
# -----------------------------------
#		case 语句
# -----------------------------------
# echo '输入 1 到 4 之间的数字:'
# echo '你输入的数字为:'
# read aNum
# case $aNum in
#     1)  echo '你选择了 1'
#     ;;
#     2)  echo '你选择了 2'
#     ;;
#     3)  echo '你选择了 3'
#     ;;
#     4)  echo '你选择了 4'
#     ;;
#     *)  echo '你没有输入 1 到 4 之间的数字'
#     ;;
# esac

```
```shell
# -----------------------------------
#		函数
# -----------------------------------
# 注意：所有函数在使用前必须定义。这意味着必须将函数放在脚本开始部分，
# 直至shell解释器首次发现它时，才可以使用。调用函数仅使用其函数名即可
# -----------------------------------
funWithReturn(){
    echo "这个函数会对输入 的两个数字进行相加运算..."
    echo "输入第一个数字: "
    read aNum
    echo "输入第二个数字: "
    read anotherNum
    echo "两个数字分别为 $aNum 和 $anotherNum !"
    return $(($aNum+$anotherNum))
}
funWithReturn
# 函数返回值在调用该函数后通过 $? 来获得。
echo "输入的两个数字之和为 $? !"

```
```shell
# -----------------------------------
#		函数参数
# -----------------------------------
# 函数返回值在调用该函数后通过 $? 来获得。
# 注意，$10 不能获取第十个参数，获取第十个参数需要${10}。当n>=10时，需要使用${n}来获取参数。
# funWithParam(){
#     echo "第一个参数为 $1 !"
#     echo "第二个参数为 $2 !"
#     echo "第十个参数为 $10 !"
#     echo "第十个参数为 ${10} !"
#     echo "第十一个参数为 ${11} !"
#     echo "参数总数有 $# 个!"
#     echo "作为一个字符串输出所有参数 $* !"
# }
# funWithParam 1 2 3 4 5 6 7 8 9 34 73
```