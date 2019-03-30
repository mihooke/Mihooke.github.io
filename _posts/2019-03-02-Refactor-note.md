---

layout: post

title:  "《重构》笔记"

categories: Refactor

tags:  Refactor BadCode OO Object

---
书中有一句名言：要嗅到坏代码的味道
# --重新组织函数--

1. Extract Method-提炼函数，从大的函数中提炼出小函数，消除临时变量，提炼关键要看函数名称和函数本体之间的语义距离。
2. Inline Method-内联函数，若函数调用中不需要一层间接性，可直接把函数内容复制到调用处
3. Inline Temp-内联临时变量，一个临时变量，被表达式赋了一次值，则可以去掉这个临时变量，取而代之的是这个表达式
4. Replace Temp With Query-以查询取代临时变量，临时变量被表达式赋值了，可以把表达式提取到一个函数中
5. Introduce Explaining Variable-引入解释性变量，如果逻辑表达式复杂，可以用临时变量来说明表达式的意义
6. Split Temporary Variable-分解临时变量，如果一个临时变量承担了多个责任，应该使用多个临时变量
7. Replace Method With Method Object-以函数对象取代函数，如果有一个大型函数，无法使用Extract Method完成分解，就可以用此法，把临时对象封装到新的对象中，新的对象用成员函数来完成各个临时变量的工作
8. Substitute Algorithm-替换算法，一段程序实现方法看上去比较笨，换一种写法

# --在对象之间搬移特性--

1. Move Method-移动函数，在一个类中，对另一个类的某些属性调用多了，可以考虑在后者中添加一个函数，前者调用后者，只保留委托调用
2. Move Field-移动字段，同上，某个成员变量移到另一个类中
3. Extract Class-提炼类，某个类做了多类事情
4. Inline Class-内联类，某个类没有做太多事情
5. Hide Delegate-隐藏委托关系，将委托类声明为私有的
6. Remove Middle Man-移除中间人，某个类做了委托类更多的工作，可以直接让用户调用委托类的接口
7. Introduce Foreign Method-引入外部函数，服务类没有提供此接口，但这个功能理应在服务类中，并且服务类不可修改，可用外部函数实现
8. Introduce Local Extension-引入本地扩展，需要为服务类提供一些函数，但服务类不可修改，可引入新的类来做，新的类可以是子类继承服务类，也可以是包装类，包装服务类所有的功能

# --重新组织数据--

1. Self Encapsulate Field-自封装字段，类内访问成员变量，可根据自己喜好封装为取值函数
2. Replace Data Value With Object-以对象取代数据值，某些成员变量可封装在一个单独的类中
3. Change Value To Reference-将值对象改为引用对象，当多个实例需要拥有一个对象时，可利用静态工厂函数预生成若干对象，然后从中取值
4. Change Reference To Value
5. Replace Array With Object-以对象取代数组，一个数组的元素代表不同东西，用对象来表示
6. Duplicate Observed Data-赋值被监视的数据，Gui的数据有时候需要在逻辑层保留一份拷贝使用，需要把它复制到新的对象中，使用observer模式来同步数据
7. Change Unidirectional Association To Bidirectional-将单向关联改为双向关联，两个类之间关系的单向引用，如a调用了b，现在需要在b中调用a的方法，让其中一个类（最好是单个类）作为控制角色
8. Change Bidirectional Association To Unidirectional-将双向关联改为单向关联
9. Replace Magic Number With Symbolic Constant-以字面常量取代魔数
10. Encapsulate Field-封装字段，Public成员变量变为private，并提供相应public成员函数来访问这些成员变量
11. Replace Record With Data Class-以数据类取代记录，这些记录是以非面向对象程序存在的，新建类表示这个记录
12. Replace Type Code With Class-以类取代类型码，比如枚举，或者一些常量数字，都有特定含义，此类存在没有类型检查，将它们封装在一个类里面，并通过成员函数来访问这些类型码
13. Replace Type Code With Subclass-以子类取代类型码，若用类取代会影响宿主类的行为，可以用子类多态来处理变化行为
14. Replace Type Code With State/Strategy-以state/Strategy模式取代类型码，类型码无法通过子类消除，并且会影响类的行为，可利用state/Strategy模式，将类型码独立成类，如果有switch判断，可以分出子类来表现不同的行为
15. Replace Subclass With Field-以字段取代子类，子类只是函数的返回值不同，可在父类中增加成员变量来表示子类的行为，并消除子类

# --简化条件表达式--

1. Decompose Conditional-分解条件表达式，把if条件提取到一个函数中，同样if和else下面的语句也分别提取到函数中
2. Consolidate Conditional Expression-合并条件表达式，检查条件不同，但结果相同，可以合并条件到一起或提取为函数，类似情况还有嵌套条件判断，可用逻辑与合并条件；合理利用三元操作符条件表达式
3. Consolidate Duplicate Conditional Fragments-合并重复的条件片段，If和else下面的语句有相同的，何以提取出来
4. Remove Control Flag-移除控制标记，控制标记即控制循环的标记，可以不必遵守条件判断的单一出口原则，适当使用break或continue或return来跳出循环
5. Replace Nested Conditional With Guard Clauses-以卫语句取代嵌套条件表达式，卫语句是如果某个检查条件很罕见，就应该单独检查，单独检查的语句就是卫语句；嵌套条件判断分为多个卫语句，可以让函数有多个出口
6. Replace Conditional With Polymorphism-以多态取代条件表达式，根据对象类型的不同作为条件，可抽象子类
7. Introduce Assertion-引入断言，某段代码需要对程序状态做出某种假设，可以用断言明确表现这种假设

# --简化函数调用--

1. Rename Method-函数改名，要想成为一个真正的编程高手，起名的水平是至关重要的
2. Add Parameter-添加参数
3. Remove Parameter-移除参数
4. Separate Query From Modifier-将查询函数和修改函数分离
5. Parameterize Method-令函数携带参数，相同功能的函数内部可能只有某几个变量不一样，可以把它们提取到参数中
6. Replace Parameter With Explicit Methods-以明确函数取代参数，函数内根据参数的类型来做不同的事情，可把不同的事情明确地放在新的函数中，从而原来函数的参数就可以去掉了
7. Preserve Whole Object-保持对象完整，从某个对象中取出若干值，并作为函数的参数，可修改为参数传递整个对象
8. Replaceparameter With Method-以函数取代参数，对象调用某个函数，并把返回值作为另一个函数的参数，而后者也可以调用前一个函数，可以改为直接让后者调用前者，从而缩短参数列表
9. Introduce Parameter Object-引入参数对象，函数的参数总是一起出现，可以考虑把一起出现的参数封装在一个类中，转而传递对象
10. Remove Setting Method-移除设值函数，如果某个成员变量，希望在初始化后不再改变，可以把设值函数去掉
11. Hide Method-隐藏函数，成员函数没有被其他类用到，设为private
12. Replace Constructor With Factory Method-以工厂函数取代构造函数
13. Replace Error Code With Exception-以异常取代错误码，函数以错误码来表示异常时，可把错误码改成异常
14. Replace Exception With Test-以测试取代异常，异常不能被滥用，只适合用在意料外的情况，假如某个判断条件可以预先知道，可换为条件测试来判断

# --处理概括关系--

概括关系主要指继承体系中的关系

1. Pull Up Field-上移字段，两个子类有相同的成员变量，可将此移至父类中
2. Pull Up Method-函数上移
3. Pull Up Constructor Body-构造函数本体上移，子类的构造函数本体几乎完全一致，可提取共同部分到父类中
4. Push Down Method-函数下移，父类中函数只被部分子类使用，可移到相应的子类中
5. Push Down Field-字段下移
6. Extract Subclass-提炼子类，类中的某些特性只被部分实例用到，可把这些特性提炼到一个新的子类中
7. Extract Superclass-提炼超类
8. Extract Interface-提炼接口
9. Collapse Hierarchy-折叠继承体系，父类与子类无太大区别
10. Form Template Method-塑造模板函数，子类们有个函数行为相似，细节操作上不同，可把函数相同部分提到父类中，子类中调用新的提炼函数
11. Replace Inheritance With Delegation-以委托取代继承，某个子类只使用父类接口的一部分，或根本不需要继承而来的数据，可用委托代替继承关系
12. Replace Delegation With Inheritance-以继承取代委托

# --大型重构--

1. Tease Apart Inheritance-梳理并分解继承体系，某个类做了若干件不同的事，就需要拆解此类
2. Convert Procedural Design To Objects-将过程化设计转化为对象设计
3. Separate Domain From Presentation-将领域和显示分离，MVC模式
4. Extract Hierarchy-提炼继承体系，某个类做了太多工作，其中一部分以大量条件表达式完成，可以以一个子类来表示特殊情况