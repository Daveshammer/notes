## Template

编译器是没办法去根据返回值推断类型的。

```cpp
float data[1024];

template <typename T> T GetValue(int i)
{
    return static_cast<T>(data[i]); 
}

float a = GetValue(0);    // 出错了！
int b = GetValue(1);    // 也出错了！
```

在部分推导、部分指定的情况下，编译器对模板参数的顺序是有限制的：**先写需要指定的模板参数，再把能推导出来的模板参数放在后面**。

在这个例子中，能推导出来的是 `SrcT`，需要指定的是 `DstT`。

```cpp
template <typename DstT, typename SrcT> DstT c_style_cast(SrcT v)    // 模板参数 DstT 需要人肉指定，放前面。
{
    return (DstT)(v);
}

int v = 0;
float i = c_style_cast<float>(v);  // 形象地说，DstT会先把你指定的参数吃掉，剩下的就交给编译器从函数参数列表中推导啦。 
```

# C++基础

## const

C++中，所有出现const常量名字的地方，都被常量的初始化替换了

```cpp
    const int a = 20;
    int *p = (int*)&a;
    *p = 30;
    printf("%d %d %d \n", a, *p, *(&a)); //20 30 30
```

```cpp
    int b = 20;
    const int a = b; //所有的a用临时变量b替换
    int *p = (int*)&a;
    *p = 30;
    printf("%d %d %d \n", a, *p, *(&a)); //30 30 30
```

**常见错误**：

1. 不能作为左值 -> 直接修改常量的值
2. 不能把常量的地址泄露给一个普通指针或者普通的引用变量 -> 间接修改常量的值

```cpp
    const int a =20
    //a = 30; // 不能作为左值 -> 直接修改常量的值
    //int *p = &a; //不能把常量的地址泄露给一个普通指针或者普通的引用变量 -> 间接修改常量的值
    const int *p = &a;
```

**总结const和指针的类型转换**：

```cpp
to              from
int*            const int*    //是错误的！
const int*      int*          //是可以的！

int**           const int**   //是错误的！
const int**     int**         //是错误的！

int**           int*const*    //是错误的！
int*const*      int**         //是可以的！
```

```cpp
    int a = 10;
    const int *p = &a;
    int *q = p; //invalid conversion from ‘const int*’ to ‘int*’
```

```cpp
    int a = 10;
    int *p = &a;
    const int **q = &p; //相当于 *q <=> p，将常量的地址泄露给一个普通指针
```

## 引用

**引用的本质是指针**

定义一个引用变量，和定义一个指针变量，其汇编指令是一模一样的；通过引用变量修改内存的值，和通过指针解引用修改指针指向的内存的值，其底层指令也是一模一样的。

```cpp
    int array[5] = {};
    int *p = array;
    // 定义一个引用变量，来引用array数组
    int (&q)[5] = array; // int (*q)[5] = &array;

    cout << sizeof(array) << endl; // 20
    cout << sizeof(p) << endl; // 8
    cout << sizeof(q) << endl; // 20
    cout << sizeof(int*) << endl; // 8
```

**右值引用**

1. 专门用来引用右值类型，指令上，可以自动产生临时量然后直接引用临时量

2. 右值引用本身是一个左值，只能用左值引用来引用它

3. 不能用一个右值引用变量，来引用一个左值

```cpp
    int a = 10; // a是左值，它有内存，有名字，值可以修改
    int &b = a;

    int &&c = 20; // 20是右值，它没有内存，没有名字
    c = 30;

    int &e = c; // 一个右值引用变量，其本身是个左值
```

## new 和 malloc

malloc 和 free，称作 c 的库函数

new 和 delete，称作运算符

new不仅可以做内存开辟，还可以做内存初始化操作

malloc开辟内存失败，是通过返回值和nullptr做比较：而new开辟内存失败，是通过抛出bad_alloc类型的异常来判断的。

new 可以认为是 malloc + 构造函数， delete 可以认为是 free + 析构函数

```cpp
    // new有多少种
    int *p1 = new int(10);
    int *p2 = new (nothrow) int;
    const int *p3 = new const int(40);

    //定位new
    int data = 0;
    int *p4 = new (&data) int(50);
    cout << data << endl; // 50
```

1. `malloc`和`new`的区别
   
   1. `malloc`按字节开辟内存；`new`开辟内存时需要指定类型，如`new int[10]`；所以`malloc`开辟内存都是`void*`
   
   2. `malloc`只负责开辟内存，`new`不仅仅有`malloc`的功能，可以进行数据的初始化
   
   3. `malloc`开辟内存返回`nullptr`指针；`new`抛出的是`bad_alloc`类型的异常

2. `free`和`delete`的区别
   
   `delete (int*)p:`先调用析构函数，后`free(p)`

```cpp
#include <iostream>
using namespace std;

void* operator new(size_t size)
{
    void* p = malloc(size);
    if (nullptr == p)
    {
        throw bad_alloc();
    }
    cout << "operator new addr:" << p << endl;
    return p;
}
void operator delete(void* ptr)
{
    cout << "operator delete addr:" << ptr << endl;
    free(ptr);
}

void* operator new[](size_t size)
{
    void* p = malloc(size);
    if (nullptr == p)
    {
        throw bad_alloc();
    }
    cout << "operator new[] addr:" << p << endl;
    return p;
}
void operator delete[](void* ptr)
{
    cout << "operator delete[] addr:" << ptr << endl;
    free(ptr);
}

class Test
{
public:
    Test(int data = 10) { cout << "Test()" << endl;}
    ~Test() { cout << "~Test()" << endl; }

#if 0
    Test(int data = 10) :ptr(new int(data)) { cout << "Test()" << endl;}
    ~Test() { cout << "~Test()" << endl; delete ptr; }
#endif
private:
    int *ptr;
};

int main()
{
#if 0
    Test *p1 = new Test();
    delete p1;
#endif

    Test *p2 = new Test[2];
    cout << "p2[0]:" << &p2[0] << endl; // p2[0]的地址和申请内存地址不一样，需要额外记录数组元素个数
    delete[] p2;

#if 0
    try
    {
        int *p = new int; // 调用operator new分配内存，然后调用int的构造函数初始化
        delete p; // 调用ptr指向对象的析构函数，然后调用operator delete释放ptr指向的内存

        int *q = new int[10];
        delete[] q;
    }
    catch(const bad_alloc& e)
    {
        std::cerr << e.what() << '\n';
    }
#endif   

    return 0;
}
```

频繁`malloc`和`free`效率低下，考虑使用对象池进行分配内存

```cpp
#include <iostream>
using namespace std;

template <typename T>
class Queue
{
public:
    Queue()
    {
        _front = _rear = new QueueItem();
    }
    ~Queue()
    {
        while (_front)
        {
            _rear = _front->_next;
            delete _front;
            _front = _rear;
        }
    }
    void push(const T &val)
    {
        QueueItem *item = new QueueItem(val);
        _rear->_next = item;
        _rear = item;
    }
    void pop()
    {
        if (empty())
        {
            return;
        }
        QueueItem *p = _front->_next; // p指向队头元素
        _front->_next = p->_next;     // 队头元素出队
        if (_front->_next == nullptr) // 如果队列中只有一个元素，出队后队列为空
        {
            _rear = _front;
        }
        delete p;
    }
    T front() const
    {
        return _front->_next->_data;
    }
    T back() const
    {
        return _rear->_data;
    }
    bool empty() const
    {
        return _front == _rear;
    }

private:
    // 产生一个QueueItem的对象池（10000个QueueItem节点）
    struct QueueItem // 队列中的节点
    {
        QueueItem(T data = T()) : _data(data), _next(nullptr) {}
        // 给QueueItem提供自定义内存管理
        void *operator new(size_t size)
        {
            if (_itemPool == nullptr)
            {
                _itemPool = (QueueItem *)new char[POOL_ITEM_SIZE * sizeof(QueueItem)];
                QueueItem *p = _itemPool;
                for (int i = 0; i < POOL_ITEM_SIZE - 1; i++)
                {
                    p->_next = (QueueItem *)((char *)p + sizeof(QueueItem));
                    p = p->_next;
                }
                p->_next = nullptr;
            }

            QueueItem *p = _itemPool; // 从对象池中取出一个节点
            _itemPool = _itemPool->_next; // 对象池中的可用节点数减1，指向下一个可用节点
            return p; // 返回取出的节点
        }
        void operator delete(void *ptr)
        {
            QueueItem *p = (QueueItem *)ptr;
            p->_next = _itemPool; // 把释放的节点放回对象池中
            _itemPool = p;
        }
        T _data;
        QueueItem *_next;
        static QueueItem *_itemPool;
        static const int POOL_ITEM_SIZE = 10000;
    };

    QueueItem *_front; // 队头的前一个节点，头节点
    QueueItem *_rear;  // 队尾
};

template <typename T>
typename Queue<T>::QueueItem *Queue<T>::QueueItem::_itemPool = nullptr;

int main()
{
    Queue<int> que;
    for (int i = 0; i < 1000000; i++)
    {
        que.push(i);
        que.pop();
    }
    cout << que.empty() << endl;

    return 0;
}
```





## 面向对象

## Template

### 函数模板

```cpp
// 函数模板，是不进行编译的，因为类型还不知道
// 函数实例化，函数调用点进行实例化
// 模板函数，才是要被编译器所编译的
// 模板的实参推演，可以根据用户传入的实参的类型，来推导出模板类型
// 模板的特例化（专用化），特殊（不是编译器提供的，而是用户提供的）实例化

template<typename T>  //定义一个模板参数列表
bool compare(T a, T b) // compare 是一个函数模板
{
  cout << "template compare" << endl;
  return a > b;
}

// 在函数调用点，编译器用用户指定的类型，从原模板实例化一份函数代码出来
/*
bool compare<int>(int a, int b) // compare 是一个函数模板
{
  return a > b;
}
*/

// 针对 compare 函数模板，提供 const char* 类型的特例化版本
template<>
bool compare<const char *>(const char *a, const char *b)
{
  return strcmp(a, b) > 0;
}

int main()
{
  // 函数调用点
  compare<int> (10, 20);
  compare<double> (10.5, 20.5);
  return 0;
}
```

模板代码是不能在一个文件定义，在另一个文件使用

模板代码调用之前，一定要看到模板定义的地方，这样的话，模板才能够进行正常的实例化，产生能够被编译器编译的代码

所以，模板代码都是放在头文件中，然后在源代码当中直接进行 `#include`包含。

### 类模板

类模板 =》 实例化 =》 模板类`class SeqStack<int>{}`

```cpp
template<typename T>
class SeqStack
{
public:
    // 构造和析构函数名不用加<T>，其它出现模板的地方都要加上类型参数列表
    SeqStack(int size = 10)
        : _pstack(new T[size])
        , _top(0)
        , _size(size)
    {}
    ~SeqStack()
    {
        delete[] _pstack;
        _pstack = nullptr;
    }
    SeqStack(const SeqStack<T>& stack)
        : _top(stack._top)
        , _size(stack._size)
    {
        _pstack = new T[stack._size];
        for (int i = 0; i < stack._top; ++i)
        {
            _pstack[i] = stack._pstack[i];
        }
        _top = stack._top;
        _size = stack._size;
    }
    SeqStack<T>& operator=(const SeqStack<T>& stack)
    {
        if (this != &stack)
        {
            delete[] _pstack;
            _pstack = new T[stack._size];
            for (int i = 0; i < stack._top; ++i)
            {
                _pstack[i] = stack._pstack[i];
            }
            _top = stack._top;
            _size = stack._size;
        }
        return *this;
    }

    void Push(const T& x)
    {
        if (IsFull())
        {
            expand();
        }
        _pstack[_top++] = x;
    }
    void Pop()
    {
        if (IsEmpty())
        {
            return;
        }
        --_top;
    }
    T& GetTop() const
    {
        return _pstack[_top - 1];
    }
    bool IsEmpty() const
    {
        return _top == 0;
    }
    bool IsFull() const
    {
        return _top == _size;
    }
    int GetSize() const
    {
        return _top;
    }

private:
    T *_pstack;
    int _top;
    int _size;

    void expand()
    {
        T *p = new T[_size * 2];
        for (int i = 0; i < _size; ++i)
        {
            p[i] = _pstack[i];
        }
        delete[] _pstack;
        _pstack = p;
        _size *= 2;
    }
};
```

上面的代码使用`SeqStack<Test> s;`会默认构造10个`Test()`，需要对容器底层内存实现额外的空间配置器

```cpp
#include <iostream>
using namespace std;

template <typename T>
class Allocator
{
public:
    T *allocate(size_t size)
    {
        return (T *)malloc(size * sizeof(T));
    }
    void deallocate(T *p)
    {
        free(p);
    }
    void construct(T *p, const T &x)
    {
        new (p) T(x);
    }
    void destroy(T *p)
    {
        p->~T();
    }
};

// 容器底层内存开辟，内存释放，对象构造和析构，都通过allocator来完成
template <typename T, typename Alloc = Allocator<T>>
class SeqStack
{
public:
    // 构造和析构函数名不用加<T>，其它出现模板的地方都要加上类型参数列表
    SeqStack(int size = 10, const Alloc &alloc = Allocator<T>()) // 需要把内存开辟和对象构造分开处理
                                                                 //  : _pstack(new T[size])
                                                                 //  , _top(0)
                                                                 //  , _size(size)
    {
        _pstack = _allocator.allocate(size);
        _top = 0;
        _size = size;
    }
    ~SeqStack()
    {
        // 析构有效的元素，然后释放_pstack指针指向的堆内存
        // delete[] _pstack;
        for (int i = 0; i < _top; ++i)
        {
            _allocator.destroy(&_pstack[i]); // 析构对象
        }
        _allocator.deallocate(_pstack); // 释放堆上的数组内存
        _pstack = nullptr;
    }
    SeqStack(const SeqStack<T> &stack)
        : _top(stack._top), _size(stack._size)
    {
        // _pstack = new T[stack._size];
        _pstack = _allocator.allocate(stack._size);
        for (int i = 0; i < stack._top; ++i)
        {
            _pstack[i] = stack._pstack[i];
        }
        _top = stack._top;
        _size = stack._size;
        cout << "SeqStack(const SeqStack<T> &stack)" << endl;
    }
    SeqStack<T> &operator=(const SeqStack<T> &stack)
    {
        if (this != &stack)
        {
            // delete[] _pstack;
            for (int i = 0; i < _top; ++i)
            {
                _allocator.destroy(&_pstack[i]); // 析构对象
            }
            _allocator.deallocate(_pstack); // 释放堆上的数组内存
            _pstack = nullptr;
            // _pstack = new T[stack._size];
            _pstack = _allocator.allocate(stack._size);
            for (int i = 0; i < stack._top; ++i)
            {
                _pstack[i] = stack._pstack[i];
            }
            _top = stack._top;
            _size = stack._size;

            cout << "SeqStack<T> &operator=(const SeqStack<T> &stack)" << endl;
        }
        return *this;
    }

    void Push(const T &x)
    {
        if (IsFull())
        {
            expand();
        }
        // _pstack[_top++] = x;
        _allocator.construct(&_pstack[_top++], x);
    }
    void Pop()
    {
        if (IsEmpty())
        {
            return;
        }
        --_top;
        _allocator.destroy(&_pstack[_top]);
    }
    T &GetTop() const
    {
        return _pstack[_top - 1];
    }
    bool IsEmpty() const
    {
        return _top == 0;
    }
    bool IsFull() const
    {
        return _top == _size;
    }
    int GetSize() const
    {
        return _top;
    }

private:
    T *_pstack;
    int _top;
    int _size;
    Alloc _allocator;

    void expand()
    {
        // T *p = new T[_size * 2];
        T *p = _allocator.allocate(_size * 2);

        for (int i = 0; i < _size; ++i)
        {
            // p[i] = _pstack[i];
            _allocator.construct(&p[i], _pstack[i]);
        }

        // delete[] _pstack;
                for (int i = 0; i < _top; ++i)
        {
            _allocator.destroy(&_pstack[i]); // 析构对象
        }
        _allocator.deallocate(_pstack); // 释放堆上的数组内存
        _pstack = nullptr;

        _pstack = p;
        _size *= 2;
    }
};

class Test
{
public:
    Test() { cout << "Test()" << endl; }
    Test(const Test &t) { cout << "Test(const Test &t)" << endl; }
    Test &operator=(const Test &t)
    {
        cout << "Test &operator=(const Test &t)" << endl;
        return *this;
    }
    Test(const Test &&t) { cout << "Test(const Test &&t)" << endl; }

    ~Test() { cout << "~Test()" << endl; }
};

int main()
{
    Test t1;
    cout << "------------" << endl;
    SeqStack<Test> s;
    s.Push(t1);
    s.Pop();
    cout << "------------" << endl;

    return 0;
}
```

输出如下：

```shell
Test()
------------
Test(const Test &t)
~Test()
------------
~Test()
```

编译器做对象运算的时候，会调用对象的运算符重载函数（优先调用成员方法）；如果没有成员方法，就在全局作用域找合适的运算符重载函数。

## 迭代器

进行元素增删之后，之前定义的迭代器会失效

```cpp
    vector<int> vec;
    for (int i = 0; i < 20; i ++)
    {
        vec.push_back(i);
    }
    auto it1 = vec.end();
    vec.pop_back(); // it1失效
```



# C++高级
