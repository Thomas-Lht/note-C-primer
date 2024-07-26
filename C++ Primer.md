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
