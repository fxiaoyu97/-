## 执行多条语句

```
#!/bin/bash
su - calos <<EOF
pwd;
exit;
EOF
```

**在这种方式中使用到文件路径时，需要使用绝对路径。**

## 执行一条语句

- 切换用户执行一条命令：`su - calos -c command`
- 切换用户执行一个shell文件：`su - calos /bin/bash test.sh`

