## 右值引用、移动语义、完美转发
**1、形参永远是左值，即使它的类型是右值引用**
```cpp
// 这里的w是一个左值
void func(Widget&& w);
```

**2、不要对const对象使用move，此时不会调用移动构造、而是调用拷贝构造**
下面操作中首先move会将text转换为右值，但是const属性仍然保存
再对value进行构造时会挑选使用 拷贝构造函数 / 移动构造函数
由于value带const属性那么就只会使用拷贝构造函数
从一个对象中移动出某个值通常代表着修改该对象，所以语言不允许const对象被传递给可以修改他们的函数
```cpp
class Demo{
    public:
        string value;
        Demo(const string& text):value(move(text)){

        }
}
class string{
    public:
        string(string&&);//移动构造函数
        string(const string&);//拷贝构造函数
}
```
**3、move()**
move的功能只是将一个实参转化为右值，并没有执行移动操作。
移动操作的执行是在使用移动构造函数或者移动赋值运算符时。
移动操作的本质是将源对象的资源转移给新的对象，例如将源对象内部成员的指针赋值给新的对象，随后自己内部成员的指针变为无效。
```cpp
//这里将s1用于s2的移动构造，s1自己的指针是不会改变的，但其内部成员c_str的指针发生了改变。
//不过string类型比较特殊，当其长度较短（一半是小于16）时会进行优化，数据会存放在栈中，此时执行复制构造函数。
//当前长度较长时才会去堆中申请内存存放数据，此时执行移动构造函数。
//对于int char float这一类基础类型也不会执行移动操作，因为复制操作的效率往往更高。
string s1 = "1234567890abcdefghijk";
cout<<&s1<<endl;
cout<<(void*)s1.c_str()<<endl;
string s2 = move(s1);
cout<<&s1<<endl;
cout<<(void*)s1.c_str()<<endl;
cout<<&s2<<endl;
cout<<(void*)s2.c_str()<<endl;
/* 打印结果
0x61fd10
0x26b2140
0x61fd10
0x61fd20
0x61fcf0
0x26b2140
*/
```
**4、forward()**
在特定条件下才会将形参转化为右值。通常和通用引用一起使用。
执行移动语义的情况，当t为右值时
```cpp
template<typename T>
T func1(T&& t){
    return forward<T>(t);
}
int main(){
    vector<int> v1{1,2,3};
    cout<<&(v1[0])<<endl; //0xea2140
    vector<int> v2 = func1(move(v1));
    cout<<&(v1[0])<<endl; //0
    cout<<&(v2[0])<<endl; //0xea2140
    return 0;
}
```
不执行移动语义的情况，当t为左值时
```cpp
template<typename T>
T func1(T&& t){
    return forward<T>(t);
}
int main(){
    vector<int> v1{1,2,3};
    cout<<&(v1[0])<<endl; //0x26c2140
    vector<int> v2 = func1(v1);
    cout<<&(v1[0])<<endl; //0x26c2140
    cout<<&(v2[0])<<endl; //0x26c2160
    return 0;
}
```