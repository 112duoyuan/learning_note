#### weak_ptr 的几个应用场景 —— 观察者、解决循环引用、弱回调

~~~markdown
weak_ptr不会改变资源的引用计数，只是一个观察者的角色，通过观察shared_ptr来判定资源是否存在
weak_ptr持有的引用计数，不是资源的引用计数，而是同一个资源的观察者的计数
weak_ptr没有提供常用的指针操作，无法直接访问资源，需要先通过lock方法提升为shared_ptr强智能指针，才能访问资源

~~~

#### weak_ptr 观察者 —— 基本功能

成员函数use_count() 观测资源引用计数

成员函数expired() 功能相当于 use_count()==0 表示被观测的资源(也就是shared_ptr的管理的资源)是否被销毁

成员函数lock()从被观测的shared_ptr获得一个可用的shared_ptr对象， 进而操作资源。但当expired()==true的时候，lock()函数将返回一个存储空指针的shared_ptr

~~~c++
class CTxxx {
public:    
	CTxxx() {printf( "CTxxx cst\n" );}
	~CTxxx() {printf( "CTxxx dst\n" );    
};
    
int main() {
    std::shared_ptr<CTxxx> sp_ct(new CTxxx);
    std::weak_ptr<CTxxx> wk_ct = sp_ct;
    std::weak_ptr<CTxxx> wka1;
    {
        std::cout << "wk_ct.expired()=" << wk_ct.expired() << std::endl;
        std::shared_ptr<CTxxx> tmpP = wk_ct.lock();
        if (tmpP) {
            std::cout << "tmpP usecount=" << tmpP.use_count() << std::endl;
        } else {
            std::cout << "tmpP invalid" << std::endl;
        }
        std::shared_ptr<CTxxx> a1(new CTxxx);
        wka1 = (a1);
    }
    std::cout << "wka1.expired()=" << wka1.expired() << std::endl;
    std::cout << "wka1.lock()=" << wka1.lock() << std::endl;
 
    std::shared_ptr<CTxxx> cpySp = wka1.lock();
    if (cpySp) std::cout << "cpySp is ok" << std::endl;
    else std::cout << "cpySp is destroyed" << std::endl;
    return 1;
}
~~~

weak_ptr解决循环引用问题 —— 引用对象，用weak_ptr

请注意强弱智能指针的一个重要应用规则：定义对象时，用强智能指针shared_ptr，在其它地方引用对象时，使用弱智能指针weak_ptr。

~~~c++
class B; // 前置声明类B
class A
{
public:
	A() { cout << "A()" << endl; }
	~A() { cout << "~A()" << endl; }
	weak_ptr<B> _ptrb; // 指向B对象的弱智能指针。引用对象时，用弱智能指针
};
class B
{
public:
	B() { cout << "B()" << endl; }
	~B() { cout << "~B()" << endl; }
	weak_ptr<A> _ptra; // 指向A对象的弱智能指针。引用对象时，用弱智能指针
};
int main()
{
    // 定义对象时，用强智能指针
	shared_ptr<A> ptra(new A());// ptra指向A对象，A的引用计数为1
	shared_ptr<B> ptrb(new B());// ptrb指向B对象，B的引用计数为1
	
    // A对象的成员变量_ptrb也指向B对象，B的引用计数为1，因为是弱智能指针，引用计数没有改变
	ptra->_ptrb = ptrb;
	// B对象的成员变量_ptra也指向A对象，A的引用计数为1，因为是弱智能指针，引用计数没有改变
	ptrb->_ptra = ptra;

	cout << ptra.use_count() << endl; // 打印结果:1
	cout << ptrb.use_count() << endl; // 打印结果:1

	/*
	出main函数作用域，ptra和ptrb两个局部对象析构，分别给A对象和
	B对象的引用计数从1减到0，达到释放A和B的条件，因此new出来的A和B对象
	被析构掉，解决了“强智能指针的交叉引用(循环引用)问题”
	*/
	return 0;
}

~~~

线程安全的对象回调与析构 —— 弱回调

~~~markdown
有时候我们需要“如果对象还活着，就调用它的成员函数，否则忽略之”的语意，就像Observable::notifyObservers()那样，我称之为“弱回调”。这也是可以实现的，利用weak_ptr，我们可以把weak_ptr绑到boost::function里，这样对象的生命期就不会被延长。然后在回调的时候先尝试提升为shared_ptr，如果提升成功，说明接受回调的对象还健在，那么就执行回调；如果提升失败，就不必劳神了。
~~~

~~~markdown
注：muduo的源代码，该源码中对于智能指针的应用非常优秀，其中借助shared_ptr和weak_ptr解决了这样一个问题，多线程访问共享对象的线程安全问题，解释如下：线程A和线程B访问一个共享的对象，如果线程A正在析构这个对象的时候，线程B又要调用该共享对象的成员方法，此时可能线程A已经把对象析构完了，线程B再去访问该对象，就会发生不可预期的错误。
~~~

~~~c++
class Test
{
public:
	// 构造Test对象，_ptr指向一块int堆内存，初始值是20
	Test() :_ptr(new int(20)) 
	{
		cout << "Test()" << endl;
	}
	// 析构Test对象，释放_ptr指向的堆内存
	~Test()
	{
		delete _ptr;
		_ptr = nullptr;
		cout << "~Test()" << endl;
	}
	// 该show会在另外一个线程中被执行
	void show()
	{
		cout << *_ptr << endl;
	}
private:
	int *volatile _ptr;
};
void threadProc(weak_ptr<Test> pw) // 通过弱智能指针观察强智能指针
{
	// 睡眠两秒
	std::this_thread::sleep_for(std::chrono::seconds(2));
	/* 
	如果想访问对象的方法，先通过pw的lock方法进行提升操作，把weak_ptr提升
	为shared_ptr强智能指针，提升过程中，是通过检测它所观察的强智能指针保存
	的Test对象的引用计数，来判定Test对象是否存活，ps如果为nullptr，说明Test对象
	已经析构，不能再访问；如果ps!=nullptr，则可以正常访问Test对象的方法。
	*/
	shared_ptr<Test> ps = pw.lock();
	if (ps != nullptr)
	{
		ps->show();
	}
}
int main()
{
	// 在堆上定义共享对象
	shared_ptr<Test> p(new Test);
	// 使用C++11的线程，开启一个新线程，并传入共享对象的弱智能指针
	std::thread t1(threadProc, weak_ptr<Test>(p));
	// 在main线程中析构Test共享对象
	// 等待子线程运行结束
	t1.join();
	return 0;
}
//运行上面的代码，show方法可以打印出20，因为main线程调用了t1.join()方法等待子线程结束，此时pw通过lock提升为ps成功，见上面代码示例。

~~~

~~~c++
//如果设置t1为分离线程，让main主线程结束，p智能指针析构，进而把Test对象析构，此时show方法已经不会被调用，因为在threadProc方法中，pw提升到ps时，lock方法判定Test对象已经析构，提升失败！main函数代码可以如下修改测试：
int main()
{
	// 在堆上定义共享对象
	shared_ptr<Test> p(new Test);
	// 使用C++11的线程，开启一个新线程，并传入共享对象的弱智能指针
	std::thread t1(threadProc, weak_ptr<Test>(p));
	// 在main线程中析构Test共享对象
	// 设置子线程分离
	t1.detach();
	return 0;
}
~~~

