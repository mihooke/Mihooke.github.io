---

layout: post

title:  "去除Python代码中的注释"

categories: Python

tags:  delComment

---
Python代码里的注释有3种：
- #后面的代码
- """  """三个双引号之间的
- '''  '''三个单引号之间的
非正式的还有双引号之间的，比如写在函数头：
```python
def f(x):
    "This is a informal annotation"
    return (x*x)
```
其实这里可以把双引号之间的当作无名的字符串。这里我们介绍如何去除上面三种注释。
一个思路是按照行遍历文件，遇到#，则删除#后面的字符；遇到"""或'''，则进行计数，如果数量是奇数，说明只接下来遍历的行在注释之间，跳过继续遍历，如果数量是偶数，说明本段注释遍历完毕，对其删除。按照这个思路，下面是参考代码：
```python
def deleteComment(file):
    with open(file, 'r') as f:
        lines = ""
        three_double_quota_count = 0
        three_single_quota_count = 0
        all_context = f.readlines()
        for i,line in enumerate(all_context):
            if (i > len(all_context)-2):
                break
            idx = line.find("#")
            if idx == 0:
                continue
            elif idx > 0:
                line = line[:idx] + "\n"
                
            idx_quota = line.find(r'"""')
            if idx_quota >= 0:
                three_double_quota_count += 1
                continue
            else:
                if three_double_quota_count % 2 == 1:
                    continue
            
            idx_quota = line.find(r"'''")
            if idx_quota >= 0:
                three_single_quota_count += 1
                continue
            else:
                if three_single_quota_count % 2 == 1:
                    continue
            lines += line
```
