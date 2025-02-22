##### 1. 讲一讲封装、继承、多态是什么？
+ 封装：将具体的实现过程和数据封装成一个函数，只通过接口访问，降低耦合性。使得类成为一个具有内部数据的功能独立的软件模块
+ 多态：不同继承类对同一消息做出不同反应。基类指针指向派生类对象。使得代码具有可扩展性，但不会影响已存在的各种性质。
+ 继承：复用基类的全体数据和成员函数。
+ 构造函数、系构函数、友元函数（定义在类的外部，不属于类的方法但可以访问类的成员）、静态成员函数都不能被继承

##### 2. 多态的实现原理？优点？
+ 动态多态：利用虚函数实现多态，系统编译时并不知道会调用哪个，只有运行时才直到
+ 静态多态：通过函数重载（函数名相同，参数不同），运算符重载，重定义/重写（继承中子类实现和父类一样的函数，名称和参数相同）实现
+ 优点：加强可扩展性，可替换性，提高使用效率

##### 3. final标识符的作用
+ 对象：类、虚函数、成员函数
+ 不能被继承、重写

##### 4. 虚函数如何实现？放在内存哪个区？什么时候生成的？
+ 虚函数表：包含虚函数的类生成，存储所有指向该虚函数实现代码的指针
+ 虚函数指针：指向对应的虚函数表
+ 当调用虚函数时，在编译阶段就会使用虚表指针查找对应的虚函数表，调用正确的虚函数
+ 在编译阶段生成，和普通代码一样存放在代码段，唯一差异是指针也存在虚表里

##### 5. 智能指针？
封装了原始指针的C++类模板，通过系构函数来自动释放资源。
实现：
+ auto_ptr：c++98提供，将原始对象拷贝给新对象，原始对象被设置为nullptr。调用原指针会崩溃，很多公司禁止使用
+ unique_ptr：c++11提供，直接禁止拷贝和赋值
+ shared_ptr：C++11提供。内部维护一个计数器，记录被几个对象共享；有一个被销毁时（调用系构函数），析构函数内计数器减一；计数器0时释放资源。对于双向链表会出现循环引用的现象
+ weak_ptr：可以指向shared_ptr且不会增加shared_ptr的计数，可以用于双向链表内部，防止循环引用（二叉树等同理）

另外删除数组时delete后面要有[]，而智能指针式后面默认不带[]，所以会报错。针对数组的智能指针需要提供自定义删除器

##### 6. 匿名函数？优点？
+ 本质是个对象，定义过程中会创建一个栈，通过重载()符号实现函数调用外表
+ 免去声明和定义。调用时才会创建，结束立刻释放，节省空间

##### 7. 右值引用？为什么要引入？
左值引用（生命周期必须小于引用对象的生命周期）：
```C++
int a = 5;
int &ref_a = a; //ok
int &ref_a = 5; //no, 5是右值
const int &ref_a = 5; //ok，但无法修改
```

右值引用（可以延长引用对象的生命周期至该右值引用消亡）
```C++
int &&ref = 5; // ok
int a = 5;
int &&ref = a; //ng

ref = 6;
```
移动语义std::move（将左值强制转换成右值，可以触发右值引用的重载函数）提高性能：
很多函数的实现都有右值引用，直接移动数据，相比涉及拷贝的左值引用，右值引用避免的深拷贝，速度更快。在 **需要拷贝且被拷贝者之后不需要的场景下**可以使用std::move触发移动语义（而不是拷贝），提升性能。

完美转发std::forward:
``std::forward<T>(u)``，当T为左值引用类型，u将被转换为T类型的左值，否则为右值。
```C++
std::forwrad<int>(ref) // 转为右值
std::forwrad<int &&>(ref) // 转为右值引用
std::forwrad<int &>(ref) // 转为左值引用
```

总结：
+ 为了支持移动语义，避免昂贵的复制操作
+ 完美转发，右值引用可以绑定到任何类型右值上，传到函数内部再根据需要转发到不同类型中
+ 扩展可变参数模板

##### 8. 左值和指针区别？
+ 初始化：指针不用，引用必须
+ 性质：指针是变量，引用是别名
+ 占用内存：指针有地址，引用和被引用占同一个空间

# 野指针
悬空指针指向已被释放地址的指针
野指针不是NULL，而是保存了非法不可用的地址，访问会造成越界、段错误等问题
造成野指针的原因：
+ 局部变量没有初始化
+ 使用已经释放的指针
+ 指针指向变量已被销毁
+ 错误的指针运算
+ 错误的强制类型转换
如何避免：
+ 定义指针初始化NULL
+ 解除引用前先判断是否是NULL
+ 使用完后赋值为NULL
+ 良好的编码和设计，谁用谁管理