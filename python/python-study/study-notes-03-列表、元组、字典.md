# 列表
- 列表中的元素可以是任何数据类型：字符串、数字、列表等  
示例：  
li =  [1, 12, 9, "age", ["石振文", ["19", 10], "庞麦郎"], "alex", True]
- 列表是有序的，就算列表li被打印了10次，其结果是一致的
```
li = [11,22,33,44]
print(li)
print(li)
print(li)
print(li)
```
- 列表以链表方式存储，列表中的元素可以直接被修改
## 基本方法
 ```
# 1. 列表中元素修改：
# 字符串创建后不可被修改，更改了需要使用变量开辟新的内存空间进行保存
# v = "alex"
# v = v.replace('l','el')
# print(v)

# li = [11, 22, 33, 22, 44]
# li[1] = 120
# print(li)
#
# li[1] = [66, 666, 6666]
# print(li)

# li[1:3] = [120,90]
# print(li)
# 输出为：[11, 120, 90, 22, 44]

# 2. 列表中元素删除
# li = [11, 22, 33, 22, 44]
# del li[1]
# del li[1:3]
# print(li)
#
# 3. 索引取值
# li = [11, 22, 33, 22, 44]
# print(li[3])
#
# 4. 切片,切片的结果也是列表
# li = [11, 22, 33, 22, 44]
# print(li[2:4])
# print(li[::2]) #从头到尾，索引位置加2取值（奇数位）
#
# 5. 能被for循环
# li = [11, 22, 33, 22, 44]
# for i in li:
#     print(i)

# 6. in 判断
# li = [1, 12, 9, "age", ["李森", ["19", 10], "庞麦郎"], "alex", True]
# v1 = "李森" in li #只能对列表中的一级元素做判断
# v2 = "alex" in li
# print(v1,v2)
# 输出为：False True

# 7. 多级元素取值
# li = [1, 12, 9, "age", ["李森", ["19", 10], "庞麦郎"], "alex", True]
# v = li[4][1][1] # 获取10
# print(v)

# 8. 转换
# 字符串转换成列表
# s = "abcdefg"
# li = list(s) #内部使用了for循环，依次将字符append进列表
# print(li)

# 列表转换成字符串
# li = [11, 22, 33, 22, "alex"] #有数字跟字符串只能自己写for循环一个个处理
# s = ""
# for i in li:
#     s =s + str(i)
# print(s)
# 输出为：11223322alex

# li = ["aa", "bb","cc"] #只有字符串可以使用join方法
# s = "".join(li)
# print(s)

# 9. 计算列表长度并输出
# li = ['alex', 'eric', 'rain']
# v = len(li)
# print(v)
# 输出为：3
 ```
## 一般方法
   ```    
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 1. 在原有值最后追加
# li = [11, 22, 33, 22, 44]
# li.append(5)
# li.append("alex")
# li.append([123, 123])
# print(li)

# 2. 清空列表
# li = [11, 22, 33, 22, 44]
# li.clear()
# print(li)

# 3. 拷贝（浅拷贝）
# li = [11, 22, 33, 22, 44]
# v = li.copy()
# print(v)

# 4.统计元素出现的次数
# li = [11, 22, 33, 22, 44]
# v = li.count(22)
# print(v)

# 5. 扩展原列表，参数为可迭代对象,依次将参数中的元素加入到列表中
# li = [11, 22, 33, 22, 44]
# li.extend([123,"很不错"])
# print(li)
# 输出为：[11, 22, 33, 22, 44, 123, '很不错']
# li.append([123,"很不错"])
# print(li)
# 输出为：[11, 22, 33, 22, 44, [123, '很不错']]

# 6. 获取值在当前列表的索引位置（左边优先）,匹配到一次就停止
# li = [11, 22, 33, 22, 44]
# v = li.index(22)
# print(v)

# 7. 在指定索引位置插入元素
# li = [11, 22, 33, 22, 44]
# li.insert(0,100)
# print(li)

# 8. 删除列表中指定元素(提供索引位置，不指定默认删除最后一个)，并获取被删除元素
# li = [11, 22, 33, 22, 44]
# li.pop(2)
# v = li.pop(2)
# print(v)
# print(li)
# 输出为：
# 33
# [11, 22, 22, 44]

# 9. 删除列表中的元素（提供具体元素，不指定默认删除第一个）
# li = [11, 22, 33, 22, 44]
# li.remove(22)
# print(li)

# 10. 对当前列表中的元素进行排序
# li = [11, 22, 33, 22, 44]
# li.sort() #升序
# li.sort(reverse=True) #降序
# print(li)

# 11. 对当前列表中的元素位置进行反转,并不排序
# li = [11, 22, 33, 22, 44]
# li.reverse()
# print(li)
   ```    
# 元组
元组，可以看做是列表的二次加工，但元组中的一级元素不可被修改即不能新增也不能删除
示例：  
tu = (111, "alex", (11,22), [(33,44)], True, 33, 44,)
> 一般写元组的时候，推荐在最后加入逗号（,）
元组也是可迭达对象，可以被for循环
```
for i i tu:
    print(i)
```

## 基本方法 
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# 获取指点元素在元组中出现的次数
# tu = (11, 22, 33, 22, 44)
# v = tu.count(22)

# 获取值在当前列表的索引位置（左边优先）,匹配到一次就停止
# tu = (11, 22, 33, 22, 44)
# i = tu.index(22)
# print(v,i)

# 索引取值
# tu = (11, 22, 33, 22, 44)
# v = tu[2]
# print(v)
# tu = (111, "alex", (11,22), [(33,44)], True, 33, 44,)
# v = tu[3][0][0]
# print(v)

# 切片
# tu = (11, 22, 33, 22, 44)
# v = tu[0:2]
# print(v)

# 转换
# s = "abcdefg"  # 字符串转元组
# v = tuple(s)
# print(v)
#
# li = ["aa", "bb", "cc", "dd"]  # 列表转元组
# v = tuple(li)
# print(v)

# tu = ("aa", "bb", "cc", "dd")  # 元组转列表
# v = list(tu)
# print(v)

# tu = ("aa", "bb", "cc", "dd")  # 元组转字符串
# tu = ("aa", "bb", "cc", 123)  # 元组中有数字参照列表写for循环进行转换
# s = "".join(tu)
# print(s)
```
# 字典
- 字典的key必须是能被hash的值，不能是列表、字典
- 字典的value可以是任何数据类型
- 字典内的元素是无序的，print 10次元素随机排列
示例：
```
info = {
    "k1": "v1", # 键值对
    "k2": "v2"
}
```
## 基本方法
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

# info = {
#     "k1": 18,
#     2: True,
#     "k3": [
#         11,
#         [],
#         (),
#         22,
#         33,
#         {
#             'kk1': 'vv1',
#             'kk2': 'vv2',
#             'kk3': (11,22),
#         }
#     ],
#     "k4": (11,22,33,44)
# }
# 索引取值
# v = info['k3'][5]['kk3'][0]
# print(v)
# 输出为：11

# 更新value
# dic = {"k1":1,"k2":2}
# dic["k2"] = 4 

# 新增字典元素
# dic = {"k1":1,"k2":2}
# dic["k3"] = 3 # key没有就新增

# del删除
# del info["k1"]
# print(info)

# 支持for循环

# 只打印字典中的key
# for i in info:
#     print(i)
# for i in info.keys():
#     print(i)

# 只打印字典中的value
# for i in info.values():
#     print(i)

# 打印字典中的key以及value
# for i in info.keys():
#     print(i,info[i])
# for k,v in info.items():
#     print(k,v)
```
## 一般方法
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen
info = {
    "k1": "v1", # 键值对
    "k2": "v2"
}
# 清空字典
# info.clear()
# print(info)

# 复制字典
# v = info.copy()
# print(v)

# 静态方法fromkeys,根据序列，创建字典，并指定统一的值
# v = dict.fromkeys(["k1",123,"k3"],123)
# print(v)
# 输出为：{'k1': 123, 123: 123, 'k3': 123}

# 根据key获取值，当key不存在时直接获取会报错，可以使用get去获取，key不存在就返回指定值
# v = info["k11"]
# print(v)
# v = info.get("k11",111)
# print(v)
# 输出为：111

# 删除指定key，并获取key对应的value,如果key不存在还可以提供默认值
# info.pop("k1")
# v = info.pop("k11",90)
# print(info,v)
# 输出为：{'k2': 'v2'} 90
# 随机删除字典中的元素（不支持指定），并获取被删除的元素
# k,v = info.popitem()
# print(info,k,v)
# 输出为：{'k1': 'v1'} k2 v2

# 设置key
# 如果key存在,不设置，获取当前key对应的值
# 如果key不存在,设置，获取当前key对应的值
# v = info.setdefault("k11",123)
# print(info,v)

# 更新值
# info.update({"k1":"11","k2":"22"})
# info.update(k1=22,k2=33)
# print(info)

# 判断key或value在不在字典中
# v = "k1" in info
# print(v)
# v = "v1" in info.values()
# print(v)
```
## 深浅拷贝
- 字典内容只有一级对象
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

import copy

test={'name':'lisen','age':20}

copy_test=copy.copy(test)
deepcopy_test=copy.deepcopy(test)

print(id(test)) # 内存地址不一样，说明他们是三个不同对象
print(id(copy_test))
print(id(deepcopy_test))

print(test)
print(copy_test)
print(deepcopy_test)

test['age']=22 # 改变了源数据
print('-------------')

# 源变了但深浅拷贝都没变
print(test)
print(copy_test)
print(deepcopy_test)

# 输出为：
# 4402309736
# 4402309808
# 4403638848
# {'name': 'lisen', 'age': 20}
# {'name': 'lisen', 'age': 20}
# {'name': 'lisen', 'age': 20}
# -------------
# {'name': 'lisen', 'age': 22}
# {'name': 'lisen', 'age': 20}
# {'name': 'lisen', 'age': 20}
```
- 字典内容有嵌套
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Author: Li Sen

import copy

test={'name':'lisen','hobby':['python','cycling']}

copy_test=copy.copy(test)
deepcopy_test=copy.deepcopy(test)

print(id(test)) # 内存地址不一样，说明他们是三个不同对象
print(id(copy_test))
print(id(deepcopy_test))

print(test)
print(copy_test)
print(deepcopy_test)

test['hobby'][0]='java' # 改变了源数据
print('-------------')

# 源变了但浅拷贝跟着变了，但深拷贝没变化
print(test)
print(copy_test)
print(deepcopy_test)

# 输出为：
# 4324997736
# 4324997808
# 4326326920
# {'name': 'lisen', 'hobby': ['python', 'cycling']}
# {'name': 'lisen', 'hobby': ['python', 'cycling']}
# {'name': 'lisen', 'hobby': ['python', 'cycling']}
# -------------
# {'name': 'lisen', 'hobby': ['java', 'cycling']}
# {'name': 'lisen', 'hobby': ['java', 'cycling']}
# {'name': 'lisen', 'hobby': ['python', 'cycling']}
```

# 布尔值
值为True 数据类型太多了,记住少数为False的  
值为False的有：None ""  () []  {} 0 ==> False
# enumerate函数
说明：  
函数原型：enumerate(sequence, [start=0])  #第二个参数为指定索引  
功能：将可循环序列sequence以start开始分别列出序列数据和数据下标
即对一个可遍历的数据对象(如列表、元组或字符串)，enumerate会将该数据对象组合为一个索引序列，同时列出数据和数据下标
示例：  
 ```
li = ['alex', 'eric', 'rain', 'rain', 'ok']
for i,v in enumerate(li,100):
    print(i,v)
输出为：
100 alex
101 eric
102 rain
103 rain
104 ok
 ```

# 作业
 1. 请用代码实现：利用下划线将列表的每一个元素拼接成字符串，li ＝ ['alex', 'eric', 'rain']
    ```
    li = ['alex', 'eric', 'rain']
    s = "_".join(li)
    print(s)
    ```
 2. 查找列表中元素，移除每个元素的空格，并查找以 a 或 A 开头 并且以 c 结尾的所有元素。
    - li = ["alec", " aric", "Alex", "Tony", "rain"]
    ``` 
      法1：
    # for i in li:
    #     i = i.split()
    #     i = "".join(i)
    #     if (i.startswith("a") or i.startswith("A"))and i.endswith("c"):
    #       print(i)
    # 法2：
    # for i in li:
    #     i = i.strip()
    #     v = i.endswith("c")
    #     v1 = i.startswith("a")
    #     v2 = i.startswith("A")
    #     if (v1 == True or v2 == True) and v == True:
    #         print(i)
    ```
    - tu = ("alec", " aric", "Alex", "Tony", "rain")
    > 元组跟列表处理方式一致，不在赘述
    - dic = {'k1': "alex", 'k2': ' aric',"k3": "Alex", "k4": "Tony"}
    ```
    # for i in dic.values():
    #     i = i.strip()
    #     v = i.endswith("c")
    #     v1 = i.startswith("a")
    #     v2 = i.startswith("A")
    #     if (v1 == True or v2 == True) and v == True:
    #         print(i) 
    ```
 3. 写代码，有如下列表，按照要求实现每一个功能  
 li = ['alex', 'eric', 'rain', 'rain', 'ok']
    ```
    # a. 计算列表长度并输出
    # v = len(li)
    # print(v)
    
    # b. 列表中追加元素 “seven”，并输出添加后的列表
    # li.append("seven")
    # print(li)
    
    # c. 请在列表的第 1 个位置插入元素 “Tony”，并输出添加后的列表
    # li.insert(0,"Tony")
    # print(li)
    
    # d. 请修改列表第 2 个位置的元素为 “Kelly”，并输出修改后的列表
    # li[1]="Kelly"
    # print(li)
    
    # e. 请删除列表中的元素 “eric”，并输出修改后的列表
    # li.remove("eric")
    # print(li)
    
    # f. 请删除列表中的第 2 个元素，并输出删除的元素的值和删除元素后的列表
    # v = li.pop(1)
    # print(v,li)
    
    # g. 请删除列表中的第 3 个元素，并输出删除元素后的列表
    # del li[2]
    # print(li)
    
    # h. 请删除列表中的第 2 至 4 个元素，并输出删除元素后的列表
    # del li[1:4]
    # print(li)
    
    # i. 请将列表所有的元素反转，并输出反转后的列表
    # li.reverse()
    # print(li)
    
    # j. 请使用 for、len、range 输出列表的索引
    # for i in range(len(li)):
    #     print(i)
    # for i in li:
    #     v = li.index(i)
    #     print(v,i)
    
    # k. 请使用 enumrate 输出列表元素和序号（序号从 100 开始）
    # for i,v in enumerate(li,100):
    #     print(i,v)
    
    # l. 请使用 for 循环输出列表的所有元素
    # for i in li:
    #     print(i)
    ```
 4. 写代码，有如下列表，请按照功能要求实现每一个功能  
  li = ["hello", 'seven', ["mon", ["h", "kelly"], 'all'], 123, 446]
    ```
    # a.请根据索引输出 “Kelly”
    v = li[2][1][1]
    print(v)
    # b.请使用索引找到'all'元素并将其修改为 “ALL”，如：li[0][1][9]...
    li[2][2] = "ALL"
    print(li) 
    ```
    > 元组是不可修改列表，其数据操作跟列表基本一致，这里不再赘述
  5. 有如下变量，请实现要求的功能
     ```
     tu = ("alex", [11, 22, {"k1": 'v1', "k2": ["age", "name"], "k3": (11,22,33)}, 44])
     # a. 讲述元祖的特性
     # 元组的一级元素不可被修改增加删除，有序，可迭代，可切片，可索引，可转换为列表。
     # b. 请问 tu 变量中的第一个元素 “alex” 是否可被修改？
     # 不可被修改
     # c. 请问 tu 变量中的"k2"对应的值是什么类型？是否可以被修改？如果可以，请在其中添加一个元素 “Seven”
     # k2对应的值是列表 可以被修改
     v = tu[1][2]["k2"].append("seven")
     print(tu)
     # 输出为：('alex', [11, 22, {'k2': ['age', 'name', 'seven'], 'k3': (11, 22, 33), 'k1': 'v1'}, 44])
     # d. 请问 tu 变量中的"k3"对应的值是什么类型？是否可以被修改？如果可以，请在其中添加一个元素 “Seven”
     # k3对应的是元组 不可修改
     ```
  6. 字典操作：
     ```
     dic = {'k1': "v1", "k2": "v2", "k3": [11,22,33]}
     # a.请循环输出所有的 key
     # for i in dic.keys():
     #     print(i)
     # b. 请循环输出所有的 value
     # for i in dic.values():
     #     print(i)
     # c. 请循环输出所有的 key 和 value
     # for k,v in dic.items():
     #     print(k,v)
     # d. 请在字典中添加一个键值对，"k4": "v4"，输出添加后的字典
     # dic.setdefault("k4","v4")
     # print(dic)
     # e. 请在修改字典中 “k1” 对应的值为 “alex”，输出修改后的字典
     # dic.update({"k1":"alex"})
     # dic.update(k1="alex")
     # print(dic)
     # f. 请在k3对应的值中追加一个元素44，输出修改后的字典
     # dic["k3"].append(44)
     # print(dic)
     # g. 请在k3对应的值的第1个位置插入个元素18，输出修改后的字典
     # dic["k3"].insert(0,18)
     # print(dic)
     ```
  7. 转换
     ```
     # a. 将字符串s = "alex"转换成列表
     # s = "alex"
     # li = list(s)
     # print(li)
     # b. 将字符串s = "alex"转换成元祖
     # tu = tuple(s)
     # print(tu)
     
     # c. 将列表li = ["alex", "seven"]转换成元组
     # li = ["alex", "seven"]
     # tu = tuple(li)
     # print(tu)
     
     # d. 将元祖tu = ('Alex', "seven")转换成列表
     # tu = ('Alex', "seven")
     # li = list(tu)
     # print(li)
     
     # e. 将列表li = ["alex", "seven"]转换成字典且字典的key按照10开始向后递增
     # li = ["alex", "seven"]
     # dic = {}
     # for i,v in enumerate(li,10):
     #    dic[i] = v
     # print(dic)
     ```
  8. 元素分类:有如下值集合 [11,22,33,44,55,66,77,88,99,90]，将所有大于 66 的值保存至字典的第一个 key 中，将小于 66 的值保存至第二个 key 的值中。即： {'k1': 大于 66 的所有值, 'k2': 小于 66 的所有值}
     ```
     # li = [11,22,33,44,55,66,77,88,99,99]
     # 法1：
     # dic = {}
     # li1 = []
     # li2 = []
     # for i in li:
     #     if i > 66:
     #         li1.append(i)
     #     else:
     #         li2.append(i)
     # dic["k1"] = li1
     # dic["k2"] = li2
     # print(dic)
     # 法2：
     # li.sort()
     # print(li)
     # z = li.index(66)
     # l = len(li)
     # print(z,l)
     # dic = {"k1":li[z+1:l],"k2":li[0:z]}
     # print(dic)
     ```
  9. 先输出商品列表，"手机", "电脑", '鼠标垫', '游艇' ，让用户选择是否进行：
        - 是否用户添加商品，并显示新商品列表 
        - 用户输入序号显示对应的商品
     ```
     #!/usr/bin/env python
     # -*- coding: utf-8 -*-
     # Author: Li Sen
     # li = ["手机", "电脑", '鼠标垫', '游艇']
     # for k,v in enumerate(li,1):
     #     print(k,v)
     # s1 = input("您是否要添加商品：(y/n)")
     # if s1 == "y" or s1 == "Y":
     #     a1 = input("请输入商品名称：")
     #     li.append(a1)
     #     for k,v in enumerate(li,1):
     #             print(k,v)
     # s2 =  input("您是否要查找商品:(y/n)")
     # if s2 == "y" or s2 == "Y":
     #     a2 = int(input("请输入商品序号："))
     #     for k,v in enumerate(li,1):
     #         print(k,v)
     #     print(li[a2-1])
     ```
  10. 将下列字典做成多级联动：用可以选择查看或者添加某一级内容
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # dic = {
      #     "植物":
      #         {"草本植物":
      #              ["牵牛花", "瓜叶菊", "葫芦", "翠菊", "冬小麦", "甜菜"],
      #          "木本植物":
      #              ["乔木", "灌木", "半灌木", "如松", "杉", "樟"],
      #          "水生植物":
      #              ["荷花", "千屈菜", "菖蒲", "黄菖蒲", "水葱", "再力花", "梭鱼草"]},
      #     "动物":
      #         {"两栖动物":
      #              ["山龟", "山鳖", "石蛙", "娃娃鱼", "蟾蜍", "龟", "鳄鱼", "蜥蜴", "蛇"],
      #          "禽类":
      #              ["雉鸡", "原鸡", "长鸣鸡", "昌国鸡", "斗鸡", "长尾鸡", "乌骨鸡"],
      #          "哺乳类动物":
      #              ["虎", "狼", "鼠", "鹿", "貂", "猴", "貘", "树懒", "斑马", "狗"]}}
      # 
      # while True:
      #     print("物种")
      #     l = {}
      #     count = 0
      #     for i in dic:
      #         count += 1
      #         print(count, i)
      #         l[count] = i
      #     w = input("请您输入您想要查找的物种（输入q则退出，输入t添加物种）:")
      #     if w == "t":
      #         a = input("请输入您想添加物种:")
      #         a1 = input("请输入您想添加分类:")
      #         a2 = input("请输入您想添加品种:")
      #         dic[a] = {a1: [a2]}
      #         continue
      #     if w == "q":
      #         exit()
      #     v1 = w.isdecimal()
      #     if v1 == False:
      #         print("没有此序号请重新输入:")
      #         continue
      #     w = int(w)
      #     if w > count or w < 1:
      #         print("没有此序号请重新输入:")
      #         continue
      # 
      #     while True:
      #         x = l[w]
      #         p = dic[x]
      #         l2 = {}
      #         count2 = 0
      #         for i2 in p:
      #             count2 += 1
      #             print(count2, i2)
      #             l2[count2] = i2
      #         w2 = input("请您输入您想要查找的分类输入q则退出,输入b返回上一级,输入t添加分类:")
      #         if w2 == "t":
      #             a1 = input("请输入您想添加分类:")
      #             a2 = input("请输入您想添加品种:")
      #             dic[x][a1]=[a2]
      #             continue
      #         if w2 == "q":
      #             exit()
      #         if w2 == "b":
      #             break
      #         v2 = w2.isdecimal()
      #         if v2 == False:
      #             print("没有此序号请重新输入:")
      #             continue
      #         w2 = int(w2)
      #         if w2 > count2 or w2 < 1:
      #             print("没有此序号请重新输入:")
      #             continue
      #         while True:
      #             x2 = l2[w2]
      #             p2 = dic[x][x2]
      #             l3 = {}
      #             count3 = 0
      #             for i3 in p2:
      #                 count3 += 1
      #                 print(count3, i3)
      #                 l3[count3] = i3
      #             f2 = input("输入q则退出,输入b返回上一级，输入t添加品种:")
      #             if f2 != "q"and f2 != "b" and f2 != "t":
      #                 continue
      #             if f2 == "q":
      #                 exit()
      #             if f2 == "b":
      #                 break
      #             if f2 == "t":
      #                 a2 = input("请输入您想添加品种:")
      #                 dic[x][x2].append(a2)
      #                 continue      
      ```
  11. 获取l1和l2中内容不同的元素：
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # l1 = [11,22,33]
      # l2 = [22,33,44]
      # l = []
      # a. 获取内容相同的元素列表
      # for i in l1:
      #     if i in l2:
      #         l.append(i)
      # print(l)
      # b. 获取l1中有，l2中没有的元素列表
      # for i in l1:
      #     if i not in l2:
      #         l.append(i)
      # print(l)
      # c. 获取l2中有，l1中没有的元素列表
      # for i in l2:
      #     if i not in l1:
      #         l.append(i)
      # print(l)
      # d. 获取l1和l2中内容都不同的元素
      # for i in l1:
      #     if i not in l2:
      #         l.append(i)
      # for i in l2:
      #     if i not in l1:
      #         l.append(i)
      # print(l)
      ```
  12. 利用 For 循环和 range 输出:
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # a. For循环从大到小输出 1 - 100
      # for i in range(1,101):
      #     print(i)
      # b. For循环从小到到输出 100 - 1
      # for i in range(100,0,-1):
      #     print(i)
      # c. While循环从大到小输出 1 - 100
      # count = 0
      # while  count < 100:
      #     count += 1
      #     print(count)
      # d. While循环从小到到输出 100 - 1
      # count = 100
      # while  count > 0:
      #     print(count)
      #     count -= 1
      ``` 
  13. 购物车
      ```
      功能要求：
      要求用户输入总资产，例如： 2000
      显示商品列表，让用户根据序号选择商品，加入购物车
      购买，如果商品总额大于总资产，提示账户余额不足，否则，购买成功。
      goods = [
      {"name": "电脑", "price": 1999},
      {"name": "鼠标", "price": 10},
      {"name": "游艇", "price": 20},
      {"name": "美女", "price": 998},
      ]
      ```
      ```
      法1：
      # #!/usr/bin/env python
      # # -*- coding: utf-8 -*-
      # # Author: Li Sen
      # goods = [
      # {"name": "电脑", "price": 1999},
      # {"name": "鼠标", "price": 10},
      # {"name": "游艇", "price": 20},
      # {"name": "美女", "price": 998},
      # ]
      # gwc = []
      # count = 0
      # # 打印物品清单
      # for i in goods:
      #     count += 1
      #     print(count,goods[count - 1]["name"],goods[count -1 ]["price"])
      # 
      # while True:
      #     sum = int(input("请输入总金额:"))
      #     # 打印可购买物品的清单
      #     count1 = 0
      #     for i1 in goods:
      #         count1 += 1
      #         aa= goods[count1 -1 ]["price"]
      #         if aa < sum:
      #             print(count1,goods[count1 - 1]["name"],goods[count1 -1 ]["price"])
      #     # 获取购买的物品
      #     while True:
      #         t1 = int(input("请输入商品序号:"))
      #         n1 = goods[t1 - 1]["name"]
      #         p1 = goods[t1 - 1]["price"]
      #         print(p1)
      #         sum -= p1
      #         # 判断余额
      #         if sum < 0:
      #             print("你的金额不足")
      #             break
      #         else:
      #             print("购买成功")
      #             gwc.append(n1)
      #             print("购物车：\n{0}".format(gwc))
      #             print("你剩余金额为：{0}".format(sum))
      #         count1 = 0
      #         count2 = 0
      #         for i1 in goods:
      #             count1 += 1
      #             aa = goods[count1 - 1]["price"]
      #             if aa < sum:
      #                 count2 += 1
      #         if count2 < 1:
      #             print("你的金额不足")
      #             break
      #         while True:
      #             c1 = input("是否继续购买(y/n):")
      #             if c1 == "y":
      #                 break
      #             elif c1 == "n":
      #                 exit()
      #             else:
      #                 print("输入错误，请重新输入：")
      #                 continue
      # 
      #         count3 = 0
      #         for i3 in goods:
      #             count3 += 1
      #             aaa = goods[count3 - 1]["price"]
      #             if aaa < sum:
      #                 print(count3, goods[count3 - 1]["name"], goods[count3 - 1]["price"])
      ```
      ```
      法2：
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # goods = [
      #     {"name": "电脑", "price": 1999},
      #     {"name": "鼠标", "price": 10},
      #     {"name": "游艇", "price": 20},
      #     {"name": "美女", "price": 998},
      # ]
      # g = int(input("请您输入您的总资产："))
      # o = []
      # count = 0
      # for i, v in enumerate(goods, 1):
      #     k = v['price']
      #     if g >= k:
      #         count += 1
      #         print(count, v['name'], v["price"])
      #         o +=[{"name": v['name'],"price":v["price"]}]
      # 
      # t = 0
      # h = []
      # while True:
      #     l = (input("请输入您想要购买的商品："))
      #     v1 = l.isdecimal()
      #     if v1 == False:
      #         print("没有此序号请重新输入")
      #         continue
      #     l = int(l)
      #     if l > count:
      #         print("没有此序号请重新输入")
      #         continue
      #     c = int(o[l - 1]["price"])
      #     s = o[l - 1]["name"]
      #     t = t + c
      #     f = (s,c)
      #     h.append(f)
      #     print("购物车：")
      #     for h1 in h:
      #         print("\t",h1[0],h1[1])
      #     p = input("是否继续购买")
      #     if p == "n" or p == "N" or p == "No" or p == "no":
      #         if t > g:
      #             print("账户余额不足")
      #             break
      #         else:
      #             print("购买成功")
      #             break
      ```
  14. 分页显示内容
      ```
      a. 通过 for 循环创建 301 条数据，数据类型不限，如：
      alex-1 alex1@live.com pwd1
      alex-2 alex2@live.com pwd2
      alex-3 alex3@live.com pwd3
      ...
      b. 提示用户 请输入要查看的页码，当用户输入指定页码，则显示指定数据
      注意：
      - 每页显示 10 条数据
      - 用户输入页码是非十进制数字，则提示输入内容格式错误
      ```
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
       
      # for i in range(302):
      #     # print("alex-{0} alex{1}@live.com pwd{2}\n".format(i,i,i))
      #     # print("alex-{k} alex{j}@live.com pwd{l}\n".format(k=i,j=i,l=i))
      #     print("alex-{k} alex{j}@live.com pwd{l}\n".format_map({"k":i,"l":i,"j":i}))
      
      # li = []
      # for i in range(302):
          # print("alex-{0} alex{1}@live.com pwd{2}\n".format(i,i,i))
          # print("alex-{k} alex{j}@live.com pwd{l}\n".format(k=i,j=i,l=i))
          # a = "alex-{k} alex{j}@live.com pwd{l}".format_map({"k":i,"l":i,"j":i})
          # li.append(a)
      # while True:
      #     a = input("请输入要看的页码：")
      #     if a.isdecimal() == False:
      #         print("输入错误，请重新输入：")
      #         continue
      #     elif int(a) < 1 or int(a) >31:
      #         print("输入错误，请重新输入：")
      #         continue
      #     else:
      #         for i in (li[(int(a) - 1) * 10:int(a) * 10]):
      #             print(i)
      #         while True:
      #             b = input("是否继续查看(y/q):")
      #             if b == "y":
      #                 break
      #             elif b == "q":
      #                 exit()
      #             else:
      #                 print("输入有误，请重新输入:")
      #                 continue
      ```
  15. 1、 2、 3、 4、 5、 6、 7、 8, 8 个数字能组成多少个互不相同且无重复数字的两位数？
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      
      # 法1：
      # li = []
      # for i in range(1,9):
      #     for i2 in range(1,9):
      #         i3=str(i) + str(i2)
      #         if i3 not in li:
      #             li.append(i3)
      # print(li)
      # a = len(li)
      # print(a)
      
      
      # 法2：
      # li = []
      # for i in range(1, 9):
      #     for x in range(1, 9):
      #         a = ("{0}{1}".format(i, x))
      #         if a not in li:
      #             li.append(a)
      # print(li)
      # a = len(li)
      # print(a)
      
      ```
  16. 利用 for 循环和 range 输出 9 * 9 乘法表
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # 法1：
      # l = []
      # for i in range(1,10):
      #     l.clear()
      #     for x in range(1,i+1):
      #         b="{0}*{1}={2} ".format(x,i,i*x)
      #         l.append(b)
      #     c="".join(l)
      #     print(c)
      # 法2：
      # for i in range(1,10):
      #     string = ""
      #     for j in range(1,i+1):
      #         string += str(j) + "*" + str(i)+"="+str(i * j)+ "\t"
      #     print(string)
      ```
  17. 用 Python 开发程序自动计算方案：
公鸡 5 文钱一只，母鸡 3 文钱一只，小鸡 3 只一文钱，用 100 文钱买 100 只鸡,其中公鸡，母鸡，小鸡都必须要有，问公鸡，母鸡，小鸡要买多少只刚好凑足 100 文钱  
      ```
      #!/usr/bin/env python
      # -*- coding: utf-8 -*-
      # Author: Li Sen
      # for g in range(1, 100//5):
      #     g1 = g * 5
      #     for m in range(1, 100//3):
      #         m1 = m * 3
      #         for x in range(1, 100):
      #             x1 = x * 1/3
      #             if g1 + m1 + x1 == 100 and g + m + x ==100:
      #                       print(" 100 文钱买 100 只鸡,其中公鸡{0}只，母鸡{1}只，小鸡{2}只".format(g,m,x))      
      ```

