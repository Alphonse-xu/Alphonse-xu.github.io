---
title: C++ Const关键字总结
date: 2020-03-20 10:49:01
tags:
     - 游戏开发
     - 编程技术
     - CPP
     - CPP面试
categories:
    - CPP
---

## C++ Const 关键字

Const 是修饰内置类型变量，自定义对象，成员函数，返回值，函数参数。

P.S.更多解释请直接阅读cppreference

## 1.内置类型变量

Const修饰全局变量时，编译器不允许修改定义好的变量。如果修改，则报错。

Const修饰局部变量时，编译器仍然不允许直接赋值修改，但是可以通过地址修改值。

``` C++
    const int  a = 7;
    int  *p = (int*)&a; //如果没有显示转换会报错：错误C2440: 无法从“ const int *”转换为“int * ”
    *p = 8;
    cout<<a;
```

调试可知a变量的值被改变为8，但是输出的结果仍然是 7。

在使用变量名输出时，编译器会出现一种类似宏定义的功能一样的行为，将变量名替换为初始值。可见，const局部变量并不能做到真正的不变，而是编译器对其进行了一些优化行为，这导致了const局部变量与真实值产生了不一致。

如果想获取修改后的const局部变量真实值，可以使用volatile关键字。volatile关键字使得程序每次直接去内存中读取变量值而不是读寄存器值，这个作用在解决一些不是程序而是由于别的原因修改了变量值时非常有用。

## 2.const 的指针与引用

### const修饰指针

指向常量的指针（pointer to const）/ (low-level const)

自身是常量的指针（常量指针，const pointer）/ (top-level const)

指针自身是一个对象，它的值为一个整数，表明指向对象的内存地址。因此指针长度所指向对象类型无关，在32位系统下为4字节，64位系统下为8字节。进而，指针本身是否是常量以及所指向的对象是否是常量就是两个独立的问题。

### const修饰引用

指向常量的引用（reference to const）

没有 const reference，因为引用本身就是 const pointer

## 3. const修饰函数参数传递

const修饰参数是为了防止函数体内可能会修改参数**原始对象**。因此，有三种情况可讨论：

1. 函数参数为值传递：值传递（pass-by-value）是传递一份参数的拷贝给函数，因此不论函数体代码如何运行，也只会修改拷贝而无法修改原始对象，这种情况不需要将参数声明为const。

2. 函数参数为指针：指针传递（pass-by-pointer）只会进行浅拷贝，拷贝一份指针给函数，而不会拷贝一份原始对象。因此，给指针参数加上顶层const可以防止指针指向被篡改，加上底层const可以防止指向对象被篡改。

3. 函数参数为引用：引用传递（pass-by-reference）有一个很重要的作用，由于引用就是对象的一个别名，因此不需要拷贝对象，减小了开销。这同时也导致可以通过修改引用直接修改原始对象（毕竟引用和原始对象其实是同一个东西），因此，大多数时候，推荐函数参数设置为pass-by-reference-to-const。给引用加上底层const，既可以减小拷贝开销，又可以防止修改底层所引用的对象。

## 4.对于 const 修饰函数的返回值

const修饰内置类型或者自定义类型的返回值，返回的值不能作为左值使用，既不能被赋值，也不能被修改。

## 5. const修饰成员函数

const修饰成员函数，是防止成员函数修改类对象的内容。良好的类接口设计应该确保，如果一个成员函数功能上不需要修改对象的内容，该成员函数应该加上const修饰。

如果const成员函数想修改成员变量值，可以用mutable修饰目标成员变量。

注意：如果一个类对象为const 对象，语义上说明该对象的值不可改变，因此该const对象只能调用const成员函数，因为非const成员函数不能保证不修改对象值，编译器会禁止这种会造成隐患的行为。

注意：const 关键字不能与 static 关键字同时使用，因为 static 关键字修饰静态成员函数，静态成员函数不含有 this 指针，即不能实例化，const 成员函数必须具体到某一实例。

---
总结与举例

1. 修饰变量，说明该变量不可以被改变；

2. 修饰指针，分为指向常量的指针（pointer to const）和自身是常量的指针（常量指针，const pointer）；

3. 修饰引用，指向常量的引用（reference to const），用于形参类型，即避免了拷贝，又避免了函数对值的修改；

4. 修饰成员函数，说明该成员函数内不能修改成员变量。

``` C++

// 类
class A
{
private:
    const int a;                // 常对象成员，只能在初始化列表赋值

public:
    // 构造函数
    A() : a(0) { };
    A(int x) : a(x) { };        // 初始化列表

    // const可用于对重载函数的区分
    int getValue();             // 普通成员函数
    int getValue() const;       // 常成员函数，不得修改类中的任何数据成员的值
};

void function()
{
    // 对象
    A b;                        // 普通对象，可以调用全部成员函数、更新常成员变量
    const A a;                  // 常对象，只能调用常成员函数
    const A *p = &a;            // 指针变量，指向常对象
    const A &q = a;             // 指向常对象的引用

    // 指针
    char greeting[] = "Hello";
    char* p1 = greeting;                // 指针变量，指向字符数组变量
    const char* p2 = greeting;          // 指针变量，指向字符数组常量（const 后面是 char，说明指向的字符（char）不可改变）
    char* const p3 = greeting;          // 自身是常量的指针，指向字符数组变量（const 后面是 p3，说明 p3 指针自身不可改变）
    const char* const p4 = greeting;    // 自身是常量的指针，指向字符数组常量
}

// 函数
void function1(const int Var);           // 传递过来的参数在函数内不可变
void function2(const char* Var);         // 参数指针所指内容为常量
void function3(char* const Var);         // 参数指针为常量
void function4(const int& Var);          // 引用参数在函数内为常量

// 函数返回值
const int function5();      // 返回一个常数
const int* function6();     // 返回一个指向常量的指针变量，使用：const int *p = function6();
int* const function7();     // 返回一个指向变量的常指针，使用：int* const p = function7();
```

---

## 参考来源

<https://www.runoob.com/w3cnote/cpp-const-keyword.html>

<https://interview.huihut.com/#/?id=const-%e7%9a%84%e6%8c%87%e9%92%88%e4%b8%8e%e5%bc%95%e7%94%a8>

<https://blog.csdn.net/u011333734/article/details/81294043>
