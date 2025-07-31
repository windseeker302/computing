# shell语言

## shell 中各种括号的作用

### ()

1. 命令组：括号中的命令将会新开一个子shell顺序执行，所以括号中的变量不能够被脚本余下的部分使用。
2. 命令替换：等同于“cmd”，shell扫描一遍命令行，发现了$(cmd)结构，便将$(ccmd)中的cmd执行一次，将其得到标准输出，再将此输出放到原来命令。
3. 初始化数组：如 array=(a b c d)



### (())

1. 整数扩展。这种扩展计算是整数型的计算，不支持浮点型。((exp))结构扩展并计算一个算术表达式的值，如果表达式的结果为0，那么返回的退出状态码为1，或者 是"假"，而一个非零值的表达式所返回的退出状态码将为0，或者是"true"。若是逻辑判断，表达式exp为真则为1,假则为0。
2. 单纯用 (( )) 也可重定义变量值，比如 a=5; ((a++)) 可将 $a 重定义为6。
3. 算数运算。



### []

Test和[]中可用的比较运算符只有==和!=，两者都是用于字符串比较的，不可用于整数比较，整数比较只能使用-eq，-gt这种形式。无论是字符串比较还是整数比较都不支持大于号小于号。如果实在想用，对于字符串比较可以使用转义形式，如果比较"ab"和"bc"：[ ab < bc ]，结果为真，也就是返回状态为0。[ ]中的逻辑与和逻辑或使用-a 和-o 表示。




### [[]]

1. [[是 bash 程序语言的关键字。并不是一个命令，[[ ]] 结构比[ ]结构更加通用。在[[和]]之间所有的字符都不会发生文件名扩展或者单词分割，但是会发生参数扩展和命令替换。
2. 使用[[ … ]]条件判断结构，而不是[ … ]，能够防止脚本中的许多逻辑错误。比如，&&、||、<和> 操作符能够正常存在于[[ ]]条件判断结构中，但是如果出现在[ ]结构中的话，会报错。



~~~shell
if ($i<5)    
if [ $i -lt 5 ]    
if [ $a -ne 1 -a $a != 2 ]    
if [ $a -ne 1] && [ $a != 2 ]    
if [[ $a != 1 && $a != 2 ]]    
     
for i in $(seq 0 4);do echo $i;done    
for i in `seq 0 4`;do echo $i;done    
for ((i=0;i<5;i++));do echo $i;done    
for i in {0..4};do echo $i;done   
~~~



### {}

- (1) 大括号拓展。(通配(globbing))将对大括号中的文件名做扩展。在大括号中，不允许有空白，除非这个空白被引用或转义。第一种：对大括号中的以逗号分割的文件列表进行拓展。如 touch {a,b}.txt 结果为a.txt b.txt。第二种：对大括号中以点点（..）分割的顺序文件列表起拓展作用，如：touch {a..d}.txt 结果为a.txt b.txt c.txt d.txt

  ~~~shell
  # ls {ex1,ex2}.sh    
  ex1.sh  ex2.sh    
  # ls {ex{1..3},ex4}.sh    
  ex1.sh  ex2.sh  ex3.sh  ex4.sh    
  # ls {ex[1-3],ex4}.sh    
  ex1.sh  ex2.sh  ex3.sh  ex4.sh    
  ~~~

  

## 单引号和双引号

在 Shell 脚本中，双引号和单引号都用于定义字符串，但它们之间有一些关键的区别。

### 单引号

- **功能**：单引号用于创建强引用的字符串，其中的特殊字符不会被解释或扩展。
- **行为**：在单引号内部，所有的特殊字符（包括变量、命令替换等）都会被视为普通字符。也就是说，单引号内的内容会原样输出，不会发生任何扩展或替换。
- **示例**：

```sh
echo 'Hello, $USER'
```

上述命令会输出 `Hello, $USER`，而不是实际用户的名称。

### 双引号

- **功能**：双引号用于创建弱引用的字符串，其中的变量会被扩展为其值，但特殊字符（除了 `$`、```、`\` 等）不会被扩展。
- **行为**：在双引号内部，变量会被替换为其值，但命令替换和转义序列（如 `\n`）会被保留。这意味着，双引号内的变量会被解释，而普通文本和大多数特殊字符则保持原样。
- **示例**：

```sh
echo "Hello, $USER"
```

上述命令会输出 `Hello,` 后跟实际用户的名称。

### 引号嵌套

- **双引号嵌套单引号**：在双引号内部，可以嵌套单引号，此时单引号内的内容会被视为普通字符串，不会发生扩展或替换。
- **单引号嵌套双引号**：在单引号内部，不能嵌套双引号来期望双引号内的内容被解释或扩展，因为单引号会忽略其内部的所有特殊字符。

### 总结

- **单引号**：适用于需要原样输出的字符串，其中的特殊字符不会被解释或扩展。
- **双引号**：适用于需要变量扩展的字符串，但大多数特殊字符仍会保持原样。

在编写 Shell 脚本时，根据实际需要选择合适的引号来定义字符串，可以确保脚本的正确性和可读性。



## shell 脚本常用的条件判断方式

- Shell脚本中常见的条件判断方式有：

  - test

  - 中括号[ ]

  - 双中括号[[ ]]

  - 双括号(( ))

### 常见使用场景

#### 数值比较

| 参数选项 | 功能说明                 |
| -------- | ------------------------ |
| -eq      | 等于（equal）            |
| -ne      | 不等于（not equal）      |
| -gt      | 大于（greater than）     |
| -lt      | 小于（less then）        |
| -ge      | 大于等于（greate equal） |
| -le      | 小于等于（less equal）   |

示例：

~~~shell
#!/bin/bash

a=1
b=2
if test $a -eq $b
then
        echo "a=b"
elif  test $a -ne $b 
then
        echo "a!=b"
fi
~~~

#### 字符串比较

| 参数选项 | 功能说明   |
| -------- | ---------- |
| -n       | 非空字符串 |
| -z       | 空字符串   |
| =        | 相等       |
| !=       | 不相等     |

示例：

~~~shell
#!/bin/bash

a=""
b="abc"

if test -n "$a"
then
        if test $a = $b
        then
                echo "a=b"
        elif test $a != $b
        then
                echo "a!=b"
        fi
elif test -z "$a"
then
        echo "a is null"
fi
~~~



#### 文件比较

| 参数选项 | 功能说明 |
| -------- | -------- |
| -d       | 目录     |
| -f       | 普通文件 |
| -e       | 存在     |
| -r       | 读权限   |
| -w       | 写权限   |
| -x       | 执行权限 |

示例：

~~~shell
if test -d my_dir;then echo "It is a directory.";fi
~~~



#### 逻辑判断

| 参数选项 | 功能说明               |
| -------- | ---------------------- |
| -a       | 两条件是否同时成立     |
| -o       | 两条件是否至少一个成立 |
| !        | 取反                   |

示例：

~~~shell
[root@openEluer ~]# if test 1 -eq 1 -a 2 -eq 2;then echo "ok";fi
ok
[root@openEluer ~]# if test ! 1 -eq 1 -a 2 -eq 2;then echo "ok";fi
[root@openEluer ~]# if test 1 -eq 2 -o 2 -eq 2;then echo "ok";fi
ok
~~~



### 括号逻辑判断简介

| 参数选项 | 功能说明 |
| -------- | -------- |
| &&       | 逻辑与   |
| \|\|     | 逻辑或   |
| !        | 逻辑非   |

- []、[[]]、(())也可进行逻辑判断，通过单语句判断来控制执行：
  - a && b ：a为真，才执行b
  - a || b ：a为假，才执行b
  - 多个&&或||组合使用时，判断如下：
    - &&前面命令执行成功，才会执行下一条命令
    - ||前面命令执行成功，后面的命令就不再执行

示例：

~~~shell
[ 1 -eq 1 ] && echo a
~~~



### 条件判断用于for循环嵌套

~~~shell
#!/bin/bash

# 指定要列出的目录路径
directory="/path/to/directory"

# 使用嵌套的 for 循环列出指定目录及其子目录中的所有文件和目录
echo "Listing files and directories in $directory and its subdirectories:"
for dir in "$directory"/*; do
    if [ -d "$dir" ]; then
        echo "Directory: $dir"
        for file in "$dir"/*; do
            echo "  File: $file"
        done
    elif [ -f "$dir" ]; then
        echo "File: $dir"
    fi
done
---
#!/bin/bash

# 打印九九乘法表
for (( i = 1; i <= 9; i++ )); do
    for (( j = 1; j <= i; j++ )); do
        # 计算乘积并输出格式化的结果
        echo -n " $j * $i = $[$i*$j]"
    done
    # 在每一行结束后换行
    echo ""
done
~~~



### 条件判断用于while循环嵌套

示例：99乘法表

```shell
#!/bin/bash
i=1
while [ $i -le 9 ]
do
        j=1
        while [ $j -le $i ]
        do
                echo -n  "$i * $j = $[i+j] " 		# -n 表示在末尾不自动添加换行符
                let j++
        done
        let i++
        echo ""
done
```



### until语句

until命令和while命令工作的方式完全相反。until命令要求你指定一个通常返回非零退出状态码的测试命令。只有测试命令的退出状态码不为0，bash shell才会执行循环中列出的命令。一旦测试命令返回了退出状态码0，循环就结束了。

until命令的格式如下。

```shell
until test commands
do
    other commands
done
```

 

示例：

本例中会测试var1变量来决定until循环何时停止。只要该变量的值等于0，until命令就会停止循环。同while命令一样，在until命令中使用多个测试命令时要注意。

~~~shell
$ cat test13
#!/bin/bash
# using the until command
var1=100
until echo $var1
[ $var1 -eq 0 ]
do
echo Inside the loop: $var1
var1=$[ $var1 - 25 ]
done
$ ./test13
100
Inside the loop: 100
75
Inside the loop: 75
50
Inside the loop: 50
25
Inside the loop: 25
0
$
~~~



## 函数

### 函数结构简介

- Shell允许将一组命令或语句形成一段代码块，这段代码块称为Shell函数，并且可以用函数名调用这段代码块
- 语法结构：
  - 标准定义function fun(){}，两种简洁定义fun(){} 、function fun{}，括号内不带任何参数
  - 函数需要先定义再调用

#### 基本的 Shell 函数定义和调用

```shell
#!/bin/bash

# 定义一个简单的函数
say_hello() {
    echo "Hello, $1!"
}

# 调用函数并传递参数
say_hello "World"
```



#### 带有返回值的 Shell 函数

```shell
#!/bin/bash

# 定义一个计算两个数之和的函数
add() {
    local result=$(( $1 + $2 ))
    echo $result
}

# 调用函数并捕获返回值
sum=$(add 5 7)
echo "The sum is: $sum"
```



#### 带有全局变量的 Shell 函数

```shell
#!/bin/bash

# 定义一个函数来修改全局变量
increment() {
    count=$(( count + 1 ))
}

# 初始化全局变量
count=0

# 调用函数多次
increment
increment
increment

# 输出全局变量的值
echo "The count is: $count"
```



#### 带有条件判断和循环的 Shell 函数

```shell
#!/bin/bash

# 定义一个函数来检查一个数是否为素数
is_prime() {
    local num=$1
    if [ $num -le 1 ]; then
        return 1
    fi
    for ((i=2; i*i<=num; i++)); do
        if [ $((num % i)) -eq 0 ]; then
            return 1
        fi
    done
    return 0
}

# 调用函数并根据返回值输出结果
number=29
if is_prime $number; then
    echo "$number is a prime number."
else
    echo "$number is not a prime number."
fi
```



#### 带有数组参数的 Shell 函数

```shell
#!/bin/bash

# 定义一个函数来打印数组中的所有元素
print_array() {
    local array=("$@")
    for element in "${array[@]}"; do
        echo $element
    done
}

# 定义一个数组
my_array=(apple banana cherry)

# 调用函数并传递数组
print_array "${my_array[@]}"
```

