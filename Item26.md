## 避免重载通用引用
**1、普通函数的重载**
func1有两个重载函数，一个参数时通用引用，一个是int。
对于int类型的形参会匹配到第二个函数。
但是对于long、short这类整形形参来说，即使short可以隐式转换为int，依然会优先会匹配到通用引用。
```cpp
template<typename T>
void func1(T&& t){
    cout<<"use universal references param func"<<endl;
}

void func1(int num){
    cout<<"use int param func"<<endl;
}

int main(){ 
    int a = 10;
    short b = 10;
    func1(a); // use int param func
    func1(b); // use universal references param func
    return 0;
}
```

**2、完美转发的构造函数**
const参数会优先匹配拷贝构造函数
none-const参数会优先匹配完美转发构造函数
```cpp
class Widget{
    public:
    int num = 10;
    Widget() = default;
    template<typename T>
    explicit Widget(T&& t){
        cout<<"perfect forward constructor"<<endl;
    }
    Widget(const Widget& w){
        cout<<"const lvalue references constructor"<<endl;
    }
};

int main(){ 
    Widget w1;
    const Widget w2;
    Widget w3(w1); // perfect forward constructor
    Widget w4(w2); // const lvalue references constructor
    return 0;
}
```