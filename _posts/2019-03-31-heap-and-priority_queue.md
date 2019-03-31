---

layout: post

title:  "heap和priority_queue"

categories: DataStructure&Algorithm

tags:  Data-Structure heap priority_queue

---

heap形象来说它是一个完全二叉树，即树是满树，因此其父子节点的对应关系是固定的，其特性是按从上往下从左往右的顺序遍历，某个节点i，其两个子节点就是2i和2i+1，因此，heap底层保存数据可利用array。

heap可分为大根堆（big-heap）和小根堆（small-heap），其根节点是最大的和最小的，这里以大根堆为例来实现代码

```c++
template <typename T>
class Maxheap
{
public:
    Maxheap(size_t size) : _data(_size+1), _size(size) {}
    Maxheap(const vector<T> &v) : _data(v.size()+1), _size(v.size())
    {
        copy(v.cbegin(), v.cend(), _data.begin()+1);
        buildHeap();
    }
    void insert(const T &value)
    {
        if (_size == _data.size())
            _data.resize(2 * _size);
        _data[++_size] = value;
        int hole = _size;
        while (_data[hole / 2] < _data[hole / 2])
        {
            swap(_data[hole], _data[hole / 2]);
            hole /= 2;
        }
    }
    void insert(T &&value)
    {
        if (_size == _data.size())
            _data.resize(2 * _size);
        _data[++_size] = move(value);
        int hole = _size;
        while (_data[hole / 2] < _data[hole / 2])
        {
            swap(_data[hole], _data[hole / 2]);
            hole /= 2;
        }
    }
    void pop(T &to)
    {
        if (_size == 0) return;
        to = move(_data[1]);
        _data[1] = move(_data[_size--]);
        percolateDown(1);
    }

private:
    void buildHeap()
    {
        for (int hole = _size / 2; hole > 0; --hole)
            percolateDown(hole);
    }
    void percolateDown(int hole)
    {
        int child;
        for (; hole * 2 <= _size; hole = child)
        {
            child  = 2 * hole;
            if (child != _size && _data[child+1] > _data[child])
                ++child;
            if (_data[child] < _data[hole])
                break;
            swap(_data[child], _data[hole]);
        }
    }

    vector<T> _data;
    size_t _size;
};
```

C++标准库里priority_queue就是利用heap来实现的，带权值的queue，即queue中元素是按照权值来放置的。堆排序也是利用了堆的特性。