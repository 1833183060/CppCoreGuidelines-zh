# <a name="main"></a>C++ Core Guidelines
August 3, 2020
# <a name="S-introduction"></a>In: 导论
这是一套现代C++(目前是C++17)的核心指南，考虑了未来可能的增强和国际标准化组织技术规范(TSs)。
目的是帮助C++程序员编写更简单、更高效、更易维护的代码。

导论摘要：
* [In.target: Target readership](#SS-readers)
* [In.aims: Aims](#SS-aims)
* [In.not: Non-aims](#SS-non)
* [In.force: Enforcement](#SS-force)
* [In.struct: The structure of this document](#SS-struct)
* [In.sec: Major sections](#SS-sec)

## <a name="SS-readers"></a>In.target: 目标读者

所有C++程序员。这包括可能会考虑c的程序员。
## <a name="SS-aims"></a>In.aims: 目标

本文的目的是帮助开发人员采用现代C++(目前是C++17)，并实现更统一的跨代码库风格。

我们不会妄想这些规则中的每一条都可以有效地应用于每个代码库. 升级旧系统很难. 然而，我们确信，使用规则的程序比不使用规则的程序更不容易出错，也更容易维护。 通常，规则会使开发的初始阶段更快、更容易。
就我们所知，这些规则使代码的性能与采用旧的、更传统的技术的代码一样好或更好；这些规则会遵循零开销原则 (即"你无需为不用的东西付费" 或者说 "当您恰当地使用抽象机制时, 您至少可以获得与使用低级语言结构手工编码一样好的性能").

将这些规则的思想用于新代码，同时在处理旧代码时找机会利用它们，并尽可能接近这些思想。
Remember:
### <a name="R0"></a>In.0: 不要慌!
花点时间去理解一份指导原则对你的程序的影响。
这些指南是根据“超集子集”原则设计的 ([Stroustrup05](#Stroustrup05)).
他们并不是简单地定义要使用的C++子集 ((可靠性、安全性、性能或其他).
而是, 强烈建议使用一些简单的“扩展”，这些“扩展”使得C++最容易出错的特性的使用变得多余，从而可以被禁止(在我们的规则集中)。

这些规则以静态类型安全和资源安全为重点.
因此，它们强调范围检查、避免对“nullptr”解引用、避免悬空指针以及系统地使用异常（通过RAII）的可能性。
为了实现这一点，同时也为了最大限度地减少作为错误来源的晦涩代码，规则还强调简单性，以及在明确指定的接口后面隐藏必要的复杂性。

许多规则是规定性的。（Many of the rules are prescriptive.）
我们对简单地说“不要那样做”而又没有提供替代方案的规则感到不安。

这样做的一个后果是，一些规则只能由启发式检查来支持，而不是精确的和机械的可验证的检查来支持。
其他规则阐明了一般原则. 越是通用的规则，就越是有更加详细和特定的规则提供部分检查。

这些指导方针致力于C++的核心及其使用。
我们预计，大多数大型组织、特定应用领域，甚至大型项目都需要进一步的规则，可能还需要进一步的限制，以及进一步的库支持。
例如，硬实时程序员通常不能自由使用自由存储(动态内存)，并且在选择库时会受到限制。
我们鼓励制定更具体的规则，作为这些核心规则的补充。
构建您理想的小型基础库并使用它，而不是将您的编程水平降低到貌似优美的汇编代码。

这些规则旨在允许[逐步采用](# S-现代化)。

一些规则旨在增加各种形式的安全，而其他规则旨在减少事故的可能性，还有很多是兼顾两者的。旨在防止事故的规则通常会禁止完全合法的C++。然而，当有两种方式表达一个想法时，其中一种方式被证明是一个常见的错误来源，而另一种方式不是，我们会试图引导程序员使用后者。

本节的规则非常笼统。

理念规则摘要：
* [P.1: 直接用代码表达思想](#Rp-direct)
* [P.2: 用ISO标准c++编写代码](#Rp-Cplusplus)
* [P.3: 明示意图](#Rp-what)
* [P.4:理想情况下，程序应该是静态类型安全的](#Rp-typesafe)
* [P.5: 优先使用编译时检查而不是运行时检查](#Rp-compile-time)
* [P.6: 不能在编译时检查的内容应该可以在运行时进行检查](#Rp-run-time)
* [P.7: 尽早捕获运行时错误](#Rp-early)
* [P.8: 不要泄露任何资源](#Rp-leak)
* [P.9: 不要浪费时间和空间](#Rp-waste)
* [P.10: 优先使用不可变数据而不是可变数据](#Rp-mutable)
* [P.11:封装凌乱的结构，而不是任其在代码中传播](#Rp-library)
* [P.12:适当使用支持工具](#Rp-tools)
* [P.13: 适当使用支持库](#Rp-lib)


理念化规则是笼统的、不能机械的检验的。.然而，反映这些理念主题的单个规则是可以机械的检验的。没有理念基础，更具体/特殊/可检验的规则缺乏合理性。

### <a name="Rp-direct"></a>P.1: 直接用代码表达思想

##### 原因

编译器不读注释（或设计文档)，许多程序员也不读注释(或设计文档）。用代码表示的内容已经定义了语义，可以（原则上）由编译器和其他工具检查。

##### 例子

    class Date {
    public:
        Month month() const;  // do
        int month();          // don't
        // ...
    };

 `month`声明的第一个版本明确返回 `Month`同时也明确表达不会改变 `Date` 对象的状态。
 第二个版本则让读者猜测，也为未捕捉的bug打开了更多的可能性。

##### 坏的例子

里面的for循环是`std::find`的受限形式

    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        int index = -1;   // 不好,外加应该使用 gsl::index
        for (int i = 0; i < v.size(); ++i) {
            if (v[i] == val) {
                index = i;
                break;
            }
        }
        // ...
    }
##### 好的例子

更加清晰的意图表达应该是:

    void f(vector<string>& v)
    {
        string val;
        cin >> val;
        // ...
        auto p = find(begin(v), end(v), val);  // 很好
        // ...
    }

设计良好的库比直接使用语言功能更能表达意图 (要做什么，而不仅仅是如何做某事)。一个C++程序员应该知道标准库的基础知识，并适时的使用它。任何程序员都应该知道正在进行的项目所使用的基础库的基础知识，并适当地使用它们。任何使用这些指南的程序员都应该知道[指南支持库](#S-gsl)，并适当地使用它。
##### 示例

    change_speed(double s);   // 不好:s 表示什么?
    // ...
    change_speed(2.3);

更好的方法是明确double的含义(新速度还是旧速度的增量？)和所使用的单位:

    change_speed(Speed s);    // 较好:指明了s的含义
    // ...
    change_speed(2.3);        // 错误: 没有单位
    change_speed(23m / 10s);  // 米/秒

本来，我们可以接受一个普通的(无单位的)“double”作为增量，但是那样容易出错。如果我们既需要绝对速度有需要增量，我们应该定义一个“增量”类型。
##### 实施（Enforcement）

总的来说很难。.

* 始终使用`const` (检查成员函数是否修改其对象；检查函数是否修改通过指针或引用传递的参数)
* flag uses of casts (类型转换会使类型系统失效)
* 检测模仿标准库的代码 (难)

### <a name="Rp-Cplusplus"></a>P.2: 用ISO标准c++编写代码

##### 原因

这本来就是编写ISO标准c++的一套指导原则.

##### Note

有些运行环境需要扩展，例如访问系统资源。在这种情况下要在局部使用必要的扩展并使用用非核心编码指南。如果可能，构建封装扩展的接口，以便可以在不支持这些扩展的系统上关闭扩展代码或不编译扩展代码。扩展通常没有严格定义的语义。由于没有严格的标准定义，甚至那些通用的、有多个编译器编译的扩展，可能有细微不同的行为和不同的边界情况行为。大量使用这种扩展将会阻碍可移植性。

##### Note

使用有效的ISO C++不保证可移植性（更不用说正确性了）。
不要依赖未定义的行为 (e.g., [undefined order of evaluation](#Res-order))并且要了解具有明确意义的实现的结构（例如, `sizeof(int)`）。
##### Note
有些环境需要限制标准C++或库特性的使用，例如，航空控制软件标准要求避免动态分配内存。在这种情况下，要通过那些为特定环境定制的编码标准的扩展来控制核心指南的使用与否。

##### 实施
使用一个最新的，带有一组不接受扩展的选项的C++编译器(目前是C++17、C++14或C++11)。
### <a name="Rp-what"></a>P.3: 明示意图

##### 原因

不说明某些代码的意图（例如，在名称或注释中），就不可能判断这些代码是否完成了它应该做的事情。
##### 示例

    gsl::index i = 0;
    while (i < v.size()) {
        // ... do something with v[i] ...
    }

这段代码并没有表达“仅仅”遍历`v` 中元素的意图。暴露了索引（index）的实现细节 (这样它可能会被滥用), 并且`i` 的生存期超过了循环的作用域,这可能是期望的也可能不是期望的。读者无法从上述代码片段获知。

改进:

    for (const auto& x : v) { /* do something with the value of x */ }

现在，没有明确提到迭代机制，循环对`const`元素的引用进行操作，因此不会发生意外的修改。如果需要修改,这么写:

    for (auto& x : v) { /* modify x */ }

for语句的更多细节, 可参考[ES.71](#Res-for-range).
有时更好的做法是，使用一个命名的算法. 这个例子使用了Ranges 技术规范中的 `for_each` ，他直接表达了意图:

    for_each(v, [](int x) { /* do something with the value of x */ });
    for_each(par, v, [](int x) { /* do something with the value of x */ });

The last variant makes it clear that we are not interested in the order in which the elements of `v` are handled.

程序员应该熟悉:

* [The guidelines support library](#S-gsl)
* [The ISO C++ Standard Library](#S-stdlib)
* 当前项目所使用的任何基础库。

##### Note

另一种说法：说应该做什么，而不是只说应该怎么做.

##### Note

有些语言结构比其他语言结构更能表达意图.

##### 示例

如果用两个 `int`来表示二维点, 比如这么写:

    draw_line(int, int, int, int);  // 模糊
    draw_line(Point, Point);        // 清晰

##### Enforcement
寻找那些有更好替代方案的通用模式。
* 简单`for` 循环vs. 范围-`for` 循环
* `f(T*, int)` 接口 vs. `f(span<T>)` 接口
* 太大的循环作用域的循环变量
* 原始`new` 和`delete`
* 带有非常多内置类型参数的函数

智能化和半自动化程序转换有着巨大的发展空间。

### <a name="Rp-typesafe"></a>P.4: 理想情况下，程序应该是静态类型安全的

##### 原因

理想情况下，程序应该是完全静态(编译时)类型安全的。
不幸的是，这是不可能的。有问题的领域:

* unions
* 类型转换
* array decay
* range errors
* 窄化转换

##### Note

这些领域的问题是严重问题的来源 (例如,崩溃和安全侵犯).
我们尽力提供替代技术。
##### Enforcement

我们可以根据单个程序的需要和可行性，分别禁止、约束或检测单个问题类别。总是建议另一种选择。例如：

* unions -- 使用`variant` (in C++17)
* 类型转换-- 尽量减少它们的使用; 模板
* array decay -- use `span` (from the GSL)
* range errors -- use `span`
* 窄化转换 -- minimize their use and use `narrow` or `narrow_cast` (from the GSL) where they are necessary

### <a name="Rp-compile-time"></a>P.5: 优先使用编译时检查而不是运行时检查

##### 原因

代码清晰度和性能。
您不必为编译时捕获的错误编写错误处理程序。

##### 示例

    // Int是用来表示整数的别名
    int bits = 0;         // 不要:可以避免的代码
    for (Int i = 1; i; i <<= 1)
        ++bits;
    if (bits < 32)
        cerr << "Int too small\n";

此示例未能实现它的目标（因为溢出是未定义的），应将其替换为简单的：`static_assert`:

    // Int is an alias used for integers
    static_assert(sizeof(Int) >= 4);    // 好: 编译时检查

或者更好的方法是使用类型系统并将“Int”替换为“int32”。

##### 示例

    void read(int* p, int n);   // 将最大的n个整数读进 *p

    int a[100];
    read(a, 1000);    // 错误, 数组越界

改进

    void read(span<int> r); // 读入span<int> r

    int a[100];
    read(a);        // 比较好:让编译器计算元素数量

**Alternative formulation**: 不要把在编译时能做好的事情推迟到运行时。
##### Enforcement

* 寻找指针参数。
* 寻找对越界的运行时检查。

### <a name="Rp-run-time"></a>P.6: 不能在编译时检查的内容应该应该在运行时进行检查

##### 原因


在程序中留下难以检测的错误是在主动要求崩溃和错误的结果
##### Note

理想情况下，我们在编译时或运行时捕获所有错误（不是程序员逻辑错误）。但实际情况是，在编译时捕获所有错误是不可能的，而且在运行时捕获所有剩余的错误通常也负担不起。但是，我们应该努力编写原则上可以检查的程序，只要有足够的资源 (分析工具，运行时检查，机器资源，时间).
##### 反例

    // f是分别编译的，而且有可能是动态加载的。
    extern void f(int* p);

    void g(int n)
    {
        // 错误：元素的数量没有传递给 f()
        f(new int[n]);
    }
在这里，一个重要的信息（元素的数量）已经被彻底地“模糊”了，以至于静态分析可能变得不可行，当`f()`是ABI的一部分我们无法“监控”该指针时，动态检查可能非常困难。我们可以在自由存储中嵌入有用的信息，但这需要对系统和编译器进行全局更改. 我们这里的设计使得错误检测非常困难。

##### 反例

当然，我们可以随指针传递元素的数量:

    // separately compiled, possibly dynamically loaded
    extern void f2(int* p, int n);

    void g2(int n)
    {
        f2(new int[n], m);  // bad: 有可能将错误的元素数量传递给f()
    }

作为参数传递元素的数量比仅仅传递指针和依赖某种（未声明的）约定来知道或发现元素的数量要好（而且更常见）。但是（如上所示），一个简单的打字错误会导致严重的错误。`f2()`的两个参数之间的联系是常规的，而不是显式的。

而且, 认为 `f2()` 会 `delete` 它的参数也不明确 （或者调用者犯了第二个错误?）。
##### 反例

The standard library resource management pointers fail to pass the size when they point to an object:

    // f3s是分别编译的，有可能是动态加载的
    // NB：假设调用代码是ABI兼容的，即使用兼容的C++编译器和相同的stdlib实现。
    extern void f3(unique_ptr<int[]>, int n);

    void g3(int n)
    {
        f3(make_unique<int[]>(n), m);    // 不好: 分别传递所有权和大小
    }

##### 示例

我们需要将指针和元素数量作为一个整体对象进行传递：

    extern void f4(vector<int>&);   // separately compiled, possibly dynamically loaded
    extern void f4(span<int>);      // separately compiled, possibly dynamically loaded
                                    // NB:假设调用代码是ABI兼容的（即使用兼容的C++编译器和相同的stdlib实现）。

    void g3(int n)
    {
        vector<int> v(n);
        f4(v);                     // 传递参照，保留所有权
        f4(span<int>{v});          // 传递快照，保留所有权
    }


这种设计将元素的数量作为对象的一个不可分割的部分进行传递，因此不太可能出现错误，而且动态(运行时)检查始终是可行的，尽管并不总是负担得起。

##### 示例
我们该如何转移所有权以及为了有效使用它所必须的信息呢？

    vector<int> f5(int n)    // OK: move
    {
        vector<int> v(n);
        // ... initialize v ...
        return v;
    }

    unique_ptr<int[]> f6(int n)    // 不好:n 丢失
    {
        auto p = make_unique<int[]>(n);
        // ... initialize *p ...
        return p;
    }

    owner<int*> f7(int n)    // 不好:n 丢失，而且有可能会忘记delete
    {
        owner<int*> p = new int[n];
        // ... initialize *p ...
        return p;
    }

##### 例子

* ???
* 当他们真的知道他们需要什么的时候，展示如何通过传递多态基类的方式避免可能的检查。
  Or strings as "free-style" options

##### Enforcement

* Flag (pointer, count)-style interfaces (this will flag a lot of examples that can't be fixed for compatibility reasons)
* ???

### <a name="Rp-early"></a>P.7:  尽早捕获运行时错误

##### 原因

避免"莫名其妙"的 崩溃。
避免导致 (可能没有被意识到) 不正确结果的错误。

##### Example

    void increment1(int* p, int n)    // 不好: 容易出错
    {
        for (int i = 0; i < n; ++i) ++p[i];
    }

    void use1(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment1(a, m);   // 也许是打字错误, 也许假设 m <= n
                            // 但是假设 m == 20
        // ...
    }

在这里，我们在 `use1`中犯了一个小错误，这将导致数据损坏或崩溃。
(pointer, count)-风格的接口 使得`increment1()`没办法抵御越界错误。
即使我们能检查越界访问的下标，也要等到访问 `p[10]`时才能发现错误。我们本可以早点检查并改进代码：
	
	void increment2(span<int> p)
    {
        for (int& x : p) ++x;
    }

    void use2(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2({a, m});    // maybe typo, maybe m <= n is supposed
        // ...
    }

现在, `m <= n`在调用点被检查(早期) 而不是后面才被检查。
如果只是有个打字错误（我们本来想用`n` 作为边界），上述代码可以进一步简化（消除了错误的可能性）：

    void use3(int m)
    {
        const int n = 10;
        int a[n] = {};
        // ...
        increment2(a);   // 不需要在此传入a的元素的个数
        // ...
    }
.
##### 反例

不要重复检查同一个值。不要将结构化数据作为字符串传递：

    Date read_date(istream& is);    // read date from istream

    Date extract_date(const string& s);    // extract date from string

    void user1(const string& date)    // manipulate date
    {
        auto d = extract_date(date);
        // ...
    }

    void user2()
    {
        Date d = read_date(cin);
        // ...
        user1(d.to_string());
        // ...
    }

date 被验证了两次 (通过 `Date` 构建函数) 并且通过字符串进行传递 (非结构化数据).

##### 示例

过多的检查成本很高。
有些情况下，早期检查是低效的，因为您可能永远不需要该值，或者可能只需要比整体更容易检查的部分值。 Similarly, don't add validity checks that change the asymptotic behavior of your interface (e.g., don't add a `O(n)` check to an interface with an average complexity of `O(1)`).

    class Jet {    // 物理定律规定: e * e < x * x + y * y + z * z
        float x;
        float y;
        float z;
        float e;
    public:
        Jet(float x, float y, float z, float e)
            :x(x), y(y), z(z), e(e)
        {
            // 应该在此处检查值的物理意义吗?
        }

        float m() const
        {
            // 应该在此处处理退化情况吗?
            return sqrt(x * x + y * y + z * z - e * e);
        }

        ???
    };

由于存在测量误差，有关 jet (`e * e < x * x + y * y + z * z`)的物理定律并不是一个不变量。

???

##### Enforcement

* 查看指针和数组:尽早进行范围检查，不要重复。
* 查看转换:消除或标记缩小转换。
* 查找来自输入的未检查值
* 寻找被转换成字符串的结构化数据(具有不变量的类的对象)L。
* ???

### <a name="Rp-leak"></a>P.8: 不要泄露任何资源

##### 原因
即使资源消耗增长缓慢，随着时间的推移，也将耗尽资源。这对于长时间运行的程序来说尤其重要，是负责任的编程行为的重要组成部分。

##### 反例

    void f(char* name)
    {
        FILE* input = fopen(name, "r");
        // ...
        if (something) return;   // 不好: 如果 something == true,文件句柄input就泄露了。
        // ...
        fclose(input);
    }

改进[RAII](#Rr-raii):

    void f(char* name)
    {
        ifstream input {name};
        // ...
        if (something) return;   // OK: 没有泄露
        // ...
    }

**See also**: [The resource management section](#S-resource)

##### Note
通俗地说，泄露是“任何没有清理干净的东西”。
更重要的分类是“任何不能再清理的东西”。例如，在堆上分配一个对象，然后丢失指向该对象的最后一个指针。不应将此规则视为要求在长效对象内的分配必须在程序关闭期间被回收。例如，依赖于系统保证的清理（例如在进程退出时关闭文件和释放内存）可以简化代码。然而，依靠隐式清理是简单的，而且通常更安全。

##### Note
实施[the lifetime safety profile](#SS-lifetime) 可以消除泄露。
当同[RAII](#Rr-raii)提供的资源安全措施结合时, 它消除了“垃圾回收”的需要(通过不产生垃圾)。将此与[类型和边界配置文件](#SS-force)的实施相结合，您将获得由工具保证的完整的类型和资源安全性。

##### Enforcement

* 检查指针：将它们分为非所有者指针（默认）和所有者指针。
 在可行的情况下，用标准库资源句柄替换所有者指针（如上例所示）。或者，使用[the GSL](#S-gsl)中的`owner`标记所有者。 
* 寻找原始 `new` 和 `delete`
* 查找已知的返回原始指针的资源分配函数 (比如 `fopen`, `malloc`, 和`strdup`)

### <a name="Rp-waste"></a>P.9: 不要浪费时间或空间

##### 原因

这可是 C++.

##### Note
您为实现目标而花费的时间和空间（例如，开发速度、资源安全或测试简化）不算是浪费。
"追求效率的另一个好处是，这个过程迫使你更深入地理解问题。." - Alex Stepanov

##### 反例

    struct X {
        char ch;
        int i;
        string s;
        char ch2;

        X& operator=(const X& a);
        X(const X&);
    };

    X waste(const char* p)
    {
        if (!p) throw Nullptr_error{};
        int n = strlen(p);
        auto buf = new char[n];
        if (!buf) throw Allocation_error{};
        for (int i = 0; i < n; ++i) buf[i] = p[i];
        // ... manipulate buffer ...
        X x;
        x.ch = 'a';
        x.s = string(n);    // give x.s space for *p
        for (gsl::index i = 0; i < x.s.size(); ++i) x.s[i] = buf[i];  // copy buf into x.s
        delete[] buf;
        return x;
    }

    void driver()
    {
        X x = waste("Typical argument");
        // ...
    }

上面的写法有点夸张，但其中的每个单一的错误我们都曾经在生产代码中遇到过，而且比上面的更糟糕。注意，X的布局至少浪费了6个字节(很可能更多)。拷贝操作的错误定义是移动语义变得不可能，所以返回操作较慢（注意返回值优化，RVO，不一定会发生）。 `buf` 的 `new` 和 `delete`操作是多余的;如果我们确实需要一个局部字符串，我们应该使用一个局部 `string`。除此之外，还有更多的性能缺陷和无端的复杂性。

##### 反例

    void lower(zstring s)
    {
        for (int i = 0; i < strlen(s); ++i) s[i] = tolower(s[i]);
    }

这是一个来自生产代码的真实的例子。可以看到，在条件语句里有个表达式 `i < strlen(s)`。这个表达式在每次循环中都会被计算，这就意味着每次循环 `strlen` 都必须遍历整个字符串来发现自己的长度。尽管字符串内容是改变的，但假设 `toLower` 不会影响到字符串的长度,因此，在循环外计算长度以免去每次循环都计算长度的负担，会更好些。

##### Note
单点浪费通常没啥影响，如果它影响较大，通常很容易被一个专家修复。但是, 在代码库中大量散布的垃圾很容易造成重大影响，而且专家也不总是像我们那样随时可用就像。那个这个规则的目标（以及支持它的更具体的规则）是在它之前消除与C++的使用有关的大多数浪费。发生了。之后我们可以看看与算法和需求相关的浪费，但这超出了这些准则的范围。
但是，代码库中大量分布的浪费很容易构成显著影响；专家们并非总是像我们希望的那样随时在场。这个规则的目的（以及支持它的更具体的规则）是在发生之前，消除与C++的使用有关的大部分资源浪费。此外，我们可以研究与算法和需求相关的浪费，但这超出了这些指导原则的适用范围。

##### Enforcement

许多更具体的规则旨在实现简单化和消除无端浪费的总体目标。

* 标记用户定义的非默认 `operator++` or `operator--`函数中未使用的返回值。而是优先使用前缀形式。 (Note: "User-defined non-defaulted" is intended to reduce noise. Review this enforcement if it's still too noisy in practice.)
### <a name="Rp-mutable"></a>P.10: 优先使用不可变数据而不是可变数据

##### 原因
对常量的推理要比对变量的推理容易。不可变量是不会意外改变的。有时不可变量还可以被更好的优化。你不能在一个常量上进行数据竞争
常量上不存在数据竞争（data race）.

见[Con: Constants and immutability](#S-const)

### <a name="Rp-library"></a>P.11: :封装那些凌乱的结构，而不是任其在代码中传播

##### 原因
混乱的代码更容易隐藏错误，更难编写。而一个好的界面更容易使用，也更安全。混乱、低级代码又会产生更多混乱、低级的代码。
##### Example

    int sz = 100;
    int* p = (int*) malloc(sizeof(int) * sz);
    int count = 0;
    // ...
    for (;;) {
        // ... read an int into x, exit loop if end of file is reached ...
        // ... check that x is valid ...
        if (count == sz)
            p = (int*) realloc(p, sizeof(int) * sz * 2);
        p[count++] = x;
        // ...
    }

This is low-level, verbose, and error-prone.
For example, we "forgot" to test for memory exhaustion.
Instead, we could use `vector`:

    vector<int> v;
    v.reserve(100);
    // ...
    for (int x; cin >> x; ) {
        // ... check that x is valid ...
        v.push_back(x);
    }

##### Note

The standards library and the GSL are examples of this philosophy.
For example, instead of messing with the arrays, unions, cast, tricky lifetime issues, `gsl::owner`, etc.,
that are needed to implement key abstractions, such as `vector`, `span`, `lock_guard`, and `future`, we use the libraries
designed and implemented by people with more time and expertise than we usually have.
Similarly, we can and should design and implement more specialized libraries, rather than leaving the users (often ourselves)
with the challenge of repeatedly getting low-level code well.
This is a variant of the [subset of superset principle](#R0) that underlies these guidelines.

##### Example

Run a static analyzer to verify that your code follows the guidelines you want it to follow.

##### Note

See

* [Static analysis tools](???)
* [Concurrency tools](#Rconc-tools)
* [Testing tools](???)

There are many other kinds of tools, such as source code repositories, build tools, etc.,
but those are beyond the scope of these guidelines.

##### Note

Be careful not to become dependent on over-elaborate or over-specialized tool chains.
Those can make your otherwise portable code non-portable.


### <a name="Rp-lib"></a>P.13: Use support libraries as appropriate

##### Reason

Using a well-designed, well-documented, and well-supported library saves time and effort;
its quality and documentation are likely to be greater than what you could do
if the majority of your time must be spent on an implementation.
The cost (time, effort, money, etc.) of a library can be shared over many users.
A widely used library is more likely to be kept up-to-date and ported to new systems than an individual application.
Knowledge of a widely-used library can save time on other/future projects.
So, if a suitable library exists for your application domain, use it.

##### Example

    std::sort(begin(v), end(v), std::greater<>());

Unless you are an expert in sorting algorithms and have plenty of time,
this is more likely to be correct and to run faster than anything you write for a specific application.
You need a reason not to use the standard library (or whatever foundational libraries your application uses) rather than a reason to use it.

##### Note

By default use

* The [ISO C++ Standard Library](#S-stdlib)
* The [Guidelines Support Library](#S-gsl)

##### Note

If no well-designed, well-documented, and well-supported library exists for an important domain,
maybe you should design and implement it, and then use it.


# <a name="S-interfaces"></a>I: 接口

接口是程序两部分之间的契约。准确地说明对服务提供者和服务用户的期望是至关重要的。拥有良好的（易于理解、鼓励高效使用、不易出错、支持测试等）接口可能是代码组织最重要的一个方面。
接口规则概要:

* [I.1: 使接口明晰](#Ri-explicit)
* [I.2: 避免非-`const` 全局变量](#Ri-global)
* [I.3: 避免单例](#Ri-singleton)
* [I.4: 使接口精确和强类型化](#Ri-typed)
* [I.5:声明先决条件 (if any)](#Ri-pre)
* [I.6: 优先使用`Expects()`表示先决条件](#Ri-expects)
* [I.7:声明后置条件](#Ri-post)
* [I.8: 优先使用 `Ensures()`表达后置条件](#Ri-ensures)
* [I.9: 如果接口是模板，则使用概念记录其参数](#Ri-concepts)
* [I.10: 使用异常来表示执行所需任务的失败](#Ri-except)
* [I.11: 永远不要通过原始指针(`T*`)或引用(`T&`)转移所有权](#Ri-raw)
* [I.12: 将一个不能为空的指针声明为`not_null`](#Ri-nullptr)
* [I.13: 不要将数组作为单个指针传递](#Ri-array)
* [I.22: 避免复杂的全局对象初始化](#Ri-global-init)
* [I.23: 函数不要有太多参数](#Ri-nargs)
* [I.24: 当改变参数顺序会改变意义时，避免使相同类型的参数相邻](#Ri-unrelated)
* [I.25: 优先选择抽象类作为接口，而不是类层次结构](#Ri-abstract)
* [I.26: 如果需要跨编译器的ABI，就使用C风格的子集](#Ri-abi)
* [I.27: For stable library ABI, consider the Pimpl idiom](#Ri-pimpl)
* [I.30: 封装违反规则的行为](#Ri-encapsulate)

**另请参阅**:

* [F: Functions](#S-functions)
* [C.concrete: Concrete types](#SS-concrete)
* [C.hier: Class hierarchies](#SS-hier)
* [C.over: Overloading and overloaded operators](#SS-overload)
* [C.con: Containers and other resource handles](#SS-containers)
* [E: Error handling](#S-errors)
* [T: Templates and generic programming](#S-templates)

### <a name="Ri-explicit"></a>I.1:使接口明晰

##### 原因

正确性。接口中没有说明的前提很容易被忽略，也很难测试。

##### 反例

通过全局 (命名空间作用域) 变量 (调用模式)控制函数的行为是不明确和容易引起混淆的。 例如:

    int round(double d)
    {
        return (round_up) ? ceil(d) : d;    // don't: "invisible" dependency
    }

两次`round(7.2)`调用的含义可能会产生不同的结果，这对调用者来说并不明显。

##### Exception

有时我们通过一个环境变量来控制一组操作的细节, 例如，正常输出还是详细输出或调试还是优化。非本地控制变量的使用可能会令人困惑，但它们仅控制其他固定语义的实现细节。

##### 反例

通过非局部变量 (e.g., `errno`) 进行报告容易被忽略。例如：

    //禁止:没有测试fprintf的返回值
    fprintf(connection, "logging: %d %d %d\n", x, y, s);

如果连接中断，因此没有日志记录输出，怎么办？See I.???.

**替代方案**: 抛出异常。异常时无法忽略的。

**Alternative formulation**:避免通过非局部或隐式状态使信息穿过接口。 注意，非常量成员函数通过它们的对象的状态将信息传递到其他成员函数。

**Alternative formulation**: 接口应该是一个函数或一组函数。
函数可以是函数模板，函数集可以是类或类模板。

##### Enforcement

* (Simple) 函数不应基于在命名空间范围内声明的变量值来做出控制流决策。
* (Simple) 函数不应该对在命名空间范围内声明的变量执行写操作。

### <a name="Ri-global"></a>I.2: 避免使用非常量全局变量

##### 原因

非常量全局变量隐藏依赖项，并使依赖项受到不可预测的更改的影响。

##### 示例

    struct Data {
        // ... lots of stuff ...
    } data;            // 非常量数据

    void compute()     // 不可这样
    {
        // ... 使用 data ...
    }

    void output()     // 不可这样
    {
        // ... 使用 data ...
    }

还有谁会修改 `data`?

**警告**:全局对象的初始化不是完全有序的。
如果使用全局对象，请使用常量初始化它。但请注意，即使对于常量对象，初始化顺序也可能是未定义的。

##### Exception

全局对象通常好于单例.

##### Note

全局常量很有用。

##### Note

针对全局变量的规则也适用于命名空间作用域的变量。

**Alternative**: 如果使用全局（通常是命名空间范围）数据来避免复制，请考虑通过常量对象引用传递数据。
另一种解决方案是将数据定义为某个对象的状态，将操作定义为成员函数。

**警告**: 小心数据竞争: 如果一个线程可以访问非本地数据（或通过引用传递的数据），而另一个线程执行被调用方，则会出现数据竞争。
每个指向可变数据的指针或引用都是潜在的数据竞争来源。

使用全局指针或引用来访问和更改非常量（或者非全局的）数据并不是比使用非常量全局变量更好的选择，因为这不能解决隐藏的依赖关系或潜在的竞争条件问题。

##### Note

不可变数据不可能有竞争条件

**参照**: See the [rules for calling functions](#SS-call).

##### Note

规则是“避免使用”，而不是“不使用”。当然就会有一些(罕见的)例外，比如 `cin`, `cout` 和 `cerr`。
##### Enforcement

(容易) 报告命名空间范围内声明的所有非常变量以及对非常量数据的全局指针/引用。

### <a name="Ri-singleton"></a>I.3: 避免单例

##### 原因

单例基本上是伪装的复杂的全局对象。

##### 示例

    class Singleton {
        // ... 若干代码，以确保只有一个单例对象被创建，
	//它是正确初始化的，等等。
    };

单例思想有许多变体。
这是问题的一部分。

##### Note

如果不希望全局对象被更改，就将其声明为`const` 或 `constexpr`。

##### Exception

如果需要，你可以使用最简单的 "单例" (它是如此简单以至于通常不被认为是一个单例)在第一次使用时进行初始化:

    X& myX()
    {
        static X my_x {3};
        return my_x;
    }

这是解决初始化顺序相关问题的最有效的解决方案之一。
在多线程环境中，静态对象的初始化不会引入竞争条件（除非您不小心从其构造函数中访问共享对象）。

注意，局部静态变量的初始化并不一定引发竞争条件。
然而，如果`X`的销毁涉及需要同步的操作，我们必须使用一个不那么简单的解决方案。
例如：
    X& myX()
    {
        static auto p = new X {3};
        return *p;  // 存在潜在的资源泄漏
    }

现在必须有人以适当的线程安全的方式“删除”该对象。
那是容易出错的，所以我们不使用那个技术，除非：

* `myX` 处于多线程代码中,
* `X` 对象需要被销毁 (比如, 因为他释放了一个资源), 并且
* `X`的析构函数需要被同步。

如果像许多人一样，将单例对象定义为只为其创建一个对象的类，那么像`myX`这样的函数就不是单例对象，而且这个有用的技术也不是"避免单例"规则的例外。

##### Enforcement

总的来说非常难。

* 寻找名字包含`singleton`的类。
* 寻找只创建单个对象的类(通过计算对象个数或检查构造函数)。
* 如果类X有一个public静态函数，该函数包含类类型为X的函数内静态变量，并返回对该变量的指针或引用，则禁止使用该函数。

### <a name="Ri-typed"></a>I.4:使接口精确和强类型化

##### 原因

类型是最简单和最好的文档，由于其定义良好的含义，可以提高可读性，并且在编译时进行检查。
而且，精确类型化的代码经常可以更好的优化。

##### 反例

Consider:

    void pass(void* data);    //弱的和受限的类型 void* 是不可靠的。

调用者不能确定哪种类型被允许，也不确定是否有可以转化为`const`的数据没有被指出来。注意，所有的指针类型都可隐式转换为 void* 所以调用者很容易错误的提供这个值。

被调用者必须通过`static_cast` 将`data`转化为未验证的类型后才能使用。
这样容易出错，代码也太冗长。

仅仅在那些无法用C++描述的设计中使用`const void*`来传递数据。考虑改用`variant`或指向base的指针。

**Alternative**: 通常，模板参数可以将`void*`转换为`T*`或`T&`,从而消除`void*`。对于泛型代码，这些`T`可以是通用或概念约束的模板参数。

##### 反例

Consider:

    draw_rect(100, 200, 100, 500); //这些数字代表什么?

    draw_rect(p.x, p.y, 10, 20); //10和20的单位是什么?

显然，调用者在描述一个矩形，但不清楚具体描述的是矩形的哪些参数。而且, `int`可以携带任意形式的信息, 包括许多单位的值, 所以我们必须猜测这四个 `int`的含义。或许, 前两个参数是 `x`,`y` 坐标, 那后两个呢?

Comments and parameter names can help, but we could be explicit:

    void draw_rectangle(Point top_left, Point bottom_right);
    void draw_rectangle(Point top_left, Size height_width);

    draw_rectangle(p, Point{10, 20});  // two corners
    draw_rectangle(p, Size{10, 20});   // one corner and a (height, width) pair

Obviously, we cannot catch all errors through the static type system
(e.g., the fact that a first argument is supposed to be a top-left point is left to convention (naming and comments)).
