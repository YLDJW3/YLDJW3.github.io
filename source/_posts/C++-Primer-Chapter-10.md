---
title: C++ Primer Chapter 10
date: 2022-02-16 16:32:13
tags: C++
categories: C++
mathjax: true
---

# 第十章 泛型算法

- 标准库定义了一组泛型算法，称其为**算法**是因为它们实现了经典算法的公共接口，称其为**泛型**是因为他们可用于不同了类型的元素和多种容器类型

## 10.1 概述

- 大多数算法都定义在头文件`algorithm`中，头文件`numeric`定义了一组数值泛型算法
- 算法不支持操作容器，而是遍历两个迭代器hiding的一个元素范围

### 算法`find`

- 接受一对迭代器`b`、`e`，和一个值`v`，在迭代器指定的范围$[b,e)$内搜索等于给定值的元素，若找到则返回指向该元素的迭代器，否则返回迭代器`e`表示搜索失败

### 迭代器使算法不依赖于容器

- 通过迭代器，无论何种容器类型都可以执行该算法

- 但算法**依赖于元素类型的操作**，如需要使用元素类型的`==`、`>`等关系运算符等

- 泛型算法永远**不会执行容器的操作**，只执行迭代器的操作

<!--more-->

## 10.2 初识泛型算法

- 大多数算法都对一个范围内的元素进行操作，称该范围为**输入范围**

### 只读算法

- 只读取输入范围内的元素，而不改变元素

#### `accumulate`算法

- 定义在头文件`numeric`中

- 接受三个参数，前两个参数指定了需要**求和**的元素范围，第三个参数是**和的初值**

- **第三个参数的类型**，决定了函数中使用的加法运算符，以及**返回值的类型**

- 将`vector<string>`中的所有string连接起来

  ```c++
  vector<string> a{...};
  string s;
  accumulate(a.begin(), a.end(), s);
  // 传入的string对象s指明了返回值类型为string, 而且加法使用的是string类型的加法运算符
  ```

#### `equal`算法

- 比较两个序列是否保存相同的值
- 接受三个参数，**前两个参数表示第一个序列的范围，第三个参数是第二个序列首元素的迭代器**
- **假定：第二个序列至少与第一个序列一样长**，它会将第一个序列中的每个元素都与第二个序列的对应元素进行比较

- 所有接受单一迭代器表示第二个序列的算法，都假定第二个序列的长度至少为第一个序列的长度

### 写容器元素的算法

#### `fill`算法

```c++
fill(b, e, v);
```

- 接受一对迭代器表示范围，接受一个值作为第三个参数
- **将给定的值赋予输入范围中的每个元素**

#### 算法不执行写检查

- 向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素

#### `fill_n`算法

```c++
fill_n(b, n, v);
```

- 将给定值`v`赋予`b`开始的`n`个元素
- 假如迭代器`b`开始的序列不足`n`个元素，将引发未定义行为

#### 插入迭代器`back_inserter`

- **插入迭代器**是一种**向容器中添加元素**的迭代器
  - 当我们通过**迭代器**向容器元素赋值时，值被赋予迭代器指向的元素
  - 当我们通过**插入迭代器**赋值时，一个与赋值号右侧相等的元素被添加到容器中

- `back_inserter`接受一个指向容器的引用，**返回一个与该容器绑定的插入迭代器**，通过该插入迭代器**赋值**时，将**调用`push_back`将一个具有给定值的元素添加到容器中**

- 使用`back_inserter`

  ```c++
  vector<int> a{};
  auto it = back_inserter(a);
  *it = 10;
  // it调用push_back将与赋值运算符右侧值(10)相等的元素添加到a中
  ```

- 将`back_inserter`运用在算法的目的位置

  ```c++
  vector<int> a{};
  fill_n(back_inserter(a), 10, 1);
  // 合法, a将添加10个值为1的元素
  ```

#### `copy`算法

- 接受三个迭代器，前两个参数代表第一个序列的范围，第三个参数第二个序列的起始位置
- 传递给`copy`的第二个序列至少要包含与输入序列一样多的元素
- `copy`返回目的位置迭代器最终的值，即**指向最后一个拷贝元素的下一个位置**

#### 算法的拷贝版本

- “拷贝版本”指算法不改变输入序列，而是创建一个新序列保存这些结果

##### `replace`算法

```c++
replace(b, e, v1, v2);
// 将输入范围内等于v1的元素的值改为v2
```

##### `replace_copy`算法

```c++
replace(b, e, b1, v1, v2);
// 输入序列的值不变, b1开始的序列存放了输入序列, 且值等于v1的元素的值改为了v2
```

### 重排容器元素的算法

#### `sort`算法

- 接受两个迭代器表示输入范围，对其进行排序

#### `unique`算法

- 接受两个迭代器表示的输入范围
- 对其重排，将相邻的重复项”消除“。**不重复元素出现在序列开始部分，返回一个指向不重复值范围末尾的迭代器**，此位置后的元素仍然存在
- 因为**算法不能直接添加或删除元素**，所以`unique`只能将重复元素排列在返回位置之后，不能直接删除它们

## 10.3 定制操作

### 向算法传递函数

#### 谓词

- **谓词**是一个**可调用的表达式**，其返回结果是**一个能用作条件的值**
- 谓词分为**一元谓词（接受单一参数）、二元谓词（接受两个参数）**
- 接受谓词参数的算法**对输入序列中的元素调用谓词**，**元素类型必须能转换为谓词的参数类型**

#### sort的重载版本

- sort的重载版本接受第三个参数，是一个谓词，它用该谓词代替`<`比较元素

- 将`vector<string>`按字典序排序

  ```c++
  bool f(const std::string& s1, const std::string& s2) {
      return s1.size() < s2.size();
  }
  
  int main(void) {
      std::vector<std::string> a{"hello", "hi", "world!"};
      sort(a.begin(), a.end(), f);
      for (auto &s : a)
          std::cout << s << " ";
      std::cout << std::endl;
  } 
  ```


#### `stable_sort`算法

- **维持相等元素的原有顺序**

#### `partition`算法

- 接受一对迭代器，和一个谓词
- 对输入范围内的元素调用该谓词，若为true则排在容器前半部分，若为false则排在后半部分
- 返回指向最后一个使谓词为true的元素之后的位置

### lambda表达式

- 谓词可以是一元谓词或二元谓词，因此它最多接受两个参数，若需要更多参数，则无法使用谓词

#### 可调用对象

- 我们可以向算法传递任何类别的可调用对象
- 若一个对象或表达式可以使用**调用运算符**，则为**可调用对象**
- 可调用对象共有四种，分别是**函数、函数指针、lambda表达式、重载了函数调用运算符的类**

#### 定义lambda表达式

- lambda表达式一是个可调用的代码单元，**包含一个返回类型、一个参数列表、一个函数体**

- 可以将其理解为未命名的内联函数

- lambda表达式**可以定义在函数内部**

- 形式

  ```c++
  [capture list] (parameter list) -> return type {funtion body}
  ```

  - 必须**使用尾置返回来指定返回类型**
  - **capture list（捕获列表）**是一个lambda所在函数中定义的局部变量的列表（通常为空）

- 定义

  ```c++
  auto f = [] { return 43; }
  ```

  - 可以**忽略参数列表、返回类型**
  - 必须包含捕获列表、函数体
  - 忽略参数列表：等价于指定**空的参数列表**
  - 忽略返回类型：若函数体**只包含一条return语句**，则返回类型从返回表达式的类型推断；**否则，返回类型为void**

#### 调用lambda表达式

- **调用方式**：与普通函数一样，**使用调用运算符**

  ```c++
  auto f = [] { return 43; }
  cout << f() << endl;
  ```

- 向lambda传参时，**不能有默认参数**，因此传的实参数量一定等于形参数量

#### 使用捕获列表

- 空捕获列表表明此lambda不使用它**所在函数的局部变量**
- 只有在捕获列表中捕获了它所在函数的局部变量，才能在函数体中使用该变量
- 捕获多个变量，通过**逗号运算符**分隔
- 捕获列表**只用于局部非static变量**，对于局部static变量以及所在函数外声明的名字，lambda表达式可直接使用

### `bigges`函数

#### `find_if`算法

- 接受一对迭代器，和一个谓词
- 对输入范围内的每个元素调用该谓词，返回第一个使谓词返回非0值的元素，若不存在则返回尾后迭代器

- `find_if`算法接受的谓词**只能是一元谓词**

#### `for_each`算法

- 接受一对迭代器，和一个可调用对象
- 对输入范围内的每个元素调用该可调用对象

### lambda捕获和返回

- **定义一个lambda**时，编译器生成一个与lambda对应的新的（未命名的）类类型
- **向一个函数传递一个lambda**时，同时定义了一个新类型和该类型的一个对象
- 从lambda生成的类都包含一个**对应lambda所捕获的变量的数据成员**，lambda的数据成员也在lambda对象创建时被初始化
- 变量捕获有两种方式：**值捕获**和**引用捕获**

#### 值捕获

- 采用值捕获的前提是变量能够被拷贝
- 被捕获的变量**在lambda对象创建时被拷贝**（不是调用时被拷贝）

#### 引用捕获

- 采用引用方式捕获变量，变量名前加`&`

  ```c++
  size_t v1 = 42;
  auto f2 = [&v1] { return v1; };
  ```

- 采用引用捕获的方式，需要**保证被捕获的局部变量在lambda执行时是存在的**

#### 隐式捕获

- 让编译器根据lambda体中的代码自动捕获变量

- 在捕获列表中写一个`&`，指示采用引用捕获方式

- 在捕获列表中写一个`=`，指示采用值捕获方式

  ```c++
  size_t v1 = 42;
  auto f2 = [=] { return v1; };	// 告诉编译器采用值捕获
  ```

#### 混合使用隐式捕获和显式捕获

- 捕获列表的第一个元素必须是`&`或`=`，指示默认捕获方式
- 混合使用时，**只有与默认捕获方式不同的变量才可以显式捕获**

#### 可变lambda

- 若希望改变值捕获的变量的值，需要**在参数列表后加上关键字`mutable`**

  ```c++
  size_t v1 = 42;
  auto f = [v1] () mutable { return ++v1; };
  auto j = f();	
  // j为43
  ```

#### 指定lambda返回类型

- 假如lambda体并非只包含一个return语句，则默认返回void
- 此时，若想返回值，必须**使用尾置返回类型的形式指定返回类型**

#### 返回lambda

- 函数可以返回lambda，但此lambda不能包含引用捕获，因为函数结束时其捕获的局部变量会被销毁，因此返回的lambda捕获的引用是未定义的

### 参数绑定

#### 标准库`bind`函数

- 定义在头文件`functional`中

- bind函数可看作一个通用的函数适配器，**接受一个可调用对象，生成一个新的可调用对象**

  ```c++
  auto newCallable = bind(callable, arg_list);
  ```

  - `newCallable`是一个可调用对象
  - `arg_list`是一个**逗号分隔的参数列表**，对应给定的`callable`的参数
  - 当我们调用`newCallable`时，它会调用`callable`，并传递给它`arg_list`中的参数

##### `arg_list`中的占位符

- `arg_list`中包含形如`_n`的名字，称为**占位符**，表示`newCallable`的参数，它们**占据了传递给`newCallable`的参数的“位置”**
- 其中`_i`表示`newCallable`第i个参数

#### 使用placeholders的名字

- 形如`_n`的占位符都定义在`placeholders`命名空间中，而该空间又定义在`std`中
- 因此`_1`的完整形式为`std::placeholders::_1`
- `placeholders`也定义在`functional`头文件中

#### 使用bind重排参数顺序

- 用bind重排`isShorter`的参数顺序，即`f=bind(isShorter, _2, _1)`

#### 绑定引用参数——标准库`ref`或`cref`

- bind中不是占用符的参数，将被**拷贝**到bind返回的可调用对象中
- 假如该参数是不可拷贝的类型，或者我们想以引用方式传递，则可以**使用标准库的`ref`函数或`cref`函数**

- `ref`**返回一个对象，包含给定的引用**，该对象是可以拷贝的

- `cref`**返回一个对象，包含给定的const引用，**该对象也是可以拷贝的

- `ref`和`cref`也定义在头文件`functional`中

  ```c++
  auto f = bind(g, _1, cin);		// 非法, cin是IO类型, 不能拷贝
  auto f = bind(g, _1, ref(cin)); // 合法, ref(cin)返回包含对cin引用的对象
  ```

## 10.4 再探迭代器

- 除了容器定义的迭代器外，标准库在`iterator`头文件中定义了几种特殊迭代器
  - **插入迭代器**，绑定到一个容器上，用于**向容器插入元素**
  - **流迭代器**，绑定到输入或输出流，**遍历所关联的IO流**
  - **反向迭代器**，不是向前移动，而是向后移动，除了`forward_list`的标准库容器都有反向迭代器
  - **移动迭代器**，用于移动元素，而不是拷贝元素

### 插入迭代器

- 插入迭代器是一种迭代器适配器，接受一个容器，生成一个迭代器
- 当通过插入迭代器赋值时，该迭代器将调用容器操作向给定容器的指定位置插入一个元素

#### `back_inserter`

- 创建一个使用`push_back`的迭代器

#### `front_inserter`

- 创建一个使用`push_front`的迭代器

#### `inserter`

- 创建一个使用`insert`的迭代器，它接受第二个参数，是一个指向给定容器的迭代器。元素将被插入到第二个参数指定的位置之前

### iostream迭代器

- `istream_iterator`读取输入流，`ostream_iterator`向一个输出流写数据

#### `istream_iterator`操作

- 创建流迭代器时，必须**指明读写的对象类型**，且该对象类型必须定义了相应的输入运算符`>>`或输出运算符`<<`

- **默认初始化迭代器**可以当作**尾后值使用**

- 用输入流对象初始化，则将`istream_iterator`**绑定到该输入流上**

  ```c++
  vector<int> a;
  istream_iterator<int> in_iter(cin);
  istream_iterator<int> eof;
  while (in_iter != eof) {
      a.push_back(*in_iter++);
  }
  ```

- 构造vector时传入一对`istream_iterator`

  ```c++
  istream_iterator<int> in_iter(cin);
  istream_iterator<int> eof;
  vector<int> a(in_iter, eof);
  ```

#### 使用算法操作流迭代器

- 使用流迭代器调用`accumulate`

  ```c++
  istream_iterator<int> in_iter(cin);
  istream_iterator<int> eof;
  cout << accumulate(in_iter, eof, 0) << endl;
  ```

#### `istream_iterator`允许使用懒惰求值

- 将一个`istream_iterator`绑定到一个流时，不保证迭代器立即从流读取数据，标准库可以**推迟从流中读取数据，直到我们使用迭代器时才真正读取**

#### `ostream_iterator`操作

```c++
ostream_iterator<T> out(os);
ostream_iterator<T> out(os, d);
```

- 必须将`ostream_iterator`绑定到一个输出流中，不允许空的或表示尾后位置的`ostream_iterator`
- `ostream_iterator`可以接受第二个参数`d`，必须是一个C风格字符串，这样通过`ostream_iterator`写入到输出流的**每个值后，都将输出一个`d`**

### 反向迭代器

- `rbegin`，`rend`，`crbegin`，`crend`

#### 反向迭代器需要递减运算符

- 反向迭代器需要递减运算符，因此`forward_list`和流迭代器没有反向迭代器

#### 反向迭代器和迭代器的关系

- 调用`reverse_iterator`的**`base`成员函数**能将反向迭代器转换回普通迭代器
- 但需要注意转换后的迭代器，与原来的反向迭代器**并非指向相同的元素**，（但是转换前后的迭代器对会指向同一范围，左闭右开区间）

## 10.5 泛型算法结构

- 算法的最基本特性是**它要求迭代器提供哪些操作**

### 5类迭代器

- **输入迭代器**，只读不写，单遍扫描，只能递增
- **输出迭代器**，只写不读，单遍扫描，只能递增
- **前向迭代器**，可读写，多遍扫描，只能递增（`forward_list`上的迭代器）
- **双向迭代器**，可读写，多遍扫描，可递增递减
- **随机访问迭代器**，可读写，多遍扫描，支持全部迭代器运算（`array`、`deque`、`string`、`vector`的迭代器，用于访问内置数组元素的指针）

### 算法形参模式

- 接受**单个目标迭代器**的算法，假定写入多少个元素都是安全的
- 接受**第二个输入序列**的算法，若接受单独的`beg2`，则假定从`beg2`开始的范围至少和第一个输入序列一样大

### 算法命名规范

- `_if`版本
- 拷贝版本`_copy`和不拷贝版本

- 同时提供版本`_copy_if`

## 10.6 特定容器算法（`list`和`forward_list`）

- 略

## 习题

- words是一个`vector<string>`，对其进行划分，并打印长度大于等于5的元素

  ```c++
  bool f(const std::string& s) {
      return s.size() >= 5;
  }
  
  int main(void) {
      std::vector<std::string> a{"hello", "hi", "world", "alpha"};
      auto e = partition(a.begin(), a.end(), f);
      for (auto it = a.begin(); it != e; ++it) {
          std::cout << *it << " ";
      }
      std::cout << std::endl;
  } 
  ```

  

 
