+ bc是bash内建的运算器，支持浮点数运算
+ 内建变量scale可以设置，默认为0

```shell
echo "43 + 5" | bc 
echo "scale=4;10/3" | bc
```

