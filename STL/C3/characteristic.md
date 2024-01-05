3.1.1nullptr and std::nullptr

//allows u to replace 0 or NULL to indicate a pointer pointing to so-called no value;
//for example:
```C++
    void f(int);
    void f(void*);

    f(0);//calls f(int)
    f(NULL); //calls f(int)
    f(nullptr); //calls f(void*)

```

3.1.2:auto is used to complete the automatic derivation(推导、派生) of the type
//for example:
```C++
double f()
auto d = f(); //d has type double
```
   
//attention: auto declares a variables whoes type is automatically deduced(推导) based on its initial value,make sure initialize it;

3.1.3 
Uniform Initialization(一致性初始化) and Initializer list(初值列)

how to Initialize a variables or objects?
```C++
//Uniform Initialization 
int values[]{1,2,4};
std::vector<int>v{2,3,5,7,4};
std::vector<std::string>str{"asda","sad","dasd"};
std::vector<double>c{4.0,3.0};//equivalent to c(4.0,3.0)

```
Initializer list would cause so called value initialization,it means though a local variable belongs to some types will be initialized to 0 or nulllptr if it is a pointer;
```C++
int i; //i has undefined value
int j{}; //j is initialized by 0
int *p; //p has undefined value
int *q{};//q is initialized by nullptr


```
but narrowing(窄化) causes the reduction of accuracy（精度降低）or numerical changing(数值变动) are Not established;
```C++

int x1(5.3);   //x1 = 5
int x2 = 5.3;//x2 = 5
int x3{5.0}; //error:narrowing
char c1{7}; //OK 7 is an int,this is not narrowing
char c2{99999};//ERROR:does not fit into a char

```
to support conception of "initializer lists for user-defined types"
C++11 provides class template std::initializer_list<> 

```C++
void print(std::initializer_list<int>vals){
    for(auto p = vals.begin();p != vals.end();++p){
        std::cout << *p << "\n";
    }
}
print({12,3,5,7,4,2,2});
```

when "indicates the number of arguments" and "indicates a Initializer list" exists,list wins;

```C++
class P {
public:
    P(int,int);
    P(std::initializer_list<int>);

};

P p(77,5);//calls P(int,int);
P q{77,5}; //calls P(std::initializer_list<int>);
P r{77,3,4};//calls P(std::initializer_list<int>);
P s = {4,6};//calls P(std::initializer_list<int>);

```
"explicit" affects constructor that accept more than one argument because "Initializer list" exixts;
u can invalidated "Multi-value automatic type conversion", even if initialized with =;

```C++

class P {
public:
    P(int a,int b){

    }
    explicit P(int a,int b,int c){

    }
};

P x(77,5);//ok
P y{77,5};//ok
P z {77,5,3};//ok
P v = {77,5}; //ok implicit type conversion allowed
P w = {44,6,2};//ERROR  no implicit type conversion allowed
```
if explicit constructor accepts "initializer list", it loses the implicit conversion ability of "initializer list with 0,1 or more intial values";

3.1.4 for 
explicit type conversion(显示类型转换)
.....

3.1.5 Move and Rvalue Reference
C++11 support move semantic(搬迁语义) allows unnecessary copy and temporary objects

```C++
void createAndInsert(std::set<X>& coll){
    X x;//create an object of type X
    ...
    coll.insert(x);//insert it into the passed collection(传递过来的集合)
}
//insert it into the passed collection,collection provides a member fun to create an internal copy of the incoming element 
```
```C++
X x
coll.insert(x);//copy

coll.insert(x+x);//moves contents of temporary rvalue

coll.insert(std::move(x)); //moves  contents of x into coll;

//x could be moved instead of being copied;however std::move itself doesn't do any moving work,it turns a actual argument into a so-called rvalue reference,which could be declared as type of X&&.This new type claims the content of rvalue can be changed;  
```
```C++
namespace std{
    template <typename T,...>class set{
    public:
        insert(const T& x);//for lvalues:copies the value 
        insert(T&& x);//for rvalues: moves the value

    };
}
class X{
public:
    X(const X&lvalue);//copy constructor
    X(X&& rvalue);  //move constructor
}

```
overloaded rules for rvalue and lvalue reference

one:
    if implements 
        void foo(X&)
    does not implement
        void foo(X&&)
    foo() can be used becaues of lvalue but rvalue;
two:
    if 
        void foo(const X&);
    does not 
        void foo(X&&)
    foo() can be used by lvalue and rvalue

three:
    if 
        void foo(X&);
        void foo(X&&);
    or 
        void foo(const X&);
        void foo(X&&);
    rvalue and lvalue using foo() is available ,version of rvalue provides move func;

return rvalue reference
```C++
X&& foo(){
    X x;
    ...
    return x;//error 
}
//rvalue reference is a reference,return a obj pointed by it,means return a  reference that points to a unexisting
```

3.1.6 string literal

Raw String Literal
character sequence(字符序列)
...

Encoded String Literal
...

3.1.7 noexcept
C++11 provides noexcept to indicate a fun can't or will no throw an exception;
    void foo() noexcept;
declares foo() won't throw an exception,procedure will terminate if foo() throw an exception;

noexcept aims at many questions showed by empty exception specification(空异常明细);

    Runtime checking(运行期检验)...

    Runtime overhead(运行期开销)...

    unusable in generic code: P25
















