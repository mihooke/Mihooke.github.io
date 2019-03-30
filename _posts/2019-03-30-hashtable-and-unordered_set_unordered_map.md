---

layout: post

title:  "hashtable和unordered_set unordered_map hash_set hash_map"

categories: DataStructure&Algorithm

tags:  Data-Structure hashtable unordered_set unordered_map hash_set hash_map

---
hashtable叫**散列表**，又叫**哈希表**，这种数据结构主要提供了**常数平均时间**的查找插入删除操作。

这里和二叉搜索树（binary search tree）做个对比，二叉搜索树是对数平均时间，但有一个假设：数据的输入必须有足够的随机性，否则的话，二叉树就不平衡了，会导致平方时间。

hashtable的关键有两个因素：表大小是一个**素数**；一个好的**散列函数**（hash function）。素数可以保证插入元素时的覆盖率（主要针对平方探测法）；散列函数提供了无限输入到有限单元的映射，从这个角度上看，hashtable可看作是一个**字典**（map），因为hashtable大小固定，因此在插入元素时会发生碰撞（即两个不同元素落在同一位置），常用的解决办法是：**线性探测法**、**平方探测法**、**分离链表法**。

在介绍这几个办法之前，先认识一个术语：填装因子（loading factor），它是指表中元素个数与表大小的比，即散列表中装的元素比例。

- 线性探测：当发生碰撞时，往后挨个遍历表，直到找到空闲位置。

- 平方探测：当发生碰撞时，按照1，3，5……2n-1 规则，直到找到空闲位置。

这两种方法要想取得好的效果，都要求填装因子<0.5，即表中元素数量少于一半，否则大量时间会花在遍历元素探测上。

- 分离链表：它是一个链表数组，当发生碰撞时，附加到链表后面，这种做法填装因子为1，主要是链表分配元素消耗时间。

线性探测/平方探测和分离链表各有优劣，可根据实际来选择使用，当内存紧张时，可选用分离链表；当考虑运行速度，可选用平方探测。在使用平方探测时，若填装因子>0.5时，其效率会降低，这时候需要进行**再散列**，具体做法是：分配更大的空间，将元素重新散列进去。

C++11之前不同平台可能有hash_set/hash_map，其底层是hashtable，C++11标准模板库里引入了unordered_set，unordered_map，他们便是用hashtable来实现的，替代了hash_set/hash_map，当然，hash_set/hash_map目前在很多平台也还是提供了。

以下是平方探测法和分离链表法的实现。

```c++
bool isPrime(size_t value)
{
    for (int i = 2; i <= sqrt(value); ++i) {
        if (value % i == 0)
            return false;
    }
    return true;
}

size_t nextPrime(size_t value)
{
    while (!isPrime(++value))
        ;
    return value;
}

template<>
class hash<string>
{
public:
    size_t operator() (const string &value) const
    {
        size_t v = 0;
        for (auto ch : value)
            v = 37 * v + ch;
        return v;
    }
};

// 平方探测法
template<typename T>
class Hashtable
{
public:
    enum HashType {Empty, Active, Deleted};
    class HashEntry
    {
    public:
        T data;
        HashType info;
        HashEntry(const T &value=T{}, HashType t=Empty) : data(value), info(t) {}
    };

    Hashtable(size_t size) : _array(nextPrime(size))
    {
        makeEmpty();
    }
    bool isActive(int pos) const
    {
        return _array[pos].info == Active;
    }
    bool contains(const T &value) const
    {
        return isActive(findPos(value));
    }
    int findPos(const T &value) const
    {
        int currentPos = hashFunction(value);
        int offset = 1;
        while (_array[currentPos].info != Empty && _array[currentPos].data != value)
        {
            currentPos += offset;
            offset += 2;
            if (currentPos >= _array.size())
                currentPos -= _array.size();
        }
        return currentPos;
    }
    bool insert(const T &value)
    {
        int currentPos = findPos(value);
        if (isActive(currentPos)) return false;
        _array[currentPos] = {value, Active};
        if (++_size > _array.size())
            rehash();
        return true;
    }
    bool insert(T &&value)
    {
        int currentPos = findPos(value);
        if (isActive(currentPos)) return false;
        _array[currentPos] = {move(value), Active};
        if (++_size > _array.size() / 2)
            rehash();
        return true;
    }
    bool remove(const T &value)
    {
        int currentPos = findPos(value);
        if (!isActive(currentPos)) return false;
        _array[currentPos].info = Deleted;
        --_size;
        return true;
    }
    void rehash()
    {
        const vector<HashEntry> old = _array;
        _array.resize(nextPrime(old.size() * 2));
        makeEmpty();
        for(const HashEntry &entry : old)
        {
            if (entry.info == Active)
                insert(move(entry.data));
        }
    }
private:
    void makeEmpty()
    {
        for (auto &entry : _array)
            entry.info = Empty;
        _size = 0;
    }
    size_t hashFunction(const T &value)
    {
        static hash<T> hf;
        return hf(value) % _array.size();
    }
    vector<HashEntry> _array;
    size_t _size;
};
```

```c++
// 分离链表法
class Hashtable
{
public:
    Hashtable(size_t size) : _array(nextPrime(size))
    {
        makeEmpty();
    }
    bool contains(const T &value) const
    {
        int pos = hashFunction(value);
        list<T> &curList = _array[pos];
        return find(curList.cbegin(), curList.cend(), value) == curList.cend();
    }
    bool insert(const T &value)
    {
        if (contains(value)) return false;
        int pos = hashFunction(value);
        _array[pos].push_back(value);
        if (++_size > _array.size())
            rehash();
        return true;
    }
    bool insert(T &&value)
    {
        if (contains(value)) return false;
        int pos = hashFunction(value);
        _array[pos].push_back(move(value));
        if (++_size > _array.size())
            rehash();
        return true;
    }
    bool remove(const T &value)
    {
        if (contains(value)) return false;
        int pos = hashFunction(value);
        list<T> &curList = _array[pos];
        auto itr = find(curList.cbegin(), curList.cend(), value);
        curList.erase(itr);
        --_size;
        return true;
    }
    void rehash()
    {
        const vector<list<T>> old = _array;
        _array.resize(nextPrime(old.size() * 2));
        makeEmpty();
        for (auto &l : old)
        {
            for (auto &v : l)
                insert(move(v));
        }
    }
private:
    void makeEmpty()
    {
        for (list<T> &l : _array)
            l.clear();
    }
    size_t hashFunction(const T &value)
    {
        static hash<T> hf;
        return hf(value) % _array.size();
    }
    vector<list<T>> _array;
    size_t _size;
};
```

