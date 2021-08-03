>  JDK 版本：8

### 1.1 不变性

先瞅瞅 String 在JDK源码中的定义：

```java
 public final class String
    implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
  ｝
```

从源码中，我们可以看到不可变的原因：

1. String被`final`修饰，说明 String 类不能被继承，也就是说对 String 的操作方法，都不会被继承覆盖；
2. String 中保存数据的是一个 char 数组`(value)`。value 也是被`final`修饰的，也就是`value`一旦被赋值，内存地址是绝对无法修改的，而且 value 的权限是`private`的，String 也没有开放出可以对 value 进行赋值的方法，外部绝对无法访问，所以说 value 一旦产生，内存地址就根本无法被修改。

如果要自定义一个不可变类，可以借助 String 的这种操作，利用`final`关键字的特性。

String具有不可变行，所以 String 的大多数方法都会返回新的 String

```java
String s = "abc";
// 替换失败
s.replace("b","d");
// 正确操作需要一个参数接收返回值
s = s.replace("b","d");
```

### 1.2 字符串乱码

字符串乱码的根源主要是两个：字符集不支持复杂汉字和二进制进行转化时字符集不匹配。

解决办法：所有用到编码的地方指定编码方式，使用 `UTF-8`。(`ISO-8859-1`对中文的支持有限，所以包含中文的时候不建议使用`ISO-8859-1`)

### 1.3 首字母大写

如果我要在让一个字符串的首字母小写，那么这时候该怎么办呢？

```java
name.substring(0,1).toLowerCase() + name.substring(1);
```

主要使用Java的两个方法：

1.  `publice String substring(int beginIndex, int endIndex)`：beginIndex表示开始位置，endIndex表示结束位置。（这是左闭右开的操作）
2.  `public String substring(int beginIndex)`：beginIndex表示开始位置，结束位置为文本末尾

如果要修改首字母大写，则

```java
name.substring(0,1).toUpperCase + name.substring(1)
```

首字母小写的应用场景，比如反射的场景下类的属性名。

### 1.4 相等判断`(equals)`

判断相等有两种方法：`equals`和`equalsIgnoreCase`。后者判断时忽略大小写

如果让你判断两个String相等的逻辑，应该如何设计代码？

**JDK源码**

```java
public boolean equals(Object anObject) {
    // 判断内存地址是否相等
    if (this == anObject) {
        return true;
    }
    // 判断待比较的对象是否是String，如果不是String，直接返回不相等
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        // 判断两个字符串的长度是否相等，不等则直接返回不相等
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 一次比较每个字符是否相等，若有一个不等，直接返回不相等
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

如果要设计代码判断两者是否相等时，我们可以参考String的方式，从两者的底层结构出发。先判断地址，再判断数据类型，最后就像 String 底层的数据结构是 char 的数组一样时，就挨个比较 char 数组中的字符是否相等即可

### 1.5 替换、删除`(replace)`

`str.replace(char oldChar, char newChar) `：替换所有匹配的字符。如果没有匹配的字符，返回的还是原来的对象，并不是新的对象。

**JDK源码**

```java
public String replace(char oldChar, char newChar) {
    if (oldChar != newChar) {
        int len = value.length;
        int i = -1;
        char[] val = value; /* avoid getfield opcode */

        while (++i < len) {
            if (val[i] == oldChar) {
                break;
            }
        }
        // 只有在遇到可以替换的字符时，才会生成新的字符串对象
        if (i < len) {
            char buf[] = new char[len];
            for (int j = 0; j < i; j++) {
                buf[j] = val[j];
            }
            while (i < len) {
                char c = val[i];
                buf[i] = (c == oldChar) ? newChar : c;
                i++;
            }
            return new String(buf, true);
        }
    }
    return this;
}
```

`replace(CharSequence target, CharSequence replacement) `：替换所有的匹配的字符串。**不支持正则表达式**

**JDK源码**

```java
// Pattern.compile(target.toString(), Pattern.LITERAL)匹配时把正则特殊字符转为普通字符。
// Matcher.quoteReplacement 匹配的结果特殊字符转为普通字符主要是$和\。
public String replace(CharSequence target, CharSequence replacement) {
    return Pattern.compile(target.toString(), Pattern.LITERAL).matcher(this).replaceAll(Matcher.quoteReplacement(replacement.toString()));
}
```

`replaceAll(String regex, String replacement)`：替换所有的匹配的字符串。支持正则表达式。

**JDK源码**

```java
public String replaceAll(String regex, String replacement) {
    return Pattern.compile(regex).matcher(this).replaceAll(replacement);
}
```

`replaceFirst(String regex, String replacement)`：替换匹配到的第一个字符串，支持正则。

**JDK源码**

```java
public String replaceFirst(String regex, String replacement) {    return Pattern.compile(regex).matcher(this).replaceFirst(replacement);}
```

字符串的删除可以通过替换，把需要删除的字符替换成 `""` 即可。

### 1.6 拆分`(split)`

`split`方法，该方法有两个参数，第一个参数是我们拆分的标准字符，第二个参数是一个 int 值`(limit)`，用来限制我们需要拆分几个元素。如果 limit 比实际能拆分的个数小，按照 limit 的个数进行拆分。

```java
String  s = "abb:dce:fbb";

// 默认情况下limit为0，结果：[abb, dce,fbb]
System.out.println(Arrays.toString(s.split(":")));
// 结果：[abb, dce:fbb]
System.out.println(Arrays.toString(s.split(":",2)));
// 结果：[abb, dce, fbb]
System.out.println(Arrays.toString(s.split(":",-1)));
```

如果字符串中存在空值，空值也会被拆分出来，但是**最后的空值不管出现几个都不会出现。**

```java
String  s = "abb:dce:fbbb";
// 输出：[a, , :dce:f]
System.out.println(Arrays.toString(s.split("b")));
```

### 1.7 合并`(join)`

`public static String join(CharSequence delimiter, CharSequence... elements) `，此方法是**静态的**，我们可以直接使用。第一个参数是合并的分隔符，第二个是合并的数据源，可变参数支持数组和 List 集合。

两个缺点：

1. 不支持依次 join 多个字符串。`String.join("," s1).join(",",s2)`，最后得到的是s1的值，第一次 join 的值`s1`会被第二次 join 的`s2`覆盖了；
2. 如果 join 的是一个 List ，无法自动过滤掉 null 值。

谷歌的 Guava 正好提供了 API 可以解决上述问题。

