---
title: C++ Primer Chapter 9
date: 2022-02-15 20:06:21
tags: C++
categories: C++
mathjax: true
---

# 第九章 顺序容器

## 9.1 顺序容器概述

### 顺序容器类型

- vector、deque、list、forward_list、array、string
- string和vector将元素保存在连续的内存空间中，元素是连续存储的
- list和forward_list的设计目的是，令容器任何位置的添加和删除操作都很快速，但访问某一个元素必须得遍历整个容器
- deque与string和vector类似，但在它两端添加或删除元素都是很快的
- forward_list是C++11新增的类型，设计目标是**达到与最好的手写的单向链表数据结构相当的性能**，它**没有`size()`操作**
- array是C++11新增的数组类型，array对象的大小是固定的，不支持增加、删除元素及改变容器大小的操作
- 对于除forward_list以外的容器，`size()`是一个快速的常量时间的操作

### 顺序容器类型选择

- 没有更好理由选择其他容器时，**通常选择`vector`**
- 程序要求频繁**在容器的中间插入或删除元素，则使用`list`或`forward_list`**
- 程序要求在**头尾位置插入或删除元素**，而不会在中间位置插入或删除元素，则**使用`deque`**

<!--more-->

## 9.2 容器库概览

### 容器类型操作的层次

- 某些操作是所有容器类型都提供的
- 一些操作仅针对顺序容器，一些操作仅针对关联容器，一些操作仅针对无需容器
- 一些操作只适用于一小部分特定容器

### 容器的定义

- 一般来说，每个容器都定义在一个头文件中，**文件名与容器类型名相同**
- 容器均定义为**模板类**，还需要额外信息生成特定的容器类型

### 对容器可以保存的元素类型的限制

- 某些容器操作对元素类型由特殊要求，可以为不支持特定操作需求的类型定义容器，但是这些容器就不能使用这些操作
- 如顺序容器构造函数的一个版本接收容器大小参数，它使用了元素类型的默认构造函数，若某类型没有默认构造函数则不能使用这种构造函数来构造容器

### 迭代器

- `forward_list`迭代器不支持递减运算符
- 迭代器支持的**算术运算（加减）只能应用于string、vector、deque和array的迭代器**，不能用于其他容器类型的迭代器

#### 迭代器范围

- 一个迭代器范围由一对迭代器表示，两个迭代器分别指向同一个容器中的元素，或该容器尾元素之后的位置，常用`begin`和`end`表示
- 可以通过反复递增`begin`来到达`end`，即`end`不在`begin`之前
- **若`begin==end`，则代表范围为空**，因此最好使用`begin!=end`，而不是`begin<end`作为循环条件

### 容器类型成员

- 每个容器都定义了多个类型，如`size_type`，`iterator`和`const_iterator`

- 大多数容器还提供**反向迭代器**，这是一种反向遍历容器的迭代器。对反向迭代器执行`++`操作会得到上一个元素

- 类型别名，我们可以在不了解容器中元素类型的情况下使用它，若需要元素类型可使用容器的`value_type`

  ```c++
  // 定义list<string>的迭代器
  list<string>::iterator it;
  ```

### begin和end成员

- `begin`和`end`成员分别生成指向容器中的第一个元素和尾后位置的迭代器
- `cbegin`和`cend`生成**const迭代器**
- `rbegin`和`rend`生成**反向迭代器**

#### 重载

- 不以c开头的成员都被重载过
- **实际上有两个版本的`begin`，分别返回迭代器和const迭代器**，若调用的容器是非常量对象将返回普通迭代器，若为常量对象则返回const迭代器

- 当不需要写操作时，应使用`cbegin`和`cend`

### 容器定义和初始化

#### 将一个容器初始化为另一个容器的拷贝

- 两种方式

  - 直接拷贝整个容器
  - 拷贝由一个迭代器对指定的元素范围

- 采用第一种方式时，两个容器的容器类型和元素类型都必须相同

- 采用第二种方式时，容器类型可以不同，元素类型也可以不同，只要能够转换即可

  ```c++
  vector<int> a1(a2);		// a2必须也是vector<int>
  vector<int> a1(a2.begin(), a2.end());	// a2元素类型可以转换为int即可
  ```

#### 列表初始化

- 对容器列表初始化

#### 与容器大小相关的构造函数

- 接受一个容器大小和一个元素初始值，或只接受一个容器大小
- 如果不提供元素初始值，则标准库会创建一个**值初始化**器
- **如果元素是没有默认构造函数的类类型，则必须提供初始值**

#### 标准库array具有固定大小

- 与内置数组一样，array的大小也是类型的一部分。定义array时，除了指定元素类型，还要指定容器大小

  ```c++
  array<int, 42> a;
  ```

- 一个默认构造的array包含了和其大小一样多的元素，这些元素被**默认初始化**

- 如果用列表初始化array，但列表初始值数量小于array大小，剩下的元素将被**值初始化**

- 我们不能对内置数组拷贝，但可以对array对象拷贝

  ```c++
  array<int, 10> a1;
  array<int, 10> a2 = a1;		// 合法, 可以拷贝array对象
  ```

  

### 赋值和swap

- 赋值运算符将左边容器中的全部元素替换为右边容器元素的拷贝
- 如果两个容器原来大小不同，则**赋值后左边容器的大小会与右边容器的大小相同**

#### array类型的赋值

- array可以赋值，左右两侧的运算对象必须**具有相同的类型**
- array类型不支持列表赋值，也不支持assign

#### 使用assign（仅顺序容器，array除外）

```c++
a.assign(b, e);		// 将容器a中元素替换为迭代器对b和e表示的元素
a.assign({1,2,3});	// 将容器a中元素替换为初始化列表中的元素
a.assign(n, t);		// 将容器a的元素替换为n个值为t的元素
```

- assign允许我们从一个**不同但相容的类型**赋值

- 传入assign的迭代器**不能指向调用assign的容器本身**

#### 使用swap

- swap**交换两个相同类型容器的内容**
- swap不会进行实际的交换，只是交换两个容器的内部数据结构，这种操作很快，能在`O(1)`时间内完成

#### assign和swap对迭代器、指针、引用的影响

- **`assign`和`=`赋值**将使指向左侧容器的**迭代器、指针、引用失效**
- **`swap`不会**导致迭代器、指针、引用失效，且它们在交换前后**指向同一个元素值，属于不同的容器**

- 特别地，对一个string调用swap将导致迭代器、引用和指针失效

#### swap两个array

- swap两个array会真正交换两个容器的元素
- 指针、引用和迭代器所绑定的元素保持不变，但**元素值会与另外的元素进行交换**

#### 非成员版本的swap

- C++11提供了非成员版本的swap，在泛型编程中非常仲要，尽量统一使用非成员版本的swap

### 容器大小操作

- `size`返回元素数目
- `empty`在不包含任何元素时返回true

- **`max_size`返回一个大于或等于该类型容器所能容纳的最大元素的值**
- **`forward_list`支持`max_size`和`empty`**，但不支持`size`

### 关系运算符

- 每个容器类型都支持相等运算符`==`和`!=`
- 除了无序关联容器外所有容器都支持关系运算符`>`，`>=`，`<`，`<=`
- 关系运算符两边的运算对象必须是相同类型的容器，存放相同类型的元素

#### 容器的关系运算符使用元素的关系运算符完成比较

- 只有元素类型定义了相应的比较运算符时，我们才可以使用关系运算符比较两个容器

## 9.3 顺序容器操作

### 向顺序容器添加元素（array除外）

#### 容器元素是拷贝

- 用一个对象初始化容器，或将对象插入到容器时，**实际上放入容器中的是对象的一个拷贝，而不是对象本身**

#### `push_back`

- 除了`array`和`forward_list`外，每个顺序容器都支持`push_back`

#### `push_front`

- `list`、`forward_list`和`deque`容器还支持`push_front`，将元素插入到容器头部

#### 在容器中的特定位置添加元素

- `vector`、`deque`、`list`和`string`都支持`insert`成员，它**接受一个迭代器和一个元素，将元素插入到该迭代器所指的位置之前**

  ```c++
  vector<int> a{1,2,3};
  a.insert(a.begin(), 0);
  // a变为{0,1,2,3}
  ```

- 将元素插入到任意位置的操作，可能很耗时（`O(N)`）

- 插入范围内的元素

  ```c++
  a.insert(iter, val);		// 插入一个元素
  a.insert(iter, b, e);		// 插入迭代器对b、e范围内的元素
  a.insert(iter, n, v);		// 插入n个元素, 初始化为v
  a.insert(iter, {...});      // 插入列表内的元素
  ```

- 若`insert`传入一对迭代器，则迭代器不能指向容器本身

#### 使用inesrt的返回值

- 接受元素个数或迭代器对的insert版本，**返回值是指向第一个新加入元素的迭代器**

#### `emplace`操作

- C++11新增三个成员`emplace`、`emplace_back`和`emplace_front`，这些操作**构造**而不是拷贝元素

- 与`push_back`、`insert`等的区别
  - 前者将传入对象的拷贝加入容器中
  - `emplace`**将参数传递给元素类型的构造函数**，并将构造的元素添加到容器中

- 传递给`emplace`的参数必须与容器的元素类型的构造函数相匹配

### 访问元素

#### `back`和`front`成员

- 每个顺序容器都有`front`成员，返回指向首元素的引用
- 除了`forward_list`外的每个顺序容器都有`back`成员，返回指向尾元素的引用

- 调用`front`和`back`之前要**确保容器非空**，如果非空则其行为是未定义的

#### 下标

- string、vector、deque和array都提供下标运算符

- `c[n]`返回c中下标为n的元素的引用，下标越界将产生未定义的行为

#### `at`成员

- `c.at(n)`返回下标为n的元素的引用，若下标越界则抛出out_of_range异常

#### 访问成员函数返回的是引用

- 上述操作都将返回指向容器元素的**引用**，如果容器是const对象则返回**const引用**

### 删除元素

#### `pop_back`和`pop_front`

- 支持`pop_back`的容器：string、vector、list、deque
- 支持`pop_front`的容器：list、forward_list、deque
- 返回void

#### 从容器内部删除一个元素

- `a.erase(it)`，删除it指向的元素，返回指向被删除元素下一位置的迭代器

#### 从容器内部删除多个元素

- `erase`接受一对迭代器的版本，`a.erase(b, e)`，返回指向最后一个被删除元素下一位置的迭代器

#### `clear`

- **删除所有元素**，返回void

#### 删除元素对迭代器、引用和指针的影响

- 删除`deque`**除首尾位置外**的任何元素，都将使所有迭代器、指针、引用失效
- 删除`vector`、`string`的元素，**指向删除点之后位置的迭代器、指针、引用会失效**

### 总结各个顺序容器操作的返回值

#### 添加元素

- 返回void：`push_back`，`push_front`，`emplace_back`，`emplace_front`
- `insert`返回指向第一个新添加元素的迭代器

#### 访问元素

- 返回指向该元素的**引用**

#### 删除元素

- 返回void：`pop_back`，`pop_front`，`clear`
- 返回指向最后一个删除元素下一位置的迭代器：`erase`

### 特殊的forward_list操作

- `insert_after`，`emplace_after`，`erase_after`，`before_begin`
- `before_begin()`返回指向链表首元素前的位置的迭代器
- `insert_after`的不同版本，返回指向指向最后一个插入元素的迭代器，若无插入元素则返回p
  - `insert_after(p, t)`，在迭代器p后的位置插入一个元素
  - `insert_after(p, n, t)`，在迭代器p后的位置插入n个初始化值为t的对象
  - `insert_after(p, b, e)`，在迭代器p后的位置插入迭代器范围`[b, e)`之间的元素的拷贝
  - `insert_after(p, li)`，在迭代器p后的位置插入列表`li`中的元素的拷贝

- `emplace_after`，将参数传递给元素类型的构造器，并将构造的对象插入到迭代器p后的位置
- `erase_after`的不同版本
  - `erase_after(p)`，删除p指向的位置之后的元素
  - `erase_after(b, e)`，删除从b后一个位置，到e之间的元素，**返回一个指向最后一个被删除元素的下一位置的迭代器**

### 改变容器大小`resize`

- `resize`的两种版本
  - `c.resize(n)`，调整容器大小为n
  - `c.resize(n, t)`，调整容器大小为n，任何新添加的元素都用值t初始化
- 假如n小于原大小，则容器后部的元素被删除
- 假如n大于原大小，则将新添加元素，**若没提供初始值将执行值初始化**，否则用值t初始化新元素
- 对于类类型，若向容器添加新元素，则必须提供初始值或默认构造函数

### 容器操作可能使迭代器失效

- 向容器添加元素或删除元素，都可能使容器元素的指针、引用、迭代器失效。使用失效的指针、引用、迭代器是一种严重的程序设计错误

#### 规则

- 添加元素时
  - 对于`string`和`vector`，若存储空间重新分配，则所有迭代器、指针、引用失效；若存储空间未重新分配，则指向插入位置后的迭代器、指针、引用失效
  - 对于`deque`，插入到首尾位置外的任何位置都将使所有迭代器、指针、引用失效；插入首尾位置，则迭代器失效，但引用、指针仍有效
  - 对于`list`和`forward_list`，所有迭代器、指针、引用仍有效
- 删除元素时
  - 对于`list`和`forward_list`，仍有效
  - 对于`deque`，删除首尾位置外的元素将使所有迭代器、指针、引用失效；删除尾元素使尾后迭代器失效；删除首元素，迭代器、指针、引用仍有效
  - 对于`string`和`vector`，指向被删元素前的迭代器、引用、指针仍有效

#### 在循环中添加/删除元素

- 考虑迭代器、引用、指针的失效问题，每个循环步都要更新迭代器、引用或指针，**运用`insert`和`erase`的返回值**
- **不要预存`end`成员返回值**，应该在每次使用时都重新调用`end()`
- 因为范围for循环预存了`end`成员，因此**不能在范围for循环中添加/删除元素**

## 9.4 vector对象是如何增长的

### vector和string对象是连续存储的

- 当没有空间容纳新元素时，vector必须分配新的内存空间，将元素从旧空间移动到新空间，添加新元素，并释放旧存储空间
- 标准库采用可减少容器空间重新分配次数的策略，即获取新空间时通常**分配比新空间需求更大的内存空间，预留这些空间作为备用**

### 管理容量的成员函数

- `shrink_to_fit`只适用于vector、string、deque
- `capacity`和`reverse`只适用于vector、string

#### `capacity`成员

- `c.capacity()`，返回不重新分配内存空间的话，c**可以保存多少个元素**

#### `reverse`成员

- `c.reverse(n)`，**分配至少能容纳n个元素的内存空间**，它不改变容器中元素的数量，只影响vector预先分配多大的内存空间

- 假如n大于当前容量，则改变vector容量，此时`reverse`**至少分配与n一样大的内存空间（可能更大）**
- 若n小于等于当前容量，则`reverse`什么也不做

#### `shrink_to_fit`成员

- **请求**容器退回不需要的内存空间（即**将`capacity()`减少为与`size()`相同的大小**），但C++**可以忽略该请求**

#### 只有在操作需求超出当前容量时，vector才会重新分配内存空间



## 9.5 额外的string操作

### 构造string的其他方法

- 从数组构造string

  - `string s(cp, n)`，s是cp所指数组的前n个字符的拷贝
  - `string s(cp)`，从数组cp创建s，**cp必须以空字符结尾**，拷贝操作遇到空字符时停止

  - 如果未提供计数值`n`且数组不以空字符结尾，或计数值n超过了数组大小，则构造函数的行为是未定义的

- 从string构造string

  - `string s(s1, pos1)`，从s1下标为pos1的字符开始拷贝
  - `string s(s1, pos1, len1)`，从s1下标为pos1的字符开始拷贝，最多拷贝len1个字符（若不足则直到s的最后一个字符）

#### substr操作

```c++
s.substr(pos, len);
```

- 返回一个string，包含s从pos开始的len个字符的拷贝
- 若s剩余字符不足len，则拷贝到结尾为止
- **若起始位置`pos`超过了s的大小，则抛出out_of_range异常**

### string搜索操作

#### 返回值

- 若搜索成功，则返回一个`string::size_type`值，表示匹配位置的下标
- 若**搜索失败，则返回一个名为`string::npos`的`static`成员**

#### 六个成员

- **`find`查找参数指定的字符串**，若找到则返回**第一个匹配位置**的下标，否则返回`npos`

- `rfind`查找指定字符串在s中的**最后一次匹配位置**的下标

- `find_first_of`查找参数中**任意一个字符**第一次出现的位置
- `find_last_of`查找参数中任意一个字符最后一次出现的位置

- `find_first_not_of`查找第一个**不在args的字符**出现的位置
- `find_last_not_of`查找最后一个不在args的字符出现的位置

#### args参数

- args必须是以下形式之一
  - `c,pos`，从s位置pos开始查找**字符c**，pos默认值为0
  - `s2,pos`，从s位置pos开始查找**string对象s2**，pos默认为0
  - `cp,pos`，从s位置pos开始查找cp指向的**C风格字符串**，pos默认为0
  - `cp,pos,n`，从s位置pos开始查找cp指向的数组的前n个字符，pos和n无默认值

### 数值转换

#### 算术类型转换为string

- `to_string(val)`

#### string转换为算术类型

- `stoi(s)`转换为`int`
- `stol(s)`转换为`long`
- `stoll(s)`转换为`long long`
- `stod(s)`转换为`double`

## 9.6 容器适配器

### 适配器

- 适配器是一种机制，能使某种事物的行为看起来像另一种事物一样
- 标准库定义了三个顺序容器适配器，`stack`、`queue`和`priority_queue`

### 定义一个适配器

- 默认构造函数创建一个空对象，如`stack<int> stk;`
- 接受一个容器的构造函数拷贝该容器来初始化适配器，如`stack<int> stk(a)`，`a`是一个vector`<int>`对象
- 默认请况下，**stack和queue是基于deque实现的，priority_queue是基于vector实现的**

### 栈适配器

- pop、push、emplace、top

- 每个适配器都定义了自己的特殊操作，我们只能使用适配器操作，**不能使用底层容器类型的操作**

### 队列适配器

- pop、front、back（只适用于queue）、top、push、emplace
- queue使用一种先进先出的存储和访问策略
- priority_queue为队列中的元素建立**优先级**，**默认情况下标准库在元素类型上使用`<`运算符来确定优先级**（因此越大的元素排列越前，相当于**大顶堆**）

