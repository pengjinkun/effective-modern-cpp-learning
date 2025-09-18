## 右值引用使用move，通用引用使用forward
**1、禁止对通用引用使用move**
通用引用在接收左值时会转化为左值引用，对左值使用move会移动该左值的数据。
**2、最后一次使用时，对右值引用使用forward，对通用引用使用move。**
**对按值返回的右值引用和通用引用执行同样的操作**
```cpp
/*
    当t是一个右值引用时，如果在func2()中就使用了forward那么t内部资源被移动了。后面的函数会执行错误的结果。
    那么func2或者func3里面是否会影响t的数据呢，在符合标准的设计中，由于t是一个左值，被传递给其他函数时不应被修改。
    因此只在最后执行foward会使用移动语义，减少复制开销，并且整个过程中t的内部资源会保持正确。
*/
template<typename T>
void func1(T&& t){
    func2(t);
    func3(t);
    func4(forward<T>(t));
}
    
// 同理，只在最后执行move会使用移动语义，减少复制开销，并且整个过程中t的内部资源会保持正确。
void func1(Widget&& w){
    func2(w);
    func3(w);
    func4(move(w));
}
```
**3、返回值优化，RVO和NRVO**
***RVO(Return Value Optimization)***
当返回值是一个匿名右值对象时，会将该匿名对象指向返回值的内存，减少移动和拷贝的开销。
```cpp
Widget GetWidget(){
    return MakeWidget();
}
```

***NRVO(Named Return Value Optimization)***
满足两个要求：
    局部变量是非volatile，非引用的。且该局部变量的类型和函数返回值类型一样。
    函数返回的就是该局部变量。
工作原理是：
    编译器直接在为返回值预留的内存上构造一个对象。
    对于函数中与该返回值类型一样且被返回的局部变量的所有操作都作用于返回值内存上。
    整个过程只创建了一个返回值对象，不需要构造一个局部变量再将该变量移动/拷贝给返回值对象。
```cpp
Widget GetWidget(){
    Widget w{};
    return w;
}
```

***函数形参为返回值不会执行RVO/NRVO优化***
RVO有以下规则
1、编译器被允许执行 RVO/NRVO 来消除拷贝/移动。
​2、如果编译器选择不执行拷贝消除（即不进行 RVO/NRVO），那么该 return语句中的局部对象会被自动视为一个右值（rvalue）。
下面代码中不会执行拷贝消除；如果有移动构造函数，会执行移动构造；没有移动构造执行拷贝构造。
```cpp
Widget GetWidget(Widget w){
    return w;
}
```

**4、如果局部对象会指向NRVO，就不要再使用move或者forward来移动局部变量了，这会适得其反**
