## 简介

+ awk是一个文本处理工具，通常用于处理数据并生成结果报告

## 语法格式

+ 第一种形式：`awk 'BEGIN{}pattern{commands}END{}' file_name`
+ 第二种形式：`standard output|awk 'BEGIN{}pattern{commands}END{}'`

| 语法格式   | 含义                     |
| ---------- | ------------------------ |
| BEGIN{}    | 正式处理数据之前执行     |
| pattern    | 匹配模式                 |
| {commands} | 处理命令，可能多行       |
| END{}      | 处理完所有匹配数据后执行 |

### awk的内置变量

| 内置变量                   | 含义                                            |
| -------------------------- | ----------------------------------------------- |
| `$0`                       | 整行内容                                        |
| `$1-$n`                    | 当前行的第1-n个字段                             |
| NF（Number Field）         | 当前行的字段个数，也就是有多少列                |
| NR（Number Row）           | 当前行的行号，从1开始计数                       |
| FNR（File Number Row）     | 多文件处理时，每个文件行号单独计数，都是从1开始 |
| FS（Field Separator）      | 输入字段分隔符。不指定默认以空格或tab键分割     |
| RS（Row Separator）        | 输入行分割符。默认回车换行                      |
| OFS（Out Filed Separator） | 输出字段分隔符。默认空格                        |
| ORS（Out Row Separator）   | 输出行分割符。默认回车                          |
| FILENAME                   | 当前输入的文件名字                              |
| ARGC                       | 命令行参数个数                                  |
| ARGV                       | 命令行参数数组                                  |

```shell
awk '{print $0}' /etc/passwd
#使用分割符处理
awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
#输出行号,文件单独输出，每个文件都是从1开始
awk '{print FNR}' list /etc/passwd
#输出每一行最后一个字段
awk '{print $NF}' list
```

### 注意事项

+ 在awk中无论是引用内部变量，还是外部变量，都不需要使用`$`,直接使用变量名即可



## awk格式化输出之printf

| 格式符 | 含义                     |
| ------ | ------------------------ |
| `%s`   | 打印字符串               |
| `%d`   | 打印十进制               |
| `%f`   | 打印一个浮点数           |
| `%x`   | 打印十六进制             |
| `%o`   | 打印八进制               |
| `%e`   | 打印数字的科学计数法形式 |
| `%c`   | 打印单个字符的ASCII码    |

| 修饰符 | 含义                                     |
| ------ | ---------------------------------------- |
| -      | 左对齐                                   |
| +      | 右对齐                                   |
| #      | 显示8进制在前面加0，显示16进制在前面加0x |

```shell
#输出的第一个字段和第二个字段强制占用20个位置，默认是右对齐方式
awk 'BEGIN{FS=":"}{printf "%20s %20s\n",$1,$7}' /etc/passwd
#使用左对齐方式，一定要使用数字限定位数
awk 'BEGIN{FS=":"}{printf "%-20s %-20s\n",$1,$7}' /etc/passwd
#使用浮点数打印
awk 'BEGIN{FS=":"}{print "%0.3f\n",$1}' /etc/passwd
```

***

## awk模式匹配的两种用法

+ 第一种模式匹配：`RegExp`，按正则表达式匹配

+ 第二种模式匹配：关系运算符

  

  | <    | 小于             |
  | ---- | ---------------- |
  | >    | 大于             |
  | <=   | 小于等于         |
  | >=   | 大于等于         |
  | ==   | 等于             |
  | !=   | 不等于           |
  | ~    | 匹配正则表达式   |
  | !~   | 不匹配正则表达式 |
  | `||` | 或               |
  | `&&` | 与               |
  | `!`  | 非               |

```shell
#匹配/etc/passwd文件行中含有root字符串的所有行
awk 'BEGIN{FS=":"}/root/{print $0}' /etc/passwd
#以":"为分隔符，匹配/etc/passwd文件中第3个字段小于50的所有行信息
awk 'BEGIN{FS=":"}$3<50{print $0}' /etc/passwd
#以:为分隔符，匹配/etc/passwd文件中第3个字段包含3个以上数字的所有行信息
awk 'BEGIN{FS=":"}$3~/[0-9]{3,}/{print $0}' /etc/passwd
#以:为分隔符，匹配/etc/passwd文件中包含hdfs或yarn的所有行信息
awk 'BEGIN{FS=":"}$1=="hdfs"||$1=="yarn"{print $0} /etc/passwd'
```

---

## awk动作中的表达式用法

| 运算符  | 含义                      |
| ------- | ------------------------- |
| `+`     | 加                        |
| `-`     | 减                        |
| `*`     | 乘                        |
| `/`     | 除                        |
| `%`     | 模                        |
| `^或**` | 乘方                      |
| `++x`   | 在返回x变量之前，x变量加1 |
| `x++`   | 在返回x变量之后，x变量加1 |

### 条件语句

```
if(条件表达式)
	动作1
else if(条件表达式)
	动作2
else
	动作3
	
awk 'BEGIN{FS=":"}{if($<50) print $1}' /etc/passwd
```

### 循环语句

+ do while

```
while(条件表达式)
		动作
do
			动作
while(条件表达式)
```

+ for

```
for(初始化计数器；计数器测试；计数器变更)
		动作
```

---

## awk中的字符串函数

| 函数名                | 含义                                                    | 返回值                    |
| --------------------- | ------------------------------------------------------- | ------------------------- |
| `length(str)`         | 计算字符串长度                                          | 整数长度值                |
| `index(str1,str2)`    | 在str1中查找str2的位置                                  | 返回值为位置索引，从1计数 |
| `tolower(str)`        | 转换为小写                                              | 转换后的小写字符串        |
| `toupper(str)`        | 转换为大写                                              | 转换后的大写字符串        |
| `substr(str,m,n)`     | 从str的m个字符开始，截取n位                             | 截取后的子串              |
| `split(str,arr,fs)`   | 按fs切割字符串，结果保存arr                             | 切割后的子串的个数        |
| `match(str,RE)`       | 在str中按照RE查找，返回位置                             | 返回索引位置              |
| `sub(RE,RepStr,str)`  | 在str中搜索符合RE的子串，将其替换为RepStr；只替换第一个 | 替换的个数                |
| `gsub(RE,RepStr,str)` | 在str中搜索符合RE的子串，将其替换为RepStr；替换所有     | 替换的个数                |

---

## awk选项总结

| 选项 | 解释            |
| ---- | --------------- |
| `-v` | 参数传递        |
| `-f` | 指定脚本文件    |
| `-F` | 指定分隔符      |
| `-V` | 查看awk的版本号 |

---

## awk数组用法

在awk中，使用数组是，不仅可以使用1,2...n作为数组下标，也可以使用字符串作为数组下标。

当使用1,2,3...n时，直接使用array[2]访问元素；需要遍历数组时，使用一下形式：

```shell
str="Allen Jerry Mike Tracy Jordan Kobe Garnet"
split()
for(i=1;i<length(array);i++)
		print array[i]
```

当使用字符串作为数组下标时，需要使用array[str]形式访问元素，遍历数组时，使用以下形式：

```shell
array["var1"]="Tom"
array["var2"]="Jerry"
array["var3"]="Pig"
for(a in array)
		print array[a]
```

