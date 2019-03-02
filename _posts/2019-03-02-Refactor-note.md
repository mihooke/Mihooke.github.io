---

layout: post

title:  "《重构》笔记"

categories: Refactor

tags:  Refactor BadCode

---
书中有一句名言：要嗅到坏代码的味道
--重新组织函数--
- 1.Extract Method-提炼函数，从大的函数中提炼出小函数，消除临时变量，提炼关键要看函数名称和函数本体之间的语义距离。
- 2.Inline Method-内联函数，若函数调用中不需要一层间接性，可直接把函数内容复制到调用处
- 3.Inline Temp-内联临时变量，一个临时变量，被表达式赋了一次值，则可以去掉这个临时变量，取而代之的是这个表达式
- 4.Replace Temp with Query-以查询取代临时变量，临时变量被表达式赋值了，可以把表达式提取到一个函数中
- 5.Introduce Explaining Variable-引入解释性变量，如果逻辑表达式复杂，可以用临时变量来说明表达式的意义
- 6.Split Temporary Variable-分解临时变量，如果一个临时变量承担了多个责任，应该使用多个临时变量
- 8.Replace Method with Method Object-以函数对象取代函数，如果有一个大型函数，无法使用Extract Method完成分解，就可以用此法，把临时对象封装到新的对象中，新的对象用成员函数来完成各个临时变量的工作
- 9.Substitute Algorithm-替换算法，一段程序实现方法看上去比较笨，换一种写法

--在对象之间搬移特性--
- 1.Move Method-移动函数，在一个类中，对另一个类的某些属性调用多了，可以考虑在后者中添加一个函数，前者调用后者，只保留委托调用
- 2.Move Field-移动字段，同上，某个成员变量移到另一个类中
- 3.Extract Class-提炼类，某个类做了多类事情
- 4.Inline Class-内联类，某个类没有做太多事情
- 5.Hide Delegate-隐藏委托关系，将委托类声明为私有的
- 6.Remove Middle Man-移除中间人，某个类做了委托类更多的工作，可以直接让用户调用委托类的接口
- 7.Introduce Foreign Method-引入外部函数，服务类没有提供此接口，但这个功能理应在服务类中，并且服务类不可修改，可用外部函数实现
- 8.Introduce Local Extension-引入本地扩展，需要为服务类提供一些函数，但服务类不可修改，可引入新的类来做，新的类可以是子类继承服务类，也可以是包装类，包装服务类所有的功能

--重新组织数据--
- 1.Self Encapsulate Field-自封装字段，类内访问成员变量，可根据自己喜好封装为取值函数
- 2.Replace Data Value With Object-以对象取代数据值，某些成员变量可封装在一个单独的类中
- 3.Change Value To Reference-将值对象改为引用对象，当多个实例需要拥有一个对象时，可利用静态工厂函数预生成若干对象，然后从中取值
- 4.Change Reference To Value
- 5.Replace Array With Object-以对象取代数组，一个数组的元素代表不同东西，用对象来表示
- 6.Duplicate Observed Data-赋值被监视的数据，GUI的数据有时候需要在逻辑层保留一份拷贝使用，需要把它复制到新的对象中，使用observer模式来同步数据
- 7.Change Unidirectional Association To Bidirectional-将单向关联改为双向关联，两个类之间关系的单向引用，如A调用了B，现在需要在B中调用A的方法，让其中一个类（最好是单个类）作为控制角色
- 8.Change Bidirectional Association To Unidirectional-将双向关联改为单向关联
- 9.Replace Magic Number With Symbolic Constant-以字面常量取代魔数
- 10.Encapsulate Field-封装字段，Public成员变量变为private，并提供相应public成员函数来访问这些成员变量
- 12.Replace Record With Data Class-以数据类取代记录，这些记录是以非面向对象程序存在的，新建类表示这个记录
- 13.Replace Type Code With Class-以类取代类型码，比如枚举，或者一些常量数字，都有特定含义，此类存在没有类型检查，将它们封装在一个类里面，并通过成员函数来访问这些类型码
- 14.Replace Type Code With Subclass-以子类取代类型码，若用类取代会影响宿主类的行为，可以用子类多态来处理变化行为
- 15.Replace Type Code With State/Strategy-以state/Strategy模式取代类型码，类型码无法通过子类消除，并且会影响类的行为，可利用state/Strategy模式，将类型码独立成类，如果有switch判断，可以分出子类来表现不同的行为
- 16.Replace Subclass With Field-以字段取代子类，子类只是函数的返回值不同，可在父类中增加成员变量来表示子类的行为，并消除子类