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

虽然如今已经有各种科学计算库可供算法工程师们选择，但有时我们依然会需要在项目中实现一些自定义的矩阵、向量等数据类型，并且让它们支持加减乘除等各种计算功能（例如[mhadow](https://github.com/dmlc/mshadow)尝试提供一种能在CPU、GPU间可以无缝切换的矩阵运算功能），在实现这些功能时常常会苦恼如何设计才能使最终的API既简单又高效、如何统一异构计算各种求值函数的差异等等，本文尝试介绍一种被广泛应用于Eigen、mshadow等矩阵计算库中的编程模式：**惰性计算(Lazy Evaluation)**，详细说明其出现的背景和原因，及其产生的强大功能。



注：本文大部分代码参考了博文[mshadow的原理--MXNet](http://www.cnblogs.com/heguanyou/p/7545344.html)。



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



## 一种理想的解决方案

那么究竟该如何解决上述问题呢？这里，先让我们抛开C++、Python、Java等各种编程语言的限制，假如我们要计算任意长度由已知运算符和运算数所构成的表达式，我们会希望这个功能实现具有怎样的特性呢？例如

$$a = b+(c-d)*e-f/g$$

总结一下，一般无非以下几点：

1. 结果正确；
2. 尽量不要产生多余的内存开销（例如总体内存占用仅为各个变量使用内存的和）；
3. 编程时代码简单（能写出诸如`a=b+c*d-e/f;`的长表达式）；
4. 速度快；


排在第一位的"结果正确性"是基本要求，由于本文谈及的优化方法不包括近似求解，因此这里暂且跳过，依次讨论下排在后面的三点。

###  尽量不要产生多余的内存开销

对这点一般最直白的解释就是我们希望计算程序只为各个输入变量和输出变量申请内存，（尽量）别为中间计算结果而申请新的内存空间。例如在计算上面的表达式$$a = b+(c-d)*e-f/g$$时我们可能非常吝啬，只愿意提供$$a, b, c, d, e, f, g$$这几个变量的内存空间，不给$$c-d, f/g$$等中间结果留资源。

###  编程时代码简单

第一次听说"编程时代码简单"这点要求时或许你会不以为然，毕竟怎样算简单怎样算复杂本来也没有严格的标准，所以这里我们如果以具体的两个例子来说明或许会更加简洁。

首先我们来讨论一下大名鼎鼎的BLAS和LAPACK，它的第三方实现如atlas、openblas、mkl等基本上是当前我们能接触到最高效的几个线性代数库，应用也是非常的广泛，不过由于这些库的功能通常都是通过各种显式命名的函数所实现（例如gemm、gemv等等），因此在使用它们时常常会感到很麻烦，每次只能进行一次运算。例如我们如果想计算$$a = b/c*d$$，那么我么必须写成类似于`tmp = b/c; a = tmp*d;`这样的形式，很难一行代码写清楚整个计算过程，事实上，这里除了代码变得复杂冗长这一问题外还存在另外几个缺点：a）对于多个矩阵相乘这样允许使用结合律的计算操作，程序无法通过调整计算顺序来优化整体算法的复杂度；b）程序无法利用加减乘除等各种operator的计算优先级，每步计算什么全靠程序员写的代码；c）上面提到的**“尽量不要产生多余的内存开销”**这一期望将变成彻底的奢望，毕竟既然这里都定义并赋值了一个变量tmp，那么又怎么可能不为它申请内存空间呢？

与之相反的一个典型则是Matlab，不管计算机专业的人如何喷Matlab语法风格混乱、不方便产品化实际应用，做理论的人就是喜欢用它，为啥呢？很明显，对于数学计算这个需求来说，Matlab写起来就完爆比C++、Python等其他语言，基本上你能写出的数学公式都能简洁直白的翻译到Matlab的一行代码里（例如`a = b+c*d-e/f^2`），并且运算速度还特别快，压根不用考虑编程能力的问题。

###  速度快

要求计算速度快也很容易理解，因为本文目的不在于为特定问题给出特定的优化算法，仅仅是希望得到一种尽可能普适且高效的科学计算模式，因此不希望这种计算模式会带来额外的计算开销（或计算复杂度）。



## 一个可能的解决方法：惰性计算(Lazy Evaluation)

首先针对之前计算过程产生临时对象造成内存浪费的问题，我们先看一个解决方案：

```C++
#include <stdlib.h>
#include <string.h>
#include <iostream>
using namespace std;

class Vec;
struct BinaryAddExp {
  const Vec &lhs;
  const Vec &rhs;
  BinaryAddExp(const Vec &lhs, const Vec &rhs)
  : lhs(lhs), rhs(rhs) {}
};

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
    ~Vec() {
      cout << "Destruction of Vec with " << len << " elements." << endl;
    }
    // here is where evaluation happens
    inline Vec &operator=(const BinaryAddExp &src) {
      if(src.lhs.len != src.rhs.len) throw runtime_error("Shape mis-match!");
      if(this->len != src.lhs.len) {
        if(this->len != 0) delete [] this->dptr;
        this->dptr = new float[src.lhs.len];
        this->len = src.lhs.len;
      }
      for (int i = 0; i < len; ++i) {
        dptr[i] = src.lhs.dptr[i] + src.rhs.dptr[i];
      }
      return *this;
    }
    int len;
    float* dptr;
};

// no evaluation happens here
inline BinaryAddExp operator+(const Vec &lhs, const Vec &rhs) {
  return BinaryAddExp(lhs, rhs);
}

int main(void) {
  Vec A(10), B(10), C(10);
  A = B + C;
  for (int i = 0; i < 10; ++i) {
    cout <<  i << ": " << A.dptr[i] << " == " << B.dptr[i] << " + " << C.dptr[i] << endl;
  }
  return 0;
}
```

初次看到这种写法的读者可能会感到困惑，因为这里的“计算”并非实现在`operator+`函数里面，而是实现在了赋值函数`operator=`里面，换句话说，由于我们把计算过程从*"运算符"*函数推迟到了*"赋值"*函数，此时我们一定能够知道计算结果该存储在哪里，完全消除了之前申请新内存创建新对象作为返回值却又不知道是否真的有必要的担心（请仔细考虑这里和之前的差异）！

不过这里也引入了一个新的问题，由于`operator+`返回的是一个`BinaryAddExp`类型，可我们没有实现其与普通`Vec`类型的`operator+`函数，所以我们将不能写出类似于`a = b+c+d;`的代码，更加糟糕的是，即使我们添加了`operator+(const Vec&, const BinaryAddExp&)`与其交换参数顺序的版本，我们还有减法、乘法、除法等其他计算符需要重载，随着计算符数量的增长，我们需要实现的重载函数个数也将以$$N^2$$的速度增长！

上面提到的这个问题显然是无法让用户接受的，下面将展示如何使用一种C++黑魔法来提供一个完美的替代方案：

```C++
#include <cstdio>

// this is expression, all expressions must inheritate it,
//  and put their type in subtype
template<typename SubType>
struct Exp {
  // returns const reference of the actual type of this expression
  inline const SubType& self(void) const {
    return *static_cast<const SubType*>(this);
  }
};

// binary add expression
// note how it is inheritates from Exp
// and put its own type into the template argument
template<typename TLhs, typename TRhs>
struct BinaryAddExp: public Exp<BinaryAddExp<TLhs, TRhs> > {
  const TLhs &lhs;
  const TRhs &rhs;
  BinaryAddExp(const TLhs& lhs, const TRhs& rhs)
      : lhs(lhs), rhs(rhs) {}
  // evaluation function, evaluate this expression at position i
  inline float Eval(int i) const {
    return lhs.Eval(i) + rhs.Eval(i);
  }
};

// binary minus expression
// note how it is inheritates from Exp
// and put its own type into the template argument
template<typename TLhs, typename TRhs>
struct BinaryMinusExp: public Exp<BinaryMinusExp<TLhs, TRhs> > {
  const TLhs &lhs;
  const TRhs &rhs;
  BinaryMinusExp(const TLhs& lhs, const TRhs& rhs)
      : lhs(lhs), rhs(rhs) {}
  // evaluation function, evaluate this expression at position i
  inline float Eval(int i) const {
    return lhs.Eval(i) - rhs.Eval(i);
  }
};

// binary multiply expression
// note how it is inheritates from Exp
// and put its own type into the template argument
template<typename TLhs, typename TRhs>
struct BinaryMulExp: public Exp<BinaryMulExp<TLhs, TRhs> > {
  const TLhs &lhs;
  const TRhs &rhs;
  BinaryMulExp(const TLhs& lhs, const TRhs& rhs)
      : lhs(lhs), rhs(rhs) {}
  // evaluation function, evaluate this expression at position i
  inline float Eval(int i) const {
    return lhs.Eval(i) * rhs.Eval(i);
  }
};

// binary div expression
// note how it is inheritates from Exp
// and put its own type into the template argument
template<typename TLhs, typename TRhs>
struct BinaryDivExp: public Exp<BinaryDivExp<TLhs, TRhs> > {
  const TLhs &lhs;
  const TRhs &rhs;
  BinaryDivExp(const TLhs& lhs, const TRhs& rhs)
      : lhs(lhs), rhs(rhs) {}
  // evaluation function, evaluate this expression at position i
  inline float Eval(int i) const {
    return lhs.Eval(i) / rhs.Eval(i);
  }
};
// no constructor and destructor to allocate
// and de-allocate memory, allocation done by user
struct Vec: public Exp<Vec> {
  int len;
  float* dptr;
  Vec(void) {}
  Vec(float *dptr, int len)
      :len(len), dptr(dptr) {}
  // here is where evaluation happens
  template<typename EType>
  inline Vec& operator= (const Exp<EType>& src_) {
    const EType &src = src_.self();
    for (int i = 0; i < len; ++i) {
      dptr[i] = src.Eval(i);
    }
    return *this;
  }
  // evaluation function, evaluate this expression at position i
  inline float Eval(int i) const {
    return dptr[i];
  }
};
// template add, works for any expressions
template<typename TLhs, typename TRhs>
inline BinaryAddExp<TLhs, TRhs>
operator+(const Exp<TLhs> &lhs, const Exp<TRhs> &rhs) {
  return BinaryAddExp<TLhs, TRhs>(lhs.self(), rhs.self());
}
// template minus, works for any expressions
template<typename TLhs, typename TRhs>
inline BinaryMinusExp<TLhs, TRhs>
operator-(const Exp<TLhs> &lhs, const Exp<TRhs> &rhs) {
  return BinaryMinusExp<TLhs, TRhs>(lhs.self(), rhs.self());
}
// template mul, works for any expressions
template<typename TLhs, typename TRhs>
inline BinaryMulExp<TLhs, TRhs>
operator*(const Exp<TLhs> &lhs, const Exp<TRhs> &rhs) {
  return BinaryMulExp<TLhs, TRhs>(lhs.self(), rhs.self());
}
// template div, works for any expressions
template<typename TLhs, typename TRhs>
inline BinaryDivExp<TLhs, TRhs>
operator/(const Exp<TLhs> &lhs, const Exp<TRhs> &rhs) {
  return BinaryDivExp<TLhs, TRhs>(lhs.self(), rhs.self());
}

const int N = 3;
int main(void) {
  float sa[N] = {1, 2, 3};
  float sb[N] = {2, 3, 4};
  float sc[N] = {3, 4, 5};
  float sd[N] = {4, 5, 6};
  float se[N] = {5, 6, 7};
  Vec A(sa, N), B(sb, N), C(sc, N), D(sd, N), E(se, N);
  // run expression, this expression is longer:)
  A = B + C + C*D -D/E;
  for (int i = 0; i < N; ++i) {
    printf("%d:%f == %f + %f + %f*%f - %f/%f\n",
        i, A.dptr[i], B.dptr[i], C.dptr[i], C.dptr[i],
        D.dptr[i], D.dptr[i], E.dptr[i]);
  }
  return 0;
}
```

编译运行上述代码将输出：

```shell
0:16.200001 == 2.000000 + 3.000000 + 3.000000*4.000000 - 4.000000/5.000000
1:26.166666 == 3.000000 + 4.000000 + 4.000000*5.000000 - 5.000000/6.000000
2:38.142857 == 4.000000 + 5.000000 + 5.000000*6.000000 - 6.000000/7.000000
```

可以看到，计算结果完全符合预估，没有问题。

上述代码的关键在于:

* 对**计算表达式**形成抽象类`Exp`，然后不管是计算数(`Vec`)还是计算符（加减乘除等）均继承自`Exp`(也就是说，**不管计算数还是计算符，其实都属于计算表达式**)；
* `Exp`类型包含一个成员函数`Exp::self()`，可以返回模板参数`SubType`对应类型的指向自身的指针，例如`Vec::self()`返回`Vec*`类型的指针，`BinaryDivExp::self()`返回`BinaryDivExp*`类型的指针；
* 计算符类型在构造时不发生实际计算，直到计算数在发生赋值操作时才进行计算，与之前所述的Lazy Evaluation原则一致；
* `operator=`实际执行计算时循环遍历结果向量的每个元素，并通过递归调用（子类）计算表达式的`SubType::Eval`成员函数来获取结果的第i个元素值。

从效果上看，这里`Exp`模板类实现的功能或许有点像C++里的虚函数，也就是尽管我们某个变量是一个基类类型的对象或指针，但依然能够调用到子类重新实现过的新函数，不过需要注意的是，这里绝对不能真的把它们划等号，因为虚函数查找发生在程序运行时，而上面模板类的类型推导发生在编译时，在科学计算这种对计算效率要求非常苛刻的场景，发生非常高频的虚函数调用绝对是一个天大的灾难。

另外上述黑魔法还有一个非常值得注意的优点，就是它实现了自动的函数内联！例如当我们计算`a = b+c*d-e`时，由于各个`Eval`和`operator`函数都是inline类型，程序在**编译**时其实会变成类似于下面的样子（注意不是运行时）

```C++
for(int i=0; i<n; i++)
  a[i] = b[i]+c[i]*d[i]-e[i];
```

此时，我们的代码将变得与手写for循环一样高效！



## 总结

以上已经把基于C++模板表达式的惰性计算从原理上解释清楚了，但是仅仅实现了向量在CPU中进行加减乘除四种操作的功能，并没有加入CPU并行、GPU并行等高级优化技术，限于篇幅，本文不再赘述，想要更加深入的了解模板表达式可以阅读[mhadow](https://github.com/dmlc/mshadow)的源码。



## 参考

* [mhadow](https://github.com/dmlc/mshadow)
* [mshadow的原理--MXNet](http://www.cnblogs.com/heguanyou/p/7545344.html)
