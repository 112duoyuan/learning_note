#### std::move 将一个左值转化为了一个右值

相当于对于 `vector<T>::push_back()`有两个版本的实现，简单写如下：

~~~c++
template<typename T>
class Vector
{
    void push_back(T& lval);
    void push_back(T&& rval);
};
//而对应的类型 T 也实现了移动拷贝，如下：
class T
{
    T(T& other)
    {
        // copy constructor
    }

    T(T&& other)
    {
        // move constructor
    }
};
~~~

#### 完美转发 std::forward()

~~~markdown
当我们将一个右值引用传入函数时，他在实参中有了命名，所以继续往下传或者调用其他函数时，根据C++ 标准的定义，这个参数变成了一个左值。那么他永远不会调用接下来函数的右值版本，这可能在一些情况下造成拷贝。为了解决这个问题 C++ 11引入了完美转发，根据右值判断的推倒，调用forward 传出的值，若原来是一个右值，那么他转出来就是一个右值，否则为一个左值。
这样的处理就完美的转发了原有参数的左右值属性，不会造成一些不必要的拷贝。代码如下：

~~~

~~~c++

#include <iostream>
#include <vector>
#include <string>

using namespace std;

int main()
{
    string A("abc");
    string&& Rval = std::move(A);//move 只是纯粹的将一个左值转化为了一个右值
    string B(Rval);    // this is a copy , not move.
    cout << A << endl;   // output "abc"
    string C(std::forward<string>(Rval));  // move.
    cout << A << endl;       /* output "" */
    return 0;
}
~~~

~~~c++
/*std::forward
右值引用类型是独立于值的，一个右值引用参数作为函数的形参，在函数内部再转发该参数的时候它已经变成一个左值，并不是他原来的类型。
如果我们需要一种方法能够按照参数原来的类型转发到另一个函数，这种转发类型称为完美转发。*/

template<typename T>
void print(T& t){
    cout << "lvalue" << endl;
}
template<typename T>
void print(T&& t){
    cout << "rvalue" << endl;
}

template<typename T>
void TestForward(T && v){
    print(v);
    print(std::forward<T>(v));
    print(std::move(v));
}

int main(){
    TestForward(1);
    int x = 1;
    TestForward(x);
    TestForward(std::forward<int>(x));
    return 0;
}

————————————————
版权声明：本文为CSDN博主「coolwriter」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/coolwriter/article/details/80970718
~~~



