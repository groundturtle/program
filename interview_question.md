<!--
 * @Author: avert-win
 * @Date: 2022-11-22 19:40:21
 * @LastEditTime: 2022-11-22 21:20:12
 * @FilePath: \ProgramDaily\interview_question.md
 * @Description: 简介
 * @LastEditors: avert-win
-->

#### inline内联函数

内联函数，是一种以代码膨胀为代价、省去函数调用的栈帧开辟、结果返回等过程的时空消耗的方法。

原理：在**编译期间**将**函数代码直接嵌入到主程序中**，而不另外生成库、运行时通过地址调用。

- inline函数改变需要重新编译，因为是在任何需要调用该函数的地方复制一段代码（而不是从库中提取）。

- 是否内联完全取决于编译器，你所加上的关键字只是一种可有可无的建议；一般复杂的（如含循环、switch、递归等）函数不会内联。

- 内联能使编译器同时看到更多源代码信息，因此**增加了优化的可能**：两个最优局部的组合，可能具有更优的解法。

- inline函数使得重复包含头文件且通过编译成为可能，但也有可能导致未定义行为。

- 在类内实现（不是声明）的函数相当于在类外实现、加了`inline`关键字。

- 虚函数只有在确定应当使用哪个类的实现时可内联，因此由实际对象调用时可内联，而由指针或引用调用时不可内联。
> 根据内联函数的原理，**需要在编译期间确定函数代码所在**、并且将其复制过来，而此时虚函数表虽然（可能）已经构建，但由于程序尚未开始运行，对象指针有实体，因此无法通过对象指针找到对象实体、进而找到该对象的虚函数表，也就无法通过虚函数表确定究竟使用哪个函数代码。

#### 虚函数、多态和相关关键字

虚函数的通常应用场景是，**使用一个基类的指针指向特定的子类**，并通过指针调用函数，**若子类未重新实现则默认使用基类的实现**。
> - 通常一个类的多个对象共用一个虚函数表，子类的虚函数表继承自基类（尽管基类对象没有调用派生类虚函数的权限，依然需要一个表）；
> - 每个对象在实例化时，都会获得一个虚函数表的地址，因此通过基类的指针可以获得派生类的虚函数表（指针指向对象，从中获取虚函数表的地址，与指针类型无关）。

实现原理：
- 在**编译期间**，为每个具有虚函数成员的类维护一个虚函数表，其中按某种顺序存储虚函数的各个实现版本的地址；
- 通常一个类的全部对象**共用一个虚函数表**，子类的虚函数表继承自基类（尽管基类对象没有调用派生类虚函数的权限，仍然需要一个虚函数表）；
- **对象在实例化时获得一个虚函数表的地址**，因此通过基类的指针可以获得派生类的虚函数表（指针指向对象，从对象中获取虚函数表的地址，与指针类型无关）。
- 据此，多态是在运行期动态绑定的（尽管虚函数表是在编译期创建的）。

由于某些基类并不适合生成对象，因此定义**抽象类**：含有**纯虚函数**的类，该类规范接口、要求派生类提供必要的功能，但并不提供标准化（通用）方法，纯虚函数必须由子类自己实现。
> 一个子类可以将基类的纯虚函数再次定义为纯虚函数，因此该子类成为一个新的抽象类。

- `virtual`：声明虚函数，可在子类中（重新）实现。

- `final`(c++11)：用在类或者类的成员函数声明之后，使其不可继承（虚函数不可实例化）。

```c++
virtual void func();            // 虚函数声明，声明者必须实现
virtual void func1() = 0     // 纯虚函数声明，子类必须实现或再次声明为纯虚函数

virtual void func2() final;      // 不可继承函数
class oxygen final : public air{};      // 不可继承类
```

#### 宏和函数

宏定义代码块有时候可以充当没有返回值的函数，类似于内联函数，但**没有类型检查**、**不能定义递归**，且简单的代码替换可能造成一些意想不到的错误，如向`#define CAL(x) (x>0 ? x++:x)`传入`i++`，可能会造成未定义行为的出现、并且至少执行多次`i++`（你期望的只是一次）。

由于宏能省去函数调用的开销，嵌入式系统中经常用宏来代替函数。但代码膨胀和不安全的问题仍然存在。

> `assert()`就是一个宏，而非函数。

```c++
#define MACROFUNC(a,b) \
(\
    cout<<a>b ? a:b<<endl;\
)
```

_2022-11-12, 21:20_

------------------

#### static

作用：

1. 使变量存储在静态区，在`main`函数运行前分配空间，生命周期贯穿整个程序运行周期。但是并**不改变作用域**。

2. 改变函数符号的链接属性（对于本文件，从`global`变为`local`），成为弱符号，使其只能在当前源文件中被使用。
    > 链接符号一共有三种：`global`, `external`, `local`。

3. 修饰成员变量（函数），使所有类生成的对象共用一个该变量（函数），不需要实际生成对象就可以访问该成员（通过类名）。
    > 静态成员函数没有`this`指针，不与具体对象关联；
    > 在类的静态成员函数中，只能访问类的静态成员（没有`this`，无法确定应访问哪个对象的成员）；

#### const

- `const`除了用于变量外，还可以用于修饰成员函数；
    > 修饰对象时，实际上修饰的是该对象的`this`指针；
    > 修饰成员函数时，修饰的是该函数的隐含参数`this`的类型：使其成为`const`。

- `const`变量无法作为非`const`参数调用函数。

```cpp
class T{
public:
    // const的作用是使该函数的this指针成为`const`类型。
    void get() const;       
    void put();
}

int main(){
    T ob;
    const T oc;     // const修饰对象，修饰的是this指针。
    ob.get();       // ob的this指针非const，作为const参数，可行；
    oc.get();       // oc的this为const，参数类型亦为const，可行；
    oc.put();       // 报错：无法将const类型变量作为非const参数调用函数。
}
```

#### c/c++结构体，类成员权限

- C中，`struct`仅是一个数据集合，不能有函数、不能初始化变量、不能继承、没有除`public`以外的权限；定义变量时必须加关键字`struct`修饰。

- C++中，`struct`可以有函数、可以继承、可以直接初始化，权限模式拥有`public`、`protect`、`private`，默认为`public`；

- C++中结构体和类的区别：
    - 默认权限不同；默认继承权限不同
    ~~- 本质上来说，`class`属于引用类型，其实例是在managed heap中保存的，栈上只保存地址；而`struct`属于值类型~~

- `class`的继承类型：
    1. `public`公有继承，可以访问公有成员和保护成员，且继承的成员权限不变；
    2. `protected`保护继承，所有被继承成员都成为子类的保护成员；
    3. `private`私有继承，所有被继承成员都成为子类的私有成员。
    
    无论哪一种继承类型，子类都只能访问基类的`public`和`protect`成员，无法访问私有成员。可以说`protect`权限就是为继承而设置的。

#### 初始化列表

初始化列表减少了一次构造函数的调用，更为高效，并且在某些情况下必须使用：
1. 常量成员，只能初始化，不能在构造函数中赋值（两者是不同的）。
2. 引用类型成员，必须在定义的时候初始化，并且不能重新赋值（见代码示例）。

        引用类型成员会引入一些奇怪的问题需要解决，如类外变量的生命周期、默认构造函数的不可用（因为必须初始化）等。
        一般不在类内定义引用成员变量。
3. 没有默认构造函数的类（可以手动实现）。

```cpp
class T{
private:
    int& r;
    const int x, y;
public:
    T(int a) : x(a), y(a/2), r(a*3){};
    // 以下构造函数错误，因为不能对常量和引用变量赋值。
/*     T(int a){
        x = a;
        y = a/2;
    }; */
};
```

#### ?友元类和友元函数?

- 友元函数可以是独立的函数，也可以是某个类的成员函数只需要在类的定义中加以声明就能访问该类的所有成员。

- 友元类：该类的所有成员函数都是另一个类的友元函数，单向。

破坏了封装性，只在部分需要运算符重载和共享数据的时候才使用，应尽量避免。

#### ?C实现C++类（封装、继承、多态特性）?

- 封装：在函数中显式引入对象指针；
- 继承：结构体嵌套；
- 多态：根据对象指针类型不同，调用不同函数。

#### using

- using 不但可以引入整个命名空间，也可以只引入该空间的一个成员，因此可以制造方便同时又避免混乱。

- 派生类可以直接使用using来复制基类的全部构造函数。

```cpp
using std::vector;
using std::cout, std::cin;      // 这种连续写法c++17才引入。

class son : public base{
public:
    using base:base;
}
```

\
_2022-11-26, 12:14_

----------------------------


#### enum枚举类型

相比C，C++的枚举类型可以限定作用域，避免不正确使用而编译器又无法识别。

```cpp
enum color{red, green};
enum class Animal {dog, cat};
color blood = color::red;
Animal a = Animal::dog;     // safe use
```