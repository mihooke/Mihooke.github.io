---

layout: post

title:  "面向对象编程与基于对象编程"

categories: C++

tags:  OO

---

* content
{:toc}

我们都知道C++是一门面向对象的语言，它有四大范式：**better C，OO，data abstraction，generic programing**，其中很重要的一个就是OO，即面向对象，下面分别举例来说明面向对象编程和基于对象的编程。
假设我们需要设计一个绘图类Graphic，它的作用是画点，直线……面向对象写法如下：
``` c++
/*
 * OO, object-oriented implementation
 * pure virtual function
 */
class Graphic
{
public:
    virtual void drawPoint() = 0;
    virtual void drawLine() = 0;
};

class SpecialGraphic : public Graphic
{
public:
    void drawPoint() override {}
    void drawLine() override {}
};
```

要实现绘图功能，通常做法是基类提供interface（纯虚函数实现），子类来实现行为，当然，不同的子类有不同的行为；这里，如果我们需要10种不同的绘图行为，那么需要10个子类来分别继承Graphic抽象父类来实现。下面看看基于对象写法：
``` c++
/*
 * object-based implementation
 * std::function & std::bind
 */
class Graphic2
{
public:
    typedef std::function<void()> DrawCallback;
    Graphic2(){}

    void drawPoint() { m_point(); }
    void setPointCallback(DrawCallback callback) { m_point = callback; }
    void drawLine() { m_line(); }
    void setLineCallback(DrawCallback callback) { m_line = callback; }
private:
    DrawCallback m_point;
    DrawCallback m_line;
};
```

可以明显看到区别，基于对象写法已经不需要继承关系了：只需要利用function&bind组合来设置消息回调，大大降低了程序之间的耦合。
两种写法的用法区别也很大：
void draw_point() {}
void draw_line() {}

``` c++
    /// OO usage
    Graphic *sg = new SpecialGraphic();
    sg->drawPoint();
    sg->drawLine();

    /// object-based usage
    Graphic2 g2;
    g2.setPointCallback(bind(draw_point));
    g2.setLineCallback(bind(draw_line));
    g2.drawPoint();
    g2.drawLine();
```
	
其实最重要的是，二进制兼容问题，**二进制兼容**是指在库文件升级时，不必重新编译使用了这个库的可执行文件或其他库文件，我们来分析下为什么面向对象不是二进制兼容的，而基于对象确是兼容的。
先说面向对象写法，假设现在需要升级库，在Graphic中添加drawCircle方法，由于interface都是虚函数，所以 新加虚函数会造成sizeof(Graphic)增大，而虚函数在内存中是由vtbl来维护的，访问虚函数是通过函数的offset来调用的，所以在可执行文件调用升级库的时候，会找不到新加的drawCircle方法，若要正常使用，可执行文件必须重新编译。
再来看基于对象写法，由于Graphic2中都是non-virtual函数，所以sizeof(Graphic2)大小并不会改变，完美实现二进制兼容。

