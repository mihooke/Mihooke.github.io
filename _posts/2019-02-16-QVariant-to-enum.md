---

layout: post

title:  "QVariant转enum"

categories: C++

tags:  QVariant enum Qt Meta-object

---
QVariant 是 Qt 中的万用类型，可表示 Qt 中大多数类型，但无法表示自定义的 enum 类型，这就需要借助 Qt 的元对象系统了。

```c++
enum Type
{
    A,
    B
};
```

添加自定义类型到元系统

Q_DECLARE_METATYPE(Type)

这个宏使系统（QMetaType）知道该自定义类型，包括 QVariant

```c++
enum Type
{
    A,
    B
};
Q_DECLARE_METATYPE(Type)
```

这样，enum 就可以被用于 QVariant 了，可以这么用：

```c++
QVariant type = QVariant::fromValue(Type::A); // 设置值
Type t = type.value<Type>(); // 取值
```

