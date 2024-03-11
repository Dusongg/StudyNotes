# python语法

## 一，基本概念

### 1注释

单行注释：# 注释内容    快捷键ctrl + /

多行注释：” “ ”    /'''

​                     注释内容

​                    “ ” “    /'''       

### 2变量

定义变量：变量名 = 值

标识符命名规则：

- 由数字，字母，下划线组成
- 不能由数字开头
- 不能使用内置关键字
- 严格区分大小写

命名习惯：

- 大驼峰：即每个单词首字母都大写，如：MyName。
- 小驼峰：第二个（含）以后的单词首字母大写，如：myName。
- 下划线：如my_name

### 3数据类型

print(type(数据))展示数据对应的数据类型

数值：              int（整型）  float（浮点型）

布尔型              True（真）False（假）

str （字符串）  数据都要带引号 

list（列表）       eg:[10,20,30]

tuple（元组）   eg:(10,20,30)

set（集合）       eg：{10,20,30}

dict（字典）      eg：{'name': 'Tom',   'age': 18}  键值对

### 4输出

#### 4.1格式化符号

| 格式化符号 | 转换           |
| ----- | ------------ |
| %s    | 字符串          |
| %d    | 有符号的十进制整数    |
| %f    | 浮点数          |
| %c    | 字符           |
| %u    | 无符号的十进制整数    |
| %o    | 八进制整数        |
| %x    | 十六进制整数(小写ox) |
| %X    | 十六进制整数(大写OX) |
| %e    | 科学计数法(小写‘e’) |
| %E    | 科学计数法（大写‘E’） |
| %g    | %f与%e的简写     |
| %G    | %f和%E的简写     |



注意：

- %.2f，表示保留小数点后两位数字。
- %06d，表示输出的整数显示位数，不足以0补全，超出当前位数则原样输出

#### **※※4.2示例：**

输出：age = 18，name=‘gary’

​           print（‘年龄是%d岁’ % age） 结果为：年龄是18岁

​          print（‘明年年龄是%d岁’ % age+1） 结果为：明年年龄是19岁

​           print（‘名字是%s，年龄是%d岁’ % (name,age)） 结果为：名字是gary，年龄是18岁

f格式化字符串：f'{表达式}'

​          print（f‘名字是{name}，年龄是{age}岁’）结果为：名字是gary，年龄是18岁

注：f-格式化字符串是Python3.6中新增的格式化方法，该方法更简单易读。

三种输出格式：

str1 = hei

**1，print('hei')**

**2，print('你好，%s' % str1)**

**3，print(f'你好，{str1}')**

#### 4.3转义字符

- \n：换行
- \t：制表符，一个tab键（4个空格）的距离。

#### 4.4结束符

print('输出的内容',end="\n")

在python中，print()，默认自带end="\n"这个换行结束符，所以导致每两个print直接会换行展示，用户可以按需求更改结束符。

### 5输入

输入语法：input("提示信息")

输入的特点：

- 当程序执行到input，等待用户输入，输入完成之后才继续向下执行。
- 在python中，input接受用户输入后，一般存储到变量，方便使用。
- 在python中，input会把接收到的任意用户输入的数据都当做字符串处理。

### 6数据类型转换 

#### 6.1转换数据类型的作用

问：input()接受用户输入的数据都是字符串类型，如果用户输入1，想得到整型该如何操作？

答：转换数据类型即可，即字符串类型转换成整型。

#### 6.2转换数据类型的函数

| 函数                    | 说明                             |
| --------------------- | ------------------------------ |
| int(x[,base])         | 将x转换为一个整数。                     |
| float(x)              | 将x转换为一个浮点数。                    |
| complex(real [,imag]) | 创建一个复数，real为实部，imag为虚部         |
| str(x)                | 将对象x转换为字符串。                    |
| repr(x)               | 将对象x转换为表达式字符串                  |
| eval(str)             | 用来计算在字符串中的有效Python表达式，并返回一个对象。 |
| tuple(s)              | 将序列s转换为一个元组。                   |
| list(s)               | 将序列s转换为一个列表。                   |
| chr(x)                | 将一个整数转换成一个Unicode字符            |
| ord(x)                | 将一个字符转换为它的ASCll整数值             |

注：eval（str），字符串里面是什么就转换成什么数据，如‘1’，‘1.0’，‘[1,2,3]’，'(1,2,3)'   --->   1，1.0，[1,2,3]，(1,2,,3)

### 7运算符

运算符类别：

- 算术运算符
- 赋值运算符
- 复合赋值运算符
- 比较运算符
- 逻辑运算符

#### 7.1算术运算符

| 运算符  | 描述   | 实例           |
| ---- | ---- | ------------ |
| +    | 加    | 1+1输出结果为2    |
| -    | 减    | 1-1输出结果为0    |
| *    | 乘    | 1*2输出结果为2    |
| /    | 除    | 4/2输出结果为2    |
| //   | 整除   | 9//4输出结果为2   |
| %    | 取余   | 9%4输出结果为1    |
| **   | 指数   | 2**4输出结果为16  |
| （）   | 小括号  | 小括号用来提高运算优先级 |

注：

- 混合运算优先级顺序：()高于** 高于 *  /  //  %  高于+   - 。

####   7.2赋值运算符

| 运算符  | 描述   | 实例                |
| ---- | ---- | ----------------- |
| =    | 赋值   | 将=右侧的结果赋值给等号左侧的变量 |

- 单个变量赋值

```python
num = 1
print(num)
```

- 多个变量赋值

```python
num，f1，str1 = 10,1.5，'hello world'
print(num)
print(f1)
print(str1)
```

- 多变量赋相同值

```python
a = b = 10
print(a)
print(b)
```

#### 7.3复合赋值运算符

| 运算符  | 描述      | 实例             |
| ---- | ------- | -------------- |
| +=   | 加法赋值运算符 | c+=a等价于c=c+a   |
| -=   | 减法赋值运算符 | c-=a等价于c=c-a   |
| *=   | 乘法赋值运算符 | c*=a等价于c=c*a   |
| /=   | 除法赋值运算符 | c/=a等价于c=c/a   |
| //=  | 整除赋值运算符 | c//=a等价于c=c//a |
| %=   | 取余赋值运算符 | c%=a等价于c=c%a   |
| **=  | 幂赋值运算符  | c**=a等价于c=c**a |

```
num = 10
num+= 1+2
print(num)    #结果为30
#先算+=右边算式再进行复合赋值运算
```

注：先算复合赋值运算符右边的表达式，再算复合赋值运算

#### 7.4比较运算符

比较运算符也叫关系运算符，通常用来做判断。

| 运算符  | 描述                                       | 实例                                       |
| ---- | ---------------------------------------- | ---------------------------------------- |
| ==   | 判断相等。如果两个操作数的结果相等，则条件结果为真(True)，否则条件结果为假(Fales) | 如a=3，b=3，则(a==b)为True                    |
| !=   | 不相等。如果两个操作数的结果不相等，则条件结果为真(True)，否则条件结果为假(Fales) | 如a=3，b=3，则(a==b)为True，如a=3，b=2，则(a！=b)为True |
| >    | 运算符左侧操作数的结果是否大于右侧操作数结果，如果大于，则条件为真，否则为假。  | 如a=6，b=3，则(a>b)为True                     |
| <    | 运算符左侧操作数结果是否小于右侧操作数结果，如果小于，则条件为真，否则为假。   | 如a=6，b=3，则(a<b)为False                    |
| >=   | 运算符左侧操作数结果是否大于等于右侧操作数结果，如果大于，则条件为真，否侧为假  | 如a=6，b=3，则(a>=b)为True                    |
| <=   | 运算符左侧操作数结果是否小于等于右侧操作数结果，如果小于，则条件为真，否侧为假  | 如a=3，b=3，则(a<=b)为True                    |

#### 7.5逻辑运算符

| 运算符  | 逻辑表达式   | 描述                                       | 实例                                |
| ---- | ------- | ---------------------------------------- | --------------------------------- |
| and  | x and y | 布尔"与"：如果x为False，x and y返回False，否则它返回y的值。 | True and False，返回False。           |
| or   | x or y  | 布尔"或"：如果x为True，它返回True，否则它返回y的值。         | False or True，返回True。             |
| not  | not x   | 布尔"非"：如果x为True，返回False。如果x为False，返回True。 | not True返回 False，not False 返回True |

```
a = 1
b = 2
c = 3
print((a<b)and(b<c))  #True
print((a>b)and(b<c))  #False
print((a<b)or(b<c))   #True
print((a>b)or(b>c))   #False
print(not False)      #True
```

## 二，基本操作

### 1判断语句

条件成立执行某些代码，不成立执行某些代码。

```python
if 条件表达式：
    执行语句1
    执行语句2
    ......
else：
    执行语句1
    执行语句2
    ......
    
#多重判断
if 条件1：
    执行语句1
    执行语句2
    ......
elif 条件2：#通else if其中任一条件满足后，之后程序不执行
    执行语句1
    执行语句2
    ......
......
else:
     以上条件都不成立执行的代码
    
# if嵌套
if 条件1：
    执行语句1
    执行语句2
    ......
    if 条件2：
        执行语句1
        执行语句2
        ......
    
```

随机数：

```python
# 1导入random模块
import 模块名（random）
# 2使用random模块中的随机整数功能
random.randint(开始，结束)  #randint随机整数

```

三目运算符：

```python
#语法： 条件成立执行的表达式 if 条件 else条件不成立执行的表达式
#示例如下
a = 1
b = 2
c = a if a>b else b
print(c)
```

### 2循环语句

```python
while 条件1：
    执行语句1
    执行语句2
    ......
# break打破循环，continue跳出此次循环，执行下一次循环

# while循环嵌套
while 条件1：
    执行语句1
    ......
    while 条件2：
         执行语句1
         ......
        
# for循环 语法如下
for 临时变量 in 序列： 
    执行语句1
    执行语句2
    ......
# 实例
str1 = 'hello'
for i in str1:
    print(i) 
'''
输出结果 
h 
e 
l 
l 
o
''' 
# while...else
while 条件：
    执行语句1
    ...
else：
    循环正常结束后需要执行的代码，一般与循环语句有依赖关系
    #如：必须循环结束后才能输出，没有执行循环输出没意义。
#else与直接print区别：print不管是否执行循环都会执行。
#while循环中，如果出现break打破循环，else之后语句不执行。else之后语句是正常结束后执行。continue不影响，正常执行。
# for...else
for 临时变量 in 序列：
    执行语句1
    ...
else：
    循环正常结束后需要执行的代码，一般与循环语句有依赖关系
#对于break，continue的用法for else与while else一致。
```

### 3字符串

- 原始字符串

```python
s = r"adsfasfd/adsfga/asf/s"
```

- 

#### 3.1，字符串特征

```python
#字符串特征
name1 = 'hello'
name2 = "hello"

#三引号字符串
name3 = '''hei'''
name4 = """hei"""
a = '''hei
    siri'''  # 结果换行输出
b = """hei 
    siri"""   # 同上
# 单双引号混用
c = "I'm good"
c = 'I\'m good' # 'I'm good'报错，\转义符号'

# 下标
str1 = "abc"
print(str1[0])  # 结果是a

```

#### ※※3.2，切片操作 —— 左闭右开（列表也有切片操作）

```python
#切片
#语法： 序列[开始位置下标：结束位置下标：步长]
'''注:
1,不包括结束位置下标对应的数据，正负整数均可。
2,步长是选取间隔，正负整数均可，默认为1.
'''
str2 = "abcdefg"
print(str2[2:5:1])  #cde
print(str2[2:5:2])  #ce
print(str2[:5])  #abcde
print(str2[1:])  #bcdefg
print(str2[:])  #abcdefg
# 步长是复数
print(str2[::-1])  #gfedcba 倒叙输出 
print(str2[1::2])  # 从下标1开始，步长为2
print(str2[-3:-1])  #ef
print(str2[-3:-1:1])  #ef
print(str2[-3:-1:-1])  #不能选取出数据，从-3到-1结束，选取方向为从左到右，但-1步长表示，从右到左选取
# 如果我们的选取方向（下标开始到结束的方向）和步长方向冲突，则无法选取数据。 
print(str2[-1:-3:-1])  #gf  方向一致

```

#### ※※3.3，字符串常用操作方法

1 查找：字符串查找方法既是查找子串在字符串中的位置或出现的次数。

```python
# 查找
'''
find():检测某个字串是否包含在这个字符串中，如果'在'返回这  个子串开始的位置下标，否则返回-1.
index():检测某个字串是否包含在这个字符串中，如果'在'返回这个子串开始的位置下标，否则则报异常。
count():返回某个子串在字符串中出现的次数。
区别：
rfind():和find()功能相同，但查找方向为右侧开始。
rindex():和index()功能相同，但查找方向为右侧开始。
下标仍是从左侧开始计算。

语法:
字符串序列.find(子串，开始位置下标，结束位置下标)。
字符串序列.index(子串，开始位置下标，结束位置下标)。
字符串序列.count(子串，开始位置下标，结束位置下标)。

注：开始和结束位置下标可以省略，表示在整个字符串序列中查找。
'''
#实例: 
str1 = "my dream is peace of world"
print(str1.find('my')) #0
print(str1.index('is')) #9
print(str1.count('world')) #1

```

2 修改：字符串修改是指通过函数形式修改字符串中的数据。

```python
'''
replace():替换
split():按照指定字符分割字符串
join():用一个字符或者子串合并字符串，既是将多个字符串合并为一个新的字符串。

语法：
字符串序列.replace(旧子串，新子串，替换次数)
字符串序列.split(分割字符，num)
字符或子串.join(多字符组成的序列)

注：
1，替换次数如果超出子串出现次数，则替换次数为该子串出现次数。
2，num表示的是分割字符出现的次数，即将来返回数据个数为num+1个。
'''
#实例：
#1，replace：
str1 = "my dream is peace of world"
print(str1.replace('my'，'him'))
#结果为：him dream is peace of world
print(str1.replace('my'，'him'，2))
#结果为：him dream is peace of world
print(str1)
#结果为：my dream is peace of world

#2，split
print(str1.split('peace'))
#结果为：['my dream is',' of world']
print(str1.split(' '))
#结果为：['my','dream','is','peace','of','world']
print(str1.split(' '，2))
#结果为：['my','dream','is peace of world']
print(str1)
#结果为：my dream is peace of world 

#3, join
mylist = ['aa','bb','cc']
new_list = '...'.join(mylist)
print(new_list)
#aa...bb...cc

#注：字符串修改不能修改原有字符串。
```

修改-大小写转换：

```python
# capitalize():将字符串第一个字符转换成大写。

str1 = "my Dream is peace of world"
print(str1.capitalize())
#结果为：My dream is peace of world 

# 注：capitalize()函数转换后，只将字符串第一个字符大写，其他字符全都小写。

# title():将字符串每个单词首字母转换成大写。
print(str1.title())
#结果为：My Dream Is Peace Of World 

#lower():将字符串中大写转小写。
print(str1.lower())
#结果为：my dream is peace of world 

#upper():将字符串中小写转大写。
print(str1.upper())
#结果为：MY DREAM IS PEACE OF WORLD 

```

修改-删除空白字：

```python
#lstrip():删除字符串左侧空白字符。
str1 = "   my Dream is peace of world   "
print(str1.lstrip())
#结果为：my Dream is peace of world   .

#rstrip():删除字符串右侧空白字符。
print(str1.lstrip())
#结果为：   my Dream is peace of world.

#strip():删除字符串两侧空白字符。
print(str1.lstrip())
#结果为：my Dream is peace of world.

```

修改-字符串对齐：

```python
'''
 ljust():返回一个原字符串左对齐，并使用指定字符(默认空格)填充至对应长度的新字符串。
 
 rjust():返回一个原字符串右对齐，并使用指定字符(默认空格)填充至对应长度的新字符串。语法与ljust()相同。
  
 center():返回一个原字符串居中对齐，并使用指定字符(默认空格)填充至对应长度的新字符串。语法与ljust()相同。
''' 

# 语法：字符串序列.ljust(长度，填充字符)

str1 = 'hello'
print(str1.ljust(10,'.'))
#结果为：hello.....

print(str1.ljust(10,'.'))
#结果为：.....hello

print(str1.ljust(10,'.'))
#结果为：..hello...

```

判断开头或结尾：

判断真假，返回结果是布尔型数据类型：True或False。

```python
'''
startswith():检查字符串是否以指定子串开头，是则返回True，否则返回False。如果设置开始和结束位置下标，则在指定范围内检查。

endswith():检查字符串是否以指定子串结尾，是则返回True，否则返回False。如果设置开始和结束位置下标，则在指定范围内检查。
'''

'''
语法：
字符串序列.startswith(子串，开始位置下标，结束位置下标)
字符串序列.endswith(子串，开始位置下标，结束位置下标)
'''
str1 = "my Dream is peace of world"
print(str1.startswirch('is'))
#结果为：False

print(str1.startswirch('world'))
#结果为：True

# 注：开头可以是m/my/my /...,结尾同理。
```

判断：

```python
'''
isalpha():如果字符串至少有一个字符并且所有字符都是字母则返回True，否则返回False。

isdigit():如果字符串只包含数字，则返回True，否则返回False。

isalnum():如果字符串至少有一个字符并且所有字符都是字母或数字则返回True，否则返回False。

isspace():如果字符串只包含空白，则返回True，否则返回False。
'''

str1 = 'hello'
str2 = 'hello123'
print(str1.isalpha())
#结果为：True
print(str2.isalpha())
#结果为：False

str3 = '123456'
str4 = '123hello'
print(str3.isdigit())
#结果为：True
print(str4.isdisit())
#结果为：False

str5 = '123aa'
str6 = '123-hello'
print(str5.isalnum())
#结果为：True
print(str6.isalnum())
#结果为：False

str7 = '     '
str8 = '123aa'
print(str7.isspace())
#结果为：True
print(str8.isspace())
#结果为：False
```

### 4列表

列表可以一次性存储多个数据，且可以为不同数据类型。

常用操作有：crud，增，删，查，改。

```python
[数据1，数据2，数据3......]
```

#### 1，查找函数

```python
'''
index():返回指定数据所在位置的下标。
语法：列表序列.index(数据，开始位置下标，结束位置下标)

count():统计指定数据在当前列表中出现的次数。

len():访问列表长度，即列表中数据的个数。
'''
#实例：
list1 = ['Tony','Gary','Rose']
print(list1.index('Tony',0,2)) # 1
#注：如果查找的数据不存在则报错。

print(list1.count('Tony')) # 1

print(len(list1)) # 3

```

#### 2，判断数据是否存在

```python
'''
in:判断指定数据在某个列表序列，如果在返回True，否则返回False。

not in:判断指定数据不在某个列表序列，如果不在返回True，否则返回False。

'''

# 实例：
list1 = ['Tony','Gary','Rose']
print('Gary' in list1)  # True
print('Garys' not in list1)  # True

```

#### 3，增加

增加指定数据到列表中。

```python
'''
append():列表结尾追加数据。
语法： 列表序列.append(数据)

extend():列表结尾追加数据，如果数据是一个序列，则将这个序列的数据逐一添加到列表。
语法：列表序列.extend(数据)

insert():指定位置新增数据。
语法：列表序列.insert(位置下标，数据)
'''
# 实例：
list1 = ['Tony','Gary','Rose']
list1.append('Hei')
print(list)
# 结果为：['Tony','Gary','Rose'，'Hei']
# 增加列表数据
list1.append([11,'Siri'])
print(list)
# 结果为：['Tony','Gary','Rose'，'Hei'，[11,'Siri']]

list1 = ['Tony','Gary','Rose']
list1.extend('Hei')
print(list)
# 结果为：['Tony','Gary','Rose'，'H','e','i']
# 增加列表数据
list1.extend([11,'Siri'])
print(list)
# 结果为：['Tony','Gary','Rose'，'Hei'，11,'Siri']

list1 = ['Tony','Gary','Rose']
list1.insert(1,'Hei')
print(list)
# 结果为：['Tony','Hei',Gary','Rose'

```

#### 4，删除

删除列表中的数据。

```python
'''
del 目标   ：删除列表
del 列表[下标]：删除指定数据

列表序列.pop(下标)  :删除指定下标的数据，如果不指定下标，默认删除最后一个数据。无论是按照下标还是删除最后一个。pop函数都会返回这个本删除的数据。

列表序列.remove(数据)：删除列表中指定的数据的第一个匹配项。

clear():清空列表数据，结果为空列表[]。

'''
```

#### 5，修改(排序)

修改列表数据。

```python
'''
列表[下标]：修改指定下标数据

列表.reverse():逆置列表中的数据

列表序列.sort(key=None，reverse=False)：排序
注：reverse表示排序规则：为True降序，为False升序(默认)

'''
list1 = [1,3,2,4,5]
list1.sort()
print(list1)    #结果为：[1,2,3,4,5]
list1.sort(reverse=True)  
print(list1)    #结果为：[5,4,3,2,1]
#注：key值可用于字典某个字段

```

#### 6，列表复制

```python
'''
列表.copy()函数:复制列表中的数据
'''
list1 = [1,2,3]
list2 = list1.copy()
print(list2)  #结果为： [1,2,3]
```

#### 7，列表循环

需求：一次打印列表中的各个数据。

```python
# while
i = 0
while i<len(列表)：
    print(列表[i])
    i+=1

# for
for i in 列表：   #i 临时变量充当列表中某个数据
    print(i)

```

#### 8，列表嵌套

就是指一个列表里面包含了其他的子列表。

```python
list1 = [['小明'，'小红'],['Tom','Gary'],[1,2]]
print(list1[0])   #结果：['小明'，'小红']
print(list1[1][1])   #结果：Gary

# 示例：随机办公室
#把八位老师随机分配到三个办公室
'''
步骤：
1，准备数据，
  1.1 八位老师---列表
  1.2 三个办公室---列表嵌套
2，分配老师到办公室---随机分配
      老师名字写入到办公室。
3，验证是否分配成功
      打印办公室人数和老师对应名字。
'''
import random
# 1.准备数据
teachers = ['a','b','c','d','e','f','g','h']
offices = [[],[],[]]

# 2.分配老师到办公室
for name in teachers：
    #列表追加数据 --append，extend，insert
    #随机一个办公室
    num = random.randint(0,2)
    offices[num].append(name)
# 3.验证信息 --- 打印每个办公室列表数据
int i = 1  #办公室编号
for office in offices:
    # 打印办公室人数
    print(f'办公室{i}的人数是：{len(office)}.\t老师分别是：')
    i += 1
    # 打印老师的名字
    # 每个列表人数不一定 --- 遍历子列表
    for name in office：
        print(name)
      
```

### 5元组

一个元祖可以存储多个数据，元组内的数据是不能修改的。

#### 1，定义元祖

元组特点：定义元祖使用小括号，且都好隔开各个数据，数据可以是不同的数据类型。

```python
# 多个数据元组
t1 = (10,20,30)

# 单个数据元组
t2 = (10,)
print(t1)   # 结果为：(10,20,30)
print(type(t1))   # 结果为：tuple

t3 = (18)
print(type(t3))   # 结果为：int

t4 = ('hello')
print(type(t4))   # 结果为：str

# 注：如果定义的元组只有一个数据，那么这个数据后面也要添加逗号，否则其数据类型为这个数据的数据类型。
```

#### 2，查找操作

元组数据不支持修改，只支持查找。

```python
# 按下表查找数据
tuple1 = ('a','b','c','a')
print(tuple1[0])  # 结果为：a

#index():查找某个数据，如果数据存在返回对应的下标，否则报错。语法和列表，字符串的index方法相同。
print(tuple1.index('a'))  # 结果为：0

#count():统计某个数据在当前元祖出现的次数。
print(tuple1.index('a'))  # 结果为：2

#len():统计元组中数据的个数。
print(len(tuple1))  # 结果为：4

```

#### 3，修改操作

注：元组内的直接数据如果修改则立即报错。

```python
tuple1 = (1,2,3)
tuple1[0] = 5
```

但是如果元组里面有列表，修改列表里面的数据则是支持的。

```python
tuple1 = ('a',[1,2,3],'c','d')
print(tuple1[1])   #访问到列表 [1,2,3]

tuple1[1][0] = 6
print(tuple1)  #结果为：('a',[6,2,3],'c','d')

```

### 6字典

字典是可变类型。

问：当数据顺序发生变化，每个数据的下标也会随之变化，如何确保数据顺序变化前后能使用同一个标准查找数据呢？（列表）

答：字典。字典中的数据是以键值对形式出现，字典数据和数据顺序没有关系，即字典不支持下标，后期无论数据如何变化，只需要按照对应的键的名字查找数据即可。

#### 1，创建字典

字典的特点：

- 符号为大括号
- 数据位键值对形式出现
- 各个键值对之间用逗号隔开

```python
# 有数据字典
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1)
#结果为：{'name':'Gary','age':25,'gender':'男'}

# 空字典
dict2 = {}
dict3 = dict()  #函数创建

# 注：一般称冒号前面的为键(Key),简称k。冒号后面的为值(value),简称v
```

#### 2，增加操作

写法：字典序列[key] = 值

注：如果key存在则修改这个 key对应的值；如果不存在则新增此键值对。

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
dict1['name'] = 'Zhixiao' 
print(dict1) 
#结果为：{'name':'Zhixiao','age':25,'gender':'男'}

dict1['id'] = 1
print(dict1) 
#结果为：{'name':'Zhixiao','age':25,'gender':'男'，'id':1}

```

#### 3，删除操作

del()/del:删除字典或删除字典中指定键值对。

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
del dict1['gender']
print(dict1) 
#结果为：{'name':'Zhixiao','age':25}

del(dict1)  # 等同于 del dict1
print(dict1) 
#结果为：报错，不存在dict1

```

clear():清空字典

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
dict1.clear()
print(dict1) 
#结果为：{}

```

#### 4，修改操作

写法：字典序列[key] = 值

注：如果key存在则修改这个 key对应的值；如果不存在则新增此键值对。

#### 5，查找操作

a.按照key值查找。

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1['name'])  # 结果为：Gary
print(dict1['id']) # 报错
```

注：如果当前查找的key值存在，则返回对应的值；否则报错。

b.函数查找

```python
'''
get() 语法：字典序列.get(key,默认值)

注：如果当前查找的key值不存在则返回第二个参数(默认值)，如果省略第二个参数，则返回None。
'''
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1.get['name'])  # 结果为：Gary
print(dict1.get['id',100]) #  100
print(dict1.get['id']) # None

'''
keys():查找字典中所有的key，返回可迭代对象。
语法：字典序列.keys()
'''
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1.keys())  
# 结果为：dict_keys(['name','age','gender'])

'''
values():查找字典中所有的value，返回可迭代对象。
语法：字典序列.values()
'''
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1.values())  
# 结果为：dict_values(['Gary','25','男'])

'''
items():查找字典中所有的键值对，返回可迭代对象，里面的数据是元组，元组数据1是字典key，元组数据2是key对应的值。
语法：字典序列.items()
'''
dict1 = {'name':'Gary','age':25,'gender':'男'}
print(dict1.items())  # 结果为：dict_items([('name','Gary'),('age',25),('gender','男')])

```

#### 6，字典的循环遍历

a.遍历字典的key

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
for key in dict1.keys():
    print(key)
# name  
# age
# gender
```

b.遍历字典的value

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
for value in dict1.values():
    print(value)
# Gary
# 25 
# 男
```

c.遍历字典的键值对

```python
dict1 = {'name':'Gary','age':25,'gender':'男'}
for item in dict1.items():
    print(item)
# ('name','Gary')
# ('age',25)
# ('gender','男')

# 遍历字典键值对---进行拆包
for key，value in dict1.items():
    print(f'{key} = {value}')
# name = Gary
# age = 25
# gender = 男
```

### 7集合

- 如果有不重复的数据，可以考虑使用集合存储。


- 集合没有顺序，不支持下标操作。

#### 1，创建集合

```python
'''
创建集合使用{}或set()，但是如果要创建空集合只能使用set()，因为{}用来创建空字典。
'''
s1 = {10,20,30,40,50}
print(s1)  #{40,10,50,20,30} 集合展示没有顺序

s2 = {10,30,20,10,30,40,50}
print(s2)  #{40,10,50,20,30}  具有去重功能

s3 = set('abc')
print(s3)  #{'c','a','b'}  作为单一数据出现

s4 = set()
print(s4)  # 结果为：set()
print(type(s4)) #set 集合

s5 = {}
print(type(s5)) #dict 字典

```

#### 2，集合常见操作---增加

因为集合有**去重**功能，所以，当向集合内追加的数据是当前集合已有的数据的话，则不进行任何操作。

```python 
'''
add():追加单一数据。

update():追加的数据是序列。
'''
s1 = {10,20}
s1.add(100)
s1.add(10)
print(s1)    # {100,10,20}
s1.add([1,2,3])
print(s1)    # 报错

s2 = {10,20}
s2.update(100)  # 报错
s2.update([100,200])
s2.update('abc')
print(s2)  # {100,10,'a',20,'c',200,'b'}

```

#### 3，集合常见操作---删除

```python
'''
remove():删除集合中指定数据，如果数据不存在则报错。

discard():删除集合中的指定数据，如果数据不存在也不会报错

pop():随机删除集合中的某个数据，并返回这个数据。
'''
s1 = {10,20,30}
s1.remove(10)
print(s1)  #{30,20}
s1.remove(1)  # error

s2 = {10,20}
s2.discard(10)
print(s2)  #{20}
s2.discard(10)  
print(s2)  #{20}

s3 = {10,20,30}
num = s1.pop()  # 随机删除集合中某个数据
print(num)
print(s3)

```

#### 4，集合常见操作---查找

- in：判断数据在集合序列。存在返回True，否则返回False
- not in：判断数据不在集合序列。不在返回True，否则返回False

```python
s1 = {10,20,30}

print(10 in s1)  #True
print(10 not in s1)   #False
```

### 8公共方法

所有的数据序列基本上都支持的操作。

#### 1.运算符

|  运算符   |   描述    |   支持的容器类型    |
| :----: | :-----: | :----------: |
|   +    |   合并    |  字符串，列表，元组   |
|   *    |   复制    |  字符串，列表，元组   |
|   in   | 元素是否存在  | 字符串，列表，元组，字典 |
| not in | 元素是否不存在 | 字符串，列表，元组，字典 |

1.1，'+'实例

```python
str1 = 'aa'
str2 = 'bb'

list1 = [1,2]
list2 = [10,20]

t1 = (1,2)
t2 = (10,20)

dict1 = {'name':'Song'}
dict2 = {'age':30}

# +：合并
print(str1 + str2)  #  aabb
print(list1 + list2)  #  [1,2,10,20]
print(t1 + t2)  #  (1,2,10,20)
print(dict1 + dict2)  #  error 字典不支持合并运算

```

1.2，'*'实例

```python
str1 = 'a'
list1 = ['hello']
t1 = ('world',)

print(str1 * 5)  #  aaaaa
print(list1 * 3)  #  ['hello','hello','hello']
print(t1 * 3)  #  ('world','world','world')
```

1.3，'in'实例，'not in'实例

```python
str1 = 'abc'
list1 = [10,20,30]
t1 = (1,2,3,4,5)
dict1 = {'name':'Tom','age':20}

print('a' in str1)  #  True
print('a' not in str1)  #  False
print(10 in list1)  #  True
print(1 in t1)  #  True
print('name' in dict1)  #  True
print('name' in dict1.keys())  #  True
print('name' in dict1.values())  #  False
```

#### 2.公共方法

| 函数                    | 描述                                       |
| --------------------- | ---------------------------------------- |
| len()                 | 计算容器中元素的个数                               |
| del或del()             | 删除                                       |
| max()                 | 返回容器中元素最大值                               |
| min()                 | 返回容器中元素最小值                               |
| range(start,end,step) | 生成从start到end的数字，步长为step，供for循环使用         |
| enumerate()           | 函数用于将一个可遍历的数据对象(如列表，元组或字符串)组合为一个索引序列，同时列出数据和数据下标，一般用在for循环当中。 |

2.1 len()

```python
str1 = 'abc'
list1 = [10,20,30]
t1 = (1,2,3,4,5)
dict1 = {'name':'Tom','age':20}

print(len(str1))  #  3
print(len(list1))  #  3
print(len(t1))  #  5
print(len(dict1))  #  2
```

2.2 del或del()

```python
str1 = 'abc'
list1 = [10,20,30]
t1 = (1,2,3,4,5)
dict1 = {'name':'Tom','age':20}

del str1
print(str1)  #  error  str1找不到
del(list1[0])
print(list1)  #  [20,30]
del(list1)
print(list1)  #  error  list1找不到
del(t1)
print(t1)  #  error  t1找不到
del(dict1['name'])
print(dict1)  #  {'age':20}
del(dict1)
print(dict1)  #  error  dict1找不到

```

2.3 max()和min()

```python
str1 = 'abc'
list1 = [10,20,30]
t1 = (1,2,3,4,5)

print(max(str1))  #  c
print(min(str1))  #  a
print(max(list1))  #  30
print(min(list1))  #  10
print(max(t1))  #  5
print(min(t1))  #  1

```

2.4  range(start,end,step)

range()生成序列不包括end数字。返回一个可迭代对象。

```python
print(range(1,10,1))  #range(1,10)
for i in rang(1,10,1):  
    print(i)        # 1 2 3 4 5 6 7 8 9                       
for i in rang(1,10,2):  
    print(i)        # 1 3 5 7 9

for i in rang(10):  
    print(i)        # 0 1 2 3 4 5 6 7 8 9 
    
# 如果不写开始，默认从0开始
# 如果不写步长，默认步长为1
```

2.5 enumerate()

- start参数用来设置遍历数据的下标的起始值，默认为0


- enumerate()：返回对象是元组，元组第一个数据是元迭代对象数据对应的下标，第二个数据是原迭代对象的数据

```python
# 语法：enumerate(可遍历对象，start=0)

list1 = ['a','b','c','d','e']
for i in enumerate(list1):
    print(i)
     #(0,'a') (1,'b') (2,'c') (3,'d') (4,'e')
for index,char in enumerate(list1,start=1):
    print(f'下标是{index},对应字符是{char}')
    #下标是1,对应字符是a
    # ......
```

#### 3.容器类型转换

3.1 tuple()

作用：将某个序列转换成元组

```python
list1 = [1,2,3,4,5]
s1 = {10,20,30,40,50}

print(tuple(list1))  # （1,2,3,4,5）
print(tuple(s1))     # （40,10,50,20,30）
```

3.2 list()

作用：将某个序列转换成列表

```python
t1 = ('a','b','c','d','e')
s1 = {10,20,30,40,50}

print(list(t1))  #  ['a','b','c','d','e']
print(list(s1))  #  [40,10,50,20,30]
```

3.3 set()

作用：将某个序列转换成集合。集合无顺序，且有去重功能。

```python
t1 = ('a','b','c','d','e')
list1 = [1,2,3,2,4,5]

print(set(t1))      #  {'c','e','d','a','b'}
print(set(list1))   #  {1,2,3,4,5}
```

3.4 eval()

定义：eval是Python的一个内置函数

作用：返回传入字符串的表达式的结果。

示例：想象一下变量赋值时，将等号右边的表达式写成字符串的格式，将这个字符串作为eval的参数，eval的返回值就是这个表达式的结果。

也可以这样来理解：eval()函数就是实现list、dict、tuple、与str之间的转化。

```python
'''
语法:   eval(expression[, globals[, locals]])
参数:
1,expression – 表达式。
2,globals – 变量作用域，全局命名空间，如果被提供，则必须是一个字典对象。
3,locals – 变量作用域，局部命名空间，如果被提供，可以是任何映射对象。
'''
x = 7
y = eval( '3 * x' ) # 21
z = eval('pow(2,2)')  # 4

a = "[[1,2], [3,4], [5,6], [7,8], [9,0]]"
print(type(a))
b = eval(a)
print(type(b))
print(b)
'''
结果为：
<class 'str'>
<class 'list'>
[[1, 2], [3, 4], [5, 6], [7, 8], [9, 0]]
'''
```

#### 4.列表推导式 (又叫列表生成式)

作用：用一个表达式创建一个有规律的列表或控制一个有规律的列表。

4.1实例：创建一个0-10的列表

while循环实现

```python
# 1. 准备一个空列表
list1 = []

# 2. 书写循环，依次追加数字到空列表list1中
i = 0
while i < 10:
    list1.append(i)
    i += 1
    
print(list1)
```

for循环实现

```python
list1 = []
for i in range(10):
    list1.append(i)
    
print(list1)
```

列表推导式实现

```python
# [],返回一个列表。
# 左边i，列表里面要返回的数据。右边i，range里面的值。
list1 = [i for i in range(10)]
print(list1)
```

4.2 带 if的列表推导式

实例：创建0-10的偶数列表

```python
# 方法一：range()步长实现
list1 = [i for i in range(0,10,2)]
print(list1)

# 方法二：if实现
list2 = [i for i in range(10) if i%2==0]
print(list2)
```

4.3 多个for循环实现列表推导式

实例：创建以下列表：

```python
[(1,0),(1,1),(1,2),(2,0),(2,1),(2,2),]
```

代码如下：

```python
list1 = [(i,j) for i in range(1,3) for j in range(3)]
print(list1)
```

#### 5 字典推导式

思考：如果有如下两个列表：

```python
list1 = ['name','age','gender']
list2 = ['Gary',25,'男']
```

如何快速合并为一个字典？

答：字典推导式

字典推导式作用：快速合并列表为字典或提取字典中目标数据。

5.1 合并列表为字典

1. 创建一个字典：字典key是1-5数字，value是这个 数字的2次方。

   ```python
   dict = {i：i**2 for i in range(1,5)}
   print(dict)  # {1:1,2:4,3:9,4:16}
   ```

2. 将两个列表合并为一个字典

   ```python
   list1 = ['name','age','gender']
   list2 = ['Gary',25,'男']

   dict1 = {list1[i]: list2[i] for i in range(len(list1))}
   print(dict1)

   # 总结：
   # 1. 如果两个列表的数据长度相同，len统计任何一个列表的长度都可以。
   # 2 .如果不同，len统计长的列表会发生列表越界错误，统计短的，长的一部分就没有收集到。
   ```

3. 提取字典中目标数据

   ```python
   counts = {'MBP':268,'HP':125,'DELL':201,'Lenovo':199,'acer':99}
   
   # 需求：提取上述电脑数量大于等于200的字典数据
   count1 = {key: value for key,value in counts.items() if value >= 200}
   print(count1) #{'MBP':268,'DELL':201}
   ```

#### 6 集合推导式 (集合具有数据重功能)

6.1 实例：创建一个集合，数据为下方列表的2次方。

```python
list1 = [1,1,2]
```

代码如下：

```python
list1 = [1,1,2]
set1 = {i ** 2 for i in list1}
print(set1)  {1,4}
```

## 三，函数

### 1函数介绍

#### 1.1 函数作用

作用：在开发过程中，可以更高效的实现代码的重用。

定义：函数就是将一段具有独立功能的代码块整合到一个整体并命 名，在需要的位置调用这个名称即可完成对应的需求。

#### 1.2 函数的使用步骤

a. 定义函数

```python
def 函数名(参数)：
    代码1
    代码2
    ......
```

b. 调用函数

1. 不同的需求，参数可有可无。
2. 在python中，函数必须先定义后调用。

```python
函数名(参数)
```

**实例**：复现ATM取钱功能

1.搭建整体框架 (复现需求)

```python
print('密码正确登录成功')

# 显示"选择功能"界面

print('查询余额完毕')

# 显示"选择功能"界面

print('取了1000元')

# 显示"选择功能"界面
```

2.确定"选择功能"界面内容

```python
print('查询余额')
print('存款')
print('取款')
```

3.封装"选择功能"

- 先定义，后调用。

```python
# 封装ATM机功能选项---定义函数
def select_func():
    print('---请选择功能---')
    print('查询余额')
	print('存款')	
	print('取款')
    print('---请选择功能---')
```

4.调用函数

在需要显示"选择功能"函数的位置调用函数。

```python
print('密码正确登录成功')
# 显示"选择功能"界面---调用函数
select_func()
print('查询余额完毕')
# 显示"选择功能"界面---调用函数
select_func()
print('取了1000元')
# 显示"选择功能"界面---调用函数
select_func()
```

#### 1.3 函数的注意事项

- 先定义，后调用。
- 如果没有调用函数，函数里面的代码不会执行。
- 函数的执行流程，当调用函数的时候，解释器会回到定义函数的地方执行下方缩进的代码。当这些代码执行完，会回到调用函数的地方，向下执行。

### 2函数参数

定义：函数调用时指定的数字和定义函数时接收的数字既是函数的参数。

```python
# 定义函数时同时定义了接收用户数据的参数a和b，a和b是形参
def add_func(a,b):
    result = a+b
    print(result)
    
# 调用函数时传入了真实的数据10和20，真实数据是实参
add_func(10,20)
```

注：定义函数有几个参数，调用函数时应该传入几个数据。

#### 2.1 位置参数

定义：调用函数时根据函数定义的参数位置来传递参数。

```python
def local_func(name,age,gender):
    print(f'您的名字是{name},年龄是{age},性别是{gender}')
    
local_func('Tom',20,'男')
```

注：传递和定义参数的顺序及个数必须一致。

#### 2.2 关键字参数

定义：函数调用，通过"键=值"形式加以指定。可以让函数更加清晰，容易使用，同时也清除了参数的顺序需求。

```python
def key_func(name,age,gender):
    print(f'您的名字是{name},年龄是{age},性别是{gender}')
    
key_func('Tom',age=20,gender='男')
key_func('Gary',gender='男',age=20)
```

注：函数调用时，如果有位置参数时，位置参数必须在关键字参数的前面，但关键字参数之间不存在先后顺序。

#### 2.3 缺省参数

定义：也叫默认参数，用于定义函数，为参数提供默认值，调用函数时可不传该默认参数的值（注意：所有位置参数必须出现在默认参数前，包括函数定义和调用）

```python
def lack_func(name,age,gender='男'):
    print(f'您的名字是{name},年龄是{age},性别是{gender}')
    
lack_func('Tom',20)
lack_func('Lily',18，'女')
```

注：函数调用时，如果为缺省参数传值则修改默认参数值；否则使用这个默认值。

#### 2.4 不定长参数

定义：也叫可变参数，用于不确定调用的时候会传递多少个参数(不传参也可以)的场景。此时，可用包裹 (packing)位置参数，或者包裹关键字参数，来进行参数传递，会显得非常方便。

- 包裹位置传递

```python
def user_func(*args):   # *必须有，args可任意变换
    print(args)
    
user_func('Tom')   # ('Tom',)
user_func('Lily',18) # ('Lily',18) 
```

注：传递的所有参数都会被args变量收集，它会根据传进参数的位置合并为一个元组(tuple)，args是元组类型，这就是包裹位置参数。

- 包裹关键字参数

```python
def key_func(**kwargs):  # **必须有，kwargs变量名
    print(kwargs)
    
key_func(name='Tom',age=18,id=66)   # {'name':'Tom','age':18,'id':66}
```

注：收集所有的关键字参数，返回一个字典。



综上：无论是包裹位置传递还是包裹关键字传递，都是一个组包 (收集零散参数，返回一个整体)的过程。

### 3函数返回值

- 当用户需要返回结果时，函数将需要的结果返回给用户使用返回值。
- 返回值，返回到函数调用的地方。

#### 3.1 return 特点

- 负责函数返回值
- 退回到当前调用函数的位置，return下方的代码都不执行。

应用：制作一个计算器，计算任意两个数字之和，并保存结果。

```python
def sum_func(a,b):
    return a+b
  
# 用result变量保存函数返回值
result = sum_func(59,1)
print(result) 
```

#### 3.2 多个返回值

注意：

- return a，b写法，返回多个数据的时候，默认是元组类型。
- return后面可以连接列表，元组或字典，以返回多个值

```python
def return_num():
    return 1,2   # 返回的是一个元组
  
result = return_num()
print(result)  # (1,2)

# 返回列表，元组或字典，返回多个值
def return_num2():
    list1 = [1,2,3]
    s1 = (1,2,3)
    dict1 = {'name':'Tom','age':18}
    return list1
    #return [1,2,3]
    #return (1,2,3)
    #return {'name':'Tom','age':18}
  
result2 = return_num2()
print(result2)  # [1,2,3]
```



### 4定义函数说明文档

- help(函数名)：查看函数的说明文档。


- 函数说明文档信息，只能在第一行书写。

```python
# 定义函数的说明文档
def 函数名(参数)：
    """ 函数说明文档信息 """  
    代码1
    ...

# 实例
def sum(a,b):
    """求和函数"""
    return a+b
  
help(sum)  # 结果为：sum(a,b) 求和函数

# 函数说明文档的高级使用
def sum(a,b):
    """
    函数说明文档信息1
    ：param a: 说明信息2
    ：param a: 说明信息3
    ：return:  说明信息4
    """
    return a+b
  
help(sum) 
'''结果为：sum(a,b) 
    函数说明文档信息1
    ：param a: 说明信息2
    ：param a: 说明信息3
    ：return:  说明信息4
'''  
```

### 5函数嵌套

定义：函数嵌套调用就是指在一个函数内部又调用了另外一个函数。

```python
def testB():
    print('---testB  start---')
    print('执行过程')
    print('---testB  end---')
    
def testA():
    print('---testA  start---')
    testB()
    print('---testA  end---')
    
testA()
```

### 6变量

变量作用域：指的是变量生效的范围，主要分为两类，局部变量和全局变量。

#### 6.1 局部变量

定义：局部变量是定义在函数体内部的变量，即只在函数体内部生效。

作用：在函数体内部，临时保存数据，即当函数调用完成后，则销毁局部变量。

```python
def testA():
    a = 100
    print(a)
    
testA()  # 100
print(a)  # error: name 'a' is not defined

# 注：变量a是定义在testA函数内部的变量，在函数外部访问则立即报错。
```

#### 6.2 全局变量

定义：全局变量指的是在函数体内，外都能生效的变量。定义在函数体外。

```python
# 定义全局变量
a = 100
b = 10

def testA():
    print(a) # 访问全局变量a，并打印变量a存储的数据
    
def testB():
    a = 200
    print(a) # 此时a为局部变量
    
testA()  # 100
testB()  # 200
print(f'全局变量为：a={a}')  # 全局变量a = 100

# 在函数体内部修改全局变量，加关键字global。
def testC():
    global b = 20
    print(b) # 此时b为全局变量
    
testC()
print(b)  # 20
```

总结：

1. 如果在函数体内直接修改全局变量对应字符的值，此时变量为局部变量。				   
2. 函数体内部修改全局变量，需使用关键字`global`声明变量，再赋值。

### 7拆包，交换变量

#### 7.1 拆包

- 拆包：元组

```python
def return_num():
    return 100,200
  
num1,num2 = return_num()
print(num1)  # 100
print(num1)  # 200
result = return_num()
print(result)  # (100,200)
```

- 拆包：字典

```python
dict1 = {'name':'gary','age':18}
a,b = dict1

# 对字典进行过拆包，取出来的是字典的key
print(a)  # name
print(b)  # age

print(dict1[a])  # gary
print(dict1[b])  # 18
```

#### 7.2 交换变量

场景：有变量a=10,b=20，交换两个变量的值。

- 方法一：借助第三个变量存储数据。

```python
# 定义中间变量
c = 0

# 将a的数据存储到c
c = a

# 将b的值赋值给a，再将a的值给b
a = b
b = c

print(a)  # 20
print(b)  # 10
```

- 方法二

```python
a,b = 1,2   # a=1,b=2
a,b = b,a
print(a)  # 2
print(b)  # 1
```

### 8引用

#### 8.1 了解引用

看python，值是靠引用来传递来的。

我们可以用id ()来判断两个变量是否为同一个值的引用。我们可以将id值理解为那块内存的地址标识。

```python
# int类型---不可变类型
a = 1
b = a
print(b) # 1

print(id(a))  # 14070804326857
print(id(b))  # 14070804326857

a = 2
print(b) # 1   说明int类型为不可变类型 

print(id(a))  # 14070804326859  此时得到的是数据2的内存地址
print(id(b))  # 14070804326857

# 列表---可变类型
aa = [10,20]
bb = aa

print(id(aa))  # 24070805784216
print(id(bb))  # 24070805784216

aa.append(30)
print(bb)  # [10,20,30]列表为可变类型

print(id(aa))  # 24070805784216
print(id(bb))  # 24070805784216
```

#### 8.2 引用当做实参

```python
def test1(a):
    print(a)
    print(id(a))
    a += a
    print(a)
    print(id(a))
    
# int：计算前后id值不同
b = 100
test1(b)

# 列表：计算前后id值相同
c = [11,22]
test1(c)

'''结果：
100
140731767570352
200
140731767573552
[11, 22]
2017164157512
[11, 22, 11, 22]
2017164157512
'''
```

### 9递归函数

特点：

- 函数内部自己调用自己
- 必须有出口

```python
#  递归实现，10以内数字累加和
def sum_func(a):
    # 如果为1，直接返回1---出口
    if a==1:
        return 1
    # 如果不为1，重复执行累加，并返回结果
    return a+sum_func(a-1)
  
sum = sum_func(10)
print(sum)  #55
```

### 10匿名函数(lambda表达式)

#### 10.1 lambda表达式(语法和实例)

如果一个函数有一个返回值，并且只有一句代码，可以使用lambda简化。

**语法**：

```python
lambda 参数列表 ： 表达式
```

注意：

- lambda表达式的参数可有可无，函数的参数在lambda表达式中完全适用。
- lambda表达式能接收任何数量的参数，但只能返回一个表达式的值。

```python
#  快速入门
def fn1():
    return 200
    
print(fn1)     #内存地址
print(fn1())   # 200

# lambda表达式
fn2 = lambda: 100
print(fn2)      #内存地址
print(fn2())    #100

# 注：直接打印lambda表达式，输出的是此lambda的内存地址
```

实例： 计算a+b

函数实现

```python
def add(a,b):
    return a+b
  
result = add(1,2)
print(result)
```

lambda实现

```python
fn1 = lambda a,b: a+b
print(fn1(1,2))
#等同于
print((lambda a,b: a+b)(1,2))
```

#### 10.2 lambda的参数形式 

10.2.1 无参数

```python
print((lambda: 100)())
#等同于
fn1 = lambda : 100
print(fn1())  
```

10.2.2 一个参数

```python
print((lambda a: a)('hello world'))
#等同于
fn1 = lambda a: a
print(fn1('hello world'))  
```

10.2.3 默认参数

```python
print((lambda a,b,c=100: a+b+c)(10,20))
#等同于
fn1 = lambda a,b,c=100: a+b+c
print(fn1(10,20))
```

10.2.4 可变参数：**args

```python
print((lambda *args: args)(10,20,30))
#等同于
fn1 = lambda *args: args
print(fn1(10,20,30))

# 注：这里的可变参数传入到lambda之后，返回值为元组。
```

10.2.5 可变参数：**kwargs

```python
print((lambda **kwargs: kwargs)(name='Tom',age=20))
#等同于
fn1 = lambda **kwargs: kwargs
print(fn1(name='Tom',age=20))

# 注：这里的可变参数传入到lambda之后，返回值为字典。
```

#### 10.3 lambda的应用

10.3.1带判断的lambda

```python
print((lambda a,b: a if a>b else b)(100,20))
```

10.3.2 列表数据按字典key的值排序

```python
students = [
  {'name':'Tom','age':20},
  {'name':'Lily','age':18},
  {'name':'Gary','age':25},
]

# 按name值升序排列
students.sort(key=lambda x:x['name'])
print(students)

# 按name值降序排列
students.sort(key=lambda x:x['name'],reverse=True)
print(students)

# 按age值升序排列
students.sort(key=lambda x:x['age'])
print(students)
```

### 11高级函数

把一个函数作为另外一个函数的参数传入。

#### 11.1 函数介绍

abs()函数：完成对数字求绝对值计算。

round()函数：完成对数字的四舍五入计算

```python
def add(a,b,f):
    return f(a)+f(b)
  
result = add(-1,-2,abs)  # 函数作为参数传递
print(result)  # 3

# 函数式编程大量使用函数，减少代码的重复，因此程序较短，开发速度较快。
```

#### 11.2内置高阶函数

11.2.1 map()函数

map(func, lst)，将传入的函数变量func作用到list变量的每个元素中，并将结果组成新的列表(Python2)/(Python3)返回迭代器。

```python
# 计算list1序列中各个数字的2次方
list1 = [1,2,3,4,5]

def func(x):
    return x ** 2
  
result = map(func,list1)

print(result) # python3以上版本，返回迭代器       <map object at 0x000002E156042518>
print(list(result))  # [1,4,9,16,25]
```

11.2.2 reduce()函数

reduce(func, lst)，其中func必须有两个参数。每次func计算的结果继续和序列的下一个元素做累积计算。

注：reduce()传入的参数func必须接受两个参数。

```python
# 计算list1序列中的各个数字的累加和。
import functools

list1 = [1,2,3,4,5]

def func(a,b):
    return a + b
  
result = functools.reduce(func,list1)

print(result)  # 15
```

11.2.3 filter()函数

filter(func, lst)函数用于过滤序列，过滤掉不符合条件的元素，返回一个filter对象。如果要转换为列表，可以使用list()来转换。

```python
list1 = [1,2,3,4,5,6,7,8,9,10]

def func(x):
    return x % 2 == 0
  
result = filter(func, list1)

print(result)#<filter object at 0x0000024DDB682518>
print(list(result)) # [2, 4, 6, 8, 10]
```

## 四，文件

### 4.1 文件操作的作用

文件：计算机中出现的都可认为是文件。

文件操作的作用：把一些内容(数据)存储存放起来，可以让程序下一次执行的时候直接使用，而不必重新制作一份，省时省力。

**写入文件的类型：字符串str，bytes类型。**

### 4.2 文件的基本操作

#### 4.2.1 文件操作步骤

1. 打开文件
2. 读写等操作
3. 关闭文件

注：可以只打开和关闭文件，不进行任何读写操作。

#### 4.2.2 具体操作

##### a，打开操作

在python，使用open函数，可以打开一个已经存在的文件，或者创建一个新文件，语法如下：

```python
open(name,mode)

'''
name：是要打开的目标文件名的字符串(可以包含文件所在的具体路径)。
mode：设置打开文件的模式(访问模式)：只读，写入，追加等。
'''
```

**访问模式**

|  模式   | 描述                                       |
| :---: | ---------------------------------------- |
| **r** | 以只读方式打开文件。文件的指针将会放在文件的**开头**，这是默认模式。     |
|  rb   | 以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头，这是默认模式。   |
|  r+   | 打开一个文件用于读写，文件指针将会放在文件的开头。                |
|  rb+  | 以二进制格式打开一个文件用于读写，文件指针将会放在文件的开头。          |
| **w** | 打开一个文件只用于写入。如果该文件已存在则打开文件，并从头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
|  wb   | 以二进制格式打开一个文件只用于写入。如果该文件已存在则打开文件，并从头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
|  w+   | 打开一个文件用于读写。如果该文件已存在则打开文件，并从头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
|  wb+  | 以二进制格式打开一个文件只用于读写。如果该文件已存在则打开文件，并从头开始编辑，即原有内容会被删除。如果该文件不存在，创建新文件。 |
| **a** | 打开一个文件用于追加，如果该文件已存在，文件指针将会放在文件的**结尾**。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。 |

- r模式：如果文件不存在，报错。
- w和a模式：如果文件不存在，创建该文件。如果文件存在，w模式覆盖原文件内容，a模式数据，追加在原内容之后。

```python
f = open('name','mode')

# f为name文件中对应的文件对象。
```

##### b，读写操作

读操作：

read()

```python
文件对象.read(num)

# num表示要从文件中读取的数据长度(单位是字节)，如果没有传入num，那么就表示读取文件中所有数据。
```

readlines()

readlines可以按照行的方式把整个文件中的内容进行一次性读取，并且返回的是一个列表，其中每一行的数据为一个元素。

```python
f = open('test.txt')    # 默认为只读模式r
content = f.readlines()

# ['hello world\n','wasd'] 
print(content)

# 关闭文件
f.close()
```

readline()

readline() 一次读取一行内容

```python
f = open('test.txt')
 
content = f.readline()
print(f'第一行：{content}')

content = f.readline()
print(f'第二行：{content}')

# 关闭文件
f.close()
```

写操作：

```python
# 文件对象.write(数据)

f = open('文件名','w')
f.write('hello world')
f.close()
```

##### c，关闭操作

```python
文件对象.close()
```

```python
# 具体实例

# 1，打开 open()
f = open('test.txt','w')   

# 2，读写操作 write(),read()
f.write('hello')


# 3，关闭 close()
f.close()
```

#### 4.2.3 seek()函数

作用：用来移动文件指针

```python
文件对象.seek(偏移量，起始位置)
```

起始位置：

- 0：文件开头
- 1：当前位置
- 2：文件末尾

### 4.3 文件备份

场景：用户输入当前目录下任意文件名，程序完成对该文件的备份功能(备份文件名为xx[备份]后缀，例如：test[备份].txt)

#### 4.3.1 步骤

1. 接受用户输入的文件名
2. 规划备份文件名
3. 备份文件写入数据

#### 4.3.2 代码实现

1. 接受用户输入的目标文件名

```python
old_name = input('请输入您要备份的文件名：')
```

2. 规划备份文件名

   2.1 提取目标文件后缀

   2.2 组织备份的文件名，xx[备份]后缀

```python
# 2.1 提取文件后缀点的下标
index = old_name.rfind('.')

# print(index)   # 后缀中.的下标

# print(old_name[:index])   # 源文件名（无后缀）

# 2.2 组织新文件名 旧文件名+[备份]+后缀
new_name = old_name[:index]+'[备份]'+old_name[index:]

# 打印新文件名（带后缀）
print(new_name)
```

3. 备份文件写入数据

   3.1 打开源文件和备份文件

   3.2 将源文件数据写入备份文件

   3.3 关闭文件

```python
# 3.1 打开文件
old_f = open(old_name,'rb')
new_f = open(new_name,'wb')

# 3.2 将源文件数据写入备份文件
while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)
    
# 3.3 关闭文件
old_f.close()
new_f.close()
```

4. 限制使有效文件名输入

```python
# 添加判断限制只有有效文件名输入
old_name = input('请输入您要备份的文件名：')

index = old_name.rfind('.')

if index > 0:
    postfix = old_name[index:]

new_name = old_name[:index]+'[备份]'+postfix
# 如果为.txt输入，则会报错，postfix未定义

print(new_name)

old_f = open(old_name,'rb')
new_f = open(new_name,'wb')

while True:
    con = old_f.read(1024)
    if len(con) == 0:
        break
    new_f.write(con)
    
old_f.close()
new_f.close()
```

### 4.4 文件和文件夹的操作

在Python中文件和文件夹的操作要借助os模块里面的相关功能，具体步骤如下：

```python
#  1.导入os模块
import os

#  2.使用os模块相关功能
os.函数名()
```

#### 4.4.1 文件操作

a，文件重命名

```python
os.rename('目标文件名'，'新文件名')
# 既可以文件重命名，也可以文件夹重命名
```

b，删除文件

```python
os.remove('目标文件名')
```

#### 4.4.2 文件夹操作

a，创建文件夹

```python
os.mkdir('文件夹名字')
```

b，删除文件夹

```python
os.rmdir('文件夹名字')
```

c，获取当前目录

```python
os.getcwd()
# 返回当前文件所在的目录文件
```

d，改变默认目录

```python
os.chdir('目录')
```

e，获取目录列表

```python
os.listdir('目录')
# 获取某个文件夹下所有文件，返回一个列表

os.listdir()
# 返回当前文件夹下所有文件，返回一个列表
```

#### 4.4.3 实例

需求：批量修改文件名，即可添加指定字符串，又能删除指定字符串。

步骤：

1. 设置添加删除字符串的标识
2. 获取指定目录的所有文件
3. 将原有文件名添加/删除指定字符串，构造新名字
4. os.rename()重命名

代码：

```python
import os

# 设置重命名标识：如果为1则添加指定字符，flag取值为2则删除指定字符
flag = 1

# 获取指定目录
dir_name = './'

# 获取指定目录的文件列表
file_list = os.listdir(dir_name)
print(file_list)

# 遍历文件列表中的数据
for name in file_list:
    # 添加指定字符
    if flag == 1:
        new_name = 'Python-'+name
    # 删除指定字符
    elif flag == 2：
        num = len('Python_')
        new_name = name[num:]
        
    # 打印新文件名，测试程序正确性
    print(new_name)
    
    # 重命名
    os.rename(dir_name+name,dir_name+new_name)
```

## 五，面向对象

面向对象是一种抽象化的编程思想，很多编程语言中都有的一种思想。

总结：面向对象就是将编程当成一个事物，对外界来说，事物是直接使用，不用去管他内部的情况。而编程就是设置事物能够做什么事。

### 5.1 类和对象

面向对象编程中，有两个重要的组成部分：类和对象。

类和对象的关系：用类去创建 (实例化)一个对象。

#### 5.1.1 类和对象基本概念

a，类

类是对一系列具有相同特征和行为的事物的统称，是一个抽象的概念，不是真实存在的事物。

- 特征即是属性
- 行为即是方法

类比如说是制造洗衣机时要用到的图纸，也就是说类是用来创建对象。

b，对象

对象是类创建出来的真实存在的事物。例如：洗衣机

注：开发中，先有类，再有对象。

#### 5.1.2 面向对象实现方法

##### a，定义类

Python2中类分为：经典类 和 新式类

```python
#  语法
class 类名():
    代码
    ......
    
# 注：类名要满足标识符命名规则，同时遵守大驼峰命名习惯
```

实践：

```python
class Washer():
    def wash(self):
        print('洗衣服')
```

- 拓展1：经典类或旧式类

不由任意内置类型派生出来的类，称之为经典类。

2.0版本解释器默认按照经典类处理执行。

```python
class 类名():
    代码
    ......
```

- 拓展2：新式类

  3.0版本解释器默认按照新式类处理执行。

```python
class 类名(object):
  代码
```

##### b，创建对象

对象又名实例。

```python
# 语法
对象名 = 类名()

# 实例：
# 创建对象
haier1 = Washer()

print(haier1)
# <_main__.Washer object at 0x0000018B7B2242240>

# haier对象调用实例方法
haier1.wash()
```

##### c，self

self指的是调用该函数的对象。

```python
# 1. 定义类
class Washer():
    def wash(self):
        print('洗衣服')
        print(self)
        # <_main__.Washer object at 0x0000018B7B2242240>
      
# 2. 创建对象
haier1 = Washer()
print(haier1)
# <_main__.Washer object at 0x0000018B7B2242240>

# haier对象调用实例方法
haier1.wash()  
# 洗衣服
# <_main__.Washer object at 0x0000018B7B2242240>

haier2 = Washer()
print(haier2)
# <_main__.Washer object at 0x0000018B7B2224880>

haier2.wash()
# 洗衣服
# <_main__.Washer object at 0x0000018B7B2224880>
```

注：

- 一个类可以创建多个对象。
- 创建的多个对象的内存地址不同。

#### 5.1.3 添加和获取对象属性

- 属性即是特征，如：洗衣机的宽度，高度，重量...
- 对象属性既可以在类外面添加和获取，也能在类里面添加和获取。

##### a，类外面添加对象属性

```python
# 语法
对象名.属性名 = 值

# 实例
haier.width = 50
haier.height = 150
```

##### b，类外面获取对象属性

```python
# 语法
对象名.属性名 

# 实例
print(f'宽度为：{haier.width}')
print(f'高度为：{haier.height}')
```

##### c，类里面获取对象属性

```python
# 语法
self.属性名

# 实例
class Washer():
    def wash(self):
        print('可以洗衣服')
    def output_info(self):
        # 类里面获取实例属性
        print(f'洗衣机宽度为：{self.width}')
		print(f'洗衣机高度为：{self.height}')
        
# 创建对象
haier1 = Washer()
haier2 = Washer()
# 添加实例属性
haier1.width = 50
haier1.height = 150

# 对象调用实例方法
haier1.output_info()
haier2.output_info() 
# haier2.output_info()报错，没有对应的属性，width，height

```

### 5.2 魔法方法

在Python中，__ xx__()的函数叫做魔法方法，指的是具有特殊功能的函数。

##### a，__ init__()

作用：初始化对象。

```python
class Washer():
    # 定义__ init__，添加实例属性
    def __ init__(self):
        # 添加实例属性
        self.width = 50
        self.height = 150
        
    def print_info(self):
        # 类里面调用实例属性
        print(f'洗衣机宽度为：{self.width}')
		print(f'洗衣机高度为：{self.height}')
        
haier1 = Washer()
haier1.print_info()
```

注：

- __ init__()方法，在创建一个对象时默认被调用，不需要手动调用
- __ init__(self)中的self参数，不需要开发者传递，Python解释器会自动把当前对象引用传递过去。

##### b，带参数的__ init__()

思考：一个类可以创建多个对象，如何对不同的对象设置不同的初始化属性呢？

答：传参数

```python
class Washer():
    # 定义__ init__，添加实例属性
    def __ init__(self,width,height):
        # 添加实例属性
        self.width = width
        self.height = height
        
    def print_info(self):
        print(f'洗衣机的宽度是{self.width}')
        print(f'洗衣机的高度是{self.height}')
        
haier1 = Washer(10,20)
haier1.print_info()

haier2 = Washer(50,60)
haier2.print_info()
```

##### c，__ str__()

当使用print输出对象的时候，默认打印对象的内存地址。如果类定义了__ str__方法，那么就会打印这个方法中return的数据。

```python
class Washer():
    def __ init__(self,width,height):
        self.width = width
        self.height = height
        
    def __ str__(self):
        return '解释说明'
        
haier1 = Washer(10,20)
print(haier1)

```

##### d，__ del__()

当删除对象时，Python解释器也会默认调用__ del__()方法。

```python
class Washer():
    def __ init__(self,width,height):
        self.width = width
        self.height = height
        
    def __del__(self):
        print('对象已经被删除')
        
haier1 = Washer(10,20)

# 调用__del__()方法一：当程序运行结束，释放内存，自然调用
# 对象已经被删除
# 调用__del__()方法二：删除对象时调用
del haier1
# 对象已经被删除
```

##### e，__ dict __

类对象和实例对象都有的属性。

作用：收集类对象或实例对象中属性和方法以及对应的值，从而返回一个字典。

```python
class A(object):
    a = 0
    
    def __init__(self):
        self.b = 1
        
aa = A()
# 返回类内部所有属性和方法对应的字典
print(A.__dict__)
# 返回实例属性和值组成的字典
print(aa.__dict__)
```

### 5.3 继承

Pyhton面向对象的继承指的是多个类之间的所属关系，即子类默认继承父类的所有属性和方法，具体如下：

```python
# 父类A
class A(object):
    def __init__(self):
        self.num = 1
        
    def info_print(self):
        print(self.num)
       
# 子类B
class B(A):
    pass
  
result = B()
result.info_print() # 1
```

- 在Python中，所有类默认继承object类，object类是顶级类或基类：其他子类叫做派生类。


##### a，单继承

故事主线：一个白胡子老爷爷，修炼内功多年，如今要把这套内功修为传给他唯一的徒弟。

分析：徒弟是不是要继承师傅的所有修为？

```python
# 1. 师傅类
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
# 2. 徒弟类
class Prentice(Master):
    pass
  
# 3. 创建对象tudi
tudi = Prentice()
# 4. 对象访问实例属性
print(tudi.kongfu)
# 5. 对象访问实例方法
tudi.make_kongfu()
```

所谓单继承：一个师傅类单一继承给一个徒弟类，这种单一的继承关系称为单继承。

##### b，多继承

故事主线：tudi是个热爱练功的好孩子，想学习更多功法，于是，准备上青云宗，入门学习更多功法。

所谓多继承：意思就是一个类同时继承了多个父类。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    pass
  
tudi = Prentice()
print(tudi.kongfu)
tudi.make_kongfu()

# 注：当一个类有多个父类的时候，默认使用第一个父类的同名属性和方法。
```

### 5.4子类重写父类同名方法和属性

故事：tudi掌握了师傅和青云宗功法后，自己潜心钻研出一套 全新的功法。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    def __init__(self):
        self.kongfu = '[独创功法]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
  
tudi = Prentice()
print(tudi.kongfu)
tudi.make_kongfu()

print(Prentice.__mro__)
```

结论：如果子类和父类拥有同名属性和方法，子类创建对象调用同名属性和方法的时候，调用到的是子类里面的同名属性和方法。

- 快速查看某个类的继承顺序：此类名.__ mro__函数

```python
# 示例：
print(Prentice.__mro__)
# 控制台输出这个类的父类及继承关系。
```

##### a，子类调用父类的同名方法和属性

故事：很多武林人士都想学习老爷子的功法和青云宗的功法。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    def __init__(self):
        self.kongfu = '[独创功法]'
        
    def make_kongfu(self):
      # 如果是先调用了父类的属性和方法，父类属性会覆盖子类属性，故在调用属性前，先调用自己子类的初始化 
        self.__init__()
        print(f'获得{self.kongfu}')
        
     # 子类调用父类的同名方法和属性，把父类的同名属性和方法再次封装 
     # 调用父类方法，但是为保证调用到的是也是父类的属性，必须在调用方法前调用父类的初始化
    def make_master_kongfu(self):
        # 父类类名.函数()
        # 再次调用初始化的原因：这里想要调用父类的同名方法和属性，属性在init初始化位置，所以需要再次调用init
        Master.__init__(self)
        Master.make_kongfu(self)
   
    def make_partment_kongfu(self):
        Partment.__init__(self)
        Partment.make_kongfu(self)
  
tudi = Prentice()
print(tudi.kongfu)
tudi.make_kongfu()

tudi.make_master_kongfu()

tudi.make_partment_kongfu()
```

##### b，多层继承

一个父类继承给一个子类，这个子类再继承给自己的子类，将这种两层以上的继承关系称为多层继承。

故事：N年后，tudi老了，想要把所有的功法传承给自己的弟子。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    def __init__(self):
        self.kongfu = '[独创功法]'
        
    def make_kongfu(self):
        self.__init__()
        print(f'获得{self.kongfu}')
        
    def make_master_kongfu(self):
        Master.__init__(self)
        Master.make_kongfu(self)
   
    def make_partment_kongfu(self):
        Partment.__init__(self)
        Partment.make_kongfu(self)
        
# 徒孙类
class Tusun(Prentice):
    pass
    
dizi = Tusun()

dizi.make_kongfu()

dizi.make_master_kongfu()

dizi.make_partment_kongfu()
```

##### c，super()调用父类方法

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(Master):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
        # 方法2.1
        # super(Partment,self).__init__()
        # super(Partment,self).make_kongfu()
        
        # 方法2.2
        super().__init__()
        super().make_kongfu()
          
class Prentice(Partment):
    def __init__(self):
        self.kongfu = '[独创功法]'
        
    def make_kongfu(self):
        self.__init__()
        print(f'获得{self.kongfu}')
        
    # 子类调用父类同名方法和属性：把父类的同名属性和方法再次封装
    def make_master_kongfu(self):
        Master.__init__(self)
        Master.make_kongfu(self)
   
    def make_partment_kongfu(self):
        Partment.__init__(self)
        Partment.make_kongfu(self)
        
    # 一次性调用父类的同名属性和方法
    def make_old_kongfu(self):
        # 方法一：代码冗余：父类类名如果变换，这里代码需要频繁修改
        # Master.__init__(self)
        # Master.make_kongfu(self) 
        # Partment.__init__(self)
        # Partment.make_kongfu(self)
        
        # 方法二：super()
        # 2.1： super(当前类名，self).函数()
        # super(Prentice,self).__init__()
        # super(Prentice,self).make_kongfu()
        # 2.2： super().函数()
        super().__init__()
        super().make_kongfu()
        
tudi = Prentice()

tudi.make_old_kongfu()
```

注：使用super()可以自动查找父类，调用顺序遵循__ mro__类属性的顺序。比较适合单继承使用。

##### d，私有属性和方法(私有权限)

在Python中，可以为实例属性和方法设置私有权限，即设置某个实例属性或实例方法不继承给子类。

故事：tudi把功法传承给弟子的同时，不想把自己的宝剑继承给弟子，这个时候就要为宝剑这个实例属性设置私有权限。

- 定义私有属性和方法

设置私有权限的方法：在属性名和方法名前面加上两个下划线__。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    def __init__(self):
        self.kongfu = '[独创功法]'
        # 定义私有属性
        self.__sword = '[宝剑]'
        
    # 定义私有方法
    def __info_print(self):
        print(self.kongfu)
        print(self.__sword)
         
    def make_kongfu(self):
        self.__init__()
        print(f'获得{self.kongfu}')
        
    def make_master_kongfu(self):
        Master.__init__(self)
        Master.make_kongfu(self)
   
    def make_partment_kongfu(self):
        Partment.__init__(self)
        Partment.make_kongfu(self)
        
# 徒孙类
class Tusun(Prentice):
    pass
    
tudi = Prentice()
# 对象不能访问私有属性和私有方法
# print(tudi.__sword)
# tudi.__info_print()

dizi = Tusun()
# 子类无法继承父类的私有属性和私有方法
# print(dizi.__sword)    # 无法访问实例属性__sword
# dizi.__info_print()

```

注：私有属性和私有方法只能在类里面访问和修改。

- 获取和修改私有属性值

在Python中，一般定义(工作习惯---非必须)函数名get_xx用来获取私有属性，定义set_xx用来修改私有属性值。

```python
class Master(object):
    def __init__(self):
        self.kongfu = '[内功修为]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Partment(object):
    def __init__(self):
        self.kongfu = '[青云宗功法秘籍]'
        
    def make_kongfu(self):
        print(f'获得{self.kongfu}')
        
class Prentice(Partment，Master):
    def __init__(self):
        self.kongfu = '[独创功法]'
        # 定义私有属性
        self.__sword = '[离火剑]'
        
    # 定义私有方法
    def __info_print(self):
        print(self.kongfu)
        print(self.__sword)
         
    # 获取私有属性
    def get_sword(self):
        return self.__sword
      
    # 修改私有属性
    def set_sword(self):
        self.__sword = '[寒冰剑]'
        
    def make_kongfu(self):
        self.__init__()
        print(f'获得{self.kongfu}')
        
    def make_master_kongfu(self):
        Master.__init__(self)
        Master.make_kongfu(self)
   
    def make_partment_kongfu(self):
        Partment.__init__(self)
        Partment.make_kongfu(self)
        
# 徒孙类
class Tusun(Prentice):
    pass
    
tudi = Prentice()

dizi = Tusun()
# 调用get_sword函数获取私有属性sword的值
print(dizi.get_sword())
# 调用set_sword函数修改私有属性sword的值
dizi.set_sword()
print(dizi.get_sword())
```

### 5.5 多态

多态：传入不同的对象，产生不同的结果。

多态指的是一类事物有多种形态。(一个抽象类有多个子类，因而多态的概念依赖于继承)。

- 定义：多态是一种使用对象的方式，子类重写父类方法，调用不同子类对象的相同父类方法，可以产生不同的执行结果。
- 好处：调用灵活，有了多态，更容易编写出通用的代码，做出通用的编程，以适应需求的不断变化！
- 实现步骤：

​               1.定义父类，并提供公共方法

​               2.定义子类，并重写父类方法

​               3.传递子类对象给调用者，可以看见不同子类执行效果不同

示例：

需求：警务人员和警犬一起工作，警犬分为两种，追击敌人和追查毒品，警务人员携带不同的警犬，执行不同的任务。

```python
class Dog(object):
    def work(self):   # 父类提供统一的方法，哪怕是空方法
        print('指哪打哪...')
        
class ArmyDog(Dog):  # 继承Dog类
    def work(self):  #  子类重写父类同名方法
        print('追击敌人...')
        
class DrugDog(Dog):  # 继承Dog类
    def work(self):  #  子类重写父类同名方法
        print('追查毒品...')     
        
class Person(object):
    # 传入不同的对象，执行不同的代码，即不同的work函数
    def work_with_dog(self,dog):
        dog.work()
      
d = Dog()
ad = ArmyDog()
dd = DrugDog()

P1 = Person()
P1.work_with_dog(d)    # 指哪打哪...
P1.work_with_dog(ad)   # 追击敌人...
P1.work_with_dog(dd)   # 追查毒品...

```

### 5.6 类属性和实例属性

类属性就是**类对象**所拥有的属性。

实例属性就是指代描述某个事物的特征。(事物的特征，归属者是实例即对象)

##### a，类属性

1. 设置和访问类属性

- 类属性就是**类对象**所拥有的属性，它被**该类的所有实例对象所共有**。
- 类属性可以使用**类对象**或**实例对象**访问。

```python
class Running(object):
    member = 7
    
gary = Running()
haha = Running()

print(Running.member)  # 7
print(gary.member)  # 7
print(haha.member)  # 7
```

类属性的优点：

- 记录的某项数据始终保持一致时，则定义类属性。
- 实例属性要求**每个对象**为其单独开辟一份内存空间来记录数据，而类属性为全类所共有，仅占用一份内存，更加节省内存空间。

2. 修改类属性

类属性只能通过类对象修改，不能通过实例对象修改，如果通过实例对象修改类属性，表示的是创建了一个实例属性。

```python
class Running(object):
    member = 7
    
gary = Running()
haha = Running()

# 修改类属性
Running.member = 8
print(Running.member)  # 8
print(gary.member)  # 8
print(haha.member)  # 8

# 不能通过对象修改属性，如果这样操作，实则是创建了一个实例属性
gary.member = 2
print(Running.member)  # 8
print(gary.member)  # 2
print(haha.member)  # 8
```

##### b，类方法

1.类方法特点

- 需要用装饰器@classmethod来标识其为类方法，对于类方法，**第一个参数必须是类对象**，一般以cls作为第一个参数。

2.类方法使用场景

- 当方法中**需要使用类对象**(如访问私有类属性等)时，定义类方法
- 类方法一般和类属性配合使用

```python
class Running(object):
    __member = 7
    
    @classmethod
    def get_member(cls):
        return cls.__member
      
gary = Running()
result = gary.get_member()
print(result)  # 7
```

##### c，静态方法

1.静态方法的特点

- 需要通过装饰器@staticmethod来进行修饰，**静态方法既不需要传递类对象也不需要传递实例对象**(形参没有**self/cls**)。
- 静态方法也能够通过**实例对象**和**类对象**去访问。

2.静态方法使用场景

- 当方法中**既不需要使用实例对象**(如实例对象，实例属性)，**也不需要使用类对象(**如类属性，类方法，创建实例等)时，定义静态方法
- **取消不需要的参数传递**，有利于**减少不必要的内存占用和性能消耗**

```python
class Run(object):    
    @staticmethod
    def info_print():
        print('hello world')
      
gary = Run()
# 静态方法既可以使用对象访问也可以使用类访问
gary.info_print()
Run.info_print()
```

## 六，异常

### 6.1 异常介绍

#### 1.**了解异常**

当检测到一个错误时，解释器就无法继续执行了，反而出现了一些错误的提示，这就是所谓的"异常"。

例如：以r方式打开一个不存在的文件。

```python
open('demo.txt','r')

# 结果为：FileNotFoundError: [Errno 2] No such file or directory: 'demo.txt'
```

#### 2. **异常的写法**

- 语法

```python
try：
    可能发生错误的代码
except：
    如果出现异常执行的代码
```

示例：以r模式打开文件，如果文件不存在，则以w模式打开。

```python
try：
    f = open('test.txt','r')
except：
    f = open('test.txt','w')
```

#### 3. 异常类型

由于书写程序时，书写的代码不一样，导致书写的异常不一样。

如：NameError，FileNotFoundError等

表现：异常类型表现在运行后，红字报错的最后一行的冒号：前面。

### 6.2 捕获异常

#### 1. 捕获指定异常

- 语法


```python
try：
    可能发生错误的代码
except 异常类型：
    如果捕获到该异常类型执行的代码
```

- 示例


```python
try：
    print(num)
except NameError：
    print('有错误')
```

注：

1. 如果尝试执行的代码的异常类型和要捕获的异常类型不一致，则无法捕获异常。
2. 一般try下方只放一行尝试执行的代码。

#### 2. 捕获多个异常

使用场景：尝试执行的代码发生的异常，不确定是except中哪一个异常类型时。

当捕获多个异常时，可以把要捕获的异常类型的名字，放在except后，并使用元组进行书写。

```python
try：
    print(1/0)
except (NameError,ZeroDivisionError)：
    print('有错误')
```

- 捕获异常描述信息

```python
try：
    print(num)
except (NameError,ZeroDivisionError) as result：
    print(result)
```

- 捕获所有异常

Exception是所有程序异常类的父类。

```python
try：
    print(num)
except Exception as result：
    print(result)
```

#### 3. 异常的else

else表示的是如果没有异常要执行的代码。

```python
try：
    print(1)
except Exception as result：
    print(result)
else:
  	print('没有异常时，执行的代码')
```

#### 4. 异常的finally

finally表示的是无论是否发生异常都要执行的代码，如：关闭文件。

```python
try：
    f = open('test.txt','r')
except Exception as result：
    f = open('test.txt','w')
else:
  	print('没有异常时，执行的代码')
finally：
	f.close()
```

### 6.3 异常传递

异常传递：指的是异常可以嵌套书写。

需求：

1. 尝试以只读方式打开demo.txt文件，如果文件存在则读取文件内容，文件不存在则提示用户即可。
2. 读取内容要求：尝试循环读取内容，读取过程中如果检测到用户意外终止程序，则except捕获异常并提示用户。

代码实现：

```python
 import time
try:
    f= open('test.txt')
    try:
        while True:
            content = f.readline()
            if len(content) ==0:
              	break
            # 休眠两秒
            time.sleep(2)
            print(content)
    except:
      	# 如果在读取文件时，产生异常，就会捕获到
        # 比如：命令提示符中按下了ctrl+c
        print('意外终止了读取数据')
    finally:
        f.close()
        print('关闭文件')
except:
    print('文件不存在')
    
```

### 6.4 自定义异常

在Python中，抛出自定义异常的语法为raise异常类对象。

作用：将不符合程序逻辑要求的代码，捕获异常，抛出异常。

需求：密码长度不足，则报异常 (用户输入密码，长度不足三位，则报错，即抛出自定义异常，并捕捉该异常)。

代码实现如下：

```python
# 自定义异常类，继承Exception
class ShortInputError(Exception):
    def __init__(self,length,min_len):
        self.length = length
        self.min_len = min_len
        
    # 设置抛出异常的描述信息
    def __str__(self):
        return f'你输入的长度是{self.length},不能少于{self.min_len}个字符'
      
def main():
    try:
        con = input('请输入密码：')
        if len(con) < 3:
            raise ShortInputError(len(con),3)
    except Exception as result:
        print(result)
    else:
        print('密码输入完毕')
            
main()   
```

## 七，模块

### 7.1 模块介绍

**定义**：Python模块(Module)，是一个Python文件，以.py结尾，包含了Python对象定义和Python语句。

模块能定义函数，类和变量，模块里也能包含可执行的代码。

- import 模块名
- from 模块名 import 功能名
- from 模块名 import *
- import 模块名 as 别名
- from 模块名 import 功能名 as 别名

#### 1. 导入模块

方法一：import 模块 语法：

```python
# 1. 导入模块
import 模块名
import 模块名1，模块名2...   # 不推荐

# 2. 调用功能
模块名.功能名()
```

示例：

```python
import math
print(math.sqrt(9))   
# 3.0 python中除法计算不管是否有浮点数参与计算，结果都为浮点数
```

方法二：from...import...  语法：

调用省去书写模块名.功能名，直接书写功能名

```python
from 模块名 import 功能名1，功能名2，功能名3...
```

示例：

```python
from math import sqrt
print(sqrt(9))  # 3.0
```

方法三：from...import  *  语法：

调用省去书写模块名.功能名，直接书写功能名

```python
from 模块名 import *
```

示例：

```python
from math import *
print(sqrt(9))  # 3.0
```

#### 2. as定义别名

原有的名字，不方便记忆和使用，可以通过定义别名更改使用。更改后只能使用更改后的别名，不能使用更改前的名字。

语法：

```python
# 模块定义别名
import 模块名 as 别名

# 功能定义别名
from 模块名 import 功能 as 别名
```

示例：

```python
# 模块别名
import time as tt

tt.sleep(1)
print('hello')

# 功能别名
from time import sleep as s1
s1(1)
print('hello')
```

### 7.2 模块制作

在Python中，每个Python文件都可以作为一个模块，模块的名字就是文件的名字。也就是说**自定义模块名必须符合标识符命名规则**。

#### 1.  定义模块

新建一个Python文件，命名为my_module1.py，并定义testA函数。

```python
def testA(a,b):
    print(a+b)
```

#### 2. 测试模块

在实际开发中，当一个开发人员编写完一个模块后，为了让模块能够在项目中达到想要的效果，这个开发人员会自行在py文件中添加一些测试信息，例如，在my_module1.py文件中添加测试代码。

```python
def testA(a,b):
    print(a+b)
    
testA(1,1)
```

此时，无论是当前文件，还是其他已经导入了该模块的文件，在运行的时候都会自动执行testA函数调用。

解决方法如下：

```python
def testA(a,b):
    print(a+b)

# 只在当前文件中调用该函数，其他导入的文件内不符合该条件，则不执行testA函数调用。 
if __name__ == '__main__':
    testA(1,1)
    
#注：__name__ 是系统变量，是模块的标识符
#   __name__ 在本文件使用的时候，打印结果为__main__
#   __name__ 在别的文件中导入调用时，打印的结果为它所在的文件名
```

#### 3. 调用模块

```python
import my_module1
my_module1.testA(1,1)
```

#### 4. 模块定位顺序

当导入一个模块时，Python解释器对模块位置的搜索顺序是：

1. 当前目录。
2. 如果不在当前目录，Python则搜索在shell变量PYTHONPATH下每个目录。
3. 如果都找不到，Python会查看默认路径，UNIX下，默认路径一般为/usr/local/lib/python/。

模块搜索路径存储在system模块的sys.path变量中。变量里包含当前目录，PYTHONPATH和由安装过程决定的默认目录。

注意：

- 自己的文件名不要和已有模块名重复，否则导致模块功能无法使用。
- 使用from模块名import功能的时候，如果功能名字重复，调用到的是最后定义或导入的功能。

#### 5. __ all __ 

如果一个模块文件中有__ all __ 变量，当使用from   xxx   import *导入时，只能导入这个列表中的元素。

- my_module1模块代码

```python
__all__ = ['testA']

def testA():
    print('testA')
    
def testB():
    print('testB')
```

- 导入模块的文件代码

```python
# 只能导入__all__列表中的内容
from my_module1 import *
testA()   # testA
testB()   # error ，没有添加到all列表
```

### 7.3 python中的包

包将有联系的模块组织在一起，即放到统一个文件夹下，并且在这个文件夹创建一个名字为__ init __.py文件，那么这个文件夹就称之为包。

#### 1. 制作包

[New] — [Python Package]— 输入包名—[OK]—新建功能模块(有联系的模块)。

注意：新建包后，包内部会自动创建__ init __.py文件，这个文件控制着包的导入行为。

示例：

1. 新建包mypackage
2. 新建包内模块：my_module1和my_module2
3. 模块内代码如下：

```python
# my_module1
print(1)

def info_print1():
    print('my_module1')
```

```python
# my_module2
print(2)

def info_print2():
    print('my_module2')
```

#### 2. 导入包

- 方法一：

```python
import 包名.模块名

包名.模块名.功能名
```

示例：

```python
import mypackage.my_module1

mypackage.my_module1.info_print1()
```

- 方法二：

注：**必须**在__ init __ .py文件中添加 __ all __ = []，控制允许导入的模块列表。 

```python
from 包名 import *

模块名.功能名
```

示例：

```python
from mypackage import *

mypackage.info_print1()
```

