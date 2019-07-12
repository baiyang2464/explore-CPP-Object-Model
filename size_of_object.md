### C++类对象的大小

Lippman的一个法国读者来信，他写出以下代码:

```c++
class X{};
class Y:virtual public X{};
class Z:virtual public X{};
class A:public Y, public Z{};
```

<p align="center">
	<img src=./pictures/diamond.png alt="Sample"  width="150">
	<p align="center">
		<em>X、Y、Z、A的钻石型继承</em>
	</p>
</p>

并打印处它们的sizeof结果：

```
sizeof X yielded 1   
sizeof Y yielded 4   
sizeof Z yielded 4    
sizeof Z yielded 8    
```

我在vs2017上运行的结果：

<p align="center">
	<img src=./pictures/result.jpg alt="Sample"  width="300">
	<p align="center">
		<em>result</em>
	</p>
</p>

#### 空类也含有一个字节

对于像X这样的一个的空类，编译器会对其动点手脚——隐晦的插入一个字节。为什么要这样做呢？插入了这一个字节，那么X的每一个对象都将有一个独一无二的地址。如果不插入这一个字节呢？哼哼，那对X的对象取地址的结果是什么？两个不同的X对象间地址的比较怎么办？

#### 采用虚继承的子类

我们再来看Y和Z。首先我们要明白的是实现虚继承，将要带来一些额外的负担——虚基类指针，其大小为4字节，然后是编译器安插进去的一个字节的char。到目前为止，对于一个32位的机器来说Y、Z的大小应该为5，而不
是8或者4。

我们需要再考虑两点因素：内存对齐（alignment）和编译器的优化。alignment[^注1]会将数值调整到某数的整数倍，32位计算机上位4bytes。内存对齐可以使得总线的运输量达到最高效率。所以Y、Z的大小被补齐到8就不足为奇了。

那么在vs2017中为什么Y、Z的大小是4而不是8呢？我们先思考一个问题，X之所以被插入1字节是因为本身为空，需要这一个字节为其在内存中给它占领一个独一无二的地址。但是当这一字节被继承到Y、Z后呢？它已经完全失去了它存在的意义，为什么？因为Y、Z各自拥有一个虚基类指针，它们的大小不是0。既然这一字节在Y、Z中毫无意义，那么就没必要留着。也就是说vs2017对它们进行了优化，优化的结果是去掉了那一个字节,而Lippman的法国读者的编译器显然没有做到这一点。

#### 钻石继承的顶点

当我们现在再来看A的时候，一切就不是问题了。

对于那位Lippman的法国读者来说，A的大小是共享的X实体1字节,Y和Z的大小分别减去虚基类带来的内存空间这是应该放到子类中去的（虚基类X在其子类中只能存在一个实体），都是4。A的总计大小为9，alignment以后就是12了。

而对于vs2017来说，那个一字节被优化后，A的大小为8，也不需再进行alignment操作。

#### 总结

影响C++类的大小的三个因素：

+ 支持特殊功能所带来的额外负担（对各种virtual的支持，虚函数表指针，虚基类表指针）。
+ 编译器对特殊情况的优化处理。
+ alignment操作，即内存对齐。

[^注1]: 关于更多的memory alignment（内存对齐）的知识见[VC内存对齐准则（Memory alignment）](http://www.roading.org/develop/cpp/vc内存对齐准则（memory-alignment）.html)