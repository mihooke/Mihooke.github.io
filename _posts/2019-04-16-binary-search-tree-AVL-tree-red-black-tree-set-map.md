---

layout: post

title:  "binary search tree AVL tree 和set map"

categories: DataStructure&Algorithm

tags:  Data-Structure binary-search-tree AVL red-black-tree set map

---

二叉搜索树（binary search tree）是这样一棵树，对于每一个节点，其左孩子比它小，右孩子比它大，由于树高为logN，因此我们总可以在logN时间内找到最小值或最大值。

实现树的各种操作利用递归思想很简单，也十分适合递归思想，由于树高平均为logN，所以不用担心栈空间被用尽，递归算法尽管很简明，但是其效率肯定没有非递归实现效率高。

下面二叉搜索树的实现是利用了递归，需要强调一下插入删除操作，在此我们不考虑重复元素。针对删除，存在三种情况：

- 所删除的节点是叶子

这种情况直接删除就行了。

- 删除的节点有一个孩子

这种情况直接让此孩子节点上移就行了。

- 所删除的节点有两个孩子

这种情况稍微复杂点，需要在所删除节点的右子树种找到最小节点，使其上移到删除位置。

最后，需要知道二叉搜索树之所以效率高，是由树高来保证的，那树高怎么保证logN呢？就是尽量保证根元素是中值，这就需要生成树时，元素的输入是随机的，否则很有可能会形成一颗左子树或右子树很大的树，导致树高远大于logN。

```c++
template <typename T>
class BinarySearchTree
{
public:
    BinarySearchTree() : _root(nullptr) {}
    BinarySearchTree(const BinarySearchTree &other)
    {
        _root = clone(other._root);
    }
    BinarySearchTree(BinarySearchTree &&other)
    {
        _root = clone(move(other._root));
    }
    ~BinarySearchTree()
    {
        makeEmpty(_root);
    }

    bool isEmpty() const {return _root == nullptr;}
    bool contains(const T &value) const
    {
        return contains(value, _root);
    }
    bool findMin(T &value) const
    {
        if (TreeNode *tn = findMin(_root))
        {
            value = tn->data;
            return true;
        }
        return false;
    }
    bool findMax(T &value) const
    {
        if (TreeNode *tn = findMax(_root))
        {
            value = tn->data;
            return true;
        }
        return false;
    }
    void insert(const T &value)
    {
        insert(value, _root);
    }
    void remove(const T &value)
    {
        remove(value, _root);
    }
    BinarySearchTree &operator=(const BinarySearchTree &other)
    {
        if (this != &other)
        {
            _root = clone(other._root);
        }
        return *this;
    }
    BinarySearchTree &operator=(BinarySearchTree &&other)
    {
        _root = clone(move(other._root));
        return *this;
    }

private:
    void makeEmpty(TreeNode * &t)
    {
        if (t != nullptr)
        {
            makeEmpty(t->left);
            makeEmpty(t->right);
            delete t;
            t = nullptr;
        }
    }
    TreeNode *clone(TreeNode *other) const
    {
        if (other == nullptr) return nullptr;
        return TreeNode{other->data, clone(other->left), clone(other->right)};
    }
    bool contains(const T &value, TreeNode *t) const
    {
        if (t == nullptr) return false;
        if (t->data > value) return contains(value, t->left);
        else if (t->data < value) return contains(value, t->right);
        return true;
    }
    TreeNode *findMin(TreeNode *t) const
    {
        if (t == nullptr) return nullptr;
        if (t->left == nullptr) return t;
        return findMin(t->left);
    }
    TreeNode *findMax(TreeNode *t) const
    {
        if (t == nullptr) return nullptr;
        if (t->right == nullptr) return t;
        return findMax(t->right);
    }
    void insert(const T &value, TreeNode * &t)
    {
        if (t == nullptr)
            t = new TreeNode(value, nullptr, nullptr);
        if (t->data > value)
            insert(value, t->left);
        else if (t->data < value)
            insert(value, t->right);
    }
    void insert(T &&value, TreeNode * &t)
    {
        if (t == nullptr)
            t = new TreeNode(move(value), nullptr, nullptr);
        if (t->data > value)
            insert(value, t->left);
        else if (t->data < value)
            insert(value, t->right);
    }
    void remove(const T &value, TreeNode * &t)
    {
        if (t == nullptr) return;
        if (t->data > value)
            remove(value, t->left);
        else if (t->data < value)
            remove(value, t->right);
        // 删除的节点有2个子节点
        if (t->left != nullptr && t->right != nullptr)
        {
            t->data = findMin(t->right)->data;
            remove(t->data, t->right);
        }
        else // 删除的节点有1个子节点
        {
            TreeNode *old = t;
            t = (t->left != nullptr) ? t->left : t->right;
            delete old;
        }
    }

private:
    class TreeNode
    {
    public:
        T data;
        TreeNode *left;
        TreeNode *right;
        TreeNode(const T &value, TreeNode *l, TreeNode *r) : data(value), left(l), right(r) {}
    };
    TreeNode *_root;
};
```



AVL Tree是二叉搜索树的平衡版本，因为当二叉搜索树发展为单侧树时，其搜索时间复杂度接近O(N)，，此时必须对树进行平衡，才能保证平均O(logN)的时间复杂度。平衡所做的事情就是保证任一节点的左右子树高度差最大为1。

在平衡过程中由于要频繁用到节点树高，因此在实现中保存树高信息，便于快速比较树高。捎带提一下树高和深度的区别：

- 高度：当前节点到叶子节点的距离。我们规定空树高度为-1，一个节点的树高度为0
- 深度：当前节点到根节点的距离

当一个节点插入平衡树中导致不平衡时，不平衡情况可分为4种：

1. 在不平衡节点的左子树左侧插入
2. 在不平衡节点的左子树右侧插入
3. 在不平衡节点的右子树左侧插入
4. 在不平衡节点的右子树右侧插入

![image](https://github.com/mihooke/Mihooke.github.io/blob/master/blog_images/single_rotate.png?raw=true)

![image](https://github.com/mihooke/Mihooke.github.io/blob/master/blog_images/double_rotate.png?raw=true)

其中1和4是镜像对称（称为**外侧插入**），进行一次单旋转可使树恢复平衡；2和3是镜像对称（称为**内侧插入**），需要进行双旋转才能使树平衡。由于AVL树本身就是一颗二叉搜索树，因此其大部分操作和BinarySearchTree一样，这里AVL实现就只列与BinarySearchTree不同部分：插入和删除

```c++
template <typename T>
class AVLTree
{
public:
    int height(TreeNode *t) const { return t == nullptr ? -1 : t->height; }
    void insert(const T &value)
    {
        insert(value, _root);
    }
    void remove(const T &value)
    {
        remove(value, _root);
    }
    TreeNode *findMin(TreeNode *t) const
    {
        if (t == nullptr) return nullptr;
        if (t->left == nullptr) return t;
        return findMin(t->left);
    }

private:
    void insert(const T &value, TreeNode *&t)
    {
        if (t == nullptr) return;
        if (t->data > value) insert(value, t->left);
        else if (t->data < value) insert(value, t->right);
        balance(t);
    }
    void remove(const T &value, TreeNode *&t)
    {
        if (t == nullptr) return;
        if (t->data > value) remove(value, t->left);
        else if (t->data < value) remove(value, t->right);
        if (t->left != nullptr && t->right != nullptr)
        {
            t->data = findMin(t->right)->data;
            remove(value, t->right);
        }
        else
        {
            TreeNode *old = t;
            t = t->left != nullptr ? t->left : t->right;
            delete old;
        }
        balance(t);
    }

    void balance(TreeNode *&t)
    {
        if (t == nullptr) return;
        if ((height(t->left) - height(t->right)) > 1)
        {
            if (height(t->left->left) >= height(t->left->right)) // 情况1，左左
                singleRotateLeft(t);
            else // 情况2，左右
                doubleRotateLeft(t);
        }
        else if ((height(t->right) - height(t->left)) > 1)
        {
            if (height(t->right->right) >= height(t->right->left)) // 情况4，右右
                singleRotateRight(t);
            else // 情况3，右左
                doubleRotateRight(t);
        }
        t->height = max(height(t->left), height((t->right))) + 1;
    }
    void singleRotateLeft(TreeNode *&k2)
    {
        TreeNode *k1 = k2->left;
        k2->left = k1->right;
        k1->right = k2;
        k2->height = max(height(k2->left), height(k2->right)) + 1;
        k1->height = max(height(k1->left), height(k1->right)) + 1;
        k2 = k1;
    }

    void singleRotateRight(TreeNode *&k2)
    {
        TreeNode *k1 = k2->right;
        k2->right = k1->left;
        k1->left = k2;
        k2->height = max(height(k2->left), height(k2->right)) + 1;
        k1->height = max(height(k1->left), height(k1->right)) + 1;
        k2 = k1;
    }
    void doubleRotateRight(TreeNode *&k3)
    {
        singleRotateRight(k3->left);
        singleRotateLeft(k3);
    }
    void doubleRotateLeft(TreeNode *&k3)
    {
        singleRotateLeft(k3->right);
        singleRotateRight(k3);
    }

private:
    class TreeNode
    {
    public:
        T data;
        TreeNode *left;
        TreeNode *right;
        int height;
        TreeNode(const T &value, TreeNode *l, TreeNode *r) : data(value), left(l), right(r) {}
    };
    TreeNode *_root;
};
```

红黑树（red black tree）是一种流行的AVL树，之所以比普通AVL树流行，是因为红黑树执行插入所需要的开销相对较低，实际中发生的旋转也相对较少。红黑树满足一下规则：

1. 每个节点非黑即红
2. 根节点是黑的
3. 如果节点是红色，其子节点必须是黑色
4. 任一节点至树尾端空节点的任何路径，都包含相同数量的黑节点

为了方便编程实现，规定空节点为黑色的。

当插入一个节点时，根据规则4，必须把新节点设为红色的，如果新节点的父节点是红色的，便违反了规则3，必须对树进行旋转调整。红黑树的旋转根据其插入位置（外侧还是内侧）和新节点伯父节点颜色（父节点的兄弟节点），可分为以下几种情况：

1. 伯父节点为黑，且外侧插入：此种情况进行一次单旋转（类似AVL单旋转），并更改父节点即祖父节点颜色
2. 伯父节点为黑，且内侧插入：需要双旋转
3. 伯父节点为红，且外侧插入：单旋转，旋转后，如果祖父节点的父节点是黑色的，便完成；否则，继续往上执行一次单旋转，直到没有父子连续为红的情况
4. 伯父节点为红，且内侧插入：只改变颜色，无需旋转

为了防止情况3向上循环单旋转，可以在旋转之前对节点进行一个处理：在插入新节点的路径上，如果遍历到某个节点的2个子节点都为红色，则把此节点改为红色，2个子节点改为黑色，之后再进行旋转操作。

std::set和std::map底层实现都是利用了红黑树，set是没有重复元素的集合，并且元素会自动排序，这里要注意的是set的迭代器是const类型，不允许写入，并且set不支持像vector那样随机访问元素，只能挨个进行访问，因为set迭代器只提供了operator++重载，并没有提供operator+重载。map同理。