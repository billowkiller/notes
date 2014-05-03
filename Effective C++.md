##让自己习惯C++
###Item 01： View C++ as a federation of languages
C++同时支持过程形式、面对对象形式、函数形式、泛型形式、元编程形式。

次语言有

*	C
*	Object-Oriented C++
*	Template C++
*	STL

C++高效编程守则视状况而变化，取决于你使用C++的那一部分。

###Item 02： Prefer consts, enums and inlines to #defines

- `#define`的变量没有进入记号表。并且没有作用域，不能提供任何封装性。
- `#define`定义宏，需要为所有实参加上小括号，且不能够使用`++`和`--`。

`#define`难以调试、行为无法预料、类型不安全。

	class GamePlayer{
	private:
		static const int NumTurns = 5; //常量申明式
	};

通常C++要求你对你所使用的任何东西提供一个定义式，但那如果它是`class`专属常量又是`static`且为整数型(ints,chars,bools)则需特殊处理。只要是不取它们的地址，你可以申明并使用它们而无需提供定义式。如果需要取址，必须提供定义式：
	
	const int GamePlayer::NumTurns; //申明时获取了初值，定义不必赋值

"The enum hack"表示一个属于枚举类型的数值可权充int被使用，于是`GamePlayer`定义为

	class GamePlayer{
	private:
		enum { NumTurns = 5 };
		static const int NumTurns = 5;
	};


- 取一个`enum`地址是非法的。`enum`和`#define`一样不会导致非必要的内存分配。
- `enum hack`是`template metaprogramming`的基础技术。

`template inline`可以提供宏带来的效率以及一般函数的所有可预料行为和类型安全。遵守作用域和访问规则。

###Item 03： Use const whenever possible

令函数返回一个常量值，往往可以降低因客户错误而造成的意外，而不至于放弃安全性和高效性。

	const Rational operator* (const Rational& lhs, const Rational& rhs);
	Rational a, b, c;
	...
	(a * b) = c;  //错误

重载`operator[]`并对不同的版本给予不同的返回类型，就可以令`const`和`non-const`获得不同的处理。返回 `char&`也是必要的。

	const char& operator[](std::size_t position) const; //operator[] for const Object
	char& operator[](std::size_t position); //operator[] for non-const object



- 成员函数式`const`有两个流行的概念：`bitwise constness`, `logical constness`。
- `mutable`变量成员可以再const成员函数中改变。
- `static_cast`将`non-const`对象转为`const`对象。`const_cast`相反。


###Item 04： Make sure that objects are initialized before they're used

- `C++`对定义与不同编译单元(文件)内的`non-local static`对象的初始化次序并无明确定义。
- 函数内的`local static`对象会在该函数被调用期间首次遇上该对象的定义式时被初始化。

为内置对象进行手工初始化，`C++`不保证初始化它们。
构造函数使用`成员初值列`，不要在函数内使用赋值操作。其排列次序应该和申明次序相同。
为免除跨编译单元初始化次序问题。以`local static`对象替代`non-local static`对象。


##Constructors, destructors, and Assignment Operators
###Item 05： Know what functions C++ silently writes and calls

- `default`构造函数和析构函数调用`base classes`和`none-static`成员变量的构造函数和析构函数。且只有base是`virtual`析构时，它才是`virtual`的。
- 如果类内含`reference`或`const`成员，或者`base classes`将`copy assignment`操作符申明为`private`，则需要自己定义`copy assignment`

###Item 06: Explicitly disallow the use of compiler-generated functions you do not want

不实现`copy`或`copy assignment`的方法：

1. 将成员函数声明为private而且故意不实现它们。
2. 设计一个专门为了组织copying动作的`base class`， 将`copy`和`copy assigment`声明为`private`。接着私有继承base class
3. `Boost`提供的class，`nonecopyable`

###Item 07: Declare destructors virtual in ploymorphic base classes

	Base *pt = new Derived;
	delete pt;

`derived class`对象经由一个`base class`指针被删除，如果`base class`有个k析构函数，则对象的`derived`成分没被销毁。

无端地将所有classes的析构函数声明为`virtual`，就像从未声明它们为`virtual`一样，都是错误的，带来对象体积的增加。只有当class内含有至少一个`virtual`函数才为它声明`virtual`析构函数。

为你希望它成为抽象的那个class(polymorphic base classes)声明一个`pure virtual`析构函数。

	class AWOV {
	public:
		virtual  ~AWOV() = 0;
	};
	AWOV::~AWOV() {}  //pure virtual 析构函数的定义 

然而必须为这个`pure virtual`函数提供一份定义，根据析构函数的运作方式，编译器会在AWOV的`derived classes`的析构函数中创建一个对~AWOV的调用动作，所以必须为这个函数提供一份定义。否则，连接器会发出抱怨。

###Item 08: Prevent exceptions from leaving destructors

- 析构函数绝对不要突出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
- 如果客户需要对某个操作函数运行期间抛出的异常作出反应，那么class应该提供一个普通函数（而非在析构函数中）执行该操作。

###Item 09: Never call virtual functions during construction or destruction

在base class构造期间，virtual函数不是virtual函数。因为derived class对象还未构造好，所以base class构造期间virtual函数绝不会下降到derived classes阶层。在derived class对象的base class构造期间，对象类型是base class而不是derived class。

在构造期间，可以借由“令derived classes将必要的构造信息向上传递至base class构造函数”替换之。

###Item 10: Have assignment operators return a reference to *this

	Widget& operator=(const widget &rhs) {
		...
		return *this;
	}

###Item 11: Handle assignment to self in operator=

容易掉进“在停止使用资源之前意外释放了它”的陷阱。

让`operator=`具备“异常安全性”往往自动获得“自我复制安全”的汇报。
	
	Widget& operator=(const widget &rhs) {
		Bitmap * pOrig = pb;
		pb = new Bitmap(*rhs.pb);
		delete pOrig;
		return *this;
	}

还可以使用`copy and swap`技术。或利用一下事实：(1)某class的`copy assignment`操作符可能被声明`by value`的方式；(2)以`by value`的方式传递东西会造成一份副本。

###Item 12: Copy all parts of an object

- 编写一个copying函数确保(1)复制所有local成员变量，(2)调用所有base classes内的适当的copy函数。
- 不要尝试以某个copying函数实现另一个copying函数。应该讲共同机能放进第三个函数中，并有两个copying函数共同调用。

##资源管理
###Use objects to manage resources.

-   获得资源后立即放进资源对象内。RAII--Resource Acquisition Is initialization
-   管理对象运用析构寒素确保资源被释放。
-   auto_ptr通过copy构造函数或copy assignment操作符复制它们，它们会变成null，而复制所得到的指针将取得资源的唯一拥有权。
-   动态分配得到的array身上使用auto_ptr或tr1::shared_ptr是个馊主意。两者再析构函数内做delete而不是delete[]动作。

###Think carefully about copying behavior in resource-managing classes
