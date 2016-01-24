###std::string 定义

STL中的字符串类string的定义如下：

	template<typename _CharT, typename _Traits , typename _Alloc> class basic_string;
	typedef basic_string <char, char_traits<char >, allocator< char> > string;

不难发现string在栈内存空间上只占用一个指针(_CharT* _M_p)的大小空间，因此**sizeof(string)==4**。其他信息都存储在**堆内存空间**上。

”string name;”这个name对象所占用的总空间为33个字节，具体如下：

	sizeof(std::string) + 0 + sizeof('') + sizeof(std::string::_Rep)

其中：sizeof(std::string)为栈空间

![string内存空间布局](http://blogs.360.cn/360cloud/files/2012/11/string-1.png)

###std::string的copy

string copy是基于引用计数和copy_on_write技术，所以每次copy的代价非常小。等到要写入的时候才copy。

![赋值后的内存布局](http://blogs.360.cn/360cloud/files/2012/11/string-2-300x278.png)

对string的修改过程分为以下两步：

1. 判断是否为共享对象，(引用计数大于0)，如果是共享对象，就拷贝一份新的数据，同时将老数据的引用计数值减1。
1. 在新的地址空间上进行修改，从而避免了对其他对象的数据污染

对上图的c修改后，内存布局如下：

![](http://blogs.360.cn/360cloud/files/2012/11/string-3-298x300.png)

由此可以看出，如果不是通过string提供的接口对string对象强制修改的话，会带来潜在的不安全性和破坏性。
例如：

	char* p = const_cast<char*>(s1.data());
	p[0] = 'a';

这段代码通过强制修改c对象内部数据的值，看似效率上比operator[] 高，但同时也修改a、b对象的值，而这可能不是我们所希望看到的。这是我们需要提高警惕的地方。

###vc6.0的string

vc6.0的string实现是基于引用计数的，但不是线程安全的。但在后续版本的vc中去掉了引用计数技术，string copy 都直接进行深度内存拷贝。
由于string实现上的细节不一致，导致跨平台程序的移植带来潜在的风险。这种场合下，我们需要额外注意。

###总结
即使是一个空string对象，其所占内存空间也达到33字节，因此在内存使用要求比较严格的应用场景，例如memcached等，请慎重考虑使用string。
string由于使用引用计数和Copy-On-Write技术，相对于strcpy，string copy的性能提升非常显著。
使用引用计数后，多个string指向同一块内存区域，因此，如果强制修改一个string的内容，会影响其他string。