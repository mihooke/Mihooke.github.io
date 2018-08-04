---

layout: post

title:  "消除工程中第三方库的警告"

categories: C++

tags:  ThirdParty

---
我们都知道编译器在编译程序阶段都会对程序进行合法性的检查，如果不合法则会报出error信息，导致编译不能通过；同时也会进行合理性检查，如果某些语句不合理，则会报出warning信息，通常warning不影响编译生成汇编代码和可执行文件，但是warning信息通常都是友好的，出现warning，一般是代码中有潜在的问题，下面是常见的warning：
- 未使用函数参数（Unsed function parameter）
- 定义了从未使用过的变量（Variable defined but never used）
- 变量使用前未经初始化（Variable be used without being initialized）
- 不是所有的分支都能正确return
- 有符号数和无符号数不匹配

通常我们在编译项目的时候，都会把警告等级调到最高，比如在vs2015下

![image](https://github.com/mihooke/Mihooke.github.io/blob/master/blog_images/warning_level.png?raw=true)


会选中启用所有警告，同样在Qt Creator下的pro文件中，我们也会加上 /Wall来开启最高等级警告，目的就是为了防止由于失误导致的隐藏性错误。
截至到这一步，所有操作都是正确的，并且是推荐的做法，直到某一天，我们的项目，需要引入第三方库，比如Boost，Eigen等等，写好程序，再次编译程序你会发现信息栏会输出一大堆的warning信息，特别是有符号数和无符号数不匹配，但是我们也不能去修改第三方库啊，指不定会出现什么样的奇怪的错误导致编译错误呢。既想关掉第三方库的警告信息，但是需要在自己的项目中继续启用所有警告，那怎么办？
有一个办法：创建一个头文件，这个头文件包含第三方库的头文件，并在这个头文件中选择性地关闭某些warning，项目中统统include这个头文件，下面是个示例：
比如第三方库头文件eigen/eigen.h
```c++
新建my_eigen.h
#pragma warning(push)	// 仅禁用此头文件
 #pragma warning(disable:4100)	// 4100就是未使用函数参数
 #include "eigen/eigen.h"
#pragma warning(pop)	// 恢复最初的warning级别
```

再次编译，会发现整个世界都清静了!