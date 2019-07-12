### 关于对象(Object Lessons)

C++通过class的pointers和references来支持多态，这种程序设计风格就称为“面向对象”

#### C++的额外成本

C++的类相较于C的struct不仅只包含了数据，同时还包括了对于数据的操作。在语言层面上C++带来了很多面向对象的新特性，类、继承、多态等等。新特性使得C++更加强大，但同时却伴随着空间布局和存取时间的额外成本。作为一个以效率为目标的语言，C++对于面向对象的实现，其实不大，这些额外成本主要由virtual引起，包括：

+ virtual function 机制，用来支持“执行期绑定”。
+ virtual base class ——虚基类机制，以实现共享虚基类的 subobject。

除此之外C++没有太多理由比C迟缓。

#### C++对象模型

C++类有两种数据成员：static和nonstatic，三种函数成员：static、nonstatic和virtual。

所有的非静态数据成员存储在对象本身中。所有的静态数据成员、成员函数（包括静态与非
静态）都置于对象之外。另外，用一张虚函数表（virtual table)**vtbl**存储所有指向虚函数的指
针，并在表头附加上一个该类的type_info对象，在对象中则保存一个指向虚函数表的指
针**vptr**。

| 存在于对象内 | 存在于对象外 |
| -|- |
| 非静态数据成员、虚函数表指针 | 静态数据成员、所有成员函数 |

假定有一个Point类，我们将用三种对象模型来表现它。Point类如下:

```c++
class Point  
{  
public:  
    Point( float xval ); 
    virtual ~Point();      //虚函数成员
    float x() const;  	   //非静态函数成员
    static int PointCount();  //静态函数成员

protected:  
    virtual ostream&  print( ostream &os ) const;//虚函数成员
    float _x;  //非静态数据成员

    static int _point_count;  //静态数据成员
};
```

C++对象模型如下图：

<p align="center">
	<img src=./pictures/cpp_object_model.png alt="Sample"  width="550">
	<p align="center">
		<em>C++ 对象模型</em>
	</p>
</p>

#### C++支持多态的方式

+ 经由一组隐含的转化操作（指针或引用）。例如把一个派生类的指针转化为一个指向其基类的指针：

```c++
shape * ps = new circle();
```

+ 虚函数机制

```c++
ps->rotate();
```

+ 经由dynamic_cast和typeid运算符
```c++
circle * pc = dynamic_cast<circle *>(ps);
```

对象的多态操作要求此对象必须可以经由一个指针或引用来存取。

#### 类对象的内存大小包括

+ 所有非静态数据成员的大小。
+ 由内存对齐而填补的内存大小。
+ 为了支持virtual在内部产生的额外负担

如下类：

```c++
class ZooAnimal {  
public:  
    ZooAnimal();  
    virtual ~ZooAnimal();  
    virtual void rotate();  
protected:  
    int loc;  
    String name;  
};
```

该类在32位计算机上所占内存为16字节：int占四字节，String占8字节（一个4字节的表示字符串长度的整数，一个4字节的字符指针），以及一个指向虚函数表的指针vptr。对于继承类则为基类的内存大小加上本身数据成员的大小。在cfront中其内存布局如下图：

<p align="center">
	<img src=./pictures/class_size.png alt="Sample"  width="350">
	<p align="center">
		<em>C++ 类的内存布局</em>
	</p>
</p>

#### 加上多态后对象的内存分布

+ 指向不同类型之各指针间的差异

指向不同类型的指针从内存需求的观点来说没有什么不同，它们需要的只是足够的内存来放置一个机器地址。它们的差异是在其所寻址出来的对象类型不同，这就导致不能类型的指针所能访问的地址范围有差别。

+ 指针加上多态之后

子类的指针与指向子类对象的基类指针有什么差异？

例如`ZooAnimal`是基类，`Bear`是子类，

```c++
class ZooAnimal { 
public: 
	ZooAnimal(); 
	virtual ~ZooAnimal();
	//...
	virtual void rotate();
protected:
	int loc;
	String name;
};
```

```c++
class Bear:public ZooAnimal{
public:
	Bear();
	~Bear();
	//...
	void rotate();
	virtual void dance();
protected:
	enum Dances{...};
	Dances dances_known;
	int cell_block;
};
```

生成对象与指针赋值

```
Bear b;
ZooAnimal * pz = &b;
Bear * pb = &b;
```

`pz`和`pb`每个都指向Bear 对象的第一各字节，其差别是pb涵盖地址包括整个Bear对象，pz所涵盖的地址只包含Bear对象中的ZooAnimal子对象部分。

+ 用子类对象初始化基类对象

此时基类对象仍然是一个基类对象，通过该对象调用的函数仍为基类的函数，不体现多态。