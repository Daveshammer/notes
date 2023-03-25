Template

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
