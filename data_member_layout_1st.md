### 初探数据成员的布局

对于一个类来说它的对象中只存放非静态的数据成员,但是除此之外，编译器为了实现virtual功能还会合成一些其它成员插入到对象中。非静态数据成员存放在程序的“全局数据段”中。

我们来看看这些成员的布局。

#### C++ 标准的规定

+ 在同一个Access Section（也就是private,public,protected片段）中，
  要求较晚出现的数据成员处在较大的内存中。这意味着同一个片段中的数据成员并不需要紧密相连，编译器所做的成员对齐就是一个例子。
+ 允许编译器将多个Acess Section的顺序自由排列，而不必在乎它们的声明次序。但似乎没有编译器这样做。
+ 对于继承类，C++标准并未指定是其基类成员在前还是自己的成员在前。
+ 对于虚基类成员也是同样的未予规定。

#### 一般的编译器怎么做？

+ 同一个Access Section（即private/public/protected区段）中的数据成员按期声明顺序，依次排列。但成员与成员之间因为内存对齐的原因可能存在空当。
+ 多个Access Section按其声明顺序排放。
+ 基类的数据成员总放在自己的数据成员之前，但虚基类除外。

#### 编译器合成的成员放在哪？

为了实现虚函数和虚拟继承两个功能，编译器一般会合成虚函数表指针Vfptr和虚基表指针Vbptr两个指针。那么这两个指针应该放在什么位置？C++标准肯定是不曾规定的。

对于Vptr来说有的编译器将它放在末尾，如Lippman领导开发的Cfront。有的则将其放在最前面，如MS的Visual Studio，但似乎没人将它放在中间。为什么不放在中间？没有理由可以让人这么做，放在末尾，可以保持C++类对C的struct的良好兼容性，放在最前可以给多重继承下的指针或引用调用虚函数带来好处。

Vbptr则是紧随Vfptr之后存放。

看一段小代码

```c++
class Base {
public:
	int b;
	virtual void vfb() {};
};
class Derived :public virtual Base
{
public:
	int d;
	virtual void vfd() {};
	virtual void vfd1() {};
};
```

Visual Studio中程序的内存布局

```shell
1>class Base	size(8):
1>	+---
1> 0	| {vfptr}			//基类的虚函数指针
1> 4	| b					//基类的数据成员
1>	+---
1>
1>Base::$vftable@:			//基类虚函数表
1>	| &Base_meta
1>	|  0
1> 0	| &Base::vfb
1>
1>Base::vfb this adjustor: 0
1>
1>class Derived	size(20):
1>	+---
1> 0	| {vfptr}			//派生类虚函数指针
1> 4	| {vbptr}			//派生类虚基类指针，Vbptr紧随Vfptr之后存放
1> 8	| d
1>	+---
1>	+--- (virtual base Base)//虚基类在派生类中的子对象
1>12	| {vfptr}
1>16	| b
1>	+---
1>
1>Derived::$vftable@Derived@:
1>	| &Derived_meta
1>	|  0
1> 0	| &Derived::vfd
1> 1	| &Derived::vfd1
1>
1>Derived::$vbtable@:
1> 0	| -4
1> 1	| 8 (Derivedd(Derived+4)Base)
1>
1>Derived::$vftable@Base@:
1>	| -12
1> 0	| &Base::vfb
1>
```

#### 虚基表与虚函数表

+ 虚基类表中的表项记录的是表项中的虚基类子对象与派生类对象中虚基表指针的偏移

```shell
1>Derived::$vbtable@:
1> 0	| -4 //派生类Derivd中vfptr距离派生类Derived对象偏移为-4
1> 1	| 8 (Derivedd(Derived+4)Base) //虚基类Base子对象距其派生类Derivd中vfptr偏移为8
```

+ 虚函数表的表象记录的是虚函数的地址

#### 小结

在VC中数据成员的布局顺序为：

1. vptr部分（如果基类有，则继承基类的）
2. vbptr （如果需要）
3. 基类成员（按声明顺序，与虚基类成员区分）
4. 自身数据成员
5. 虚基类数据成员（按声明顺序）