---

layout: post

title:  "引用限定（Ref-qualifiers）"

categories: C++

tags:  Ref-qualifiers c++11 efficiency

---
先看一个例子：

```c++
class A
{
    vector<int> _v;
public:
    void setter(const vector<int> &v)
    {
        cout << "lvalue setter" << endl;
        _v = v;
    }
    void setter(vector<int> &&v)
    {
        cout << "rvalue setter" << endl;
        _v = move(v);
    }
    const vector<int>& getter() const
    {
        cout << "getter" << endl;
        return _v;
    }
};

A AFactory()
{
    A a1;
    a1.setter(vector<int>{1});
    return a1;
}

A a2;
a2.setter(AFactory().getter());

输出：
rvalue setter
getter
lvalue setter
```

注意第三个输出，表示 a2 对象调用了左值引用版本。但实际上呢？

```c++
AFactory().getter()
```

的返回值是一个左值引用，a1 对象是临时对象，在语句执行完后就被销毁了，因此，a1.getter() 返回值也会被销毁，既然都会被销毁，那么 a2.setter() 期望调用的是右值重载版本，当然，这需要能够识别出一个对象是否是临时对象，c++11 提供了引用限定的成员函数（Ref-qualifiers for member function），能够根据对象来调用不同的成员函数。书写方式是在函数名括号后面加**&**或**&&**。

上述代码可修改为：

```c++
// 非临时对象调用版本
const vector<int>& getter() const &
{
    cout << "lvalue getter" << endl;
    return _v;
}
// 临时对象调用版本
vector<int>&& getter() &&
{
    cout << "rvalue getter" << endl;
    return move(_v);
}

输出：
rvalue setter
rvalue getter
lvalue setter
```

