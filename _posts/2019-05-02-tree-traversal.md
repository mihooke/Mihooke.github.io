二叉树的遍历

二叉树的遍历操作实际中很少使用，二叉树的特性通常更适合进行查找。遍历分为先序，中序，后序和层次遍历。

先序遍历指先遍历父节点，再依次遍历左孩子和右孩子；中序遍历指先遍历左孩子，再依次遍历父节点和右孩子；后序遍历指先遍历右孩子，再依次遍历左孩子和父节点；层次遍历指从根节点开始，按照层级遍历节点。

遍历的实现当中，可分为递归实现和非递归实现。递归实现思路简单，非递归实现巧妙地利用了stack和queue的特性，值得学习。

```c++
template <typename T>
struct TreeNode
{
    TreeNode *left;
    TreeNode *right;
    T data;
};

/// 递归遍历
/// 先序
void preOrder(const TreeNode *t)
{
    if (!t) return;

    cout << t->data << endl;
    preOrder(t->left);
    preOrder(t->right);
}
/// 中序
void inOrder(const TreeNode *t)
{
    if (!t) return;

    inOrder(t->left);
    cout << t->data << endl;
    inOrder(t->right);
}
/// 后序
void postOrder(const TreeNode *t)
{
    if (!t) return;

    postOrder(t->left);
    postOrder(t->right);
    cout << t->data << endl;

}

/// 非递归遍历
/// 先序
void preOrder(const TreeNode *t)
{
    TreeNode *p = t;
    stack<TreeNode*> s;
    while (p || !s.empty())
    {
        if (p)
        {
            cout << p->data << endl;
            s.push(p);
            p = p->left;
        }
        else
        {
            p = s.top();
            s.pop();
            p = p->right;
        }
    }
}
/// 中序
void inOrder(const TreeNode *t)
{
    TreeNode *p = t;
    stack<TreeNode*> s;
    while (p || !s.empty())
    {
        if (p)
        {
            s.push(p);
            p = p->left;
        }
        else
        {
            p = s.top();
            s.pop();
            cout << p->data << endl;
            p = p->right;
        }
    }
}
/// 后序
void postOrder(const TreeNode *t)
{
    TreeNode *cur = t;
    TreeNode *prevVisited = nullptr;
    stack<TreeNode*> s;
    while (cur || !s.empty())
    {
        while (cur)
        {
            s.push(cur);
            cur = cur->left;
        }
        cur = s.top();
        if (!cur->right || cur->right == prevVisited)
        {
            cout << cur->data << endl;
            prevVisited = cur->right;
            s.pop();
            cur = nullptr;
        }
        else
            cur = cur->right;
    }
}
/// 层次遍历
void layerOrder(const TreeNode *t)
{
    TreeNode *p = t;
    queue<TreeNode*> q;
    q.push(p);
    while (!q.empty())
    {
        p = q.front();
        q.pop();
        cout << p->data << endl;
        if (p->left) q.push(p->left);
        else if (p->right) q.push(p->right);
    }
}
```

