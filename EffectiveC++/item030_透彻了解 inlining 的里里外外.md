# 条款 30：透彻了解 inlining 的里里外外

Understand the ins and outs of inlining.

### inlining 的理念

inline 函数背后的整体观念是将”对此函数的每一个调用“都以函数本体替换。

但是过度的 inlining 会造成程序体积太大，即使拥有虚内存，inline 造成的代码膨胀也会导致额外的换页行为（paging），降低指令高速缓存装置的击中率（instruction cache hit rate），以及伴随这些而来的效率损失。反之，如果 inline 函数的本体很小，编译器针对”函数本体“所产出的码可能比针对”函数调用“所产出的码更小。

inline 只是对编译器的一个申请，不是强制命令。并且这项申请可以隐喻提出：

```c++
class Person {
    ...
    int age() const { return _age; } // 一个隐喻的 inline 申请，将函数实现放在 class 的定义式中
};
```

Inline 函数通常被置于头文件内，因为大多数建置环境（build environments）在编译过程中进行 inlining，而为了将一个”函数调用“替换为”被调用函数的本体“，编译器需要知道函数的实现。

### inline 失败的情况

大部分编译器拒绝将太过复杂（例如带有循环或递归）的函数 inlining，而所有对 virtual 函数的调用也会使 inlining 失败，因为 virtual 意味着”直到运行期才确定调用哪个函数“。

函数是否被 inlining 取决于所使用的编译器，如果它们不能将要求的函数 inlining，则会给出一个警告信息。有时候虽然编译器有意愿 inlining 某个函数，但还是可能为该函数生成一个函数本体。例如当程序需要取某个 inline 函数的地址时。

不要将构造和析构函数 inlining，因为 C++ 对对象的创建和被销毁做了各种其他的保证。

### 评估 inline 函数

inline 函数无法随着程序库的升级而升级。即如果 `f` 是程序库内的一个 inline 函数，一旦程序库的设计者决定改变 `f`，所有用到 `f` 的客户端程序都必须重新编译；但如果 `f` 是 non-inline 函数，则只需要重新链接即可。

> 80-20 经验法则：平均而言，一个程序往往将 80% 的执行时间花费在 20% 的代码上。

**最后总结一下：**

- 将大多数 inline 函数限制在小型，被频繁调用的函数身上。这可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。