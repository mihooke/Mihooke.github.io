---
layout: post

title:  "deque和queue stack"

categories: DataStructure&Algorithm

tags:  Data-Structure deque queue stack
---

deque是双向开头的连续线性容器，即容器的头和尾都可以进行插入删除。当然vector和list也是这样的容器，不过

vector是

#### **头部操作很耗费时间**

list是

#### **不是连续内存，元素访问耗费时间**

#### **deque结合了vector和list的优点：是连续内存，头部操作线性时间**

deque的实现也借鉴了vector的预分配机制，内部使用一个二维数组来存储数据，如图所示结构

![image](https://github.com/mihooke/Mihooke.github.io/blob/master/blog_images/deque.png?raw=true)

map数组存储了一系列指针，每个指针指向一块连续内存，使用的内存从start开始标记，直到finish结束，也就意味着deque是从中间开始使用，这样就很方便在头尾进行操作了，如果头或尾内存用尽了，便会像vector那样重新分配所有内存。

以下是deque的数据结构，deque的核心部分是在start或finish所指的空间用完之后，如何跳到下一部分，为此，我们在iterator中保存了3个指针：first end current，来判断当前部分内存是否用完了，还保存了指向总内存的地址，以此来跳到下一部分。

```c++
template <typename T, size_t BufSize=512>
class Deque
{
public:
    struct iterator
    {
        T **map = nullptr;
        T *first = nullptr;
        T *end = nullptr;
        T *current = nullptr;
        void setNode(T **node)
        {
            map = node;
            first = *node;
            end = first + BufSize;
            current = first;
        }
    };
    Deque(size_t n, const T &value) : _start{}, _finish{},
        _size(0)
    {
        size_t usedSize = n / BufSize;
        _size = 2 * usedSize;
        for (int i = 0; i < _size; ++i)
            _map[i] = new T[BufSize];
        T **nStart = _map + usedSize / 2;
        T **nFinish = nStart + usedSize - 1;
        int j = 0;
        for (iterator cur = nStart; cur != nFinish; ++cur)
        {
            for (int i = 0; i < BufSize; ++i)
                if (j < n)
                    cur[i] = value;
            j += BufSize;
        }
        _start.setNode(nStart);
        _finish.setNode(nFinish);
    }

private:
    T **_map = nullptr;
    iterator _start;
    iterator _finish;
    size_t _size;
};
```

vector，list，deque是三个基本的数据结构，有了它们，我们可以方便地实现stack和queue，利用某个容器来存储数据，只暴露相应接口。