# C++面向对象编程（下）

#### 模板特化

- 针对模板，面对独特的类型要做特殊的设计（比如针对某个模板参数，有更高效的仅适合该类型的实现方式）

![image](https://user-images.githubusercontent.com/106053649/177123860-7d97bed9-8d4a-4e5b-abb3-c0d9017d99a5.png)

- 针对hash这个struct，模板参数为key（随意指定无所谓），当key为char，int，long时，有其特化部分

#### 模板偏特化

- 个数的偏
  - 如有多个模板参数，指定其中部分参数单独设计class

- 范围的偏
  - 指定的模板参数为T，为class C<T*>单独设计



#### 虚函数、虚表

- 只要一个类有虚函数（无论几个），对象大小都会多4（多一个虚函数指针）
- 虚函数指向虚表，虚表里放的都是函数指针，指向虚函数所在的位置
- 对象模型：
  - 如果类有虚函数（定义和重载），最开始是一个vptr（大小为4）
  - 之后是类内的各个成员变量

- 虚函数调用机制

- ```c++
  class Ibase{
  public:
      virtual void func() = 0;
  };
  class child : public Ibase{
      void func(){
          //...
      }
  };
  int main(){
      Ibase* ptr = new child();
      child->func();
  }
  ```

- 调用func()，实际上是ptr（其实就是this）找到对应对象，通过其指向虚函数表的vptr，找到对应的虚函数，调用func()

- ```c++
  (*(ptr->vptr)[n])	//实际调用的语句，编译器这样干的
  ```

- ptr->vptr就是找到其虚函数表，n表示第几个函数，编译器会确定，拿到的是函数指针，通过*解引用就可以调用该函数



#### this指针

- 通过对象调用函数，对象的地址就是this指针



#### new、delete补充

- new和delete是表达式，表达式的行为不能变，不能重载（具体流程不能变）
- 但分解后所调用的函数可以重载（operator new，operation delete及[]版本）
