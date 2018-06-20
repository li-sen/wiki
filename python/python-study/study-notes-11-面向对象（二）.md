# 类的内置方法
## 反射
通过操作字符串的方式来对对象相关属性进行检查，属性名称是字符串类型，得加引号，适用于类或实例。
- hasattr(object,name) 判断对象中有没有一个name字符串对应的方法或者属性
- getattr(object, name, default=None) 获得一个name字符串对应的方法或者属性，如果没有，不设置默认值会报错
- setattr(x,y,z) 修改一个对象x中，属性名为y,属性为z,如果没有就新增
- delattr(x,y) 删除一个对象x中的y属性，如果没有会报错
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Zufang:
    period='year'
    def __init__(self,name,addr):
        self.name=name
        self.addr=addr

    def rent_hourse(self):
        print("%s 正在租房子，地点在%s" %(self.name,self.addr))

z1=Zufang('我爱我家',"文三路")
z1.rent_hourse()

print(hasattr(z1,'name')) # True z1中是否有属性name

print(getattr(z1,"name")) # 我爱我家 得到z1中的name属性
# print(getattr(z1,"aa")) #  没有会报错
print(getattr(z1,"aa",'god job')) #  没有，设置默认值，不会报错

setattr(z1,"name","链家") # 修改name属性
print(z1.name)
setattr(z1,"bb","豪世华邦") # 没有就新增
print(z1.bb)
# 新增实例函数属性
setattr(z1,"func",lambda x:x+1)
setattr(z1,"func1",lambda self:self.name+"高端中介")
print(z1.__dict__)
print(Zufang.__dict__)
print(z1.func(10)) # 11
print(z1.func1(z1)) # 链家高端中介

delattr(z1,'name') # 删除属性
# delattr(z1,'period') # 不能删除类属性
# delattr(z1,"cc") # 没有，会报错
```
反射好处：
可以先定义好接口，后续执行，即插即用
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 譬如这是另一个人写的模块，但出差去了,具体功能还没实现
class FtpClient:
    'ftp客户端功能，还没实现具体的功能'
    def __init__(self,addr):
        self.addr=addr
        print("正在连接服务器%s" %self.addr)

# from module import FtpClient 模拟导入
f1=FtpClient('192.168.1.2')
if hasattr(f1,"get"): # 先检查判断有没有这个方法，有就直接处理
    fun_get=getattr(f1,"get")
    fun_get()
else:
    print("不存在此方法，处理其他逻辑") # 没有就处理其他逻辑，虽然张三出差了，但李四还是照样可以继续他写他的功能模块
```
## attr方法
类的内置attr方法，你没定义就应用默认的，你定义你就用定义的，使用的场景都是一样，只是你根据实际情况能定义不同的处理，一般就__getattr__用的比较多，其他基本没啥用，__getattr__默认的处理方式，如果没有对应的属性名就报错，你可以根据实际情况做处理。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# __getattr__,__setattr__,__delattr__

# class Foo:
#     x=1
#     def __init__(self,y):
#         self.y=y
#
#     def __getattr__(self, item): # 只有在使用点调用,也就是通过实例调用,属性且属性不存在的时候才会触发
#         print("from getattr:你找的属性不存在")
#
# f1=Foo(10)
# print(f1.z) # 不存在z属性，所有执行__getattr__
# # 输出为：
# # from getattr:你找的属性不存在
# # None

# class Foo:
#     x=1
#     def __init__(self,y):
#         self.y=y
#
#     def __delattr__(self, item): # 只有在使用点调用,也就是通过实例调用,删除属性的时候会触发
#         # del self.item # 这样会无限递归，因为只要删除就会触发delattr,而你这里有进行了删除操作，继而又触发delattr,无限循环
#         self.__dict__.pop(item)
#         print("from delattr:删除属性")
#
# f1=Foo(10)
# print(f1.__dict__)
# del f1.y
# del f1.z # 不存在的属性会报错

# class Foo:
#     x=1
#     def __init__(self,y):
#         self.y=y
# 
#     def __setattr__(self, key, value): # 只有在使用点调用,也就是通过实例调用,添加/修改属性会触发它的执行
#         # self.key=value # 这样写会是代码无限递归，从而报错
#         self.__dict__[key]=value # 应该这样写，本质上self.key也是在操作属性字典
#         print("from setattr:添加、修改属性")
# 
# f1=Foo(10)
# print(f1.__dict__)
# f1.z=20
# print(f1.__dict__)
# 输出为：
# from setattr:添加、修改属性
# {'y': 10}
# from setattr:添加、修改属性
# {'y': 10, 'z': 20}
```
## 二次加工标准类型（包装）
python本身提供了丰富的数据类型，以及相关内置方法，但在一些特殊场景我还是得基于标准的数据类型来制定特定的数据类型，譬如:
- 示例1：list append新元素必须是字符串，实现方式也是基于类的继承/派生而实现的
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class List(list):
    def append(self, object):
        self.object=object
        if type(self.object) is str:
            super().append(object)
        else:
            print("添加的元素不是字符串类型")

l1=List("hello")
print(l1)
l1.append("girl")
print(l1)
l1.append(123)
print(l1)
# 输出为：
# ['h', 'e', 'l', 'l', 'o']
# ['h', 'e', 'l', 'l', 'o', 'girl']
# 添加的元素不是字符串类型
# ['h', 'e', 'l', 'l', 'o', 'girl']
```
- 示例2：我要写一个文件，但写入的每行必须带时间前缀，读取采用默认方式，关键点使用__getattr__和getattr
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

import time
class FileHandle:
    def __init__(self,filename,mode='r',encoding='utf-8'):
        self.file=open(filename,mode,encoding=encoding)
        self.mode=mode
        self.encondig=encoding

    def write(self,line):
        t=time.strftime('%Y-%m-%d %X')
        self.file.write('%s %s' %(t,line))

    def __getattr__(self, item):
        return getattr(self.file,item)

f1=FileHandle("test.txt","w+")
f1.write("11111\n")
f1.write("22222\n")
f1.write("33333\n")
```
## isinstance、issubclass
isinstance(obj,cls)检查对象(obj)是否是类(类的对象)

issubclass(sub, super)检查字(sub)类是否是父( super) 类的派生类
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    pass

class Bar(Foo):
    pass

f1=Foo()
print(isinstance(f1,Foo))
b1=Bar()
print(issubclass(Bar,Foo))
print(isinstance(b1,Foo)) # 父类也为真

# 输出为：
# True
# True
# True
```
## \_\_getattr__和__getattribute__
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __getattr__(self, item): # 实例调用属性，属性没有才会触发
        print('执行的是getattr')

    def __getattribute__(self, item): # 无论实例调用的属性有没有，都会触发，如果跟getattr同时存在，只会执行getattribute
        # 除非getattribute在执行过程中通过raise抛出异常AttributeError，才会去执行getattr，防止异常终止
        print('执行的是getattribute')
        raise AttributeError("抛出一个异常")

f1=Foo()
f1.xxxx
# 输出为：
# 执行的是getattribute
# 执行的是getattr
```
## \_\_setitem__,\_\_getitem__,\_\_delitem__
item只能通过字典方式调用才能触发，跟attr类似，attr是通过f1.xx方式
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __getitem__(self, item):
        print("getitem")
        return self.__dict__[item]

    def __setitem__(self, key, value):
        print("setitem")
        self.__dict__[key]=value

    def __delitem__(self, key):
        print("delitem")
        self.__dict__.pop(key)

f1=Foo()
f1["name"]="lisen"
f1["age"]=20
print(f1.__dict__)

f1["name"]

del f1["age"]
print(f1.__dict__)
# 输出为：
# setitem
# setitem
# {'name': 'lisen', 'age': 20}
# getitem
# delitem
# {'name': 'lisen'}
```
## \_\_str__,\_\_repr__,\_\_format__
- 改变对象的字符串显示__str__,\_\_repr__
```
str函数或者print函数--->obj.__str__()
repr或者交互式解释器--->obj.__repr__()
如果__str__没有被定义,那么就会使用__repr__来代替输出
```
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Bar:
    def __init__(self,name,age):
        self.name=name
        self.age=age

    def __str__(self):
        return "str-->名字是%s，年龄是%s" % (self.name,self.age)

    def __repr__(self):
        return "repr-->名字是%s，年龄是%s" %(self.name,self.age)

b1=Bar("lisen",20)
print(b1)
```
> 注意:这俩方法的返回值必须是字符串,否则抛出异常
- format 自定义格式
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

m={
    "ymd":"{0.y}{0.m}{0.d}",
    "y-m-d":"{0.y}-{0.m}-{0.d}"
}

class Date:
    def __init__(self,y,m,d):
        self.y=y
        self.m=m
        self.d=d

    def __format__(self, format_spec):
        if not format_spec or format_spec not in m:
            format_spec = "ymd"
        fm=m[format_spec]
        return fm.format(self)

d1=Date(2018,4,19)
print(format(d1,"y-m-d")) # 本质上是执行d1.__format__
```
## \_\_slots__
```
1. __slots__是一个类变量，变量值可以是列表,元祖,或者可迭代对象,也可以是一个字符串(意味着所有实例只有一个数据属性)
2. 使用点来访问属性本质就是在访问类或者对象的__dict__属性字典(类的字典是共享的,而每个实例的是独立的)；字典会占用大量内存,如果你有一个属性很少的类,但是有很多实例,为了节省内存可以使用__slots__取代实例的__dict__
3. 使用了__slots__实例通过一个很小的固定大小的数组来构建,而不是为每个实例定义一个字典,这跟元组或列表很类似。在__slots__中列出的属性名在内部被映射到这个数组的指定小标上。使用__slots__一个不好的地方就是我们不能再给实例添加新的属性了,只能使用在__slots__中定义的那些属性名。
```
> __slots__的很多特性都依赖于普通的基于字典的实现，定义了__slots__后的类不再支持一些普通类特性了，比如多继承。大多数情况下,一般是用不到的，除非你的实例数量很大。
> __slots__的一个常见误区是它可以作为一个封装工具来防止用户给实例增加新的属性。尽管使用__slots__可以达到这样的目的，但是这个并不是它的初衷，更多的是用来作为一个内存优化工具。
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    # __slots__ = 'name'
    __slots__ = ['name','age']

f1=Foo()
f1.name='lisen'

print(f1.name) # {'__module__': '__main__', '__slots__': ['name', 'age'], 'age': <member 'age' of 'Foo' objects>, 'name': <member 'name' of 'Foo' objects>, '__doc__': None}
print(Foo.__dict__)
# print(f1.__dict__) # 报错 AttributeError: 'Foo' object has no attribute '__dict__'
# f1.gender='male' # 报错 AttributeError: 'Foo' object has no attribute 'gender'
```
## \_\_doc__
```
这只是类的描述信息，描述这个的作用及功能，无法继承该属性(__doc__)
```
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    "这是测试"
    pass
print(Foo.__doc__)  # 查看Foo类的注释

class Bar(Foo):
    pass
print(Bar.__doc__)  # 没有注释信息，打印的是None
```
## \_\_module__、\_\_class__
```
__module__ 表示当前操作的对象在那个模块
__class__ 表示当前操作的对象的类是什么
```
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

from lib.aa import Bar

b1=Bar()
print(b1.name)

print(b1.__class__)
print(b1.__module__)
# 输出为：
# bar
# <class 'lib.aa.Bar'>
# lib.aa
```
## \_\_del__
这为析构方法，当对象在内存中被删除时就会被自动触发执行
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __init__(self,name):
        self.name=name
    def __del__(self):
        print('我执行了哈哈哈')

f1=Foo('lisen')
# print('----->') # 当解释器执行完毕后，回收f1实例的内存是 触发 __del__
# 输出为：
# ----->
# 我执行了哈哈哈

# del f1.name # 删除字典，不会触发
del f1 # 删除了实例f1，触发__del__
print('----->') # 当解释器执行完毕后，回收f1实例的内存是 触发 __del__
# 输出为：
# 我执行了哈哈哈
# ----->
```
> python为高级语言，内存分配和回收有解释器自己完成，使用者无需关心
## \_\_call__
对象后面加括号，触发执行。
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __call__(self, *args, **kwargs):
        print('__call__执行啦')

f1=Foo()

f1() # 执行的是Foo下的 __call__
# 输出为：__call__执行啦
Foo() # 执行的object下的 __call__
```
## \_\_next__和__iter__

```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __init__(self, n):
        self.n = n

    def __iter__(self):
        return self

    def __next__(self):
        if self.n == 13:
            raise StopIteration('终止了')  # 抛出异常
        self.n += 1
        return self.n


f1 = Foo(10)
print(f1.__next__())
print(f1.__next__())
print(f1.__next__())
print(f1.__next__())

# for i in f1: # 本质上是先obj=iter(f1)-->f1.__iter__()
#     print(i) # obj.__next__()
```
示例：
```
#! /usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Fib:
    def __init__(self):
        self.a=1
        self.b=1

    def __iter__(self):
        return self
    def __next__(self):
        self.a,self.b=self.b,self.a+self.b # 变量交换
        return self.a
f1=Fib()
print(next(f1))
print(next(f1))
print(next(f1))
print(next(f1))
print(next(f1))
print(next(f1))
```
## 描述符（\_\_get__,\_\_set__,\_\_delete__）
- 描述符本质就是一个新式类，在这个新式类中，至少实现了__get__(),\_\_set__(),\_\_delete__()中的一个方法，也称为描述符协议。
```
__get__()调用一个属性时，触发
__set__()为一个属性赋值是，触发
__delete__()del删除属性时，触发
```
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo: # python3 都是新式类
    def __get__(self, instance, owner):
        print("from __get__")

    def __set__(self, instance, value):
        print("from __set__")

    def __delete__(self, instance):
        print("from __delete__")

f1=Foo()
f1.name='lisen'  # 自己调用自己是不会触发这三种方法的

class Bar:
    x=Foo() # 在另一个类Bar中定义被描述的属性x，没被描述的属性则无效。

b1=Bar()
b1.x  # 调用触发
b1.x='lisen'  # 赋值触发
del b1.x # del删除触发
```
- 描述符种类
  - 数据描述符：至少实现了__get__()、\_\_set__()
  - 非数据描述符：没有实现了__set__()
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo: # 数据描述符
    def __get__(self, instance, owner):
        pass
    def __set__(self, instance, value):
        pass

class Bar: # 非数据描述符
    def __get__(self, instance, owner):
        pass
```
- 注意点：
```
一、描述符本身应该定义成新式类，被代理的类也也该是新式类。

二、必须把描述符定义成这个类的类属性，不能定义到构造函数中。

三、要严格遵循该优先级，优先级由高到低分别是。

1.类属性

2.数据描述符

3.实例属性

4.非数据描述符

5.找不到的属性会触发__getattr__()
```
1. 类属性优先级高于数据描述符
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:  # python3 都是新式类
    def __get__(self, instance, owner):
        print("from __get__")

    def __set__(self, instance, value):
        print("from __set__")

    def __delete__(self, instance):
        print("from __delete__")


class Bar:
    name = Foo()

    def __init__(self, name):
        self.name = name


Bar.name  # 调用类属性name,本质就是在调用描述符Foo,触发了__get__()

Bar.name = 'lisen'  # 重新赋值了Bar.name，所以不会触发，所以类属性优先级高于数据描述符
```
2. 数据描述符优先级高于实例属性
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:  # python3 都是新式类
    def __get__(self, instance, owner):
        print("from __get__")

    def __set__(self, instance, value):
        print("from __set__")

    def __delete__(self, instance):
        print("from __delete__")


class Bar:
    name = Foo()

    def __init__(self, name):
        self.name = name


b1 = Bar("lisen")  # name会被代理，触发__set__()
print(b1.__dict__)  # 如果描述符是一个数据描述符(即有__get__又有__set__),那么b1.name的调用与赋值都是触发描述符的操作,与实例b1本身无关了,相当于覆盖了实例的属性。
```
3. 实例属性优先级高于非数据描述符
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:  # python3 都是新式类
    def __get__(self, instance, owner):
        print("from __get__")


class Bar:
    name = Foo()

    def __init__(self, name):
        self.name = name


b1 = Bar("lisen")  # name会被代理，但Foo方法没有实现set方法，所以使用实例本身的set方法
print(b1.__dict__) # {'name': 'lisen'}
```
> pycharm做的好事（坑）：一旦你新建项目，pycharm会将新项目的路径加入到sys.path中；为了避免找不到模块，正确方法是在运行的文件中引入base_dir
```
base_dir=os.path.dirname(os.path.dirname(os.path.abspath(__file__)))
sys.path.append(base_dir)
```
## 描述符的应用
一般函数的参数是没类型限制的，如果有需要可以利用描述符对函数的参数做类型限制

- 第一步：使用描述符后能将实例相关的属性关联到属性字典，之前描述符并没关联到实例的属性字典 
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen


class Str:
    def __init__(self,name):
        self.name=name

    def __get__(self, instance, owner):
        print("from __get__",instance,owner)
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print("from __set__",instance,value)
        instance.__dict__[self.name]=value

    def __delete__(self, instance):
        print("from __delete__",instance)
        instance.__dict__.pop(self.name)

class People:
    name=Str('name')
    def __init__(self,name,age,salary):
        self.name=name
        self.age=age
        self.salary=salary

p1=People('lisen',18,3333.3)


p1.name # __get__
print(p1.__dict__)

p1.name='zhangsan' # __set__
print(p1.__dict__)

del p1.name # __delete__
print(p1.__dict__)

People.name  # 报错，类去操作属性时，会把None传给instance，需要做判断，后续在做优化。
```
- 第二步：对参数做类型限制
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Typed:
    def __init__(self,name,expected_type): # 新增期望类型参数
        self.name=name
        self.expected_type=expected_type

    def __get__(self, instance, owner):
        print("from __get__",instance,owner)
        if instance is None:  # 优化类直接操作属性时报错：People.name
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print("from __set__",instance,value)
        if not isinstance(value,self.expected_type):
            raise TypeError('%s is not %s' %(value,self.expected_type))
        instance.__dict__[self.name]=value

    def __delete__(self, instance):
        print("from __delete__",instance)
        instance.__dict__.pop(self.name)

class People:
    name=Typed('name',str) # 新增参数类型限制
    age=Typed('age',int) # 新增参数类型限制
    salary=Typed('salary',float) # 新增参数类型限制
    def __init__(self,name,age,salary):
        self.name=name
        self.age=age
        self.salary=salary

# p1=People('lisen',18,3333.3)
# p1=People(111,18,3333.3) # 报错 对name的类型限制成功
# p1=People('lisen','aa',3333.3) # 报错 限制成功
p1=People('lisen',18,3333) # 报错 限制成功
```
上面虽然完成了对相关参数的类型限制，但方式很丑陋，这里目前只有三个参数，如果有多个，得在People类中一个个新增参数类型限制，太麻烦了；有没有更好的方法？有，类装饰器！
## 类装饰器
之前学过了函数的装饰器，通过高阶函数、函数嵌套以及闭包完美的完成了相关需求，类装饰器也类似
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Typed:
    def __init__(self,name,expected_type): # 新增期望类型参数
        self.name=name
        self.expected_type=expected_type

    def __get__(self, instance, owner): # owner为类People
        print("from __get__",instance,owner)
        if instance is None:  # 优化类直接操作属性时报错：People.name
            return self
        return instance.__dict__[self.name]

    def __set__(self, instance, value):
        print("from __set__",instance,value)
        if not isinstance(value,self.expected_type):
            raise TypeError('%s is not %s' %(value,self.expected_type))
        instance.__dict__[self.name]=value

    def __delete__(self, instance):
        print("from __delete__",instance)
        instance.__dict__.pop(self.name)

# 有参装饰器
def deco(**kwargs):
    def wrapper(obj):
        for key,val in kwargs.items():
            setattr(obj,key,Typed(key,val))
        return obj
    return wrapper

@deco(name=str,age=int,salary=float) # 本质上是运行People=wrapper(People)
# 实际运行的是wrapper(return wrapper)，但此时的wrapper因为是函数嵌套的关系会多两个参数name=str,age=int也就是kwargs，
# 然后运行wrapper(obj)，obj就是类People(装饰器不能改变调用方式)；
# 最后依次加入到类的属性字典里譬如：'name': <__main__.Typed object at 0x1024e3be0>，会进行Typed验证
class People:
    def __init__(self,name,age,salary):
        self.name=name
        self.age=age
        self.salary=salary

print(People.__dict__)
# p1=People('lisen',18,3333.3) # 正确参数
# p1=People(111,18,3333.3) # 报错 对name的类型限制成功
# p1=People('lisen','aa',3333.3) # 报错 限制成功
p1=People('lisen',18,3333) # 报错 限制成功
```
> 描述符是可以实现大部分python类特性中的底层魔法,包括@classmethod,@staticmethd,@property甚至是__slots__属性，描述符也是很多高级库和框架的重要工具之一，通常是使用到装饰器或者元类的大型框架中的一个组件。
> 简单来说，一般工作中基本用不到，除非你是大牛自己写框架。。。

示例：
- 自定义一个@property 实现延迟计算（本质就是把一个函数属性利用装饰器原理做成一个描述符：类的属性字典中函数名为key，value为描述符类产生的对象）
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Lazyproperty:  # 自定义的property
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        print('get')
        # print(instance) # <__main__.Room object at 0x107bb5b00>
        # print(owner) # <class '__main__.Room'>
        # print(self.func) # <function Room.area at 0x107bb89d8>
        if instance is None:  # 防止类直接调用报错，之前提过
            return self
        res = self.func(instance)
        setattr(instance, self.func.__name__, res)  # 把结果设置到实例属性字典中，下次再计算直接取，不用重复去计算，也就是所谓的延迟计算。。。
        return res
    # def __set__(self, instance, value): # 不能设置__set__方法，不然成了数据描述符他的优先级是高于实例属性字典的，这样会造成延迟计算失败，因为它每次计算会走__set__方法，而你上次的计算的结果是放置在实例字典中的
    #     pass


class Room:
    def __init__(self, name, width, length):
        self.name = name
        self.width = width
        self.length = length

    @Lazyproperty  # area=Lazypropery(area) 定义了一个类属性，相当于area被非数据描述符Lazyproperty给代理了
    def area(self):
        return self.width * self.length


r1 = Room('厕所', 1, 1)

# 实例调用
print(r1.area)
print(Room.__dict__)

# 类调用
print(Room.area)

# 延迟计算
print(r1.area)  # 只会打印一次get，后面的直接从实例属性字典里取
print(r1.__dict__)

print(r1.area)
print(r1.area)
print(r1.area)
```
- 利用描述符原理完成一个自定制@classmethod
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class ClassMethod:
    def __init__(self,func):
        self.func=func

    def __get__(self, instance, owner): #类来调用,instance为None,owner为类本身,实例来调用,instance为实例,owner为类本身,
        def feedback(*args,**kwargs):
            print('在这里可以加功能啊...')
            return self.func(owner,*args,**kwargs)
        return feedback

class People:
    name='lisen'
    @ClassMethod # say_hi=ClassMethod(say_hi)
    def say_hi(cls,msg):
        print('你好啊,帅哥 %s %s' %(cls.name,msg))

People.say_hi('你是那偷心的贼')

p1=People()
p1.say_hi('你是那偷心的贼')
```
- 利用描述符原理完成一个自定制的@staticmethod
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class StaticMethod:
    def __init__(self,func):
        self.func=func

    def __get__(self, instance, owner): #类来调用,instance为None,owner为类本身,实例来调用,instance为实例,owner为类本身,
        def feedback(*args,**kwargs):
            print('在这里可以加功能啊...')
            return self.func(*args,**kwargs)
        return feedback

class People:
    @StaticMethod# say_hi=StaticMethod(say_hi)
    def say_hi(x,y,z):
        print('------>',x,y,z)

People.say_hi(1,2,3)

p1=People()
p1.say_hi(4,5,6)
```

## 再看property
一个静态属性property本质就是实现了get，set，delete三种方法
- 方式一
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    @property
    def AAA(self):
        print('get的时候运行我啊')

    @AAA.setter
    def AAA(self,value):
        print('set的时候运行我啊')

    @AAA.deleter
    def AAA(self):
        print('delete的时候运行我啊')

#只有在属性AAA定义property后才能定义AAA.setter,AAA.deleter
f1=Foo()
f1.AAA
f1.AAA='aaa'
del f1.AAA
```
- 方式二
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def get_AAA(self):
        print('get的时候运行我啊')

    def set_AAA(self,value):
        print('set的时候运行我啊')

    def delete_AAA(self):
        print('delete的时候运行我啊')
    AAA=property(get_AAA,set_AAA,delete_AAA) # 内置property三个参数与get,set,delete顺序要一一对应

f1=Foo()
f1.AAA
f1.AAA='aaa'
del f1.AAA
```
- 应用
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Goods:

    def __init__(self):
        # 原价
        self.original_price = 100
        # 折扣
        self.discount = 0.8

    @property
    def price(self):
        # 实际价格 = 原价 * 折扣
        new_price = self.original_price * self.discount
        return new_price

    @price.setter
    def price(self, value):
        self.original_price = value

    @price.deleter
    def price(self):
        del self.original_price


obj = Goods()
obj.price  # 获取商品价格
obj.price = 200  # 修改商品原价
print(obj.price)
del obj.price  # 删除商品原价
```
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class People:
    def __init__(self,name):
        self.name=name #实例化就触发property

    @property
    def name(self):
        # return self.name #无限递归
        print('get------>')
        return self.DouNiWan

    @name.setter
    def name(self,value):
        print('set------>')
        if not isinstance(value,str):
            raise TypeError('必须是字符串类型')
        self.DouNiWan=value

    @name.deleter
    def name(self):
        print('delete------>')
        del self.DouNiWan

p1=People('lisen') #self.name实际是存放到self.DouNiWan里
print(p1.__dict__)
p1.name='zhangsan'
print(p1.__dict__)
```
## \_\_enter__和__exit__
一般对文件操作我们都会这样操作
```
with open('a.txt','r',encoding='utf8') as f:
    pass
```
这种with语句就叫做上下文管理协议，为了让一个对象兼容with语句，必须在这个对象的类中声明__enter__和__exit__方法
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __init__(self,name):
        self.name=name

    def __enter__(self): # with一运行就触发__enter__，如果有返回值就赋值给as的对象f
        print("from __enter__")
        return self  # 不返回值给f，f则为空，后面print(f.name)会报错

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("from __exit__")

with Foo("a.txt") as f:
    print(f)
    print(f.name)

print("aaaa")
# 输出为：
# from __enter__
# <__main__.Foo object at 0x10692b630>
# a.txt
# from __exit__
# aaaa
```
\_\_exit__()中的三个参数分别代表异常类型(exc_type)，异常值(exc_val)和追溯信息(exc_tb),即with语句中代码块出现异常，则with后的代码都无法执行
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __init__(self,name):
        self.name=name

    def __enter__(self): # with一运行就触发__enter__，如果有返回值就赋值给as的对象f
        print("from __enter__")
        return self  # 不返回值给f，f则为空，后面print(f.name)会报错

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("from __exit__")
        print(exc_tb) # NameError
        print(exc_val) # name 'bbbb' is not defined
        print(exc_tb) # Traceback (most recent call last)

with Foo("a.txt") as f:
    print(bbbb) # 报错了后续代码不执行了

print("aaaa")
```
如果__exit__()返回值为True，那么异常会被清空，就好像啥都没发生一样，with后语句正常执行
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Foo:
    def __init__(self,name):
        self.name=name

    def __enter__(self): # with一运行就触发__enter__，如果有返回值就赋值给as的对象f
        print("from __enter__")
        return self  # 不返回值给f，f则为空，后面print(f.name)会报错

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("from __exit__")
        print(exc_tb) # NameError
        print(exc_val) # name 'bbbb' is not defined
        print(exc_tb) # Traceback (most recent call last)
        return True   # 吞下异常

with Foo("a.txt") as f:
    print(bbbb) # 因为__exit__()返回值为True，后续代码正常执行

print("aaaa")
# 输出为：
# from __enter__
# from __exit__
# <traceback object at 0x1103c0d88>
# name 'bbbb' is not defined
# <traceback object at 0x1103c0d88>
# aaaa
```
> 使用上下文管理的最大好处：在需要管理一些资源比如文件，网络连接和锁的编程环境中，可以在__exit__中定制自动释放资源的机制。
## metaclass
python中一切皆对象，类也不例外；类一般都是使用关键字class生成，也可以使用python内建元类type生成，如果内建元类type不能满足你需求，你也可以自定义一个元类，来定制化产生的类。
### type
type 接收三个参数：  
第 1 个参数是字符串 ‘FFo’，表示类名  
第 2 个参数是元组 (object, )，表示所有的父类  
第 3 个参数是字典，表示定义类的属性和方法  
补充：若FFo类有继承，即class FFo(Bar):.... 则等同于type('Foo',(Bar,),{})
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 使用class 关键字产生的类
class Foo(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def test(self):
        print('=====>')
print(type(Foo)) # <class 'type'> 有元类type产生

# 使用typy自定义产生类的大概流程
def __init__(self, name, age):  # 定义构造方法
    self.name = name
    self.age = age

def test(self):  # 定义类方法属性
    print('=====>')

FFo = type('FFo', (object,), {'__init__': __init__, 'test': test})

f1 = Foo('lisen', 18)
f2 = FFo('zhangsan', 20)

print(f1.__dict__)
print(f2.__dict__)

print(Foo.__dict__)
print(FFo.__dict__)
```
### 自定义metaclass
简单实现：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class MyType(type):
    def __init__(self,a,b,c):
        print(self) # Foo
        print('元类的构造函数执行')
        # print(a) # 类名
        # print(b) # 继承的父类
        # print(c) # 属性字典
    def __call__(self, *args, **kwargs):
        obj=object.__new__(self) # 产生的新对象 = object.__new__(继承object类的子类)
        self.__init__(obj,*args,**kwargs) # 触发类Foo下的__init__
        return obj
class Foo(metaclass=MyType): # Foo=MyType(Foo,'Foo',(),{})--->__call__
    def __init__(self,name):
        self.name=name
f1=Foo('alex')
```
### 自定义metaclass示例
在元类中控制自定义的类无需__init__方法
1. 元类帮其完成创建对象，以及初始化操作；
2. 要求实例化时传参必须为关键字形式，否则抛出异常TypeError: must use keyword argument
3. key作为用户自定义类产生对象的属性，且所有属性变成大写
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

class Mymetaclass(type):
    # 在元类中控制把自定义类的数据属性都变成大写，可以用下面注释这段
    # def __new__(cls,name,bases,attrs):
    #     update_attrs={}
    #     for k,v in attrs.items():
    #         if not callable(v) and not k.startswith('__'):
    #             update_attrs[k.upper()]=v
    #         else:
    #             update_attrs[k]=v
    #     return type.__new__(cls,name,bases,update_attrs)

    def __call__(self, *args, **kwargs):
        if args:
            raise TypeError('must use keyword argument for key function')
        obj = object.__new__(self) #创建对象，self为类Foo

        for k,v in kwargs.items():
            obj.__dict__[k.upper()]=v
        return obj

class Chinese(metaclass=Mymetaclass):
    country='China'
    tag='Legend of the Dragon' #龙的传人
    def walk(self):
        print('%s is walking' %self.name)


# p=Chinese('egon',18,'male') # 报错
p=Chinese(name='egon',age=18,sex='male')
print(p.__dict__)
```