# shell语言

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

