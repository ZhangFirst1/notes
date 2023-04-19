#读取
```
n=int(input())#读取一个数
a,b=map(int,input().split())#读取两个用空格隔开的数
#input()返回的是一个字符串
#字符串有split()函数，默认是将字符串按空格分隔，并返回一个列表
a=list(map(int,input().split()))#将一行有空格的数字转化为一个数组
a=[0]+list(map(int,input().split())) #数组从1开始计数 
s=input()#读取一个字符串abcd
s=list(input())#读取一个字符串abcd并将其转化为一个数组 
```
#排序
```
#sort 是应用在 list 上的方法，sorted 可以对所有可迭代的对象进行排序操作。
#list 的 sort 方法返回的是对已经存在的列表进行操作，无返回值
#而内建函数 sorted 方法返回的是一个新的 list，而不是在原来的基础上进行的操作。

#给列表排序
a.sort()#从0到len(a)-1排序
sorted(a[1:len(a)])#从1到len(a)排序
 
#给集合排序：
s=set()
for i in range(0,len(a)):
    s.add(a[i])
s=sorted(s)

#逆序排序
a.sort(reverse=True)//从大到小排序

#新建比较函数   
#1.使用匿名表达式定义关键字
li.sort(key=lambda x:x[1])#一定要保证li中的每个元素都是拥有两个元素的列表
#2.自定义cmp函数
#对于cmp(x,y):如果想要x排在y前面那么返回一个负数，如果想x排在y后面那么返回一个正数
from functools import cmp_to_key
def cmp(a,b):
    return a-b
li=[3,5,2,52,45,24,6436,24]
li.sort(key=cmp_to_key(cmp))#b=sorted(a, key=cmp_to_key(cmp))
print(li)
```
#序列：
```
序列（sequence）是一种可迭代的、元素有序的容器类型的数据。
序列包括列表（list）、字符串（str）、元组（tuple）和字节序列（bytes）等。
序列中的元素都是有序的，每一个元素都带有序号，这个序号叫作索引。索引有正值索引和负值索引之分
函数：max(),min(),len()
加和乘操作：a*=2,a+=[1]
切片：[start：end：step] 切片的三个索引都可以省略
成员测试：in和not in 用于测试是否包含某一个元素
```
#列表：
```
'''
创建列表：
1.list(iterable)函数：参数iterable是可迭代对象（字符串、列表、
元组、集合和字典等）。
2.[元素1，元素2，元素3，⋯]：指定具体的列表元素，元素之间以
逗号分隔，列表元素需要使用中括号括起来
'''
li.append(x)#追加一个元素 
li.extend(t)#追加多个元素 
li+=t#追加多个元素 
li.count(x)#值等于x的元素有多少个 
li.insert(i,x)#插入元素,i指定索引位置,x是要插入的元素 
li.remove(x)#删除值等于x的元素。若有多个，则只删除第一个。一定要保证li中有这个元素，否则会报错O 
li.pop()#默认删除最后一个元素 
li.pop(i)#删除索引为i的元素并返回删除的值 
 
```
#元组
```
元组是一种不可变序列类型
'''
创建元组:
1.tuple(iterable)函数：参数iterable是可迭代对象(字符串、列表
、元组、集合和字典等)
2.(元素1，元素2，元素3，⋯)括号可省略
t=(1,)#创建只有一个元素的元素，后面的逗号不可以省略
t=()#创建空元组
tup=(1,2)
a,b=tup#元组赋值
'''
```
#集合：
```
集合(set)是一种可迭代的、无序的、不能包含重复元素的容器类型的数据。
'''
创建集合：
1.set(iterable)函数：参数iterable是可迭代对象（字符串、列表、元组、集合和字典等）
2.{元素1,元素2,元素3,⋯}：指定具体的集合元素,元素之间以逗号分隔
'''
add(elem)#添加元素,如果元素已经存在,则不能添加,不会抛出错误
remove(elem)#删除元素,如果元素不存在,则抛出错误
clear()#清除集合
```
#字典：
```
字典(dict)是可迭代的、通过键(key)来访问元素的可变的容器类型的数据
'''
键必须是可哈希的  
可哈希：所有不可变类型(数字,字符串,元组(元组中必须保证没有可变类型的元素))
不可哈希：可变类型(列表,字典) 
'''

'''
创建字典：
1.dict()函数。
2.{key1：value1，key2：value2，...，key_n：value_n} 
'''
dic={}#创建字典 dic=dict()
dic[1]='李四'#如果字典键1在字典中，则是修改字典值；否是是添加一对字典键值
dic.pop(1)#删除键值，一定要保证在字典中，否则会报错

#遍历字典
dic={1:"a",2:"b",3:"c"}
for i in dic.keys():
    print(i)
for j in dic.values():
    print(j)
for i,j in dic.items():
    print(i,j)
    

#字典排序
dic=dict()
dic[2]="ad"
dic[1]="sd"
dic[3]="gfj"
print(dic)#无序的
#sorted()函数,以列表形式返回
print(sorted(dic))#默认排序keys
print(sorted(dic.keys()))#排序keys
print(sorted(dic.values()))#排序values
#dic.items()将原来的字典中的键值对,分别搜存入到一个元组中–>(key,value)
print(sorted(dic.items()))#默认根据元组的第一个元素排序
print(sorted(dic.items(),key=lambda x:x[1]))
print(dict(sorted(dic.items())))#列表又转化为字典
```
##字符串：
```
字符串(str)是一种不可变的字符序列
#查找子字符串a.find(sub,start,end)
#在索引start到end之间查找子字符串sub,如果找到则返回最左端位置的索引；如果没有找到，则返回-1。
a="hello"
print(a.find("l",3))

'''
a.find(old,new,count) 不在原字符串上修改,返回一个新的字符串
用new子字符串替换old子字符串
count参数指定了替换old子字符串的个数
如果count被省略，则替换所有old子字符串
'''
a='AB CD EF GH' 
print(a.replace(' ','+',2))

'''
字符串分割a.split(sep,maxsplit=),返回一个列表
maxsplit是最大分割次数,可省略
'''
a='AB CD EF GH' 
print(a.split(" ",2))

endswith()#用于判断字符串是否以指定后缀结尾，如果以指定后缀结尾返回True，否则返回False
```
#类型转化：
```
int("80")#字符串转化为整数
float("80.0")#字符串转化为浮点数
int('AB',16)#字符串转化为数字，按照16进制
str(123)#数字转化为字符串

ord('A')#求字母的ASCII码
chr(65)#将ASCII码转化为字母

s.isdigit()#检查一个字符串中的字符是否全部为数字字符
s.isalpha()#检查一个字符串中所有的字符是否都是由字母构成的，并且至少有1个字符
"".isalpha()#False
"a b d".isalpha()#False
```
#控制输出
```
占位符
'''
"... %[key][flags][width][.precision][length type]conversion type ..." % values
%: 必须要有的符号。它标记占位符的开始。
key: 选填。映射的键，由带括号的字符序列组成，一般用于后面的values是是字典的场景。
flags: 选填。转换标志(Conversion flags), 会影响某些转换类型的结果。
width: 选填。最小字段宽度。如果指定为“*”（星号），则实际宽度从值中元组的下一个元素读取，要转换的对象位于最小字段宽度和可选精度之后。
precision: 选填。精度，写法为.precision（点+精度）。如果指定为“*”（星号），则实际宽度从值中元组的下一个元素读取，要转换的值位于精度之后。
length type: 选填。长度修改器。
Conversion type: 必须要有的符号。转换类型，也标记占位符的开始。
 
Conversion type	说明
s	字符串（使用str()方法转换任何Python对象）,一个非常通用的类型
d	十进制整数
f	十进制浮点数,自动保留六位小数
'''
print("%s"%3.25)#s类型很通用
print("%.2f"%3.529)#精确到几位小数
#设置字段的最小占位宽度
print("%05d"%3)
print("%010f"%3.14)

format()函数
a,b="alice",200
print("{} {}".format(a,b))


join()函数
'''
'sep'.join(seq)
sep:代表分隔符, 可以是单个字符如: , . - ; 等，也可以是字符串如: 'abc'。
seq:代表要连接的元素序列，可以是字符串、元组、列表、字典等。
'sep'和seq都只能是string型，不能是int型和float型。
'''
a=[1,2,3]  
print(" ".join(str(i) for i in a))
a="abc"
print(" ".join(a))

f-表达式
name="alice"
b=15
print(f"{name} is {b}")
 
```
#特殊语句
```
#try-except语句
while 1:
    try:
        m,n=map(int,input().split())
        print(dfs(0,0,m))
    except:
        break
        
'''
for-else语句
如果for循环正常执行完整，执行else语句
若中间break退出了循环，则不执行else语句
'''
for i in range(100):
    if i%36==0:
        print("yes")
        break
    else:
        print("no")
```
#变量在函数中值的修改
```
#global声明为全局变量
idx=0
def add():
    idx+=1
add()
print(idx) 
#报错：local variable 'idx' referenced before assignment变量在使用前没有被声明
 


'''
Python函数的值传递、引用传递https://blog.csdn.net/qq_41987033/article/details/81675514
对x赋值，相当于改变了x指向的内存地址

1.数字、字符、字符串、元祖都是值传递，因为你传递进去生成了一个新的变量x，你修改x的值无非是修改了x指向的内存块，而对原变量没有影响。
2.对于列表，字典这种能增删的变量类型，通过新变量x能修改内部元素指向的地址（但不改变x的指向），因而也会导致原变量被修改。

'''
 
#没有改变值
a=[1,2,3]
print(id(a))
def f(x):#这时x和a指向同一个内存块(地址)
    print(id(a),id(x))
    x=[1]#让x指向了另一个内存块，但是没有改变a的指向
    print(id(a),id(x))
print(a)
f(a)
print(a)#所以a的值没有改变
 
#改变了值
a=[1,2,3]
def f(x):#这时x和a指向同一个内存块
    print(id(a[0]),id(x[0]))
    x[0]=100#让x[0]即a[0]重新指向一个内存块，改变了值
    print(id(a[0]),id(x[0]))
print(a)
f(a)
print(a)
```
#赋值、浅拷贝。深拷贝
```
赋值、浅拷贝、深拷贝
https://blog.csdn.net/lovelyaiq/article/details/55102518
1. 赋值操作（包括对象作为参数、返回值）不会开辟新的内存空间，它只是复制了新对象的引用
2. 浅拷贝：浅拷贝会创建新对象，其内容是原对象的引用。浅拷贝之所以称为浅拷贝，是它仅仅只拷贝了一层
切片操作：b = a[:] 或者 b = [each for each in a]
工厂函数：b=list(a)
copy函数：b=copy.copy(a)
3. 深拷贝：深拷贝拷贝了对象的所有元素，包括多层嵌套的元素
b = copy.deepcopy(a)
```
#常用的数据结构
```
列表list
li.append(x)    O(1)
li.extend(t)    O(k)  
li.count(x)     O(n)
li.insert(i,x)  O(n)
li.remove(x)    O(n)
li.pop()        O(1)
li.pop(i)       O(n)
li.sort()       O(nlogn)
li.reverse()    O(n)
max()           O(n)
min()           O(n)
in              O(n) 
len(li)         O(1)
 
双端队列collections.deque 
append      O(1) 
appendleft  O(1) 
pop	        O(1) 
popleft	    O(1) 
extend	    O(k) 
extendleft	O(k) 

集合set  
x in s	    O(1) 
并集 s|t	O(len(s)+len(t))	 
交集 s&t	O(min(len(s), len(t)) 
差集 s-t	O(len(s))	     

字典dict
复制 	    O(n) 
取元素	    O(1) 
更改元素 	O(1) 
删除元素	O(1) 
遍历 	    O(n) 
```
#math库
```
import math
print(pow(10,2))#100
print(math.pow(10,2)) #100.0
math.pi
math.e
math.inf
math.ceil(10/3)#向上取整
math.floor(10/3)#向下取整
math.gcd(12,36)#最大公约数
math.log(8,2)
math.log(math.e)#后面省略默认求的是Inx

#Python内置快速幂函数
pow(a,b,p)#通过内置的方法直接调用，内置方法会把参数作为整型
math.pow(a,b,p)#math模块则会把参数转换为 float
```
#栈
```
s=list()
s.append(1)#进栈
t=s.pop()#出栈
```
#队列：
```
from collections import deque
q=deque()
q.append(3)#入队
t=q.popleft()#出队
```
#优先队列
```
from queue import PriorityQueue
q = PriorityQueue() 
q.put((0, 1))#入队
t=q.get()#出队
```
#小根堆
```
from heapq import *
heapify(arr)#将一个列表转化为小根堆O(n)
heappop(arr)#弹出堆顶O(logn)
heappush(arr,a)##向堆添加元素O(logn) 
```
#设置递归深度
```
import sys
sys.setrecursionlimit(10000)#设置递归深度
```
#读入优化
```
from sys import stdin
n=int(stdin.readline().split()
```
#子序列
```
原来序列中找出一部分组成的序列
```