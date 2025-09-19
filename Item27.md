## 重载通用引用的替代品
**1、tag dispatch**
如果想让整型数据走一个逻辑，非整型参数走另一个逻辑。可以使用以下方式。
```cpp
template<typename T>
void func1Logic(T&& t, false_type){
    cout<<"not int param logic"<<endl;
}

void func1Logic(int t, true_type){
    cout<<"int param logic"<<endl;
}
// 使用 is_integral<remove_reference_t<T>> 获取参数的类型，true_type代表是整型，false_type代表不是。
// 根据参数类型使用对应的 func1Logic 函数。
// 这里的remove_reference_t消除T的引用，因为 is_integral<int&> 的类型会是 false_type
template<typename T>
void func1(T&& t){
    is_integral<remove_reference_t<T>> type;
    func1Logic(forward<T>(t), type);
}

int main(){ 
    int a = 1;
    func1(a);
    long b = 1;
    func1(b);
    unsigned short c = 1; 
    func1(c);
    float d = 1.2f;
    func1(d);
    string s = "abc";
    func1(s);
    char c1 = 'a';
    func1(c1);
    return 0;
}
```

**禁止模板使用某种类型**
在声明模板时使用enable_if限定可以使用的类型。

```cpp
    Widget w1;
    Widget w2(w1);
    Widget w3(move(w1));
```

构造w2时，T的类型为Widget&，类型和Widget&一样，不使用完美转发构造函数，最终选择拷贝构造函数
构造w3时，T的类型为Widget，类型和Widget&不一样，使用完美转发构造函数
```cpp
    template<
        typename T,
        typename = enable_if_t<!is_same<Widget&, T>::value>
    >
    explicit Widget(T&& t){
        cout<<"const perfect forward constructor"<<endl;
    }

    Widget(const Widget& w){
        cout<<"const lvalue references constructor"<<endl;
    }
```

构造w2时，T的类型为Widget&，类型和Widget不一样，由于不是const的，所以使用完美转发构造函数
构造w3时，T的类型为Widget，类型和Widget一样，不使用完美转发构造函数，最终选择拷贝构造函数
```cpp
    template<
        typename T,
        typename = enable_if_t<!is_same<Widget, T>::value>
    >
    explicit Widget(T&& t){
        cout<<"const perfect forward constructor"<<endl;
    }

    Widget(const Widget& w){
        cout<<"const lvalue references constructor"<<endl;
    }
```

decay_t会移除类型所有的修饰，包括const，引用，volatile。
构造w2和w3时，decay_t\<T>的类型为Widget，类型和Widget一样，不使用完美转发构造函数，最终选择拷贝构造函数
```cpp
    template<
        typename T,
        typename = enable_if_t<!is_same<Widget, decay_t<T>>::value>
    >
    explicit Widget(T&& t){
        cout<<"const perfect forward constructor"<<endl;
    }

    Widget(const Widget& w){
        cout<<"const lvalue references constructor"<<endl;
    }
```