# 异常处理 
## 错误
- 语法错误
语法错误就是写代码没按照python规定的要求写，譬如if for 后面少冒号，print少括号等等，这种错误，根本过不了python解释器的语法检测，必须在程序执行前就改正。
```
# 语法错误示范一
a=1
if a >1
    print(a)
# 语法错误示范二
for i in range(0,10)
    print(i)
# 语法错误示范三
def test:
    pass
# 语法错误示范四
print(haha
```
- 逻辑错误
逻辑错误就是语法没问题，但实际运行中得到的结果与事实不符，违背了一些自然逻辑。
```
# TypeError:int类型不可迭代
for i in 3:
    pass
    
# ValueError
num=input(">>: ") #输入hello
int(num)

# NameError
aaa

# IndexError
l=['egon','aa']
l[3]

# KeyError
dic={'name':'egon'}
dic['age']

# AttributeError
class Foo:pass
Foo.x

# ZeroDivisionError:无法完成计算
res1=1/0
res2=1+'str'
```
- 常见错误种类
```
SyntaxError       语法错误
NameError         使用一个还未被赋予对象的变量
TypeError         传入对象类型与要求的不符合
AttributeError    试图访问一个对象没有的属性，比如foo.x，但是foo没有属性x
ValueError        传入一个调用者不期望的值，即使值的类型是正确的
KeyError          试图访问字典里不存在的键
IndexError        下标索引超出序列边界，比如当x只有三个元素，却试图访问x[5]
IOError           输入/输出异常；基本上是无法打开文件
ImportError       无法引入模块或包；基本上是路径问题或名称错误
IndentationError  语法错误（的子类）；代码没有正确对齐
KeyboardInterrupt Ctrl+C被按下
UnboundLocalError 试图访问一个还未被设置的局部变量，基本上是由于另有一个同名的全局变量，导致你以为正在访问它
```
python中所有的错误都是从BaseException类派生的  
[常见的错误类型和继承关系](https://docs.python.org/3/library/exceptions.html#exception-hierarchy)
## 异常
异常就是程序运行时发生错误的信号（在程序出现错误时，则会产生一个异常，若程序没有处理它，则会抛出该异常，程序的运行也随之终止）
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

test

print('异常')

# 输出为：
# Traceback (most recent call last):
#   File "/Users/lisen/PycharmProjects/test/test.py", line 5, in <module>
#     test
# NameError: name 'test' is not defined
```
## 异常处理
为了保证程序的健壮性与容错性，即在遇到错误时程序不会崩溃，我们需要对异常进行处理。
- 如果错误发生的条件是可预知的，我们需要用if进行处理：在错误发生之前进行预防
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

AGE=10
while True:
    age=input('>>: ').strip()
    if age.isdigit(): # 只有在age为字符串形式的整数时,下列代码才不会出错,该条件是可预知的
        age=int(age)
        if age == AGE:
            print('you got it')
            break
```
- 如果错误发生的条件是不可预知的，则需要用到try...except：在错误发生之后进行处理
```
# 基本语法为
try:
    被检测的代码块
except 异常类型1：
    try中一旦检测到对应的异常，就执行这个位置的逻辑
except 异常类型2：
    try中一旦检测到对应的异常，就执行这个位置的逻辑
```
示例：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try: # 下面三个条件，如果有任意一个出现错误，就会终止，并根据相应的错误跳到相应的except，最后执行print
    age=input('输入年龄：')
    int(age)

    l=[1,2]
    l[3]

    dic={'a':1}
    dic['b']

except ValueError as e:
    print('值错误:',e)
except IndexError as e:
    print('索引错误',e)
except KeyError as e:
    print('key错误',e)

print('继续运行我')
```
或者用元祖的方式
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try: # 下面三个条件，如果有任意一个出现错误，就会终止，并根据相应的错误跳到相应的except，最后执行print
    age=input('输入年龄：')
    int(age)

    l=[1,2]
    l[3]

    dic={'a':1}
    dic['b']

except (ValueError,IndexError,KeyError) as e:
    print('多种异常统一处理:',e)

print('继续运行我')
```
- 万能异常Exception 

如果你想要的效果是，无论出现什么异常，我们统一丢弃，或者使用同一段代码逻辑去处理他们，只有一个Exception就足够了。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try: # 下面三个条件，如果有任意一个出现错误，就会终止，并根据相应的错误跳到相应的except，最后执行print
    age=input('输入年龄：')
    int(age)

    # l=[1,2]
    # l[3]

    dic={'a':1}
    dic['b']

# except ValueError as e:
#     print('值错误:',e)
# except IndexError as e:
#     print('索引错误',e)
# except KeyError as e:
#     print('key错误',e)

except Exception as e:
    print('统一来处理',e)

print('继续运行我')
```
或者也可以多分支后来一个万能的Exception来处理其他异常
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try: # 下面三个条件，如果有任意一个出现错误，就会终止，并根据相应的错误跳到相应的except，最后执行print
    age=input('输入年龄：')
    int(age)

    # l=[1,2]
    # l[3]

    dic={'a':1}
    dic['b']

except ValueError as e:
    print('值错误:',e)
except IndexError as e:
    print('索引错误',e)
except Exception as e:
    print('其他的我来处理',e)

print('继续运行我')
```
- else、finally
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try: # 下面三个条件，如果有任意一个出现错误，就会终止，并根据相应的错误跳到相应的except，最后执行print
    age=input('输入年龄：')
    int(age)

    # l=[1,2]
    # l[3]
    #
    # dic={'a':1}
    # dic['b']

except ValueError as e:
    print('值错误:',e)
except IndexError as e:
    print('索引错误',e)
except Exception as e:
    print('其他的我来处理',e)

else:
    print('try内代码块没有异常则执行我')
finally:
    print('无论try内代码是否异常,都会执行该模块,通常是进行清理工作')
```
- 主动触发异常
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

try:
    raise TypeError('类型错误') # 使用TypeError()实例化来主动产生异常
except Exception as e:
    print(e)
```
- 自定义异常
python本身内置多种异常类：TypeError、NameError、ValueError等，我们自己也可以根据实际情况来自定义异常类
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class MyError(BaseException): # 必须的继承一个异常类的基类
    def __init__(self,msg): # 提供一个参数
        self.msg=msg

raise MyError('自定义的异常')
```
如果你要自定义一个类似TypeError的异常类，也可以跟TypeError一样继承Exception
- 断言
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

def test1():
    '一堆逻辑'
    res=1 # 得到结果
    return 1

res1=test1()

assert res1 == 2
# assert 相当于下面的if
# if res1 != 2:
#     raise AssertionError

# 后续一堆代码要根据res1进行操作
```
# 异常处理总结
首先try...except是你附加给你的程序的一种异常处理的逻辑，与你的主要的业务逻辑是没有关系的，这种东西加的多了，会导致你的代码可读性变差

Python内置的try...except...finally用来处理错误十分方便。出错时，会分析错误信息并定位错误发生的代码位置才是最关键的。

然后异常处理本就不是你低端错误的解决手段，只有在错误发生的条件无法预知的情况下，才应该加上try...except
# 作业
选课系统