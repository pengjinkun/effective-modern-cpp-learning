## 区分右值引用和通用引用
**通用引用的两种情况**
通用引用出现在类型推导时，且通用引用是一个引用，它的值必须被初始化（典型的未初始化对象就是类的const static类型成员）。
1、函数模板形参类型为T&&时。
```cpp
template<typename T>
void func(T&&){

}
```
2、auto&&推导类型时。
```cpp
auto&& a = 10;
```
**右值引用的情况**
1、没有进行类型推导，这里的Widget&&就是一个右值引用
```cpp
void func(Widget&& w){

}
```
2、作为模板形参但格式不是T&&类型的，而是加了其他修饰
```cpp
//这里T的类型虽然有推导，但最终v只会是一个右值引用而不是左值引用，所以不算通用引用。
template<typename T>
void func(vector<T>&& v){

}
//这个是有争议的，T为左值时，v会被推导为const T&。T为右值时，v会被推导为const T&&。应该算是通用引用。
//不过这个用法并不常见，没特殊情况也不要使用，推荐使用const T&接收左值，使用T&&接收右值。
//因为左值作为参数传递时是不建议被修改的，可能影响引用代码的数据。
//而右值通常支持修改，而且可以通过forward转发给其他函数。
template<typename T>
void func(const T&& v){

}
```
3、模板类的函数形参是T&&，但是在类实例化时已经生成了确定类型的形参。
```cpp
template<template T, template Allocator = allocator<T>>
class vector
{
public:
    void push_back(T&& x);//x是一个右值引用

    template <class... Args>
    void emplace_back(Args&&... args);//Args是一个通用引
}
/*
    在使用vector<Widget> v时，已经实例化了一个void push_back(Widget&& x);函数，所以x是一个右值引用

    而emplace_back使用了可变参数模板，而且Args是和T独立的，实例化vector<Widget>是Args的类型并不确定，所以args是一个通用引用。
*/
```