~~~markdown
以下几种方式依赖==用来确保线程安全

使用互斥量（mutex）来对共享资源进行保护。互斥量可以用来防止多个线程同时访问共享资源，从而避免数据竞争的问题。

使用读写锁（reader-writer lock）来对共享资源进行保护。读写锁允许多个读线程同时访问共享资源，但是写线程必须独占资源。这样可以在保证线程安全的同时，也尽可能地提高系统的并发性。

使用原子操作来对共享资源进行保护。在 C++ 中，可以使用 std::atomic 类型来定义原子变量，并使用原子操作来对共享资源进行操作。这样可以确保在多线程环境中，原子变量的操作是安全的。

使用条件变量（condition variable）来协调线程间的协作。条件变量可以用来在线程之间传递信号，从而控制线程的执行流程。

使用线程本地存储（thread-local storage）来保存线程的私有数据。线程本地存储可以用来给每个线程分配独立的存储空间，从而避免数据冲突的问题。


~~~

~~~c++
//使用互斥量（mutex）来保护共享资源：

#include <iostream>
#include <thread>
#include <mutex>

std::mutex g_mutex;  // 全局互斥量
int g_counter = 0;   // 共享资源

void incrementCounter()
{
    std::lock_guard<std::mutex> lock(g_mutex);  // 上锁
    ++g_counter;
}

int main()
{
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    std::cout << "g_counter = " << g_counter << std::endl;

    return 0;
}
————————————————
版权声明：本文为CSDN博主「从三岁开始。。。」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/u013820121/article/details/128542514
~~~

~~~c++
//使用读写锁（reader-writer lock）来保护共享资源：
#include <iostream>
#include <thread>
#include <shared_mutex>
std::shared_mutex g_mutex;  // 全局读写锁
int g_counter = 0;   // 共享资源

void incrementCounter()
{
    std::unique_lock<std::shared_mutex> lock(g_mutex);  // 上写锁
    ++g_counter;
}
void readCounter()
{
    std::shared_lock<std::shared_mutex> lock(g_mutex);  // 上读锁
    std::cout << "g_counter = " << g_counter << std::endl;
}
int main()
{
    std::thread t1(incrementCounter);
    std::thread t2(readCounter);

    t1.join();
    t2.join();
    return 0;
}
~~~

~~~c++
//使用原子操作来保护共享资源：
#include <iostream>
#include <thread>
#include <atomic>

//如果需要操作布尔型，可以使用 std::atomic 类型。
std::atomic<int> g_counter{0};  // 原子变量
void incrementCounter()
{
    ++g_counter;
}

int main()
{
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    std::cout << "g_counter = " << g_counter.load() << std::endl;

    return 0;
}
~~~

~~~c++
//使用条件变量（condition variable）来协调线程间的协作：

#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex g_mutex;
std::condition_variable g_cv;
bool g_flag = false;

void thread1()
{
    std::unique_lock<std::mutex> lock(g_mutex);
    g_flag = true;
    g_cv.notify_one();  // 通知线程 2
}

void thread2()
{
    std::unique_lock<std::mutex> lock(g_mutex);
    while (!g_flag)
    {
        g_cv.wait(lock);  // 等待通知
    }
    std::cout << "thread 2 finished" << std::endl;
}

int main()
{
    std::thread t1(thread1);
    std::thread t2(thread2);

    t1.join();
    t2.join();

    return 0;
}
~~~

~~~c++
//使用线程本地存储（thread-local storage）来保存线程的私有数据
#include <iostream>
#include <thread>

thread_local int g_counter = 0;  // 线程本地变量
void incrementCounter()
{
    ++g_counter;
    std::cout << std::this_thread::get_id() << ": " << g_counter << std::endl;
}

int main()
{
    std::thread t1(incrementCounter);
    std::thread t2(incrementCounter);

    t1.join();
    t2.join();

    return 0;
}
~~~

