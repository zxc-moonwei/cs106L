预测函数

- 预测函数的参数数量不尽相同

  一元

  ```C++
  bool isLowercaseA(char C) {
  	return C == 'a';
  }
  bool isVowel (char C) {
  	std: :string vowels = " aeiou" ;
  	return vowels.find(C) != std::string::npos ;
  }
  
  ```

  二元

  ```C++
  bool isMoreThan (int num, int limit) {
  	return num > 1imit;
  }
  bool isDivisibleBy(int a, int b) {
  	return (a % b == 0);
  }
  ```

使用预测函数后，代码改写成：

```C++
template <typename InputIt, typename UniPred>
int count_ occurrences ( InputIt begin, InputIt end, UniPred pred ){          //参数为一元预测函数 
	int count = 0;
	for (auto iter = begin; iter != end; ++iter) {
		if (*iter == val pred(*iter) ) count++ ;
	}
	return count;
}
// 具体的一元预测函数，判断应该字符是否是元音
bool isVowel (char c) {
	std::string vowels = " aeiou" ;
	return vowels. find(c) != std::string::npos;
}

//Usage: 
std::string str = "Xadia";
count_ occurrences(str.begin(), str.end(), isVowel) ;

```

问题来了：`UniPred`是什么类型？

它是一个函数指针。

- 就和其他指针类似
- 可以被当作参数传递，如同上例中的`isVowel`一样

- 它们可以像函数一样被调用



`UniPred`有什么问题？

不够通用，一元预测函数能涵盖的功能的功能太有限了，万一我想用二元、甚至更多元的预测函数怎么办？

```C++
bool isMoreThan3(int num) {
	return num > 3;
}
bool isMoreThan4 (int num) {
	return num > 4;
}
bool isMoreThan5(int num) {
	return num > 5;
}
// a generalized version of the above
bool isMoreThan(int num, int limit) {
	return num > 1 imit;
}
```



试着编写下可以应对该种情况的模板函数：

```C++
template <typename InputIt, typename_ BinPred>
int count_ occurrences ( InputIt begin, InputIt end, BinPred pred) {
	int count = 0;
	for(auto iter = begin; iter != end; ++iter) {
		if (pred(*iter， ???)) count++;
    }
    return count;
}

```

现在我们知道预测函数需要两个参数？那么另外一个参数填什么呢，总不能在模板函数的参数中吧预测函数的另一个参数传进来吧？如果是三元、四元呢，注意也太不优雅了。



我们想要预测函数知道更多的信息，却无法传递更多的参数进来。

如何在不需要增加参数的情况下，让预测函数知道这些信息呢？



Lambda

内联，匿名，可以访问相同作用域下的变量及函数。

![image-20231012145233701](C:\Users\tfsx01\AppData\Roaming\Typora\typora-user-images\image-20231012145233701.png)

Capure Clauses

"捕获"意味着将外部变量引入到匿名函数的内部，使匿名函数可以访问和使用这些外部变量。

类似这样：

![image-20231012145719075](C:\Users\tfsx01\AppData\Roaming\Typora\typora-user-images\image-20231012145719075.png)

既可以捕获值，也可以捕获引用

![image-20231012145842142](C:\Users\tfsx01\AppData\Roaming\Typora\typora-user-images\image-20231012145842142.png)

这样问题即可解决

![image-20231012150134195](C:\Users\tfsx01\AppData\Roaming\Typora\typora-user-images\image-20231012150134195.png)



Functor(函数对象)

函数对象是任意一个实现了operator()函数的类。

下面演示利用函数对象如何生
