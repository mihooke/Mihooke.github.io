---

layout: post

title:  "做一个自己的Python库"

categories: Python

tags:  Python wheel setuptools

---
本文的目标是创建一个自己的Python库，并安装到Python系统目录下，就像调用通常的第三方包一样。
包结构如下：
![image](https://github.com/mihooke/Mihooke.github.io/tree/master/blog_images/python_setuptools.png)

其中mihooke_package下面是要打包的库文件，包括3个文件__init__.py，client.py，server.py；setup.py位于mihooke_package同级目录，是安装库的安装文件，此文件必不可少。
# 1.编写setup.py
```python
from setuptools import setup

setup(
    name = "MihookePackage",
    description = "A package for test",
    author = "Mihooke",
    packages = ["mihooke_package"]
    )
```
setup函数有很多参数，不过对于安装一个简单的包，上述已经足够了。
这里值得注意的是，setup函数用的是setuptools包里面的，而非distutils.core里的，我们不用distutils.core，是因为在Python3中setuptools是更高级的更好用，功能更多。

# 2.组织自己的包类似上述结构
setup.py一定要和module同级目录，并在包的最外层

3.在此目录下运行命令
# python .\setup.py build
此命令会build你的代码，在此目录下生成build文件夹

4.继续运行命令
# python .\setup.py bdist_wheel
此命令生成安装文件wheel文件，若提示bdist_wheel命令不识别，则是电脑里没有装wheel库，需要执行pip install wheel安装。
命令执行后，会在此目录下生成dist和egg-info文件夹，在dist下面会有wheel文件存在

5.继续运行命令
# pip install .\dist\MihookePackage-0.0.0-py3-none-any.whl
注意，此命令需要管理员权限执行，会提示安装成功与否的信息。可到Python安装目录Python36\Lib\site-packages下查看是否存在刚才安装的包