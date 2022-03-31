## 一、基础配置

### 1、配置用户信息

```shell
# 配置用户名称
git config --global user.name 'your_name'
# 配置用户邮箱
git config --global user.email 'your_email'
```

### 2、不同的参数对应的作用域不同

-   `git config --local`：只对当前仓库生效
-   `git config --global`：对当前用户的所有仓库生效
-   `git config --system`：对系统所有登录的用户生效

**当不同作用域存在同名的配置时，小作用域的配置后覆盖上层的相同配置。例如当前仓库的配置会覆盖当前用户的配置。**

### 3、显示 config 配置，加 `--list`

```shell
git config --list --local
git config --list --global
git config --list --system
```

## 二、文本换行符配置

在不同的系统上，行尾结束符是不一样的，Windows使用回车和换行两个字符结束一行， 而Mac和Linux之使用换行一个字符。

用`core.autocrlf`配置相关的功能，可以让Git 在提交代码时自动把行位结束符`CRLF`转换成`LF`，获取代码时把 `LF` 转换成`CRLF`。它总共有三个值可以设置：`true,false,input`

设置成`true`，提交代码时，将`CRLF`转换成`LF`，下载代码时，LF 会被转换成 CRLF：

```shell
git config --global core.autocrlf true
```

设置为`false`时，提交或者下载代码时，文本文件保持原来的样子

```shell
git config --global core.autocrlf false
```

设置为`input`时，提交代码时，把`CRLF`转换成`LF`，下载代码时不转换

Linux 或 Mac 系统使用 `LF` 作为行结束符，下载代码时不想自动转换，可以这样设置

```shell
git config --global core.autocrlf input
```

## 三、文本编码设置

+ `core.quotepath`： 路径是否转义
+ ` gui.encoding`：图形化界面编码
+ `i18n.commitEncoding`：`git commit log`存储时，采用的编码，默认 UTF-8
+ `i18n.logOutputEncoding`：`git log`时，显示采用的编码，建议设置 UTF-8

```shell
# 图形化界面编码
git config --global gui.encoding utf-8
# 提交信息编码
git config --global i18n.commitencoding utf-8
# 输出 log 编码
git config --global i18.logoutputencoding utf-8
# 路径不转义，可以显示路径中的中文
git config --global core.quotepath false
```

也可以通过修改`etc\config`文件实现

```
[core]
    quotepath = false
[gui]
    encoding = utf-8
[i18n]
    commitencoding = utf-8
    logoutputencoding = utf-8
```

## 四、HTTP/HTTPS 协议认证

```shell
# 设置口令缓存，自动记录上次认证的结果
git config --global credential.help store
# 关闭ssl校验
git config http.sslverify false
```

## 五、创建仓库

1、已有项目代码纳入Git管理

```shell
cd 项目代码所在文件夹
# 初始化仓库
git init
```

2、新建的项目直接用Git管理

```shell
# 在 project 文件夹下创建一个 test 项目仓库
cd project
git init test
```