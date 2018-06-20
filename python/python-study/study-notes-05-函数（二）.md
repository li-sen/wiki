# 全局变量
- 在子程序（函数）中定义的变量称为局部变量，在程序的一开始定义的变量称为全局变量。
- 全局变量作用域是整个程序，局部变量作用域是定义该变量的子程序。
- 就近原则：当全局变量与局部变量同名时，在定义局部变量的子程序内，局部变量起作用；在其它地方全局变量起作用。
> 一般全局变量大写，局部变量小写。
## 示例说明
```
name = "lisen" # 顶头写的为全局变量，作用域为全局
def change_name():
    name = "李森" # 函数中的变量为局部变量，作用域为函数内部
    print("局部",name)
change_name()
print(name)
# 输出为：
# 我的名字 李森
# lisen
```
## global/nonlocal
- global
```
name = "lisen"
def change_name():
    global name # 在函数中引用全局变量name，并将其重新赋值
    name = "zhangsan"
    print(name)
change_name()
print(name)
# 输出为：
# zhangsan
# zhangsan
```
> 对于不可变类型，有global引用，函数才可以对全局变量进行操作；对于不可变类型，函数可以直接引用全局变量对于其进行元素更改。
```
NAME = ["李森","张三"]
def foo():
    NAME.append('王五')
    print( NAME)
foo()
print(NAME)
# 输出为：
# ['李森', '张三', '王五']
# ['李森', '张三', '王五']
```
- nonlocal
```
name = "lisen"
def change_name():
    name = "wangwu"
    def aa():
        nonlocal name # nonlocal，指定上一级变量，如果没有就继续往上直到找到为止,但不会影响全局变量
        name = "zhangsan"
        print(name)
    aa()
    print(name)
change_name()
print(name)
# 输出为：
# zhangsan
# zhangsan
# lisen
```
# 函数即变量
先定义后调用原则，定义函数也就是定义变量，调用之前得先定义加载
```
# 报错NameError: global name 'logger' is not defined
# def foo():
#     print('from foo')
#     bar()
# foo()
# def bar():
#     print('from bar')

# 正确方式
# def foo():
#     print('from foo')
#     bar()
#
# def bar():
#     print('from bar')
# foo()

# 正确方式
# def bar():
#     print('from bar')
# def foo():
#     print('from foo')
#     bar()
# foo()
```
# 函数递归调用
在函数内部，可以调用其他函数。如果一个函数在内部调用自身本身，这个函数就是递归函数。
```
# def calc(n):
#     print(n)
#     if int(n / 2) == 0:
#         return n
#     return calc(int(n / 2))
# 
# calc(10)
# 输出为：
# 10
# 5
# 2
```
## 递归特性
- 必须有一个明确的结束条件
- 每次进入更深一层递归时，问题规模相比上次递归都应有所减少
- 递归效率不高，递归层次过多会导致栈溢出（在计算机中，函数调用是通过栈（stack）这种数据结构实现的，每当进入一个函数调用，栈就会加一层栈帧，每当函数返回，栈就会减一层栈帧。由于栈的大小不是无限的，所以，递归调用的次数过多，会导致栈溢出）

# 嵌套函数和作用域
```
name = "Alex"
def change_name():
    name = "Alex2"

    def change_name2():
        name = "Alex3"
        print("第3层打印", name)

    change_name2()  # 调用内层函数
    print("第2层打印", name)

change_name()
print("最外层打印", name)
# 输出为：
# 第3层打印 Alex3
# 第2层打印 Alex2
# 最外层打印 Alex
```
> 多层函数嵌套：子级函数，只在子级函数内部生效；父级函数能调用子级函数的功能。  
此时，在最外层调用change_name2()会报错了,报is not defined,因为函数的作用域在定义函数时就已经固定住了，不会随着调用位置的改变而改变
```
# 示例：
# name='alex'
# 
# def foo():
#     name='lhf'
#     def bar():
#         name='wupeiqi'
#         def tt():
#             print(name)
#         return tt
#     return bar
# 
# func=foo()
# func()()
```