---

layout: post

title:  "Python property装饰器"

categories: Python

tags:  property

---
Python的property由两种用法：
property函数形式：返回类的属性，类的方法由参数指定
property装饰器形式：由@property修饰类成员函数
这里暂不提property的实现，我们先来看看property的使用。为什么要用property？原因是由于Python中类的属性都是public的，即外部可以访问类的所有，即便你把变量声明为_a，__b，这只是一个约定俗成，告诉你这是类的私有变量，你不应该在外部去访问它们，但是你依然可以访问修改它们，有时候这是很方便直观的，但在有些时候，你需要对某些变量加一些约束，那么只好定义一些接口（getter和setter）来处理了。在C++中，这也是很正常很推荐的操作，通过接口来访问成员变量。但在Python中，这样会让程序员觉得多此一举，我明明可以直接访问成员变量，为啥还要访问接口呢？
这时候运用property就显得很有作用了，定义property后，你依然可以享受直接访问成员的便利，还可以自定义该成员变量的约束，下面用例子来说明用法。

property函数
```python
class Mihooke(object):
    def __init__(self):
        self._daughter = None
    
    def daughter_setter(self, d):
        self._daughter = d
    
    def daughter_getter(self):
        return self._daughter
    
    def daughter_grow_up(self):
        del self._daughter
    
    daughter = property(daughter_getter, daughter_setter, daughter_grow_up, "Mihooke has a daughter, Wow!")
```
我可以这么用：
```python
    m = Mihooke()
    m.daughter = "LingYue" # invoke daughter_setter()
    print(m.daughter) # invoke daughter_getter()
    del m.daughter #invoke daughter_grow_up()
```
property()函数有4个参数：
- getter -- 获取属性值的函数
- setter -- 设置属性值的函数
- del -- 删除属性值函数
- doc -- 属性描述信息
注意：property()函数实在类的内部使用的。

property装饰器
装饰器则是修饰一个函数，这里使用property来修饰。
```python
class Mihooke(object):
    def __init__(self):
        self._daughter = None
        
   @property
    def daughter(self):
        return self._daughter
    
    @daughter.setter
    def daughter(self, d):
        self._daughter = d
    
    @daughter.deleter
    def daughter(self):
        del self._daughter
```
用法和上面相同。
注意：使用装饰器后，3个函数名字是相同的，函数名字就是属性的名字

***后记***：这个property特性很有用，在实际项目工程中，假设Mihooke类已经封装成一个模块了，这里的代码不能做修改，必须要在外部动态改变它的属性值，从而来获知这个改动，那么就可以用property来实现。同样的功能在Qt中也有体现，那就是Q_PROPERTY，这也是一个很有用的功能，之后的文章我会来剖析它。
