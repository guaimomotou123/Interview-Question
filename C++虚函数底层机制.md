## C++虚函数底层机制

C++的虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的。简称为V-Table。 在这个表中，主是要一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其容真实反应实际的函数。这样，在有虚函数的类的实例中这个表被分配在了 这个实例的内存中，所以，当我们用父类的指针来操作一个子类的时候，这张虚函数表就显得由为重要了，它就像一个地图一样，指明了实际所应该调用的函数。

**1．** **无继承的情况**

```
#include <iostream>
using namespace std;

class Base {
public:
    virtual void f() { cout << "Base::f()" << endl; }
    virtual void g() { cout << "Base::g()" << endl; }
    virtual void h() { cout << "Base::h()" << endl; }
};

int main()
{
    typedef void (*Fun)();

    Base *b = new Base;
    cout << *(int*)(&b) << endl; //虚函数表的地址存放在对象最开始的位置

    Fun funf = (Fun)(*(int*)*(int*)b);
    Fun fung = (Fun)(*((int*)*(int*)b + 1));
    Fun funh = (Fun)(*((int*)*(int*)b + 2));


    funf();
    fung();
    funh();

    cout << (Fun)(*((int*)*(int*)b + 3)); // 最后一个位置为0，表明虚函数表的结束

    return 0;
}
```

![](F:\F\Interview Question\image\2.png)

注意：在上面代码中，虚函数表中最后一个节点相当于字符串的结束符，其标志了虚函数表的结束，在Codeblocks下打印为0。  



**2．** **继承，无虚函数覆盖的情形** 

```
#include <iostream>
using namespace std;

class Base {
public:
    virtual void f() { cout << "Base::f()" << endl; }
    virtual void g() { cout << "Base::g()" << endl; }
    virtual void h() { cout << "Base::h()" << endl; }
};

class Derive: public Base {
    virtual void f1() { cout << "Derive::f1()" << endl; }
    virtual void g1() { cout << "Derive::g1()" << endl; }
    virtual void h1() { cout << "Derive::h1()" << endl; }
};

int main()
{
    typedef void (*Fun)();

    Base *b = new Derive;
    cout << *(int*)b << endl;
    Fun funf = (Fun)(*(int*)*(int*)b);
    Fun fung = (Fun)(*((int*)*(int*)b + 1));
    Fun funh = (Fun)(*((int*)*(int*)b + 2));
    Fun funf1 = (Fun)(*((int*)*(int*)b + 3));
    Fun fung1 = (Fun)(*((int*)*(int*)b + 4));
    Fun funh1 = (Fun)(*((int*)*(int*)b + 5));


    funf(); // Base::f()
    fung(); // Base::g()
    funh(); // Base::h()
    funf1(); // Derive::f1()
    fung1(); // Derive::g1()
    funh1(); // Derive::h1()

    cout << (Fun)(*((int*)*(int*)b + 6)) << endl;
    return 0;
}
```

![](F:\F\Interview Question\image\3.png)

从上表可以发现：

1．  虚函数按照其声明顺序放于表中。

2．  父类的虚函数在子类的虚函数前面。 



**3．** **继承，虚函数覆盖的情形** 

```
#include <iostream>
using namespace std;

class Base {
public:
	virtual void f() { cout << "Base::f()" << endl; }
	virtual void g() { cout << "Base::g()" << endl; }
	virtual void h() { cout << "Base::h()" << endl; }
};

class Derive : public Base {
	virtual void f() { cout << "Derive::f()" << endl; }
	virtual void g1() { cout << "Derive::g1()" << endl; }
	virtual void h1() { cout << "Derive::h1()" << endl; }
};

int main()
{
	typedef void(*Fun)();

	Base *b = new Derive;
	cout << *(int*)b << endl;
	Fun funf = (Fun)(*(int*)*(int*)b);
	Fun fung = (Fun)(*((int*)*(int*)b + 1));
	Fun funh = (Fun)(*((int*)*(int*)b + 2));
	Fun fung1 = (Fun)(*((int*)*(int*)b + 3));
	Fun funh1 = (Fun)(*((int*)*(int*)b + 4));


	funf(); // Derive::f()
	fung(); // Base::g()
	funh(); // Base::h()
	fung1(); // Derive::g1()
	funh1(); // Derive::h1()

	cout << (Fun)(*((int*)*(int*)b + 5)) << endl;
	return 0;
}
```

![](F:\F\Interview Question\image\4.png)

从上表可以看出：

1．  覆盖的f()函数被放到了虚表中原来父类虚函数的位置。

2．  没有被覆盖的函数依旧。

3．  可通过获取获取成员函数指针来调用成员函数（即使是private类型的），带 来一定安全性的影响。



**4．** **多继承的情形** 

```
#include <iostream>
using namespace std;

class Base1 {
public:
    virtual void f() { cout << "Base1::f()" << endl; }
    virtual void g() { cout << "Base1::g()" << endl; }
    virtual void h() { cout << "Base1::h()" << endl; }
};

class Base2 {
public:
    virtual void f() { cout << "Base2::f()" << endl; }
    virtual void g() { cout << "Base2::g()" << endl; }
    virtual void h() { cout << "Base2::h()" << endl; }
};


class Base3 {
public:
    virtual void f() { cout << "Base3::f()" << endl; }
    virtual void g() { cout << "Base3::g()" << endl; }
    virtual void h() { cout << "Base3::h()" << endl; }
};


class Derive: public Base1,public Base2, public Base3 {
    virtual void f() { cout << "Derive::f()" << endl; }
    virtual void g1() { cout << "Derive::g1()" << endl; }
};

int main()
{
    typedef void (*Fun)();

    Derive d;
    Base1 *b1 = &d;
    Base2 *b2 = &d;
    Base3 *b3 = &d;


    b1->f(); //Derive::f()
    b2->f(); //Derive::f()
    b3->f(); //Derive::f()
    b1->g(); //Base1::g()
    b2->g(); //Base2::g()
    b3->g(); //Base3::g()


    Fun b1fun = (Fun)(*(int*)*(int*)b1);
    Fun b2fun = (Fun)(*(int*)*((int*)b1+1));
    Fun b3fun = (Fun)(*(int*)*((int*)b1+2));

    b1fun(); // Derive::f()
    b2fun(); // Derive::f()
    b3fun(); // Derive::f()

    return 0;
}
```

![](F:\F\Interview Question\image\5.png)

从上表可以看出：

1． 每个父类都有自己的虚表。

2． 子类的成员函数被放到了第一个父类的表中。（所谓的第一个父类是按照声明顺序来判断的）

3． 对于多继承无虚函数覆盖的情况，布局与上图类似(Derive的位置对应Base)。

此文章转载自：http://blog.chinaunix.net/uid-20196318-id-28833.html