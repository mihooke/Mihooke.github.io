---

layout: post

title:  "array和vector"

categories: Data Structure and Algorithm

tags:  Data-Structure array vector

---
array 就是原始数组，之所以使用vector封装array操作，是因为原始数组是静态的，一旦分配了，不能修改大小，这就导致了在实际使用数组时，如果不确定大小，不能很好地分配内存，这便是设计vector的初衷，以下Vector类实现了setter类函数，通常getter类函数只是返回对应成员值，故不再实现，另外，以下实现也未考虑一些异常情况，只是描述了某个功能的实现。

```c++
template <typename T>
class Vector
{
public:
    typedef T* iterator;
    typedef const T* const_iterator;
    static const int INIT_CAPACITY = 128;
    Vector() : _size(0), _capacity(INIT_CAPACITY), _data(new T[_capacity]) {}
    Vector() : _size(0), _capacity(INIT_CAPACITY), _data(new T[_capacity]) {}
    Vector(size_t size) : _size(size), _capacity(size+INIT_CAPACITY),
        _data(new T[_capacity]) {}
    Vector(size_t size, const T &value) : _size(size), _capacity(size+INIT_CAPACITY),
        _data(new T[_capacity])
    {
        for (int i=0; i<_size; ++i)
        {
            _data[i] = value;
        }
    }
    Vector(const Vector &other)
    {
        _data = new T[other._capacity];
        for (int i=0; i<other._size; ++i)
        {
            _data[i] = other._data[i];
        }
        _size = other._size;
        _capacity = other._capacity;
    }
    Vector(Vector &&other)
    {
        _data = other._data;
        _size = _size;
        _capacity = other._capacity;
    }
    Vector& operator=(const Vector &other)
    {
        if (this == &other) return *this;
        T *data = new T[other._capacity];
        for (int i=0; i<other._size; ++i)
        {
            data[i] = other._data[i];
        }
        std::swap(data, _data);
        _size = other._size;
        _capacity = other._capacity;
        delete []data;
        return *this;
    }
    ~Vector() {delete []_data;}
    void resize(size_t size)
    {
        if (size > _capacity)
        {
            Vector copy = *this;
            for (int i=0; i<_size; ++i)
            {
                _data[i] = copy._data[i];
            }
        }
        _size = size;
        _capacity = size + _capacity;
    }
    void reserve(size_t capacity)
    {
        if (capacity < _size) return;
        T *data = new T[capacity];
        for (int i=0; i<_size; ++i)
        {
            data[i] = _data[i];
        }
        std::swap(data, _data);
        _capacity = capacity;
        delete []data;
    }
    void push_back(const T &value)
    {
        if (_size == _capacity)
        {
            reserve(2*_capacity);
        }
        _data[_size++] = value;
    }
    void push_back(T &&value)
    {
        if (_size == _capacity)
        {
            reserve(2*_capacity);
        }
        _data[_size++] = std::move(value);
    }
    template <typename... Args>
    void emplace_back(Args&&... value)
    {
        if (_size == _capacity)
        {
            reserve(2*_capacity);
        }
        _data[_size++] = std::move(T(std::forward(value...)));
    }
    void pop_back() {_size--;}
    size_t size() const {return _size;}
    size_t capacity() const {return _capacity;}
    const T &operator[](size_t index) const {return _data[index];}
    T &operator[](size_t index) {return _data[index];}
    iterator erase(iterator pos)
    {
        iterator end = _data + _size;
        if (pos+1 != end)
        {
            std::copy(pos+1, end, pos);
        }
        _size--;
        return pos;
    }
    iterator erase(iterator begin, iterator end)
    {
        iterator tail = _data + _size;
        std::copy(end, tail, begin);
        _size = _size - (end - begin);
        return begin;
    }

private:
    size_t _size;
    size_t _capacity;
    T *_data = nullptr;
};
```

