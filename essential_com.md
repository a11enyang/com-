## 第一章 com是一个更好的C++

### 动态链接和C++

![关系](https://raw.githubusercontent.com/wiki/a11enyang/Picture/essential_com_2020-03-26/Snipaste_2020-03-30_08-25-43.png) 

引出表：在运行时期将方法的名字解析到内存中对应的地址

import library：包含一些简单的引用，这些引用指向DLL的文件名和被引出的符号的名字



![运行模型](https://raw.githubusercontent.com/wiki/a11enyang/Picture/essential_com_2020-03-26/Snipaste_2020-03-26_09-42-20.png)

不同的程序包含各自的引入库，但是在计算机的硬盘中只有一个Dll文件



![实例](https://raw.githubusercontent.com/wiki/a11enyang/Picture/essential_com_2020-03-26/Snipaste_2020-03-26_09-55-41.png)

![实例](https://raw.githubusercontent.com/wiki/a11enyang/Picture/essential_com_2020-03-26/Snipaste_2020-03-26_09-59-42.png)

 

![抽象模型](https://raw.githubusercontent.com/wiki/a11enyang/Picture/essential_com_2020-03-26/Snipaste_2020-03-26_10-03-37.png)



### C++和可移植性

​	编译器编译程序的过程，Dll文件是单独存在的，运行时期程序编译,相关名字符号与Dll文件进行绑定，从而可以使用dll提供的方法。

​	？ 存在的问题：C++缺少二进制一级标准，各个厂家的编译标准是不一样的。当使用A编译器构建Dll文件时，如果在B编译器上使用是不能进行链接的，因为名字符号不同，无法找到相应的方法。



> 注意， extern C 是不能对类的成员函数进行使用的



### 封装性和C++

​	C++模型要求客户必须知道对象布局模型（理解：了解对象，从而为对象分配内存，但是不同计算机的内存结构是不同的，C++程序跟随计算机的变化而不同）。

​	简单地把C++类的定义从DLL文件中引出来的这种方案并不能提供合理的二进制组件结构。

  封装(encapsulation)的概念以 **“把一个对象的外观(接口)同其实际工作方式(实现)分离开来”** 为基础。 C++ 的问题在于这条原则没有被应用到二进制层次上，因为 C++ 类既是接口也是实现。

书中举了一个例子，给 FastString 增加一个成员 m_cch 用于记录字符串长度：



```cpp
// faststring.h 2.0 版
class __declspec(dllexport) FastString
{
    const int m_cch;    // 记录字符串长度，提高 Length 函数效率
    char* m_psz;
public:
    FastString(const char* psz);
    ~FastString();
    int Length(void) const;
    int Find(const char* psz) const;
}
```

在客户看来，上述改动在语法上是没有问题的， FastString 的 public 接口没有变。但在二进制层次上， FastString 对象的大小从 4 byte 变成了 8 byte. 如果客户程序还是像之前那样为 FastString 对象分配 4 byte 的内存，就极有可能引发异常。



> 参考文章：https://www.jianshu.com/p/eb19ab8d1aa8
>
> 参考文章：https://www.cnblogs.com/klb561/p/10555571.html

### 把接口从实现中分离出来

​	把接口和实现做成两个类。一个 C++ 类代表指向一定数据类型的接口，另一个 C++ 类作为数据类型的实际实现。这样一来，实现类可以被修改，而接口类保持不变。我们还需要一种方法把 接口类 和 实现类 联系起来

```c++
// faststringitf.h 接口类头文件
class __declspec(dllexport) FastStringItf
{
    class FastString;       // 导入 实现类 的名称
    FastString* m_pThis;    // 实现类指针，大小保持不变，客户不需要知道实现
public:
    FastStringItf(const char* psz);
    ~FastStringItf();
    int Length(void) const;
    int Find(const char* psz) const;
}

// faststringitf.cpp
FastStringItf::FastStringItf(const char* psz)
{
    m_pThis = new FastString(psz);
}

FastStringItf::~FastStringItf()
{
    delete m_pThis;
}

int FastStringItf::Length(void) const
{
    return m_pThis->Length();
}

int FastStringItf::Find(const char* psz) const
{
    return m_pThis->Find(psz);
}
```

缺点：每个方法增加了两个函数的调用（一个调用到接口，一个调用到实现部分），开销不理想；

而且句柄类并没有完全解决编译器链接器兼容的问题

### 抽象基类作为二进制接口



