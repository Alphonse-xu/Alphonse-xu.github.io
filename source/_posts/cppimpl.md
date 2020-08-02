---
title: C++编程方法：Pimpl
date: 2020-03-09 20:27:26
tags:
     - 游戏开发
     - 编程技术
     - CPP
categories:
    - CPP
---

最近参与制作游戏需要跨平台，发现windows和ps4的API有很大的不同。为了代码复用，学习了Pimpl方法，可以较好的解决这个问题。

P.S.更多解释请直接阅读cppreference

## 什么是Pimpl

Pimpl（pointer to implementation）是“指向实现的指针”。

Pimpl利用了c++的一个特点，即可以将类的数据成员定义为指向某个已经声明过的类型的指针，这里的类型仅仅作为名字引入，并没有被完整地定义。

该技巧可以避免在头文件中暴露私有细节，因此是促进API接口与实现保持完全分离的重要机制。

此外由于声明了析构函数，导致默认的移动构造/赋值函数不能生成，若默认行为符合自己的需求，则需显式声明 = default（当只在.h里，Impl是个不完整的类型，所以无法在.h类直接 = default，而是在.h声明，在.cpp使= default）

若需要给类提供拷贝性质的函数，需要额外花点心思处理std::unique_ptr(该智能指针不支持拷贝)。

![pimpl](https://github.com/Alphonse-xu/Alphonse-xu.github.io/blob/master/image/pimpl.png)

``` C++
// my_class.h
#pragma once
#include <memory>

class my_class {
    //  ... 所有的公有/保护接口都可以放在这里 ...
    my_class();
    ~my_class();
    my_class(my_class&& v);    //移动构造
    my_class& operator=(my_class&& v);    //移动赋值
private:
    class Impl; 
    std::unique_ptr<Impl> pimpl;
};
```

```C++
// my_class.cpp
// ...include其它要依赖的头文件...
#include "my_class.h"

class my_class::Impl {
  // 在这里定义所有私有变量和方法(换句话说是my_class类的具体实现细节内容)
  // 现在可以改变实现，而依赖my_class.h的其他类无需重新编译...
};

my_class::my_class():pimpl(std::make_unique<Impl>()){
    // ...初始化pimpl... 
}
my_class::~my_class() = default;
my_class::my_class(my_class&& v) = default;
my_class::my_class& operator=(my_class&& v) = default;
```

## 举例

例如“自动定时器”的API， 它是一个具名对象，当对象被销毁时打印出其生存时间。

AutoTimer.h 如下:

``` C++
#ifdef _WIN32
#include <windows.h>
#else
#include <sys/time.h>
#endif
#include <string>
class AutoTimer
{
    public:
        explicit AutoTimer(const std::string name);
        ~AutoTimer();
    private:
        double getElapsed() const;
        std::string mName;

    #ifdef _WIN32
        DWORD mStartTime;
    #else
        struct timeval mStartTime;
    #endif
};
```

从上面的h文件可知，该API暴露了定时器在不同平台上存储的底层细节，利用Pimpl特性， 可轻易的隐藏掉这些底层实现的细节， 利用Pimpl特效后新的h文件如下:

``` C++
#include <string>

class AutoTimer{
public:
    explicit AutoTimer(const std::string& name);
    ~AutoTimer();
private:
    class Impl;
    Impl* mImpl;
};
```

新API更简洁， 没有任何平台相关的预处理指令， 他人也不能通过头文件了解类的任何私有成员。

现在AutoTimer的功能实现都放在内嵌类Impl中了， 具体的AutoTimer的cpp文件如下:

```C++
#include "AutoTimer.h"

#ifdef _WIN32
#include <windows.h>
#else

#include <sys/time.h>
#include <iostream>

#endif

class AutoTimer::Impl
{
public:
    double getElapsed() const
    {
#ifdef _WIN32
        return (GetTickCount() - mStartTime) / 1e3;
#else
        struct timeval end_time;
        gettimeofday(&end_time, NULL);
        double t1 = mStartTime.tv_usec / 1e6 + mStartTime.tv_sec;
        double t2 = end_time.tv_usec / 1e6 + end_time.tv_sec;
        return t2 - t1;
#endif
    }

    std::string mName;
#ifdef _WIN32
    DWORD mStartTime;
#else
    struct timeval mStartTime;
#endif

};

AutoTimer::AutoTimer(const std::string &name)
: mImpl(new AutoTimer::Impl())
{
    mImpl->mName = name;
#ifdef _WIN32
    mImpl->mStartTime = GetTickCount();
#else
    gettimeofday(&mImpl->mStartTime, NULL);
#endif
}

AutoTimer::~AutoTimer()
{
    std::cout << mImpl->mName << ":took " << mImpl->getElapsed() << " secs" << std::endl;
    delete mImpl;
    mImpl = NULL;
}
```

将Impl类声明为AutoTimer类的私有内嵌类， 可以避免与该实现相关的符号污染全局命名空间。

## 总结

### Pimpl的优点

- 接口和实现的分离，私有成员完全可以隐藏在共有接口之外，给用户一个间接明了的使用接口，尤其适合闭源API设计。

- 降低耦合，对Impl类进行修改，无需重新编译原类

- 加速编译，减少原类不必要的头文件的依赖

- 更好的二进制兼容性。二进制兼容性指的是在升级库文件的时候（windows下的dll， uinx下的so），不必重新编译使用这个库的可执行文件或使用这个库的其他库文件，程序的功能不被破坏

- 可使用惰性分配技术：类的某部分实现可以写成按需分配或者实际使用时再分配，从而节省资源

### 缺点

- 添加了一层封装， 可能进入性能问题

- 代码可读性降低

---

## 参考来源

<https://zh.cppreference.com/w/cpp/language/pimpl>

<https://www.jianshu.com/p/b2be3fb1e1d1> 

<https://www.cnblogs.com/KillerAery/p/9539705.html>

<https://goodspeedlee.blogspot.com/2016/01/c-pimpl.html>

<https://blog.csdn.net/lihao21/article/details/47610309>
