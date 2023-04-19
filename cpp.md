## 1. 指针常量和常量指针

**1.指针常量：**不能修改指针所指向的地址。定义同时必须初始化。

```
int a = 10;
int* const p = &a;	//指针常量
*p = 1;	//正确
p = &b;	//错误 指针指向的地址不能改变
```

**2.常量指针：**不能修改指针所指向地址的**内容**。但可以改变指针指向的地址。

```
const int* p = &a;	//常量指针
int const* p;	//同上
*p = 1;		//错误，指针指向的值不能改变
p = &b;		//正确
```

**3.指向常量的常指针常量:** 二者均不可修改。

```
const int a = 10;
const int* const p = &a;
```

**4.类中使用const：** **常成员函数**，只能调用常成员函数和常成员数据。

```
class A{
	public:
	A(int x) : num(x), b(x){}
	int getNum() const;	//const修饰不改变成员变量的函数
	const A operator*(const A& left, const A& right);	//const修饰成员函数返回值
	const getConstNum() const;	//二者结合
}
//常成员函数
//1.不能修改类的任何非静态数据成员
//2.不能调用类的任何非const函数
//3.const可以用于对重载函数的区分
//ps:类中const成员变量必须通过初始化列表进行初始化
```

**5.小结：**

· 如果const位于*右侧，则修饰指针本身，为指针常量。

· 如果const位于*左侧，则修饰指针所指向的变量，为常量指针。

· 可以将常量指针指向非const数据，但不能用常量指针修改该数据的内容。

· 对于变量来说值传递和引用传递效率差距不大，但对象引用传递效率更高。



## 2. const

**const永远先修饰左边的符号，若左边没有则修饰右边的。**

**1.const的引用：**对常量的引用不能被用作修改它所绑定的对象

```
const int ci = 1024;
const int &r1 = ci;
r1 = 42;		//错误 r1是对常量的引用
int &r2 = ci;	//错误 试图让一个非常量的引用指向一个常量对象
const &r3 = 12;	//正确(表达式作为初始值)

int i = 42;
int &r1 = i;
const int &r2 = i;	//正确 const引用 引用一个非const对象，r1可修改i的值，r2不能
```

注意：引用并不是一个对象，”常量引用“是对const的引用

**允许用任意表达式作为初始值，**只要结果能转换成引用的类型即可（见上r3）

```
double dval = 3.14;
const int &r1 = dval;	//正确 编译器会产生如下代码

const int temp = dval;	//生成一个临时的整型常量
const int &r1 = templ	//r1绑定这个临时量
```

 **2.指针和const：**常对象地址必须要用常指针存，而常指针未必要指向常对象。

见1.指针常量和常量指针

**3.顶层const和底层const：** **顶层const**表示指针本身是个常量（指针常量）

​												 **底层const**表示指针指向的对象是一个常量（常量指针）

**4.constexpr和常量表达式：**值不会改变并且在编译过程就能得到计算结果的表达式

```
const int max_files = 20;	//max_files是常量表达式
int staff_size = 27;		//不是 非const
const int sz = get_size();	//不是 具体值知道运行时才得到
# 当确定了变量是一个常量表达式，就将其生成为constexpr类型
constexpr int limit = max_files+1;
```

4.1 一个constexpr指针的初始值必须是nullptr或者0，或者存于某个固定地址中的对象。

4.2 constexpr中定义了一个指针，则只对指针有效，和其所指的对象无关。

```
const int *p = nullptr;		//指向整形常量的指针
constexpr int *q = nullptr;	//指向整数的常量指针
```

## 3. 参数传递

**1.传值参数：**实参的值被拷贝给形参，二者为相互独立的对象。

​	**1.1指针形参**

```
void reset(int *ip){
	*ip = 0;	//改变ip所指对象的值
	ip = 0;		//只改变了ip的局部拷贝，实参未被改变
}

int i = 42;
reset(&i);		// i = 0
```

**2.传引用参数：**形参是引用类型，或者函数被传引用调用

```
void reset(int &i){
	i = 0;	//i只是j的一个别名
}

int j = 42;
reset(j);		//j = 0
```

​	**2.1使用引用避免拷贝：**拷贝大的类类型对象或容器对象

```
//形参无需修改时，最后声明为常量引用
bool isShorter(const string &s1, const string &s2){
	return s1.size() < s2.size();
}
```

**3.const形参和实参：**实参初始化形参时会忽略顶层const

```
void fun(const int i) { /*能读取i 但不能向i写值*/}
void fun(int i) 	  { /*重复定义了fun(int)*/}
```

注意：尽量使用常量引用，不能把const对象、字面值、需要类型转换的对象传递给普通的引用形参。

**4.数组形参：**

​	4.1 显式传递一个表示数组大小的形参

```
//const int a[] 等价于 const int* a
void print(const int a[], size_t size){
	for(size_t i = 0;i != size; ++i){
		cout << a[i] << endl;
	}
}

int j[] = {0, 1};
print(j, end(j) - begin(j));
```

​	4.2 数组引用形参

```
//arr是具有10个整数的整型数组的引用
void print(int (&arr)[10]){
	for(auto elem : arr)
		cout << elem << endl;
}
```

**5.多维数组传递：**

```
1.给定二维数组长度
void print(int a[][4])
2.给定二维长度的指针
void print(int (*p)[4])
3.二维数组指针转为一维数组指针
void print(int *p)
4.二级指针
void print(int **p){
	cout << *((int*)p + i * 4 + j) << endl;
}
```

**6.含有可变性惨的函数：**所有形参类型相同，可传递initializer_list，类似vector，但其内部值均为常量。

```
void error_msg(initializer_list<string> il){
	for(auto beg = il.begin(); beg != il.end(); ++beg)
		cout << *beg << " ";
	cout << endl;
}

error_msg({"functionX", expected, actual});		//调用时参数要用{}括起
```

**7.返回数组指针:**

```
typedef int arrT[10];
arrT* fun(int i);	//返回一个指向含有十个整数的数组的指针

若不使用别名
int (*func(int i))[10];
//1.func(int i) 需要一个int类型参数
//2.(*func(int i)) 可以对函数调用的结果解引用
//3.(*func(int i))[10] 解引用的结果是一个大小是10的数组
//4.int (*func(int i))[10] 数组中的元素是int类型

c++11中可用尾置返回类型
auto func(int i) -> int(*)[10];
```

## 4.函数指针

**1.函数指针指向的是函数而非对象。**

```
bool (*pf)(const string&, const string&);	//pf是一个指向函数的指针
bool compareLength(const string& s1, const string& s2) {
	return s1.size() > s2.size() ? 1 : 0;
}
//则可以使用pf指向compareLength函数
pf = compareLength;
bool b1 = pf("abc", "ac");	//通过pf调用函数compareLength
```

**2.指向不同函数类型的指针不存在转换规则**，因此使用指向重载函数的指针必须明确精确匹配函数类型。

```
void ff(int*);
void ff(unsigned int);

void (*pf1)(unsigned int) = ff;	//pf1 指向 ff(unsigned)
void (*pf2)(int) = ff;			//错误：没有任何一个ff与形参列表匹配
double (*pf3)(int*) = ff;			//错误：ff和pf3返回值不相同
```

**3.函数指针形参**

```
void useBigger(const string& s1, const string& s2, bool (*pf)(const string&, const string&));

useBigger("abc", "bc", comparelength);	//自动将函数compare转换成指向该函数的指针
可以使用decltype简化生命
typedef decltype(lengthCompare) Fun;	//Fun是函数类型
useBigger("abc", "ab", Fun);
```



## 5.容器

**5.1：容器操作可能使迭代器失效**

​	当添加元素时，如果容器是vector或string，且存储空间被重新分配，则只向容器的迭代器、指针和引用都会失效。

​	删除元素时，尾后迭代器总是会失效

```
auto begin = v.begin(), end = v.end();	//不要保存end返回的迭代器！
//程序会死循环，添加元素时导致end中的迭代器失效
while(begin != end){
	++begin;
	begin = v.insert(begin, 42);	
	++begin;
}
```

**5.2访问成员函数返回的是引用**

```
if (!c.empty()){
	c.front() = 42;
	auto &v = c.back();	
	v = 1024;			//会改变c中的值
	auto v2 = c.back();	//auto不会隐式的指明值是否为引用，需要加&
	c = 0;				//不会改变c的值
}
```

**5.3容器元素是拷贝：**将一个对象插入到容器中，实际上插入的是对象值的一个拷贝，而非对象本身。改变容器中的元素与实际对象之间没有任何关联，反之亦然。



## 6.泛型算法

**6.1** 泛型算法不会执行容器的操作，只会运行于迭代器之上。同样迭代器令算法不依赖于容器，但算法依赖于元素类型的操作。

```
int val = 42;	//要查找的值
auto result = find(vec.cbegin(), vec.cend(), val); //找到返回指向它，不然为vec.cend()
```

**6.2只读算法：**只会读取输入范围内的元素，而从不改变元素，如find、accumulate等 在头文件numeric

```
int sum = accumulate(vec.begin(), vec.end(), 0); //和的初始值为0
```

操作两个序列的算法假定第二个序列至少与第一个一样长。

```
equal(roster1.cbegin(), roster1.cend(), rester1.cbegin());
```

**6.3写容器元素的算法：**要确保序列的原大小至少不小于要求算法写入的元素数目。

```
fill(vec.begin(), vec.end(), 0);	//每个元素重置为0

vector<int> vec;
fill_n(vec.begin(), vec.size(), 0);	//正确，fill_n接受操作个数
fill_n(vec.begin(), 10, 0);			//错误，算法不会检查写操作的范围，向空容器中修改10个数

int a1[] = {1, 2, 3};
int a2[sizeof(a1)/sizeof(*a1)];
auto ret = copy(begin(a1), end(a1), a2);	//把a1的内容拷贝给a2
```

重排容器元素的算法

```
void elimDups(vector<string> &words){
	sort(words.begin(), word.end());
	//unique并不会改变元素个数，返回一个指向不重复值范围末尾的迭代器
	auto end_unique = unique(words.begin(), words.end());
	//删除剩余元素
	words.erase(end.unique, words.end());	
}
```

**6.4 lambda**： 约等于一个未命名的内联函数

```
基本形式：[capture list](parameter list) -> return type { function body }
```

1. lambda不能有默认参数，调用的是参数永远与形参数目相等。
2. 一个lambda只有在捕获列表中捕获一个他所在函数中的局部变量，才能在函数体中使用。


```
vector<string> words;
words.push_back("aaa");
words.push_back("bbb");
size_t sz = 4;
//获取一个迭代器，指向第一个满足size() >= sz的元素
auto wc = find_if(words.begin(), words.end(), [sz](const string& a) {return a.size() >= sz; });
//输出大于size >= sz的元素数目
auto count = words.end() - wc;
cout << count << endl;
```

**值捕获：**被捕获的变量的值实在lambda创建时拷贝，而不是调用时拷贝

```
void fun1(){
	size_t v1 = 42;
	auto f = [v1] { return v1; };
	v1 = 0;
	auto j = f();	//j的值为42，f保存了创建它时v1的拷贝
}
```

**引用捕获：**必须保证lambda执行时变量是存在的

```
void fun2(){
	size_t v1 = 42;
	auto f2 = [&v1] { return v1; };
	v1 = 0;
	auto j = f2();	// j为0，f2保存v1的引用，而非拷贝
}
```

**隐式捕获：**让编译器根据lambda中的代码推断要使用哪些变量。&表示引用捕获，=表示值捕获

```
auto wc = find_if(words.begin(), words.end(), [=](const string& a) {return a.size() >= sz; });	//隐式值捕获
```

**可变lambda：**若希望改变一个被捕获的变量的值，需加上关键字mutable

```
void fun3(){
	size_t v1 = 42;
	auto f = [v1] () mutable { return ++v1; };
	v1 = 0;
	auto j = f();	//v1 = 43;
}
```

**指定lambda返回类型：**若lambda体包含return之外的任何语句，则编译器假定lambda返回void。

```
/* transform接受三个迭代器和一个可调用对象，前两个为输入序列，第三个为目的位置，当输入迭代器与目标迭代器相同时，transform将输入序列每个元素替换
*/
// 捕获为空，接受一个int参数， 返回一个int值
// 定义返回类型时 必须使用尾置指针返回类型
transform(vi.begin(), vi.end(), vi.begin(), [](int i) -> int
{ if(i < 0) return -i; else return i; });
```

**6.5流迭代器：**绑定到输入输出流上，可用来遍历关联的IO流。

1. istream_iterator

```
dadasdasd
```

