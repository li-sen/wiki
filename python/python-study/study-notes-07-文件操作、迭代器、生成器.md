# 文件操作
## 文件操作流程
1. 打开文件，得到文件句柄并赋值给一个变量
2. 通过句柄对文件进行操作
3. 关闭文件
## python中操作文件
```
# 打开文件，得到文件句柄并赋值给一个变量
f = open("test.txt", "r", encoding="utf-8") # 默认打开模式就为r
# 通过句柄对文件进行操作
data = f.read()
print(data)
# 关闭文件
f.close()
```
## 文件句柄/文件描述符
文件句柄:是windows下概念,在linux/unix下没有句柄这一说法；  
在linux/unix下都是"文件描述符",内核（kernel）利用文件描述符（file descriptor）来访问文件。文件描述符是非负整数。打开现存文件或新建文件时，内核会返回一个文件描述符。读写文件也需要使用文件描述符来指定待读写的文件。
## 注意点
- 资源回收
```
打开一个文件包含两部分资源：操作系统级打开的文件+应用程序的变量。在操作完毕一个文件时，必须把与该文件的这两部分资源一个不落地回收，回收方法为：
1、f.close() #回收操作系统级打开的文件
2、del f #回收应用程序级的变量

其中del f一定要发生在f.close()之后，否则就会导致操作系统打开的文件还没有关闭，白白占用资源，
而python自动的垃圾回收机制决定了我们无需考虑del f，这就要求我们，在操作完毕文件后，一定要记住f.close()
推荐使用with方式，会自动关闭文件回收资源。
with open('a.txt','w') as f:
    pass
 
with open('a.txt','r') as read_f,open('b.txt','w') as write_f:
    data=read_f.read()
    write_f.write(data)
```
- 字符编码
```
f=open('a.txt','r',encoding='utf-8')
f=open(...)是由操作系统打开文件，那么如果我们没有为open指定编码，那么打开文件的默认编码很明显是操作系统说了算了，操作系统会用自己的默认编码去打开文件，在windows下是gbk，在linux下是utf-8。
```
> 要保证不乱码，文件以什么方式存的，就要以什么方式打开。
## 打开文件的模式
```
#1. 打开文件的模式有(默认为文本模式)：
r ，只读模式【默认模式，文件必须存在，不存在则抛出异常】
w，只写模式【不可读；不存在则创建；存在则清空内容】
a，追加写模式【不可读；不存在则创建；存在则只追加内容】

#2. 对于非文本文件，我们只能使用b模式，"b"表示以字节的方式操作（而所有文件也都是以字节的形式存储的，使用这种模式无需考虑文本文件的字符编码、图片文件的jgp格式、视频文件的avi格式）
rb 
wb
ab
注：以b方式打开时，读取到的内容是字节类型，写入时也需要提供字节类型，不能指定编码

#3. 了解部分
"+" 表示可以同时读写某个文件
r+， 读写【可读，可写】
w+，写读【可读，可写】
a+， 写读【可读，可写】


x， 只写模式【不可读；不存在则创建，存在则报错】
x+ ，写读【可读，可写】
xb
```
示例：
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# with open("a.txt","r",encoding="utf-8") as f:
#     print(f.read) # 文件不存在报错

# with open("a.txt","w",encoding="utf-8") as f:
#     f.write("aaa") # w,只写不可读；不存文件就创建，存在则清空写入新内容aaa

# with open("a.txt","x",encoding="utf-8") as f:
#     f.write("aaaa") # x，只写不可读，不存在就创建，存在则报错
#
# with open("a.txt","a",encoding="utf-8") as f:
#     f.write("11111") # a,追加不可读，不存在就创建，存在往末尾追加

# b模式：
# with open("a.txt","rb") as f: # b模式不能指定编码
#     data = f.read()
#     print(type(data),data) # 打印出读取的内容，为数据类型为字节
#     str_data=str(data,encoding="utf-8") #字节转化为字符串，指定编码
#     print(str_data)
# with open("a.txt","wb") as f: #清空文件内容新增内容
#     str_data = "李森"
#     byte_data = bytes(str_data,encoding="utf-8") # 将字符串转为字节类型
#     f.write(byte_data)
# with open("a.txt","ab") as f: # 追加行尾内容
#     str_data = "张三"
#     byte_data = bytes(str_data,encoding="utf-8") # 将字符串转为字节类型
#     f.write(byte_data)
```

## 操作文件的方法
```
#掌握
f.read() #读取所有内容,光标移动到文件末尾
f.readline() #读取一行内容,光标移动到第二行首部
f.readlines() #读取每一行内容,存放于列表中

f.write('1111\n222\n') #针对文本模式的写,需要自己写换行符
f.write('1111\n222\n'.encode('utf-8')) #针对b模式的写,需要自己写换行符
f.writelines(['333\n','444\n']) #文件模式
f.writelines([bytes('333\n',encoding='utf-8'),'444\n'.encode('utf-8')]) #b模式

#了解
f.readable() #文件是否可读
f.writable() #文件是否可读
f.closed #文件是否关闭
f.encoding #如果文件打开模式为b,则没有该属性
f.flush() #立刻将文件内容从内存刷到硬盘
f.name
```
## 文件内光标移动
1. read(3)：
   - 文件打开方式为文本模式时，代表读取3个字符
   - 文件打开方式为b模式时，代表读取3个字节
   > 其余的文件内光标移动都是以字节为单位如seek，tell，truncate
2. tell/seek：
   - f.tell()   读取当前文件光标的位置
   - f.seek(0)  设置当前文件光标的位置  
    seek有三种移动方式0，1，2，默认为0，表示从0开始，1表示相对位置，从上次光标位置开始，2表示从文件末尾开始；其中1和2必须在b模式下进行，但无论哪种模式，都是以bytes为单位移动的
   - truncate是截断文件，所以文件的打开方式必须可写，但是不能用w或w+等方式打开，因为那样直接清空文件了，所以truncate要在r+或a或a+等模式下测试效果
```
# with open('1.txt', 'r+', encoding='utf-8') as f:
#     print(f.tell())  # 打印下 文件开始时候指针指向哪里 这里指向 0
#     print(f.read())  # 读出文件内容'字符串'，
#     print(f.tell())  # 文件指针指到 9，一个汉子三个字符串，指针是以字符为单位
#     f.write('科比')  # 写入内容'科比',需要特别注意此时文件指到文件末尾去了
#     print(f.read())  # 指针到末尾去了，所以读取的内容为空
#     print(f.tell())  # 指针指到15
#     f.seek(0)  # 将指针内容指到 0 位置
#     print(f.read())  # 因为文件指针指到开头去了，所以可以读到内容 字符串科比
# 
# # w+形式 存在的话先清空 一写的时候指针到最后
# with open('1.txt', 'w+') as f:
#     f.write('Kg')  # 1.txt存在，所以将内面的内容清空，然后再写入 'kg'
#     print(f.tell())  # 此时指针指向2
#     print(f.read())  # 读不到内容，因为指针指向末尾了
#     f.seek(0)
#     print(f.read())  # 读到内容，因为指针上一步已经恢复到起始位置

# f=open('seek.txt','rb')
# print(f.tell())
# f.seek(10,1)
# print(f.tell())
# f.seek(3,1)
# print(f.tell())


# f=open('seek.txt','rb')
# print(f.tell())
# f.seek(-5,2)
# print(f.read())
# print(f.tell())
# f.seek(3,1)
# print(f.tell())
# 
# # a+打开的时候指针已经移到最后，写的时候不管怎样都往文件末尾追加
# # x+文件存在的话则报错
```
## 文件的修改
文件的数据是存放于硬盘上的，因而只存在覆盖、不存在修改这么一说，我们平时看到的修改文件，都是模拟出来的效果，具体的说有两种实现方式：
- 方式一：将硬盘存放的该文件的内容全部加载到内存，在内存中是可以修改的，修改完毕后，再由内存覆盖到硬盘（word，vim，nodpad++等编辑器）
```
import os

with open('a.txt') as read_f,open('.a.txt.swap','w') as write_f:
    data=read_f.read() #全部读入内存,如果文件很大,会很卡
    data=data.replace('alex','SB') #在内存中完成修改

    write_f.write(data) #一次性写入新文件

os.remove('a.txt')
os.rename('.a.txt.swap','a.txt')
```
- 方式二：将硬盘存放的该文件的内容一行一行地读入内存，修改完毕就写入新文件，最后用新文件覆盖源文件
```
import os

with open('a.txt') as read_f,open('.a.txt.swap','w') as write_f:
    for line in read_f:
        line=line.replace('alex','SB')
        write_f.write(line)

os.remove('a.txt')
os.rename('.a.txt.swap','a.txt')
```

## 练习
1. 利用b模式，编写一个cp工具，要求如下：
　 - 既可以拷贝文本又可以拷贝视频，图片等文件
　 - 用户一旦参数错误，打印命令的正确使用方法，如usage: cp source_file target_file
   >　可以用import sys，然后用sys.argv获取脚本后面跟的参数
   ```
   vim copy.py
   #!/usr/bin/env python
   # -*- coding utf-8 -*-

   import sys
   if len(sys.argv) != 3:
       print("argv is error:usage: cp source_file target_file")
       sys.exit()
   sf,tf=sys.argv[1],sys.argv[2]
   with open(sf,"rb") as r_f,open(tf,"wb") as w_f:
       for line in r_f:
           w_f.write(line)
   ```
2. 基于seek实现tail -f功能
   ```
   #!/usr/bin/env python
   # -*- coding: utf-8 -*-
   # Author: Li Sen
   import time,sys
   if len(sys.argv) != 2:
       print("usage: tailfile filename")
       sys.exit()
   filename=sys.argv[1]
   
   with open(filename,"rb") as f:
       f.seek(0,2)
       while True:
           line = f.readline()
           if line:
               print(line.decode("utf-8"))
           else:
               time.sleep(1)
   ```
3. 修改文件
   ```
   import os
   # with open("a.txt") as read_f,open("a.txt.swap","w") as write_f:
   #     data = read_f.read() #一次全部读入内存，文件大会卡
   #     data = data.replace('alex',"lisen")
   #     write_f.write(data)
   # os.remove("a.txt")
   # os.rename("a.txt.swap","a.txt")
   
   with open("a.txt") as read_f,open("a.txt.swap","w") as wirte_f:
       for line in read_f:
           line = line.replace('lisen',"shuaige")
           wirte_f.write(line)
   os.remove("a.txt")
   os.rename("a.txt.swap","a.txt")
   ```
# 迭代器
## 什么是迭代器
迭代器是访问集合元素的一种方式。  
迭代器对象从集合的第一个元素开始访问，直到所有的元素被访问完结束。迭代器只能往前不会后退，不过这也没什么，因为人们很少在迭代途中往后退。另外，迭代器的一大优点是不要求事先准备好整个迭代过程中所有的元素。迭代器仅仅在迭代到某个元素时才计算该元素，而在这之前或之后，元素可以不存在或者被销毁。这个特点使得它特别适合用于遍历一些巨大的或是无限的集合，比如几个G的文件
```
while True: #只是单纯地重复，因而不是迭代
    print('===>') 
    
l=[1,2,3]
count=0
while count < len(l): #迭代
    print(l[count])
    count+=1
```
## 特点：
- 访问者不需要关心迭代器内部的结构，仅需通过next()方法不断去取下一个内容
- 不能随机访问集合中的某个值，只能从头到尾依次访问
- 访问到一半时不能往回退
- 便于循环比较大的数据集合，节省内存

## 两个基本方法:iter ,next 方法
- 内置函数iter(),next()  本质上都是用的对象.__iter__(),__next__()的方法
- 内置函数 iter(iterable)，表示把可迭代对象 变成迭代器(iterator)
- 内置函数next(iterator) ,表示查看下一次迭代的值(当然也可以用 iterator.__next__() ,查看下一次迭代的值)
```
l = ["hello","world",1,2]
aa = l.__iter__() # 生成列表迭代器
print(type(aa))
print(aa.__next__())
print(aa.__next__())
print(aa.__next__())
print(next(aa))
print(next(aa)) #没有可迭代的值了也就是迭代完了，会报错：StopIteration
```

```
#1、为何要有迭代器？
对于序列类型：字符串、列表、元组，我们可以使用索引的方式迭代取出其包含的元素。但对于字典、集合、文件等类型是没有索引的，若还想取出其内部包含的元素，则必须找出一种不依赖于索引的迭代方式，这就是迭代器

#2、什么是可迭代对象？
可迭代对象指的是内置有__iter__方法的对象，即obj.__iter__，如下
'hello'.__iter__
(1,2,3).__iter__
[1,2,3].__iter__
{'a':1}.__iter__
{'a','b'}.__iter__
open('a.txt').__iter__

#3、什么是迭代器对象？
可迭代对象执行obj.__iter__()得到的结果就是迭代器对象
而迭代器对象指的是即内置有__iter__又内置有__next__方法的对象

文件类型是迭代器对象
open('a.txt').__iter__()
open('a.txt').__next__()

#4、注意：
迭代器对象一定是可迭代对象，而可迭代对象不一定是迭代器对象
```
## 迭代器协议
```
1.迭代器(iterator)协议是指:对象必须提供一共next方法，执行该方法妖魔返回迭代中的下一项，要么就引起一个Stopiteration异常，已终止迭代。
2.可迭代对象:实现了迭代器协议的对象，(该对象内部定义了一个__iter__()的方法  例：str.__iter__())就是可迭代对象
3.协议是一种约定，可迭代对象实现了迭代器协议，python的内部工具(如。for循环，sum，min，max函数等)使用迭代器协议访问对象
```
## for循环
for循环本质：循环所有对象，全部是使用迭代器协议。  
(字符串，列表，元祖，字典，集合，文件对象)，这些都不是可迭代对象，只不过在for循环，调用了他们内部的__iter__方法，把他们变成了可迭代对象。然后for循环调用可迭代对象的__next__方法去去找，然后for循环会自动捕捉StopIteration异常，来终止循环。
```
#基于for循环，我们可以完全不再依赖索引去取值了
dic={'a':1,'b':2,'c':3}
for k in dic:
    print(dic[k])

#for循环的工作原理
#1：执行in后对象的dic.__iter__()方法，得到一个迭代器对象iter_dic
#2: 执行next(iter_dic),将得到的值赋值给k,然后执行循环体代码
#3: 重复过程2，直到捕捉到异常StopIteration,结束循环
```
## 迭代器的优缺点
- 优点：
  - 提供一种统一的、不依赖于索引的迭代方式
  - 惰性计算，节省内存
- 缺点：
  - 无法获取长度（只有在next完毕才知道到底有几个值）
  - 一次性的，只能往后走，不能往前退

# 三元运算
三元运算可以简化条件语句的缩写，可以使代码看起来更加简洁，三元可以简单的理解为有三个变量，它的形式是这样的 name= k1 if 条件 else k2 ，如果条件成立，则 name=k1，否则name=k2，下面从代码里面来加深一下理解，从下面的代码明显可以看出三目运算符可以使代码更加简洁。
```
a = 2
b = 4
if a < b:
    k = a
else:
    k = b
print(k)

k=a if a<b else b # 三元运算更简洁
print(k)
``` 
# 列表解析
```
# 一般方法
egg_list = []
for i in range(10):
    egg_list.append("鸡蛋%s" %i)
print(egg_list)

# 列表解析
egg_list2 = ["鸡蛋%s" %i for i in range(10)]
egg_list3 = ["鸡蛋%s" %i for i in range(10) if i > 5]  # 三元表达式
print(egg_list2)
print(egg_list3)
```
# 生成器
> 在python中，一边循环，一边计算的机制，称为生成器(generator),可以理解为一种数据类型，这种类型自动实现了迭代器协议。(其他的数据类型需要调用自已的内置__iter__方法或则iter()的内置函数)，所以生成器就是一个可迭代对象。  
生成器分类以及在python中的表现形式。(python有两种不同的方式提供生成器)
- 生成器函数：使用yield语句而不是return的函数。在调用生成器运行过程中，每次遇到yield是函数会暂停并保存当前所有的运行信息，返回yield值。并在下一次执行next方法时，从当前位置继续运行。
```
#只要函数内部包含有yield关键字，那么函数名()的到的结果就是生成器
def func():
    print('====>first')
    yield 1
    print('====>second')
    yield 2
    print('====>third')
    yield 3
    print('====>end')

g=func()
print(g) #<generator object func at 0x0000000002184360>
```
- 生成器表达式：类似于列表推导，但是生成器返回按需产生结果的一种对象，而不是一次构建一个结果列表
```
# 与列表解析类似，只不过列表解析的结果是一个列表，而生成器表达式是生成器
egg_list4 = ("鸡蛋%s" %i for i in range(10) if i > 5)  # 生成器表达式
print(egg_list4)
print(egg_list4.__next__())
print(egg_list4.__next__())
# 输出为：
# <generator object <genexpr> at 0x000001E3850CA258>
# 鸡蛋6
# 鸡蛋7
```
- 生成器的优点：  
函数是顺序执行，遇到return语句或则最后一行函数函数语句就返回。
而变成generator的函数，在每次调用next()的时候执行，遇到yield语句返回，再次执行从上次返回的yield语句处继续执行
python使用生成器对延迟操作提供了支持。所谓延迟操作，是指需要的时候才产生结果，而不是立即生成结果。
```
# 先生产再消费的模型
# 效率低
import time
def producer():
    ret = []
    for i in range(20):
        time.sleep(0.1)
        ret.append("包子%s" %i)
    return ret
def consumer(res):
    for index,baozi in enumerate(res):
        time.sleep(0.1)
        print("第%s个人，吃了%s" %(index,baozi))

res=producer()
consumer(res)

# 边生产边消费
def consumer(name):
    print("我是 %s，我开始吃包子了" %name)
    while True:
        baozi = yield
        time.sleep(1)
        print("%s 很开心的把%s 吃掉了" %(name,baozi))

def producer():
    c1 = consumer("lisen")
    c2 = consumer("zhangsan")
    c1.__next__()
    c2.__next__()
    for i in range(10):
        time.sleep(1)
        c1.send("包子 %s" %i)
        c2.send("包子 %s" %i)
producer()
#yield 3相当于return 控制的是函数的返回值
#baozi = yield的另外一个特性，接受send传过来的值,赋值给baozi
```
# 作业
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 1、列举布尔值为False的值
# None,0 False '' [] {} ()

# 2、写函数：
# 根据范围获取其中 3 和 7 整除的所有数的和，并返回调用者：符合条件的数字个数以及符合条件的数字的总和
# def fun(a,b):
#    geshu = 0
#    geshu_sum = 0
#    for i in range(a,b+1):
#        if i % 3 == 0 and i % 7 ==0:
#            print(i)
#            geshu += 1
#            geshu_sum += i
#    print("个数为%s,总和为%s" %(geshu,geshu_sum))
#
# fun(1,200)

# 3、函数的默认返回值是什么？
# 　　None

# 4、简述break/continue/return的区别
# 　　break        结束当前层循环
# 　　continue    结束本次循环
# 　　return       作为函数运行结束标志

# 5、函数传递参数时，是引用还是复制值？并证明提示：可以用 id 进行判断
# 　　引用
# n = 5
# def fib(n):
#     print("函数中",id(n))
# print("函数外",id(n))
# fib(5)

# 6、简述三元运算书写格式以及应用场景
# a = 2
# b = 4
# c =a if a > b else b
# print(c)

# 7、简述
# lambda 表达式书写格式以及应用场景
# test =lambda x,y:x+y
# print(test(2,3))
# 简单的函数写成匿名函数，省内存 减少代码

# 8、使用 set 集合获取两个列表l1=[11,22,33],l2=[22,33,44]中相同的元素集合
# l1=[11,22,33]
# s1 = set (l1)
# l2=[22,33,44]
# s2 = set (l2)
# s3 = s1.intersection(s2)
# print(s3)

# 9、定义函数统计一个字符串中大写字母、小写字母、数字的个数，并以字典为结果返回给调用者
# def fun(string):
#     count1,count2,count3 = 0,0,0
#     for i in string:
#         i = str(i)
#         if i.isupper():
#             count1 += 1
#         elif i.islower():
#             count2 += 1
#         elif i.isdecimal():
#             count3 += 1
#     return ({"大写字母":count1,"小写字母":count2,"数字":count3})
# res = fun("abcEFGG12344")
# print(res)

# 10、简述函数的位置参数、关键字参数、默认参数、可变长参数的特点以及注意事项
# 　　位置参数：按照对应形参位置一一对应传入参数，普通参数
# 　　关键字参数：传入实参是指定形参的值，允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict
# 　　默认参数：形参直接指定默认值的参数
# 　　可变长参数：*args **kwargs,传入的参数个数是可变的，可以是1个、2个到任意个，还可以是0个

# 11、检查代码，如有错误请改正(禁止运行代码)：
# def func(x,y,z):
#     print(x,y,z)
# func(1,2,3)
# def func(x,z,y=5): # 有赋值的参数，传入的参数必须在右边，否则报错
#     print(x,y,z)
# func(1,3)
# def func(x,y,*z): #列表传入
#     print(x,y,z)
# func(1,2,3,4,5,6)
# 输出为：1 2 (3, 4, 5, 6)
# def func(x,*z,**y):
#     print(x,y,z)
# func(1,3,4,5,y=5) # 有赋值的参数，传入的参数必须在右边，否则报错
# 输出为：1 {'y': 5} (3, 4, 5)
# def func(*y,**z):
#     print(y,z)
# func(1,2,3,4,5)
# 输出为：(1, 2, 3, 4, 5) {}
# def func(*y,**z):
#     print(y,z)
# func([1,2,3,4,5])
# 输出为：([1, 2, 3, 4, 5],) {}
# def func(*y,**z):
#     print(y,z)
# func(*[1,2,3,4,5])
# 输出 ((1,2,3,4,5) {})
#
# def func(*y,**z):
#     print(y,z)
# func(*[1,2,3,4,5],name="alex",age=19)
# 输出((1,2,3,4,5) {'name':'alex','age':19})
#
# def func(x,*z,**y):
#     print(x,y,z)
# func(1,2,3)
# 输出 (1 {} (2,3))
#
# def func(*y,**z):
#     print(y,z)
# func(*[1,2,3,4,5],**{"name":"alex","age":19})
# 输出 ((1,2,3,4,5) {'name':'alex','age':19})
# def func1(x=1,*y,**z):
#     print(x,y,z)
#     return y
#     print(x)
# def func2(arg):
#     ret=func1(name=arg)
#     print(ret)
# res = func2("funck")
# print(res)
# 输出为：
# 1 () {'name': 'funck'}
# ()
# None

# 12、简述 Python3 中的 range 函数和 Python2.7 中的 range 函数有什么区别?
# 返回值的不同： python3中range生产的是一个可迭代对象，不会马上生成全部值，用的时候才生成 python2中range会直接生成一个全部值的列表返回

# 13、内置函数 all 和 any 的区别
# >> > any(['a', 'b', '', 'd'])  # 列表list，存在一个为空的元素
# True
# >>> all(['a', 'b', '', 'd'])  #列表list，存在一个为空的元素
# False

# 14、利用内置函数 zip()，实现功能
# l1=["alex",22,33,44,55]
# l2=["is",22,33,44,55]
# l3=["good",22,33,44,55]
# l4=["guy",22,33,44,55]
# #  请获取字符串s="alex_is_good_guy"
# res=list(zip(l1,l2,l3,l4))
# # print(res)
# s='_'.join(res[0])
# print(s)

# 15、判断输出结果是否相同？并可得到什么结论？
# def f1(arg):
#     print(id(arg))
# n = 111
# print(id(n))
# f1(n)
# 输出的两个值是否相同：相同  对于传参数的时候传值还是传引用？python不允许程序员选择采用传值还是传引用。Python参数传递采用的肯定是“传对象引用”的方式。这种方式  相当于传值和传引用的一种综合。如果函数收到的是一个可变对象（比如字典或者列表）的引用，就能修改对象的原始值－－相当于通过“传引用”来传递对象。如果函数收到的是一  个不可变对象（比如数字、字符或者元组）的引用，就不能直接修改原始对象－－相当于通过“传值'来传递对象。

# 16、看代码写结果
# name = "root"
# def func():
#     name = "seven"
#     def aa():
#         name = "lisen"
#         def bb():
#             global name
#             name = "zhangsan"
#         print(name)
#     print(name)
#
# ret = func()
# print(ret)
# print(name)
# 输出为：
# seven
# None
# root
# 内置函数没调用就只是个变量，没起任何作用

# name = "root"
# def func():
#     name = "seven"
#     def outer():
#         name = "eric"
#         def inner():
#             global name
#             name = "蒙逼了吧..."
#         print(name)
#         inner()
#     o = outer()
#     print(o)
#     print(name)
#
# ret = func()
# print(ret)
# print(name)
# 输出为：
# eric
# None
# seven
# None
# 蒙逼了吧...

# name = "root"
# def func():
#     name = "seven"
#     def outer():
#         name = "eric"
#         def inner():
#             nonlocal name
#             name = "蒙逼了吧..."
#         inner()
#         print(name)
#     o = outer()
#     print(o)
#     print(name)
#
# ret = func()
# print(ret)
# print(name)
# 输出为：
# 蒙逼了吧...
# None
# seven
# None
# root

# name = "苍老师"
# def outer(func):
#     name = 'alex'
#     func() # 实际执行的就是show函数
# def show():
#     print(name)
# outer(show)
# 输出为：苍老师

# name = "苍老师"
# def outer():
#     name = "波多"
#     def inner():
#         print(name)
#     return inner()    # 调用了inner
# ret = outer()
# print(ret)
# 输出为：
# 波多
# None

# name = "苍老师"
# def outer(func):
#     def inner():
#         name = "李杰"
#         func()
#     return inner
# def show():
#     print(name)
# outer(show)()
# 输出为：苍老师

# def f5(arg):
#     arg.append('偷到 500 万')
#
# def f4(arg):
#     arg.append('开第四个门')
#     f5(arg)
#     arg.append('关第四个门')
#
# def f3(arg):
#     arg.append('开第三个门')
#     f4(arg)
#     arg.append('关第三个门')
#
# def f2(arg):
#     arg.append('开第二个门')
#     f3(arg)
#     arg.append('关第二个门')
#
# def f1(arg):
#     arg.append('开一个门')
#     f2(arg)
#     arg.append('关一个门')
#
# user_list = []
# result = f1(user_list)
# print(user_list)
# print(result)
# 输出为：
# ['开一个门', '开第二个门', '开第三个门', '开第四个门', '偷到 500 万', '关第四个门', '关第三个门', '关第二个门', '关一个门']
# None

# def f5(arg):
#     arg = arg + 5
#
# def f4(arg):
#     arg = arg + 4
#     f5(arg)
#     arg = arg + 4
#
# def f3(arg):
#     arg = arg + 3
#     f4(arg)
#     arg = arg + 3
#
# def f2(arg):
#     arg = arg + 2
#     f3(arg)
#     arg = arg + 2
#
# def f1(arg):
#     arg = arg + 1
#     f2(arg)
#     arg = arg + 1
#     return arg
#
# num = 1
# result = f1(num)
# print(num)
# print(result)
# 输出为：
# 1
# 3
# 利用递归实现上题的功能
# def func(x, y=0):
#     y += 1
#     if y == 5:
#         return x + y
#     x += y
#     func(x, y)
#     x += y
#     return x
# num = 1
# res = func(num)
# print(num)
# print(res)

# 17、利用递归实现1*2*3*4*5*6*7
# 法1：
# def func(n):
#     if n == 1:
#         return 1
#     return n*func(n-1) # 7*func(6) 6*func(5) 5*func(4) 4*func(3) 3*func(2) 2*func(1)--return 1
# res = func(7)
# print(res)
# 法2：
# from functools import reduce
# print(reduce(lambda x,y:x*y,[x for x in range(1,8)]))

# 18、写程序
# a. 利用filter、自定义函数获取 l1 中元素大于 33 的所有元素l1=[11,22,33,44,55]
# 自定义函数：
# l1=[11,22,33,44,55]
# res = []
# for i in l1:
#     if i > 33:
#         res.append(i)
# print(res)
# # filter:
# print(list(filter(lambda x:x>33,l1)))

# b.利用map、自定义函数将所有是奇数的元素加100
# l1 = [11, 22, 33, 44, 55]
# def fib(n):
#     if n % 2 != 0:
#         n += 100
#     return n
# res = list(map(fib, l1))
# print(res)

# 19、写函数：
# 如有以下两个列表
# l1=[...]
# l2=[]
# 第一个列表中的数字无序不重复排列，第二个列表为空列表
# 需求：
# 取出第一个列表的最小值放到第二个列表的首个位置，
# 取出第一个列表的最小值（仅大于上一次的最小值）放到第二个列表的首个位置，
# 取出第一个列表的最小值（仅大于上一次的最小值）放到第二个列表的首个位置，
# ...
# 依此类推，从而获取一个有序的列表 l2，并将其返回给函数调用者。
# l1 = [50,70,30,80,75]
# l2 = []
# def deffunc(n1,n2):
#     while True:
#         if l1:
#             res=min(n1)
#             n1.remove(res)
#             n2.insert(0,res)
#         else:break
#     return n2
# res = deffunc(l1,l2)
# print(res)

# 20、猴子第一天摘下若干个桃子，当即吃了一半，还不过瘾就多吃了一个。第二天早上又将剩下的桃子吃了一半，还是不过瘾又多吃了一个。以后每天都吃前一天剩下的一半再加一个。到第 10 天刚好剩一个。问猴子第一天摘了多少个桃子？
# 法1：
# p = 1
# print('第10天吃之前就剩1个桃子')
# for i in range(9, 0, -1):
#     p = (p + 1) * 2
#     print('第%s天吃之前还有%s个桃子' % (i, p))
# print('第1天共摘了%s个桃子' % p)
# 输出：
# 第10天吃之前就剩1个桃子
# 第9天吃之前还有4个桃子
# 第8天吃之前还有10个桃子
# 第7天吃之前还有22个桃子
# 第6天吃之前还有46个桃子
# 第5天吃之前还有94个桃子
# 第4天吃之前还有190个桃子
# 第3天吃之前还有382个桃子
# 第2天吃之前还有766个桃子
# 第1天吃之前还有1534个桃子
# 第1天共摘了1534个桃子
#
# 法2：
# s = 1
# func = lambda x: (x + 1) * 2
# for i in range(9):
#     s = func(s)
# print(s)
# 输出：
# 1534
```

 

