---
date: 2025-03-24
title: C++多态：除了virtual我们还有啥？
tags:
  - C++
description: 多态≠虚函数（）就我个人目前的了解尝试梳理C++目前各种多态接口的实现方式
---

## 前言

C++演进了几十年，现在除了虚函数也有其它一些做多态接口的方法。虚函数虽然有它实用的地方，但上来不管三七二十一先搞个纯虚函数+虚析构有些时候并不是最好的办法。

我想就我自己的理解写写都有哪些方法，以及它们各自的优缺点。水平有限，写到哪算哪。

<!-- ## TL;DR

写文章其实也是学习的过程。这次写作总得来说有以下几点体会：
* CRTP（含用deducing this改进后的版本）如果不用来做Mixin，而是做网上常见的那种“静多态”，完全是多此一举，可以用concept+简单的模板函数接口代替
*  -->

## 多态

首先，当我们谈论多态接口的时候，我们究竟在讨论什么东西？

照搬wikipedia的定义的话，**多态（Polymorphism）** 指的是为不同数据类型的实体提供统一接口，或者说用同一套符号处理多种不同的类型。按调用形式可以分成：

- ad hoc多态，为固定的一组类型提供统一的接口，在C++中对应**函数重载**
- 参数化多态，使用抽象的符号来代表可能的所有类型，在C++中对应**模板**相关的泛型机制
- 子类型多态，使用继承公共父类的方法来提供统一接口，在C++中对应虚函数相关的OOP设施
- ...

另一个我们关心的问题是，既然要处理不同的类型。那么如何确定对给定的数据，要调用哪种类型的处理方法呢？也就是**分派（dispatch）** 时机的问题，按这个分类，编译时能确定的就叫**静态分派（static dispatch）**；反之要到运行时才能确定的叫**动态分派（dynamic dispatch）**。重载和模板是很典型的静态分派实现的多态，而虚函数（不考虑devirtualize）则是动态分派实现的多态。

可以看到多态的内涵还是很广的，并不是说只有用了继承+虚函数才能叫多态，只要能通过某种机制对不同类型overload/override同个接口就能算是多态。是否使用虚函数来实现多态则是需要我们另外考虑取舍的。

## Basics：虚函数

不管怎么说，还是先看看最基础的虚函数多态：

```c++
struct IRun {
    virtual void run() = 0;
    virtual ~IRun() = default; // don't forget it!!!
    // virtual std::unique_ptr<IRun> clone() const = 0; // slicing is evil
};

struct Dog : IRun {
    void run() override { std::println("Dog is running"); }
};

struct Cat : IRun {
    void run() override { std::println("Cat is running"); }
};

void make_animal_run(IRun &animal) { animal.run(); }

int main() {
    /// ad-hoc like
    Dog dog;
    Cat cat;
    make_animal_run(dog);
    make_animal_run(cat);

    /// type erasure
    std::vector<IRun *> animals;
    animals.push_back(new Dog());
    animals.push_back(new Cat());
    for (auto animal : animals) {
        animal->run();
    }
}

// OUTPUT:
// Dog is running
// Cat is running
// Dog is running
// Cat is running
```

代码很简单：定义了一个抽象类作为接口，要求继承它的子类必须实现`run`方法，然后`Dog`和`Cat`分别override了`run`方法以实现多态。`main`里面是两种常见的使用`IRun`接口的办法：

- 把`IRun &`作为函数入参用来要求入参必须实现`run`方法
- 把满足`IRun`要求的对象装载到`std::vector<IRun *>`中，统一调用接口定义的`run`方法

这俩类的内存布局大致如下：

![](https://s2.loli.net/2025/03/25/5fudYyewKa6MVx4.png =50%)

使用虚函数来做多态的限制在于：

并非**零开销抽象**：

- 如果我们不需要类型擦除，只需要前半边类似ad-hoc多态的效果，那么虚表跳转的开销完全是多余的。虚表的存在还会阻碍函数内联，devirtualize虚函数对编译器来说难得多。
- 为子类添加虚接口约束（比如给`Cat`添加`IRun`约束）会改变内存布局，复杂继承关系会让结构的内存布局膨胀，进而影响结构体访问的性能。

> 说到底是虚函数这一套动态分派和接口继承强绑定的设计不太灵活。如果只需要其中部分功能，不得不为没用到的能力付出代价，不符合**don't pay for what you don't use**的原则。

虚函数相关语言机制的问题：

- 由于对象切片的存在，不能使用值语义做多态。想使用虚函数多态必须借助指针或引用，从而不得不引入生命周期与RAII相关设施。
- 为了类似`std::unique_ptr<IRun>`的RAII设施正确，必须记得虚析构和提供虚拷贝接口
- 使用侵入式的继承设计势必引入**脆弱基类**问题，继承树膨胀后，接口的任何变动都需要修改无数的子类。
- ...

> 在动多态部分会再讨论其他的问题

由于虚函数现存的一些问题，各路C++大神们发明了不少替代方案，提供与虚函数方案不同的trade off.

## 静多态

有些时候是用不到`std::vector<IRun *>`这样的类型擦除异构容器的，只是借虚函数的机制方便定义统一接口。这种情况下具体调用派生类的哪个方法实际上是编译时就能知道的，我们完全可以静态分派方法从而避免虚函数跳转的开销。这样的方案我们称之为“静多态”，一般都是用模板技术在编译期实现的。

### CRTP

最经典的静多态方法就是[CRTP](https://en.cppreference.com/w/cpp/language/crtp)（奇异递归模板模式），在写法上，需要接口模板接受子类作为模板参数然后被子类继承，也是“奇异递归”这一名称的由来。还是前面的例子，改用CRTP的话写法如下：

```c++
template <typename Derived>
struct IRun {
    /// virtual void run() = 0;
    void run() { static_cast<Derived *>(this)->run_impl(); }

    /// virtual void name() const;
    void name() const { static_cast<Derived const *>(this)->name_impl(); }

    // fallback for name
    void name_impl() const { std::println("IRun::name (fallback)"); }
};

struct Dog : IRun<Dog> { void run_impl() { std::println("Dog is running"); } };
struct Cat : IRun<Cat> { void run_impl() { std::println("Cat is running"); } };
struct Pig : IRun<Pig> {};

template <typename T>
void make_animal_run(IRun<T> &animal) { animal.run(); }

int main() {
    Dog dog;
    Cat cat;
    Pig pig;
    make_animal_run(dog);
    make_animal_run(cat);
    // make_animal_run(pig); // This will not compile, as Pig does not implement run.

    dog.name(); // safe to call fallback
}

// OUTPUT:
// Dog is running
// Cat is running
// IRun::name (fallback)
```

利用模板类的成员函数被用到时才实例化的特性，我们可以在CRTP父类的成员函数里随意使用子类的类型而不产生编译错误。这样一来就可以实现在编译期分派方法，消除虚函数的开销。

:::details 思考：CRTP的接口和实现能同名吗？

上面CRTP父类的接口和子类实现分别命名成`xxx`和`xxx_impl`主要是为了避免子类没实现时会导致无限递归以及方便提供默认实现。实际上借助`static_assert`和`if constexpr`也（勉强）能实现类似的效果，同时保证接口和子类实现使用同一个名字，而不是分成`xxx`和`xxx_impl`，看上去更清爽些。

```c++
#define CRTP_DERIVED_HAS(func) (!std::is_same_v<decltype(&Derived::func), decltype(&std::remove_cvref_t<decltype(*this)>::func)>)

template <typename Derived>
struct IRun {
    /// virtual void run() = 0;
    void run() {
        static_assert(CRTP_DERIVED_HAS(run), "Derived class must implement run()");
        static_cast<Derived *>(this)->run();
    }

    /// virtual void name() const;
    void name() const {
        if (CRTP_DERIVED_HAS(name)) {
            static_cast<Derived const *>(this)->name();
        } else {
            std::println("IRun::name (fallback)");
        }
    }
};

struct Dog : IRun<Dog> {
    void run() { std::println("Dog is running"); }
};

struct Cat : IRun<Cat> {
    void run() { std::println("Cat is running"); }
};

struct Pig : IRun<Pig> {};

template <typename T>
void make_animal_run(IRun<T> &animal) { animal.run(); }

int main() {
    Dog dog;
    Cat cat;
    Pig pig;
    make_animal_run(dog);
    make_animal_run(cat);
    // make_animal_run(pig); // This will not compile, as Pig does not implement run.

    pig.name(); // safe to call fallback
}
```

不过上述实现并不能很好处理含有重载成员函数或者模板成员函数的情况。

:::

### CRTP with Deducing This

C++23最重要的特性之一就是这个[显式对象参数（Deducing This，推导this）](https://devblogs.microsoft.com/cppblog/cpp23-deducing-this/)了，它允许我们像Python那样显式地把对象自身声明为成员函数的参数：

```c++
struct Foo {
    template <typename Self>
    void increment(this Self&& self) {
        self.count++;
    }

    int count = 0;
};
```

> ~~这里**Self**和**self**让我想起了rust~~

这个特性出发点是提供完美转发`this`的能力，这样就不必实现本质trivial的各种cvref版本的成员函数了。但这里有个问题，继承关系里面`this`类型如何推导？

```c++
struct Bar : Foo {};

Bar{}.increment(); // what's the type of self?
```

其实如果了解[统一调用语法(UFCS) P3021](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p3021r0.pdf)的话，比较好理解这里的语义。我们希望`foo.bar(x)`和`bar(foo, x)`是相同的（因为实现上确实如此）。对于`Bar{}.increment()`，语义上应该相当于`increment(Bar{})`，而`increment`本身就是个普通的模板函数，那自然第一个万能引用参数应该被推导为`Bar &&`。

这就有意思了——利用Deducing This特性，我们也可以在父类的成员函数里面使用子类的类型和引用！这不就是前面CRTP通过递归模板想要做的么？

使用新特性，我们可以简化上一节中静多态的例子：

```c++
struct IRun {
    /// virtual void run() = 0;
    template <typename Self>
    void run(this Self &&self) { self.run_impl(); }

    /// virtual void name() const;
    template <typename Self>
    void name(this Self const &self) { self.name_impl(); }

    // fallback for name
    void name_impl() const { std::println("IRun::name (fallback)"); }
};

struct Dog : IRun { void run_impl() { std::println("Dog is running"); } };
struct Cat : IRun { void run_impl() { std::println("Cat is running"); } };
struct Pig : IRun { void name_impl() const { std::println("Piggy"); } };
```

这样一来，我们把模板从类上移到了成员函数上，子类直接继承`IRun`就好了。我们消除了模板递归，现在大家有一个公共的父类，而不是像CRTP的`IRun<T>`那样各不相同。原先复杂的嵌套CRTP在deducing this的框架下也很容易写，就简单继承静多态接口就行，而不用给上一级接口写转发。

#### 新特性，新坑

然而统一接口类型也不完全是好事，考虑：

```c++
void make_animal_run(IRun &animal) { animal.run(); }
void get_animal_name(IRun const &animal) { animal.name(); }

int main() {
    Dog dog;
    make_animal_run(dog); // compile error: run_impl not found
    Pig pig;
    get_animal_name(pig); // OUTPUT: IRun::name (fallback)
}
```

由于现在接口统一了，很容易想当然地写出`void make_animal_run(IRun &animal)`这样的接口。但是说到底我们还是静态分派，`IRun`接口里没有虚表，把子类对象传给`IRun &`只会切片并不会产生运行时多态，还是只有父类的信息。

正确的做法是借助模板，进一步还可以加上[Concept](https://en.cppreference.com/w/cpp/language/constraints)来约束：

```c++
template <typename T>
concept IRunable = std::derived_from<T, IRun> && requires(T t) { t.run_impl(); };

template <IRunable T>
void make_animal_run(T &animal) { animal.run(); }
```

为了防止有人试图用`IRun`做类型擦除，可以用concept进一步给父类的接口做约束：

```c++
template <typename Self, typename InterfaceT> // [!code ++]
concept implemented = !std::is_same_v<std::remove_cvref_t<Self>, InterfaceT>; // [!code ++]

struct IRun {
    template <typename Self>            // [!code --]
    template <implemented<IRun> Self>   // [!code ++]
    void run(this Self &&self) { self.run_impl(); }

    template <typename Self>            // [!code --]
    template <implemented<IRun> Self>   // [!code ++]
    void name(this Self const &self) {
        self.name_impl();
    }

    void name_impl() const { std::println("IRun::name (fallback)"); }
};

void get_animal_name(IRun const &animal) { animal.name(); } // No matching member function for call to 'name'
```

> 用`static_assert`其实也行，不过用concept的话clangd会在调用的位置报错，静态断言报错位置在实现的地方，稍微差一些。

实话说，和原始的CRTP相比甚至还变得更麻烦了。对库编写者来说得写更多的模板，好处是在使用者的角度提供了更好的接口。

> Deducing This相比于CRTP还有两个限制：
>
> - 推导this，前提得有this，所以只能在非静态成员函数里头用。CRTP则没有限制，可以给子类添加静态成员函数。
> - CRTP因为基类类型各不相同，所以可以按照类型分别做处理。一个实用的例子是给所有子类添加一个该子类instance计数器。这种需求不能用deducing this实现。

### Re-consider: 有必要用CRTP实现静多态吗？

我们回过头看看CRTP和Deducing This静多态，很容易发现其实所有的静态分派都是通过那个函数模板实现的。

```c++
/// CRTP
template <typename T>
void make_animal_run(IRun<T> &animal) { animal.run(); }

/// Deducing This
template <typename T>
concept IRunable = std::derived_from<T, IRun> && requires(T t) { t.run_impl(); };

template <IRunable T>
void make_animal_run(T &animal) { animal.run(); }
```

这样一看在子类继承`IRun`，只是为了告诉用户：这个类实现了`IRun`接口。欸，但这又有个问题：这两个方案里，等价于纯虚函数的强制实现接口如果没被子类实现，不会立即报编译错误，而是得等到相关调用发生时才报错。

```c++
template <typename Self, typename InterfaceT>
concept implemented = !std::is_same_v<std::remove_cvref_t<Self>, InterfaceT>;

struct IRun {
    /// virtual void run() = 0;
    template <implemented<IRun> Self>
    void run(this Self &&self) { self.run_impl(); }
};

struct Pig : IRun {
    // run_impl not implemented. // But NO error here :(
};

Pig pig;
pig.run(); // compile error: constraint not satisfied
```

这样一来这个继承的意义其实也并不大。那么如果继承关系并不起什么实际作用，我们为啥不直接去掉又臭又长的模板代码，直接把那个函数模板作为多态接口就好了呢？这一点在deducing this的例子里尤其明显：我们实现了`IRun`作为子类的接口约束，然而在函数模板这块又实现了`IRunable`约束。同一个约束重复了两次，违反了DRY原则，一定有哪里不对！

我们试着直接用concept约束重写前面的静多态：

```c++
#define impl_for(Interface, T) static_assert(Interface<T>, #T " must implement " #Interface)

template <typename T>
concept IRun = requires (T t) {
    { t.run() } -> std::same_as<void>;
};

struct Dog { void run() { std::println("Dog is running"); } };
impl_for(IRun, Dog);
struct Cat { void run() { std::println("Cat is running"); } };
impl_for(IRun, Cat);
struct Pig { void name() const { std::println("Piggy"); } };
// impl_for(IRun, Pig); // will fail

template <IRun T>
void make_animal_run(T &animal) { animal.run(); }

int main() {
    Dog dog;
    Cat cat;
    Pig pig;
    make_animal_run(dog);
    make_animal_run(cat);
    // make_animal_run(pig); // This will not compile, as Pig does not implement run.

    pig.name(); // Piggy
}
```

这看上去舒服多了：

- 简单直观的用户接口，没有复杂的模板和继承
- 非侵入式：用concept声明接口而不需要任何继承去破坏原有的类
- 顺便也解决了CRTP中接口和实现得区别命名的问题
- 类定义后直接静态断言，不必等到`make_animal_run`实例化，提前暴露错误

唯一的问题可能是`requires`语句描述的接口可读性比直接一个接口类要差不少，还得学习相关的语法，但除此之外可就都强太多了。

但是现在只相当于实现了纯虚函数的部分，如果我们想要为这个方案引入默认实现呢？那当然也有办法：

```c++
#define impl_for(Interface, T) static_assert(Interface<T>::v, #T " must implement " #Interface)

template <typename T>
struct IRun : T {
    /// 可惜不能在类里面声明concept...
    constexpr static const bool v = requires(T t) {
        { t.run() } -> std::same_as<void>;
    };

    IRun(T const &) requires v {}

    void name() const {
        if constexpr (requires(T const &t) { { t.name() } -> std::same_as<void>; }) {
            this->T::name(); // Call the "derived" class's name() method if it exists
        } else {
            std::println("Default name"); // fallback
        }
    }
};

struct Dog { void run() { std::println("Dog is running"); } };
impl_for(IRun, Dog);

struct Cat {
    void run() { std::println("Cat is running"); }
    void name() const { std::println("Catty"); }
};
impl_for(IRun, Cat);

struct Pig {
    // void run() { std::println("Pig is running"); }
    void name() const { std::println("Piggy"); }
};
// impl_for(IRun, Pig); // will fail

template <typename T> requires IRun<T>::v
void make_animal_run(T &animal) { IRun(animal).run(); }

template <typename T> requires IRun<T>::v
void get_animal_name(T &animal) { IRun(animal).name(); }

int main() {
    Dog dog;
    Cat cat;
    Pig pig;

    make_animal_run(dog);
    make_animal_run(cat);
    // make_animal_run(pig); // compiles error: constraints not satisfied

    get_animal_name(dog); // Defaulr name
    get_animal_name(cat); // Catty
}
```

为了支持默认实现，接口定义的代码还是复杂了不少，不过总体上还是保持了上一版实现的优势：非侵入，提前静态断言接口发现实现错误，不需要将接口和实现分别命名。

> 接口和实现的关系与CRTP相比完全倒过来了

### Mixin: 你应该用CRTP干的事

如此看来在Concept和requires进入标准之后，确实不太有用CRTP（以及使用推导this的变种）实现仿虚函数接口的静多态的必要了。那CRTP是不是就没用了？

那倒也不是，说到底CRTP本质是给我们提供了在父类里用子类方法实现功能的能力。我们利用CRTP可以很方便地给子类mixin一些特定的能力。像标准库的`std::enable_shaderd_from_this`就是经典的用CRTP来做mixin。这里列举一些CRTP的用例：

#### 根据前缀自增实现后缀自增

```c++
struct AddPostInc {
    template <typename Self>
    auto operator++(this Self &&self, int) {
        auto tmp = self;
        ++self;
        return tmp;
    }
};

struct Foo : AddPostInc {
    using AddPostInc::operator++;
    Foo &operator++();
};
```

#### 自动实现单例模式

因为需要用到静态成员，所以不能用推导this

```c++
template <typename T>
struct Singleton {
    static T &getInstance() {
        static T instance{_{}};
        return instance;
    }

    Singleton(const Singleton &) = delete;
    Singleton &operator=(Singleton) = delete;

protected:
    struct _ {};
    Singleton() = default;
};

struct Logger : Singleton<Logger> {
    Logger(_) {} // 禁止从外部构造
    // ...
};

auto &logger = Logger::getInstance();

// 至于该不该用单例模式，这就是另一个问题了...
```

#### 统计子类型实例个数

因为需要各类型分开统计，因此也不能用推导this

```c++
template <typename T>
struct InstanceStat {
    inline static std::atomic<std::size_t> count = 0;
    InstanceStat() { ++count; }
    ~InstanceStat() { --count; }
};

struct Foo : InstanceStat<Foo> {};

int main() {
    Foo a;
    {
        Foo b;
        std::cout << Foo::count; // 2
    }
    std::cout << Foo::count;     // 1
}
```

## 动多态

上述利用模板技术实现的静多态虽然避开了虚表跳转的开销，但也因此损失了把同一接口的对象塞到同一个异构容器（例如`std::vector<IRun *>`）里面的能力。这也正常，为了把不同的子类塞到同一个容器，那么类型必须统一，于是装载子类对象的过程在编译期损失了具体的类型信息。为了在运行时能正确调用子类的方法，必须得把类型信息通过某种方式存到运行时，供程序查阅。在虚函数这套方案里面，这个类型信息就由虚表和虚表指针来承载。

无论如何都需要运行时信息的话，我们能不能做得比虚函数更好呢？

### Recap: 虚函数

再次回顾一下虚函数的方案，这次我们用个绘制形状的例子：

```c++
struct Shape {
    Shape() = default;
    virtual ~Shape() = default;
    virtual void draw() = 0;
    virtual std::unique_ptr<Shape> clone() const = 0; // virtual clone to avoid slicing
};

struct Circle : Shape {
    explicit Circle(int radius) : radius_(radius) {}
    void draw() override {
        std::println("Drawing Circle with radius: {}", radius_);
    }
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Circle>(radius_);
    }
private:
    int radius_;
};

struct Square : Shape {
    explicit Square(int side) : side_(side) {}
    void draw() override {
        std::println("Drawing Square with side: {}", side_);
    }
    std::unique_ptr<Shape> clone() const override {
        return std::make_unique<Square>(side_);
    }
private:
    int side_;
};

std::vector<std::unique_ptr<Shape>> copy(const std::vector<std::unique_ptr<Shape>>& shapes) {
    std::vector<std::unique_ptr<Shape>> copies;
    copies.reserve(shapes.size());
    for (const auto& shape : shapes) {
        assert(shape != nullptr);
        copies.emplace_back(shape->clone());
    }
    return copies;
}

int main() {
    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.emplace_back(std::make_unique<Circle>(5));
    shapes.emplace_back(std::make_unique<Square>(10));
    auto shapes_copy = copy(shapes);
    for (const auto& shape : shapes_copy) {
        shape->draw();
    }
}
```

这也是个经典的案例了，我们引入了成员变量补上了虚拷贝，总体上更贴近实际的场景。因为需要实用指针/引用语义来获得虚函数的多态性，因此我们引入RAII设施来存放所有的形状对象。

即便需要类型擦除的多态，虚函数的方案还是有以下的问题：

- 无法使用值语义来做多态，导致大量零碎的内存申请
- 指针语义导致接口调用需要两次间接跳转（第一次指针访问父类，然后再虚表指针访问方法），cache不友好
- 如果有第三方接口相容的图形类，没法将它添加到这套继承树里

### Pimpl：虚函数，但是值语义

为了改善指针语义的问题，其实就是需要把虚函数从接口上隔离开，但同时不能影响逻辑。这种需求很容易就想到COM技术，我们用[Pimpl模式](https://zhuanlan.zhihu.com/p/458947637)把具体实现给隔离了不就好了？

:::danger 施工中
没写完QAQ
:::
