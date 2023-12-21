## Recap1:: iterator

所有的容器都实现了一个叫迭代器的东西（iterator）

- 迭代器可以使你以编程的方式访问容器元素

- 迭代器有确定的元素顺序：它知道接下要访问的元素是什么 

  - 每一次的元素顺序不一定相同！

- 所有的迭代器都实现了一些相同的操作：

  - 初始化----------->      iter = s.begin();
  - 自增-------------->     ++iter;
  - 解引用------------>    *iter;
  - 比较--------------->    iter != s.end();
  - 复制---------------->   new_iter = iter;

  

## Recap2:: Const and Const Collectness

- 应用代码中尽可能使用const参数和const变量
- 所以不改变成员变量的成员函数应该被标记为const
- 不要重复造轮子，用static_cast/const_cast来复用non-const的代码以实现const版本
- `auto` 关键字进行类型推断时，它会忽略原始变量的 `const` 和引用 (`&`) 修饰符。
- 为你的类创建iterator和const_iterator两种迭代器
  - const iterator：不能++iter，可以改变*iter
  - const_iterator：可以++iter，不能改变*iter
  - const const_iterator：不能++iter，不能改变*iter



## Template Functions

不用重载函数来应对不同类型的参数，模板函数可以处理不同类型的参数。

定义模板函数（其实和实现模板类的成员函数类似）：

```C++
template<typename Type>
Type myMin(Type a, Type b){
    return a < b ? a:b;
    
}
```

设置默认类型：

```C++
template<typename Type=int>
Type myMin(Type a, Type b){
    return a < b ? a:b;
    
}
```

使用模板函数

1. 传入参数类型，和模板类的使用类似（explicitly）

   ```c++
   template<typename Type>
   Type myMin(Type a, Type b){
       return a < b ? a:b;   
   }
   // 省略main()
   cout<< myMin<int>(5,3) << endl;
   cout<< myMin<float>(5.3,3.14) << endl;
   ```

2. 让编译进行类型推导（implicitly）

   ```C++
   template<typename T,typename U>
   auto smartMyMin(T a, U b){
       return a < b ? a:b;
       
   }// 返回值是auto，因为不能在没执行前不确定返回值类型
   // 省略main()
   cout<< smartMyMin(5.3,3) << endl
   
   ```

   

模板函数的实例化

和模板类一样，模板函数只有使用到的时候才会生成

- 根据不同参数类型，编译器根据模板产生不同的实例
- 编译过后，就像你自己写过该版本的函数一样

等等，在使用它之前，代码是不存在的。这样更快---->能否运用这点？

模板元编程。

通常来说，代码在runtime时才运行

用模板元编程，代码在生成时即可以运行



## 模板元编程

使用模板在编译时求解斐波那契数列

```C++
template<unsigned n>
struct Factorial {
	enum { value = n * Factorial<n - 1>: :value } ;
};
template<> // template class " special ization"
struct Factorial<0> {
	enum { value=1 };
};
std::cout << Factorial<10>::value << endl; // prints 3628800， but run during compile time !
```

<!--template<>为模板特例化，即Factorial<0>不依赖上面的通用的模板定义-->



constexpr

编译时将代码运行的另一种方式是constexpr

该关键字指定了一个常量表达式：

- 常量表达式是指值不会改变，能在编译时求值的表达式。

- 常量表达式必须立即被初始化，并且在编译期间得到执行。
- 传递给常量表达式的参数也必须为常量表达式。

变量也可以被声明为常量表达式。

```C++
constexpr double fib(int n) { // function declared as constexpn
 if (n == 1) return 1;
 return fib(n-l) * n;
}
int main() {
 const long long bigval = fib(20);
 std: : cout << bigval << std::endl;
}
```



编译时运行的好处：

- 生成代码变得更小
- 只在编译时运行一次，并且可在运行时重复使用

模板元编程（TMP）是被发现的，不是被发明的！



TMP的应用：

- **优化矩阵、树等数学结构操作**：TMP可以用于优化涉及数学结构（如矩阵、树等）的操作。通过在编译时进行计算，可以生成高效的代码，用于执行这些数学操作，从而提高程序性能。

- **基于策略的设计**：TMP也可以用于实现基于策略的设计模式。这种设计模式允许根据不同的策略来配置或自定义软件组件的行为，而TMP可以在编译时选择适当的策略，从而提供灵活性和性能优势。

- **游戏图形**：TMP可能在游戏图形方面发挥作用。它可以用于在编译时生成图形渲染代码，以优化游戏中的图形效果或性能。



## Algorithms

统计一个字符串字符出现频率？

统计一个vector中数字出现频率？

统计一个stream中单词出现频率？

它们均是一类问题！如何编写出通用的算法？（generics）





