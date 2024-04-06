1、""不会被bash翻译，想要输出有""需要用转义

```bash
a="hello,\"world\""
echo $a
output:hello,"world"
```

2、

```bash
\" 、\\、 \`、\$三个会被bash直接转义，echo时直接就变成"、\、`、$
但是\t、\n、\r三个不会被bash直接转义，需要echo时加参数-e
例子见1中world

注意`是反引号，不是单引号
```

3、单引号中所有的变量、转义符包括$等都不会被执行，因此尽量用双引号。同时，根据这个特性也可以用'$'在双引号中输出$，避免被判断为变量标识符。

4、${a}和$a是同样的，但是{}更好的规定了边界，否则

```bash
a=b
echo $ac  //输出为空，没有定义ac
echo ${a}c  //输出为bc
```

5、定义数组

```bash
my_array=(1 2 3 4 5)   //整数数组

//索引数组
declare -A associative_array
associative_array["name"]="John"
associative_array["age"]=30

```

5、脚本必须赋予可执行权限才能执行

```bash
chmod -x  xxx.sh
```

