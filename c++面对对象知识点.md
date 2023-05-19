[toc]


# 头文件与类的声明


## 头文件正规写法

![image.png](https://note.youdao.com/yws/res/21032/WEBRESOURCEdf407cdf137724c9334af7117f2972fc)

如果第一次调用，没有定义，所以将会定义，并且执行里面的代码。
如果第n(n>1)次调用，已经定义过了，所以不会执行里面的代码。

现在通常也用`#pragma once`：防止重复包含

## 头文件的布局

0. 前置声明
1. 类的声明
2. 类的定义

- 类的声明

分为`class head` 和 `class body`

`class body`中有些函数在此直接定义，有些在`body`外定义

函数若在`class body`内定义完成，便形成`inline`。

### `inline`

内联函数会将函数的代码副本放置在每个调用函数的地方。

即：程序在编译器运行时，编译器将程序中出现的内联函数的调用表达式用内联函数的函数体进行替换，而对于其他的函数，都是在运行时候才被替代。空间代价换时间的i节省

在使用内联函数时：

1. 在内联函数内不允许使用循环语句和开关语句；
2. 内联函数的定义必须出现在内联函数第一次调用之前；
3. 类结构中所在的类说明内部定义的函数是内联函数。

写`inline`只是对编译器的建议，最终还是由编译器决定

## 构造函数

```
class complex
{
    public:
        complex(double r =0 ,double i =0): re(r) , im(i) {};
    //           默认构造函数                 初始化列表
}
```

# 重载

函数重载的关键：函数的参数列表

1. c++允许函数定义名称相同，但它们的特征表（函数的参数列表）不同。

2. 如果被重载的函数不与任何原型匹配，则视为错误（如果只有一个函数，不发送重载，则会发生类型转换，调用该函数）

3. 类型引用和类型本身，视为同一个特征标

4. `const` 与 非 `const` 有时可以重载，有时不能重载 （注意非`const`可作为`const`的参数，但是`const`不可作为非`const`的参数）

```c
Record lookup(Phone);
Record lookup(const Phone); //重复声明

Record lookup(Phone *);
Record lookup(Phone * const); //重复声明 const关键字表示指针所指向的对象是不可修改的

```

```c
Record lookup(Account&);  //函数作用于Account的引用
Record lookup(const Account&); // 新函数，作用于常量引用

Record lookup(Account*); //新函数，作用于指向Account的指针
Record lookup(const Account*); //新函数，作用于指向常量的指针

//成员函数的const属于特征表，如何理解？this指针，加和不加const分别是 普通指针和指向常量的指针
double imag(const double im) {...} 
double imag(const double im)const {...}

```

5. 重载是和特征标有关系！所以和返回类型无关。

6. `main`函数不能重载

7. 省略形参名和形参名字不同但类型相同，不能重载。因为不影响形参列表的内容，类型别名只是为已存在的类型提供一个名字，本质没有什么不同。

# 将构造函数放在`private`里的情况

不允许被外界创建对象。

![image.png](https://note.youdao.com/yws/res/21033/WEBRESOURCE16913ac5d0c541591d3168fd018e26e6)

如何调用？

`A::getInstance().setup()`;


# 值传递、引用传递

1. 引用底层是指针。

> 参数传递

2. 值传递，相当于把要传递的内容复制一份，然后传递过去。引用传递，相当于把要传递的内容的地址发送过去。（由于1）
3. 引用加`const`即：`complex & operator += (const complex&);`，此时代表不希望传入的变量被修改。
4. 尽量传递引用

> 返回值传递

5. 返回值`by value`，则相当于拷贝一份，然后返回，函数结束后原件内存空间释放。返回值`by reference`，则相当于把传递的内容地址发送过去。（此时就要考虑到，如果`return`返回的内容是临时变量的问题）

```c
Obj& fun() {
  Obj obj;
  // do sth;
  return obj;
}
```

> 在函数返回时会发生以下两个操作：1.编译器会创建一个新的 Obj 类型的对象，将 obj 的值复制到新的对象中。2.返回新创建的对象的副本。这个过程叫做“返回值优化”（Return Value Optimization, RVO），它使得返回值的对象不需要被复制多次，而是可以直接在调用函数的地方创建。在这个例子中，尽管返回的是一个临时变量，但由于返回值优化的机制，它并不会被销毁，而是直接被传递到调用函数的地方。这样做可以避免不必要的内存分配和复制，提高程序的性能。

```c
Obj fun() {
  Obj obj;
  // do sth;
  return obj;
}
```

> 此时返回值是值传递，那么在返回时会触发拷贝构造函数或移动构造函数，将函数内部的临时变量的值复制或移动到返回值所在的位置。

6. 返回临时变量，尽量用`by value`
6. 尽量传递引用
7. 传递者无需知道接受者是以什么形式（by reference/by value）接受


# 移动拷贝构造函数

传统 C++ 通过拷贝构造函数和赋值操作符为类对象设计了拷贝/复制的概念，但为了实现对资源的移动操作， 调用者必须使用先复制、再析构的方式，否则就需要自己实现移动对象的接口。右值引用的出现恰好就解决了这个问题

使用移动构造函数可以避免进行不必要的拷贝操作，提高程序的性能。特别是当对象的资源非常大时，移动构造函数可以将对象的资源从一个对象移动到另一个对象，而不需要进行昂贵的拷贝操作。

![image.png](https://note.youdao.com/yws/res/21034/WEBRESOURCE6e6b1f1bd48c60af2b316dc937b67e30)

`a`在函数作用域内，正常构造和析构。
此时`b`触发移动拷贝构造函数，将将亡值的指针指向`nullptr`，返回右值的指针。相当于将临时变量的地址空间保留下来了，不会被销毁。

# 友元


1. 成为**友元**，类可以允许其他类或函数访问它的非公有成员
2. 最好在友元声明之外再专门对函数进行一次声明

```c
class Sales_data{
    //为Sales_data的非成员函数所做的友元声明
    friend Sales_data add(const Sales_data &,const Sales_data &);
    friend std::istream &read(std::istream &,Sales_data &);
    
    public:
        //-------
    private:
        std::string bookNO;
        // ----
}

//Sales_data接口的非成员组成部分的声明
Sales_data add(const Sales_data &,const Sales_data &);
std::istream &read(std::istream &,Sales_data&);
```

3. 可以在一个类中声明友元类，则友元类的成员函数可以访问此类包括非公有成员在内的所有成员。每个类负责控制自己的友元类或者友元函数，不能传递。
4. 可以令某个成员函数作为友元。但必须按如下方式设计

```c
class Screen{
    //Window_mgr::clear 必须在Screen类之前被声明
    friend void Window_mgr::clear(ScreenIndex);
}

//1. 定义Window_mgr类，其中声明clear函数，但不能定义它
//2. 定义Screen,包括对clear的友元声明
//3. 定义clear，此时它才可以使用Screen的成员
```

4. **相同`class`的各个`object`互为`friends`**


## 设计类需要注意的地方

1. 数据在`private`里
2. 参数尽可能以`reference`传递，加不加`const`看情况，如果不需要改变所传递的参数，则加`const`
3. 返回值尽可能以`reference`传递
4. `class body`需要加`const`的地方一定需要加。例如成员函数不改变成员变量的值，`int add () const { return re;};`
5. 构造函数初始化列表，尽可能使用


# 操作符重载

## 成员函数重载

1. 所有成员函数都带着一个隐藏的参数：`this` - 谁调用函数，`this`就是那个调用者;

> 注意：`this` 不能在参数列表中写出

```c
inline complex & complex::operator += (const complex & r){
    return _doapl(this,r);
}

--- 等价于
  c2  +=             c1
(this,const complex & r)
```


## 非成员函数重载

```c
inline complex operator + (const complex& x, const complex & y){
    return complex(real(x) +real(y) , imag(x) + imag(y));
}
```

> 对于 `<<`操作符，只能用非成员函数重载

因为如果是成员函数写法，对于`cout << c1`，`<<`是作用于`cout`上，即`count.<<()`形式。
所以只能用非成员函数重载方法

```c
ostream & operator << (ostream& os, const complex& x){
    return os << '(' << real(x) << ',' << imag(x) << ')';
}
```

对于`cout << c1 << conj(c1);` 先执行 `count << c1`，再执行后面的

# 三大函数：拷贝构造、拷贝复制、析构

如果`class`带指针，不能用编译器默认的拷贝构造和拷贝复制，需要自己写。
不然的话，则为浅拷贝，会指向同一处地址空间，而不是两个空间同一内容。

```c
class String
{
public:
    String(const char* cstr = 0);
    String(const String& str);//拷贝构造函数
    String& operator = (const String& str);//拷贝复制函数
    ~String();//析构函数
}
```

拷贝构造函数：

```c
inline String::String(const String& str){
    m_data = new char[strlen(str.m_data) + 1];
    strcpy(m_data,str.m_data);
}
```


```c
{
    String s2(s1);//1
    String s2 = s1;//2
    // 1等同于2
}
```

拷贝复制函数：

```c
inline String& String::operator = (const String& str){
    if(this == &str) return *this;
    delete[] m_data;
    m_data = new char[strlen(str.m_data) +1]；
    strcpy(m_data, str.m_data);
    return *this;
}
```

# 堆、栈和内存管理

栈-`stack`：是存在于某个作用域的一块内存空间。例如当你调用函数，函数本身即会形成一个`stack`用来放置它所接受的参数、返回地址以及临时变量。

堆-`heap`：由操作系统提供的一块全局的内存空间。程序可动态分配从中获得若干区块


```c
class Complex { ... };
...
{
    Complex c1(1,2);//c1所占用的空间来自stack
    Complex* p = new Complex(3);//
}
```

- 生命周期

`stack`的生命期：所在作用域范围

`static local objects`的生命期：在作用域结束仍存在，直到整个程序结束

`global object`的生命期：在整个程序结束之后才结束，作用域是整个程序

`heap object`的生命期：在它被`deleted`之后结束

- new 操作

![image.png](https://note.youdao.com/yws/res/21035/WEBRESOURCE95ce40a1e62a91d9629d42879c1d764e)

`new`：先分配内存，再调用构造函数

- delete 操作

![image.png](https://note.youdao.com/yws/res/21036/WEBRESOURCE44e6021ac377d3b384d00e2c495f537b)

![image.png](https://note.youdao.com/yws/res/21037/WEBRESOURCE360241eb4283cdd17225ccfa28658387)

`delete`：先调用析构函数，再释放内存空间

- 分配所得的内存块

![image.png](https://note.youdao.com/yws/res/21038/WEBRESOURCE78a2dcd1b00960d6a0cdd75cd141247d)

1. 上下`cookie`4字节大小，用于记录内存块的大小，内存块的大小必须是16字节的整数倍，所以最后4位都为0，故用最后1为 0/1 表示 操作系统是借出（1）还是收回（0） 即 程序是获得（1）还是失去（0）

2. 灰色：每一格4字节，调试模式下需要的内存

3. `array new`要搭配`array delete`

![image.png](https://note.youdao.com/yws/res/21039/WEBRESOURCE2237af3960490fe96293081a3e2671ce) 

![image.png](https://note.youdao.com/yws/res/21040/WEBRESOURCE4d4fed44ce48ca53cad49278606550cb)

如果使用`delete[] p`，会调用三次析构函数，而使用`delete p`，只会调用一次析构函数，则指针指向的地址空间有两块没有清除。注意两种方法都会清理指针所在的地址空间。


# static

1. 静态成员变量只有一份，静态成员函数也只有一份
2. 静态成员函数没有`default point`,所以不能访问非静态成员变量
3. 如果是静态成员变量，则要在类外进行初始化
4. 调用静态函数的方式

```c
//1.通过object调用
a.set_rate(5.0);
//2.通过class name调用
Account::set_rate(5.0);
```

```
//class 只产生一个对象 单例模式
class A{
public:
    static A& getInstance (return a);
    setup(){....};
private:
    A();
    A(const A& rhs);
    static A a;
    .....
}

A::getInstance().setup();
```

如果把函数成员声明为静态的，就可以把函数与类的任何特定对象独立开来。 静态成员函数即使在类对象不存在的情况下也能被调用， 静态函数 只要使用类名加范围解析运算符 `::` 就可以访问。


# 模板

1. 对`class template`要指出`type`要绑定为什么，对`function template`不用指出，编译器会对其进行实参推导

> 成员模板

![image.png](https://note.youdao.com/yws/res/21054/WEBRESOURCEa88ef0b0e7bd145963b6f932fbc38c6e)

`U1`和`U2`必须满足`pair`初始化列表的赋值动作

例如：

```c
class Base1{};
class Derived1::public Base1{};

class Base2{};
class Derived2::public Base2{};

pair<Derived1,Derived2>p;

//下面是否可以？
pair<Base1,Base2>p2(p);
//上述等价于
pair<Base1,Base2>p2(pair<Derived1,Derived2>());
//可以，因为满足pair的构造函数

//反过来是否可以
pair<Derived1,Derived2>p3(p2);
//不可以，因为不满足pair的构造函数，父类对象不能赋给子类对象
```

# namespace

1. 有二种使用方法

```
namespace std { .... }

//1. using directive  
using namespace std;
//2. using declaration
using std::cout;
```

![image.png](https://note.youdao.com/yws/res/21053/WEBRESOURCE06e831bc64ebf3778bf68ec25fc033a5)

# Composition 组合

> 非指针类型

![image.png](https://note.youdao.com/yws/res/21043/WEBRESOURCE397d775b987cf56ccf21f6a158622402)

![image.png](https://note.youdao.com/yws/res/21041/WEBRESOURCE4a800b8af075b717d6e5af2be787d827)

- 构造由内而外

`Container`的构造函数首先调用`Component`的`default`构造函数，然后才执行自己的

```c
Container::Container(...):Component(){...};
```

- 析构由外而内

`Container`的析构函数先执行自己，然后才调用`Conponent`的析构函数

```c
Container::~Container(...){...~Component() };
```


> 指针类型：

![image.png](https://note.youdao.com/yws/res/21042/WEBRESOURCE73c857efb408798459fb9866db06de4c)

# 继承

> 继承下的构造和析构

![image.png](https://note.youdao.com/yws/res/21044/WEBRESOURCE8be1093f7098f166b8225678d54a18f0)

- 构造由内而外

`Derived`的构造函数首先调用`Base`的`default`构造函数，然后才执行自己的

```c
Derived::Derived(...):Base(){...};
```

- 析构由外而内

`Derived`的析构函数先执行自己，然后才调用`Base`的析构函数

```c
Derived::~Derived(...){...~Base() };
```

1. `base class`的析构函数必须是`virtual`,否则会出现`undefined behavior`

- 如果为该情况：

![image.png](https://note.youdao.com/yws/res/21046/WEBRESOURCEc266ea20e7f87f7159e5481139d7e2db)

![image.png](https://note.youdao.com/yws/res/21047/WEBRESOURCEab298c7e67caeb02a3fbe274e057fef0)

# 虚函数

- `non-virtual`函数：不希望`derived class`重新定义 

`int objectID()const;`

- `virtual`函数：希望`derived class`重新定义，且它已经有默认定义

`virtual void error(const std::string& msg);`

- `pure virtual`函数：希望`derived class`一定要重新定义，你对它没有默认定义

`virtual void draw() = 0;`


> 实例

```
CDocument::OnFileOpen(){
    ...
    Serialize();//1
    ...
}

class CMyDocument : public CDocument
{
    virtual Serialize() {....}//2
};

main(){
    CMyDocument myDoc;
    ...
    myDoc.OnFileOpen();//实际为CDocument::OnFileOpen(&myDoc);
}

```


![image.png](https://note.youdao.com/yws/res/21045/WEBRESOURCE8633711addf25fac1d716939a76b245f)

所以此时步骤1中的`Serialize();`转而执行步骤2的`Serialize();`，等价于`this->Serialize();`，此时`this`为`myDoc`对象即`class CMyDocument`

# 关于父类和子类转换问题

1. “先上转型”（即派生类指针或引用类型转换为其基类类型），本身就是安全的

```c
#include <iostream>
using namespace std;
 
 
class Base
{
public:
    Base() {};
    virtual void Show() { cout << "This is Base calss"; }
};
class Derived :public Base
{
public:
    Derived() {};
    void Show() { cout << "This is Derived class"; }
    void Dfun() { cout << "Dfun"; }
};
int main11()
{
    Base *base;
    Derived *der = new Derived;
    //base = dynamic_cast<Base*>(der); //正确，但不必要。
    base = der; //先上转换总是安全的
    base->Show();   // This is Derived class      子类转父类，安全，且转换后，还是调用子类的虚函数，
    //base->Dfun(); //ERR: Dfun不是Base的成员   子类转父类，安全，但是转换后，不能再使用子类的非虚函数，
    return 0;
}
```


2. dynamic_cast用于类继承层次间的指针或引用转换。主要还是用于执行“安全的向下转型（safe downcasting）”，也即是基类对象的指针或引用转换为同一继承层次的其他指针或引用。对于“向下转型”有两种情况。一种是基类指针所指对象是派生类类型的，这种转换是安全的；另一种是基类指针所指对象为基类类型，在这种情况下dynamic_cast在运行时做检查，转换失败，返回结果为0；

```c
#include <iostream>
using namespace std;


class Base
{
public:
    Base() {};
    virtual void Show() { cout << "This is Base calss" << endl; }
};
class Derived :public Base
{
public:
    Derived() {};
    virtual void Show() { cout << "This is Derived class" << endl; }
    void Dfun() { cout << "Dfun" << endl; }
    char name[10];
};
int main()
{
    //场景一：
    Base* base = new Base;
    //Derived *der = base; //error C2440 : “初始化”: 无法从“Base * ”转换为“Derived *”
    Derived* der = dynamic_cast<Derived*>(base);   //执行完后  der是空指针,所以这个转换时无意义的

    //der->Show();  //编译没问题，运行就会报  der是nulptr的段错误。
    der->Dfun(); //OK   实际上同下一条语句
    //((Derived*)(0))->Dfun(); //OK
    //cout<<sizeof(((Derived*)(0))->name)<<endl;  // OK  10

    //正确使用场景： 父类指针指向子类对象，需要使用子类对象的非虚接口时
    Base* pB = new Derived;
    //Derived *pD = pB; //“初始化”: 无法从“Base * ”转换为“Derived *”  解析：多态是运行时才有的，
    Derived* pD = dynamic_cast<Derived*>(pB);
    pD->Dfun();  // 如果直接使用 pB->Dfun(); 就会报错，error C2039: 'Dfun' : is not a member of 'Base'
    cout << sizeof(pD->name) << endl;  // OK  10
    pD->Show();

    return 0;
}
```


# 转换函数

> `Fraction`转为`double`

```c
class Fraction
{
public:
    Fraction(int num, int den =1) : m_numerator(num),m_denominator(den){}
    operator double() const {return (double) (m_numerator/m_denominator);}
private:
    int m_numerator;//分子
    int m_denominator;//分母
};

Fraction f(3,5);
double d = 4 + f;//调用operator double

```

# `expilit`和`non-explicit`

> non-explicit-one-argument ctor

```c
class Fraction
{
public:
    Fraction(int num, int den=1):m_numerator(num),m_denominator(den){}
    Fraction operator + (const Fraction& f) { return Fraction(...);}

private:
    int m_numerator;
    int m_denominator;
};

Fraction f(3,5);
Fraction d2 = f + 4;// 4调用 non-explicit ctor将4转为Fraction ，然后调用operator +
```

> conversion function vs non-explicit-one-argument ctor

```c
class Fraction
{
public:
    Fraction(int num, int den=1):m_numerator(num),m_denominator(den){}
    operator double() const {return (double) (m_numerator/m_denominator);}
    Fraction operator + (const Fraction& f) { return Fraction(...);}

private:
    int m_numerator;
    int m_denominator;
};

Fraction f(3,5);
Fraction d2 = f + 4;// error ambiguous
```

> expilcit-one-argument ctor

**`expilit`基本只用于构造函数上**

```c
class Fraction
{
public:
    explicit Fraction(int num, int den =1) : m_numerator(num),m_denominator(den){}
    operator double() const {return (double) (m_numerator/m_denominator);}
    Fraction operator + (const Fraction& f) { return Fraction(...);}
private:
    int m_numerator;//分子
    int m_denominator;//分母
};

Fraction f(3,5);
Fraction d = f + 4;// error conversion from `double` to `fraction` requested

```

# 让1个`class`像指针

## 智能指针

![image.png](https://note.youdao.com/yws/res/21048/WEBRESOURCE7ed079d69fc2f5438c1eef111424ae76)

```c
struct Foo
{
    ....
    void method(void) {...}
};

shared_ptr<Foo> sp (new Foo); //调用shared_ptr的构造函数

//此时将sp当成指针使用 

Foo f(*sp);
sp->method(); //1

```

> 对于1，如果调用`sp->`，不应该是换成`px`吗，为什么还能转化为`px->method()`调用函数

`->`会将得到的东西继续用`->`作用上去

# 迭代器

迭代器：指向容器里的一个元素，可认为是一个智能指针

![image.png](https://note.youdao.com/yws/res/21049/WEBRESOURCE72b0e7a63e0ea61accd1146fca870c83)


```c
list<Foo>::iterator ite;
ite->method();
//意思是调用Foo::method();
//相当于(*ite).method();
//相当于(&(*ite))->method();
```

# 让1个`class`像函数

## 仿函数

如果任何一个东西能接受`()`操作符，则称这个东西为仿函数

![image.png](https://note.youdao.com/yws/res/21051/WEBRESOURCEf40751e8bdc57453fa63fd8eb955bf41)

> 标准库中，仿函数所使用的奇特的base classes

![image.png](https://note.youdao.com/yws/res/21052/WEBRESOURCE9fabb67a8a2ceb7a2d0f126c970f0bff)

## `pair`

```c
template <class T1, class T2>
struct pair{
  T1 first;
  T2 second;
  pair() : first(T1()),second(T2()) {}
  pair(const T1& a, const T2& b):first(a),second(b){}
....
};
```

# 模板特化，模板偏特化

> 模板特化

![image.png](https://note.youdao.com/yws/res/21082/WEBRESOURCEa379e0b7a91e4b528b6834b70ecfd92d)

> 模板偏特化

1. 个数上的偏特化

![image.png](https://note.youdao.com/yws/res/21091/WEBRESOURCE6075efba270fb1bf4b4328f24e67264f)

绑定一定要从左到右，不能跳

2. 范围上的偏特化

任意类型缩小为指针

![image.png](https://note.youdao.com/yws/res/21100/WEBRESOURCE2af3be839957594f5d54f3ae165cfa76)

特化和泛化的`typename`可以不同

# 模板模板参数

![image.png](https://note.youdao.com/yws/res/21104/WEBRESOURCE6a55751ab913807ae1c93b0666ff251a)

`XCls<string,list>mylist1;`：第二个参数为指定容器类型（`vector`、`array`），第一个参数为该容器的类型(`string`、`char`)，则需要用到模板模板参数

> 为什么1不行呢？

因为容器有第二模板类型，或者第三第四。平常没写是因为有默认值，这种情况不行

`allocator`：分配器，用以分配内存，默认参数

![image.png](https://note.youdao.com/yws/res/21126/WEBRESOURCE7249fff3d67216c0df63b6cf3cc724bb)

这个不是模板模板参数：

![image.png](https://note.youdao.com/yws/res/21132/WEBRESOURCE98645bf3bf70e532c21894d87784e989)

因为模板模板参数，是一个未知类型的 容器/智能指针/.... 里的未知类型，即模板里面的模板


# 变参数模板

```c
void print(){ } //1

//2
template <typename T,typename...Types>
void print (const T& firstArg, const Types& ...args){
    cout<< firstArg <<endl;
    print(args...);//拆包，继续调用，直到没有参数调用1
}

print(7.5,"hello",bitset<16>(377),42);
```

`sizeof...(args)`可以获得参数的数量

![image.png](https://note.youdao.com/yws/res/21222/WEBRESOURCEabac23e7747d53e95c99b73c4926cf5b)


`...`就是一个所谓的包


# 引用

![image.png](https://note.youdao.com/yws/res/21229/WEBRESOURCEba3f6972521dd1395cf9bcb28411d082)

```c
typedef struct Stag { int a,b,c,d;} S;

int main(){
    double x =0;
    double* p = &x; //p指向x,p的值是x的地址
    double& r = x;  //r代表x,现在r,x都是0
    
    cout<< sizeof(x) << endl;//8
    cout<< sizeof(p) << endl;//8
    cout<< sizeof(r) << endl;//8
    cout<<  p <<endl;         //0065FDFC
    cout<< *p <<endl;         //0
    cout<<  x <<endl;         //0
    cout<<  r <<endl;         //0
    cout<< &x <<endl;         //0065FDFC
    cout<< &r <<endl;         //0065FDFC
    
    S s;
    S& rs = s;
    cout<< sizeof(s) << endl; //16
    cout<< sizeof(s) << endl; //16
    cout<< &s        << endl; //0065FDE8
    cout<< &rs       << endl; //0065FDE8
}

```

![image.png](https://note.youdao.com/yws/res/21318/WEBRESOURCE95592d0441686e0fb26216242f6b136e)


# `vptr`和`vtbl` 虚指针、虚表

不管有多个虚函数，内存空间会多一个指针，该指针指向虚函数表

继承函数是继承函数的调用权，而不是内存大小

![image.png](https://note.youdao.com/yws/res/21343/WEBRESOURCEf18d4905948002439284a32c8aabe9af)

> 静态绑定

类似于`call xxx(地址)`，相当于将代码和函数绑定在一起，一定调用到某个地址

> 动态绑定

满足三个条件：1.指针调用 2.指针向上转型 3.调用虚函数

则编译器将调用动作编译成 `(* (p->vptr)[n])(p)`，调用地址需要看`p`指的是什么

## `this`

![image.png](https://note.youdao.com/yws/res/21397/WEBRESOURCEe8f86625ace647e667e2f029207cdb0a)

![image.png](https://note.youdao.com/yws/res/21379/WEBRESOURCEb339a308a725aa6aa19e2be60cbec011)

![image.png](https://note.youdao.com/yws/res/21377/WEBRESOURCE099f9b4cc59acebdff20471b784ff06b)


# const 

![image.png](https://note.youdao.com/yws/res/21539/WEBRESOURCE0853a22496f21d5c26161fac9bbf8b23)

1. `const`对象不能调用`non-const`函数

2. 当成员函数的`const`和`non-const`版本同时存在，`const object` 只会调用`const`版本，`non-const object` 只会调用`non-cosnt`版本


## 重载`::operator new`，`::operator delete`,`::operator new[]`,`::operator delete[]`

> 全局

![image.png](https://note.youdao.com/yws/res/21558/WEBRESOURCE8b484b6bdc691699505cf680f51e96f5)

## 重载`member operator new/delete`

> class成员重载

![image.png](https://note.youdao.com/yws/res/21570/WEBRESOURCEea805af1cdb56ba49d4f57e39c33d038)

## 重载`member operator new[]/delete[]

> class成员重载

```c
class Foo{
public:
    void* operator new[](size_t);//1
    void operator delete[](void*,size_t);//2
}

Foo* p = new Foo[N];//1
...
delete[] p;//2

//1
try{
    void* men = operator new(sizeof(Foo)*N + 4);
    p = static_cast<Foo*>(men);
    p->Foo::Foo(); //N次
}

//2
p->~Foo();//N次
operator delete(p);

```

> 为什么Foo数组的大小要加上4?

会有一个空间大小记录下面有几个元素（计数器）

![image.png](https://note.youdao.com/yws/res/21636/WEBRESOURCEe1596329ab74eb903033c8d6db32ff75)

如果有虚函数，大小会+8（不管是不是数组），因为有虚函数，则会存在一个虚指针的地址大小。

> 实例

注意：如果使用`::new`会调用全局的`new`

![image.png](https://note.youdao.com/yws/res/21628/WEBRESOURCE1eccc717ff44867a7ca0e189491b057a)

```c
#include <iostream>
using namespace std;

class Foo
{
public:
    int _id;
    long _data;
    string _str;

public:
    Foo() :_id(0) { cout << "default ctor.this = " << this << "id = " << _id << endl; }
    Foo(int i) :_id(i) { cout << "ctor.this = " << this << "id = " << _id << endl; }
    ~Foo() { cout << "dtor.this = " << this << "id = " << _id << endl; }

    static void* operator new(size_t size);
    static void operator delete(void* pdead,size_t size);
    static void* operator new[](size_t size);
    static void operator delete[](void* pdead,size_t size);
};

void* Foo::operator new(size_t size) {
    cout << "size =" << size << endl;
    Foo* p = (Foo*)malloc(size);

    return p;
}

void Foo::operator delete(void* pdead, size_t size) {
    free(pdead);
}

void* Foo::operator new[](size_t size) {
    cout << "size = " << size << endl;
    Foo* p = (Foo*)malloc(size);
    return p;
}

void Foo::operator delete[](void* pdead, size_t size) {
    free(pdead);
}

int main()
{
    Foo* p = new Foo(7);
    delete p;

    Foo* pArray = new Foo[5];
    delete[]pArray;

    return 0;
}
```

- 结果：

![image.png](https://note.youdao.com/yws/res/21649/WEBRESOURCE1e73aeb6832d51706ad58173419f2a89)

这里`string`占了28个字节

## 重载`new()`,`delete()`

![image.png](https://note.youdao.com/yws/res/21658/WEBRESOURCE7e671416e05ef2245444aaff59d3fe53)

> 不论什么版本，第一个参数列一定要为`size_t`，所以上图例子实际参数有3个

> 实例

![image.png](https://note.youdao.com/yws/res/21664/WEBRESOURCE14d5485418943b588a14e7621117b1b7)


> 什么时候调用重载的`operator delete`呢？

`operator new`调用重载的函数，分配内存后，调用构造函数，如果构造函数发出异常，说明构造失败，要释放刚刚分配的内存，此时调用重载的`operator delete`

![image.png](https://note.youdao.com/yws/res/21679/WEBRESOURCE9973755b69105d2f7b55e75a366fb237)

