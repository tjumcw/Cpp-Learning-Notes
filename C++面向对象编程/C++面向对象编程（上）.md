# C++面向对象编程笔记（上）

#### inline函数

- inline即直接定义在class内部，声明在里面但定义在外面不算inline

- 在类外加inline关键字也是声明为inline
- 对于上面的两种情况，但编译器具体是否将其编为inline那就不确定了
  - 只是给编译器的一种建议，太复杂无法实现为inline



#### const成员函数

- 理解const修饰及其位置
  - const放类型前面，如const int a = 1；表示其不可修改，常用作函数参数
  - const修饰函数返回值，如const int getNum()；表示函数返回值是不可修改的
  - const修饰成员函数时，放函数名后面，如int getNm() const;
    - 表示函数体内不能修改类内的成员变量
    - 表示函数体内不能调用其他类内的非const成员函数

- 若对象声明为const类型，即上述int换成自定义类型，如const myClass a;
  - 只能访问const成员函数，非const对象可以访问任意的成员函数
  - const对象的成员是不可修改的,然而const对象通过指针维护的对象却是可以修改的
    - 改的不是指针，改的是指针指向的内容，对于指针本身没有发生改变
  - mutable修饰符的数据成员,对于任何情况下通过任何手段都可修改,



#### pass by value / reference

- 值传递就是整个传过去，value多大就传多大，传的动作实际是压到函数的栈
  - 即这个值占几个字节就传几个字节
- 引用底部就是指针，传引用类似传指针，就是4个字节
  - 会随着参数的修改而修改实际值
  - 如既想要通过传引用加速，又不希望改值，加const，如const int& a



#### return by value / reference

- 也是尽量传引用
- 返回临时对象不能return by reference



#### friend友元

- 若字段声明为private，非成员函数是不能直接访问的，但若将类外定义的函数以友元的形式声明在类内，则在类外的具体函数内可以直接访问private的成员变量

- 相同class的各个objects互为friend

  - 即类内的成员函数参数若为同类对象，可在成员函数内直接访问参数的私有成员

  - ```c++
    class Complex{
    public:
        int func(const Complex& a){
            return a.re + a.im;
        }
    private:
        double re, im;
    };
    ```



#### 操作符重载

- 成员函数的操作符重载，其默认会带一个隐藏的this指针,少一个操作数

- ```c++
  complex::operator += (this, const Complex& a){};
  ```

- 非成员函数的操作符重载，该几个操作数就需要有几个对应的函数参数



#### 拷贝构造，拷贝赋值，析构 

- 若类内有指针，需要自己实现拷贝构造和拷贝赋值函数，以自己写String类为例

- ```c++
  class String{
  public:
      String(const char* cstr = 0);				//有参构造
      String(const String& str);					//拷贝构造
      String& operator = (const String& str);		//拷贝赋值
      ~String();
  private:
      char* m_data;
  }
  ```

- 为了防止浅拷贝的出现



#### new：先分配memory，再调用ctor

- 若有一条动态创建复数对象的语句，如下

- ```c++
  Complex* pc = new Complex(1, 2);
  ```

- 则根据new的机制，编译器将其转化为

- ```c++
  Complex* pc;
  
  void* mem = operator new(sizeof(Complex));	//第一步，分配内存
  pc = static_cast<Complex*>(mem);			//第二步，转型
  pc->Complex::Complex(1,2);					//第三步，调构造函数
  ```

- 其中，operator new内部调用malloc(n)



#### delete：先调用dtor，再释放memory

- 如对动态创建的复数对象pc进行释放，如下

- ```c++
  Complex* pc = new Complex(1,2);
  //...
  delete pc;
  ```

- 编译器将其转化为

- ```c++
  Complex::~Complex(pc);		//第一步，调析构函数
  operator delete(pc);		//第二部，释放内存
  ```

- 其中，operator delete内部调用free(pc)



#### array new 一定要搭配 array delete

- 首先回顾String类，其成员只有一个char*类型指针，指向的是new出来的char数组
- 给出关于delete[]及delete对于String[]的对比

![image](https://user-images.githubusercontent.com/106053649/177124002-37095bc4-0354-4a83-b2ca-137648e204ee.png)



- 首先理解上述new的过程
  - 明确若直接定义在栈上，即String str；其内部也会有动态分配的内存(构造函数将new char[]赋给成员变量m_data)
  - 由于在析构函数中写了delete，故对象生命周期结束时，会通过析构函数释放new出来的内存
- 其次理解new语句为String* p = new String[3];
  - 动态创建了3个这样的对象，每个对象内部（无论对象在栈上还是堆上）都有动态分配的空间(m_data)，因为类内部构造函数自带new
  - 若是动态创建复数数组，如Complex* p = new Complex[3]，则其只有图中任意半边的左边部分(没有3根箭头指出去的部分)
- 再根据上述的delete调用规则，对每次delete操作，都是
  - 先调用dtor，再释放memory

- 通过[]符号，编译器就知道是数组，会调用多次析构函数
  - 每次析构函数都会把类内自己在构造时各自动态分配的内存杀掉(m_data)
    - 上述new语句生成了3个对象，每个对象内部都有各自new出来的m_data(即被灰色部分指向的单个白色块，3个对象所以有3个)
  - 三次析构完了后，delete的第一个环节结束，再去释放通过new分配的内存(即灰色部分的三块)，一次性释放3块
- 若直接delete，看图中右半部分，只会调用一次析构函数
  - 通过一次析构函数只能释放掉一块类内自己new出来的m_data(白色块)
  - 还剩下两块白色块(m_data)没有被释放
  - 析构完了之后释放memory，将左边灰色部分一次删干净
- 可见，对于array new出来的内存使用delete，会造成内存泄漏，但不是常规意义的内存泄漏
- 直接delete能显式释放掉new语句对应的内存，若是遇到带指针的类，其内部自己new出来的内存只能删一块，不能清理干净
- 对于复数类（内部不带指针），delete与delete[]效果一致，但这样的习惯不好（歪打正着）



#### static

- non-static的对象声明后，每个成员在内存中都有自己的成员
- 但是non-static的函数只有一份在代码区，一份函数需要处理不同的对象（通过传this进来，每个对象的this不一样）
  - 每个non-static成员函数都有一个隐藏的this指针作为参数
- static修饰的成员变量只有一个，成员函数也只有一个（与一般的成员函数类似），但没有this指针
- 所以static成员函数只能处理静态的成员变量
- 静态成员变量需要在类外初始化
- 调用静态函数可以用类名也可以用对象



#### 类的复合（组合的是类），表示has-a

- 构造函数首先调用组件的构造函数，再调用拥有者的构造函数
- 析构函数首先调用拥有者自己的析构函数，再去调用组件的析构函数



#### 类的委托（组合的是类指针），也表示has-a

- 设计模式最喜欢的方式
- 其实也可以叫做类的组合



#### 继承，表示is-a（public）

- 利用虚函数机制实现接口类很常用

- 构造函数，先调用基类，再调用子类
- 析构函数，先调用子类，再调用基类
  - 基类的析构函数记得要声明为virtual类型



#### 虚函数（单独整理，继承里内容太多）

- virtual函数，希望子类重新定义（override），且它已有默认定义
- pure virtual函数，子类必须要重新定义（override），没有默认定义
