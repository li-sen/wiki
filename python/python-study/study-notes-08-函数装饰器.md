# 装饰器
## 装饰器定义
器即函数  
装饰即修饰，意指为其他函数添加新功能  
装饰器定义：本质就是函数，功能是为其他函数添加新功能  
装饰器=高阶函数+函数嵌套+闭包
## 装饰器的基本要求
1. 不修改被装饰函数的源代码（开放封闭原则）
2. 为被装饰函数添加新功能后，不修改被修饰函数的调用方式
## 高阶函数
高阶函数定义:
1. 函数接收的参数是一个函数名
2. 函数的返回值是一个函数名
3. 满足上述条件任意一个,都可称之为高阶函数
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

import time
def foo():
    time.sleep(3)
    print("hello world!")
# def timer(func):
#     start_time = time.time()
#     func()
#     stop_time = time.time()
#     print("函数的运行时间为 %s" %(stop_time - start_time))
# timer(foo)
# 这样是能完成为foo函数加时间统计功能了，但是得timer(foo)运行，改变了函数原本的调用方式
def timer(func):
    start_time = time.time()
    func()
    stop_time = time.time()
    print("函数的运行时间为 %s" %(stop_time - start_time))
    return func
foo = timer(foo) # 高阶函数的两个特点结合，接收的参数是函数，返回的结果也是函数
foo() # 既不用更改原本函数的源代码，也不改变原本函数的调用方式，但是这种方式使函数运行了2次也不合理（timer函数中已经运行了一次foo）
# 运行两次说明下：foo = timer(foo)运行了一次 foo()又运行了一次
```
## 函数的嵌套
python语言中的嵌套：定义一个函数的时候，函数体还能定义另一个函数；
或者说在一个函数调用另一个函数，叫嵌套
```
def test(name):
    print('第一层是%s' %name)
    def test1():
        print('第二层是%s' %name)
        def test2():
            print('第三层是%s' %name)
        test2()
    test1()
test('lisen')
```
## 闭包
- 函数嵌套中的函数作用域的行为叫做闭包
- 一个闭包就是一个函数，只不过函数内部带上了一个额外的变量。
- 闭包关键特点就是它会记住自已被定义时的环境
```
# 闭包:在一个作用域里放入定义变量,相当于打了一个包
def test(name):
    print('第一层是%s' %name)
    def test1():
        name = "zhangsan"
        print('第二层是%s' %name)
        def test2():
            name = "wangwu"
            print('第三层是%s' %name)
        test2()
    test1()
test('lisen')
# 输出为;
# 第一层是lisen
# 第二层是zhangsan
# 第三层是wangwu
```
## 逐步实现装饰器
### 基本框架
```
import time
# def foo():
#     time.sleep(3)
#     print("hello world!")

def timer(func):
    def wrapper():
        start_time = time.time()
        func()
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
    return wrapper
# foo = timer(foo) # 返回的是wrapper的地址
# foo() # 执行的是wrapper函数

# 语法糖@
@timer # 相当于foo = timer(foo)
def foo():
    time.sleep(3)
    print("hello world!")
foo()
```
### 被修饰的函数有返回值
对于被修饰的函数没有返回值的情况下，用上面的装饰器是没有问题的，因为他们的返回值都是None,但被修饰的函数有返回值时，上面的装饰器是达不到要求的，因为被修饰后函数的返回值还是None
```
import time

def timer(func):
    def wrapper():
        start_time = time.time()
        func()
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
    return wrapper

@timer
def foo():
    time.sleep(3)
    print("hello world!")
    return "foo"
res = foo()
print(res) # 还是为None，被修饰前后返回值不一致，不符合要求

######## 修改代码后 ########
import time

def timer(func):
    def wrapper():
        start_time = time.time()
        aa = func() # 这里实际运行的是foo,也就是被修饰的函数，将它赋值给aa,然后return aa即可
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
        return aa
    return wrapper

@timer
def foo():
    time.sleep(3)
    print("hello world!")
    return "foo"
res = foo()
print(res) # 修饰前后返回值一致了
```
### 被修饰的函数有参数
```
import time

def timer(func):
    def wrapper():
        start_time = time.time()
        aa = func()
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
        return aa
    return wrapper

@timer
def foo(name,age):
    time.sleep(3)
    print("hello world! 我是%s,年龄为%d" %(name,age))
    return "foo"
foo("lisen",20) # 报错
# TypeError: wrapper() takes 0 positional arguments but 2 were given
# 运行foo("lisen",20)实际上运行的是wrapper(),因为timer函数return的是wrapper，可wrapper()没有参数,所以报错

######## 修改代码后 ########
import time

def timer(func):
    def wrapper(name,age): # 添加参数
        start_time = time.time()
        aa = func(name,age) # 添加参数
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
        return aa
    return wrapper

@timer
def foo(name,age):
    time.sleep(3)
    print("hello world! 我是%s,年龄为%d" %(name,age))
    return "foo"
foo("lisen",20) 
# 添加参数后，装饰器可以正常修饰了，但是这样装饰器只能修饰foo(name,age)函数，用到其他参数不一致的函数就装饰器就又不能用了，局限性太大了

######## 继续完善 ########
import time

def timer(func):
    def wrapper(*args,**kwargs): # 更改为可变长参数
        start_time = time.time()
        aa = func(*args,**kwargs) # 更改为可变长参数
        stop_time = time.time()
        print("函数的运行时间为 %s" %(stop_time - start_time))
        return aa
    return wrapper

@timer
def foo(name,age):
    time.sleep(3)
    print("hello world! 我是%s,年龄为%d" %(name,age))
    return "foo"
foo("lisen",20)

@timer
def aa(name,age,gender):
    time.sleep(3)
    print("hello world! 我是%s,年龄为%d,性别为%s" %(name,age,gender))
    return "foo"
aa("lisen",20,"male")
# 这样两个不同参数的函数都能用timer装饰器了
```
## 装饰器示例：
- 增加认证功能
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

def auth_func(func):
    def wrapper(*args,**kwargs):
        username = input("请输入用户：").strip()
        print(username)
        pw= input("请输入密码：").strip()
        print(pw)
        if username == "lisen" and pw == "123":
            aa = func(*args,**kwargs)
            return aa
        else:
            print("用户密码错误")
    return wrapper

@auth_func
def shopping_car(name):
    print("添加%s到购物车" %name)

@auth_func
def usercenter():
    print("用户中心")

shopping_car("红酒")
usercenter()
```
- 模拟session
> 跟网站一样，一次认证后，后面就不用重复认证了
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 用户列表
user_list=[
    {'name':'zhangsan','passwd':'123'},
    {'name':'wangwu','passwd':'123'},
    {'name':'lisen','passwd':'123'},
]

current_user={'username':None,'login':False} #判断用户是否登录

def auth_func(func):
    def wrapper(*args,**kwargs):
        if current_user['username'] and current_user['login']:
            aa = func(*args,**kwargs)
            return aa
        username = input("请输入用户：").strip()
        pw= input("请输入密码：").strip()
        for i in user_list:
            if username == i["name"] and pw == i["passwd"]:
                current_user["username"] = username # 登陆成功更改状态
                current_user["login"] = True
                aa = func(*args,**kwargs)
                return aa
        else:
            print("用户密码错误")
    return wrapper

@auth_func
def shopping_car(name):
    print("添加%s到购物车" %name)

@auth_func
def usercenter():
    print("用户中心")

shopping_car("红酒") # 一次登陆就行了，usercenter不用认证了
usercenter()
```
## 有参装饰器
还是拿上面的列子来阐述，认证的方式有很多种，目前我们是很简陋的将用户信息放在列表中，实际生产中一般是数据库，或者ldap等，这样上面的列子就不能满足需求了，需要让装饰器有参数，来选择认证方式了。
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 用户列表
user_list=[
    {'name':'zhangsan','passwd':'123'},
    {'name':'wangwu','passwd':'123'},
    {'name':'lisen','passwd':'123'},
]

current_user={'username':None,'login':False} #判断用户是否登录
def auth(auth_type="file"):
    def auth_func(func):
        def wrapper(*args,**kwargs):
            if auth_type == "file":
                if current_user['username'] and current_user['login']:
                    aa = func(*args,**kwargs)
                    return aa
                username = input("请输入用户：").strip()
                pw= input("请输入密码：").strip()
                for i in user_list:
                    if username == i["name"] and pw == i["passwd"]:
                        current_user["username"] = username # 登陆成功更改状态
                        current_user["login"] = True
                        aa = func(*args,**kwargs)
                        return aa
                else:
                    print("用户密码错误")
            elif auth_type == "database":
                print("数据库认证流程，省略")
                aa = func(*args,**kwargs)
                return aa
            elif auth_type == "ldap":
                print("ldap认证流程，省略")
                aa = func(*args,**kwargs)
                return aa

        return wrapper
    return auth_func

# 增加了一层函数嵌套，增加了一个参数也就是一个变量
@auth(auth_type="database")
def shopping_car(name):
    print("添加%s到购物车" %name)

@auth(auth_type="ldap")
def usercenter():
    print("用户中心")

@auth(auth_type="file")
def order():
    print("查看订单")

shopping_car("红酒") # 一次登陆就行了，usercenter不用认证了
usercenter()
order()
```
# 序列解包
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# a,b,c = (1,2,3)
# print(a)
# print(b)
# print(c)
# 必须得一一对应

l = [1,2,3,4,5]
# 只想取列表中的第一个跟最后一个，可以用索引方式来完成，不过用序列解包更快
x,*y,z =l
print(x) # 第一个
print(z) # 最后一个
print(y) # 中间不想要的
# 输出为：
# 1
# 5
# [2, 3, 4]
```
# 变量交换
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 想让两个变量值进行交换
a = 1
b = 2
# 一般方法,用中间变量过度下
x = a
a =2
a=b
b=x
print(a,b)

# 变量互换
c =3
d=4
c,d=d,c
print(c,d)
```
# 作业

```
# 1. 多层循环，一个状态标识符退出所有循环
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

tag=True
while tag:
    print('leve1')
    choice=input("level1>>: ").strip()
    if choice == 'quit':break
    if choice == 'quit_all': tag = False
    while tag:
        print('level2')
        choice = input("level2>>: ").strip()
        if choice == 'quit': break
        if choice == 'quit_all': tag = False
        while tag:
            print('level3')
            choice = input("level3>: ").strip()
            if choice == 'quit': break
            if choice == 'quit_all': tag = False
            
# 2.haproxy配置文件的增删改查
# 2.1初步实现查询功能
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

def fetch(data):
    backend_data="backend %s" %data
    with open("haproxy.conf","r") as read_f:
        tag=False
        ret=[]
        for read_line in read_f:
            if read_line.strip() == backend_data:
                tag=True
                continue
            if tag and read_line.startswith("backend"): # 结束打印
                break
            if tag:
                print("%s" %read_line,end="")
                ret.append(read_line.strip())
    return ret


def add():
    pass

def change():
    pass

def delete():
    pass

if __name__ == "__main__":
    msg='''
    1:查询
    2:添加
    3:修改
    4:删除
    5:退出
    '''
    msg_dic={
        '1':fetch,
        '2':add,
        '3':change,
        '4':delete,
    }

    while True:
        print(msg)
        choice=input("请输入你的选项：").strip()
        if not choice:continue
        if choice == "5":break
        data=input("请输入数据：").strip()
        res=msg_dic[choice](data)
        print("结果是：",res)

# 2.2 终极版本
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

#_*_coding:utf-8_*_
import os
def file_handle(filename,backend_data,record_list=None,type='fetch'): #type:fetch append change
    new_file=filename+'_new'
    bak_file=filename+'_bak'
    if type == 'fetch':
        r_list = []
        with open(filename, 'r') as f:
            tag = False
            for line in f:
                if line.strip() == backend_data:
                    tag = True
                    continue
                if tag and line.startswith('backend'):
                    break
                if tag and line:
                    r_list.append(line.strip())
            for line in r_list:
                print(line)
            return r_list
    elif type == 'append':
        with open(filename, 'r') as read_file, \
                open(new_file, 'w') as write_file:
            for r_line in read_file:
                write_file.write(r_line)

            for new_line in record_list:
                if new_line.startswith('backend'):
                    write_file.write(new_line + '\n')
                else:
                    write_file.write("%s%s\n" % (' ' * 8, new_line))
        os.rename(filename, bak_file)
        os.rename(new_file, filename)
        os.remove(bak_file)
    elif type == 'change':
        with open(filename, 'r') as read_file, \
                open(new_file, 'w') as write_file:
            tag=False
            has_write=False
            for r_line in read_file:
                if r_line.strip() == backend_data:
                    tag=True
                    continue
                if tag and r_line.startswith('backend'):
                    tag=False
                if not tag:
                    write_file.write(r_line)
                else:
                    if not has_write:
                        for new_line in record_list:
                            if new_line.startswith('backend'):
                                write_file.write(new_line+'\n')
                            else:
                                write_file.write('%s%s\n' %(' '*8,new_line))
                        has_write=True
        os.rename(filename, bak_file)
        os.rename(new_file, filename)
        os.remove(bak_file)

def fetch(data):
    backend_data="backend %s" %data
    return file_handle('haproxy.conf',backend_data,type='fetch')
def add(data):
    backend=data['backend']
    record_list=fetch(backend)
    current_record="server %s %s weight %s maxconn %s" %(data['record']['server'],\
                                                         data['record']['server'],\
                                                         data['record']['weight'],\
                                                         data['record']['maxconn'])
    backend_data="backend %s" %backend

    if not record_list:
        record_list.append(backend_data)
        record_list.append(current_record)
        file_handle('haproxy.conf',backend_data,record_list,type='append')
    else:
        record_list.insert(0,backend_data)
        if current_record not in record_list:
            record_list.append(current_record)
        file_handle('haproxy.conf',backend_data,record_list,type='change')
def remove(data):
    backend=data['backend']
    record_list=fetch(backend)
    current_record="server %s %s weight %s maxconn %s" %(data['record']['server'],\
                                                         data['record']['server'],\
                                                         data['record']['weight'],\
                                                         data['record']['maxconn'])
    backend_data = "backend %s" % backend
    if not record_list or current_record not in record_list:
        print('\033[33;1m无此条记录\033[0m')
        return
    else:
        #处理record_list
        record_list.insert(0,backend_data)
        record_list.remove(current_record)
        file_handle('haproxy.conf',backend_data,record_list,type='change')
def change(data):
    backend=data[0]['backend']
    record_list=fetch(backend)

    old_record="server %s %s weight %s maxconn %s" %(data[0]['record']['server'],\
                                                         data[0]['record']['server'],\
                                                         data[0]['record']['weight'],\
                                                         data[0]['record']['maxconn'])

    new_record = "server %s %s weight %s maxconn %s" % (data[1]['record']['server'], \
                                                        data[1]['record']['server'], \
                                                        data[1]['record']['weight'], \
                                                        data[1]['record']['maxconn'])
    backend_data="backend %s" %backend

    if not record_list or old_record not in record_list:
        print('\033[33;1m无此内容\033[0m')
        return
    else:
        record_list.insert(0,backend_data)
        index=record_list.index(old_record)
        record_list[index]=new_record
        file_handle('haproxy.conf',backend_data,record_list,type='change')
def qita(data):
    pass


if __name__ == '__main__':
    msg='''
    1：查询
    2：添加
    3：删除
    4：修改
    5：退出
    6:其他操作
    '''
    menu_dic={
        '1':fetch,
        '2':add,
        '3':remove,
        '4':change,
        '5':exit,
        '6':qita,
    }
    while True:
        print(msg)
        choice=input("操作>>: ").strip()
        if len(choice) == 0 or choice not in menu_dic:continue
        if choice == '5':break

        data=input("数据>>: ").strip()

        #menu_dic[choice](data)==fetch(data)
        if choice != '1':
            data=eval(data)
        menu_dic[choice](data) #add(data)

# [{'backend':'www.oldboy1.org','record':{'server':'2.2.2.4','weight':20,'maxconn':3000}},{'backend':'www.oldboy1.org','record':{'server':'2.2.2.5','weight':30,'maxconn':4000}}]
```