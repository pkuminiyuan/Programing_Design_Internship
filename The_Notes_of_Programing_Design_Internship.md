<style>
    h1 { color: lightyellow; }
    h2 { color: lightblue; }
    h3 { color: Lavender; }
</style>

# 程序设计实习笔记

## 1. 类和对象

### 1.1 使用类的成员

对象名.成员名；指针->成员名；引用名.成员名

### 1.2 引用

左值：可以持久储存的变量，可以进行取地址操作。

右值：临时对象或者字面量，不能进行取地址操作。

左值引用：常用于函数参数和返回值，表示对已存在对象的引用，避免拷贝，提高效率。

右值引用：常用于移动语义和完美转发，避免不必要的拷贝。

特殊地，const左值引用可以绑定到左值和右值。

注意，无论左值引用还是右值引用，引用本身都是左值。且引用必须再初始化时进行绑定，绑定后就不能重新绑定。

```const Type& ref = n```意为不能通过```ref```修改```n```的值，但是```n```本身可以修改，因为```n```本身并不知道```ref```的存在。

```const T&```可以绑定到```T&```, ```T```；```T&```不可以绑定到```const T```, ```const T&```，本质上 是确保```const```对象不被改变。

### 1.3 类成员访问权限

private: 私有成员, 只能在成员函数内访问

public: 公有成员, 可以在任何地方访问

protected: 保护成员，类内部和类外部的private，派生类的public

以上三种关键字出现的次数和先后次序都没有限制。

### 1.4 缺省参数

注意只能是最右边的连续若干个。

### 1.5 构造函数

如果不进行初始化，一般类型的值是未定义的。

如果定义了构造函数, 则编译器不生成默认的无参数的构造函数。

对象一旦生成, 就再也不能在其上执行构造函数。

一般而言constructor是public的，但是在单例模式中它是private的，避免用户自行创建实例。

<details>
<summary>+/-单例模式</summary>

```cpp
class Singleton {
private:
    static Singleton* instance;
    Singleton() {} // 私有构造函数

public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

Singleton* Singleton::instance = nullptr;

int main() {
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();
    std::cout << (s1 == s2) << std::endl; // 输出1，表示s1和s2是同一个实例
    return 0;
}
```

</details>

### 1.6 复制构造函数

形如```X::X( X& )```或```X::X( const X& )```二者选一, 后者能以右值对象作为参数。不允许有形如```X::X( X )```的构造函数。

默认复制构造函数是```const X&```类型的。如果定义的自己的复制构造函数, 则默认的复制构造函数不存在。

调用的三种情况：

1. 用一个对象初始化同类的另一个对象：```MyClass c2(c2)```或者```MyClass c2=c1```

2. 传入```MyClass```类型的形参

3. 返回```MyClass```类型的值

若加上引用则不会调用。

### 1.7 类型转换构造函数

只有一个参数又不是复制构造函数的构造函数

```explicit Complex(int i)```显式类型转换构造函数

### 1.8 析构函数

一个类最多只有一个析构函数，析构函数对象消亡时自动被调用，缺省析构函数什么也不做，如果定义了析构函数, 则编译器不生成缺省析构函数。

```delete```运算导致析构函数调用。

<details>
<summary>+/-复制构造函数与析构函数例一</summary>

```cpp
class Demo {
    int id;
public:
    Demo(int i) {
        id = i;
        printf( "id=%d, Construct\n", id);
    }
    ~Demo() {
        printf( "id=%d, Destruct\n", id);
    }
};
Demo d1(1);
void fun(){
    static Demo d2(2);
    Demo d3(3);
    printf( "fun \n");
}
int main (){
    Demo d4(4);
    printf( "main \n");
    { Demo d5(5); }
    fun();
    printf( "endmain \n");
    return 0;
}
// 输出：
// id=1, Construct
// id=4, Construct
// main
// id=5, Construct
// id=5, Destruct
// id=2, Construct
// id=3, Construct
// fun
// id=3, Destruct
// endmain
// id=4, Destruct
// id=2, Destruct
// id=1, Destruct
```

</details>

<details>
<summary>+/-复制构造函数与析构函数例二</summary>

```cpp
#include <iostream>
using namespace std;
class CMyclass {
public:
    CMyclass() {};
    CMyclass( CMyclass & c )
    {
    cout << "copy constructor" << endl;
    } ~
    CMyclass() { cout << "destructor" << endl; }
};
void fun(CMyclass obj_ ) {
    cout << "fun" << endl; 
}
CMyclass c;
CMyclass Test( ) {
    cout << "test" << endl;
    return c;
}
int main(){
    CMyclass c1;
    fun(c1);
    Test();
    return 0;
}
// 输出
// copy constructor
// fun
// destructor //参数消亡
// test
// copy constructor
// destructor // 返回值临时对象消亡
// destructor // 局部变量消亡
// destructor // 全局变量消亡
```

</details>

<details>
<summary>+/-复制构造函数与析构函数例三</summary>

```cpp
#include<iostream>
using namespace std;

class CMyclass {
public:
	CMyclass() { cout << "constructor" << endl; }
	~CMyclass() { cout << "destructor" << endl; }
	CMyclass(const CMyclass& x)
	{
		cout << "copy" << endl;
	}
	CMyclass operator=(const CMyclass& x)
	{
		cout << "equal" << endl;
		return x;
	}
};

CMyclass obj;
CMyclass fun(CMyclass sobj) { 
	return sobj;
}

int main() {
	obj = fun(obj);
	return 0;
}
// 输出
// constructor
// copy
// copy
// destructor
// equal
// copy
// destructor
// destructor
// destructor
```

</details>

### 1.9 静态成员

在说明前面加了```static```关键字的成员（变量or函数）

```sizeof```运算符不会计算静态成员变量

静态成员函数并不具体作用于某个对象

访问静态成员：

1. 类名::成员名

2. 同访问普通成员的三种方法，但本质上并不具体作用于该对象

静态成员变量本质上是全局变量：哪怕一个对象都不存在, 类的静态成员变量也存在

静态成员函数本质上是全局函数

注意：

1. 必须在类定义的外部专门对静态成员变量进行声明，同行可进行初始化，若不初始化则全局变量自动初始化为全0

2. 局部类/结构体不能定义静态成员变量，因为静态成员变量本质上是全局变量，超出了该类/结构体的生存周期

3. 静态成员不能访问非静态成员，因为不知道是谁的非静态成员；同理静态成员函数真实参数中不含this指针

### 1.10 常量对象和常量方法

常量对象只能使用:构造函数, 析构函数和常量方法

常量方法不能使用：非静态属性，非常量方法且非静态成员函数

<details>
<summary>+/-强制类型转换</summary>

```cpp
void PrintfObj( const Sample & o ){
    o.nonConstFun(); //error
    o.ConstFun(); //ok
    Sample & r = (Sample &) o; //必须强制类型转换
    r.nonConstFun(); //ok
}
```

</details>

注意：

1. 两个函数, 名字和参数表都一样，但是一个是const, 一个不是, 算重载

2. ```mutable Type```类型的成员变量可以在常量方法中修改

### 1.11 成员对象和封闭类

成员对象: 一个类的成员变量是另一个类的对象

封闭类 (enclosing)：有成员对象的类

1. 封闭类对象生成时

先成员对象后封闭类，且成员对象的构造函数调用次序和成员对象在类中的说明次序一致，与它们在成员初始化列表中出现的次序无关

2. 封闭类的对象消亡时

先封闭类后成员对象，次序和构造函数的调用次序相反

### 1.12 const成员和引用成员

初始化```const```成员和引用成员, 必须在成员初始化列表中进行

### 1.13 友元

一个类的友元函数（可以是其他类的成员函数）可以访问该类的私有成员

友元类: 如果A是B的友元类, 那么A的成员函数可以访问B的私有成员

注意：友元类的关系不能传递，不能继承

## 2. 运算符重载

运算符```()``` ```[]``` ```->``` ```=```必须重载为成员函数（等括箭->等扩建）。

运算符```+``` ```*```等常重载为友元函数。

以下运算符不能被重载:```.``` ```::``` ```?:``` ```sizeof```。```&```只能重载为按位与。

运算符重载不改变运算符的优先级。

重载运算符是为了让它能作用于对象, 因此重载运算符不允许操作数都不是对象。

类型选择分析：

1. 返回值类型：一般不使用```const```修饰，除非为了安全性考虑不想让返回值成为左值；是否引用取决于返回的是当前对象本身还是临时对象（当然可以结合运算符原本的性质考虑）。
2. 传入参数类型：一般均为```const ClassName &```

### 2.1 +/-运算符重载

重载为成员函数则首个操作数必须为对象，故往往重载为**友元函数**以满足交换律。

返回类型为临时对象，与原运算符的性质契合。如果返回类型设定为引用，则会产生**悬垂引用**。

<details>
<summary>+/-重载的实现</summary>

```cpp
class Complex {
public:
    double real, imag;
    Complex( double r = 0.0, double i= 0.0 ):real(r), imag(i) { }
    Complex operator-(const Complex & c);
};
Complex operator+( const Complex & a, const Complex & b){
    return Complex( a.real+b.real, a.imag+b.imag);
    //返回一个临时对象
}
Complex Complex::operator-(const Complex & c){
    return Complex(real - c.real, imag - c.imag);
    //返回一个临时对象
}
```

</details>

### 2.2 =赋值运算符重载

只能重载为**成员函数**（因为赋值运算符的左侧必须是可赋值对象lvalue，不能是临时对象或者右值rvalue，这样操作可以确保这一点）。

返回值类型为```ClassName &```或者```const ClassName &```，用引用是为了支持链式操作，至于```const```修饰与否的差别主要在于是否支持```(a = b) = c```这样的操作，有时为了灵活性选择不修饰，有时为了安全性选择修饰。(本质上是加了```const```之后就不能是lvalue了)

<details>
<summary>右值为基本类型的实现</summary>

```cpp
class String {
private:
    char * str;
public:
    String () : str(new char[1]) { str[0] = 0; }
    const char * c_str() { return str; }
    String & operator = (const char * s);
    ~String() { delete [] str; }
};

String & String::operator = (const char * s){
    delete [] str;
    str = new char[strlen(s)+1];
    strcpy(str, s);
    return *this;
}

int main(){
    String s;
    s = "Good Luck," ;
    cout << s.c_str() << endl;
    s = "Shenzhou 13!";
    cout << s.c_str() << endl;
    return 0;
}
```
</details>

<br>

<details>
<summary>右值为类对象的实现</summary>

```cpp
String & operator = (const String & s) {
    if(this == &s) return * this;//避免自赋值时的问题
    delete [] str;
    str = new char[strlen(s.str)+1];
    strcpy(str, s.str);
    return * this;
}
```
</details>

<br>

<details>
<summary>复制构造函数的实现（避免shallow copy）</summary>

```cpp
String(const String & s) {
    str = new char[strlen(s.str)+1];
    strcpy(str, s.str);
}
```
</details>

### 2.3 流运算符重载

```istream```和```ostream```类本身可以完成对基本类型的输入输出，其内部会管理一个输入输出的缓冲区。此处重载是为了直接输入输出类的对象。

返回值类型为```istream &```和```ostream &```，不用```const```修饰是为了支持链式操作（否则后续无法修改缓冲区），使用引用是为了支持链式操作（连续地对同一个流对象进行操作），且每次返回会创建一个copy造成不必要的性能开销。

通常把流运算符重载为友元函数（因其首个操作数显然不为对象），但输出流也可以强行重载为成员函数，输入流不可（其第一个参数必须为```istream &```）


<details>
<summary>右值为基本类型的实现</summary>

```cpp
ostream & operator<< ( ostream & o,
    const CStudent & s){
    o << s.nAge;
    return o;
}
```
</details>

### 2.4 []运算符重载

大部分时候默认复制构造函数都会导致shallow copy，要记得写自己的复制构造函数。

<details>
<summary>[]重载的实现</summary>

```cpp
class Array{
public:
    Array(int n = 10) : size(n) { ptr = new int[n]; }
    ~Array() { delete [] ptr; }
    int & operator[](int subscript){
        return ptr[subscript];
    }
private:
    int size;
    int *ptr;
};
```
</details>

<br>

<details>
<summary>相应的赋值重载以及复制构造函数</summary>

```cpp
const Array & operator=( const Array & a)
{
    if( ptr == a.ptr ) return * this;
    delete [] ptr;
    ptr = new int[ a.size ];
    memcpy( ptr, a.ptr, sizeof(int ) * a.size);
    size = a.size;
    return * this;
} //返回const array & 类型是为了高效实现
//a = b = c; 形式

Array(Array & a) {
    ptr = new int[ a.size ];
    memcpy( ptr, a.ptr, sizeof(int) * a.size);
    size = a.size;
}
```
</details>

### 2.5 类型转换运算符重载

必须为成员函数, 不指定返回类型（因为运算符已经指明了类型）, 形参为空；一般不改变被转换对象（常定义为const）；触发类型转换时自动调用。

<details>
<summary>类型转换运算符重载的实现</summary>

```cpp
class Sample {
private :
    int n;
public:
    Sample(int i){
        n=i;
        cout<<"constructor called"<<endl;
    }
    Sample operator+(int k){
    Sample tmp(n + k);
        return tmp;
    }
    operator int ( ) { //重载类型强制转换运算符
        cout<<"int convertor called"<<endl;
        return n;
    }
};
```
</details>

### 2.6 ++/--运算符重载

无参为前置，有int参为后置。

前置的返回值类型选取```ClassName &```，因为返回的是当前对象的引用，从而支持链式操作并且提高效率；后置的返回值类型选取```ClassName```，因为返回的是一个临时对象。

成员函数中无需多言；友元函数中，传入的对象类型为```ClassName &```从而直接对当前对象进行操作。

至于为什么都不用```const```：返回值类型需要确保可以继续操作，故而不用```const```；参数类型需要对传入的引用进行修改，显然不能```const```。

<details>
<summary>++/--运算符重载的实现</summary>

```cpp
class CDemo {
private :
    int n;
public:
    CDemo(int i=0):n(i) { }
    CDemo & operator++( ); //用于前置形式
    CDemo operator++( int ); //用于后置形式
    operator int ( ) { return n; }
    friend CDemo & operator--( CDemo & );
    friend CDemo operator--( CDemo &, int);
};

// ++s即为: s.operator++();
CDemo & CDemo::operator++()
{ //前置 ++
    n ++;
    return * this;
}

// s++即为: s.operator++(0);
CDemo CDemo::operator++( int k )
{ //后置 ++
    CDemo tmp(*this); //记录修改前的对象
    n ++;
    return tmp; //返回修改前的对象
}

//--s即为: operator--(s);
CDemo & operator--(CDemo & d)
{ //前置--
    d.n--;
    return d;
}

//s--即为: operator--(s, 0);
CDemo operator--(CDemo & d, int)
{ //后置--
    CDemo tmp(d);
    d.n --;
    return tmp;
}
```
</details>


## 3. 继承和派生

### 3.1 概念

```class DerivedClass : public/protected/private BaseClass```
```public```时只有```private```不能访问

注意：

1. 继承是“是”关系

2. 复合（不同于enclosing）是“有”关系，且为了避免循环定义，需要提前声明类&&使用对象指针

<details>
<summary>+/-复合类</summary>

```cpp
class CDog;
class CMaster
{
    CDog * dogs[10];
};
class CDog
{
    CMaster * m;
};
```

</details>

### 3.2 内存

派生类对象的大小, 等于基类对象的大小 + 派生类对象自己的成员变量的大小

在派生类对象中, 包含着基类对象, 而且基类对象的存储位置位于派生类对象新增的成员变量之前

### 3.3 覆盖

覆盖：派生类中定义一个和基类成员同名的成员

访问时:

1. 缺省的情况访问派生类中定义的成员

2. ```BaseClass::```访问由基类定义的同名成员

### 3.4 protected访问权限

可以访问当前对象的基类保护成员：若覆盖则指明作用域

也可以访问其它**同类对象**的基类保护成员

派生类的友元函数也可以访问基类的保护成员（甚至不需传入类参数）

注意：不能访问基类对象的保护成员

### 3.5 三种继承方式

|         | public | protected | private |
|---------|--------|-----------|---------|
| 公有继承 | public | protected |    无   |
| 私有继承 | private | private  |    无   |
| 保护继承 | protected | protected | 无   |

### 3.6 构造函数和析构函数

1. 无成员对象时：

构造函数：先基类后派生类（内存区域从前到后？）

析构函数：先派生类后基类（内存区域从后到前？）

1. 有成员对象时：

构造函数：先基类再成员对象类最后派生类

析构函数：先派生类再成员对象类最后基类

### 3.7 public赋值兼容

```public```派生类的对象可以

1. 赋值给基类对象

2. 初始化基类引用

3. 地址可以赋值给基类指针

注意：如果派生方式是```private```或```protected```, 则上述三条不可行。本质上是因为属性的类别不同了。

<details>
<summary>强制类型转换</summary>

```cpp
Base * ptrBase = &objDerived;
Derived *ptrDerived = (Derived * ) ptrBase;
```
</details>

### 3.8 直接基类与间接基类

在声明派生类时, 派生类的首部只需要列出它的直接基类，不要列出它的间接基类。

<details>
<summary>反例</summary>

```cpp
class A {};

class B : public A {};

class C : public A, public B {};
// C会包含两份A
// 可以使用虚继承
```
</details>

<details>
<summary>虚继承</summary>

```cpp
class A {};

class B : virtual public A {};

class C : virtual public A, public B {};
```
</details>

| 特性 | 普通继承 | 虚继承 |
|------|----------|--------|
| 基类子对象数量 | 可能多份（菱形继承问题） | 仅一份 |
| 初始化责任 | 由直接派生类初始化 | 由最底层派生类初始化 |
| 适用场景 | 单继承或简单多重继承 | 菱形继承 |
| 性能开销 | 无额外开销 | 需维护虚基类表（略慢） |

### 3.9 多继承

构造函数：先基类（按继承顺序）再成员对象类最后派生类

析构函数：与构造反过来即可

## 4. 多态

### 4.1 概念

虚函数：```virtual int fun()```或者```int virtual fun()```

注意：

1. 只需要在定义时添加说明符即可，写函数体时不能添加

2. 静态成员函数不能是虚函数，因为静态成员函数没有```this```指针，且静态函数是类级别的

3. 多层继承时，从有```virtual```开始的派生类中同名函数均为虚函数

### 4.2 多态使用方法

1. 用基类指针：

- 若该指针指向一个基类的对象, 则被调用是基类的虚函数
- 若该指针指向一个派生类的对象, 则被调用的是派生类的虚函数

2. 用基类引用：

- 若该引用的是一个基类的对象, 那么被调用是基类的虚函数
- 若引用的是一个派生类的对象, 那么被调用的是派生类的虚函数

### 4.3 动态联编

动态联编：一条函数调用语句在编译时无法确定调用哪个函数，运行到该语句时才确定调用哪个函数

### 4.4 多态的实现原理

虚函数的调用版本是由对象的**动态类型**决定的，具体实现是通过**虚函数表**（Virtual Table，简称 vtable）和**虚函数指针**（Virtual Pointer，简称 vptr）来完成的。

1. 虚函数表（vtable）:

    每个包含虚函数的类在编译时都隐式生成一个 vtable 作为静态数据成员，其中存储了该类中所有虚函数的实际地址。基类和派生类的vtable是不同的。

2. 虚函数指针（vptr）:

    每个对象在创建时，都会被分配一个隐藏的 vptr，其由构造函数进行设置（故构造函数不能是虚函数），指向其所属类的 vtable。当调用虚函数时，程序通过对象的 vptr 找到其 vtable，然后通过 vtable 调用相应的虚函数。

<details>
<summary>+/-多态的实现原理</summary>

```cpp
class Base {
public:
    virtual void func1() { /* Base的实现 */ }
    virtual void func2() { /* Base的实现 */ }
    int data;
};

class Derived : public Base {
public:
    void func1() override { /* Derived的实现 */ }
    virtual void func3() { /* 新增虚函数 */ }
};
// Base对象:
// [vptr] -> Base的vtable [&Base::func1, &Base::func2]
// [data]

// Derived对象:
// [vptr] -> Derived的vtable [&Derived::func1, &Base::func2, &Derived::func3]
// [data]
```

</details>

### 4.5 虚函数的访问权限

访问权限检查是根据指针类型来的

<details>
<summary>+/-访问权限例一</summary>

```cpp
class Base {
private:
    virtual void fun2() { cout << "Base::fun2()" << endl; }
};
class Derived:public Base {
public:
    virtual void fun2() { cout << "Derived:fun2()" << endl; }
};
Derived d;
Base * pBase = & d;
pBase -> fun2(); // 编译出错
```

</details>

<details>
<summary>+/-访问权限例二</summary>

```cpp
class Base {
public:
    virtual void fun2() { cout << "Base::fun2()" << endl; }
};
class Derived:public Base {
private:
    virtual void fun2() { cout << "Derived:fun2()" << endl; }
};
Derived d;
Base * pBase = & d;
pBase -> fun2(); // 输出Derived::fun2()
```

</details>

### 4.6 虚析构函数

1. 构造函数：

    总是由实际创建的对象类型决定，而与声明类型无关；且构造函数不能 virtual

2. 析构函数：

    - 对于栈对象：总是调用完整析构链

    - 对于堆对象：

    virtual析构函数会实现多态析构

    非virtual析构函数会导致派生部分不被析构

注意：

1. ```Base* p = new Derived()```中，```p```存放在栈中，但是由```new```/```malloc```分配的指针指向堆内存，故此时析构不得不虚

2. 无论是构造函数还是析构函数，调用虚函数时都会在编译时确定，而不会动态联编。因为执行它们的百分百是当前类的对象（毕竟名字都不同）

### 4.7 纯虚函数和抽象类

纯虚函数：没有函数体的虚函数，如```virtual void Print( ) = 0 ;```

抽象类：包含纯虚函数的类叫抽象类

注意：

1. 抽象类只能作为基类来派生新类使用, 不能创建抽象类的对象

2. 抽象类的指针和引用可以指向由抽象类派生出来的类的对象

在抽象类的成员函数内可以调用纯虚函数，但是在构造函数或析构函数内部不能调用纯虚函数（因为构造与析构都会直接调用纯虚函数导致无定义）

如果一个类从抽象类派生而来, 那么当且仅当它实现了基类中的所有纯虚函数, 它才能成为非抽象类


## 5. 输入输出与文件操作

### 5.1 有关类

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Class Inheritance Tree</title>
    <style>
        .tree {
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: Arial, sans-serif;
        }
        .node {
            border: 1px solid #ccc;
            padding: 10px;
            border-radius: 5px;
            background-color:rgba(75, 81, 118, 0.85);
        }
        .node > .node {
            margin-top: 20px;
        }
        .line {
            border-left: 1px solid #ccc;
            border-top: 1px solid #ccc;
            width: 50px;
            height: 50px;
            position: relative;
            top: -25px;
            left: 25px;
        }
        .line::before {
            content: '';
            position: absolute;
            top: -25px;
            left: 0;
            border-left: 1px solid #ccc;
            border-top: 1px solid #ccc;
            width: 25px;
            height: 25px;
        }
        .line::after {
            content: '';
            position: absolute;
            top: 0;
            left: 25px;
            border-left: 1px solid #ccc;
            border-top: 1px solid #ccc;
            width: 25px;
            height: 25px;
        }
    </style>
</head>
<body>
    <div class="tree">
        <div class="node">ios_base
            <div class="node">ios
                <div class="node">istream
                    <div class="node">ifstream</div>
                    <div class="node">fstream</div>
                </div>
                <div class="node">ostream
                    <div class="node">ofstream</div>
                    <div class="node">fstream</div>
                </div>
            </div>
        </div>
    </div>
</body>
</html>

- istream：用于输入的流类, cin 就是该类的对象

- ostream：用于输出的流类, cout 就是该类的对象

- iostream：既能用于输入, 又能用于输出的类

- ifstream：用于从文件读取数据的类

- ofstream：用于向文件写入数据的类

- fstream：既能从文件读取数据, 又能向文件写入数据的类

### 5.2 标准流对象与重定向

重定向: 将输入的源或输出的目的地改变

1. 输入：

    输入流对象：```cin```

    重定向：```freopen("filename.txt", "r", stdin);```

2. 输入：

    输入流对象：```cout```, ```cerr```, ```clog```

    - cerr (无缓冲)

        数据会立即输出到设备, 如终端或日志文件, 不经过缓冲区；适用于紧急错误、调试信息 (确保即使程序崩溃也能输出)

    - clog (带缓冲)

        数据先存入缓冲区, 缓冲区满或手动刷新时才输出; 适用于非紧急日志记录, 性能更高 (减少频繁I/O操作), 常规日志记录 (如程序运行状态、非关键警告, 允许延迟输出)

        可用```flush```手动冲刷


    重定向：```freopen("filename.txt", "w", stdout);```

    注意：这只是重定向了```std::out```，并没有改变另外两个输入流对象。

### 5.3 判断输入流结束

重载```operator bool()```强制类型转换

### 5.4 istream类

标志状态：ios_base类的静态成员常量

- ```std::ios_base::goodbit```：表示输入流状态良好，没有发生任何错误
- ```std::ios_base::eofbit```：表示输入流已经到达文件末尾（End Of File，EOF）
- ```std::ios_base::failbit```：表示输入流发生了格式错误或读取失败
- ```std::ios_base::badbit```：表示输入流发生了严重错误，例如无法读取文件

可以通过```object.eof()```, ```object.fail()```, ```object.bad()```分别进行访问

```istream & getline(char * buf, int bufSize);```
从输入流中读取(bufSize-1)个字符到缓冲区, 或读到```\n```为止(哪个先到算哪个)

```istream & getline(char * buf, int bufSize, char delim);```
从输入流中读取(bufSize-1)个字符到缓冲区, 或读到```delim```为止

注意：

1. 两个函数都会自动在缓冲区中读入数据的结尾添加 ```\0```

2. ```\n```或```delim```都不会被读入缓冲区, 但会被从输入流中取走

3. 如果输入流```\n```或```delim```之前的字符个数超过了```bufSize```个, 就导致读入出错（标志状态设为```std::ios_base::failbit```）, 其结果就是: 本次读入已经完成, 但之后的读入就都会失败(可以用```cin.clear()```清除状态标志)。

更多成员函数：

```bool eof();```：判断输入流是否结束

```int peek();```：返回下一个字符, 但不从流中去掉

```istream & putback( char c );```：将字符```c```放回输入流

```istream & ignore(int nCount = 1, int delim = EOF );```：从流中删掉最多```nCount```个字符, 遇到```delim```时结束

### 5.5 流操纵算子

```#include<iomanip>```头文件

注：```endl```除了换行还可以冲刷缓冲区，```\n```不行

1. 整数流的基数: 

    ```dec```, ```oct```, ```hex```, ```setbase```(只能2, 8, 10, 16)

2. 浮点数的精度:

    成员函数：```cout.pricision(num)```

    流操纵算子：```cout<<pricision(num)```

    功能：

    - 指定输出浮点数的有效位数（非定点方式输出时）

    - 指定输出浮点数的小数点后的有效位数（定点方式输出时```std::fixed```，确保不使用科学计数法```scientific```）

    也可以```setiosflags(ios::fixed)```定点
    ```seiosflags(ios::scientific)```科学计数法
    ```resetiosflags(...)```取消之前的设置

    注意：

    1. 输出最多```num```位有效数字（可以不到）

    2. 会四舍五入


3. 设置域宽:

    域宽：输出时所占的最小字符数，默认为1

    成员函数：```cin.width(num)```无参输出当前域宽

    流操纵算子：```setw(num)```

    注意：

    1. 输入操作提取字符串的最大宽度比定义的域宽小1, 因为在输入的字符串后面必须加上一个空字符

    2. 每次读入和输出之前都要设置宽度

4. 用户自定义的流操纵算子

    有对```<<```的重载

    ```cpp
    ostream& operator<<(ostream& (*p)(ostream&))
    {
        return p(*this);
    }
    ```

    故直接

    ```cpp
    ostream& tab(ostream& out)
    {
        return out<<"\t";
    }
    ```

    即可自定义

### 5.6 建立顺序文件

顺序文件：将所有记录顺序地写入一个文件

1. 建立：

    ```cpp
    #include <fstream> // 包含头文件

    ofstream fileObject("filename.txt", ios::out|ios::binary); //打开文件

    //另一种方法
    ofstream fout;
    fout.open("filename.txt", ios::out|ios::binary);

    // 判断打开是否成功：
    if (!fout) { cerr << "File open error!" <<endl; }
    ```

    其中
    - ```ios::out``` 输出到文件, 删除原有内容
    - ```ios::app``` 输出到文件, 保留原有内容, 总是在尾部添加(append)
    - ```ios::ate``` 输出到文件, 保留原有内容, 在文件任意位置
    添加(at end)
    - ```ios::binary``` 以二进制文件格式打开文件
2. 读写

    对于输入文件, 有一个读指针

    对于输出文件, 有一个写指针

    对于输入输出文件, 有一个读写指针

    读指针（get pointer）：

    ```fin.tellg```：取得读指针的位置

    ```fin.seekg(location)```：将读指针移动到第```location```个字节处（可以为负）

    ```fin.seekg(location, ios::beg);```：从头数```location```

    ```fin.seekg(location, ios::cur);```：从当前数```location```

    ```fin.seekg(location, ios::end);```：从尾数```location```

    读指针（put pointer）：

    ```fout.tellp```：取得读指针的位置

    ```fout.seekp(location)```：将读指针移动到第```location```个字节处（可以为负）

    ```fout.seekp(location, ios::beg);```：从头数```location```

    ```fout.seekp(location, ios::cur);```：从当前数```location```

    ```fout.seekp(location, ios::end);```：从尾数```location```

    读操作：

    ```fin.read((char *)(&x), sizeof(Type));```

    ```fin.gcount();```看上一次读入几个字节

    ```fin.get(char c)```每次读取一个字符

    写操作：

    ```fout.write((const char *)(&x), sizeof(Type));```

    ```fout.put(char c)```每次写入一个字符

    注意：读无```const```因为会修改指针指向区域，写有```const```因为不能修改原有数据

3. 关闭

    需要显示关闭文件

    ```fin.close()```和```fout.close()```

### 5.7 二进制文件和文本文件

1. Linux / Unix下的换行符号：

    ```\n```(ASCII码: ```0x0a```)，故Unix/Linux下打开文件, 用不用```ios::binary```没区别

2. Windows下的换行符号：

    文本文件：```\r\n```(ASCII码：```0x0d 0x0a```)
    
    二进制文件：```\n```

    - 若不使用```ios::binary```，则读取时, 系统会将```0x0d 0x0a```只读入```0x0a```；写入时, 对于```0x0a```系统会自动补上```0x0d```

    - 若使用```ios::binary```，则读取和写入都一板一眼

3. Mac OS下的换行符号：```\r```(ASCII码：```0x0d```)

故Linux, Mac OS 文本文件在Windows 记事本中打开时不换行


## 6. 模板

### 6.1 函数模板

由编译系统根据调用时实参的类型, 自动生成相应的模板函数

也可以在```<>```中指定类型参数所对应的具体类型来实例化

函数模板的参数类型：可以用类型参数说明；也可以用基本数据类型, 其他的类说明

同名函数模板也可以重载，参数（类型参数，形参）数量不同即可

注意：分开定义时，后面也要写```template<>```

**调用顺序**：
先找参数完全匹配的函数，再找参数完全匹配的模板，再找参数自动转换后能匹配的函数（要求没有二义性）。总之，越精确越好。

### 6.2 类模板

主模板声明时

```cpp
template <类型参数表>
class 类模板名;
```

在类模板外定义时（注意```T```只是一个记号）

```cpp
template <类型参数表>
返回值类型 类模板名<类型参数名列表>::成员函数名(参数表){
    ...
}
```

同⼀个类模板的两个模板类是不兼容的

类模板中的成员函数可以是⼀个函数模板，该成员函数只有在被调⽤时才会被实例化

可以包括非类型参数（函数模板也可以），用以说明类模板中的属性

### 6.3 继承与派生

类模板可以派生类模板，模板类可以派生类模板，普通类可以派生类模板，模板类可以派生普通类

注：本质上模板类和普通类没有区别，都失去了随机性（塌缩了）。可以认为只有四种关系，四种里面只有随机派生出确定不行（类模板派生模板类/普通类不行，因为构造时必须传入类型参数，传入不了）

### 6.4 类模板与友元函数

简单来说就是都可以

```cpp
#include <iostream>
using namespace std;

template<class T>
class FriendTemplate;

// 定义一个普通类
class MyClass {
private:
    int secret;

public:
    MyClass(int s) : secret(s) {}

    // 将类模板 FriendTemplate 声明为友元
    template<class T>
    friend class FriendTemplate;
};

// 定义一个类模板
template <typename T>
class FriendTemplate {
public:
    void display(const MyClass& x) {
        cout << "Displaying: " << x.secret << endl;
    }
};

int main() {
    MyClass obj(42);

    // 创建类模板的实例
    FriendTemplate<int> ft;
    ft.display(obj); // 友元类可以访问私有成员

    return 0;
}
```

### 6.5 特化

使用```template<>```可以进行显示特化，

<details>
<summary>+/-显示特化类模板</summary>

```cpp
#include <iostream>
using namespace std;

// 定义一个类模板
template <typename T>
class MyTemplate {
public:
    void display() {
        cout << "Generic template for type " << typeid(T).name() << endl;
    }
};

// 显式特化类模板
template<>
class MyTemplate<int> {
public:
    void display() {
        cout << "Explicit specialization for int" << endl;
    }
};

int main() {
    MyTemplate<double> obj1;
    obj1.display(); // 使用通用模板

    MyTemplate<int> obj2;
    obj2.display(); // 使用显式特化

    return 0;
}
```

</details>

<details>
<summary>+/-显式特化函数模板</summary>

```cpp
#include <iostream>
using namespace std;

// 定义一个函数模板
template <typename T>
void print(T value) {
    cout << "Generic template: " << value << endl;
}

// 显式特化函数模板
template<>
void print<int>(int value) {
    cout << "Explicit specialization for int: " << value << endl;
}

int main() {
    print(3.14); // 使用通用模板
    print(42);   // 使用显式特化
    return 0;
}
```

</details>


<details>
<summary>+/-偏特化</summary>

```cpp
#include <iostream>
using namespace std;

// 定义一个类模板
template <typename T1, typename T2>
class MyTemplate {
public:
    void display() {
        cout << "Generic template for types " << typeid(T1).name() << " and " << typeid(T2).name() << endl;
    }
};

// 偏特化，为第一个参数是 int 的情况提供特化
template <typename T2>
class MyTemplate<int, T2> {
public:
    void display() {
        cout << "Partial specialization for int and " << typeid(T2).name() << endl;
    }
};

int main() {
    MyTemplate<double, float> obj1;
    obj1.display(); // 使用通用模板

    MyTemplate<int, float> obj2;
    obj2.display(); // 使用偏特化

    return 0;
}
```

</details>


## 7.
