const-interface: All member functions marked const in a class definition. Objects of type const ClassName may only use the const-interface.

const对象只能访问该类的const成员函数。



## mark const&create const version

类的成员函数是否应该为const?

- 如果这个函数不改变这个对象的成员，标记为const
- 否则，是否可以再创建一个这个函数const版本？



1. mark const

   ```c++
   size_t size() const;
   ```

2. create const version

   来看这个遍历函数为什么选择第二种方式：

   ```C++
   std::string& at(size_t indx); // like vec[] but with error checking
   ```

   它返回的容器元素的引用，所以可能会改动对象，所以不能直接mark const，举个例子：

   ```C++
   Example
   // StrVector my_vec = {"sarah", "haven"};
   std::string& elem_ref = my_vec.at(1);
   elem_ref = "Now I'm Different" ;
   // my_vec = {"sarah", "Now I'm Different"}
   ```

   creat const version

   ```C++
   const std::string& at(size_t indx) const; 
   ```

   实现：用一个类型转换的trick，来复用non-const版本的实现

   ```C++
   const std::string& StrVector::at(size_t index) const {
    return static_cast<const
   std::string&>(const_cast<StrVector*>(this)->at(index));}
   ```

   因为调用该函数的对象肯定为const对象。

   1. 转换为non-const对象，以调用non-const版本

      `const_cast<StrVector*>(this)->at(index))`

   2. 再将返回值转成const

      `return static_cast<const std::string&>(...)`

迭代器

```c++
iterator begin();
iterator end();
```

这两个函数要不要`mark const`?

```C++
iterator begin() const;
iterator end() const; 
```

看似它不会改动对象。 考虑一个这个函数，它接受一个const对象，功能上没有丝毫问题：

```c++
void printVec(const StrVector& vec){
cout << "{ ";
for(auto it = vec.begin(); it != vec.end(); ++it){
	cout << *it << endl;
}
cout << " }" << endl;
}

```

再考虑一下这个函数：

```C++
void printVec(const StrVector& vec){
cout << "{ ";
for(auto it = vec.begin(); it != vec.end(); ++it){
	*it = "dont mind me modifying a const vector :D";
}
cout << " }" << endl;
}

```

返回的iterator是vevtor的一个内置数据类型

```c++
//vector.h
template<typename T> class vector {
public:
	using iterator = T* // something internal like T*
	iterator begin();
}

```

所以这个函数和`at()`的逻辑是一样的，需要create const version

```C++
iterator begin();
iterator end(); 
const iterator begin() const;
const iterator end() const; 
```

现在我们再传入一个const对象，执行类似操作，就会报错了。

```C++
void printVec(const StrVector& vec){
cout << "{ ";
for(auto it = vec.begin(); it != vec.end(); ++it){
	*it = “HELLO”; //compile error!
}
cout << " }" << cout;
}
```

const iterator vs const_iterator(Nitty Gritty 关键细节)

```C++
using iterator = std::string*;
using const_iterator = const std::string*;
```



1. const iterator

   ```C++
   const iterator it_c = vec.begin(); //string * const, const ptr to non-const obj
   *it_c = "hi"; //OK! it_c is a const pointer to non-const object
   it_c++; //not ok! can’t change where a const pointer points! 
   ```

   不可以改变指向，但可以改变指向的内容（const修饰的是iterator这个指针，不可变）

2. const_iterator 

   ```C++
   const_iterator c_it = vec.begin();
   c_it++; // totally ok! The pointer itself is non-const
   *c_it = "hi" // not ok! Can’t change underlying const object
   cout << *c_it << endl; //allowed! Can always read a const object, just can't change
   ```

   不可以改变的指向的内容，但可以改变指向（const修饰的是变量）

3. const const_iterator

   ```C++
   const const_iterator c_it_c = vec.begin();
   cout << c_it_c << " points to " << *c_it_c << endl; //only reads are allowed!
   c_it_c++; //not ok! can’t change where a const pointer points!
   *c_it_c = "hi" // not ok! Can’t change underlying const object
   
   ```

   不可以改变指向，也不可以改变指向的内容

总结：

- [x] |     iterator类型     | Increment Iterator? | Change underlying value? |
  | :------------------: | :-----------------: | :----------------------: |
  |       iterator       |          Y          |            Y             |
  |    const iterator    |          N          |            Y             |
  |    const_iterator    |          Y          |            N             |
  | const const_iterator |          N          |            N             |


## static_cast and const_cast

1. static_cast
   - 主要用于执行非常普通的类型转换，通常用于安全的转换，例如基本数据类型之间的转换、子类到父类指针或引用的转换等
   - 在编译的时候就会进行类型转换的安全性检查，通常不会引起运行时错误
   - ==不会删除或添加 `const` 修饰符，所以它不能用于改变一个对象的常量性。==（can not remove constness）
2. const_cast
   - 用于添加或删除 `const` 修饰符。它通常用于去除对象的 `const` 限制，当然也可以为non-const对象添上const限制



## Recap1:: Const and Const-correctness

- 应用代码中尽可能使用const参数和const变量
- 所以不改变成员变量的成员函数应该被标记为const
- 不要重复造轮子，用static_cast/const_cast来复用non-const的代码以实现const版本

- 为你的类创建iterator和const_iterator两种迭代器

-  `auto` 关键字进行类型推断时，它会忽略原始变量的 `const` 和引用 (`&`) 修饰符。

  ```C++
  const int x = 42;
  auto y = x; // y 的类型将是 int，不再是 const int
  
  int a = 10;
  int& b = a;
  auto c = b; // c 的类型将是 int，不再是 int&
  ```

  除非你显式加上，即使用 `const auto` 或 `auto&` 来显式指定。



## Recap2:: Template classes

- 头文件进行类定义时，加上`template<class T1,typename T2,...>`

  ```C++
  template<typename First, typename Second> class MyPair {
  public:
      First getFirst();
      Second getSecond();
      void setFirst(First f);
      void setSecond(Second f);
  private:
      First first;
      Second second;
  };
  ```

- cpp文件实现成员函数时，在函数签名前也需要加上`template<class T1,typename T2,...>`，并且注意函数的namespace不仅仅是类名。

  ```C++
  template<class First, typename Second>
  	First MyPair<First, Second>::getFirst(){
  	return first;
  }
  ```

- 当返回类型为在类中使用`using`定义的nested type，如`iterator`，使用`typename ClassName<T1,T2...>::iterator`作为返回类型

  ```C++
  template <typename T>
  typename vector<T>::iterator vector<T>::begin() {...}
  ```

  