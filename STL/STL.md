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



#### 容器list（GNU2.9）

- list类大小为4，只有一个list_node*类型的指针
- list_node类内，有两个指针prev和next以及一个模板类型T的data
- 在list类内需要对Iterator的操作符重载
  - 如*Iterator就是取node的data（实际是node指针指向的list_node类内的data）

- 是双向环状的，为了符合前闭后开区间，增加了一个空的node作为end()



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

- Iterator_Traits能区分传进来的迭代器是class还是普通的指针，相当于一个中间层
