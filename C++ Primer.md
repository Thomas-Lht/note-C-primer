# 第六章 函数

## 6.1

函数：返回类型,函数名,形参列表,函数体
实参、形参
局部静态变量
例:

```c++
size_t count_calls() {
    static size_t count = 0;
    return ++count;
}
```

count在程序的执行路径第一次经过对象定义语句是初始化，直到程序结束才会销毁。
在此期间即使对象所在的函数结束执行也不会对它产生影响。

## 6.2

当初始化一个非引用类型变量时，初始值被拷贝给变量。函数的形参做的所有操作都不会影响实参

**使用引用避免使用拷贝**

当函数无需修改引用形参的值时最好使用常量引用
initializer_list形参:函数实参数量位置但参数类型相同

```c++
void error_msg(initializer_list<string> il)
{
    for (auto beg = il.begin(); beg!= il.end(); ++beg)
        cout << *beg << " ";
    cout << endl;  
}
```

## 6.3

**不要返回局部对象的引用或指针**
返回数组指针
*p == a[]
**p == *a[]
(*p)[] == a[][]
如果知道函数返回的类型，可以使用decltype关键字

```c++
int odd[] = {1, 3, 5, 7, 9}
int even[] = {2, 4, 6, 8, 10}
decltype(odd) *arrPtr(int i)
{
    return (i % 2 == 0)? &even : &odd;
} 
```

## 6.4

**函数重载**:同一作用域内函数名相同但形参列表不同
例

```c++
void print(const char* cp);
void print(const int *beg,const int *end);
void print(const int ia[],size_t size);
```

不允许形参列表一样但返回类型不同
顶层const不影响传入函数度对象。一个拥有顶层const的形参无法和另一个没有顶层const的形参区分。

```c++
Record lookup(Phone);
Record lookup(const Phone); //重复声明
```

如果在内层作用域中声明名字，则内层作用域的声明会屏蔽外层作用域的声明。

## 6.5

默认实参

```c++
string screen(sz ht = 24,sz wid = 80,char backgrnd = ' ')
string window;
window = screen(); //调用默认参数
window = screen(66);//等价于screen(66,80,' ')
window = screen(66,256); //等价于screen(66,256,' ')
window = screen(66,256,'#');//等价于screen(66,256,'#') 
```

incline内联函数

```c++
incline const string &shortString(const string &s1, const string &s2)
{
    return s1.size() < s2.size()? s1 : s2;
}
```
内联机制用于优化规模较小、流程直接、频繁调用的函数。

constexpr函数:在程序编译时求值函数

```c++
constexpr int square(int x) {
    return x * x;
}

int main() {
    const int result = square(5); // 编译时计算结果
    std::cout << result << std::endl;

    return 0;
}
```

## 6.6

候选函数:重载函数集
可行函数:从候选函数中选出实参调用函数
寻找最佳匹配
精确匹配：

1.实参类型和形参类型相同

2.实参从数组类型或函数类型专户只能成对应的指针类型

3.添加或删除顶层const

## 6.7
**函数指针**
例

```c++
bool (*pf)(const string&,const string &)
```
函数指针使用

```c++
pf = length Compare;
pf = &lengthCompare;
bool b1 = pf("hello","goodbye");
bool b2 = (*pf)("hello","goodbye");
bool b3 = lengthCompare("hello","goodbye");
```

重载函数指针

```c++
void ff(int*);
void ff(unsigned int);
void (*pf)(unsigned int) == ff;
```

# 第七章 类

## 7.1 定义抽象数据类型
类的基本思想时**数据抽象**和**封装**。数据抽象时一种依赖接口和实现分离的编程技术。
成员都必须在类的内部声明，但成员函数可以定义在类内也可以定义在类外。
在成员函数内部，可以直接使用函数的对象的成员变量。
定义成员函数

```c++
std::string isbn() const { resurn bookNo; }
```
**引入this指针**

```c++
total.isbn()
```
成员函数通过一个名为**this**的额外的隐式参数来访问调用他的那个对象。调用成员函数是，用请求该函数的对象地址初始化为this。
等价于
```c++
Sales_data::isbn(&total)
```
当isbn使用bookNo时，它隐式地使用this指向的成员，就像我们书写了this->bookNo。
我们也可以把isbn定义为如下形式：
```c++
std::string isbn() const { return this->bookNo; }
```
**const成员函数**
C++语言允许把const关键字放在成员函数的参数列表之后，此时，紧跟在参数列表后面的const表示this是一个指向常量的指针。像这样使用const的成员函数被称作常量成员函数
```c++
std::string Sales_data::isbn(const Sales_data *const this)
{return this->isbn;}
``` 
因为this是常量的指针所以常量成员函数不能改变调用他的对象内容。
### 构造函数
我们没有为对象提供初始值因为我们知道它们执行了默认初始化。类通过一个特殊构造函数来控制默认初始化进程，这个函数叫做**默认构造函数**。
```c++
Sales_data total;
Sales_date trans;
```
在C++11中新标准中，如果我们需要默认的行为，那么可以通过在参数列表后面写上 **= default**要求编译器生成构造函数。
```c++
Sales_data() = default;
Sales_data(const std::string& s) : bookNo(s) { }
Sales_data(const std::string& s, unsigned n, double p) :
	bookNo(s), units_sold(n), revenue(p* n) { }
Sales_data(std::istream&);
```
第二三个构造函数冒号以及冒号和花括号之间的代码，其中花括号定义了空的函数体。我们把新出现的部分称为**构造函数初始列表**，负责为新创建的对象一个或几个数据成员赋初值。
### 拷贝、赋值和析构
## 7.2 访问控制与封装
C++中使用**访问说明符**加强类的封装性

- 定义在**public**说明符之后的成员在整个程序内可被访问
- 定义在**private**说明符之后的成员在类内部的成员函数访问，但不能被使用该类的代码访问


```c++
class Sales_data {
	friend Sales_data add(const Sales_data&, const Sales_data&);
	friend std::ostream& print(std::ostream&, const Sales_data&);
	friend std::istream& read(std::istream&, Sales_data&);
public:
	// constructors
	Sales_data() = default;
	Sales_data(const std::string& s) : bookNo(s) { }
	Sales_data(const std::string& s, unsigned n, double p) :
		bookNo(s), units_sold(n), revenue(p* n) { }
	Sales_data(std::istream&);

	// operations on Sales_data objects
	std::string isbn() const { return bookNo; }
	Sales_data& combine(const Sales_data&);
	double avg_price() const;
private:
	std::string bookNo;
	unsigned units_sold = 0;
	double revenue = 0.0;
};
```
**使用class或struct关键字**
出于统一编程风格的考虑，当我们希望定义的类的所有成员是public时，我们可以用struct关键字；反之如果希望成员是private，使用class。
### 友元
类允许其它类或者函数访问它的非公有成员，方法是令其他类或者函数成为它的**友元**。
**友元声明**
为了使友元对类的用户可见，我们通常把友元的声明与类本身放置在同一个头文件中。
## 7.3 类的其他特性
```c++
class Screen {
public:
    typedef std::string::size_type pos;
	Screen() = default;  // needed because Screen has another constructor
	// cursor initialized to 0 by its in-class initializer
    Screen(pos ht, pos wd, char c): height(ht), width(wd), 
	                                contents(ht * wd, c) { }
    Screen(pos ht = 0, pos wd = 0): 
       cursor(0), height(ht), width(wd), contents(ht * wd, ' ') { }
    char get() const              // get the character at the cursor
	    { return contents[cursor]; }       // implicitly inline
    inline char get(pos ht, pos wd) const; // explicitly inline
	Screen &clear(char = bkground);
private:
     // function to do the work of displaying a Screen
     void do_display(std::ostream &os) const {os << contents;}
    pos cursor = 0;
    pos height = 0, width = 0;
    std::string contents;
};
```
因为我们已经提供了一个构造函数，所以编译器将不会自动生成默认的构造函数。如果我们的类需要默认构造函数，必须显式地声明成员函数。
**令成员作为内联函数**
我们可以在类内部把incline作为声明的一部分显式地声明成员函数
```c++
inline                   // we can specify inline on the definition
Screen &Screen::move(pos r, pos c)
{
    pos row = r * width; // compute the row location
    cursor = row + c;    // move cursor to the column within that row
    return *this;        // return this object as an lvalue
}

char Screen::get(pos r, pos c) const // declared as inline in the class
{
    pos row = r * width;      // compute row location
    return contents[row + c]; // return character at the given column
}
```
**重载成员函数**
和非成员函数一样，成员函数也可以被重载，只要函数之间在参数的数和/或类型上有区别就行。

**可变数据成员**
一个**可变数据成员**永远不是const，即使他是const对象成员。
```c++
class Screen {
public:
    void some_member() const
private:
    mutable size_t access_ctr;
};
void Screen::some_member() const {
    ++access_ctr; 
}
```
### 返回*this的成员函数
```c++
class Screen {
public:
    Screen &set(char);
    Screen &set(pos, pos, char);
};
inline Screen &Screen::set(char c) 
{ 
    contents[cursor] = c; // set the new value at the current cursor location
    return *this;         // return this object as an lvalue
}
inline Screen &Screen::set(pos r, pos col, char ch)
{
	contents[r*width + col] = ch;  // set specified location to given value
	return *this;                  // return this object as an lvalue
}
```
从const成员函数返回*this

基于const的重载
```c++
class Screen {
public:
    Screen &display(std::ostream &os) 
                  { do_display(os); return *this; }
    const Screen &display(std::ostream &os) const
                  { do_display(os); return *this; }

private:
    void do_display(std::ostream &os) const {os << contents;}
};
```
当我们在某个对象上调用display时，给对象是否是const决定了应该调用display的哪个版本。
```c++
Screen myScreen(5,3);
const Screen blank(5,3);
myScreen.set('#').display(cout); // calls the non-const version
blank.display(cout);            // calls the const version
```
### 类类型
每个类定义了唯一的类型。对于两个类来说，即使他们的成员完全一样，这两个类也是两个不同的类型。
```c++
struct First{
    int memi;
    int getMen();
};
struct Second{
    int memi;
    int getMen();
};
```
我们可以把类名作为类型的名字使用，从而直接指向类类型。
```c++
Sales_data item1;
class Sales_data item; // equivalent to the previous line
```
### 友元再探
如果一个类指定了友元类，则友元类的成员函数可以访问此类包括非公有成员在内的所有成员。
**令成员函数作为友元**
除了令整个class作为友元之外，我们还可以只令某个成员函数作为友元。
```c++
class Screen{
    friend void Window_mgr::clear(ScreenIndex);
}
```
定义方式：
- 首先定义Window_mgr类，其中声明clear函数，但不能定义它。在clear使用Screen的成员之前必须先声明Screen。
- 接下来定义Screen类，包括对clear的友元声明。
- 最后定义clear，此时它才可以使用Screen的成员

## 7.4 类的作用域
### 名字查找与类的作用域
编译器处理完类中的全部声明后才会处理成员函数的定义
用于类成员声明的名字查找
成员定义中的普通块作用域的名字查找
类作用域之后，在外围的作用域中查找（*尽管外层的对象被隐藏掉了，但我们仍然可以用作用域符访问它*）
在文件名字的出现处对其进行解析

## 7.5 构造函数再探

### 构造函数的初始值列表
有时我们可以忽略数据成员初始化和赋值之间的差异，但并非总能这样。如果成员是const或者引用的话，必须将其初始化。

```c++
class  ConstRef {
public:
    CondtRef(int ii);
private:
    int i;
    const int ci;
    int &ri;
}
```
和其他常量对象或者引用一样，成员ci和ri都必须被初始化。
如果没有为它们提供初始值的话将引发错误

```c++
ConstRef::ConstRef(int ii)
{
    i = ii;
    ci = 0; // error: ci not initialized
    ri = i; // error: ri not initialized
}
```
构造函数的正确形式应该是:
```c++
ConstRef::ConstRef(int ii) : i(ii), ci(ii), ri(i) { }
```
### 构造委托函数
一个委托构造函数使用它所属类的其他构造函数执行它自己的初始化过程，或者说它把它自己的一些职责委托给了其他构造函数。

```c++
class Sales_data {
public:
    //非委托构造函数是与哦那个对应的实参初始化成员
    Sales_data(std::string s, unsigned cnt, double price):
        bookNo(s), units_sold(cnt), revenue(price * cnt) { }
    //其余构造函数全都委托给另一个构造函数
    Sales_data(): Sales_data("", 0, 0){}
    Sales_data(std::string s): Sales_data(s, 0, 0) {}
    Sales_data(std::istream &is): Sales_data() 
        { read(is, *this); }
}
```

### 默认构造函数的作用
```c++
class NoDefault {
public:
    NoDefault(const std::string&);
};
struct A {
    NoDefault my_men;
};
A a;            //错误：不能为A合成构造函数
struct B {
    B() {}      //错误：b_member没有初始值
    NoDefault b_member;
}
```
**使用默认构造函数**
```c++
Sale_data obj();                         // 正确：定义了一个函数而非对象
if (obj.isbn() == Primer_5th_ed.isbn())  // 错误：obj是一个函数
```
如果想使用默认构造函数进行初始化对象，正确的方法是去掉对象名之后的空的括号对：
```c++
Sale_data obj;                           // 正确：obj是个默认初始化的对象
```
### 隐式类型转换
如果构造函数只接受一个实参，则他实际上定义了转换为此类型转换类型的隐式转换机制，有时我们把这种转换称作**转换构造函数**。
在Sales_data类中，接受string的构造函数和接受istream的构造函数分别定义了从这两种类型向Sales_data隐式转换的规则。
```c++
string null_book = "9-999-99999-9";
//构造一个临时的Sales_data对象
//该对象的units_sold和revenue等于0，bookNo等于null_book
item.combine(null_book);
```
**只允许一步类型转换**
因为下面的代码隐式地使用了两种转换规则，所以它是错误的：
```c++
//错误：需要用户定义的两种类型转化
//(1)把“9-999-99999-9”转换为string
//(2)再把这个(临时的)string转换为Sales_data
item.combine("9-999-99999-9");
//正确：显式地转换string，隐式地转换Sales_data
item.combine(string("9-999-99999-9"));
//正确：隐式地转换string，显式地转换Sales_data
item.combine(Sales_data("9-999-99999-9"));
```
**抑制构造函数的隐式转换**
在要求隐式转换的程序上下文时，我们可以通过将构造函数声明为**explicit**加以阻止：
```c++
class Sales_data {
public:
    Sales_data(const std::string& s, unsigned n, double p) :
        bookNo(s), units_sold(n), revenue(p* n) { }
    explicit Sales_data(const std::string &s):bookNo(s) { }
    explicit Sales_data(std::istream &);
};
```
**explicit构造函数只能用于直接初始化**
发生隐式转换的一种情况是当我们执行拷贝形式的初始化时(使用=)。 此时，我们只能使用直接初始化而不能使用**explicit**构造函数:
```c++
Sales_data item1(null_book);      //正确：直接初始化
Sales_data item2 = null_book;     //错误：不能将explicit构造函数用于拷贝形式的初始化过程
```
### 聚合类
**聚合类**使用户可以直接访问其成员，并具有特殊的初始化语法形式。当一个类满足如下条件是时，我们说他是聚合的：
- 所有成员都是public的
- 没有定义任何构造函数。
- 没有类内的初始值
- 没有基类，也没有virtual的函数
```c++
struct Data{
    int ival;
    string s;
}
```
我们可以提过那个一个花括号的成员初始值列表，并与哦那个它初始化聚合类的数据成员：
```c++
Data val1 = { 0, "Anna"};
```
初始值的顺序必须与声明一致
值得注意的是，显式地初始化的类的对象存在三个明显的缺点：
- 要求类的所有成员都是public的
- 将正确初始化每个对象的每个成员的重任交给了类的用户。因为用户很容易忘记某个初始值，或者提供一个不恰当的初始值，所以这样的初始化过程冗长乏味且容易出错。
- 添加或删除一个成员之后，所有的初始化语句都需要更新。
### 字面值常量类
数据成员都是字面值类型的聚合类是字面值常量类。如果一个类不是聚会类，但它符合下述要求，则它也是字面值常量类：
- 数据成员都必须是字面类型
- 类内必须至少有一个 constexpr构造函数
- 如果一个数据成员含有类内初始，则内置类型成员必须是一条常量表达式或者如果成员属于某种类类型的，则初始值必须使用成员自己的constexpr构造函数
- 类必需使用析构函数的默认定义，该成员负责销毁类的对象
```c++
class Debug {
public:
    constexpr Debug(bool b = true): hw(b), io(b), other(b) {}
    constexpr Debug(bool h, bool i, bool o):
        hw(h), io(i), other(o) {}
    constexpr bool any() {return hw || io || other;}
    void set_io(bool b) {io = b;}
    void set_hw(bool b) {hw = b;}
    void set_other(bool b) {other = b;}
private:
    bool hw;
    bool io;
    bool other;
}
```
## 7.6 类的静态成员
**声明静态成员**
我们通过在成员声明之前加上关键字static使得其与类关联在一起。
```c++
class Account {
public:
    void calculate() { amount += amount * interestRate; }
    static double rate() { return interestRate; }
    static void rate(double);
private:
    std::string owner;
    double amount;
    static double interestRate;
    static double initRate();
};
```
类的静态成员存在于任何对象之外，对象中不包含任何与静态数据成员有关的数据。
静态成员函数也不与任何对象绑定在一起，他们不包含this指针。
**使用类的静态成员**
使用作用域符直接访问静态成员：
```c++
double r;
r = Account::rate();
```
虽然静态类成员不属于类的某个对象，但我们仍然可以使用类的对象、引用或指针来访问静态成员：
```c++
Account ac1;
Account *ac2 = &ac1;
r = ac1.rate();
r = ac2->rate();
```
**定义静态成员**
==和类的所有成员一样，当我们指向类外部的静态成员时，必须指明长远所属的类名。static关键字则只出现在类内部的声明语句中。==
想要确保对象只定义一次，最好的办法是把静态数据成员的定义与其他非内敛函数的定义放在一个文件中。
**静态类成员的类内初始化**  
即使一个常量静态成员在类内部被初始化了，通常情况下也应该在类的外部定义一下该成员。
# 第8章 IO库
IO库设施：
- istream类型，提供输入操作
- ostream类型，提供输出操作
- cin，一个istream对象，从标准输入读取数据
- cout，一个ostream对象，向标准输出写入数据
- cerr，一个ostream对象，通常用于输出程序错误信息，写入到标准错误
- \>\>运算符，用来从一个istream对象读取输入数据
- <<运算符，用来向一个ostream对象写入输出数据
- getline函数，从一个给定的istream对象中读取一行数据，存入一个给定的string对象中
## 8.1 IO类
| 头文件 | 类型 |
| --- | --- |
|iosteam| istream,wistream从流读取数据<br>ostream,wostream向流写入数据<br>iostream,wiostream读写流 |
|fstream|ifstream,ofstream从文件读取数据<br>fstream,wfstream向文件写入数据<br>fsream,wfsream读写文件 |
|sstream|istringstream,ostringstream从string读取数据<br>stringstream,wstringstream向string写入数据<br>strstream,wstrstream读写string |
### IO对象无拷贝或赋值
```c++

```






# 第9章 顺序容器
## 9.1顺序容器概述
|顺序容器类型||
|---|---|
|vector|可变大小数组。支持快速随机访问。在尾部之外的位置插入或删除元素可能很慢|
|deque|双端队列。支持快速随机访问。在头尾位置插入/删除速度很快|
|list|双向链表。只支持双向顺序访问。在list中任何位置进行插入/删除操作都很块|
|forward_list|单向链表。只支持单向顺序访问。在链表任何位置进行插入/删除操作速度都很快|
|array|固定大小数组。支持快速随机访问。不能添加或删除元素|
|string|与vector相似容器，但专门用于保存字符，随机访问快。在尾部插入/删除速度快|

**确定使用哪种顺序容器**
- ==除非你有更好的理由选择其他容器，否则应使用vector==
- 如果你的程序有很多小的元素，且空间的额外开销很重要，则不要使用list或forward_list
- 如果程序要求随机访问元素，应用vector或deque
- 如果程序要求在容器的中间插入或删除元素，应使用list或deque
- 如果程序要在头尾插入或删除元素，但不会在中间位置进行插入或删除操作，则使用deque
- 如果程序只有在读取输入时才需亚欧在容器中间位置插入元素，随后需要随机访问元素，则：
> - 首先，确定是否真的需要在容器位置添加元素。当处理输入数据时，通常可以很容易地向vector追加数据，然后在调用标准库sort函数来重排容器中的元素，从而避免在中间位置添加元素
> - 如果必须在中间位置插入元素，考虑在输入阶段使用list，一旦输入完成，将list中的内容拷贝到一个vector中。

## 9.2容器库预览
|容器操作||
|---|---|
|**类型别名**||
|iterator|此容器类型的迭代器类型|
|const_iterator|可以读取元素，但不能修改元素的迭代器类型|
|size_type|五符号 整数类型，足够保存此种容器类型最大可能容器的大小|
|difference_type|带符号整数类型，足够保存两个迭代器之间的距离|
|value_type|元素类型|
|reference|元素的左值类型；与value_type&含义相同|
|const_reference|元素的const左值类型|
|**构造函数**||
|C c;|默认构造函数，构造空容器|
|C c1(c2);|构造c2拷贝c1|
|C c(b, e);|构造c，将将迭代器b与e指定的范围内的元素拷贝到c|
|C c{a, b, c...};|列表初始化c|
|**赋值与swap**||
|c1 = c2|将c1中的元素替换为c2中的元素|
|c1 = {a, b, c...}|将c1中的元素替换为列表中的元素|
|a.swap(b)|交换a和b的元素|
|swap(a,b)|与a.swap(b)等价|
|大小||
|c.size()|c中元素数目|
|c.max_size()|c中可保存的最大元素数目|
|c.empty()|若c中存储了元素,则返回false,否则返回true|
|**添加/删除元素**||
|c.insert(args)|将args中的元素拷贝进c|
|c.empty(inits)|使用inits构造c中的一个元素|
|c.erase(args)|删除args指定的元素|
|c.clear()|删除c中所有元素，返回void|
|**关系运算符**||
|==,!=|所有容器都支持相等运算符|
|<, <=, >, >=|关系运算符|
|**获取迭代器**||
|c.begin(),c.end()|返回指向c的首元素和尾元素之后位置的迭代器|
|c.cbegin(),c.cend()|返回const_iterator|
|**反向迭代器的额外成员**||
|reverse_iterator|按逆序寻址元素的迭代器|
|const_reverse_iterator|不能修改元素的顺序迭代器|
|c.rbegin(),c.rend()|返回指向c的尾元素和首元素之前位置的迭代器|
|c.crbegin(),c.crend()|返回const_reverse_iterator|
### 迭代器
==迭代器范围的概念是标准库的基础==
一个**迭代器范围**由一对迭代器表示，两个迭代器分别指向同一个容器中的元素或者尾元素之后的位置。
这种元素范围被称为**左闭合区间**
- 如果begin与end相等，则范围为空
- 如果begin与end不等，则范围至少包含一个元素，且begin指向该范围中的第一个元素
- 我们可以对begin递增若干次，使得begin==end
```c++
while(begin != end)
{
    *begin = val;       //正确:范围非空，因此begin指向一个元素
    ++begin;            //移动迭代器，获取下一个元素
}
```
### 容器类型成员
类型别名
```c++
// iter是通过list<string>定义的一个迭代器类型
list<string>::iterator iter;
//count是通过vector<int>定义的一份difference_type类型
vector<int>::difference_type coount;
```
### begin和end成员
begin和end操作生成指向容器中第一个元素和尾元素之后位置的迭代器。
```c++
list<string> a = {"Milton", "Shakespeare", "Austen"};
auto it1 = a.begin();   //list<string>::iterator
auto it2 = a.rbegin();  //list<string>::reverse_iterator
auto it3 = a.cbegin();  //list<string>::const_iterator
auto it4 = a.crbegin(); //list<string>::const_reverse_string
```
==当不需要访问时，应使用cbegin和cend==
### 容器定义和初始化
|容器定义和初始化||
|---|---|
|C c;|默认构造函数。如果C是一个array，则c中元素按默认方式初始化；否则c为空|
|C c1(c2);<br>C c1=c2;|c1初始化为c2的拷贝,相同的容器类型，且保存的是相同的元素类型|
|C c{a, b, c...}<br>C c={a, b, c...}|c初始化为初始化列表中元素的拷贝|
|C c(b,e)|c初始化为迭代器b和e指定范围中的元素拷贝|
|C seq(n)|seq包含n个元素，这些元素进行了值初始化|
|C seq(n,t)|seq包含n 个初始化为值t的元素|

==当将一个容器初始化为另一个容器的拷贝时，两个容器的容器类型和元素类型都必须相同==
**标准库array具有固定大小**
```c++
array<int, 42>;     //类型为：保存42个int类型
array<string, 10>;  //类型为：保存10个string数组
```
初始化array
```c++
array<int, 10> ia1;         //10个默认初始化的int
array<int, 10> ia2 = {0,1,2,3,4,5,6,7,8,9};  //列表初始化
array<int, 10> ia3 = {42};  //ia3[0]为42，剩余元素为0
```
### 赋值和swap
|容器赋值运算||
|---|---|
|c1=c2 |将c1中的元素替换为c2中的拷贝|
|c = {a,b,c...}|将c1中的元素替换为初始化列表中的元素拷贝|
|swap(c1,c2)<br>c1.swap(c2)|交换c1和c2中的元素。c1和c2必须具有相同的类型。swap通常比从c2向c1拷贝元素快得多|
|seq.assign(b,e)|将seq中的元素替换为迭代器b和e所表示的范围中的元素。|
|seq.assign(il)|将seq中的元素替换为初始值列表il中的元素|
|seq.assign(n,t)|将seq中的元素替换为n个值为t的元素|

==赋值相关运算会导致指向左边容器内部的迭代器、引用和指针失效。而swap操作将容器内容交换不会导致容器的迭代器、引用和指针失效==
### 容器大小操作
成员函数size返回元素数目；empty当size为0时返回布尔值true，否则返回false；max_size返回一个大于或等于该类型容器所能容纳的最大素数元素的值。
### 关系运算符
- 如果两个容器具有相同大小且所有元素位置都两两对应，则两个容器相等
- 如果两个容器大小不同，但较小容器中每个元素都等于较大容器中的对应元素，则较小的容器小于较大的容器
- 如果两个容器不是另一个容器的前缀子序列，则他们的比较结果取决于第一个不相等的元素比较结果
  
## 9.3顺序容器操作
### 向顺序容器添加元素
|向顺序容器中添加元素||
|---|---|
|c.push_back(t)<br>c.emplace_back(args)|在c的尾部创建一个值为t或由args创建的元素。返回void|
|c.emplace_font(t)<br>c.emplace_font(args)|在c的头部创建一个值为t或由args创建的元素。返回void|
|c.insert(p,t)<br>c.emplace(p,args)|在迭代器p指向的元素之前创建一个值为t或由args创建的元素。返回指向新添加的第一个元素迭代器|
|c.insert(p,n,t)|在迭代器p指向的元素之前插入n个值为t的元素。返回指向新添加的第一个元素迭代器;若n为0,则返回p|
|c.insrert(p,b,e)|将迭代器b和e指定的范围内的元素插入到迭代器p指向的元素之前。b和e不能指向c中的元素。返回指向新添加的第一个元素的迭代器;若范围为空,则返回p|
|c.insert(p,il)|il是一个花括包围的元素值列表。将这些给定的值插入到迭代器p指向的元素之前。返回指向新添加的第一个元素的迭代器;若列表为空,则返回p|

### 访问元素
|在顺序容器中访问元素的操作||
|---|---|
|c.back()|返回c中尾元素的引用。若c为空,函数行为未定义|
|c.front()|返回c中首元素的引用。若c为空，函数行为未定义|
|c[n]|返回c中下标为n 的元素引用,n是一个无符号整数。若n>=c.size()则函数行为未定义|
|c.at(n)|返回下标为n的元素的引用。如果下标越界，则抛出一out_of_range异常|

**访问成员函数返回的是引用**
```c++
if(!c.empty())
{
    c.front() = 42;         //将42赋予c中的第一个元素
    auto &v = c.back();     //获得指向最后一个元素的引用
    v = 1024;               //改变c中的元素
    auto v2 = c.back();     //v2不是一个引用，它是c.back()的一个拷贝
    v2 = 0;                 //未改变c中的元素
}
```
### 删除元素
|顺序容器的删除操作||
|---|---|
|c.pop_back()|删除c中尾元素。若c为空，则函数行为未定义。函数返回void|
|c.pop_front()|删除c中首元素。若c为空，则函数行为未定义。函数返回void|
|c.erase(p)|删除迭代器p所指的元素，返回一个指向被删除元素之后元素的迭代器,若p指向尾元素,则返回尾后迭代器。若p是尾后迭代器则函数行为未定义|
|c.erase(b,e)|删除迭代器b和e所指定范围内的元素，返回一个指向最后一个被删除元素之后元素的迭代器,若e本身就是尾后迭代器,则函数也返回尾后迭代器|
|c.clear()|删除c中所有元素。返回void|

### 特殊的forward_list操作
|在forward_list中插入或删除元素的操作||
|---|---|
|lst.before_begin()<br>lst.cbefore_begin()|返回指向链表首元素之前不存在的元素迭代器。此迭代器不能解引用。cbefore_begin()返回一个const_iterator()|
|lst.insert_after(p,t)<br>lst.insert_after(p,n,t)<br>lst.insert_after(p,b,e)<br>lst.insert_after(p,li)|在迭代器p之后的位置插入元素。t是一个对象,n是数量,b和e是表示范围的一对迭代器,li是一个花括号列表。返回一个指向最后一个插入元素的迭代器。如果范围为空,则返回p。若p为尾后迭代器，则函数行为未定义|
|emplace_after(p,args)|使用args在p指定的位置之后创建一个元素。返回一个指向这个新元素的迭代器。若p为尾后迭代器，则函数行为未定义|
|lst.erase_after(p)<br>lst.erase_after(b,e)|删除p指向的位置之后的元素，或删除从b之后直到e之间的元素。返回一个指向被删除元素之后元素的迭代器,若不存在这样的元素,则返回尾后迭代器。如果p指向lst的尾元素或者是一个尾后迭代器，则函数行为未定义|
### 改变容器大小
|顺序容器大小操作||
|---|---|
|c.resize(n)|调整c的大小为n的元素。若n < c.size(),则多出的元素被丢弃。若必须添加新元素,对新元素进行初始化|
|c.resiize(n,t)|调整c的大小为n个元素。任何新添加的元素都初始化为值t|
### 容器操作可能使迭代器失效
==使用失效的迭代器、指针或引用时严重的运行时错误==

## 9.4vector对象时如何增长的
|容器大小管理操作||
|---|---|
|c.shink_to_fit()|请将capacity()减小为与size()相同大小|
|c.capacity()|不重新分配内存空间的话,c可以保存多少元素|
|c.reverve(n)|分配至少能容纳n个元素的内存空间|

## 9.5额外的string操作
### 构造string的其他方法
|构造string的其他方法||
|---|---|
|string s(cp,n)|s是cp指向的数组中前n个字符的拷贝。此数组至少应噶u包含n个字符|
|string s(s2,pos2)|s是string s2从下标pos2开始的字符拷贝。若pos2>s2.size(),构造函数的行为为定义|
|string s(s2,pos2,len2)|s是string s2从下标pos2开始len2个字符拷贝。若pos2>s2.size(),构造函数行为为定义。不管len2的值是多少,构造函数至多拷贝s2.size()-pos2个字符|

**substr操作**
substr操作返回一个string,它是原始string的一部分或全部拷贝。可以传递给substr一个可选的开始位置和计数值:
```c++
string s("hello world");
string s2 = s.substr(0, 5);     //s2 = hello
string s3 = s.substr(6);        //s3 = world
string s4 = s.substr(6, 11);    //s3 = world
string s5 = s.substr(12);       //抛出一个out_of_range异常
```

### 改变string的其他方法
string定义了两个额外的成员函数:append和replace
```c++
string s("C++ Primer"), s2 = s;     //将s和s2初始化为"C++ Primer"
s.insert(s.size(), " 4th Ed.")      // s == "C++ Primer 4th Ed."
s.appand(" 4th Ed.")        //等价方法:将" 4th Ed."追加到s2; s == s2
//将“4th”替换为“5th”的等价方法
s.erase(11, 3);             // s == "C++ Primer Ed."
s.insert(11, "5th");        // s == "C++ Primer 5th Ed."
//从位置11开始,删除3个字符并插入"5th"
s2.replace(11, 3, "5th");   //等价方法: s == s2
```

|修改string的操作||
|---|---|
|s.insert(pos,args)|在pos之前插入args指定的字符。pos可以是一个下标或是一个迭代器。接受下标的版本返回一个指向s的引用;接受的迭代器的版本返回指向第一个插入字符的迭代器|
|s.erase(pos,len)|删除从位置pos开始的len个字符。如果len被省略,则删除从pos开始直至s末尾的所有字符。返回一个指向s的引用|
|s.assign(argx)|将s中的字符替换为args制定的字符。返回一个指向s的引用|
|s.append(args)|将args追加到s。返回一个指向s的引用|
|s.replace(range,args)|删除s中范围range内的字符,替换为args指定的字符。range或者是一个下标和一个长度,或者是一对指向s的迭代器。返回一个指向s的引用|
|str|字符串str|
|str,pos,len|str中从pos开始最多len个字|
|cp,len|从cp指向的字符数组的前len个字符|
|cp|cp指向以空字符串结尾的字符数组|
|n,c|n个字符c|
|b,e|迭代器b和e指定范围内的字符|
|初始化列表|花括号包围的，以逗号分隔的字符列表|

|replace|replace|insert|insert|args可以是|
|---|---|---|---|---|
|(pos,len,args)|(b,e,args)|(pos,args)|(iter,args)|
|是|是|是|否|str|
|是|否|是|否|str,pos,len|
|是|是|是|否|cp,len|
|是|是|否|否|cp|
|是|是|是|是|n,c|
|否|是|否|是|b2,e2|
|否|是|否|是|初始化列表|

**改变string的多种重载函数**
assign和append函数无须指定要替换string中哪个部分:assign总是替换string中的所有内容, append总是将新字符追加到string末尾。

### string搜索操作
==string搜索函数返回string::size_type值,该类型是一个unsigned类型。因此,用一个int或其他带符号类型来保存这些函数的返回值不是一个好主意==

|string搜索操作||
|---|---|
|s.find(args)|查找s中args第一次出现的位置|
|s.rfind(args)|查找s中args最后一次出现的位置|
|s.find_first_of(args)|在s中查找args中任何一个字符第一次出现的位置|
|s.find_last_of(args)|在s中查找args中任何一个字符最后一侧出现的位置|
|s.find_first_not_of(args)|在s中查找第一个不在args中的字符|
|s.find_last_not_of(args)|在s中查找最后一个不在args中的字符|
|args必须是一下形式之一||
|c,pos|从s中位置pos开始查找字符c。pos默认为0|
|s2,pos|从s中位置pos开始查找字符串s2。pos默认为0|
|cp,pos|从s中位置pos开始查找指针cp指向的以空字符串结尾的C风格字符串。pos默认为0|
|cp,pos,n|从s中位置pos开始查找指针cp指向的数组的前n个字符。pos和n无默认值|

**指定在哪里开始搜索**
```c++
string::size_type pos = 0;
//每步循环查找name中下一个数
while ((pos = name.find_first_of(number, pos))
                            != string::npos) {
    cout << "found numbers at index: " << pos
         << " element is " << name[pos] << endl;
    ++pos;  //移动到下一个字符
    }
```
**逆向搜索**
- find_last_of搜索与给定string中任何一个字符匹配的最后一个字符
- find_last_not_of搜索最后一个不出现在给定string中的字符。
  
### compare函数
|s.compare的几种参数形式||
|---|---|
|s2|比较s和s2|
|pos1, n1, s2|将s中从pos1开始的n1个字符与s2进行比较|
|pos1, n1, s2, pos2, n2|将s中从pos1开始的n1个字符与s2中从pos2开始的n2个字符进行比较|
|cp|比较s与cp指向的以空字符串结尾的字符串数组|
|pos1, n1, cp|将s中从pos1开始的n1个字符与cp指向的以空字符串结尾的字符串数组进行比较|
|pos1, n1, cp, n2|将s中从pos1开始的n1个字符与指针cp指向的地址开始的n2个字符进行比较|

### 数值转换
新标准中引入了多个函数，可以实现数值数据与标准库数据string之间的转换
```c++
int i = 42;
string s = to_string(i);        //将整数i转换为字符表示形式
double d = stods(s);            //将字符串s转换为浮点数
```
|string和数值之间的转换||
|---|---|
|to_string(val)|一组重载函数,返回数值val的string表示。val可以是任何算数类型。对每个浮点类型和int或更大的整型,都有相应版本的to_string。与往常一样,小整型会被提升|
|stoi(s, p, b)<br>stol(s, p, b)<br>stoul(s, p, b)<br>stoll(s, p, b)<br>stoull(s, p, b)<br>stoull(s, p, b)|返回s的起始字串的数值,返回值类型分别为int、long、unsigned long、long long、unsigned long long。b表示转换所用的基数,默认值为10。p是size_t指针,用来保存s中第一个非数值字符的下标,p默认为0,即,函数不保存下标|
|stof(s, p)<br>stod(s, p)<br>stold(s, p)|返回s的起始字符串的数值,返回值类型分别是float、double、long double。参数p 的作用与整数转换函数中的一样|
## 9.6容器适配器
|所有容器适配器都支持的操作和类型||
|---|---|
|size_type|一种类型,足以保存当前类型的最大对象的大小|
|value_type|元素类型|
|container_type|实现适配器的底层容器类型|
|A a;|创建一个名为a的空适配器|
|A a(c);|创建一个名为a的适配器,带有容器c的一个拷贝|
|关系运算符|每个适配器都支持所有的关系运算符:==、！=、<、<=、>、>=这些运算符返回底层容器的比较结果|
|a.empty()|若a包含任何元素,返回false,否则返回true|
|a.size()|返回a中的元素数目|
|swap(a,b)<br>a.swap(b)|交换a和b的内容,a和b必须有相同类型,包括底层容器类型也必须相同|

**栈适配器**
```c++
stack<int> intStack     //空栈
//填满栈
fot (size_t ix = 0; ix != 10; ++ix)
    intStack.push(ix);  //intStack保存0到9十个数
while (!intStack.empty()) {  //intStack中有值就循环
    int value = intStack.top();
    //使用栈顶值的代码
    intStack.pop();     //弹出栈顶元素,据需循环
}
```
|为列出的栈操作||
|---|---|
|s.pop()|删除栈顶元素,但不返回该元素值|
|s.push(item)<br>s.emplace(args)|创建一个新元素压入栈顶,该元素通过拷贝或移动item而来,或者有args构造|
|s.top()|返回栈顶元素,但不将元素弹出栈|
**队列适配器**
|为列出的queue和priority_queue||
|---|---|
|q.pop()|返回queue的首元素或priority_queue的最高优先级的元素,但不返回此元素|
|q.front()|返回首元素或尾元素,但不删除此元素|
|q.back()|只适用于queue|
|q.top()|返回最高优先级元素,但不删除元素|
|q.push(item)<br>q.emplace(args)|在queue末尾或priority_queue中恰当的位置创建一个元素,其值为item,或者由args构造|

# 第10章 泛型算法
## 10.1概述
大多数算法都定义在头文件algorithm中。
```c++
int val = 42;       //我们将查找的值
//如果在vec中找想要的元素,则返回结果指向它,否则返回结果为vec.cend()
auto result = find(vec.cbegin(), vec.cend(), val);
//报告结果

cout  << "The value " << val
      << (result == vec.cend()
            ? " is not prensent" : " is present") << endl;
```
## 10.2初识泛型算法
### 只读算法
一些算法只会读取输入范围中的元素,而不改变元素
```c++
//对vec中的元素求和,和的初值是0
int sum = accumulate(vec.cbegin(), vec.cend(), 0);
```
**算法和元素类型**
accumulate将第三个参数作为求和起点,这蕴含这一个编程假定:将元素类型加到和的类型上的操作必须是可行的。
```c++
string sum = accumulate(v.cbegin(), v.cend(), string(""));
```
**操作两个序列的算法**
另一个制度算法是equal,用于确定两个序列是否保存相同的值。
```c++
//roster2中的元素数目至少与roster1一样多
equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
```
### 写容器的算法
算法fill接受一对迭代器表示一个范围,还接受一个值作为第三个参数。fill将给定的这个值赋予输入序列中的每个元素。
```c++
fill(vec.cbegin(), vec.cend(), 0);      //将每个元素重置为0
//容器的一个子序列设置为10
fill(vec.cbegin(), vec.cbegin() + vec.size()/2, 10);
```
**算法不检查写操作**
==向目的位置迭代器写入数据的算法假定目的位置足够大,能容纳要写入的元素。==
**介绍back_inserter**
一种保证算法有足够元素空间来容纳输出数据的方法是使用**插入迭代器**。back_inserter是定义在头文件interator中的一个函数。
back_inserter接受一个指向容器的引用,返回一个与该容器绑定的插入迭代器。当我们通过此迭代器赋值时,赋值运算符会调用push_back将一个具有给定值的元素添加到容器中:
```c++
vector<int> vec;
auto it = back_inserter(vec);   //通过它赋值会将元素添加到vec中
*it = 42;   //vec中现在有一个元素,值为42
```
**拷贝算法**
我们可以用copy实现内置数组的拷贝
```c++
int a1[] = {0,1,2,3,4,5,6,7,8,9};
int a2[sizeof(a1)/sizeof(*a1)];     //a2与a1大小一样
//ret指向拷贝到a2的尾元素之后的位置
auto ret = copy(begin(a1), end(a1), a2);    //把a1的内容拷贝给a2
```
### 重排容器元素的算法
**消除重复单词**
```c++
void elimDups(vector<string> &word)
{
    //按字典序排序words,以便查找单词
    sort(words.begin(), words.end());
    //unique重排输入范围i,使得每个单词只出现一次
    //排列在范围的前部,返回指向不重复区域之后一个位置的迭代器
    auto end_unique = unique(words.begin(), words.end());
    //使用向量操作erase删除重复单词
    words.erase(end_unique, words.end());
}
```
**使用容器操作删除元素**
为了真正地删除无用元素,我们必须使用容器操作。
## 10.3定制操作
### 向算法传递函数
为了按长度重排vector,我们将使用sort的第二个版本,此版本是重载过的,它接受第三个参数,此参数是一个谓词。
**谓词**
谓词是一个可调用的表达式,其返回结果是一个能用作条件的值。标准库算法所使用的谓词分为两类:一元谓词和二元谓词。
### lambda表达式
**介绍lambda**
我们可以向一个算法传递任何类别的**可调用对象**(函数、函数指针、重载了函数调用运算符的类以及lambda表达式)
一个lambda表达式表示一个可调用的代码单元。我们可以将其理解为一个未命名的内联函数。与任何函数类似,一个lambda具有一个返回类型、一个参数列表和一个函数体。但与函数不同,lambda可能定义在函数内部。一个lambda表达式具有如下形式
[capture list](parameter list) -> return type { function body }
==如果lambda的函数体包含任何单一return语句之外的内容,且未指定返回类型,则返回void==
**向lambda传递参数**
```c++
[](const string &a, const string &b)
    { return a.size() < b.size();};
```
**使用捕获列表**
```c++
[sz](const string &a)
    { return a.size() >= sz; };
```
==一个lambda只有在其捕获列表中捕获一个它所在函数中的局部变量,才能在函数体中使用该变量==
**调用find_if**
```c++
//获取一个迭代器,指向第一个满足size()>= sz的元素
auto wc = find_if(words.begin(), words.end(),
    [sz](const string &a)
        {return a.size() >= sz; });
```
**for_each**
```c++
//打印长大于等于给定值的单词,每个单词后面接一个空格
for_each(wc, words.end(),
        [](const string &s){cout << s << "";});
cout << endl;
```
### lambda捕获和返回
**值捕获**
与参数不同,被捕获的变量的值是在lambda创建是拷贝的,而不是调用时拷贝:
```c++
void fcn1()
{
    size_t v1 = 42; //局部变量
    //将v1拷贝到名为f的可调用对象
    auto f = [v1]{return v1;};
    v1 = 0;
    auto j = f();   //j为42;f保存了我们创建它时v1的拷贝
}
```
**引用捕获**
```c++
void fcn2()
{
    size_t v1 = 42; //局部变量
    //对象f2包含v1的引用
    auto f2 = [&v1] {return v1;};
    v1 = 0;
    auto j = f2();  //j为0;f2保存v1的引用,而非拷贝
}
```
==当以引用方式捕获一个变量时,必须保证在lambda执行时变量是存在的==
**隐式捕获**
|lambda捕获列表||
|---|---|
|[ ]|空捕获列表。lambda不能使用所在函数中的变量。一个lambda只有捕获变量后才能使用它们|
|[names]|names是一个逗号分隔的名字列表,这些名字都是lambda所在函数的局部变量。默认情况下,捕获列表中的变量都被拷贝。名字前如果使用了&,则采用引用捕获的方式|
|[&]|隐式捕获列表,采用引用捕获方式。lambda体所使用的来自所在函数实体都采用引用的方式使用|
|[=]|隐式捕获列表,采用隐式捕获方式。lambda体将拷贝所使用的来自所在函数的实体的值|
|[&,identifier_list]|identifier_list是一个逗号分割的列表,包含0和多个来自所在函数的变量。这些变量采用值捕获的方式,而任何隐式捕获的变量都采用引用的方式捕获。identifier_list中的名字前面不能使用&|
|[=,identifier_list]|identifier_list中的变量都采用引用方式捕获,而任何隐式捕获的变量都采用值方式捕获。identifier_list中的名字不能包括this,且这些名字之前必须使用&|
### 参数绑定
**标准库bind函数**
调用bind的一般形式
auto *newCallable* = bind(callable,arg_list);
其中*newCallable*本身是一个可调用对象,arg_list是一个逗号分隔的参数列表,对应给定的callable参数。即,当我们调用*newCallable*时,*newCallable*会调用callable,并传递给它arg_list中的参数。
arg_list中的参数可能包含形如_n的名字,其中n是一个整数。这些参数是占位符,表示*newCallable*的参数,它们占据了传递给*newCallable*的参数的“位置”。
**绑定check_size的sz参数**
```c++
//check6是一个可调用对象,接受一个string类型的参数
//并用此string和值6来调用check_size
auto check6 = bind(check_size, _1, 6);
```
使用bind,我们可以将原来基于lambda的find_if调用:
```c++
auto wc = find_if(words.begin(),words.end(),
                [sz](const string &a);
```
替换为如下使用check_size的版本:
```c++
auto wc = find_if(words.begin(),words.end(),
                bind(check_size, _1, sz));
```
**使用placeholders名字**
```c++
using namespace namespsace_name;
```
这种形式说明希望所有来自namespace_name的名字都可以在我们的程序中直接使用。
## 10.4再探迭代器
### 插入迭代器
|插入迭代器操作||
|---|---|
|it = t|在it指定的当前位置插入值t。假扮c是it绑定的容器,依赖于插入迭代器的不同种类,此赋值会分别调用c.push_back(t)、c.push_front(t)或c.insert(t,p),其中p为传递给inseter的迭代器位置|
|*it,++it,it++|这些操作虽然存在,但不会对it做任何事情。每个操作都返回it|
- **back_inserter**创建一个使用push_back的迭代器
- **front_inserter**创建一个使用push_front的迭代器
- **inserter**创建一个使用insert的迭代器。此函数接受第二个参数,这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定的迭代器所表示的元素之前
### iostream迭代器
|istream_iterator操作||
|---|---|
|istream_interator<T> in(is);|in从输入流is读取类型为T的值|
|istream_interator<T> end;|读取类型为T的值的istream_interator迭代器,表示尾后位置|
|in1 == in2<br>in1 != in2|in1与in2必须读取相同类型。如果它们都是尾后迭代器,或绑定到相同的输入,则两者相等|
|*in|返回从流中读取的值|
|in->mem|与(*in).mem的含义相同|
|++in, in++|使用元素类型所定义的>>运算符从输入流中读取下一个值。与往常一样,前置版本返回一个指向递增后迭代器的引用,后置版本返回旧值|
|ostream_iterator操作||
|---|---|
|ostream_interator<T> out(is);|out将类型为T的值写到输入流os中|
|ostream_interator<T> out(os,d);|out将类型为T的值写到输入流os中,每个值后面都输出一个d。d指向一个空字符结尾的字符数组|
|out = val|用<<运算符将val写入到out所绑定的ostream中。val的类型必须与out可写的类型兼容|
|*out,++out, out++|这些运算符是存在的,但不对out做任何事情。每个运算符都返回out|
**使用流迭代器处理类类型**
```c++
istream_interator<Sales_item> item_iter(cin), eof;
ostream_interator<Sales_item> out_iter(cout,"\n");
Sales_item sum = *item_iter++;
while (item_iter != eof){
    if (item_iter->isbn() ==sum.sibn())
        sum += *item_iter++;
    else{
        out_iter = sum;
        sum = *item_iter++;
    }
}
out_iter = sum;
```

### 反向迭代器
```c++
vector<int> vec = {0,1,2,3,4,5,6,7,8,9};
for(auto r_iter = vec.crbegin();
        r_iter != vec.crend();
        ++r_iter)
    cout << *r_iter << endl;
```
## 10.5泛型算法结构
|迭代器类别||
|---|---|
|输入迭代器|只读、不写;单遍扫描,只能递增|
|输出迭代器|只写、不读;单遍扫描,只能递增|
|前向迭代器|可读写;多遍扫描,只能递增|
|双向迭代器|可读写;多遍扫描,可递增递减|
|随机访问迭代器|可读写,多遍扫描,支持全部迭代器运算|
### 5类迭代器
**迭代器类别**
**输入迭代器**
- 用于比较两个迭代器的相等和不相等运算符
- 用于推进迭代器的前置和后置递增运算
- 用于读取元素的解引用运算符(*);解引用只会出现在赋值运算的右侧
- 箭头运算符(->),等价于(*it).member,即,解引用迭代器,并提取对象的成员
**输出迭代器**
- 用于推进迭代器的前置和后置递增运算(++)
- 解引用运算符(*),只出现在赋值运算的左侧
**前向迭代器**
**双向迭代器**
**随机访问迭代器**
- 用于比较两个迭代器相对位置的关系运算符
- 迭代器和一个整数值的加减运算,计算结果是迭代器在序列中前进给定整数个元素后的位置
- 用于两个迭代器上的减法运算,得到两个迭代器的距离
- 下标运算符(iter[n]),与*(iter[n])等价
### 算法的命名规范
**一些算法使用重载形式传递一个谓词**
```c++
unique(beg, end);       //使用 == 比较运算符
unique(beg, end, comp); //使用 comp比较元素
```
**区分拷贝元素的版本和不拷贝的版本**
```c++
reverse(beg, end);          //反转输入范围中的元素顺序
reverse(beg, end, dest);    //将元素按逆序拷贝到dest
```
## 10.6特定容器算法
|list和forward_list||
|---|---|
|lst.merge(lst2)<br>lst.merge(lst2,comp)|将来自lst2的元素合并入lst。lst和lst2都必须是有序的。元素将从lst2中删除。在合并之后,lst2变为空。第一版本使用<运算符;第二个版本使用给定的比较操作|
|lst.remove(val)<br>lst.remove_if(pred)|调用erase删除掉与给定值相等(==)或令一元谓词为真的每个元素|
|lst.reverse()|反转lst中的元素|
|lst.sort()<br>lst.sort(comp)|使用<或给定比较操作排序元素|
|lst.unique()<br>lst.unique(pred)|调用erase删除同一个值的连续拷贝。第一个版本使用==;第二个版本使用给定的二元谓词|


# 第11章 关联容器
|关联容器类型||
|---|---|
|**按关键字有序保存元素**||
|map|关联数组;保存关键字-值对|
|set|关键字即值,即只保存关键字的容器|
|multimap|关键字可重复出现的map|
|multiset|关键字可重复出现的set|
|无序集合||
|unordered_map|用哈希函数组织的map|
|unordered_map|用哈希函数组织的set|
|unordered_multimap|哈希组织的map;关键字可以重复出现|
|unordered_multiset|哈希组织的det;关键字可以重复出现|

## 11.1 使用关联容器
map类型通常被称为**关联数组**
set是关键字的简单集合
## 11.2 关联容器概述
关联容器的迭代器都是双向的
### 定义关联容器
```c++
map<string, size_t> word_count;     //空容器
//列表初始化
set<string> exclude = {"the", "but", "and", "or", "an", "a",
                        "The", "But", "And", "Or", "An", "A"};
//三个元素;authors将姓映射为名
map<string, string> authors = {{"Joyce", "James"},
                                {"Austen", "Jane"},
                                {"Dickens", "Charles"}};
```
**初始化multimap或multiset**
```c++
//定义一个有20个元素的vector,保存0到9每个整数的两个拷贝
vector<int> ivec;
for(vector<int>::size_type i = 0;i != 10; ++i)
{
    ivec.push_back(i);
    ivec.push_back(i);//每个数重复保存一次
}
//iset包含来自ivec的不重复元素;miset包含所有20个元素
set<int> iset(ivec.cbegin(), ivec.cend());
multiset<int> miset(ivec.cbegin(), ivec.cend());
cout << ivec.size() << endl;    //打印出20
cout << iset.size() << endl;    //打印出10
cout << miset.size() << endl;   //打印出20
```
### 关键字类型的要求
==传递给排序算法可调用对象必须满足与关联容器中关键字一样的类型要求==
**有序容器的关键词类型**
无论怎么定义比较函数，它必须具备如下性质:
- 两个关键字不能同时“小于等于”对方;如果k1“小于等于”k2,那么k2绝对不能“小于等于”k1
- 如果k1“小于等于”k2,且k2“小于等于”k3,那么k1必须“小于等于”k3
- 如果存在两个关键字,任何一个都不“小于等于”另一个,那么我们称这两个关键字是“等价”的。如果k1“等价于”k2,且k2“等价于”k3,那么k1必须“等价于”k3

**使用关键字类型的比较函数**
```c++
bool compareIsbn(const Sales_data &lhs, const Sales_data &rhs)
{
    return lhs.isbn() < rhs.isbn();
}
//bookstore中多条记录可以有相同的ISBN
//bookstore中的元素以ISBN的顺序进行排列
multiset<Sales_data, decltype(compareIsbn)*>
    bookstore(compareIsbn);
```
当用dacltype来获得一个函数指针类型时，必须加上一个*来指出我们要使用一个给定的函数指针
### pair类型
在介绍关联容器操作之前,我们需要了解名为pair的标准库类型,它定义在头文件utility中。
```c++
pair<string, string> anon;          //保存两个string
pair<string, size_t> word_count;    //保存一个string和一个size_t
pair<string, vector<int>>line;      //保存string和vector<int>
```
|pair上的操作||
|---|---|
|pair<T1, T2> p;|p是一个pair,两个类型分别为T1和T2的成员都进行了初始化|
|pair<T1, T2> p(v1, v2)|p是一个成员类型为T1和T2的pair;first和second成员分别用v1和v2进行初始化|
|pair<T1, T2> p={v1, v2};|等价于p(v1, v2)|
|make_pair(v1, v2)|返回一个用v1和v2初始化的pair。pair类型从v1和v2的类型推断出来|
|p.first|返回p的名为first的数据成员|
|p.second|返回p的名为second的数据成员|
|p1 relop p2|关系运算符(<、>、<=、>=)按字典序定义:例如,当p1.first<p2.first或!(p2.first < p1.first) && p1.second < p2.second成立时,p1 < p2为true。关系运算利用元素的<运算符来实现|
|p1 == p2<br>p1 !=p2|当first和second成员分别相等时,两个pair相等。相等性判断利用元素的==运算符实现|

## 11.3 关联容器操作
|关联容器额外的类型别名||
|---|---|
|key_type|此容器类型的关键字类型|
|mapped_type|每个关键字关联的类型;只适用于map|
|value_type|对于set,与key_type相同<br>对于map,为pair<const key_type, mapped_type>|
对于set类型,key_type和key_type是一样的;set中保存的值就是关键字。由于我们不能改变一个元素的关键字,因此这些pair的关键字部分是const的:
```c++
set<string>::value_type v1;         //v1是一个string
set<string>::key_type v2;           //v2是一个string
map<string, int>::value_type v3;    //v3是一个pair<const string, int>
map<string, int>::key_map v4;       //v4是一个string
map<string, int>::mapped_type v5;   //v5是一个int 
```
### 关联容器迭代器
**set的迭代器是const的**
虽然set类型同时定义了interator和const_iterator类型,但两种类型都只允许访问set中的元素。

### 添加元素
|关联容器insert操作||
|---|---|
|c.insert(v)<br>c.emplace(args)|v是value_type类型的对象;args用来构造一个元素<br>对于map和set,只有当元素关键字不在c中时才插入元素。函数返回一个pair,包含一个迭代器,指向具有关键字的元素,以及一个指向插入是否成功的bool值<br>对于multimap和multiset,总会插入给定元素,并返回一个指向新元素的迭代器|
|c.insert(b, e)<br>c.insert(il)|b和e是迭代器,表示一个c::value_type类型值的范围;il是这种花括号列表。函数返回void<br>对于map和set,只插入关键字不在c中的元素。对于multimap和multiset,则会插入范围中的每个元素|
|c.insert(p, v)<br>c.emplace(p, args)|类似insert(v),但将迭代器p作为一个提示,指出从哪里开始搜索新元素应该存储的位置。返回一个迭代器,指向具有给定关键字的元素|
### 删除元素
|从关联容器删除元素||
|---|---|
|c.erase(k)|从c中删除每个关键字为k的元素。返回一个size_type值,指出删除的元素数量|
|c.erase(p)|从c中删除迭代器p指定的元素。p必须只想c中一个真实元素,不能等于c.end()。返回指向p之后元素的迭代器,若p指向c中的尾元素,则返回c.end()|
|c.erase(b, e)|删除迭代器对b和e所表示的范围中的元素。返回e|
### map的下标操作
==对一个map使用下标操作,其行为与数组或vector上的下标操作很不相同:使用一个不在容器中的关键字作为下标,会添加一个具有此关键字的元素到map中==
|map和unordered_map的下标操作||
|---|---|
|c[k]|返回关键字为k的元素;如果k不在c中,添加一个关键字为k的元素,对其进行值初始化|
|c.at(k)|访问关键字为k的元素,带参数检查;若k不在c中,抛出一个out_of_range异常|
### 访问元素
|在一个关联容器中查找元素操作||
|---|---|
|c.find(k)|返回一个迭代器,指向第一个关键字为k的元素,若k不在容器中,则返回尾后迭代器|
|c.count(k)|返回关键字等于k的元素数量。对于不允许重复关键字的容器,返回值永远是0和1|
|c.lower_bound(k)|返回一个迭代器,指向第一个关键字不小于k的元素|
|c.upper_bound(k)|返回一个迭代器,指向地一个关键字大于k的元素|
|e.equal_range(k)|返回一个迭代器pair,表示关键字等于k的元素范围。若k不存在,pair的两个成员均等于c.end()|
## 11.4 无序容器
|无序容器的管理操作||
|---|---|
|**桶接口**||
|c.bucket_count()|正在使用的桶的数目|
|c.max_bucket_count()|容器中能容纳的最多桶的数量|
|c.bucket_size(n)|第n个桶中有多少个元素|
|c.bucket(k)|关键字为k的元素在哪个桶中|
|**桶迭代**||
|local_iterator|可以用来访问桶中元素的迭代器类型|
|const_local_interator|桶迭代器的const版本|
|c.begin(n), c.end(n)|桶n的首元素迭代器和尾后迭代器|
|c.cbegin(n), c.cend(n)|与钱两个函数类似,但返回const_local_iterator|
|**哈希策略**||
|c.load_factor()|每个桶的平均元素数量,返回float值|
|c.max_load_factor()|c试图维护的平均桶大小,返回float值。c会在需要时添加新的桶,使得load_factor<=max_load_factor|
|c.rehash(n)|重组存储,使得bucket_count>=n且bucket_count>size/max_load_factor|
|c.reverse(n)|重组存储,使得c可以保存n个元素且不必rehash|
# 第12章 动态内存
## 12.1动态内存与智能指针
### shared_ptr类
|shared_ptr和unique_ptr都支持的操作||
|---|---|
|share_ptr<T> sp<br>unique_ptr up|空智能指针,可以指向类型为T的对象|
|p|将p用作一个条件判断,若p指向一个对象,则为true|
|*p|解引用p,获取它指向的对象|
|p->mem|等价于(*p).mem|
|p.get()|返回p中保存的指针。要小心使用,若智能指针释放了其对象,返回的指针所指向的对象也就消失了|
|swap(p, q)<br>p.swap()|交换p和q中的指针|

|shared_ptr独有的操作||
|---|---|
|make_shared<T>(args)|返回一个shared_ptr,指向一个动态分配的类型为T的对象。使用args初始化此对象|
|shared_ptr<T>p(q)|p是shared_ptr q的拷贝;此操作会递增q中的计数器。q中的指针必须能转化为T*|
|p = q|p和q都是shared_ptr,所保存的指针必须能相互转换。此操作会递减p的引用次数,递增q的引用次数;若p的引用计数变为0,则将其管理的原内存释放|
|p.unique()|若p.use_count()为1,返回true;否则返回false|
|p.use_count()|返回与p共享对象的智能指针数量;可能很慢,主要用于调试|
### 直接管理内存
**使用new动态分配和初始化对象**
```c++
int *pi = new int;  //pi指向一个动态分配的、为初始化的无名对象
```
==出于与变量初始化相同的原因,对动态分配内存的对象进行初始化通常是个好主意==
**动态分配的const对象**
用new分配const对象是合法的
```c++
//分配并初始化一个const int
const int *pci = new const int(1024);
//分配并默认初始化一个const的空string
const string *pcs = new const string;
```
**释放内存**
我们通过**delete表达式**来将动态内存归还给系统
### share_ptr和new结合使用
如果我们不初始化一个智能指针,他就会被初始化为一个空指针。
我们还可以用new返回的指针来初始化智能指针
```c++
shared_ptr<double> p1;  //shared_ptr可以指向一个double
shared_ptr<int> p2(new int(42));    //p2指向一个值为42的int
```
|定义和改变shated_ptr的其他方法||
|---|---|
|shared_ptr<T> p(q)|p管理内置指针q所指向的对象;q必须指向new分配的内存,且能够转化为T*类型|
|shared_ptr<T> p(u)|p从unique_ptr u那里接管了对象的所有权;将u置为空|
|shated_ptr<T> p(q, d)|p接管了内置指针q所指向的对象的所有权。q必须能转化为T*类型。p将使用可调用对象d来代替delete|
|shared_ptr<T> p(p2, d)|p是shared_ptr p2的拷贝,唯一的区别是p将可调用对象d来代替delete|
|p.reset()<br>p.reset(q)<br>p.reset(q, d)|若p是唯一指向其对象的shared_ptr,reset会释放此对象。若传递了可选的参数内置指针q,会令p指向q,否则会将p置为空。若还传递了参数d,会将调用d而不是delete来释放q|

### 智能指针和异常
如果使用智能指针,即使程序块过早结束,智能指针类也能确保在内存不再需要时将其释放
```c++
void f()
{
    shared_ptr<int> sp(new int(42));    //分配一个对象
    //这段代码抛出一个异常,且在f中未被捕获
};  //在函数结束时shared_ptr自动释放内存
```
与之相对的,发生异常时,我们直接管理的内存是不会被释放的。如果使用内置指针管理内存,且在new之后在对应的delete之前发生了异常,则内存不会被释放
```c++
void f()
{
    int *ip =  new int(42);     //动态分配一个新对象
    //这段代码抛出一个异常,且在f中未被捕获
    delete ip;      //在推出之前释放内存
};
```
### unique_ptr
一个unique_ptr拥有它所指向的对象。与shared_ptr不同,某个时刻只能有一个unique_ptr指向一个给定对象。当unique_ptr被销毁时,它所指向的对象也被销毁
与shared_ptr不同,没有类似的make_sharedi标准库函数返回一个unique_ptr。当我们定义一个unique_ptr时,需要将其绑定到一个new返回的指针上。类似shared_ptr,初始化unique_ptr必须采用直接初始化形式
```c++
unique_ptr<double> p1;  //可以指向一个double的unique_ptr
unique_ptr<int> p2(new int(42));    //p2指向一个值为42的int
```
|unique_ptr 操作||
|---|---|
|unique_ptr<T> u1<br>unique<T, D> u2|空unique_ptr,可以指向类型为T的对象。u1会使用delete来释放它的指针;u2会使用一个类型为D的可调用对象来释放它的指针|
|unique_ptr<T, D>u(d)|空unique_ptr,指向类型为T的对象,用类型为D的对象d代替delete|
|u = nullptr|释放u指向的对象,将u置为空|
|u.reset()<br>u.reset(q)<br>u.reset(nullptr)|释放u指向的对象<br>如果提供了内置的q,令u指向这个对象;否则将u置为空|
### weak_ptr
weak_ptr是一种不控制所指向对象生存期的智能指针,它指向由一个shared_ptr管理对象。
将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数。
|weak_ptr||
|---|---|
|weak_ptr<T> w|空weak_ptr可以指向类型为T的对象|
|weak_ptr<T> w(sp)|与shared_ptr sp指向相同对象的weak_ptr。T必须能转化为sp指向的类型|
|w = p|p可以是一个shared_ptr或一个weak_ptr。赋值后w与p共享对象|
|w.reset()|将w置为空|
|w.use_count()|与w共享对象的shared_ptr数量|
|w.expired()|若w.use_count()为0,返回true,否则返回falase|
|w.lock()|如果expired为true,返回一个空shared_ptr;否则返回一个指向w的对象的shared_ptr|

## 12.2动态数组
### new和数组
**分配一个数组会得到一个元素类型的指针**
==要记住我们所说的动态数组并不是数组类型,这是很重要的==
**初始化动态分配对象的数组**
```c++
int *pia = new int[10];     //10个未初始化的int
int *pia2 = new int[10]();  //10个值初始化为0的int
string *psa = new string[10];   //10个空string
string *psa2 = new string[10]();    //10个空string
```
**动态分配一个空数组是合法的**
```c++
size_t n = get_size();      //get_size返回需要的元素的数目
int* p = new int[n];        //分配数组保存元素
for (int* q = p; q != p + n; ++q)
    /* 处理数组 */ ;
```
**释放动态数组**
```c++
delete p;       //p必须指向一个动态分配的对象或为空
delete [] pa;   //pa必须指向一个动态分派的数组或为空
```
|指向数组的unique_ptr||
|---|---|
|unique_ptr<T[]> u|u可以指向一个动态分配的数组,数组元素类型为T|
|unique_ptr<T[]> u(p)|u指向内置指针p所指向的动态分配的数组,。p必须能转化为类型T*|
|u[i]|返回u拥有的数组中位置i处的对象<br>u必须是一个数组|

### allocator类
```c++
string *const p = new string[n];    //构造n个空string
string s;
string *q = p;              //p指向第一个string
while(cin >> s && q != p +n)
    *q++ = s;               //赋予*q一个新值
const size_t size q - p;
//使用数组
delete[] p;     //p指向一个数组;记得用delete[]来释放
```
**allocate类**
|标准库allocator||
|---|---|
|allocaor<T> a|定义了一个名为a的allocator对象,它可以为类型T的对象分配的内存|
|a.allocate(n)|分配一段原始的、未构造的内存,保护n个类型为T的对象|
|a.deallocate(p, n)|释放从T*指针p中地址开始的内存,这块内存保存了n个类型为T的对象;p必须是一个先前由allocate返回的指针,且n必须是p创建时所要求的大小。在调用deallocate之前,用户必须是p创建时所要求的大小。在调用deallocate之前,用户必须对每个在这块内存中创建对象调用destroy|
|a.construct(p, args)|p必须是一个类型为T*的指针,指向一块原始内存;args被传递给类型为T的构造函数,用来在p指向内存中构造一个对象|
|a.destroy(p)|p为T*类型的指针,此算法对p指向的对象执行析构函数|



# 第13章 拷贝控制
一个类通过定义五种特殊的成员函数来控制这些操作，包括:**拷贝构造函数、拷贝赋值运算符、移动构造函数、移动赋值运算符和析构函数**
我们称这些操作为**拷贝控制操作**

## 13.1拷贝、赋值与销毁
### 拷贝构造函数
```c++
class Foo{
public:
    Foo();                   //默认构造函数
    Foo(const Foo&);         //拷贝构造函数
}
```
拷贝构造函数第一个参数必须是一个引用类型
**合成拷贝构造函数**
```c++
class Sales_data {
public:
    //其他成员和构造函数的定义，如前
    //与合成的拷贝构造函数等价的拷贝构造函数声明
    Sales_data(const Sales_data&);
private:
    std::string bookNo;
    int units_sold = 0;
    double revenue = 0.0;
};
//与Sales_data的合成的拷贝构造函数等价
Sales_data::Sales_data(const Sales_data &orig):
    bookNo(orig.bookNo),        //使用string的拷贝构造函数
    units_sold(orig.units_sold),//拷贝orig.units_sold
    revenue(orig.revenue)       //拷贝orig.revenue
    {   }                       //空函数体
```

**拷贝初始化**
```c++
string dots(10, '.');               //直接初始化
string s(dots);                     //直接初始化
string s2 = dots;                   //拷贝初始化
string null_book = "9-999-99999-9";  //拷贝初始化
string nines = string(10, '9');      //拷贝初始化
```

### 拷贝赋值运算符
**重载赋值运算符**
重载运算符本质上是函数，其名字由operator关键字后面接表示要定义的运算符的符号组成。
```c++
class Foo{
public:
    Foo& operator=(const Foo&);  //赋值运算符
}
```
==赋值运算符通常应该返回一个指向其左侧运算对象的引用==
**合成赋值运算符**
```c++
Sales_data&
Sales_data::operator=(const Sales_data &rhs)
{
    bookNo = rhs.bookNo;            //调用string::operator=
    units_sold = rhs.units_sold;    //使用内置的int赋值
    revenue = rhs.revenue;          //使用内置的double赋值
    return *this;                   //返回一个此对象的引用
}
```

### 析构函数
析构函数释放对象使用的资源，并销毁对象的非static数据成员。
```c++
class Foo {
public:
    ~Foo();  //析构函数
}
```
由于析构函数不接受参数，因此它不能被重载。
==隐式销毁一个内置指针类型的成员不会delete它所指的对象==

**什么时候会调用析构函数**
无论何时一个对象被销毁，就会自动调用其析构函数。
- 变量在离开其作用域时被销毁
- 当一个对象被销毁，其成员被销毁
- 容器被销毁时，其元素销毁
- 对于动态分配的对象，当对指向它的指针应用delete运算符时被销毁
- 对于临时变量，当创建它的完整表达式结束时被销毁
```c++
{//新的作用域
    //p和p2指向动态分配对象
    Sales_data *p = new Sales_data;     //p是个内置指针
    auto p2 = make_shared<Sales_data>(); //p2是个shared_ptr
    Sales_data item(*p);                 //拷贝构造函数将*p拷贝到item中
    vector<Sales_data> vec               //局部对象
    vec.push_back(*p2);                 //拷贝p2指向的对象
    delete p;                           //对p指向的对象执行析构操作
}//推出局部作用域:对item、p2和vec调用析构函数
 //销毁p2会递减其引用计数;如果引用计数变为0,对象被释放
 //销毁vec会销毁它的元素
```
==当指向一个对象的引用或指针离开作用域时,析构函数不会执行==
**合成析构函数**
```c++
class Sales_data {
public:
    //成员会被自动销毁,除此之外不需要做其他事情
    ~Sales_data() { }
};
```
### 三/五法则
**需要析构函数的类也需要拷贝和赋值操作**
==如果一个类需要自定义析构函数,几乎可以肯定它也需要自定义拷贝赋值运算符和拷贝构造函数==

### 使用=default

```c++
class Sales_data {
public:
    //拷贝控制成员;使用default
    Sales_data() = default;
    Sales_data(const Sales_data&) = default;
    Sales_data& operator=(const Sales_data&) = default;
    ~Sales_data() = default;
    //其他成员的定义,如前
};
```
==我们只能对具有合成版本的成员函数使用=default==
### 阻止拷贝
**定义删除函数**
我们可以通过将拷贝构造函数和拷贝赋值运算符定义为删除的函数来阻止拷贝。
```c++
struct NoCopy {
    Nocopy() = default;     //使用合成的默认构造函数
    Nocopy(const Nocopy&) = delete;   //阻止拷贝
    Nocopy& operator=(const Nocopy&) = delete;  //阻止赋值
    ~Nocopy() = default;    //使用合成的析构函数
};
```
**析构函数不能是被删除的成员**
```c++
struct NoDtor {
    NoDtor() = default;     //使用合成默认构造函数
    ~NoDtor() = delete;     //我们不能销毁NoDtor对象
};
NoDtor nd;   //错误:NoDtor的析构函数是删除的
NoDtor *p = new NoDtor();   //正确:但我们不能delete p
delete p;   //错误:NoDtor的析构函数是删除的
```

## 13.2拷贝控制和资源管理
### 行为像值的类

```c++
class HasPtr {
public:
    HashPtr(const std::string &s = std::string()):
        ps(new std::stringz(s)), i(0) { }
    HashPtr(const HashPtr &p):
        ps(new std::string(*p.ps)), i(p.i) { }
    HashPtr& operator=(const HashPtr &);
    ~HasPtr() { delete ps; }
private:
    std::string *ps;
    int i;
}
```
### 定义行为像指针的类
```c++
class HasPtr {
public:
    HasPtr(const std:: sting &s = std::string()):
        ps(new std::string(s)), i(0), use(new std::size_t(1)) {}
    HasPtr(const HasPtr &p):
        ps(p.ps), i(p.i), use(p.use) { ++*use; }
    HasPtr& operator=(const HasPtr&);
    ~HasPtr();
private:
    stdd::string *ps;
    int i;
    std::size_t *use;
};
```
## 13.3交换操作
```c++
class HashPtr {
    friend coid swap(HashPtr&, HashPtr&);
};
incline
void swap(HashPtr &lhs, HashPtr& rhs)
{
    using std::swap;
    swap(lhs.ps, rhs.ps);
    swap(lhs.i, rhs.i);
}
```
## 13.4拷贝控制示例

## 13.5动态内存管理类

## 13.6对象移动
### 右值引用
```c++
int i = 42;
int &r = i;         //正确:r引用i
int &&rr = i;       //错误:不能将一个右值引用绑定到一个左值上
int &r2 = i * 42;   //错误:i*42是一个右值
const int &r3 = i * 42;     //正确:我们可以将一个const的引用绑定到一个右值上
int &&rr2 = i * 42; //正确:将rr2绑定到乘法结果上
```
**标准库move函数**
```c++
int &&rr3 = std::move(rr1);
```
move告诉编译器:我们有一个左值,但我们希望像一个右值一样处理它。我们必须认识到,调用move就意味着承诺:除了rr1赋值或销毁它外,我们将不再使用它

# 第14章 重载运算与类型转化
## 14.1基本概念
==当一个重载运算符是成员函数时,this绑定到左侧运算对象。成员运算符函数的显式参数数量比运算对象的数量少一个==
**直接调用一个重载的运算符**
```c++
\\一个非成员运算符的函数的等价调用
data1 + data2;              //普通的表达式
operator+(data1, data2);    //等价的函数调用
```
==通常情况下,不应该重载逗号、取地址、逻辑与和逻辑或运算符==

## 14.2输入和输出运算符

### 重载输出运算符<<
**Sales_data的输出运算符**
```c++
ostream &operator<<(ostream &os, const Sales_data &item)
{
    os << item.isbn() << " " << item.units_sold << " "
       << item.revenue << " " << item.avg_price();
    retuen os;
}
```

### 重载输入运算符>>
**Sales_data的输入运算符**
```c++
istream &operator>>(istream &is, Sales_data &item)
{
    double price;
    is >> item.bookNo >> item.units_sold >> price;
    if(is)
        item.revenue = item.units_sold * price;
    else
        item = Sales_data();
    return is;
}
```
## 14.3算数和关系运算符
```c++
Sales_data
operator+(const Sales_data &lhs, const Salse_data &rhs)
{
    Sales_data sum = lhs;
    sum += rhs;
    return sum;
}
```
### 相等运算符
```c++
bool operator==(const Sales_data &lhs, const Sales_data &rhs)
{
    return lhs.isbn() == rhs.isbn() &&
           lhs.units_sold == rhs.units_sold &&
           lhs.revenue == rhs.revenue;
}
bool operator!=(const Sales_data &lhs, const Sales_data &rhs)
{
    return !(lhs == rhs);
}
```
### 关系运算符
1.定义顺序关系,令其与关联容器中对关键字的要求一致
2.如果类同时也含有==运算符的话,则定义一种关系令其与==保持一致。特别是,如果两个对象是!=的,那么一个对象
## 14.4赋值运算符
```c++
StrVec &StrVec::operator=(initializer_list<string> il)
{
    auto data = alloc_n_copy(il.begin(), il.end());
    free();
    element = data.first;
    first_free = cap = data.second;
    return *this;
}
```
**复合重载运算符**
```c++
Sales_data &Sales_data::operator+=(const Sales_data &rhs)
{
    units_sold += rhs.units_sold;
    revenue += rhs.revenue;
    return this;
}
```
## 14.5下标运算符
==下标运算符必须是成员函数==
==如果一个类包含下标运算符,则它通常会定义两个版本:一个返回普通引用,另一个是类的常量成员并且返回常量引用==
## 14.6递增和递减运算符
**定义前置递增/递减运算符**
```c++
class StrBlobPtr {
public:
    StrBlobPtr& operator++();
    StrBlobPtr& operator--();
}
```
==为了与内置版本保持一致,前置运算符应该返回递增和递减后对象的引用== 
**区分前置和后置运算符**
```c++
class StrBlobPtr {
public:
    StrBlobPtr operator++(int);
    StrBlobPtr operator--(int);
}
```
==为了与内置版本保持一致,后置运算符应该返回对象的原值,返回的形式是一个值而非引用==
## 14.7 成员访问运算符
```c++
class StrBlobPtr{
public:
    std::string& operator*() const
    {
        auto p = check(curr, "dereference past end");
        return (*p)[curr];
    }
    std::string* operator->() const
    {
        return & this->operator*();
    }
}
```
==箭头运算符必须是类的成员。解引用运算符通常也是类的成员,尽管并非必须如此。==
## 14.8函数调用运算符
```c++
struct absInt {
    int operator()(int val) const {
        return val < 0 ? -val : val;
    }
}
```
### lambda是函数对象
```c++
class ShorterString {
public:
    bool operator()(const string &s1, const string &s2) const
    { return s1.size() < s2.size(); }
}
```
### 标准库定义的函数对象
||标准库函数对象||
|---|---|---|
|**算数**|**关系**|**逻辑**|
|plus<Type>|equal_to<Type>|logical_and<Type>|
|minus<Type>|not_equal_to<Type>|logical_or<Type>|
|multiplies<Type>|greater<Type>|logical_noe<Type>|
|divides<Type>|greater_equal<Type>||
|modulus<Type>|less<Type>||
|negate<Type>|less_equal<Type>||
### 可调用对象与function
C++语言中有几种可调用的对象:函数、函数指针、lambda表达式、bind创建对象以及重载了函数调用的运算符的类
**不同类型可能具有相同的调用形式**
```c++
int add(int i, int j) { return i + j; }
auto mod = [](int i, int j) { return i % j; };
struct divide {
    int operator()(int denominator, int divisor) {
        return denominator / divisor;
    }
};
```

|function的操作||
|function<T> f;|f是一个用来存储可调用对象的空function,这些可调用对象的调用形式应该与函数类型T相同|
|function<T> f(nullptr);|显示地构造一个空function|
|function<T> f(obj);|在f中存储可调用对象obj的副本|
|f|将f作为条件:当f含有一个可调用对象为真;否则为假|
|f(args)|调用f中的对象,参数是args|
|result_type|该function类型的可调用对象返回的类型|
|argument_type<br>first_argument_type<br>second_argument_type|当T有一个或两个实参时定义的类型。如果T只有一个实参,则argument_type是该类型的同义词;如果T有两个实参,则first_argument_type和second_argument_type分别代表两个实参的类型|
## 14.9重载、类型转换与运算符
### 类型转换运算符
```c++
operator type() const;
```
**定义含有类型转换运算符的类**
```c++
class SmallInt {
public:
    SmallInt(int i = 0): val(i)
    {
        if(i < 0 || i > 255)
            throw std::out_of_range("Bad SmallInt Value");
    }
    operator int() const { return val; }
private:
    std::size_t val;
}
```
### 避免有二义性的类型转换
### 函数匹配与重载运算符

# 第15章 面向对象设计

## 15.1 OOP:概述
**面向对象程序设计**的核心思想是数据抽象、继承和动态绑定。
**继承**
通过**继承**联系在一起的类构成一种层次关系。通常在层次关系的根部有一个基类,其他类则直接或间接地从基类即成而来,这些继承得到的类称为**派生类**。
对于某些函数,基类希望它的派生类个自定义适合自身的版本,此时基类就将这些函数声明成**虚函数**。
```c++
class Quote {
public:
    std::string isbn() const;
    virtual double net_price(std::size_t n) const;
}
```
派生类必须通过使用类派生列表明确指出他是从哪个基类继承而来的。
```c++
class Bulk_quote : public Quote {
public:
    double net_price(std::size_t n) const;
}
```
**动态绑定**
通过使用**动态绑定**,我们能用同一段代码分别处理Quote和Bulk_quote的对象。
```c++
//计算并打印销售给定数量的某种书籍所得的费用
double print_total(ostream &os,
                    const Quote &item, size_t n)
{
    //根据传入item形参的对象类型调用Quote::net_price
    //或者Bulk_quote::net_price
    double ret = item.net_price();
    os << "ISBN： " << item.isbn()      //调用Quote::isbn
       << " # sold: " << n << " total due:" << res << endl;
    return ret;
}
```

## 15.2 定义基类和派生类

### 定义基类
```c++
class Quote {
public: 
    Quote() = default;
    Quote(const std::string &book, double sales_price):
                    bookNo(book), price(sales_price){ }
    std::string isbn() const { return bookNo; }
    //返回给定数量的书籍的销售总额
    //派生类负责改写并使用不同的折扣计算方法
    virtual double net_price(std::size_t n) const
                        {return n * price; }
    virtual ~Quote() = default;     //对析构函数进行动态绑定
private:
    std::string bookNo;
protected:
    double price = 0.0;             //代表普通状态下打折的价格
};
```
### 定义派生类
派生类必须通过使用**类派生列表**明确指出它是从哪个基类继承而来的。
```c++
class Bulk_quote() : public Quote {
public:
    Bulk_quote() = default;
    Bulk_quote(const std::string&, double, std::size_t, double);
    //覆盖基类的函数版本以实现基于大量购买的折扣力度
    double net_price(std::size_t) const override;
private:
    std::size_t min_qty = 0;        //适用折扣政策的最低购买量
    double discount = 0.0;          //以小数表示折扣额
}
```
**派生类中的虚函数**
派生类经常覆盖它继承的虚函数。如果派生类没有覆盖其基类中的某个虚函数,则该虚函数的行为类似于其他的普通成员,派生类会直接继承其所在的基类中的版本。
**派生类对象及派生类向基类类型转换**
因为在派生类对象中含有与其基类对应的组成部分,所以我们能把派生类的对象当成基类对象来使用而且我们也能将基类的指针或引用绑定到派生类对象中的基类部分上。
```c++
Quote item;         //基类对象
Bulk_quote bulk;    //派生类对象
Quote *p = &item;   //p指向Quote对象
p = &bulk;          //p指向bulk的Quote部分
Quote &r = bulk;    //r绑定到bulk的Quote部分  
```
这种转换通常称为**派生类到基类的**类型转换。
==在派生类对象中含有与其基类对应的组成部分,这一事实是继承的关键所在==
**派生类构造函数**
```c++
Bulk_quote(const std::string& book, double p,
            std::size_t qty, double disc) :
            Quote(book, p), min_qty(qty), discount(disc){ }
```
**派生类使用基类成员**
```c++
double Bulk_quote::net_price(size_t cnt) const
{
    if  (cnt >= min_qty)
    {
        return cnt * (1 - discount) * price;
    }
    else
    {
        return cnt * price;
    }
}
```
**继承静态类成员**
```c++
class Base {
public:
    static void statmem();
};
class Derived : public Base {
    void f(const Derived&);
};

void Deriverd::f(const Deriverd &deriverd_obj)
{
    Base::statmem();        //正确:Base定义了statmem
    Derived::statmem();     //正确:Derived继承了statmem
    //正确:派生类的对象能访问基类的静态成员
    derived_obj.statmem();  //通过Derived对象访问
    statmem();              //通过this对象访问
}
```
**防止继承的发生**
```c++
class NoDerived final { /* */ };    //NoDerived不能作为基类
class Base { /* */ };
// Last是final的;我们不能即成Last
class Last final : Base { /* */ };  //Last不能作为基类
class Bad : NoDerived { /* */ };    //错误:NoDrived是final的
class Bad2 : Last { /* */ };        //错误:Last是final的
```
==**存在继承关系的类型之间的转换规则**
1.从派生类向基类的类型转换只对指针或引用类型有效
2.基类和派生类不存在隐式类型转换
3.和任何其他成员一样,派生类向基类的类型转换也可能会由于访问受限而变得不可行。==

## 15.3 虚函数
**对虚函数的调用可能在运行时才被解析**
当某个虚函数通过指针或引用调用时,编译器产生的代码直到运行时才能确定应调用哪个版本的函数。被调用函数是与绑定到指针或引用上的对象的动态类型相匹配的那一个
**派生类中的虚函数**
==基类中的虚函数在派生类中隐含地也是一个虚函数。当派生类覆盖了某个虚函数时,该函数在基类中的形参必须与派生类中的形参严格匹配==
**final和override说明符**
```c++
struct B {
    virtual void fi(int) const;
    virtual void f2();
    void f3();
};
struct D1 : B {
    void f1(int) const override;        //正确:f1与基类中的f1匹配
    void f2(int) override;              //错误:B没有形如f2(int)的函数
    void f3() override;                 //错误:f3不是虚函数
    void f4() override;                 //错误:B没有名为f4的函数
};
```
我们还能把某个指定函数定为final,如果我们已经把函数定义成final了,则之后任何尝试覆盖该函数的操作都将引发错误
```c++
struct D2 : B {
    //从B继承f2()和f3(),覆盖f1(int)
    void f1(int) const final;   //不允许后续的其他覆盖f1(int)
};
struct D3 : D2 {
    void f2();                  //正确:覆盖从间接基类的B继承而来的f2
    void f1(int) const;         //错误:D2已经将f2声明成final
};
```
## 15.4抽象基类
**纯虚函数**
```c++
//用于保存折扣和购买量的类,派生类使用这些数据可以实现不同的价格策略
class Disk_quote : public Quote {
public:
    Disk_quote() = default;
    Disk_quote(const std::string& book, double price,
                std::size_t qty, double disc):
        Quote(book, price),
        quantity(qty), discount(disc)  {}

    double net_price(std::size_t) const = 0;
protected:
    std::size_t quantity = 0;       //折扣适用的购买量
    double discount = 0.0;          //表示折扣的小数值
};
```
**含有纯虚函数的类是抽象基类**
含有纯虚函数的类是**抽象基类**。抽象基类负责定义接口,而后续的其他类可以覆盖接口.我们不能创建一个抽象基类对象。
```c++
//Disc_quote声明了纯虚函数,而Bulk_quote将覆盖该函数
Disc_quote discounted;      //错误:不能定义Disc_quote的对象
Bulk_quote bulk;            //正确:Bulk_quote中没有纯虚函数
```
==我们不能创建基类的对象==

## 15.5访问控制与继承
**收保护的成员**
- 和私有成员类似,受保护的成员对于类的用户来说是不可访问的
- 和公有成员类似,受保护的成员对于派生类的成员和友元来说是可访问的
- 派生类的成员或友元只能通过派生类的对象来访问基类的受保护成员。派生类对于一个基类对象中的受保护成员没有任何访问特权

**公有、私有和受保护继承**
```c++
class Base {
public:
    void pub_mem();
protected:
    int prot_mem();
private:
    int priv_mem();
};
struct Pub_Derv : public Base {
    //正确:派生类能i访问protected成员
    int f() { return prot_mem ;}
    //错误:private成员对于派生类来说是不可访问的
    char g() {return priv_men ;}
};
struct Priv_Derv : private Base {
    // private不影响派生类的访问权限
    int f1 const { return prot_mem; }
};
```
**友元和继承**
```c++
class Base {
    //添加friend声明,其他成员与之前的版本一致
    friend class Pal;       //Pal在访问Base的派生类时不具有特殊性
};
class Pal {
public:
    int f(Base b) {return b.prot_mem; }     //正确:Pal是Base的友元
    int f2(Sneaky s) {return s.j; }         //错误:Pal不是Sneaky的友元
    //对基类的访问权限由基类本身控制,即使对于派生类的基类部分也是如此
    int f3(Sneaky s) { return s.prot_mem; } //正确:Pal是Base的友元
}
```
==不能继承友元关系;每个类负责控制各自成员的访问权限==
**改变个别成员的可访问性**
```c++
class Base {
public:
    std::size_t size() const { return n; }
protected:
    ste::size_t n;
};
class Derived : private Base {      //注意:private继承
public:
    //保持对象尺寸相关的成员访问级别
    using Base::size;
protected:
    using Base::n;
}
```
**默认的继承保护级别**
```c++
class Base { /*...*/ };
struct D1 : Base { /*...*/ };       //默认public继承
class D2 : Base { /*...*/ };        //默认private继承
```

## 15.6继承中的作用域
**名字冲突与继承**
```c++
struct Base {
    Base(): mem(0) { }
protected:
    int mem;
};
struct Derived : Base {
    Derived(int i): mem(i) { }      //用i初始化Derived::mem
                                    //Base::mem进行默认初始化
    int get_mem() { return mem; }   //返回Derived::mem
protected:
    int mem;                        //隐藏基类中的mem
};
```
**虚函数与作用域**
```c++
class Base {
public:
    virtual int fun();
};
class D1 : public Base {
public:
    //隐藏基类的fcn,这个fcn不是虚函数
    //D1继承了Base::fcn()的定义
    int fcn(int);       //形参列表与Base中的fcn不一致
    virtual void f2();  //是一个新的虚函数,在Base中不存在
};
class D2 : public D1 {
public:
    int fcn(int);       //是一个非虚函数,隐藏了D1::fcn(int)
    int fcn();          //覆盖了Base的虚函数fcn
    void f2();          //覆盖了D1的虚函数f2
};
```
## 15.7构造函数与拷贝控制
和其他类一样,位于继承体系中的类也需要控制当其对象执行一系列操作时发生什么样的行为,这些操作包括创建、拷贝、移动、赋值和销毁。