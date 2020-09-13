<font color=red>**`如果需要做跨平台应用，那么不要吧 #include非标准库的写在头文件中`**</font>

#    makefile是什么？

实际开发中，一个工程是有许多源文件的，手动一个个编译时不可能的。

这时就需要Makefile了，写好之后只需要一个make命令就可以完成了，make命令会自动的根据当前的文件修改情况来确定哪些文件需要重新编译，从而自己编译所需要的文件和连接目标程序。

# Makefile 的规则

```
target:prerequisites...
commond	
```

target也就是一个目标文件，也可以是Object File，也可以是执行文件。

prerequisites就是，要生成那个target所需要的文件或是目标。

command也就是make需要执行的命令。（任意的Shell命令）

**实例**

```makefile
# 变量
objects = hello.o speak.o

hello_demo : ${objects}
	g++ -o hello_demo  ${objects}
hello.o: hello.cpp  speak.h
	g++ -c hello.cpp
speak.o: speak.cpp  speak.h
	g++ -c speak.cpp
clean:
	rm hello_demo  speak.o  hello.o
	# - 符号相当于注释作用
	-	rm hello_demo  speak.o  hello.o
```

# 工作流程

1. make会在当前目录下找名字叫“Makefile”或“makefile”的文件。
2. 如果找到，它会找文件中的第一个目标文件（target），在上面的例子中，他会找到“hello_demo”这个文件，并把这个文件作为最终的目标文件。
3. 如果hello文件不存在，或是hello所依赖的后面的 .o 文件的文件修改时间要比hello这个文件新，那么，他就会执行后面所定义的命令来生成hello这个文件。
4. 如果hello所依赖的.o文件也不存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到则再根据那一个规则生成.o文件。（像不像堆栈过程？）
   5、当然，我们的C文件和H文件都存在，于是make会生成 .o 文件，然后再用 .o 文件生命make的终极任务，也就是执行文件hello了。

# 变量

重复度高的字符串我们可以通过变量来替换。

**变量定义**

`变量名=值`

使用变量`${变量名}`

```makefile
objects = hello.o speak.o
hello_demo : ${objects}
```

访问变量可以用`${}`也可以用`$()`

**自带变量**

`$@`： 生成目标

```makefile
hello.o: hello.cpp  speak.h
	g++ -c hello.cpp -o $@
```

这儿的`%@`代表`hello.o`。

`@+`: 指定依赖项

```makefile
hello.o: hello.cpp  speak.h
	g++ -c $+ -o $@
```

**简化**

通过上面的变量，我们可以继续简化一个Makefile

```makefile
CC=g++
OCC = $(CC) $+ -o$@ -c

testLog:testLog.o log.o
	${CC} testLog.o log.o -o$@
testLog.o:testLog.cpp
	${OCC}
log.o:log.cpp
	${OCC}
clean:
	rm testLog.o log.o testLog
```

这个clean就相当于一个`标签`

同时我们也可以直接通过make调用

`make clean`

# 自动推导

make很强大，他可以`自动推导文件以及文件依赖关系后面的命令`。所以我们没必要在每一个`.o`文件后写上类似的命令，因为make会自动识别并自己推导命令。

比如前面的makefile案例

```makefile
# 变量
objects = hello.o speak.o
${objects}: speak.h
clean:
	rm hello_demo  speak.o  hello.o
```

没必要为每个` .o`文件写依赖那些`.c、.cpp`文件，他会自动推导的(编译命令也是)，当然头文件还是需要的。