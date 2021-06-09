# WhoseTinySTL
环境：gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04) <br>

主要参考对象是《STL源码剖析》第一版和https://github.com/zouxiaohang/TinySTL <br>
次要参考对象是大量网络博客以及https://github.com/karottc/sgi-stl <br>
为了尊重侯捷老师的习惯，我把非实现部分的练习代码都放入以jj开头的文件夹中😀 <br>
绝大部分内容就是原封不动地照着参考对象撸了一遍，并在注释中补充了大量个人浅见和查阅资料 <br>
下文中以及代码的注释中，”项目作者“指的就是[TinySTL的作者](https://github.com/zouxiaohang) ，“书”指的就是《STL源码剖析》 <br>
主要参考对象可以通过关键字“项目作者”和“书”查到引用出处，其余注释中，大段文字如果也是引用的话，一般会给出出处链接，简短的注释如果是查阅的话我也不给出处了，请见谅。 <br>

## 参考开源项目仿写微型STL的好处
1.他在实现时会帮你略掉对于核心功能不太必要的东西，比如计算机内存耗尽时的异常处理。这并不是个常见的情况，而且内存都耗尽了程序也不该再运行下去了，代码应该改进。该作者在实现时就没有写异常处理代码。这部分代码难写么？不难。但略掉它对于不熟悉异常处理的菜鸟（我）来说，读通整个代码逻辑的顺畅程度一下就提高了。在科学研究中，懂得忽略什么和懂得重视什么一样重要。 <br>
2.这个作者的代码是能跑通的代码，而不是书里的部分代码样例，更不是书里说的可运行的案例其实缺东少西而运行不了的案例。诚然，书在序言就说了这书不是给新手看的，类里缺几个简单的方法自己都不会补还说自己不是新手？但是，对于学生党来说，你再反复练学生管理系统、图书馆管理系统也不见得就能看懂这书了。而学生管理系统并不能帮你掌握面试官希望你知道的工业环境下常用的技术。再举个例子，内存池技术难么？不难。但是如果你不看这书，你得没事闲逛多少博客和公众号才能了解到这么具体的一个技术？学习路线大框是不会具体到这么细的。这种技术知识密集的书对于新手来说，是必看的——比如说，你要是不看几本这样的书，在参加工作在工业场景里学会前，就一直是新手，而新手和高薪的缘分一直都不怎么好。如果毕业前就想懂工业场景中的技术，除了实习，就得看这种书，而有的实习你不看这种书或者类似深度的系列博客都过不了。不得不说很遗憾，这里不再是学习曲线，而是学习阶跃函数，就得啃一啃这块硬骨头。而这时，一份可运行的代码对于新手来说，就至关重要了。 <br>
3.好，现在我已经初步实现了一个能跑的vector了，我收回第2点里一开始的言论：这个作者的代码我并不能很流畅地跑起来。有可能是我的编译器版本导致的，但跑不起来也不只因为编译器，项目作者的手误打错的、语法逻辑不对的地方也是有的。但我仍然**非常**感谢作者，他的代码99%我都是可以直接用的，而且还能看懂是啥意思，可读性极高，没有他的代码我也就没机会仿写了。 <br>
4.与第一点相关，当项目作者舍弃书中的一堆对于核心问题不是很重要的代码时，也就不必调用某些你见都没见过的库了。不少库你调了又会引发编译器的问题，这些问题新手难以解决，这也会给你省不少时间。 <br>

## 具体说明
1.如书中55页所说，这里的alloc实现版本不考虑多线程的情况 <br>
2.alloc的实现是没有内存不足时的异常处理机制的，“内存不足”指的是整个系统的内存完全耗尽了 <br>
3.iterator中，书101页上说列出的iterator代码来自SGI的<stl_iterator.h>，但我是在SGI的<stl_iterator_base.h>中找到的。但两者较为一致的是distance和advance系列函数的实现都写在了这个文件里。然而项目作者把这两个系列的函数都写进了<Algorithm.h>里。从书中100页的总结来看，应当写到algorithm或具体的容器里，因为操作iterator的是算法或容器，不是它自己。但是如果这样，那容器本来该和算法分离的，现在却由于这种写法不得不引用algorithm。两种说法我觉得都有道理，但为了先写出能运行的代码，我决定效仿项目作者，把这两个系列函数写进algorithm里。完成完整项目后再考虑更新。 <br>
4.UinitializedFunctions.h里一共有三组函数模板，每组函数都是一个调用两个重载的函数模板。项目作者把每个重载的函数模板声明都在该文件里写了一遍。鉴于每个模板和它的声明的间隔都只有10行左右，且该文件只有这三组函数而没有别的内容，私以为实在没必要将声明和定义分开，于是自己书写时就把每组里的两个重载函数模板的声明和定义写在了一起。侯捷老师提供的样例也是这样做的，见书70页。 <br>
5.关于移动操作有件事要提。当你使用了移动操作，指向被移动对象（尤其是用了std::move的非临时对象)的指针很有可能就不好使了。下面以vector为例：使用STL的时候如果遇到这个问题的话，就会在没有报错的情况下自动终止程序；使用我的WhoseTinySTL的话，就会报“段错误：核心已转储”。千万要避免用std::move把被指针指向的对象变为右值的用法！ <br>
6.项目作者vector的构造函数和insert函数有个技巧，作者写了注释“处理指针和数字间的区别的函数”，但我觉得说明一下比较好。就是对于vec(n,val)这种调用构造函数或vec.insert(n,val)这种调用insert的形式在两个输入都是相同类型的integer时（比如int，unsigned int等等），本来应该调用把val复制n份的那个重载，但是实际上却会调用vec.f(first,last)这种重载形式。因为first和last这种重载形式是模板，而上面那种是普通的函数。当传了两个integer进来时，我们希望调用的vec.f(n,val)由于第一个参数n的类型是size_t，也就是unsigned long long，integer需要型转，而调用模板的话，直接生成两个integer，没有型转，结果就错误调用了另一个函数重载。但没有问题，作者专门写了辅助函数（就是函数名里有_aux的）来处理这问题。通过判断传进来的对象的类型是不是integer来进行辅助函数重载即可。 <br>
