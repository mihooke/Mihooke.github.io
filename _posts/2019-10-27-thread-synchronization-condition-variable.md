---

layout: post

title:  "线程间同步之条件变量的边边角角"

categories: C++

tags:  thread condition_variable mutex

---
先来看一下条件变量的一般用法：

```c++
// 等待端：线程 t1
{
    lock_guard(&mutex);
    while (!满足唤醒的目的)
        cond.wait();
}

// 唤醒端：线程 t2
{
    lock_guard(&mutex);
    cond.notify();
}
```

以上就是条件变量的一般用法。这里有几点值得我们注意。

1. t1 和 t2线程都有用到了mutex锁，并且在此情形下是同时使用，那会不会造成t1 已经在wait中，t2 获取不到mutex的情况呢？答案肯定是不会的，因为cond.wait()会释放锁，当cond进行wait时，t1 线程就挂起了，为了保证mutex能继续使用，只能先释放。
2. 唤醒端在notify时，并不一定要持有mutex
3. 等待端必须用while循环，而不是if来判断一次，这么做是为了防止虚假唤醒（spurious weakup）。虚假唤醒是在t2唤醒后，共享变量被线程t3 修改了，并在t1 获取到CPU时间片开始运行之前又修改回了原值，那t1 就认为共享变量是没有改变的，因此本次唤醒是无效的。为了防止这种情况，因此使用while循环来保证，只有进行了有效唤醒，则不再wait。

Linux下pthread_cond 的用法如上，C++11对于虚假唤醒有了新的用法：

```c++
// wait有重载版本
template< class Predicate >
void wait( std::unique_lock<std::mutex>& lock, Predicate pred );

// 使用
bool couldWeakup{false};
{
    std::unique_lock<std::mutex>& lock(mutex);
    cond.wait(lock, []{ return couldWeakup; });
}

{
    std::lock_guard<std::mutex> lk(mutex);
    couldWeakup = true;
    cond.notify_all();
}
```

wait()的第二个参数表示：等待是否应该继续持续，如果是false，则继续等待。等价于while循环写法。

