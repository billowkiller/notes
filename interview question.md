### 怎么理解面向对象编程 ###

### 线程、进程的区别 ###

线程是指进程内的一个执行单元,也是进程内的可调度实体.

与进程的区别

1. 地址空间:进程内的一个执行单元;进程至少有一个线程;它们共享进程的地址空间;而进程有自己独立的地址空间;
2. 资源拥有:进程是资源分配和拥有的单位,同一个进程内的线程共享进程的资源
1. 线程是处理器调度的基本单位,但进程不是.
1. 二者均可并发执行.

进程和线程都是由操作系统所体会的程序运行的基本单元，系统利用该基本单元实现系统对应用的并发性。进程和线程的区别在于：

简而言之,一个程序至少有一个进程,一个进程至少有一个线程. 线程的划分尺度小于进程，使得多线程程序的并发性高。 

另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。 

线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。 
从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位. 

线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源. 

一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.
### static在C和C++里各代表什么含义 ###

- 在**C语言**中，static可以用来修饰局部变量，全局变量以及函数
	1. 一般情况下，对于局部变量是存放在栈区的，并且局部变量的生命周期在该语句块执行结束时便结束了。但是如果用static进行修饰的话，该变量便存放在静态数据区，其生命周期一直持续到整个程序执行结束。但是在这里要注意的是，虽然用static对局部变量进行修饰过后，其生命周期以及存储空间发生了变化，但是其作用域并没有改变，其仍然是一个局部变量，作用域仅限于该语句块。在用static修饰局部变量后，该变量只在初次运行时进行初始化工作，且只进行一次。
	2. static对全局变量进行修饰改变了其作用域的范围，由原来的整个工程可见变为本源文件可见。
	3. 用static修饰函数的话，情况与修饰全局变量大同小异，就是改变了函数的作用域。

- 在**C++中**static还具有其它功能，如果在C++中对类中的某个函数用static进行修饰，则表示该函数属于一个类而不是属于此类的任何特定对象；如果对类中的某个变量进行static修饰，表示该变量为类以及其所有的对象所有。它们在存储空间中都只存在一个副本。可以通过类和对象去调用。

### const在C/C++里什么意思 ###
[http://blog.csdn.net/Eric_Jo/article/details/4138548](http://blog.csdn.net/Eric_Jo/article/details/4138548)

### select/Poll模型 ### [http://blog.csdn.net/tianmohust/article/details/6677985](http://blog.csdn.net/tianmohust/article/details/6677985)

### std::move和std::forward ###

**左值和右值**概念的本质区别就是，左值是用户显示声明或分配内存的变量，能够直接用变量名访问，而右值主要是临时变量。

**std::move**是一个用于提示优化的函数，过去的c++98中，由于无法将作为右值的临时变量从左值当中区别出来，所以程序运行时有大量临时变量白白的创建后又立刻销毁，其中又尤其是返回字符串std::string的函数存在最大的浪费。因为并不是所有情况下，C++编译器都能进行返回值优化，所以在C++11中，编码者可以主动提示编译器，返回的对象是临时的，可以被挪作他用. 

	std::string fileContent = “oldContent”;
	s = std::move(readFileContent(fileName));
	
对象s在被赋值的时候，方法std::string::operator =(std::string&&)会被调用，符号&&告诉std::string类的编写者，传入的参数是一个临时对象，可以挪用其数据，于是std::string::operator =(std::string&&)的实现代码中，会置空形参，同时将原本保存在中形参中的数据移动到自身。

**std::forward**是用于模板编程中的.当一个临时变量传入形参为T&& t的模板函数时，T被推演为U，参数t所引用的临时变量因为开始能够被据名访问了，所以它变成了左值。这也就是std::forward存在的原因！当你以为实参是右值所以t也应该是右值时，它跟你开了个玩笑，它是左值！如果你要进一步调用的函数会根据左右值引用性来进行不同操作，那么你在将t传给其他函数时，应该先用std::forward恢复t的本来引用性，恢复的依据是模板参数T的推演结果。虽然t的右值引用行会退化，变成左值引用，但根据实参的左右引用性不同，T会被分别推演为U&和U.

使用std::forward的原因：由于声明为f(T&& t)的模板函数的形参t会失去右值引用性质，所以在将t传给更深层函数前，可能会需要回复t的正确引用行，当然，修改t的引用性办不到，但根据t返回另一个引用还是可以的。

### 随机算法 ###

- Las Vegas算法：这个算法总是能产生正确的结果，否则就不会返回结果。

- Monte Carlo算法：
总是会返回结果，但是结果不一定是正确的。但是出现不正确的答案的几率可以非常非常低，而且我们可以多次运行，每次做不同的随机选择，那么几率就更加低了。


随机算法的效率：比如一般算法的效率是O(n)，那么随机算法可以是O(1)，我们只选择一个实例来验证就可以了，比如前面的列子。当然我们也可以根据我们的需要选择多个实例m。那么效率就是O(m)。这个m要比n少很多。那么效率就高很多了。

### linux IPC ###

[http://www.cnblogs.com/itech/archive/2010/06/29/1767056.html](http://www.cnblogs.com/itech/archive/2010/06/29/1767056.html)