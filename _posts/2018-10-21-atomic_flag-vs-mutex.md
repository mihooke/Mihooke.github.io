---

layout: post

title:  "atomic_flag vs mutex"

categories: C++

tags:  atomic_flag mutex spin thread yield

---

atomic_flag使用参考：
https://en.cppreference.com/w/cpp/atomic/atomic_flag

mutex使用参考：
https://en.cppreference.com/w/cpp/thread/mutex

**atomic_flag**是一个原子bool类型，它保证是lock free(与锁无关)，并且它不提供加载或存储操作。利用atomic_flag可以实现自旋互斥(spin mutex），结合thread中的yield，亦可实现真正的互斥锁。既然C++11已经提供了方便的mutex以及RAII用的lock_guard，那么为什么还要用atomic_flag来手动实现一个互斥锁呢？答案是更高效。

看下面关于atomic_flag 和 mutex的测试代码：
编译环境：vs2015
CPU：i5
```c++
#include <iostream>
#include <atomic>
#include <thread>
#include <mutex>
#include <vector>
#include <chrono>

using namespace std;
using namespace chrono;

atomic_flag afLock = ATOMIC_FLAG_INIT;
atomic<bool> isStart(false);
mutex mLock;

void atomicFunc(int id)
{
    // spin with while loop
    while (afLock.test_and_set(memory_order_acquire)) ;
    int sum = 0;
    for (int i = 0; i < 1000000; ++i)
    {
        sum += i;
    }
    cout << "thread " << id << endl;
    afLock.clear(memory_order_release);
}
void atomicFunc2(int id)
{
    // yield when not satisfy
    while (!afLock.test_and_set(memory_order_acquire))
    {
        this_thread::yield();
    }
    int sum = 0;
    for (int i = 0; i < 1000000; ++i)
    {
        sum += i;
    }
    cout << "thread " << id << endl;
    afLock.clear(memory_order_release);
}

void mutexFunc(int id)
{
    lock_guard<mutex> l(mLock);
    int sum = 0;
    for (int i = 0; i < 1000000; ++i)
    {
        sum += i;
    }
    cout << "thread " << id << endl;
}

int main(int argc, char *argv[])
{
    {
        auto start = chrono::system_clock::now();
        vector<thread> vt;
        for (int i = 0; i < 10; ++i)
        {
            vt.emplace_back(atomicFunc, i);
        }
        for (auto &t : vt)
        {
            t.join();
        }
        auto end = chrono::system_clock::now();
        auto duration = chrono::duration_cast<microseconds>(end-start);
        cout << "atomicFunc:" << double(duration.count()) * microseconds::period::num / microseconds::period::den << endl;
    }
    {
        auto start = chrono::system_clock::now();
        vector<thread> vt;
        for (int i = 0; i < 10; ++i)
        {
            vt.emplace_back(atomicFunc2, i);
        }
        for (auto &t : vt)
        {
            t.join();
        }
        auto end = chrono::system_clock::now();
        auto duration = chrono::duration_cast<microseconds>(end-start);
        cout << "atomicFunc2:" << double(duration.count()) * microseconds::period::num / microseconds::period::den << endl;
    }
    {
        auto start = chrono::system_clock::now();
        vector<thread> vt;
        for (int i = 0; i < 10; ++i)
        {
            vt.emplace_back(mutexFunc, i);
        }
        for (auto &t : vt)
        {
            t.join();
        }
        auto end = chrono::system_clock::now();
        auto duration = chrono::duration_cast<microseconds>(end-start);
        cout << "mutexFunc:" << double(duration.count()) * microseconds::period::num / microseconds::period::den << endl;
    }
    return 0;
}
```
benchmark：

atomicFunc:0.048341

atomicFunc2:0.024079

mutexFunc:0.036097

某一次运行结果如上，多次运行，结果类似。
## 结论：
atomic_flag完全可以实现一个功能相同的mutex，如果使用自旋模式，那么效率并没有原生mutex高，是因为while loop，部分时间花在了CPU等待上；利用yield可实现一个高效的mutex