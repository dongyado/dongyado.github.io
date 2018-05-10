---
title: 高效数值计算之Lazy Evaluation
date: 20180502
author: nicklhy
layout: post
categories:
  - 其他
status: publish
type: post
published: true
---

## 简介

注意，本文提及的高效数值计算涉及到的主要是一个完整表达式的结果计算，而非单个operator的高效实现。

## 一个Naive的向量类实现

假设我们现在需要实现一个向量的类型，能够支持最简单的element-wise plus操作，那么我们很自然的可能会写出类似下面的代码

```C++
/* naive_vec.cpp */
class Vec {
  public:
  Vec(int len=0): len(len), dptr(0) {
    cout << "Construct Vec, len = " << len << endl;
    if(len == 0) {
      dptr = 0;
      len = 0;
    }
    else dptr = new float[len];
  }
  Vec(const Vec& src) : len(src.len) {
    cout << "Copy construction of Vec, len = " << len << endl;
    dptr = new float[len];
    memcpy(dptr, src.dptr, sizeof(float)*len );
  }
  Vec& operator=(const Vec& src) {
    if(dptr) delete [] dptr;
    cout << "Assign of Vec, len = " << len << endl;
    dptr = new float[len];
    memcpy(dptr, src.dptr, sizeof(float)*len );
    return *this;
  }
  ~Vec(void) {
    cout << "Destruction of Vec, len = " << len << endl;
    if(dptr != 0) delete [] dptr;
  }
  int len;
  float* dptr;
};
inline Vec operator+(const Vec &lhs, const Vec &rhs) {
  Vec res(lhs.len);
  for (int i = 0; i < lhs.len; ++i) {
    res.dptr[i] = lhs.dptr[i] + rhs.dptr[i];
  }
  return res;
}
```

那么当我们实际使用时，你会发现下列两种调用会导致不同的开销，前者将会调用4次构造函数，后者调用3次，

```C++
/* naive_vec.cpp */
Vec v1(10), v2(10);

// 方法一
Vec v3(10);
v3 = v1+v2;

// 方法二
Vec v4 = v1+v2;
```

另一方面，如果我们把计算表达式变长，例如`a=b+c+d+e+f;`你会发现任意两个变量完成加法操作后，我们都不得不新建一个临时向量变量进行结果存储，即计算次数$$S = N+(N-1) = 2N-1$$。

这里的原因其实也很容易理解，即在上述代码中不论我们是否事先为加法结果分配内存，`operator+`都会生成一个新的Vec对象，那么如何避免这种情况呢？或许大家会想到实现一个`vec_plus`函数来代替`operator+`，

```C++
/* naive_vec2.cpp */
void vec_plus(const Vec& lhs, const Vec& rhs, Vec& dst) {
  if(lhs.len != rhs.len) throw std::runtime_error("Size mis-match!"); // error

  if(dst.len != lhs.len) {
    if(dst.len > 0) delete [] dst.dptr;
    dst.dptr = new float[lhs.len];
  }
  for (int i = 0; i < lhs.len; ++i) {
    dst.dptr[i] = lhs.dptr[i] + rhs.dptr[i];
  }
}
```

这里先不考虑非成员函数随意修改对象成员变量的风格问题，这里问题的关键在于：

1. 其会限制我们写出类似`a=b+c+d+e`这样简洁美观的代码，而必须改成类似于`vec_plus(b, c, a); vec_plus(d, a, a); vec_plus(e, a, a);`的代码，一旦计算表达式过长，我们将不得不进行令人疲惫的复制粘贴工作，因为此时我们必须每次手动指定该加法函数的三个参数；
2. 另一方面，由于必须手动写出每次二元计算的调用代码，因此我们没法利用语言为我们设置的运算符优先级便利，到底先计算表达式中的哪一步分完全由用户指定！

## 一个可能的解决方法：惰性计算(Lazy Evaluation)

那么究竟该如何解决上述问题呢？这里，先让我们抛开C++、Python、Java等各种编程语言的限制，假如我们要计算任意长度由已知运算符和运算数所构成的表达式，我们会希望这个功能实现具有怎样的特性呢？例如

$$a = b+(c-d)*e-f/g$$

总结一下，一般无非以下几点：

1. 结果正确；
2. 尽量不要产生多余的内存开销（例如总体内存占用仅为各个变量使用内存的和）；
3. 编程时代码简单（能写出诸如`a=b+c*d-e/f;`的长表达式）；
4. 速度快；


排在第一位的"结果正确性"是基本要求，由于本文谈及的优化方法不包括近似求解，因此这里暂且跳过，依次讨论下排在后面的三点。

###  尽量不要产生多余的内存开销

关于这一点一般最直白的表达就是我们希望计算程序只为各个输入变量和输出变量申请内存，（尽量）别为中间计算结果而申请新的内存空间。例如在计算上面的表达式$$a = b+(c-d)*e-f/g$$时我们可能非常吝啬，只愿意提供$$a, b, c, d, e, f, g$$这几个变量的内存空间，不给$$c-d, f/g$$等中间结果留资源。

###  编程时代码简单

第一次听说"编程时代码简单"这点要求时或许你会不以为然，毕竟怎样算简单怎样算复杂本来也没有严格的标准，所以这里我们如果以具体的两个例子来说明或许会更加简洁。

首先我们来讨论一下大名鼎鼎的BLAS和LAPACK，它的第三方实现如atlas、openblas、mkl等基本上是当前我们能接触到最高效的几个线性代数库，应用也是非常的广泛，不过由于这些库的功能通常都是通过各种显式命名的函数所实现（例如gemm、gemv等等），因此在使用它们时常常会感到很麻烦，每次只能进行一次运算。例如我们如果想计算$$a = b/c*d$$，那么我么必须写成类似于`tmp = b/c; a = tmp*d;`这样的形式，很难一行代码写清楚整个计算过程，事实上，这里除了代码变得复杂冗长这一问题外还存在另外几个缺点：a）对于多个矩阵相乘这样允许使用结合律的计算操作，程序无法通过调整计算顺序来优化整体算法的复杂度；b）程序无法利用加减乘除等各种operator的计算优先级，每步计算什么全靠程序员写的代码；c）上面提到的**“尽量不要产生多余的内存开销”**这一期望将变成彻底的奢望，毕竟既然这里都定义并赋值了一个变量tmp，那么又怎么可能不为它申请内存空间呢？

与之相反的一个典型则是Matlab，不管计算机专业的人如何喷Matlab语法风格混乱、不方便产品化实际应用，做理论的人就是喜欢用它，为啥呢？很明显，对于数学计算这个需求来说，Matlab写起来就完爆比C++、Python等其他语言，基本上你能写出的数学公式都能简洁直白的翻译到Matlab的一行代码里（例如`a = b+c*d-e/f^2`），并且运算速度还特别快，压根不用考虑编程能力的问题。

###  速度快

这点其实要说的不多，因为本文不针对特定的计算问题给出特定的优化算法，仅仅是希望得到一种尽可能普适的科学计算模式，并且这种计算模式不会造成计算过程效率的降低。



## 参考

* [mshadow的原理--MXNet](http://www.cnblogs.com/heguanyou/p/7545344.html)
