# STL

#### STL六大部件

- 容器（Containers）
- 分配器（Allocators）
- 算法（Algorithms）
- 迭代器（Iterators）
- 适配器（Adapters）
- 仿函数（Functors）

![image](https://user-images.githubusercontent.com/106053649/177124202-7735c921-67da-4269-a624-9b7c7976c57b.png)

- 通常使用STL，第一个直接接触容器，容器要放东西，东西需要占内存
- 容器使使用者无需关注内存，只需要把东西一个一个放进去，背后是分配器的支持
- 操作容器需要通过算法
- 为了算法的适配性和复用性，将算法和容器本身隔离，通过迭代器来沟通
- 适配器可以对容器、迭代器及仿函数做转换



#### 前闭后开区间

- 所有容器的迭代器：begin指向第一个元素，end指向最后一个元素的下一个元素



#### 容器——结构与分类

- 序列式容器
  - Array（连续）
  - Vector（连续）
  - Deque（分段连续造成连续假象）
  - List（双向链表）
  - Forward List
- 序列式容器
  - Set/Multiset（类内部复合一个rb_tree做底层支撑）
  - Map/Multimap（类内部复合一个rb_tree做底层支撑）
  - 包括unordered版本
- 其中Stack和Queue是Deque的适配器，即复合了一个Adapter
- heap里有一个vector（复合，支撑底层实现），priority_queue里有一个heap（复合，支撑底层实现）



#### 容器扩充

- Array不能扩充
- Vector一旦放不下，两倍扩充
- List和Forward List每次扩充一个
- Deque每次扩充一个buffer（无论前后），具体扩充的大小即单个buffer大小



#### OOP vs GP

- 面向对象编程企图把datas（成员变量）和methods（成员函数）关联在一起
- 泛型编程却是把datas（成员变量）和methods（成员函数）分开来
  - 利用迭代器把methods应用到特定的datas上

- 泛型编程的好处
  - 容器和算法可以各自闭门造车，利用迭代器沟通即可
  - 算法通过迭代器确定操作范围，并通过爹大气取用容器元素

- 除了Array和Vector，所有容器的迭代器都是class



#### 模板

- 函数模板能通过编译器自动推导实参（实现编译时多态），类模板必须显式指定
- 类模板具有特化功能（全特化、偏特化）
- typename和class关键字在模板里可以互换



#### 六大部件——分配器allocators

- 所有的分配动作都会跑到malloc()，所以先看operator new()和malloc()
- operator new()里会调用malloc()，malloc分配时比要的大小会大一些
  - 上下的cookie（释放时用来找到对应内存块）以及往16倍数靠的padding
  - 每次调用malloc()分配都会这样，分配10000个元素，会调10000次malloc()，也是10000个这样的块
  - 可通过 _pool_alloc分配器缓解上述开销（池的思想）

- 分配器类allocator（模板类）最关键的两个函数
  - allocate()，其中会调用Allocate()，在Allocate()中调用operator new()，operator new()调malloc()
    - 不同实现可能没有Allocate()，但流程必定是allocate()->operator new()->malloc()
  - deallocate()，其中会调用operator delete()，operator delete()调用free()

- 容器的元素都分配在堆上（分配器机制），与容器本身在哪无关（容器本身可以在堆或栈上）



#### 迭代器Iterator

- 所有Iterator都要做很多操作符重载
  - 包括但不限于*，->，前++，后++
- 几乎每个容器的Iterator都一定要做5个typedef，牵涉到traits（具体看traits）



#### Iterator Traits

- Iterator Traits用来萃取出Iterator的特性，故Iterator需要满足一定的原则

- Iterator是沟通算法和容器的桥梁，算法通过Iterator可知操作范围，并通过Iterator取元素对其操作

- 算法在运算过程中很可能想知道Iterator的特性，以便在算法中选择最适合的子函数调用

  - 如算法想知道迭代器移动的性质，通过iterator_traits去看iterator_category()

  - ```c++
    return typename iterator_traits<_Iter>::iterator_category();
    ```

  - 因为算法和容器分开，算法写了很多种子函数，知道其分类以便选择最佳调用方式

- difference_type指该类型两个Iterator 的距离该用什么类型表现

- value_type指Iterator 所指向的元素的type

- 算法提出问题问Iterator ，Iterator 要能够回答上述的问题，即上述的三种

  - 5种Iterator 的相关类型（associated types），剩下的reference和pointer在标准库中没被使用过

- 迭代器必须定义出这5种相关类型，以便算法提问（通过Iterator Traits）

  - iterator_category
  - value_type
  - difference_type
  - reference
  - pointer

- 但实际上若是类的话，算法能直接提问并拿到想要的问题，无需traits，直接提问方式如下：

- ```c++
  template<typename I>
  void algorithm(I first, I last){
      //...
      I::iterator_category
      I::value_type
      I::difference_type
      I::reference
      I::pointer
      //...
  }
  ```

- 直接这样提问就能拿到想要的答案，那为什么要设计iterator_traits呢

  - 只有类才能typedef，如果迭代器并不是个class而是普通指针呢（被视为退化的迭代器）

- Iterator_Traits能区分传进来的迭代器是class还是普通的指针，相当于一个中间层，实现间接问

![image](https://user-images.githubusercontent.com/106053649/177180049-d4290eef-0866-4d52-8e10-e94dca862a02.png)

- 根据图片，可知iterator_traits作为中间层，对指针类型做了偏特化，实现算法提问形式的一致性（无论是普通指针还是设计为类迭代器）

- 算法的提问形式为：

- ```c++
  template<typename I,...>
  void algorithm(...){
      typename iterator_traits<I>::value_type v1;
  }
  ```

- 即通过iterator_traits，取得类型I的迭代器的value_type，并命名为v1拿来给自己用

- 根据传入iterator_traits模板参数，若为class iterator

- ```c++
  template<class I>
  struct iterator_traits{
  	typedef typename I::value_type value_type
  }
  ```

- 将类型I的value_type重命名为value_type，这样算法通过上述的提问方式直接拿到value_type（即I::value_type）

- 根据传入iterator_traits模板参数，若为模板参数的指针形式（给出无const版本，有const类似）

- ```c++
  template<class T>
  struct iterator_traits{
  	typedef T value_type;
  }
  ```

- 偏特化后，传入的模板参数若是T*形式，则T就为其value_type，算法的提问方式也和上述一致



- 完整的iterator_traits如下

![image](https://user-images.githubusercontent.com/106053649/177180099-3f264bec-73e8-4098-9cb7-323eedf2b214.png)

- 可见，对于class型的迭代器，iterator_traits相当于就是简单的重命名
- 对于指针型的迭代器，iterator_traits根据其指针所指向的类型给出回答并命名为一致形式
  - 指针型迭代器的iterator_category一定是支持随机访问的



#### 容器list（GNU2.9）

- list类大小为4，只有一个list_node*类型的指针
- list_node类内，有两个指针prev和next以及一个模板类型T的data
- 在list类内需要对Iterator的操作符重载
  - 如*Iterator就是取node的data（实际是node指针指向的list_node类内的data）

- 是双向环状的，为了符合前闭后开区间，增加了一个空的node作为end()



#### 容器vector（GNU 2.9）

- 所有的扩充都不能在原地扩充（不止vector）

  - 因为你要了一块内存之后，这一块内存后面的内存接下去可能就被使用了
  - 需要在内存中找到另一块空间，把原来的元素搬过去

- 在STL中vector的类实现，有三个成员变量用于扩充（vector对象大小为12），如下：

- ```c++
  iterator start;
  iterator finish;
  iterator end_of_storage
  ```

- 元素的拷贝会引发拷贝构造函数，并把原来的部分调析构函数

- vector顺序存储较为简单，迭代器不是通过类来设计的，而是普通的指针

- ```c++
  template<class T, class Alloc = alloc>
  class vector{
  public:
      typedef T value_type;
      typedef value_type* iterator;	//实际就是T*
      //...
  }
  ```

- 可见vector的迭代器就是传入模板参数对应的指针T*



#### 容器array

- array即普通数组，为什么要把它也包装成容器呢

- 通过将其按照STL的要求重新包装，使其也同样通过迭代器机制，可享受定义的算法，仿函数等STL方便的部分

- 实际上平时也几乎不会用，声明方式如下（不能扩充，需指定大小）：

- ```c++
  array<int, 10> myArray;
  ```

- 其类的关键细节如下：

- ```c++
  template<typename _Tp, std::size_t _Nm>
  struct array{
      typedef _Tp value_type;
      typedef value_type* iterator;				//可见其迭代器就是其模板参数对应的指针
      
      value_type _M_instance[_Nm ? _Nm : 1];		//其结构体内唯一的成员变量，是一个_Tp类型的，且大小为_Nm为数组(_Nm为0则大小为1)
  }
  ```



#### 容器deque

- deque的中控器是vector，其内的每个元素都是一个指向一个缓冲区buffer的指针，buffer也是连续的
- 中控器满了就会2倍成长（vector），缓冲区大小为512/类型大小
  - 中控器扩充时，会把原来的元素放在新的空间的中间，预留前后空间大小相等
- deque类内有2个iterator，1个指向中控器的指针以及一个int型的中控器大小
- deque的迭代器有4个字段，都是指针
  - cur：当前缓冲区目前指向的元素
  - first：当前缓冲区的第一个
  - last：当前缓冲区的最后一个的后一个
  - node：指向当前缓冲区对应中控器中的元素
- 故deque大小为16 * 2 + 4 + 4 = 40



#### 容器stack、queue

- 复合一个deque对象（不是指针），将前后端封闭就形成了queue和stack

- stack和queue根据其特性（不能随意insert），不允许遍历，所以也不提供iterator

- 成员函数也只是通过deque对象调用deque的成员函数

- stack和queue还可以选择list作为底层结构，但一般都使用deque为底层结构

  - 其中，stack还可以选择vector作为底层结构（先进后出符合vector只能尾插）

  - ```c++
    stack<string, vector<string>> s;
    ```

  - 测试所有函数都能通过，因为本身就是转调用第二个模板参数的函数（默认是deque）



#### 容器rb_tree（GNU 2.9）->幕后英雄（底层容器）

- 红黑树提供遍历操作以及迭代器，按正常规则（++iterator）遍历，能获得排序状态的元素

![image](https://user-images.githubusercontent.com/106053649/177180146-c8b50a01-b20d-4976-a438-cfc95a48d576.png)

- begin记录最坐标的元素，end记录最右边的元素
- header类似双向链表中的end空节点，是为了实现方便刻意制造出来的
- 提供两种插入操作
  - insert_unique()，key必须独一无二
  - insert_equal()，允许key重复

- 构造红黑树需要的模板参数如下（一般不用，直接用上层的set、map）：

- ```c++
  template<class Key,
  		 class Value,
  		 class KeyOfValue,
  		 class Compare,
  		 class Alloc = alloc>
  ```

- 其中，key表示键，Value不是指正常意义的value，而是指键值对整体

  - （key -> data）称为Value

- KeyOfValue指怎么拿到Key，因为Value可以比较复杂，将Key和data合成一包，需要知道怎么拿出key，也是仿函数

- Compare是一个仿函数，给出排序规则

- 最后一个模板参数是有默认值的分配器

- 红黑树类只有三个protected成员变量（容器的成员变量大多都是protected）
  - size_type类型的node_count，表示有几个元素，大小为4
  - link_type类型的header，是一个指向rb_tree_node的指针，大小为4
  - Compare类型的key_compare，是一个空类，重载了()运算符，虽然大小是0但要设为1
  - 故红黑树大小为9，内存对齐后为12（GNU4.9为24）



#### 容器set,multiset

- 以红黑树作为底层结构，正常遍历（++iterator）得到的是排序结果
- set的insert()调用的是rb_tree的insert_unique()
- multiset的insert()调用的是rb_tree的insert_equal()
- set/multiset的迭代器拿的是rb_tree的const_iterator，保证不能通过iterator修改set元素
- set/multiset也是转调用rb_tree的函数，也可看作是容器适配器（类似stack、queue）
- set/multiset的成员变量只有一个private的rb_tree对象，对象的大小即为红黑树类对象的大小



#### 容器map,multimap

- 以红黑树作为底层结构，正常遍历（++iterator）得到的是排序结果，排序依据key
- insert()方式类似set，map/multimap分别调用insert_unique()/insert_equal()
- 成员变量也只有一个private的rb_tree对象，对象的大小即为红黑树类对象的大小
- 由于map/multimap虽然key同样不能改，但可修改data，故其的迭代器拿的是红黑树普通的iterator
  - 但是map类内通过定义其value_type为pair<const key, T>的形式保证了key不可变（key，T为模板参数）
- map,multimap重载了[]运算符，通过map[key]能得到data，set,multiset不行



#### 容器hashtable ->幕后英雄（底层容器）

- 发生碰撞时
  - 若是普通的连续空间，可以通过重映射（通过新的一次方程式/二次方程式等新的映射方式）映射到新位置
  - 若是开链法（数组加链表），就在链表上一直往后排
- bucket指的是数组部分的每个位置，每个bucket都是一个链表
- 如果元素个数大于bucket个数时（一般有些bucket为空，有些bucket元素很多），需要rehashing
- rehashing会扩充bucket，回去寻找一开始bucket数量2倍附近的质数（默认一开始53）
  
- 开销很大，不仅需要像vector那样全部搬过去，还要重新计算每个元素应该在哪个bucket上
  
- hashtable的模板参数

- ```c++
  template<class Value, class Key, class HashFcn, class ExtractKey,
  		 class EqualKey, class Alloc = alloc>
  ```

- hashtable的模板参数
  - Value和Key概念同红黑树
  - HashFcn就是一个仿函数，通过其得到元素的hashcode
  - ExtractKey类似红黑树的KeyOfValue，从一包中提取出Key，也是仿函数
  - EqualKey就是指定按照key的排序规则，仿函数
- hashtable的成员变量
  - 模板参数中三个仿函数的对象，大小为1 + 1 + 1 = 3
  - size_type类型的num_elements，大小为4
  - vector<node*, Alloc> buckets，vector本身是三个指针，大小为12
  - hashtable大小为19



#### unordered系列的set/multiset，map/multimap

- 以hashtable为底层结构



#### STL小理解

- 算法看不见容器，对其一无所知
- 所以它所需要的一切信息都必须从迭代器取得
- 迭代器（由容器供应）必须能够回答算法的所有提问，才能搭配该算法的所有操作 



#### 迭代器的iterator_category

- input_iterator_tag及output_iterator_tag
- farward_iterator_tag（单向）
  - Forward_List
- bidirectional_iterator_tag（双向）
  - List，以rb_tree实现的set/multiset，map/mu;timap
- random_access_iterator_tag（随机访问）
  - Array，Vector，Deque（虽然假连续，但封装后暴露出的使用方式是连续的）
- 其中，hashtable的bucket是链表，需要看链表是以什么方式实现的，确定其是单向的还是双向的

- 5种类型的类层次结构如下

![image](https://user-images.githubusercontent.com/106053649/177180193-39a2516e-a3b2-47df-9023-2e113b2e33bb.png)

- 同样的算法，对于不同的iterator_category，会调用不同的子函数实现功能，性能相差极大

- 算法的参数对迭代器类型没有强制要求，但会给出暗示（通过模板参数的命名）

- ```c++
  template<class RandomAccessIterator>
  inline void sort(RandomAccessIterator first, RandomAccessIterator last){
      //...
  }
  ```

- 模板参数表示随意给什么类型，命名也随意（一般用T），这里用明明来暗示迭代器类型



#### 算法示例accumulate

- 有2种版本，一种是默认版本

- ```c++
  template<class InputIterator, class T>
  T accumulate(InputIterator first, InputIterator last, T init){
      for(; first != last; ++first){
          init = init + *first;	//将初值累加到每个元素上返回
      }
      return init;
  }
  ```

- 一种是给函数功能的版本（全部函数，仿函数都可以），着重关注该实现方式（有助于自己理解并应用）

- ```c++
  template<class InputIterator, class T>
  T accumulate(InputIterator first, InputIterator last, T init, BinaryOperation binary_op){
      for(; first != last; ++first){
          //对元素累计算至初值上，每作用完一个元素init会变化
          init = binary_op(init, first);	
      }
      return init;
  }
  ```

- BinaryOperation表示两个操作数的函数，binary_op(init, first)给出了仿函数实际调用的形式

  - binary_op是一个函数对象，重载了()，以此来看binary_op(init, first)能加深理解
  - 类比对一个对象重载+，若调用：对象名+某个类型，表示对该对象执行重载+函数的内部操作
  - 这里类似，无非是一个不需要其他操作数的重载，表示执行重载()函数的内部内容，就会把第二个小括号的参数传进去进行特定操作（实现了函数功能）



#### 仿函数functors

- 仿函数为算法服务，为算法提供一些特定的操作准则（算法的第二版本）
- 必须重载()，自己写的functor如果想要融入STL，必须选适当的类型继承
  - 这样才能回答Function Adapter（函数适配器）提出的问题



#### 适配器Adapter

- 函数适配器，容器适配器，迭代器适配器
- 实现适配，比如A是使用者，B隐藏在幕后，A就是Adapter，需要取用B的功能
  - 要么A继承B，要么A组合B，适配器的实现都是组合的方式
- 容器适配器
  - stack和queue组合了一个deque

- 函数适配器
  - 比如less<int>()，原先是比较参数x和y的大小
  - 通过bind2nd(less<int>(), 40)，绑定了第二个参数y为40，具体功能为比较x和40大小，小于40
  - 再比如not1，如not1(bind2nd(less<int>(), 40))，继续修饰上文的语义，整体功能为大于40

- 迭代器适配器
  - reverse_iterator（反向迭代器，以普通迭代器改造而得到）



#### 函数适配器中的bind（单独拿出来讲）

- std::bind可以绑定
  - 函数
  - 函数对象
  - 成员函数，占位符1必须是某个object地址
  - 成员变量，占位符1必须是某个object地址 

- bind函数示例如下：

- ```c++
  using namespace std::placeholders;
  
  double my_divide(double x, double y){
      { return x / y; }
  }
  
  auto fn_five = bind(my_divide, 10, 2);			//	returns 10 / 2
  cout << fn_five() << '\n';						//	5
  
  auto fn_half = bind(my_divide, _1, 2);			//	return x / 2,只绑定了第二个参数
  cout << fn_half(10) << '\n';					//	5
  
  auto fn_invert = bind(my_divide, _2, _1);		//	return y / x,两个参数都没绑定，并且交换了顺序
  cout << fn_invert(10, 2) << '\n';				//	0.2
  
  auto fn_rounding = bind<int>(my_divide, _1, _2)	//	return int(x / y),两个参数都没绑定,且指定了返回值类型
  cout << fn_rounding(10, 3) << '\n';				//	3
  ```

- bind成员函数以及成员变量示例如下：

- ```c++
  using namespace std::placeholders;
  
  struct MyPair{
  	double a, b;
      double multiply() { return a * b; }
  };
  
  MyPair ten_two{10, 2};								//member function其实有个参数:this
  													
  auto bound_memfn = bind(&MyPair::multiply, _1);		//	return x.multiply(),未绑定第一个参数
  cout << bound_memfn(&ten_two) << '\n';				//	20,传第一个参数，即对象地址,&MyPair::multiply表示取成员函数地址
  
  auto bound_memdata = bind(&MyPair::a, &ten_two);	//	return ten_two.a	
  cout << bound_memdata() << '\n';					//	10
  
  auto bound_memdata2 = bind(&MyPair::b, _1);			//	return x.a	
  cout << bound_memdata2(&ten_two) << '\n';			//	10,传第一个参数，即对象地址
  ```



#### 元组tuple

- 声明、初始化

- ```c++
  tuple<string, int, int, complex<double>>t;		//声明
  tuple<int, float, string>t1(41, 6.3, "nico");	//声明加初始化
  auto t2 = make_tuple(22, 44, "stacy");			//实参自动类型推导
  ```

- 取元素，赋值

- ```c++
  get<0>(t1);					//取出tuple对象t1的第0个元素
  get<1>(t1) = get<1>(t2)		//将t2的第1个元素赋值给t1的第1个元素
  ```

- tuple利用可变参数作为模板参数实现，将一组数据分为1和n-1，并一直递归

- ```c++
  template<typename Head, typename... Tail>
  class tuple<Head, Tail...> : private tuple<Tail...>{		//继承自己的尾部部分部分，如一包5个，就继承4个的自己
      typedef tuple<Tail...> inherited;
  public:
      tuple(){}
      tuple(Head v, Tail... vtail):m_head(v), inherited(vtail...){}	//有参构造，取出第一个元素给成员变量m_head,把剩下部分交给继承,递归
      typename Head::type head() { return m_head; }//这句话有问题，对照示例传进来Head为int，int没有什么type的定义
      inherited& tail() {return *this}			 //在后面的视频里，改为Head head(){ return m_head; }
  protected:
      Head m_head;		//Head的字段，即元组的第一个元素，继承了剩下的部分
  }
  ```

- 其中，inherited(vtail...)会呼叫基类的构造函数并赋予参数（通过initalization list初始化列表的方式）

- 递归到头必然会有一个空的tuple，所以设计一个偏特化的空模板类，其中还有一个只有一包可变参数的基类（没有一开始的一个）：

- ```c++
  template<typename... Values> calss tuple;	//只有一包参数基类，供2个模板参数的tuple继承
  template<>class tuple<>{};					//空类
  ```

- 使用示例：

- ```c++
  tuple<int, float, string> t(41, 6.3, "nico");
  t.head()			//取得41
  t.tail();			//取得剩下的一包，即6.3与"nico"的组合
  t.tail().head();	//取得6.3
  ```

  

#### type traits（GNU 2.9）后续变了但思想一致

![image](https://user-images.githubusercontent.com/106053649/177235503-633f70cf-a5b8-48bc-9609-55d77e930a55.png)

- 回答关于类型的一些问题，默认都把它设成false，如提问是否有不重要的拷贝构造函数，默认是_false_type即没有不重要，表示重要

- 对于简单的基本类型有其特化版本（如基本类型int，double），其三大件都是不重要的，不需要自己写特别的函数

- ```c++
  __type_traits<Foo>::has_trivial_destructor		//提问方式，对Foo类的析构函数是否重要提问（编译通过即得到了答案）
  ```



#### cout

- 是一个对象（不可能直接拿类来用，肯定是对象）
