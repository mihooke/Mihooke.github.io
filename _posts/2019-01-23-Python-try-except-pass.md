---

layout: post

title:  "Python try except pass使用注意"

categories: Python

tags:  except pass

---
### Python try except使用注意

在 Python 中，我们可能会经常使用 try except 来捕获异常，比如：

```python
try:
    value = info_dict["key"]
except KeyError:
    print("Error key")
```

这段代码捕获的是 dict 的 KeyError 异常，当发生此异常时，会打印一句话。

有时候，为了图省事方便，你可能会这么写：

```python
try:
    # some operation
except:
    pass # pass 意味着什么都不做
```

当心！这么写是一个很糟糕的习惯。

我们来看看糟糕在哪里：

1. 隐藏了错误
2. 捕获了所有异常，甚至包括系统异常

#### 隐藏了错误

在 except 分支中只有 pass 语句，即发生异常时，什么也不做，程序继续执行，此时的状态很可能是错误的，正确的做法是在异常分支中打印适当的信息，以告诉用户发生了什么。

#### 捕获了所有异常，甚至包括系统异常

没有指定异常类型，程序便会捕获所有异常，包括Python中的各种异常，也包括内存满了，CPU 爆了，程序退出异常等系统性异常，正确的做法是指定对应的异常：

```python
try:
    # some operation
except Exception as e:
    print("Exception occured:", e)
```

指定 Exception 会捕获所有 Python 的异常，并打印出异常信息。