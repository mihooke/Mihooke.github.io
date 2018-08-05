---

layout: post

title:  "Python字典遇到键值错误的处理"

categories: Python

tags:  dict keyError

---
通常在Python中处理字典是这么写的：
```python
    dic = {}
    try:
        print(dic["key1"])
    except KeyError:
        print("key1 not exists")
```
在字典元素不确定的时候要写很多try except块，对程序阅读很不直观，如果我们不想对KeyError当作异常处理，只是想在键不存在的时候赋一个默认值，这可怎么办呢？
字典有一个方法get()，可以这么做：
```python
    dic = {}
    print(dic.get("key1", "value1"))
```