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
###Item 13: Use objects to manage resources.

-   获得资源后立即放进资源对象内。`RAII`--Resource Acquisition Is initialization
-   管理对象运用析构函数确保资源被释放。
-   auto_ptr通过copy构造函数或copy assignment操作符复制它们，它们会变成null，而复制所得到的指针将取得资源的唯一拥有权。
-   动态分配得到的`array`身上使用auto_ptr或tr1::shared_ptr是个馊主意。两者再析构函数内做delete而不是delete[]动作。

###Item 14: Think carefully about copying behavior in resource-managing classes

当一个RAII对象被复制，考虑两种可能性：

-	禁止复制
-	对底层资源采用`引用计数法`， 
	-	`shared_ptr`的缺省行为是“当引用次数为0时删除其所指之物”，允许制定特定的删除器(`deleter`)
-	复制底部资源
-	转移底部资源的拥有权

###Item 15: Provide access to raw resources in resource-managing classes

智能指针重载了指针取值操作符(operator-> 和 operator*)，它们允许隐式转换至底部原始指针。

隐式转换举例：

	class Font {
	public:
		...
		operator FontHandle() const //隐式转换函数
		{ return f; }
		...
	}

-	API往往要求访问原始资源，所以每个RAII class应该提供一个取得原始资源的方法。
-	对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐私转换对客户比较方便。

###Item 16: Use the same form in corresponding uses of new and delete

new对应delete，new[] 对应delete[]

对于typedef,必须要在程序中说明清楚
	
	typedef std::string AddressLines[4];
	std::string * pal = new AddressLines;
	delete [] pal; //delete pal 导致行为未有定义
因此，最好尽量不要对数组形式做typedef动作。

###Item 17: Store newed objects in smart pointers in standalone statements.

假设有个函数解释处理程序的优先权，另一个函数用来在某动态分配所得的Widget上进行某些带有优先权的处理：

	int priority();
	void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);

如果这样调用
	
	processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
编译器创建代码，做以下三件事：

- 调用priority
- 执行new Widget
- 调用tr1::shared_ptr构造函数。

只能保证new Widget在shared_ptr构造函数之前被调用。如果priority在两者中间被调用，而且导致异常。那么new Widget返回的指针将会遗失。

##设计与声明
###Item 18: Make interfaces easy to use correctly and hard to use incorrectly

- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
- “促进正确使用”的办法包括借口的一致性，以及与内置类型的行为兼容。任何接口如果要求客户必须记得做某些事情，就是有着“不正确使用”的倾向，因为客户可能会忘记做那件事。
- “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
- tr1::shared_ptr支持定制性删除器(custom deleter)。可以防范`cross-DLL problem`(在一个DLL中被new创建，却在另一个DLL中被delete销毁)，自动解除互斥锁等等。

###Item 20: Prefer pass-by-reference-to-const to pass-by-value.

- 尽量以`pass-by-reference-to-const`替换`pass-by-value`。前者通常比较高效，并可以避免切割问题。
- 以上规则并不适用于内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适当。

###Item 21: Don't try to return a reference when you must return an object

绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。

###Item 22: Declare data members private.

- 封装的重要性比你最初见到它时还要重要
- 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允诺约束条件获得保证，并提供class作者以充分的实现弹性。
- protected并不比public更具有封装性。

###Item 23: Prefer non-member non-friend functions to member functions

- 提供更大的封装性
- 比较自然的做法是让 non-member non-friend函数作为便利函数位于类所在的同一个namespace内。并将同类便利函数声明放在同一头文件中。
- 将所有便利函数放在多个头文件内但隶属于同一个命名空间，意味客户可以轻松扩展这一组便利函数。

###Item 24: Declare non-member functions when type conversions should apply to all parameters

	class Rational {
	public:
		const Rational operator* (const Rational &rhs) const;
	};

	Rational oneHalf(1, 2);
	result = oneHalf * 2; // oneHalf.operator*(2)，可以执行
	result = 2 * oneHalf; // 2.operator*(oneHalf)，错误

所以让函数称为一个`non-member`函数

	const Rational operator* (const Rational &lhs， const Rational &rhs);	
允许编译器在每一个实参身上执行隐私转换类型。

###Item 25: Consider support for a non-throwing swap

	namespace std {
		template<> //全特化
		void swap<Widget>( Widget &a, Widget &b) {
			swap(a.pImpl, b.pImpl); //私有变量，编译不通过，需要创建类的成员函数
		} }

	class Widget {
	public:
		void swap(Widget& other) {
			using std::swap; //声明是有必要的，下个swap调用std版本的
			swap(pImpl, other.pImpl);

- 当`std::swap`对你的类型效率不高时，提供一个`swap`成员函数，并确定这个函数不抛出异常
- 如果你提供一个`member swap`，也应该提供一个`non-member swap`用来调用前者。对于classes（而非templates），也请特化`std::swap`
- 调用`swap`时应针对`std::swap`使用`using`声明式，然后调用`swap`并且不带任何“命名空间资格修饰”。
- 为“用户定义类型”进行`std templates`全特化是好的，但千万不要尝试在`std`内加入某些对`std`而言是全新的东西

##实现
###Item 26: Postpone variable definitions as long as possible

不止应该延后变量的定义，这道非得使用该变量的前一刻为止，甚至应该尝试延后这份定义知道能够给它初值实参为止。

###Item 27: Minimize casting

- `const_cast`被用来将对象的常量性转除，也是唯一有此能力的c++ style转型操作符
- `dynamic_cast`主要用来执行“safe downcasting”，可能需要耗费重大运行成本。
- `reinterpret_cast`执行低级转型，实际动作及结果可能取决于编译器，也就表示它不可移植。
- `static_cast`用来强迫隐式转换。例如将non-const转为const对象，将int转为double。

单一对象(例如一个类型为Derived的对象)`可能拥有一个以上的地址`(例如“以Base* 指向它”时的地址和“以Derived* 指向它”时的地址)。这至少意味着你通常应该避免作出“对象在C++中如何布局”的假设。

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免`dynamic_cast`。如果有个设计需要转向动作，试着发展无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需将转型放进他们自己的代码内。
- 宁可使用C++ style转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分门别类的职掌。

###Item 28: Avoid returning "handles" to object internals

避免返回handles(包括references、指针、迭代器)指向对象内部。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生dangling handles的可能性降至最低。

考虑一个例子：
	
	const Point& Rectangle::upperLeft() const { return pData->ulhc; }	

	class GUIObject{};
	const Rectangle boundingBox(const GUIObject &obj);

	GUIObject *pgo;
	const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft()); //指向一个不存在的对象

###Item 29: Strive for exception-safe code.

- 异常安全函数即使发生异常也不会泄露资源或允许任何数据结构败坏。这样的函数区分三种可能的保证：基本型、强烈型、不抛异常型。
- 强烈保证往往能够以copy-and-swap实现出来，但强烈保证并非对所有函数都可实现或具备现实意义。
- 函数提供的异常安全保证通常最高只等于其所调用之各个函数的异常安全保证中的最弱者。

###Item 30: Understand the ins and outs of inlining
