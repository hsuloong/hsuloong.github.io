---
title: Linux下C++实现一个简单的线程池
urlname: linux-cpp-thread-pool
date: 2018-04-8 20:18:17
tags:
- 操作系统
- Linux
categories:
- 操作系统
---

## 线程池基本概念

线程池是一种线程使用模式，其思想非常类似于内存池，均是从性能出发而开发出来的一种优化技巧。线程池主要用在需要频繁执行较短的任务的情况下，由于短时间内如果进行大量线程的创建与销毁带来的开销是不可接受，因此通常预先创建好一些工作线程，然后在需要使用时直接将任务分派给空闲线程即可，同时可以根据任务数量动态增加减少内存池中的线程数量，在尽量将少资源占用的情况下获得较好的性能。


## 线程池特点

- 线程池数量限制

由于线程自身会占用系统资源，带来系统开销，因此个数并不是越多越好，但是如果太少将会降低并发性能，使得任务长期得不到服务，因此线程池数量需要有一个合理值。

- 适用于大量较短暂任务

一般而言，线程数量会小于并发任务量，因此如果任务时间很长，甚至和进程生命周期处在同一个数量级上，此时直接使用普通线程即可，无需使用线程池。

- 线程数量选取

一般而言，对于IO密集型线程，线程数量一般大于CPU数量，而计算密集型线程数量则取和CPU相同比较合理。

- 根据任务情况动态增减线程池中线程数量

初始线程池数量一般不直接创建到最大数量，因此在运行中需要根据任务情况动态进行线程的创建和销毁。

## 线程池基本结构

线程池的目的以及特点，可总结出线程池以及任务需要实现以下功能：

	1. 线程池初始化：用于创建一定数量的线程
	2. 线程池增加线程数量：用于任务过多时增加线程数量
	3. 线程池销毁：销毁线程池
	4. 往线程池中增加任务

结构体如下：

```cpp
//Condition.h文件

#include <pthread.h>

//封装Linux下互斥锁与条件变量，后期平台迁移只需要修改内部实现即可
class Condition
{
public:
    Condition();
    ~Condition();
    int lock(); //加锁
    int unlock(); //解锁
    int wait(); //等待条件变量
    int timeWait(timespec &waitTime); //等待一段时间
    int signal(); //给信号
    int broadcast(); //广播信号
private:
    void init(); //初始化锁和条件变量
    void destroy(); //销毁锁和条件变量
private:
    pthread_mutex_t mutext;
    pthread_cond_t cond;
};


//ThreadPool.h

#include "Condition.h"
#include <queue>
#include <set>
#include <string>
using std::queue;
using std::set;
using std::string;

//封装任务虚基类，可以方便扩充任务类别
class TaskBase
{
public:
    TaskBase(string tmpName, void *arg) : taskName(tmpName), taskArg(arg) {}
    virtual void taskRun() = 0;
protected:
    string taskName; //任务名称，用于测试
    void *taskArg; //任务参数
};

//封装好的线程池实现，只开放增加任务接口
class ThreadPool
{
public:
    ThreadPool(int initNum, int maxNum);
    ~ThreadPool();
    void addTask(TaskBase *tmpTask); //增加任务
    friend void *threadRoutine(void *arg); //将线程入口函数定义为友元
private:
    void initThreadPool(); //初始化线程池
    void destroyThreadPool(); //销毁线程池
private:
    int initThreadNum; //初始线程池数量
    int maxThreadNum; //最大允许的线程数量
    queue<TaskBase *> taskQueue; //任务队列
    set<pthread_t> threadIDs; //创建线程的ID集合
    int nowThreadNum; //当前线程池线程数量
    int idleThreadNum; //当前空闲线程数量
    Condition cond; //锁和条件变量
    bool isDestroyNotify; //线程销毁通知
};
```

## 线程池实现

TaskBase是一个抽象基类，需要派生出不同的任务，所有任务需要的参数可以全部打包到void *taskArg成员变量中。以下为线程池的具体实现。

```cpp
//class Condition，实现于Condition.cpp

#include "Condition.h"
#include <iostream>
using std::cerr;
using std::endl;
using std::cout;

Condition::Condition()
{
    init();
}

Condition::~Condition()
{
    destroy();
}

void Condition::init()
{
    int mIrc = pthread_mutex_init(&mutext, nullptr);
    int cIrc = pthread_cond_init(&cond, nullptr);
    if (mIrc != 0 || cIrc != 0) {
        cerr << "init failed" << endl;
    }
}

void Condition::destroy()
{
    int mIrc = pthread_mutex_destroy(&mutext);
    int cIrc = pthread_cond_destroy(&cond);
    if (mIrc != 0 || cIrc != 0) {
        cerr << "destroy failed" << endl;
    }
}

int Condition::lock()
{
    return pthread_mutex_lock(&mutext);
}

int Condition::unlock()
{
    return pthread_mutex_unlock(&mutext);
}

int Condition::wait()
{
    return pthread_cond_wait(&cond, &mutext);
}

int Condition::timeWait(timespec &waitTime)
{
    return pthread_cond_timedwait(&cond, &mutext, &waitTime);
}

int Condition::signal()
{
    return pthread_cond_signal(&cond);
}

int Condition::broadcast()
{
    return pthread_cond_broadcast(&cond);
}

//class ThreadPool，实现于ThreadPool.cpp文件

#include "ThreadPool.h"
#include <time.h>
#include <iostream>
using std::endl;
using std::cout;

void *threadRoutine(void *arg)
{
    ThreadPool *thPool = static_cast<ThreadPool *>(arg);
    bool exit = false; timespec timeOut = {0}; const time_t maxWaitTime = 5;
    while (!exit) {
        thPool->cond.lock(); //访问互斥资源时先加锁
        ++(thPool->idleThreadNum); //此时线程空闲，因此空闲线程数+1
	//测试是否有任务或者收到线程销毁通知
        while (thPool->taskQueue.empty() && !thPool->isDestroyNotify) {
            cout << "Thread " << pthread_self() << " is waiting." << endl;
            clock_gettime(CLOCK_REALTIME, &timeOut);
            timeOut.tv_sec += maxWaitTime;
            if (thPool->cond.timeWait(timeOut) == ETIMEDOUT) {
                cout << "Thread " << pthread_self() << " waiting timeout." << endl;
                exit = true; break; //超时退出
            }
        }
        cout << "Thread " << pthread_self() << " is working." << endl;
        --(thPool->idleThreadNum); //开始执行说明当前线程不再空闲
        if (!thPool->taskQueue.empty()) {
            TaskBase *tmpTask = thPool->taskQueue.front(); thPool->taskQueue.pop();
            thPool->cond.unlock();
            tmpTask->taskRun(); //执行具体任务时解锁
            thPool->cond.lock();
        }
	//如果没有任务且接到销毁或者退出信号
        if ((thPool->isDestroyNotify || exit) && thPool->taskQueue.empty()) {
            --(thPool->nowThreadNum);
            if (thPool->nowThreadNum <= 0) {
                thPool->cond.signal();
            }
            exit = true;
        }
        thPool->cond.unlock();
    }
    cout << "Thread " << pthread_self() << " exited." << endl;
    return arg;
}

ThreadPool::ThreadPool(int initNum, int maxNum)
        : initThreadNum(initNum), maxThreadNum(maxNum), nowThreadNum(0),
          idleThreadNum(0), isDestroyNotify(false)
{
    initThreadPool();
}

ThreadPool::~ThreadPool()
{
    destroyThreadPool();
}

void ThreadPool::addTask(TaskBase *tmpTask)
{
    if (tmpTask == nullptr) return;
    this->cond.lock();
    taskQueue.push(tmpTask);
    if (this->idleThreadNum > 0) {
        this->cond.signal();
    }
    else {
        if (this->nowThreadNum < this->maxThreadNum) {
            pthread_t threadID = 0;
            int irc = pthread_create(&threadID, nullptr, &threadRoutine, static_cast<void *>(this));
            if (irc == 0) {
                this->threadIDs.insert(this->threadIDs.end(), threadID); ++nowThreadNum;
            }
        }
    }
    this->cond.unlock();
}

void ThreadPool::initThreadPool()
{
    pthread_t threadID = 0; int irc = 0;
    this->cond.lock(); //创建时会读写互斥资源
    for (int i = 0; i < this->initThreadNum; i++) {
        irc = pthread_create(&threadID, nullptr, &threadRoutine, static_cast<void *>(this));
        if (irc == 0) {
            this->threadIDs.insert(this->threadIDs.end(), threadID); ++nowThreadNum;
        }
    }
    this->cond.unlock();
}

void ThreadPool::destroyThreadPool()
{
    this->cond.lock();
    if (!isDestroyNotify) {
        isDestroyNotify = true;
    }
    if (this->idleThreadNum > 0) {
        this->cond.broadcast();
    }
    if (this->nowThreadNum > 0) {
        while (this->nowThreadNum > 0) {
            this->cond.wait();
        }
    }
    auto iter = this->threadIDs.begin();
    for ( ; iter != this->threadIDs.end(); iter++) {
        pthread_join(*iter, nullptr);
    }
    this->cond.unlock();
}
```

测试代码如下所示：

```cpp
#include "ThreadPool.h"
#include <iostream>
#include <unistd.h>
using std::cout;
using std::endl;

class RealTask : public TaskBase
{
public:
    RealTask(string tmpName, void *outString) : TaskBase(tmpName, outString) {};
    void taskRun()
    {
        sleep(3);
        cout << static_cast<char *>(taskArg) << endl;
        cout << taskName << " finished.\n";
    }
};

int main()
{
    char workData[10][30] = {"0", "1", "2", "3", "4", "5", "6", "7", "8", "9"};
    RealTask task[10] = {{"Task0", workData[0]}, {"Task1", workData[1]}, {"Task2", workData[2]},
                         {"Task3", workData[3]}, {"Task4", workData[4]}, {"Task5", workData[5]},
                         {"Task6", workData[6]}, {"Task7", workData[7]}, {"Task8", workData[8]},
                         {"Task9", workData[9]}};
    ThreadPool testPool(2, 8);
    for (int i = 0; i < 10; i++) {
        testPool.addTask(&task[i]);
    }
    return 0;
}
```

线程池的代码可见：[一个简单的Linux线程池实现](https://github.com/hsuloong/simple_thread_pool)

## 参考文章

[线程池-维基百科](https://zh.wikipedia.org/zh-cn/线程池)

[C++线程池实现原理](https://yanyiwu.com/work/2015/12/16/cpp-thread-pool-programming.html)

[C++ 11简化线程池的实现](https://blog.csdn.net/shreck66/article/details/50412986)

[基于C++ 11的100行实现简单线程池](https://blog.csdn.net/gcola007/article/details/78750220)

[线程池原理及创建（C++实现）](https://www.cnblogs.com/lidabo/p/3328402.html)

[c++简单线程池实现](https://www.cnblogs.com/yangang92/p/5485868.html)