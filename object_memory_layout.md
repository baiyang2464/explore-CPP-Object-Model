## 对象成员内存布局

####单继承（只有一个父类）

类的继承关系为：class Derived : public Base

```c++
class Base {
public:
	int b;
	virtual void vfb() {};
};

class Derived :public Base
{
public:
	int d;
};
```



![img](https://upload-images.jianshu.io/upload_images/53611-a6abdd99c2ed1a78?imageMogr2/auto-orient/strip%7CimageView2/2/w/246/format/webp)

vs2017中内存布局信息

```shell
1>class Base	size(8):
1>	+---
1> 0	| {vfptr}
1> 4	| b
1>	+---
1>
1>Base::$vftable@:
1>	| &Base_meta
1>	|  0
1> 0	| &Base::vfb
1>
1>Base::vfb this adjustor: 0
1>
1>class Derived	size(12):
1>	+---
1> 0	| +--- (base class Base)
1> 0	| | {vfptr}
1> 4	| | b
1>	| +---
1> 8	| d
1>	+---
1>
1>Derived::$vftable@:
1>	| &Derived_meta
1>	|  0
1> 0	| &Base::vfb
```

Derived类的对象的内存布局为：虚函数表指针、Base类的非static成员变量、Derived类的非static成员变量。

#### 多重继承（多个父类）
类的继承关系如下：class Derived : public Base1, public Base2

```c++
class Base1 {
public:
	int b1;
	virtual void vfb1() {};
};
class Base2 {
public:
	int b2;
	virtual void vfb2() {};
};

class Derived :public Base1,Base2
{
public:
	int d;
};
```



![img](https://upload-images.jianshu.io/upload_images/53611-489fc115e463fbb4?imageMogr2/auto-orient/strip%7CimageView2/2/w/246/format/webp)



vs2017中内存布局信息

```shell
1>class Derived	size(20):
1>	+---
1> 0	| +--- (base class Base1)
1> 0	| | {vfptr}
1> 4	| | b1
1>	| +---
1> 8	| +--- (base class Base2)
1> 8	| | {vfptr}
1>12	| | b2
1>	| +---
1>16	| d
1>	+---
1>
1>Derived::$vftable@Base1@:
1>	| &Derived_meta
1>	|  0
1> 0	| &Base1::vfb1
1>
1>Derived::$vftable@Base2@:
1>	| -8
1> 0	| &Base2::vfb2
1>
```

Derived类的对象的内存布局为：基类Base1子对象和基类Base2子对象及Derived类的非static成员变量组成。基类子对象包括其虚函数表指针和其非static的成员变量。

#### 重复继承（继承的多个父类中其父类有相同的超类）

类的继承关系如下：
class Base1 : public Base
class Base2: public Base
class Derived : public Base1, public Base2

```c++
class Base {
public:
	int b;
	virtual void vfb() {};
};
class Base1 :public Base{
public:
	int b1;
	virtual void vfb1() {};
};
class Base2 :public Base {
public:
	int b2;
	virtual void vfb2() {};
};

class Derived :public Base1,Base2
{
public:
	int d;
};
```



<p align="center">
	<img src=./pictures/123.jpg alt="Sample"  width="550">
	<p align="center">
		<em> </em>
	</p>
</p>

vs2017中内存布局信息

```shell
1>class Derived	size(28):
1>	+---
1> 0	| +--- (base class Base1)
1> 0	| | +--- (base class Base)
1> 0	| | | {vfptr}
1> 4	| | | b
1>	| | +---
1> 8	| | b1
1>	| +---
1>12	| +--- (base class Base2)
1>12	| | +--- (base class Base)
1>12	| | | {vfptr}
1>16	| | | b
1>	| | +---
1>20	| | b2
1>	| +---
1>24	| d
1>	+---
1>
1>Derived::$vftable@Base1@:
1>	| &Derived_meta
1>	|  0
1> 0	| &Base::vfb
1> 1	| &Base1::vfb1
1>
1>Derived::$vftable@Base2@:
1>	| -12
1> 0	| &Base::vfb
1> 1	| &Base2::vfb2
1>
```

Derived类的对象的内存布局与多继承相似，但是可以看到基类Base的子对象在Derived类的对象的内存中存在一份拷贝。这样直接使用Derived中基类Base的相关成员时，就会引发歧义，可使用多重虚拟继承消除之。

菱形继承导致了共有基类被继承了两次，可使用虚拟继承来消除，此外还可使用临时作用域来显示明确调用。

[C++菱形继承，二义性问题以及解决方法](<https://blog.csdn.net/liu_zhen_kai/article/details/81590467>)

#### 多重虚拟继承（使用virtual方式继承，为了保证继承后父类的内存布局只会存在一份）
 类的继承关系如下：
 class Base1 : virtual public Base
 class Base2:  virtual public Base
 class Derived : public Base1, public Base2

```c++
class Base {
public:
	int b;
	virtual void vfb() {};
};
class Base1 :virtual public Base{
public:
	int b1;
	virtual void vfb1() {};
};
class Base2 :virtual public Base {
public:
	int b2;
	virtual void vfb2() {};
};
```



![img](https://upload-images.jianshu.io/upload_images/53611-8e940f35b94afec6?imageMogr2/auto-orient/strip%7CimageView2/2/w/246/format/webp)

vs2017中内存布局信息

```shell
1>class Derived	size(36):
1>	+---
1> 0	| +--- (base class Base1)
1> 0	| | {vfptr}
1> 4	| | {vbptr}
1> 8	| | b1
1>	| +---
1>12	| +--- (base class Base2)
1>12	| | {vfptr}
1>16	| | {vbptr}
1>20	| | b2
1>	| +---
1>24	| d
1>	+---
1>	+--- (virtual base Base)
1>28	| {vfptr}
1>32	| b
1>	+---
1>
1>Derived::$vftable@Base1@:
1>	| &Derived_meta
1>	|  0
1> 0	| &Base1::vfb1
1>
1>Derived::$vftable@Base2@:
1>	| -12
1> 0	| &Base2::vfb2
1>
1>Derived::$vbtable@Base1@:
1> 0	| -4
1> 1	| 24 (Derivedd(Base1+4)Base)
1>
1>Derived::$vbtable@Base2@:
1> 0	| -4
1> 1	| 12 (Derivedd(Base2+4)Base)
1>
1>Derived::$vftable@Base@:
1>	| -28
1> 0	| &Base::vfb
```

Derived类的对象的内存布局与重复继承的类的对象的内存分布类似，但是**在对象的内存中仅存在在一个共有虚基类Base类的子对象。且共有的虚基类的非static成员变量放置在对象的末尾处。**