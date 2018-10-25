# collections模块

tuple的简单介绍

tuple是元组，不可变的（immutable）。可迭代的（iterable）。

```python
tuple1=('zhangsan',28,180)
tuple1[0]=('lisi')
#运行报错，提示tuple是不可变的对象
tuple1=('lisi',28,180)#此时tuple已经指向了新的内存地址，不再是原来的tuple指向的地址，因为python中的变量只是一个符号，是对内存地址的引用。
```

tuple的不可变不是绝对的

```python
tuple1=('zhangsan',[28,180])
tuple1[1].append(10)
print(tuple1)
#得到 ('zhangsan',[28,180,10])   
#tuple的不可变指的是tuple中的元素指向内存id不可变，对于可变对象list发生改变时，其id不变，所以可以对tuple中的可变对象元素进行数据更改。
```

tuple的拆包   可以简化代码，list的拆包与tuple相同,dict的拆包比较特殊，得到的是字典的key，其实就相当于tuple的拆包（因为字典的key和tuple都是hashable），字典有自己的item用法，拆包不适用。

```python
tuple1=('zhangsan',28,180)
name,age,height=tuple1
print(name,age,height)
#得到 zhangsan 28 180
name,*other=tuple1
print(name,other)
#得到 zhangsan [28,180]
#另外，tuple可以作为一个整体被拆包，并将拆包后的元素存放到另一个tuple中
tuple2=(*tuple1,'abc',1234,'hah')
print(tuple2)
#得到  （'zhangsan',28,180，'abc',1234,'hah'）

#list的拆包
list1=['sl',25,180]
name,*other=list1
list2=[*list1,'master']
print(name,other,list2)
#sl [25, 180] ['sl', 25, 180, 'master']
 
    
#字典的拆包
dict1={'name':'wangwu','age':22,'height':178}
abc,*other=dict1
print(abc,other)
#得到    name ['age', 'height']
dict2={*dict1:,'edu':'master'}   #这种写法本身就是错的，不符合字典格式
```

tuple比list好的地方

```python
#tuple比list突出特点是immutable。也就是不可变性。
1.性能优化，python是一门解释型语言，代码运行时会变成pyc字节码，指出元素全部为immutable的tuple会作为常量在编译时确认，因此显著增加了运行速度。
2.线程安全
3.可以作为dict的key。dict的key必须是hashable,可哈希的对象。list不可以作为dict的key。
list1=['zhangsan',28,180]
tuple1=('zhangsan',28,180)
dict1={}
dict1[tuple1]='zhangsan'
#{('sl', 25, 180, 'GBD', 'XUESHI'): 'sl'}
dict1[list1]='zhangsan'
#报错    TypeError: unhashable type: 'list'  列表是可变的，不是可哈希对象
```

### nametuple：命名元组

nametuple的创建和类的创建类似，需要类名和属性，当不需要创建方法时，可以使用nametuple可以让代码更简洁。也更节省资源。

nametuple方法返回一个类，这个类继承tuple，实质就是一个tuple对象。在处理数据的时候经常使用，例如处理mysql数据库中的表的数据。

MyClass=nametuple(typename,field_names)         使用nametuple函数返回了一User类的实例化对象MyUser，字段为列表中的字段，实际上MyUser就是一个tuple，不过具有nametuple函数赋予的方法和属性。

```python
from collections import nametuple
#第一个参数是类名，第二个参数是字段名，生成一个MyClass对象，是个tuple
MyClass=nametuple(typename,field_names)
#使用nametuple函数返回了一User类的实例化对象MyUser，字段为列表中的字段，实际上MyUser就是一个tuple，不过具有nametuple函数赋予的方法和属性。
MyUser=nametuple('User',['name','age','height','education'])
user_tuple=('lisi',23,180)
user_list=['wangwu',2,13]
user_dict={'name':'wangwu','age':22,'height':178}
#实例化对象MyUser接收所有的iterable，即使是不适合拆包的字典，只要在可迭代对象前加上*号，就可以将其中的参数拆解为tuple元素传入MyUser中。
use_info=MyUser(*user_tuple,'master')
print(use_info,type(use_info))
#User(name='lisi', age=23, height=180, edu='master')   <class '__main__.User'>
use_info=MyUser(*user_list,'chuzhong')
print(use_info,type(use_info))
#User(name='wangwu', age=2, height=13, edu='chuzhong') <class '__main__.User'>
user_info=MyUser(*user_dict,'gaozhong')
print(use_info,type(use_info))
#User(name='name', age='age', height='height', edu='gaozhong') <class '__main__.User'>


#MyUser的方法：

 # _make(cls,iterable,*)第一个参数是类本身，第二个参数是一个可迭代对象,传入的这个对象中的元素必须与类的字段一一对应，缺一不可，否则报错。(tuple,list,dict)都是一样的。
user_info=MyUser._make(user_tuple)
print(user_info,type(user_info))
#报错  TypeError: Expected 4 arguments, got 3
user_info=MyUser._make(user_tuple,'xiaoxue')
#报错  TypeError: 'str' object is not callable
#需要重新定义一个和类对象的字段一一对应的tuple
standertuple=('lisi',23,180,'xiaoxue')
user_info=MyUser._make(standertuple)
print(user_info,type(user_info))
#User(name='lisi', age=23, height=180, edu='xiaoxue') <class '__main__.User'>

#  _asdict()：类对象调用此方法会返回一个OrderedDict对象，实质是一个字典
user_info_dict=user_info._asdict()
print(user_info_dict,type(user_info_dict))
#OrderedDict([('name', 'lisi'), ('age', 23), ('height', 180), ('edu', 'xiaoxue')]) <class 'collections.OrderedDict'>

```

### defaultdict：缺省字典，默认字典

defaultdict(callable)参数是一个可调用对象，返回一个字典，如果传入的是一个函数，函数必须是不带参数或者参数有默认值的。因为参数callable是不接受小括号的。

需要传入一个可调用对象：通过类型去判断Python对象是否可调用，需要同时判断是函数（FunctionType）还是方法（MethodType），或者类是否实现__call__方法。如果只是单纯判断python对象是否可调用，用callable()方法会更稳妥。

```python
#查询列表中元素出现的次数
user_dict={}
users=['bobo','bibi','didi','bobo','didi','didi']
for user in users:
    #dict.setdefault(self,k,d=none)给字典的键设定默认值，如果键第一次出现其值将会是默认的，而不会报错
    user_dict.setdefault(user,0)
    user_dict[user]+=1
print(user_dict)
#{'bobo': 2, 'bibi': 1, 'didi': 3}

#六大数据类型在定义类时都有__call__方法，所以都是可调用对象，传入int，默认是0.
user_dict=defaultdict(int)
users=['bobo','bibi','didi','bobo','didi','didi']
for user in users:
    user_dict[user]+=1
print(user_dict)
#defaultdict(<class 'int'>, {'bobo': 2, 'bibi': 1, 'didi': 3})

#如果想传入一个指定的dict作为默认值，需要定义一个函数（函数不能带参数）返回这个dict，并把函数作为可调用对象传入
def fun():
    return {'sex':'famale','age':0,'height':0}
# print(callable(fun))
new_dict=defaultdict(fun)
info_dict={
    'zhangsan':{'sex':'male','age':23,'height':180},
    'lisi':{'sex':'male','age':25,'height':170},
    'wangwu':{'sex':'male','age':15,'height':156},
}
info_dict['xiaofang']=new_dict['xiaofang']
print(info_dict)
#{'zhangsan': {'sex': 'male', 'age': 23, 'height': 180}, 'lisi': {'sex': 'male', 'age': 25, 'height': 170}, 'wangwu': {'sex': 'male', 'age': 15, 'height': 156}, 'xiaofang': {'sex': 'famale', 'age': 0, 'height': 0}}

```

### deque :双端队列

deque(iterable=())传入一个可迭代对象，元组，列表，字典，生成器，迭代器都可以。双端队列在队列两端都可以进行数据操作



在python3中queue模块的Queue就是通过deque来构造的。同时，Queue是线程安全的，因为受到全局解释器锁GIL的保护，所以在执行put等更改操作时，可以直接实现。而list是不安全的，所以在多线程编程的过程中，多个线程要对同一个list做处理就需要加入线程锁lock。

```python
from collections import deque


# append()，appendleft()
userdeque=deque(['lili','lucy','danny'])
#在队尾添加元素
userdeque.append('xiaosan')
#在队首添加元素
userdeque.appendleft('renzha')
print(userdeque)
# deque(['renzha', 'lili', 'lucy', 'danny', 'xiaosan'])


# pop() , popleft()
userdeque.popleft()
print(userdeque)
# deque(['lucy', 'danny'])
userdeque.pop()
print(userdeque)
# deque(['lucy'])
#extend(),extendleft()
userdeque2=deque(['jiji','didi','kuji'])


#extend(),extendleft()
#在队尾扩充
userdeque2.extend(userdeque)
print(userdeque2)
#deque(['jiji', 'didi', 'kuji', 'lili', 'lucy', 'danny'])
#在队首扩充
userdeque2.extendleft(userdeque)
print(userdeque2)
# deque(['danny', 'lucy', 'lili', 'jiji', 'didi', 'kuji'])


#index(value,start,stop)  根据值查询索引
print(userdeque2.index('lili',0,4))#根据值查询索引
#2


#count(value) 查询元素出现的次数
print（userdeque2.count('didi'))
# 1


#insert(index,object)  在指定索引前插入对象
userdeque2.insert(2,'dali')
print(userdeque2)
# deque(['danny', 'lucy','dali', 'lili', 'jiji', 'didi', 'kuji'])

#remove(value)  删除元素
userdeque2.remove('dali')
print(userdeque2)
# deque(['danny', 'lucy', 'lili', 'jiji', 'didi', 'kuji'])

#reverse()   逆序排列
userdeque2.reverse()
print(userdeque2)
#deque(['kuji', 'didi', 'jiji', 'lili', 'lucy', 'danny'])

#rotate(int)   把队尾元素放到队首,int默认为1，输入几代表执行几次。
userdeque2=deque(['danny', 'lucy', 'lili', 'jiji', 'didi', 'kuji'])
userdeque2.rotate(2)
print(userdeque2)
#deque(['didi', 'kuji', 'danny', 'lucy', 'lili', 'jiji'])

#根据索引查询值
print(userdeque2[0])
#  didi

#不能切片
print(userdeque2[0：3])
#TypeError: sequence index must be integer, not 'slice'
```

### Counter : 计数器

Counter是collections模块中专门用来做数据统计的。继承的是dict类。Counter(iterable)括号中传入任意的可迭代对象，如字符串，元组，列表，字典等，都会以字典的方式返回每个元素和该元素出现的次数。

```python
from collections import Counter
#创建可迭代对象list1
list1=['a','b','c','a','d','b']
#生成计数字典
coun1=Counter(list1)
coun2=Counter('abscsdankjasjasd')
print(coun1)
print(coun2)
#Counter({'a': 2, 'b': 2, 'c': 1, 'd': 1})
#Counter({'a': 4, 's': 4, 'd': 2, 'j': 2, 'b': 1, 'c': 1, 'n': 1, 'k': 1})

#update(iterable)跟新迭代对象  同时可以传入一个Counter对象
coun1.update('hhiijj')
print(coun1)
#Counter({'a': 2, 'b': 2, 'h': 2, 'i': 2, 'j': 2, 'c': 1, 'd': 1})
coun1.update(coun2)
print(coun1)
#Counter({'a': 6, 's': 4, 'b': 3, 'd': 3, 'c': 2, 'j': 2, 'n': 1, 'k': 1})


most_common（n）方法  使用了_headq这个数据结构，即常说的堆，返回的是_headq.nlargest()。堆就是专门用来解决top n的问题。
#most_common(n) 查询出现次数最多的元素，返回一个列表，列表的每个元素都是一个查询到的元素和次数组成的元组。n代表top n，就是出现次数最多的前n项
print(conu1.most_common(2))
# [('a', 6), ('s', 4)]


```

### OrderedDict ：有序字典

OrderedDict是一个有序的字典，会根据元素创建的顺序依次排列。当然在python3中也是默认按顺序排列的，但在python2中不是。

```python
from collections import OrderedDict
#实例化OrderedDict()
dict1=OrderedDict()
dict1['c']='lili'
dict1['b']='luci'
dict1['a']='mibi'
print(dict1)
#OrderedDict([('c', 'lili'), ('b', 'luci'), ('a', 'mibi')])


popitem() 
#删除最后一项，返回item
print(dict1.popitem())
print(dict1)
#('a', 'mibi')
#OrderedDict([('c', 'lili'), ('b', 'luci')])

pop(k)
#根据k值删除item，返回v
print(dict1.pop('c'))
print(dict1)
#lili
#OrderedDict([('b', 'luci'), ('a', 'mibi')])


move_to_end(k)
#将k和v移动到最后,没有返回值
print(dict1.move_to_end('c'))
print(dict1)
#None
#OrderedDict([('b', 'luci'), ('a', 'mibi'), ('c', 'lili')])
```

### Chainmap   ：链式映射

Chainmap（dict1，dict2.....）接受多个字典，返回的是一个迭代器，可以帮助我们访问多个dict就像访问一个dict一样方便，其数据来源还是引用的原来数据，并非拷贝。

```python
from collections import ChainMap
dict1={'a':'lili','b':'nancy'}
dict2={'c':'pipi','d':'gigi'}
newdict=ChainMap(dict1,dict2)
print(newdict)
for k,v in newdict.items():
    print(k,v)
#ChainMap({'a': 'lili', 'b': 'nancy'}, {'c': 'pipi', 'd': 'gigi'})
#a lili
#c pipi
#d gigi
#b nancy

如果字典中有重复的k，则后面出现的元素将会被丢弃，不会被打印出来
dict3={'c':'pipi','a':'gigi'}
newdict=ChainMap(dict1,dict3)
print(newdict)
for k,v in newdict.items():
    print(k,v)
#ChainMap({'a': 'lili', 'b': 'nancy'}, {'c': 'pipi', 'a': 'gigi'})
#c pipi
#a lili
#b nancy

maps属性，返回的是每个成员字典组成的列表
print(newdict.maps)
#[{'a': 'lili', 'b': 'nancy'}, {'c': 'pipi', 'a': 'gigi'}]

可以根据maps的返回列表进行字典v的查询和修改
newdict.maps[0]['a']='mimi'
print(newdict.maps)
for k,v in newdict.items():
    print(k,v)
#[{'a': 'mimi', 'b': 'nancy'}, {'c': 'pipi', 'a': 'gigi'}]
#b nancy
#c pipi
#a mimi


```

