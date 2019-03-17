---

layout: post

title:  "list和slist"

categories: DataStructure&Algorithm

tags:  Data-Structure list slist

---

我们已经了解了vector的弊端：虽然访问元素速度很快，但插入删除操作是O(n)时间复杂度，因为插入删除还需要移动后续元素。
如果你选用了vector作为数据的容器，但插入删除操作很频繁，那建议使用list，可以说vector和list是优缺点互补的；list是一个一个的node通过指针相连的，要访问某个位置的元素，必须从头或从尾挨个遍历，时间复杂度是O(n)，但插入删除就很快了，背后是指针之间的交换。
标准库提供了list容器，它是双向链表，也就是说每个node都保存了指向它的前驱和后继指针；slist是单链表，只有后继指针，因为找不到当前node的上一个node，slist不支持插入删除操作，只支持在链表头进行操作。

下面简单实现了list，其中iterator是简化版，我们只需知道每个容器内部都实现了对应的迭代器，方便容器的遍历，list的iterator其实保存了当前node，提供++操作。

list的结构模型是有一个head node和tail node，它们不保存数据，只是起到区分list头和尾的作用，数据的保存是在head和tail之间。这两个node的好处在于简化了代码的实现，比如对于list中第一个元素和最后一个元素的插入和删除操作，否则的话必须特殊对待。

```c++
template <typename T>
class List
{
public:
    struct Node
    {
        T data;
        Node *pre;
        Node *next;
        Node(const T &value=T{}, Node *p = nullptr, Node *n = nullptr)
            : data(value), pre(p), next(n)
        {}
    };
    class const_iterator
    {
    protected:
        Node *current;
    };
    class iterator : public const_iterator
    {};
public:
    List() : _head(new Node), _tail(new Node), _size(0) {}
    List(const List &other) : _head(new Node), _tail(new Node), _size(0)
    {
        for (auto & v : other)
            push_back(v);
    }
    List(List &&other) : _head(other._head), _tail(other._tail), _size(other._size)
    {
        other._head = nullptr;
        other._tail = nullptr;
        other._size = 0;
    }
    List& operator=(const List &other)
    {
        if (this != &other)
        {
            for (auto &v : other)
                push_back(v);
        }
        return *this;
    }
    List& operator=(List &&other)
    {
        std::swap(_head, other._head);
        std::swap(_tail, other._tail);
        std::swap(_size, other._size);
        return *this;
    }

    iterator begin() { return _head->next; } // Node隐式转换为iterator
    const_iterator begin() const { return _head->next; }
    iterator end() { return _tail; }
    const_iterator end() const { return _tail; }

    void push_front(const T &value) { insert(begin(), value); }
    void push_front(T &&value) { insert(begin(), std::move(value)); }
    void push_back(const T &value) { insert(end(), value); }
    void push_back(T &&value) { insert(end(), std::move(value)); }
    void pop_front() { erase(begin()); }
    void pop_back() { erase(--end()); }

    iterator insert(iterator pos, const T &value)
    {
        Node *currentNode = pos.current;
        Node *newNode = new Node(value, currentNode->pre, currentNode);
        // 先操作插入结点的前驱和后继
        newNode->pre = currentNode->pre;
        newNode->next = currentNode;
        // 再操作前驱的后继，后继的前驱
        currentNode->pre->next = newNode;
        currentNode->pre = newNode;
        ++_size;
        return newNode;
    }
    iterator insert(iterator pos, T &&value)
    {
        Node *currentNode = pos.current;
        Node *newNode = new Node(std::move(value), currentNode->pre, currentNode);
        newNode->pre = currentNode->pre;
        newNode->next = currentNode;
        currentNode->pre->next = newNode;
        currentNode->pre = newNode;
        ++_size;
        return newNode;
    }
    iterator erase(iterator pos)
    {
        Node *delNode = pos.current;
        iterator ret = delNode->next;
        delNode->pre->next = delNode->next;
        delNode->next->pre = delNode->pre;
        delete delNode;
        return ret;
    }
    iterator erase(iterator from, iterator to)
    {
        iterator itr = from;
        while (itr != to)
            itr = erase(itr);
        return itr;
    }

private:
    Node *_head;
    Node *_tail;
    size_t _size;
};
```

