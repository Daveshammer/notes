## C ++

**new和malloc**

- `new` 可以认为是 `malloc` + 构造函数。
- `new`是运算符，可以重载；`malloc`是函数，不能重载。
- `new`内存分配失败时，会抛出`bad_alloc`异常；`malloc`内存分配失败返回`nullptr`。

**free后的内存状态**

- 调用free(p) 之后，只是将p所指向的之前申请的内存区域标志为已回收，p指向的内存的值不确定，有可能变，也有可能不变。

**new执行时做了哪些事？**

- 先调用`operator new`分配内存，主要是对`malloc`的封装

- 再调用构造函数构造对象

**`malloc(0)`会发生什么**

- 会为其分配最小内存，64位机器下为8字节

**`final`和`override`的作用？`final`为什么能提高代码执行效率？**

- `override`：保护派生类中声明的重载函数，与基类的虚函数有相同的签名，用于编译器代码检查

- `final`：阻止类的进一步派生；阻止虚函数的进一步重写，这样可以在编译期间确定虚函数的调用版本，避免了动态绑定的开销

**`static`的作用**

- 用于函数内部修饰变量，即函数内的静态变量。这种变量的生存期长于该函数，使得函数具有一定的“状态”。
- 用在文件级别（函数体之外），修饰变量或函数，表示该变量或函数只在本文件可见，其他文件看不到、也访问不到该变量或函数。
- 用于修饰class的数据成员，即所谓“静态成员”。这种数据成员的生存期大于class的对象（实体／instance）。静态数据成员是每个class有一份，普通数据成员是每个instance有一份，因此也分别叫做class variable和instance variable。
- 用于修饰class的成员函数，即所谓“静态成员函数”。这种成员函数只能访问class variable和其他静态程序函数，不能访问instance variable或instance method。

**sizeof和strlen的区别**

- `sizeof`是一个操作符，`strlen`是库函数

- `sizeof`的参数可以是数据的类型，也可以是变量；而`strlen`只能以`\0`的字符串作为参数

- `sizeof`在编译时计算结果，`strlen`在运行时计算结果

- `sizeof`计算的是数据类型占用内存的大小，`strlen`是字符串实际的长度

**结构体对齐规则**

- 先确定对齐单位，由一下三个因素决定
  
  - CPU周期：WIN默认8字节；Linux 32位默认4字节，64位默认8字节。
  
  - 结构体最大成员（基本数据类型变量）
  
  - 预编译指令`#pragma pack(n)`手动设置，一般为1、2、4、8、16

- 除结构体第一个成员外，其它所有成员的地址相对与**结构体地址**（即首个成员的地址）的偏移量必须为**实际对齐单位**或**自身大小**的整数倍

- 结构体的整体大小必须是**实际对齐单位**的整数倍

**memcpy和strcpy区别**

- 复制内容：`strcpy`只能复制字符串，`memcpy`可以复制任意内容

- 复制方法：`strcpy`不需要指定长度，遇到`\0`就会结束；而`memcpy`根据第3个参数决定了复制的长度

**为什么`noexcept`能提高性能**

- 为实现异常捕获的功能，C++引入了“**栈回退**”机制，编译器会为函数额外生成“栈回退”的代码，使用`noexcept`可以避免生成额外的代码来处理异常情况

**`new`，`placement new`，`operator new`的区别？如何把对象`new`在栈上**

- `operator new`作用是分配一块内存，`placement new`作用是在已分配内存地址处，创建一个对象，`new`的作用则等于`operator new` + `placement new`。

- 先在栈上声明一个数组，然后通过`placement new` 在这段地址处创建对象，这就实现了在栈上`new`一个对象。

**变量声明和定义的区别？**

- 定义为变量分配地址和存储空间，声明不分配分配地址。
- 一个变量可以在多个地方声明，但只能在一个地方定义。

**构造函数可以是虚函数吗？**

- 构造函数不能是虚函数，从⽬前编译器实现虚函数进⾏多态的⽅式来看，虚函数的调⽤是通过实例化之后对象的虚函数表指针来找到虚函数的地址进⾏调⽤的，如果说构造函数是虚的，那么虚函数表指针则是不存在的，⽆法找到对应的虚函数表来调⽤虚函数，那么这个调⽤实际上也是违反了先实例化后调⽤的准则。

- 语法上通过，语义上有问题。  derived class对象内的base class成分会在derived class自身构造之前构造完毕。因此，在base class的构造函数中执行的virtual函数将会是base class的版本，决不会是derived class的版本。

**类成员函数模板可以是虚函数吗？**

- 类成员函数模板不能是虚函数，因为C++在链接前是不知道成员函数模板被实例化多少次的，这就会导致编译器无法在编译期间确定虚表的大小。

**虚函数表存放在内存的什么区?**

- 虚函数则位于代码段`(.text)`，也就是C++内存模型中的代码区。

- 一个类里面定义了虚函数，那么编译阶段，编译器给这个类类型产生一个唯一的vftable虚函数表，里面存放着RTTI指针和虚函数的地址。

- 当程序运行时，每一张虚函数表都会加载到内存的`.rodata`区。

**成员函数指针和普通函数指针区别？**

- 通常来说，函数指针的长度等于机器字长，而成员函数指针长度比函数指针更长，其内部存放了对象地址和成员函数地址信息。在没有给出对象地址的情况下，调用成员函数指针会报错。

**函数指针和指针函数**

- 指针函数，就是一个返回指针的函数，其本质是一个函数，而该函数的返回值是一个指针，如`int *fun(int x,int y);`和`fun = Function;`

- 函数指针，其本质是一个指针变量，该指针指向这个函数，如`int (*fun)(int x,int y);`

**悬空指针和野指针**

- 悬空指针产生于指针指向的内存被释放后，该指针未置为空、**而仍指向原地址**，如此会导致潜在的安全隐患和不可预知的错误。
  
  - **局部变量的引用：** 当函数执行完毕后，局部变量的存储空间将被系统回收。如果有一个指针变量指向了一个局部变量，当函数返回后，这个指针即成为悬空指针。
  
  - **动态内存释放后的引用：** 使用`malloc()`或`calloc()`分配的内存，在调用`free()`释放后，如果仍旧有指针指向该内存地址，这些指针即变为悬空指针。

- 野指针则是指向非法或随机内存地址的指针，其通常源于未初始化的指针变量。
  
  - **指针变量未初始化：** 新声明的指针变量如果没有明确的初始化，则内容为随机值，这种情况下的指针称为野指针。
  
  - **内存越界访问：** 指针操作超过了分配的内存范围，可能导致指向不可知的内存区域，成为野指针。

**避免和处理悬空指针的方法**

- **立即将释放的指针置为NULL**：这是确保它不再是悬空指针的简单有效方法。

- **使用智能指针**：在使用C++等高级语言时，可通过智能指针（如`std::shared_ptr`、`std::unique_ptr`等）来自动管理内存，避免悬空指针的产生。

**避免和处理野指针的方法**

- **指针声明时初始化**：对所有指针进行初始化，可以是直接赋值为NULL或者有效的内存地址。

- **确保指针在生命周期结束前有效**：注意指针赋值的作用域和时间点。

**`inline`的作用和原理？**

- C++17以前，`inline`关键字主要有两个作用：第一，作为内联优化建议，只不过是否展开函数还是由编译器决定；第二，解决符号重定义问题，不同文件内定义了同签名的函数，若被inline关键字修饰，则不会引发符号重定义错误。

- c++17开始，inline只保留第二个作用，若用户希望函数内联展开，则可以使用`__attribute((always_inline))__` 关键字，它是 GCC 和 Clang 中的一个扩展，用于强制内联函数。

- 原理上，第一，内联展开相比于普通函数调用，少了函数上下文压栈的过程，因此效率更高，缺点就是容易引起代码膨胀。第二，被`inline`关键字修饰的函数名，编译期间会被标记为weak符号，链接目标文件的时候，多个同签名weak符号不会引发编译器报错，运行期间，会选取其中一个函数进行调用。

**内存泄漏**

- 指程序在申请内存后，无法释放已申请的内存空间

- 造成内存泄漏的三种情况
  
  - 指针重新赋值
  
  - 错误的内存释放（仅释放部分内存）
  
  - 返回值的不正确处理（并未处理该内存位置的返回地址）

- 如何避免内存泄漏
  
  - 确保没有在访问空指针
  
  - 每个内存分配函数都应该有一个 `free` 函数与之对应，`alloca` 函数除外
  
  - 始终正确处理返回动态分配的内存引用的函数返回值

**virtual函数能声明为内联吗？**

- 任何函数都可以被`inline`修饰，包括构造函数、析构函数、虚函数。这里提一下为什么虚函数可以内联，`inline`函数涉及到的是编译期解析，虚函数地址大多数情况下在运行期解析，但是某些情况下，具体调用哪个虚函数可以在编译期间确定，这个时候虚函数就能内联展开了。  
- `inline`只作为内联建议，是否展开由编译器决定，因此是可以获取`inline`函数指针的。

**`static inline`和`extern inline`含义？**

- `static inline`指的是具有文件作用域的`inline`函数；`extern inline`作用比较特殊，外部单元把它当作普通函数进行调用，同单元内把它当作`inline`函数调用。

**指针和引用的区别？**

- 引用可以看作是指针常量(指针本身是个常量，但是指向的内容可以修改)，只能在声明的时候初始化，相比于指针，引用的优势在于编译器帮我们检查地址是否初始化。
- 程序在编译时分别将指针和引用添加到符号表上，符号表上记录的是变量名及变量所对应地址。指针变量在符号表上对应的地址值为指针变量的地址值，而引用在符号表上对应的地址值为引用对象的地址值。符号表生成后就不会再改，因此指针可以改变其指向的对象（指针变量中的值可以改），而引用对象则不能修改。

**指针和引用作为形参的区别？**

- 空指针传递：指针形参可以接受空指针作为实参，而引用形参不允许为空。
- 可以修改指针本身：指针形参可以修改指针本身的值，例如将指针重新指向其他对象，而引用形参无法修改引用本身。
- 存在空间要求：指针形参需要占用额外的内存空间来存储地址，而引用形参不需要额外的内存空间。

**右值引用与移动语义**

- 将亡值用来出发移动构造或者移动赋值，并进行资源转移，之后将调用析构函数。

- 左值可以通过`std::move()`转换为右值，来避免资源的重新分配。

**万能引用与完美转发**

- 完美转发`std::forward<T>(t)`借助于万能引用，可以保证被转发的参数左右值属性不变。

- 如果没有`move`和`forward`，任何实参传递给函数后，都会变成左值类型，这就会造成参数类型为右值引用的函数永远无法被调用。

**万能引用与引用折叠**

- 参数以`int`类型为例，参数为左值或者左值引用，`T &&`将转换为`int &`。

- 参数为右值或者右值引用，`T &&`将转换为`int &&`。

**可以在运行时访问private成员吗？**

- 可以，访问权限关键字只在编译期有效，运行期是没有访问权限关键字这些概念的。

**继承基类成员访问方式的变化**

```
类成员/继承方式   | public继承          | protected继承      | private继承
基类public成员   | 派生类的public成员   | 派生类的protected成员| 派生类的private成员
基类protected成员| 派生类的protected成员| 派生类的protected成员| 派生类的private成员
基类private成员  | 派生类中不可见        | 派生类中不可见       | 派生类中不可见  
```

**动态库和静态库的区别？知道动态库延迟加载优化吗？**

- 链接动态库和静态库的时候，静态库会被复制到可执行程序当中，而动态库不会。相比动态库，静态库的执行效率更高，但占用磁盘空间更多，不方便更新。动态库的延迟加载指的是，在运行时按需加载动态链接库中的函数和数据，而不是在启动的时候加载库函数和数据，从而降低启动时间，在linux系统下，延迟加载是通过PLT表和GOT表配合实现的。

**可变参数函数的实现原理？**

- 可变参数函数在编译期间，可以获取到实际参数数量和参数类型；可变参数函数运行的时候，会根据编译期间获取到的类型和大小信息，分配相应大小的栈空间，把参数按照从右到左的顺序依次压入当前函数的栈帧；当前栈帧的地址完全是可以获取到的，出于安全考虑，获取参数值前，需要通过`va_start`把参数值拷贝到`va_list`指向的堆空间，遍历`va_list`就可以获取到每个参数的值了。

**`any`可以替代`void*`吗？**

- 可以，与`void*`相比，`any`优势在于：第一，其它类型被转换成`void*`后，类型信息会发生丢失，而`any`会存储类型信息，进行类型转化时更安全；第二，`any`析构时，会自动析构堆上的对象，而`void*`需要手动管理内存。

- 劣势在于：第一，`any`内存占用更高；第二，进行类型转化，会进行类型检查，效率更低，但这点劣势可忽略不计。

**`variant`可以替代`union`吗？**

- 可以，`variant`通过可变参数模板和递归`union`的方式实现，`variant`实例化对象内存放有数据类型信息。相比于`union`，`variant`优势在于：第一，可以存储复杂类型，而`union`只能存储trivial类型；第二，`variant`存储了数据的类型信息，可以进行安全的类型转换。劣势在于，内存更大、访问效率更低。

**`sizeof(function)`大小是多少？`function`的实现方式？**

- `std::function`的大小等于2个机器字长，这是因为`std::function`可以封装任何可调用对象，其中普通函数指针长度为1个机器字长，成员函数指针长度为2个机器字长，因此为了能够封装任何可调用对象，其内存大小为2个机器字长。`std::function`内部封装有一个函数指针指向可调用对象，然后重载实现构造函数、`=`运算符、`()`运算符。

 **`function`的作用**

- function的作用是提供一个通用的函数对象包装器，可以用于存储和调用任何可调用对象，包括函数指针、函数对象和Lambda表达式。

- function提供了一种统一的接口来处理不同类型的函数对象，使得它们可以像普通函数一样进行调用。

- function具有类型擦除的特性，允许我们在运行时存储和传递不同类型的函数对象。

**模板的全特化和偏特化是什么？**

- 首先，模板函数只有全特化，没有偏特化，模板类有全特化和偏特化。模板全特化指的是，在元编程中给定所有模板参数的具体类型；模板偏特化也称为部分特化，指的是在元编程中给定部分模板参数的具体类型。引入特化和偏特化是为了给特定的类型提供单独的实现。

**模板的匹配规则是什么？**

- 模板函数的匹配顺序是：首先根据函数名进行匹配；若找到多个函数名匹配的模板，再根据参数列表进行匹配，这种匹配过程被叫做重载决议。

- 模板类的匹配顺序是：首先根据类名进行匹配；若找到多个类名匹配的模板，再按照全特化、偏特化、通用模板的的优先级进行匹配。

**`push_back()`和`emplace_back()`区别？**

- 假设容器内存放的元素类型是`T`，其定义了`T(int)`构造函数，`T(const T&)`拷贝构造，`T(T&&)`移动构造函数。首先，`push_back()`是单参数函数，参数类型是`const T&`或者`T&&`，`emplace_back()`是不定参数函数，其参数列表必须和T的普通构造函数、拷贝构造函数或者移动构造函数的参数相同；第二，`emplace_back()`通过可变参数成员函数模板实现，其编译时长比`push_back()`更长；第三，`push_back()`和`emplace_back()`的执行效率完全相同，只有一种例外情况：`push_back(2)`比`emplace_back(2)`多执行一次普通构造函数，因为前者需要通过构造函数把`2`转化为`T`类型，然后调用`push_back()`函数。

 **线程和协程的区别？为什么引入协程？**

- 两者区别是，第一，线程的调度需要切换到内核态，由操作系统完成，协程的调度在用户态完成，由用户程序程序进行调度；

- 第二，抢占式协程和抢占式线程的实现原理不同，抢占式协程由编译器插入时间片或者由操作系统信号实现，线程的抢占通过时间片中断实现。引入协程是为了实现异步非阻塞编程，传统的线程在异步资源返回前，往往会阻塞当前线程，但是引入协程后，在异步资源返回前，当前线程不必阻塞，而可以执行其它协程中的任务。

**条件变量的实现原理？**

- 互斥锁只能实现对资源的互斥访问，而不能实现线程同步，引入条件变量就是为了实现线程同步。条件变量是基于互斥锁和等待队列实现的，`wait()`调用后，若发现条件变量没有被占用，则继续执行，若发现条件变量被占用，则释放锁并阻塞当前线程，把线程放到等待队列上；`notify()`调用后，会唤醒阻塞的线程。

**条件变量和信号量区别？**

- 首先，信号量是通过互斥锁、等待队列、计数器实现的，条件变量是通过互斥锁和等待队列实现，没有计数功能；第二，信号量既可以充当互斥锁，也可以充当条件变量；第三，如果等待队列上没有任务，信号量调用notify后，信号会被保存，条件变量调用notify后，信号会丢失；通过信号量可以实现线程同步，条件变量需要和互斥锁配合使用才能实现线程同步，前者使用起来更简单，后者内存占用和性能更好。

**为什么要在父线程执行`detach()`或者`join()`？**

- 调用`detach()`或者`join()`后，子线程的状态会从`unjoinable`变为`joinable`，父线程执行结束的时候，若发现子线程状态是`joinable`则会调用`terminate()`终止子线程，反之则什么也不做。在父进程内不调用`detach()`或者`join()`，如果父线程执行完子线程还没执行完，会导致子线程异常终止。

**多态**

- 多态，即多种状态（形态）。简单来说，我们可以将多态定义为消息以多种形式显示的能力。
- 多态是以封装和继承为基础的。
- C++ 多态分类及实现：
  1. 重载多态（Ad-hoc Polymorphism，编译期）：函数重载、运算符重载
  2. 子类型多态（Subtype Polymorphism，运行期）：虚函数
  3. 参数多态性（Parametric Polymorphism，编译期）：类模板、函数模板
  4. 强制多态（Coercion Polymorphism，编译期/运行期）：基本类型转换、自定义类型转换

**如何定义一个只能在堆上（栈上）生成对象的类？**

**只能在堆上**

- 方法：将析构函数设置为私有

- 原因：C++ 是静态绑定语言，编译器管理栈上对象的生命周期，编译器在为类对象分配栈空间时，会先检查类的析构函数的访问性。若析构函数不可访问，则不能在栈上创建对象。

**只能在栈上**

- 方法：将 new 和 delete 重载为私有

- 原因：在堆上生成对象，使用 new 关键词操作，其过程分为两阶段：第一阶段，使用 new 在堆上寻找可用内存，分配给对象；第二阶段，调用构造函数生成对象。将 new 操作设置为私有，那么第一阶段就无法完成，就不能够在堆上生成对象。

**智能指针的线程安全问题：**

主要是以下几个方面：  

- **引用计数**的加减操作是线程安全的，底层使用的是`atomic`原子变量。  

-  **修改shared_ptr指向**不是线程安全的。  
  
  - 对于shared_ptr的复制则分为两个步骤：
    
    1. 复制ptr指针
    
    2. 复制引用计数指针

**不要使用裸指针初始化智能指针？**

- 因为可能存在同一个裸指针初始了多个智能指针，在智能指针析构时会造成资源的多次释放。

**知道在释放资源的时候`shared_ptr`和`unique_ptr`有什么不同吗？**

- `unique_ptr`调用了`reset()`之后就会直接释放掉，`shared_ptr`则会在所有引用计数变为0的时候才会释放申请的内存

- `unique_ptr`的`release()`方法，并不会释放资源，只会把`unique_ptr`置为空指针，原来那个资源可以继续调用

**借助shared_ptr实现copy-on-write**

- `shared_ptr::unique()`来判断是不是有人在读，如果有人在读，那么我们不能直接修改，因为read并没有全程加锁，只在获取`g_foos`时有锁
- 精髓：
  - 如果你是数据的唯一拥有者，那么你可以直接修改数据。   
  - 如果你不是数据的唯一拥有者，那么你拷贝它之后再修改。

```cpp
using FooList = std::vector<Foo>;
using FooListPtr = std::shared_ptr<FooList>;
std::mutex mutex;
FooListPtr g_foos;


void read()
{
    FooListPtr foos;
    {
        lock_guard<std::mutex> lock(mutex);
        foos = g_foos;
    }
    for (auto it : foos)
    {
        it->doit();
    }
}
void write(const Foo& f) //多个写线程，全程加锁
{
    lock_guard<std::mutex> lock(mutex);
    if (!g_foos.unique())
    {
        // g_foos.reset(new FooList(*g_foos));
        FooListPtr newPtr = new FooList(*g_foos);
        g_foos.swap(newPtr);
    }
    g_foos->push_back(f);
}
void write(const Foo& f) //仅有单个写线程，交换前加锁
{
    FooListPtr newPtr = new FooList(*g_foos);
    newPtr.push_back(f);
    if (newPtr)
    {
        lock_guard<std::mutex> lock(mutex);
        g_foos.swap(newPtr);
    }
}
```

**`lock_guard`和`unique_lock`区别？**

- 所有权
  
  - `unique_lock`可以手动解锁，`lock_guard`只能出作用域自动解锁

- 条件遍历的支持
  
  - 由于 `lock_guard` 无法手动解锁，因此无法满足条件变量等待和通知的需求。
  
  - `unique_lock`支持与条件变量一起使用。

**constexpr和const的区别？**

- constexpr 只能定义编译期常量，必须在编译时计算和初始化，⽽ const 可以定义编译期常量，也可以定义运⾏期常量。

**四种类型转换：**

- `static_cast`
  
  - 用于基本数据类型之间的转换，以及把任意表达式类型转换成`void`类型
  
  - 进行上行转换（派生类指针或引用转换为基类的）是安全的，由于没有动态类型检查，所以进行下行转换是不安全的

- `dynamic_cast`
  
  - 用于父子关系的强制类型转换，在下行转换时，具有类型检查功能
  - 基类一定需要虚函数：运行时转换需要类对象的信息，而类对象的信息存储在虚函数表中

- `reinterpret_cast`
  
  - 转换用于任意指针（引用）类型之间的转换，不进行类型检查。

- `const_cast`
  
  - 去掉指针或者引用的const或volatile属性

**智能指针的上下类型转换**

`static_pointer_cast`、`dynamic_pointer_cast`、`const_pointer_cast` `reinterpret_pointer_cast`

**main运行前可运行哪些代码**

- 全局对象的构造函数

- 全局变量、对象和静态变量、对象的空间分配和赋初值
  
  - 全局变量的赋值函数
  
  - 全局lambda变量调用

- 通过关键字__attribute__

**coredump文件**

- gdb可以用于分析coredump文件，文件含有进程被终止时内存/CPU寄存器和各种函数调用栈的信息；

- 产生coredump文件的原因：
  
  - 内存访问越界
  
  - 多线程使用了线程不安全的函数
  
  - 多线程读写的数据未加锁保护
  
  - 栈溢出

- core文件没有符号表信息，必须结合可执行文件才可调试

**线程安全的内存池**

## 网络

### TCP连接与关闭

**三次握手过程**

![](assets/2023-08-29-10-26-33-image.png)

**为什么是三次握手？**

- 三次握手才可以阻止重复历史连接的初始化（主要原因）
  
  - 如果采用两次握手建立 TCP 连接的场景下，服务端在向客户端发送数据前，并没有阻止掉历史连接，导致服务端建立了一个历史连接，又白白发送了数据，妥妥地浪费了服务端的资源。要解决这种现象，最好就是在服务端发送数据前，也就是**建立连接之前，要阻止掉历史连接**，这样就不会造成资源浪费，而要实现这个功能，就需要三次握手。

- 三次握手才可以同步双方的初始序列号
  
  - 接收方可以去除重复的数据；接收方可以根据数据包的序列号按序接收；可以标识发送出去的数据包中， 哪些是已经被对方收到的。

- 三次握手才可以避免资源浪费
  
  - 如果是**两次握手**，客户端发送的 `SYN` 报文在网络中阻塞了，重复发送多次 `SYN` 报文，那么服务端在收到请求后就会**建立多个冗余的无效链接，造成不必要的资源浪费。**

![](assets/2023-08-30-21-42-49-image.png)

**什么是 SYN 攻击？**

我们都知道 TCP 连接建立是需要三次握手，假设攻击者短时间伪造不同 IP 地址的 `SYN` 报文，服务端每接收到一个 `SYN` 报文，就进入`SYN_RCVD` 状态，但服务端发送出去的 `ACK + SYN` 报文，无法得到未知 IP 主机的 `ACK` 应答，久而久之就会**占满服务端的半连接队列**，使得服务端不能为正常用户服务。

**如何避免 SYN 攻击？**

- 调大 netdev_max_backlog；
  - 当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这些数据包。
- 增大 TCP 半连接队列；
- 开启 tcp_syncookies；
  - 开启 syncookies 功能就可以在不使用 SYN 半连接队列的情况下成功建立连接，相当于绕过了 SYN 半连接来建立连接。
- 减少 SYN+ACK 重传次数

**四次挥手过程**

![](assets/2023-08-29-10-27-16-image.png)

**为什么挥手需要四次？**

- 关闭连接时，客户端向服务端发送 `FIN` 时，仅仅表示客户端不再发送数据了但是还能接收数据。
- 服务端收到客户端的 `FIN` 报文时，先回一个 `ACK` 应答报文，而服务端可能还有数据需要处理和发送，等服务端不再发送数据时，才发送 `FIN` 报文给客户端来表示同意现在关闭连接。

**为什么 TIME_WAIT 等待的时间是 2MSL？**

- `MSL` 是 Maximum Segment Lifetime，**报文最大生存时间**，它是任何报文在网络上存在的最长时间，超过这个时间报文将被丢弃。

- 如果被动关闭方没有收到断开连接的最后的 ACK 报文，就会触发超时重发 `FIN` 报文，另一方接收到 FIN 后，会重发 ACK 给被动关闭方， 一来一去正好 2 个 MSL。可以看到 **2MSL时长** 这其实是相当于**至少允许报文丢失一次**。

**为什么需要 TIME_WAIT 状态？**

- 这个时间足以让两个方向上的数据包都被丢弃，使得原来连接的数据包在网络中都自然消失，再出现的数据包一定都是新建立连接所产生的。

- 等待足够的时间以确保最后的 ACK 能让被动关闭方接收，从而帮助其正常关闭。

**TIME_WAIT 过多有什么危害？**

- 第一是占用系统资源，比如文件描述符、内存资源、CPU 资源、线程资源等；
- 第二是占用端口资源，端口资源也是有限的，一般可以开启的端口为 `32768～61000`，也可以通过 `net.ipv4.ip_local_port_range`参数指定范围。

**服务器出现大量 TIME_WAIT 状态的原因有哪些？**

首先要知道 **TIME_WAIT 状态是主动关闭连接方才会出现的状态**，所以如果服务器出现大量的 TIME_WAIT 状态的 TCP 连接，就是说明服务器主动断开了很多 TCP 连接。

- 第一个场景：HTTP 没有使用长连接

- 第二个场景：HTTP 长连接超时，nginx触发回调关闭超时连接

- 第三个场景：HTTP 长连接的请求数量达到上限，此时nginx 会主动关闭这个长连接

**服务器出现大量 CLOSE_WAIT 状态的原因有哪些？**

CLOSE_WAIT 状态在服务器停留时间很短，如果你发现大量的 CLOSE_WAIT 状态，那么就意味着被动关闭的一方没有及时发出 FIN 包，一般有如下几种可能：

- 程序问题：如果代码层面忘记了 close 相应的 socket 连接，那么自然不会发出 FIN 包，从而导致 CLOSE_WAIT 累积；或者代码不严谨，出现死循环之类的问题，导致即便后面写了 close 也永远执行不到。

- 响应太慢或者超时设置过小：如果连接双方不和谐，一方不耐烦直接 timeout，另一方却还在忙于耗时逻辑，就会导致 close 被延后。响应太慢是首要问题，不过换个角度看，也可能是 timeout 设置过小。

**如果已经建立了连接，但是服务端的进程崩溃会发生什么？**

TCP 的连接信息是由内核维护的，所以当服务端的进程崩溃后，内核需要回收该进程的所有 TCP 连接资源，于是内核会发送第一次挥手 FIN 报文，后续的挥手过程也都是在内核完成，并不需要进程的参与，所以即使服务端的进程退出了，还是能与客户端完成 TCP 四次挥手的过程。

### Socket 编程

![](assets/2023-08-29-12-32-05-image.png)

Linux内核中会维护两个队列：

- 半连接队列（SYN 队列）：接收到一个 SYN 建立连接请求，处于 SYN_RCVD 状态；
- 全连接队列（Accpet 队列）：已完成 TCP 三次握手过程，处于 ESTABLISHED 状态；

![](assets/2023-08-29-12-33-57-image.png)

**三次握手状态：**

![](assets/2023-08-29-12-36-24-image.png)

**四次挥手状态：**

![](assets/2023-08-29-12-37-41-image.png)

**TCP有限机状态图**

![](assets/2023-09-15-11-29-34-image.png)

**listen和accept阻塞？**

- listen函数不会阻塞，它只是相当于把socket的属性更改为被动连接，可以接收其他进程的连接。listen侦听的过程并不是一直阻塞，直到有客户端请求连接才会返回，它只是设置好socket的属性之后就会返回。监听的过程实质由操作系统完成。

- accept会阻塞（也可以设置为非阻塞），如果listen的套接字对应的连接请求队列为空（没有客户端连接请求），它会一直阻塞等待。

### TCP 拥塞控制

**拥塞控制**

- 拥塞窗口cwnd是发送⽅维护的⼀个状态变量，根据⽹络拥塞程度⽽变化。

- 发送窗⼜的值是swnd = min(cwnd, rwnd)，也就是拥塞窗口和接收窗口中的最⼩值。

- ⽹络中没有出现拥塞，cwnd增⼤，出现拥塞，cwnd减⼩。

拥塞控制就是防止过多的数据注入到网络中，这样可以使网络中的路由器或链路不致过载。

- 慢开始：发包的个数是指数性的增长。1->2->4->8的次⽅。
- 拥塞避免：每当收到⼀个 ACK 时，cwnd 增加1/cwnd。将原本慢启动算法的指数增长变成了线性增长。
- 拥塞发生
  - 超时重传：将ssthresh设为cwnd/2，cwnd重置为1。
  - 快速重传：收到三个重复得ACK，cwnd = cwnd/2 ，也就是设置为原来的⼀半，并将ssthresh置为cwnd。
  - 快速恢复：cwnd = ssthresh + 3。

![](assets/2023-11-15-19-32-15-image.png)

**TCP有哪些措施保证可靠性?**

- 序列号和确认应答：每个TCP报文段都会分配一个唯一的序列号，接收端通过发送确认应答来告知发送端已成功接收到数据。

- 超时重传：发送端在发送数据后会启动一个定时器，如果在规定时间内没有收到确认应答，就会重新发送该数据段。

- 滑动窗口：TCP使用滑动窗口机制来实现流量控制和拥塞控制。接收端通过通告窗口大小来告诉发送端可以接收的数据量，从而避免过多的数据堆积。

- 丢包重传：如果发生丢包，接收方可以通过请求重传丢失的数据段来实现可靠性。

- 流量控制：TCP使用滑动窗口机制来进行流量控制，根据接收方的处理能力和网络状况调整发送速率，避免过载造成数据丢失或延迟。

- 拥塞控制：TCP通过拥塞窗口、慢启动、拥塞避免等算法来感知并控制网络拥塞情况，以避免过度拥塞导致网络性能下降。

**`RST`产生的几种情况**

**为什么会产⽣粘包和拆包呢?**

- 要发送的数据⼩于 TCP 发送缓冲区的⼤⼩，TCP 将多次写⼊缓冲区的数据⼀次发送出去
- 接收数据端的应⽤层没有及时读取接收缓冲区中的数据
- 要发送的数据⼤于 TCP 发送缓冲区剩余空间⼤⼩
- 待发送数据⼤于 MSS（最⼤报⽂长度），TCP 在传输前将进⾏拆包。即 TCP 报⽂长度-TCP头部长度 > MSS

**TCP的粘包和拆包解决方法**

- 发送端将每个数据包封装为固定长度

- 在数据尾部增加特殊字符进⾏分割

- 将数据分为两部分，⼀部分是头部，⼀部分是内容体；其中头部结构⼤⼩固定，且有⼀个字段声明内容体的⼤⼩

**从输入域名到浏览器看见页面经历了什么过程？**

- 解析URL：分析URL所需要的传输协议和请求的资源路径。

- 缓存判断：如果请求的资源在缓存里没有失效，那么直接使用，否则向服务器发起新的请求。

- DNS解析：浏览器会向本地DNS服务器发送域名解析请求，本地DNS服务器会逐级查询，最终找到对应的IP地址。

- 获取MAC地址：通过将IP地址与本机的子网掩码相结合，可以判断是否与请求主机在同一个子网里，如果在同一个子网里，可以使用ARP协议获取到目的主机的MAC地址；如果不在一个子网里，那么请求应该转发给网关，由它代为转发，此时同样可以通过ARP协议来获取网关的MAC地址，此时目的主机的MAC地址应该为网关的地址。

- 建立TCP连接

- HTTPS的TLS四次握手：如果使用的是HTTPS请求，在通信前还存在TLS的四次握手。

- 发送HTTP请求：请求包含了用户需要获取的资源的信息，例如网页的URL、请求方法（GET、POST）等。

- 服务器处理请求并返回响应

## 系统

**页面置换算法**

在地址映射过程中，若在页面中发现所要访问的页面不在内存中，则产生缺页中断。当发生缺页中断时，如果操作系统内存中没有空闲页面，则操作系统必须在内存选择一个页面将其移出内存，以便为即将调入的页面让出空间。而用来选择淘汰哪一页的规则叫做页面置换算法。

全局：

- 工作集算法
- 缺页率置换算法

局部：

- 最佳置换算法（OPT）
- 先进先出置换算法（FIFO）
- 最近最久未使用（LRU）算法
- 时钟（Clock）置换算法

**vmalloc()和kmalloc()区别**

- kmalloc保证分配的内存在物理上是连续的,那么它对应的虚拟内存肯定也是连续的;而vmalloc保证的是在虚拟地址空间上的连续

- kmalloc一般分配较小的内存,vmalloc分配较大的内存

- vmalloc比kmalloc要慢,且分配的虚拟地址空间位置不同.这两个函数所分配的内存虽然都处于内核空间(3GB～4GB),但对应的具体位置不同，kmalloc()分配的内存处于3GB～high_memory之间，而vmalloc()分配的内存在VMALLOC_START～VMALLOC_END之间，也就是非连续内存区。kmalloc分配的内存,它的物理地址与虚拟地址只有一个PAGE_OFFSET偏移，不需要为地址段修改页表,而vmalloc分配内存时需要修改主内核页表

**僵尸进程**

- 子进程先结束，而父进程没有回收子进程，释放子进程占用的资源，此时子进程将成为一个僵尸进程。

**孤儿进程**

- 父进程先结束，而子进程仍然存活，此时子进程称为孤儿进程，将由系统的init进程负责回收相关资源。

**进程间通信方式**

![](assets/2023-10-25-14-48-53-image.png)

**进程之间私有和共享的资源**

- 私有：地址空间(用户空间)

- 共享：代码段，公共数据，内核空间，进程目录，进程ID

**线程之间私有和共享的资源**

- 私有：线程栈，寄存器，程序计数器，线程ID，错误码errno

- 共享：地址空间（包括代码段、数据段、bss段、堆区等）

**多进程和多线程的应用场景**

- IO密集型：一般不同任务间需要大量的通信，由于线程间通信比进程间通信的切换开销更低，使用多线程的场景比多进程多；

- CPU密集型：多进程有更高的容错性，一个进程的崩溃不会导致整个系统的崩溃，在任务安全性较高的情况下，采用多进程；

**栈保存哪些信息**

- 函数的参数和返回地址

- 临时变量：包括函数的非静态局部变量以及编译器自动生成的临时变量

- 保存上下文：包括函数调用前后需要保持不变的寄存器

**地址空间（用户空间）**

- 代码段text：程序的二进制可执行代码，只读

- 数据段data：已初始化的全局变量和静态变量

- BSS段：未初始化的静态变量和全局变量

- 堆段：动态分配的内存，从低地址开箱向上增长

- 文件映射段：包括动态库、共享内存等，从低地址开始向上增长

- 栈段：包括函数的参数值和局部变量、函数调用的上下文等

**ELF 格式的二进制文件中的 Section 是如何加载并映射进虚拟内存空间？**

- 完成这个映射过程的函数是`load_elf_binary`
  
  - `setup_new_exec`设置虚拟内存中的内存映射区域起始地址`mmap_base`
  
  - `setup_arg_pages`创建并初始化栈对应的`vm_area_struct`结构。`置mm->start_stack`是栈的起始地址也就是栈底，并将`mm->arg_start`是指向栈底的
  
  - `elf_map`将ELF格式的二进制文件中的`.text, .data, .bss`部分映射到虚拟内存空间的代码段，数据段，BSS段中
  
  - `set_brk`创建并初始化堆对应的`vm_area_struct`结构，设置`current->mm->start_brk = current->mm->brk`，设置堆的起始地址`start_brk`，结束地址`brk`。起初两者相等表示堆是空的
  
  - `load_elf_interp`将进程依赖的动态链接库`so`文件银蛇到虚拟内存空间中的内存映射区域
  
  - 初始化内存描述符`mm_struct`

**虚拟内存的作用**

- 使得进程对运行内存超过物理内存大：因为程序运行符合局部性原理，CPU 访问内存会有很明显的重复访问的倾向性，对于那些没有被经常使用到的内存，我们可以把它换出到物理内存之外，比如硬盘上的 swap 区域。

- 将主存看出一个存储在磁盘上的地址空间的告诉缓存，在主存中只保存活动区域，并根据需要在磁盘和主存之间来回传送数据（页面调度）。

- 解决了多进程之间地址冲突的问题：由于每个进程都有自己的页表，所以每个进程的虚拟内存空间就是相互独立的，进程也没有办法访问其他进程的页表。

- 提供了更好的安全性：页表里的页表项中除了物理地址之外，还有一些标记属性的比特，比如控制一个页的读写权限，标记该页是否存在等。

**malloc 是如何分配内存的？**

- 小于 128 KB：通过 brk() 系统调用从堆分配内存
- 大于 128 KB：通过 mmap() 系统调用在文件映射区域分配内存；

**为什么不全部使用 mmap 来分配内存？**

- 申请内存的操作应该避免频繁的系统调用，如果都用 mmap 来分配内存，等于每次都要执行系统调用。

- mmap 分配的内存每次释放的时候，都会归还给操作系统，于是每次 mmap 分配的虚拟地址都是缺页状态的，然后在第一次访问该虚拟地址的时候，就会触发缺页中断。

- 为了改进这两个问题，malloc 通过 brk() 系统调用在堆空间申请内存的时候，由于堆空间是连续的，所以直接预分配更大的内存来作为内存池，当内存释放的时候，就缓存在内存池中。

**为什么不全部使用 brk 来分配？**

- 随着系统频繁地 malloc 和 free ，尤其对于小块内存，堆内将产生越来越多不可用的碎片，导致“内存泄露”。而这种“泄露”现象使用 valgrind 是无法检测出来的。

**用户态到内核态**

内核态用于执行一些特权指令，比如中断机制，原语，进程管理等，目的是保护系统程序。

- 系统调用：包括文件（open、chmod）、进程、设备（read、ioctl）、信息（getXXX）、通信（pipe、mmap）

- 中断：IO完成，定时器时钟中断。

- 异常：越界，溢出，缺页，异常不能被屏蔽，一旦出现应立即处理。

**中断处理程序的处理流程**

- 在I/O时，设备控制器如果已经准备号数据，则会通过中断控制器向CPU发送中断请求

- 保存被中断进程的CPU上下文

- 转入相应的设备中断处理函数

- 进行中断处理

- 恢复被中断进程的上下文

![](assets/2023-09-29-09-19-20-image.png)

**中断**

- 定义：中断即打断，由于系统出现了某种需要处理的紧急情况，CPU暂停正在执行的程序，转而去执行另一段特殊程序来处理的紧急事务，处理结束后CPU自动返回到原先暂停的程序中去执行，这种执行过程由于外界的原因被打断的情况称为中断。

- 作用：中断使得计算机系统具备应对处理突发事件的能力，提高了CPU的工作效率，如果没有中断系统，CPU就只能轮询处理，不能及时响应紧急事件。

**中断机制**

- 外设速度远远慢于CPU的速度，所以需要外设资源准备好以后，利用中断机制主动通知操作系统。
- 内部中断/异常：CPU执行指令期间检测到非法的条件（除零，地址访问越界）
- 硬中断/外中断/中断：IO中断，时钟中断，信号中断产生的时间不确定。
- 软中断：应用程序使用系统调用而引发的事件。
- 中断描述符表
  - os中预先设置一些中断处理函数，当CPU接收到中断时，会根据中断号去查对应的处理函数，中断向量表就是记录中断号与中断函数映射关系的表。
- 中断机制是为了弥补CPU速度和外设速度数量级差异的机制，它的核心是中断向量表。

**程序编译过程**

- **预处理：**
  
  - 将所有的`#define`删除，并展开所有的宏定义
  
  - 处理所有的条件预编译指令
  
  - 处理`#include`预编译指令，将被包含的文件直接插入到预编译指令的位置。
  
  - 删除所有的注释
  
  - 添加行号和文件标识，以便编译时和调试时使用
  
  - 保留所有的`#pragma`编译器指令，因为编译器需要使用它们
  
  - 通常使用`gcc -E hello.c -o hello.i`进行预处理

- **编译：** 将预处理完的`.i`文件进行词法分析、语法分析，语义分析、源代码优化，代码生成、目标代码优化，翻译成`.s`文件

- **汇编：** 将`.s`文件翻译成机器语言指令，并将这些指令打包成可重定位目标程序的格式，保存在`.o`文件中

- **链接：** 将各种代码和数据片段收集并组合成一个单一文件的过程，链接可以执行于**编译时**，也可以执行于**加载时**和**运行时**。

**动态链接和静态链接**

- **静态链接**： 1) 函数和数据被编译进一个二进制文件。在使用静态库的情况下，在编译链接可执行文件时，链接器从库中复制这些函数和数据并把它们和应用程序的其它模块组合起来创建最终的可执行文件。 1) **空间浪费**：因为每个可执行程序中对所有需要的目标文件都要有一份副本，所以如果多个程序对同一个目标文件都有依赖，会出现同一个目标文件在多个程序内都存在一个副本； 1) **更新困难**：每当库函数的代码修改了，这个时候就需要重新进行编译链接形成可执行程序。 1) **运行速度快**：但是静态链接的优点就是，在可执行程序中已经具备了所有执行程序所需要的任何东西，在执行的时候运行速度快。

- **动态链接**： 1) 动态链接的基本思想是把程序按照模块拆分成各个相对独立部分，在程序运行时才将它们链接在一起形成一个完整的程序，而不是像静态链接一样把所有程序模块都链接成一个单独的可执行文件。 1) **共享库**：就是即使需要每个程序都依赖同一个库，但是该库不会像静态链接那样在内存中存在多个副本，而是这多个程序在执行时共享同一份副本； 1) **更新方便**：更新时只需要替换原来的目标文件，而无需将所有的程序再重新链接一遍。当程序下一次运行时，新版本的目标文件会被自动加载到内存并且链接起来，程序就完成了升级的目标。 1) **性能损耗**：因为把链接推迟到了程序运行时，所以每次执行程序都需要进行链接，所以性能会有一定损失。

- 区别
  
  - 使用静态链接生成的可执行文件可能会存在共享库的多个复本, 而使用动态链接库的可执行文件只有存在一份
  - 使用静态链接库的可执行程序不需要依赖动态链接库, 依赖关系简单; 而使用动态链接库的可执行程序需要引用动态链接库, 故而依赖关系复杂
  - **静态链接生成的静态链接库不能再包含其他的动态链接库或则静态库, 而动态链接库可以包括其他的动态库或则静态库.**

**静态链接过程**

- **空间和地址分配**：扫描所有目标文件，获取他们各个段的长度，属性和位置，并且将各个文件符号表中的所有符号定义和符号引用收集起来，统一放到一个全局符号表中。

- **符号解析**：将每个符号引用正好和它输入的可重定位目标文件的符号表中的一个确定的符号定义关联起来。即每个符号对应于一个函数、一个全局变量或一个静态变量。

- **重定位**：编译器和汇编器生成的是从地址0开始的代码和数据节，链接器通过把每个符号定义与一个内存位置关联起来，从而重定位这些节，然后修改所有对这些符号的引用，使它指向这个内存位置。

**多线程数量**

- 线程数量过多，会导致过多的上下文切换；数量过少，会导致任务堆积甚至OOM；

- IO密集型：线程数 = CPU核心数 * (1 + IO耗时 / CPU耗时)
  
  - IO密集型任务，可能是未就绪需要阻塞的IO事件，如果IO事件已经就绪，随时可以读写，就不会对线程产生阻塞，转变为CPU密集型任务；因此可以对阻塞的IO密集型任务使用IO多路复用，去掉不必要的阻塞。

- CPU密集型：线程数 = CPU核心数 + 1

**断点实现原理**

- 断点实现原理是在程序运行中**设置特殊的指令**（通常为`int 3`），当程序**执行到断点位置时触发中断异常**，然后进行**中断处理**（由操作系统保存现场、处理中断、恢复现场）；

- 中断处理完成后，操作系统会返回到断点处继续执行，然后向调试器发送一个中断事件通知，调试器接收到通知后会进入调试模式，暂停程序的执行；

**原子变量原理**

- Bus Lock：锁住Bus会导致后续无关内存操作都不能继续；

- Cacheline Lock：借助多核Cache一致性协议MESI实现原子操作；

**互斥锁原理**

- mutex的本质就是一个内存标志，这个标志可以是一个flag（占用标志），也可以是一个指针，指向一个持有者的线程ID，也可以是两个都有，以及一个等待阻塞队列，以及若干其它信息等。

- 这个占用标志flag，利用硬件的原子操作来实现，比如x86汇编里有CMPXCHG指令，就可以实现比较+原子交换，锁都是用这个实现的。单核CPU就是用这个实现的。

- spinlock能实现多核的锁定，spinlock锁定多核以后，再操作锁的占用标志就是被保护的状态，操作完以后还要用memory barriar（内存屏障）保证锁的状态可以同步到所有的核心。

**异步IO原理**

- 底层将数据准备好后，内核会给进程发送一个异步通知信号SIGIO29通知进程，然后进程调用信号处理函数去读数据，没准备好，数据就忙自己的事情。

**文件系统层次结构**

- 用户空间
- 内核空间
  - 虚拟文件系统VFS
  - 块设备文件系统、内存文件系统、闪存文件系统
  - 页缓存 Page Cache
  - 块设备层
  - 块缓存
  - IO调度器
  - 块设备驱动程序
- 硬件
  - 机械硬盘、固态硬盘
  - 闪存

## 设计模式

**订阅发布模式与观察者模式**

- 订阅发布模式主要以异步的方式（消息队列）实现，更像是一种跨应用程序的模式

- 观察者模式大多以同步方式实现，且通常在单个程序的地址空间中实现

![](assets/2023-11-16-18-48-00-image.png)
