#### 一、赋值初始化

~~~c++
//如果类有默认构造函数
object * p = new object[3];

//类没有构造函数
//没有默认构造，有自定义的(object(contx* c,stack* s){})
object *p = new object[3]{{cct,this},{cct,this},{cct,this}};
（但这个要求object构造函数前不能有explicit，否则无法将{cct,this}隐式转换成object）
或
 object *p = new object[3]{object（cct,this）,object（cct,this）,object（cct,this）};
（要求有拷贝构造函数 object::object(const object&)）
    
 
~~~

~~~c++
#include <iostream>
using namespace std;
class Acct
{
public:
    // Define default constructor and a constructor that accepts
    //  an initial balance.
    Acct() {
      balance = 0.0;
      cout << "no var create...." <<  endl;
     }
    Acct( double init_balance ,double init_cc ) { 
         balance = init_balance;
		 cc = init_cc;
        cout <<  "with var create..." << endl; 
    }
  ~Acct(){
      cout <<  "delete..." << endl;
   }
private:
    double balance;
	double cc;
};
 
int main()
{
    //栈中创建对象数组
     Acct myAcct[6];
    //堆中创建对象数组
    Acct *CheckingAcct = new Acct[3];
    Acct *SavingsAcct  = new Acct[3] {Acct(34.98,2), Acct(131.4,2), Acct(521.1,2)};
	Acct *SavingsAcct2 = new Acct[3] {{34.98,2}, {131.4,2}, {521.1,2}};
    delete [] CheckingAcct;
    delete [] SavingsAcct ;
    // ...
}
~~~

#### 二、指针数组

~~~c++
typedef  Acct* ACCP;
ACCP bestPieces[10];
for(int i = 0;i < 10;i++){
 	bestPieces[i] = new Acct(ba;ance,cc);   
}//注：记得将此数组所指的所有对象删除。如果忘了会产生资源泄露。
~~~

#### 三、上面的只适合静态数组，动态数组用C++11的allocator

~~~c++
//#include "CAnimal.h"
#include <memory>
#include <iostream>
 
using namespace std;
 
class Animal{
public:
#if 0        //即使为0，没有默认构造也是可以，
    Animal() : num(0){
        cout << "Animal constructor default" << endl;
    }
#endif
    Animal(int _num) : num(_num){
        cout << "Animal constructor param" << endl;
    }
 
    ~Animal() {
        cout << "Animal destructor" << endl;
    }
 
    void show(){
        cout << this->num << endl;
    }
private:
    int num;
};
 
/*
    由于allocator将内存空间的分配和对象的构建分离
    故使用allocator分为以下几步:
    1.allocator与类绑定，因为allocator是一个泛型类
    2.allocate()申请指定大小空间
    3.construct()构建对象，其参数为可变参数，所以可以选择匹配的构造函数
    4.使用，与其它指针使用无异
    5.destroy()析构对象，此时空间还是可以使用
    6.deallocate()回收空间
*/
 
int main()
{
    allocator<Animal> alloc;        //1.
    Animal *a = alloc.allocate(5);    //2.
 
    //3.
    /*void construct(U* p, Args&&... args);
   在p指向的位置构建对象U，此时该函数不分配空间，pointer p是allocate分配后的起始地址
   constructor将其参数转发给相应的构造函数构造U类型的对象，相当于 ::new ((void*) p) 
   U(forward<Args> (args)...);
   */
    alloc.construct(a, 1);
    alloc.construct(a + 1);
    alloc.construct(a + 2, 3);
    alloc.construct(a + 3);
    alloc.construct(a + 4, 5);
 
    //4.
    a->show();
    (a + 1)->show();
    (a + 2)->show();
    (a + 3)->show();
    (a + 4)->show();
 
    //5.
    for (int i = 0; i < 5; i++)
    {
        alloc.destroy(a + i);
    }
    //对象销毁之后还可以继续构建，因为构建和内存的分配是分离的
    //6.
    alloc.deallocate(a, 5);
 
    cin.get();
    return 0;
}
 
如果构造函数是多个参数，则可以这样：
  //3.
    alloc.construct(a,    Animal(50,"dog"));
    alloc.construct(a + 1,Animal(30,"cat"));
    alloc.construct(a + 2,Animal(100,"goal"));
    alloc.construct(a + 3,Animal(65,"cow"));
    alloc.construct(a + 4,Animal(5,"bird"));
 
~~~

#### 类有默认构造函数

~~~c++
class animal
{
public:
    animal():num(0)
    {}
    ~animal()
    {}
private:
    int num;
};
 
Animal *ani = new Animal[5];
delete[]ani;
~~~



