# C++标准库：泛型算法

## 10.1 概述
- 大多数算法都**定义在头文件 algorithm** 中。
- 标准库还在**头文件 numeric** 中定义了一组**数值泛型算法**。  
- 一般情况下，这些算法并**不直接操作容器**，而是遍历由两个迭代器指定的一个元素范围来进行操作。
	- `find` 算法示例
```c++
int val = 42; // 我们将查找的值

// 如果在vec 中找到想要的元素，则返回结果指向它，否则返回结果为第二个参数表示搜索失败。
auto result = find(vec.cbegin(), vec.cend(), val);

// 报告结果
cout << "The value " << val<< (result == vec.cend()? " is not present" : " is present") << endl;
```

#### 算法如何工作
以`find`算法为例：
- `find`的工作是在一个未排序的元素序列中查找一个特定的元素。步骤如下：
1. 访问序列中的首元素。  
2. 比较此元素与我们要查找的元素。  
3. 如果此元素与我们要查找的值匹配，`find` 返回标识此元素的值。  
4. 否则，`find` 前进到下一个元素，重复执行步骤 2 和 3。  
5. 如果达到序列尾，`find` 应停止。  
6. 如果 `find` 达到序列末尾，它应该返回一个指出元素未找到的值。此值和步骤 3 返回的值必须具有相容的类型。
这些步骤都**不依赖于容器所保存的元素类型。** 因此，只要有一个**迭代器**可用来访问元素， `find` 就完全不依赖于容器类型。

#### 迭代器令算法不依赖于容器
上述的 `find` 函数的流程中，除了第 2 步外，其他步骤都可以用迭代器操作来实现。

#### 但算法依赖于元素类型的操作
虽然迭代器的使用令算法不依赖于容器类型，但大多数算法都使用了一个（或多个）元素类型上的操作。
例如，在步骤 2 中，`find` 用元素类型的 `==` 运算符完成每个元素与给定值的比较。其他算法可能要求元素类型支持 `<` 运算符。不过，我们将看到，大多数算法提供了一种方法，允许我们使用自定义的操作来替代默认的运算符。

>[!warning]
>泛型算法本身不会执行容器的操作，它们只会运行于迭代器之上，执行迭代器的操作。


## 10.2 初识泛型算法
标准库提供了超过100个算法，这些算法都是有一致的结构，下面以几种具体的算法为例介绍标准库中算法的结构。

### 10.2.1 只读算法
- **只读算法：** 只会读取其输入范围内的元素，而从不改变元素。
>例如，`find`算法，`accumulate`算法，`equal`算法。

#### accumulate算法
- `accumulate`算法：对一个序列进行求和。定义在头文件`numeric`中。接受三个参数：
	- 前两个参数：指出了需要求和的元素的范围
	- 第三个参数：是和的初始值。

```c++
// vec 是一个整数序列
// 对 vec 中的元素求和，和的初值是0
int sum = accumulate(vec.cbegin(), vec.cend(), 0);
```
>`accumulate` 的第三个参数的类型决定了函数中使用哪个加法运算符以及返回的类型.

#### 算法和元素类型
`accumulate` 将第三个参数作为求和起点，这蕴含这一个编程假定： 
- 将元素类型加到和的类型上的操作必须可行。 即**序列中的元素的类型必须与第三个参数匹配**，或者**能够转换**为第三个参数的类型，如果类型是自定义类型，那么自定义类型必须定义 `+` 运算符。

```c++
// v 的类型是 vector<string>
string sum = accumulate(v.cbegin(), v.cend(), string("")); // 正确，string可以使用+进行连接

string sum = accumulate(v.cbegin(), v.cend(), ""); // 错误： const char* 上没有定义 + 运算符
```
在第二个求和的调用中，第三个参数是字符串字面值，类型是 `const char*` ，由于 `const char*` 并没有 `+` 运算符，此调用将产生编译错误。

>对于**只读取而不改变元素**的算法，通常最好使用 `cbegin()` 和 `cend()` 。如果需要改变元素则使用 `begin()` 和 `end()`。

#### 操作两个序列的算法
- `equal` 算法：用于确定两个序列是否保存相同的值。它将第一个序列中的每个元素与第二个序列中的对应元素进行比较。
	- 如果所有对应元素都相等，则返回 `true` ， 否则返回 `false` 。
	- 此算法接受三个迭代器：
		- 前两个参数：表示第一个序列中的元素范围，
		- 第三个参数：表示第二个序列的首元素（指需要比较的元素范围的起始位置）
```c++
// roster2 中的元素数目应该至少与 roster1 一样多
equal(roster1.cbegin(), roster1.cend(), roster2.cbegin());
```
由于 `equal` 利用迭代器完成，因此我们可以调用 `equal` 来**比较两个不同类型的容器中的元素**。而且**元素类型也不必一样**，**只要我们能用 `==` 来比较两个元素类型即可**（即两个类型需要有重载的 `==` 支持它们之间进行比较）。

但是，`equal` 基于一个非常重要的假设：它假定第二个序列至少与第一个序列一样长。
>[!warning]
>那些只接受一个单一迭代器来表示第二个序列的算法，都假定第二个序列至少与第一个序列一样长。如`equal`等

### 10.2.2 写容器元素的算法
一些算法将新值赋予序列中的元素。当我们使用这类算法时，必须注意**确保序列原大小至少不小于我们要求算法写入的元素数目**。因为算法不会执行容器操作，它们自身不能改变容器的大小。

- **算法 `fill` ：** 它的接受一对迭代器表示一个范围，还接受一个值作为第三个参数。 `fill` 将给定的这个值赋予输入序列中的每个元素。
```c++
fill(vec.begin(), vec.end(), 0); // 将每个元素重置 0

// 将容器的一个子序列设置为10
fill(vec.begin(), vec.end() + vec.size()/2, 10)
```
由于`fill`向给定输入序列中写入数据，因此，只要我们传递了一个有效的输入序列，写入操作就是安全的。

>[!note]
>一些算法接收两个序列。第二个序列参数传递的方式有两种：
>- 用单一迭代器表示第二个序列：必须保证第二个序列至少比第一个一样长。如果第二个序列是第一个序列的子集，那么将会产生一个严重的错误。算法会试图访问第二个序列末尾之后不存在的元素。
>- 用两个迭代器表示第二个序列。

#### 算法不检查写操作
一些算法接受一个迭代器来指出起始位置，使用一个计数值表示需要写入的个数。
- `fill_n` 函数接受一个单迭代器、一个计数值和一个值，它的功能是从给定迭代器的位置写入 “计数值” 个*数*，*这个数*是由第三个参数给出。
	- 第一个参数：迭代器表示写入目的位置
	- 第二个参数：写入的个数
	- 第三个参数：写入的值
```c++
vector<int> vec; // 空vector

// 使用 vec，赋予它不同的值

fill_n(vec.begin(), vec.size(), 0); // 将所有元素重置为0
```

函数 `fill_n` 假定`dest`指向一个元素，而从`dest`开始的序列至少包含`n`个元素。 `fill_n`假定写入指定元素是安全的，即，如下形式的调用：
```c++
fill_n(dest, n, val)

vector<int> vec; // 空vector
// 灾难：修改 vec 中的10个（不存在）元素
fill_n(vec.begin(), 10, 0);
```

>[!warning]
>- 如果容器不包含写入位置的元素，那么则不能使用该算法。例如，空容器。
>- 向目的位置迭代器写入数据的算法假定目的位置足够大，能容纳要写入的元素  
>- 算法并不会检查写入元素的个数，也不会自动改变容器的大小

#### 介绍back_inserter
- **插入迭代器**：是一种向容器添加元素的迭代器。
- 使用**插入迭代器**：保证调用写入算法有足够的空间来容纳输出数据。
- **插入迭代器与普通迭代器比较**：通常情况，当我们通过一个迭代器向容器元素赋值时，值被赋予迭代器指向的元素。而当我们通过一个**插入迭代器赋值**时，一个与赋值号右侧值相等的元素被添加到容器。
- 调用`back_inserter` 即可获取到容器的插入迭代器，它定义在头文件iterator中。
	- 接收的参数：一个指向容器的引用。
	- 返回值：一个与该容器绑定的插入**迭代器**。当我们通过此迭代器赋值时，赋值运算符会调用`push_back`将一个具有给定值的元素添加到容器中。

```c++
vector<int> vec; // 空 vector

// it 的类型为 std::back_insert_iterator<std::vector<int>>
auto it = back_inserter(vec); // 通过它赋值会将元素添加到 vec 中
*it = 42; // 42会添加到vec中
```

在算法中，常常使用`back_inserter`来创建一个迭代器，作为目的位置使用：
```c++
vector<int> vec; // 空 vector

// 正确：back_inserter 创建一个插入迭代器，可用来向vec添加元素
fill_n(back_inserter(vec), 10, 0); // 添加 10 个元素到 vec
```
>在每步迭代中，`fill_n`向给定序列的一个元素赋值。由于我们传递的参数是`back-inserter`返回的迭代器，因此每次赋值都会在`vec`上调用`push_back`.最终，这条`fill_n`调用语句向`vec`的末尾添加了10个元素，每个元素的值都是0

#### 拷贝算法
- **拷贝算法**：拷贝算法(`copy`)是一个向目的位置迭代器指向的输出序列的元素写入数据的算法。此算法接受三个迭代器：
	- 前两个参数：表示一个输入范围。
	- 第三个参数：表示目的序列的起始位置。 目的序列至少要包含与输出序列一样多的元素。

- 使用 `copy` 实现内置数组的拷贝：
```c++
int a1[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9};

int a2[sizeof(a1)/sizeof(*a1)]; // a2和a1大小一样

// ret 指向拷贝到 a2 的尾元素之后的位置
auto ret = copy(begin(a1), end(a1), a2); // 把 a1 的内容拷贝给 a2
```
>使用[[CPP#sizeof运算符|sizeof]]确保a2与数组a1包含同样多的元素。a1可以理解为指向数组第一个元素的指针，解引用得到第一个的值

`copy` 返回的是其目的位置迭代器（递增后）的值。即，`ret` 指向拷贝到 `a2` 的尾元素之后的位置。

多个算法都提供所谓的“拷贝"版本。这些算法计算新元素的值，但不会将它们放置在输入序列的末尾，而是创建一个新序列保存这些结果。
例如，比如 `replace` 和 `replace_copy`  （拷贝版本）
这两个函数都是替换序列中元素的值。`replace` 接受 4 个参数： 前两个是迭代器，表示输出序列，后两个一个是要被替换的值，另一个是替换后的新值。
```c++
// 将ilst 中所有值为 0 的元素改为 42
replace(ils.begin(), ilst.end(), 0, 42);
```
**`replace` 是在原始序列上改动，而 `replace_copy` 是将改动后的序列拷贝一份，原序列保持不变。** 此算法接受额外第三个迭代器参数，指出调整后序列的保存位置。
```c++
// 使用 back_inserter 按需要增长目标序列
replace_copy(ilst.cbegin(), ils.cend(),back_inserter(ivec), 0, 42);
```
此调用，`ilst` 并未改变，`ivec` 包含 `ilst` 的一份拷贝，不过原来 `ilst` 中的值为 0 的元素在 `ivec` 中都变为 42


### 10.2.3 重排容器元素的算法
**排序算法**：这类算法是利用元素类型的 `<` 运算符来实现排序的。  
  
下面以一段文本为例，这段文本保存在 `vector` 中，我们希望简化这个 `vector` ，使得每个单词只出现一次。
```Plain Text
the quick red fox jumps over the slow red turtle
```
去重排序后的 `vector`：
```Plain Text
fox jumps over quick red slow the turtle
```

#### 消除重复单词
为了消除重复单词，首先将`vector`排序，使得重复的单词都相邻出现。一旦`vector`排序完毕，我们就可以使用另一个称为`unique`的标准库算法来重排`vector`，使得不重复的元素出现在`vector`的开始部分。
>由于算法不能执行容器的操作，我们将使用`vector`的`erase`成员来完成真正的删除操作：
```c++
void eliDups(vector<string> &words)
{
	// 按字典排序 words ，以便查找重复单词
	sort(words.begin(), words.end());
	// unique 重排输入范围，使得每个单词只出现一次
	// 排序在范围的前部，返回指向不重复区域之后一个位置的迭代器
	auto end_unique = unique(words.begin(), words.end());
	// 使用容器操作 erase 删除重复元素
	words.erase(end_unique, words.end());
}
```

- `sort`算法：接受两个迭代器，表示要排序的元素范围。
完成sort之后，words顺序如下：
```Plain Text
fox jumps over quick red red slow the the turtl
```

#### 使用unique
- `unique`算法：将相邻的重复项“消除”，并返回一个指向不重复值范围末尾的迭代器。
调用 `unique` 后 `vector` 将变为：
![[Pasted image 20230127135712.png]]

`words` 的大小并没有改变，它仍有10个元素。`unique`并不真的删除任何元素，它只是覆盖相邻的重复元素，使得不重复元素出现在序列开始部分。`unique` 返回的迭代器指向最后一个不重复元素之后的位置。**此位置之后的元素依然存在，但我们不知道它们的值是什么。**

>[!note]
>标准库算法对迭代器而不是容器进行操。因此，算法不能（直接）添加或删除元素。

#### 使用容器操作删除元素
为了删除无用的元素，我们必须使用容器操作。所有最后使用了 `erase` 删除重复的元素。我们删除从`end_unique`开始直至`words`末尾的范围内的所有元素。


## 10.3 定制操作
很多算法都会比较输入序列中的元素。默认情况下，这类算法使用元素类型的 `==` 或 `<` 运算符来完成比较。标准库还为这些算法定义了额外的版本，允**许我们提供自定义的操作来代替默认运算符。**
例如，`sort`算法默认使用元素类型的`<`运算符。但可能我们希望的排序顺序与`<`所定
义的顺序不同，或是我们的序列可能保存的是未定义`<`运算符的元素类型（如`sales_data`)在这两种情况下，都需要**重载**`sort`的默认行为。

### 10.3.1 向算法传递函数
对于 `elimDups` ([[#10.2.3 重排容器元素的算法]])还希望单词按其长度排序，大小相同的再按字典序排列。为了实现按长度重排 `vector` ，我们可以使用 `sort` 的第二个版本，它接受第三个参数，此参数是一个 **谓词**。

#### 谓词
- **谓词**：谓词是一个可调用的表达式，其返回结果是一个能用作条件的值。可调用表达式是指以 `express(args)` 这种方式调用，这里的 "可调用的表达式"可以简单的理解为就是函数指针（目前我们只知道函数和函数指针这两种可调用对象，后面章节还会介绍其他可调用对象）。标准库算法的谓词分为两类：
	- **一元谓词**：意味着它们只接受单一参数。 ^46f254
	- **二元谓词**：意味着它们有两个参数
- **接受谓词参数的算法对输入序列中的元素调用谓词。** 因此，元素类型必须能转换为谓词的参数类型。

- 接受一个二元谓词参数的 `sort` 版本用这个谓词来代替 `<` 来比较元素。
```c++
// 比较函数，用来按长度排序单词， 作为一个二元谓词传递给 sort
bool isShorter(const string& s1, const string& s2)
{
	return s1.size() < s2.size();
}

// 按长度有短至长排序 words
sort(words.begin(), words.end(), isShorter);
```

#### 排序算法
在我们将 `words` 按大小重排的同时，还希望具有相同长度的元素按字典序排序。为了保持相同长度的单词按字典序排序，可以使用 `stable_sort` 算法。这种稳定排序算法维持相等的元素原有顺序。
```c++
elimDups(words); // 将words按字典序重排，并消除重复单词

// 按长度重新排序，长度相同的单词维持字典序
stable_sort(words.begin(), words.end(), isShorter);

for (const auto &s : words)
	cout << s << " ";

cout << endl;
```
假定在此调用前`words`是按字典序排列的，则调用之后，`words`会按元素大小排序，而
长度相同的单词会保持字典序。如果我们对原来的`vector`内容运行这段代码，输出为：
```Plain Text
fox red the over slow jumps quick turtle
```


### 10.3.2 lambda表达式
根据算法接受一元谓词还是二元谓词，我们传递给算法的谓词必须严格接受一个或两个参数。但是，有时我们希望进行的操作需要更多参数，超出了算法对谓词的限制。
一个例子，我们修改 [[#10.3.1 向算法传递函数]] 节的程序，求大于等于一个给定长度的单词有多少，并将这些单词打印出来。
```c++
void biggies(vector<string>& words,
vector<string>::size_type sz)
{
	elimDups(words); // 将words按字典序重排，并消除重复单词
	// 按长度重新排序，长度相同的单词维持字典序
	stable_sort(words.begin(), words.end(), isShorter);
	// 1. 获取一个迭代器，指向第一个满足 size() >= sz 的元素
	// 2. 计算满足 size() >= sz 的元素的数目
	// 3. 打印
}
```
- 在步骤1，我们可以使用标准库 `find_if` 算法来查找第一个具有特定大小的元素。- 
	- `find_if` 算法接受一对迭代器，表示一个范围，第三个参数是一个谓词。`find_if` 算法对输入序列中的每个元素调用给定的这个谓词。它返回第一个使谓词返回非0值的元素，如果不存在这样的元素，则返回尾迭代器。
编写一个函数，令其接受一个 `string` 和一个长度，并返回一个 `bool` 值表示该 `string` 的长度是否大于给定长度。但是，`find_if` 接受一元谓词，不能传递一个二元谓词，为了解决此问题，需要使用 **`lambda`表达式。**

#### 介绍lambda
我们可以向一个算法传递任何类别的**可调用对象**。
- **可调用对象：** 对于一个对象或一个表达式，如果可以对其使用调用运算符`()` ，则称它为可调用对象。即，如果 `e` 是一个可调用表达式，则我们可以编写代码 `e(args)`，其中 `args` 是一个逗号分隔得一个或多个参数得列表。
- 可调用对象有：
	- 函数  
	- 函数指针  
	- 重载了函数调用运算符的类  
	- **lambda 表达式**。

C++11 一个 `lambda` 表达式表示一个可调用的代码单元。**我们可以将其理解为一个未命名的内联函数。** 与任何函数类似，一个 `lambda` 具有返回类型、参数列表和函数体。但与函数不同，`lambda` 可能定义在函数内部。
>与python中的lambda类似

- `lambda`表达式形式：
 ```c++
 [capture list] (parameter list) -> return type { function body }
 ```
 - `capture list` : 捕获列表是一个 `lambda` 所在函数中定义的局部变量。
 - `return type`、`parameter list` 和 `function body` 与普通函数一样，分别表示**返回类型**、**参数列表**和**函数体**。但是，与普通函数不同，`lambda` 必须[[CPP#使用尾置返回类型|使用尾置返回类型]]。
 - 可以忽略参数列表和返回类型，但**必须永远包含捕获列表和函数体**
```c++
// 可以忽略参数列表和返回类型，但必须永远包含捕获列表和函数体
auto f = [] { return 42; }; // 定义一个可调用对象 f,忽略了括号和空参数列表，所以该函数无需参数
cout << f() << endl; // 调用，打印42

```
在 `lambda` 中忽略括号和参数列表等价于指定一个空参数列表。
如果忽略返回类型，`lambda` 根据函数体中的代码推断出返回类型。如果函数体**只是**一个 `return` 语句，则返回类型从返回的表达式的类型推断而来。**否则返回类型为 void**。
>[!note]
>如果`lambda`的函数体**包含任何单一`return`语句之外**的内容，且**未指定返回类型**，则返回`void.`

#### 向lambda传递参数
与普通函数调用类似，调用一个 `lambda` 是给定的实参被用来初始化 `lambda` 的形参。通常，实参和形参的类型必须匹配。但与普通函数不同，lambda不能有[[CPP#默认实参|默认参数]]。因此，一个lambda调用的实参永远与形参数目相等。

一个带参数的lambda的例子：
```c++
[] (const string &a, const string &b)

{ return a.size() < b.size(); }
```
空捕获列表表明此`lambda`不使用它所在函数中的任何局部变量。

#### 使用捕获列表
一个`lambda`通过将局部变量包含在其捕获列表中来指出将会使用这些变量。捕获列表指引`lambda`在其内部包含访问局部变量所需的信息。
示例，`lambda` 捕获 `sz` 并只有单一的 `string` 参数.
```c++
[sz] (const string &a)
{ return a.size() >= sz; }
```
>[!note]
>一个`lambda`只有在其捕获列表中捕获一个它所在函数中的局部变量，才能在函数体中使用该变量。

#### 调用find_if
使用此 `lambda`，我们就可以查找第一个长度大于等于 sz 的元素：
```c++
// 获取一个迭代器，指向第一个满足 size() >= sz 的元素
auto wc = find_if(words.begin(), words.end(),
[sz] (const string &a)
{ return a.size() >= sz; });
```
这里对`find_if`的调用返回一个迭代器，指向第一个长度不小于给定参数`sz`的元素。如果这样的元素不存在，则返回`words.end()`的一个拷贝。

#### for_each算法
问题的最后一部分是打印 `words` 中长度大于等于 `sz` 的元素。
```c++
// 打印长度大于等于给定值的单词，每个单词后面接一个空格
for_each(wc, words.end(),
[] (const string &s) { cout << s << " ";});
cout << endl;
```
>[!note]
>捕获列表**只用于局部非`static`变量**，`lambda`可以直接使用局部`static`变量和在它所在函数之外声明的名字。比例，上面代码的`lambda`直接使用了`cout`，该函数之外包含了`iostream`头文件，即可使用。

#### 完整的biggies
```c++
void biggies(vector<string> &words,
vector<string>::size_type sz)
{
	elimDups(words); // 将 words按字典序排序，删除重复单词
	// 按长度排序，长度相同的单词维持字典序，第三个参数是lambda表达式
	stable_sort(words.begin(), words.end(),
	[] (const string &a, const string &b)
	{ return a.size() < b.size();} );
	// 获取一个迭代器，指向第一个满足size() >= sz 的元素
	auto wc = find_if(words.begin(), words.end(),
	[sz] (const string &a)
	{ return a.size() >= sz;} );
	// 计算满足 size >= sz 的元素的数目
	auto count = words.end() - wc;
	cout << count << " " << make_plural(count, "words", "s")
	<< " of length " << sz << " or longer" << endl;
	// 打印长度大于等于给定值的单词，每个单词后面接一个空格
	for_each(wc, words.end(),
	[] (const string &s) {cout << s << " ";});
	cout << endl;
	}
```


### 10.3.3 lambda捕获和返回
当定义一个 `lambda`时，编译器生成一个与`lambda`对应的新的（未命名的）类类型。在14.8.1将介绍这种类时如何生成的。目前，可以这样理解。当向一个函数传递一个`lambda`时，同时定义一个新类型和该类型的一个对象：传递的参数就是此编译器生成的类类型的未命名对象。

当使用`auto`定义一个用`lambda`初始化的变量时，定义了一个从生成的类型的对象。
默认情况下，从`lambda`生成的类都包含一个对应该`lambda`所捕获的变量的数据成员。
类似任何普通类的数据成员，`lambda`的数据成员也在`lambda`对象创建时被初始化。

#### 值捕获
`lambda`捕获方式和参数传递类似，可以是值或引用，下表列出了几种不同的捕获列表方式
![[Pasted image 20230127151649.png]]

- 采用**值捕获的前提是变量可以拷贝。** 与传值参数不同的是，被捕获的变量的值是在`lambda`**创建时拷贝**，而不是调用时拷贝：
```c++
void fcn1()
{
	size_t v1 = 42; // 局部变量
	// 将 v1 拷贝到名为f的可调用对象
	auto f = [v1] { return v1; }
	v1 = 0;
	auto j = f(); // j 为42；f将保存了我们创建它时v1的拷贝
}
```
由于被捕获变量的值是在`lambda`创建时拷贝，因此随后对其修改不会影响到`lambda`内对应的值。

#### 引用捕获
我们定义`lambda` 时可以采用**引用方式捕获**变量。
```c++
void fcn2()
{
	size_t v1 = 42;
	auto f2 = [&v1] {return v1;};
	v1 = 0;
	auto j = f2(); // j 为0； f2保存v1的引用，而非拷贝
}
```
当我们在`lambda`函数体内使用此变量时，实际上使用的是引用所绑定的对象。
>引用捕获和返回引用有着相同的问题和限制。在调用`lambda`表达式和返回引用的时候，我们要保证捕获的引用变量是存在的，我没有被销毁。

引用捕获有时是必要的，有些对象是不允许拷贝的，比如输入和输出流.
```c++
void biggies(vector<string> &words,
vector<string>::size_type sz,
ostream &os = cout , char c = ' ')
{
	// 与之前例子一样的重排words的代码
	// 打印 count 的语句改为打印到 os
	for_each(words.begin(), words.end(),
	[&os, c] (const string &s) {os << s << c;});
}
```

>[!warning]
>当以引用方式捕获一个变量时，必须保证在`lambda`执行时是存在的。

#### 隐式捕获
除了显式的列出我们希望使用来自所在函数的变量之外，还可以让编译器根据 `lambda` 体中的代码来推断我们要使用哪些变量。
为了指示编译器推断捕获列表，应在捕获列表中写一个 `&` 或 `=` 。
- `&` 告诉编译器采用捕获引用方式，
- `=` 则表示采用值捕获方式。
```c++
// sz 为隐式捕获，值捕获方式
wc = find_if(words.begin(), words.end(),
[=] (const string &s)
{ return s.size() >= sz; });
```

如果我们希望对**一部分变量采用值捕获**，对**其他变量采用引用捕获**，可以混合使用隐
式捕获和显式捕获：
```c++
void biggies(vector<string> &words,
vector<string>::size_type sz,
ostream &os = cout, char c = ' ')
{
	// 其他处理与前例一样
	// os 隐式捕获，引用捕获方式； c显式捕获，值捕获方式
	for_each(words.begin(), words.end(),
	[&, c](const string &s) { os << s << c; });
	// os 显式捕获，引用捕获方式； c 隐式捕获，值捕获方式
	for_each(words.begin(), words.end(),
	[=, &os](const string &s) {os << s << c;});
}
```
**当我们混合使用隐式捕获和显式捕获时，捕获列表中的第一个元素必须是一个`&`或`=`。** 此符号指定了**默认**捕获方式为引用或值。

当混合使用隐式捕获和显式捕获时，**显式捕获的变量必须使用与隐式捕获不同的方式**。即，
- 如果隐式捕获是引用方式（使用了`&`），则显式捕获命名变量必须采用值方式，此不能在其名字前使用`&`
- 如果隐式捕获采用的是值方式（使用了`=`），则显式获命名变量必须采用引用方式，即，在名字前使用`&`。

#### 可变lambda
默认情况下，值拷贝的变量，`lambda`不会改变其值，如果需要改变一个被捕获变量的值，就必须在参数列表首加上关键字 `mutable`。[[CPP#^2fec59|mutable在类内的应用]]
```c++
void fcn3()
{
	size_t v1 = 42; // 局部变量
	// f 可以改变它所捕获的变量的值
	auto f = [v1] ()mutable {return ++v1;};
	v1 = 0;
	auto j = f(); // j 为 43
}
```

对于引用方式捕获的变量，是否可修改依赖于引用是否为 `const`
```c++
void fcn4()
{
	size_t v1 = 42; // 局部变量
	// v1 是一个非 const 变量的引用
	// 可以通过f2中的引用来改变它
	auto f2 = [&v1] {return ++v1;};
	v1 = 0;
	auto j = f2(); // j 为1
}
```

#### 指定lambda返回类型
到目前为止，我们所编写的`lambda`都只包含单一的 `return` 语句，这种形式的 `lambda`表达式无须指定返回类型，编译器会自动推断返回类型。
如果一个`lambda`表达式包含 `return` 之外的任何语句，则编译器假定此 `lambda` 表达式返回 `void` .
```c++
//将一个序列中的每个负数替换为其绝对值：
transform(vi.begin(), vi.end(), vi.begin(),
[](int i) { return i < 0 ? -i : i; });
```

如果就上面改成 `if` 语句，就会产生编译错误
```c++
transform(vi.begin(), vi.end(), vi.begin(),
		  [](int i) {if (i < 0) return -1; else return i; });
```
编译器推断的返回类型是 `void` , 但它返回了一个 `int` 值，正确的做法是指定返回类型，而且必须使用**尾置返回类型**。
```c++
transform(vi.begin(), vi.end(), vi.begin(),
[](int i) -> int
{if (i < 0) return -1; else return i; });
```


## 10.3.4 参数绑定
在使用一个接受一元谓词的算法 `find_if` , 我们通过给算法传递`lambda`表达式实现调用外部的变量，除了`lambda`表达式的方法，还可以使用**参数绑定**。

#### 标准库bind函数
```c++
bool check_size(const string &s, string::size_type sz)
{
	return s.size() >= sz;
}
```
该函数不能作为`find_if`的一个参数，因为`find_if`只接受一个[[#^46f254|一元谓词]]
使用`bind`函数可以解决该问题。

`bind`函数：它定义在头文件functional中。可以将`bind`函数看作一个通用的[[CPP#容器适配器|函数适配器]]，它接受一个可调用对象，生成一个新的可调用对象来适应原对象的参数列表。

调用 `bind` 的一般形式为：
```c++
 auto newCallable = bind(callable, arg_list);
```
- `newCallable` 本身是一个可调用对象，
- `arg_list` 是一个逗号分隔的参数列表，对应给定的`callable` 的参数。  
- **当我们调用`newCallable`时，`newCallable`会调用`callable`，并传递给它 `arg_list` 中的参数。**
- `arg_list` 中的参数可能包含形如 `_n` 的名字， 其中 `n` 是一个整数。这些参数是 “占位符”，表示 `newCallable` 的参数，它们占据了传递给`newCallable`的参数的“位置”。数值 `n` 表示生成的可调用对象中参数的位置：`_1` 为 `newCallable` 的第一个参数，`_2`为第二个参数，依此类推。

#### 绑定check_size的sz参数
作为一个简单的例子，我们将使用`bind`生成一个调用`check_size`的对象，如下所示，它用一个定值作为其大小参数来调用`check_size`：
```c++
auto check6 = bind(check_size, _1, 6);
//调用
string s = "hello";
bool b1 = check6(s); // check6(s) 会调用check_size(s,6)
// 使用bind,我们可以将原来基于 lambda 的 find_if 调用：
auto wc = find_if(words.begin(), words.end(),
[=] (const string &s) { return s.size() >= sw; });
//替换为：
auto wc = find_if(words.begin(), words.end(),
bind(check_size, _1, sz)));
```
- 此`bind`调用**只有一个占位符**，**表示`check6`只接受单一参数**。
- 占位符出现在的`arg_list`的第一个位置，表示`check6`的此参数对应`check_size`的第一个参数。此参数是一个`const string&`。因此，调用`check6`必须传递给它一个`string`类型的参数，`check6`会将此参数传递给`check_size`。
![[Pasted image 20230127163227.png]]

#### 使用placeholders名字
名字 `_1` 都定义在一个名为 `placeholders` 的命名空间中，而这个命名空间本身定义在 `std` 命名空间中。为了使用这些名字，我们可以使用以下声明：
```c++
using std::placeholders::_1; // 只使用 _1
// 或者
using namespace std::placeholders; //使用全部的参数
```

这种形式说明希望所有来自`namespace_name`的名字都可以在我们的程序中直接使用。
```c++
using namespace namespace_name;
```

placeholders命名空间也定义在functional头文件中。

#### bind的参数
如前文所述，我们可以用`bind`修正参数的值。更一般的，可以用`bind`绑定给定可调用对象中的参数或重新安排其顺序。
例如，假定`f`是一个可调用对象，它有5个参数:
```c++
// g 是一个有两个参数的可调用对象
auto g = bind(func, a, b, _2, c, _1);
```
生成一个新的可调用对象，它有两个参数，分别用占位符`_2`和`_1`表示。这个新的可调用
对象将它自己的参数作为第三个和第五个参数传递给`func`。`fu'n'cunc`的第一个、第二个和第四个参
数分别被绑定到给定的值`a`、`b`和`c`上。
```c++
g(e,f);
// 实际调用：
f(a, b, f, c, e);
```
>[!note]
>实际上调用`bind()`返回的函数实际上是接受参数，然后接受的参数和bind中的参数都传递给`bind`参数中的实际调用函数。

#### 用bind重排参数顺序
下面是用 `bind` 重排参数顺序的一个具体例子，我们可以用`bind`颠倒 `isShroter` 的含义。
```c++
// 按单词长度有短至长排序
sort(words.begin(), words.end(), isShorter);

// 按单词长度有长至短排序
sort(words.begin(), words.end(), bind(isShorter, _2, _1));
```
第一个调用中`isShorter(A,B)`，调用传递给`isShorter`的参数被交换过来了。因此，当`sort`比较两个元素时，就好像调用`isShorter(B,A)`—样。

#### 绑定引用参数
默认情况下，`bind`的那些不是占位符的参数(例如[[#bind的参数]]中的a、b、c)被拷贝到`bind`返回的可调用对象中。
但是，与`lambda`类似，有时对有些绑定的参数我们希望以引用方式传递，或是要绑定参数的类型无法拷贝。
例如，为了替换一个引用方式捕获`ostream`的`lambda`：
```c++
// os 是一个局部变量，引用一个输出流
// c 是一个局部变量，类型为char
for_each(words.begin(), words.begin(),
[&os, c](const string &s){os << s << c;});
//
ostream &print(ostream &os, const string &s, char c)
{
	return os << s << c;
}

// 错误：不能拷贝 os
for_each(words.begin(), words.begin(),
bind(print, os, _1, ' '));

// 使用标准库ref函数，传递引用
for_each(words.begin(), words.begin(),
bind(print, ref(os), _1, ' '));
```
`bind`拷贝其参数，而我们不能拷贝一个`ostream`。如果我们希望传递给`bind`一个对象而又不拷贝它，就必须使用标准库 **`ref`函数**：
函数 `ref` 返回一个对象，包含给定的引用，此对象是可以拷贝的。标准库中还有一个 `cref` 函数，生成一个保存 `const` 引用的类。它们都定义在头文件 functional 中。


## 10.4 再探迭代器
除了为每个容器定义的迭代器之外，标准库在头文件`iterato`r中还定义了额外几中迭代器：
- **插入迭代器**：这些迭代器被绑定到一个容器上，可用来向容器插入元素。
- **反向迭代器**：这些迭代器向后而不是向前移动。除了`forward_list`之外的标准库容器都有反向迭代器。
- **移动迭代器**：这些专用的迭代器不是拷贝其中的元素，而是移动它们。

### 10.4.1 插入迭代器
**插入迭代器**是一种迭代器适配器，它接受一个容器，生成一个迭代器，能实现向给定容器添加元素。
![[Pasted image 20230127165405.png]]

- [[#介绍back_inserter||back_insert]]：创建一个使用`push_back`的迭代器。
- `front_insert`：创建一个使用`push_front`的迭代器。
- `inserter`：创建一个使用`insert`的迭代器。此函数接受第二个参数，这个参数必须是一个指向给定容器的迭代器。元素将被插入到给定迭代器所表示的元素**之前**。
>[!note]
>- 只有在容器支持`push_front`的情况下，我们才可以使用`front_inserter`
>- 只有在容器支持`push_back`的情况下，我们才能使用`back_inserter`

以`inserter`为例，
当调用`inserter(c,iter)`时，我们得到一个迭代器，接下来使用它时，会将元素**插入到iter原来所指向的元素之前的位置**。即如果it是由inserter生成的迭代器，则：
```c++
*it = val;

//等价于
it = c.insert(it,val);//it指向新加入的元素
++it;//递增it使它指向原来的元素
```

以`front_inserter`为例，
使用`front_inserter`时，元素总是插入到容器**第一个元素之前**。只要我们在此元素之前插入一个新元素，此元素就不再是容器的首元素了：
```c++
list<int> lst = {1,2,3,4};
list<int> lst2,lst3;//空list

//拷贝完成之后，lst2包含4，3，2，1
copy(lst.cbegin(),lst.cend(),front_inserter(lst2));
//拷贝完成后，lst3包含1，2，3，4
copy(lst.cbegin(),lst.cend(),inserter(lst3,lst3.begin()))
```
- 使用`front_inserter`：每次插入都是在第一个元素之前，每次调用之后首元素都会改变，返回的是指向首元素的迭代器。
- 使用`inserter`：每次插入都是在指定位置之前，返回的是指向插入之前位置的迭代器。

### 10.4.2 iostream迭代器
虽然`iostream`类型不是容器，但标准库定义了可以用于这些IO类型对象的迭代器。
- `istream_iterator`：读取输入流。
- `ostream_iterator`：向一个输出流写数据。
这些迭代器将它们对应的**流当作一个特定类型的元素序列**来处理。

#### istream_iterator操作
当创建一个流迭代器时，必须指定迭代器将要读写的对象类型。
由于`istream_iterator`使`>>`来读取流，因此`istream_iterator`迭代器的类型必须定义例如输入运算符。
```c++
istream_iterator<int> int_it(cin)//从cin读取int
istream_iterator<int> int_eof;//默认初始化迭代器，为空，可以看成创建了一个尾后迭代器。
ifstream in("afile");
istream_iterator<string> strt(in)；//从"afile"读取字符串
```

下面是一个用`istream_iterator`从标准输入读取数据，存入一个`vector`的例子：
```c++
istream_iterator<int> in_iter(cin);//从cin读取int
istream_iterator<int> eof;//istream尾后迭代器
while(in_iter!=eof)//当有数据可供读取时
	//后置递增运算读取流，返回迭代器的旧值
	//解引用迭代器，获得从流读取的前一个值
	vec.push_back(*in_iter++);
```
后置递增运算会从流中读取下一个值，向前推进，但返回的是迭代器的旧值。迭代器的旧值包含了从流中读取的前一个值，对迭代器进行解引用就能获得此值。
```c++
//等价于
istream_iterator<int> initer(cin),eof;//从cin读取int
//从迭代器范围构造vec
vector<int> vec(initer,eof);
```
![[Pasted image 20230127173500.png]]

#### 使用算法操作流迭代器
由于**泛型算法使用迭代器操作来处理数据**，而流迭代器又至少支持某些迭代器操作，因此我们至少可以用某些算法来操作流迭代器。
使用一对`istream_iterator`来调用[[#accumulate算法]]
```c++
istream_iterator<int> in(cin),eof;
cout<<accumulate(in,eof,0)<<endl;
```
![[Pasted image 20230127173749.png]]

#### istream_iterator允许使用懒惰求值
- `istream_iterator`的惰性求值：当我们将一个`istream_iterator`绑定到一个流时，标准库并不保证迭代器立即从流读取数据。具体实现可以推迟从流中读取数据，**直到我们使用迭代器时才真正读取**。

- 用途：
	- 创建了一个`istream_iterator`，没有使用就销毁
	- 从两个不同的对象同步读取同一个流


#### ostream_iterator操作
我们可以对任何具有输出运算符（`<<`运算符）的类型定义`ostream_iterator`。
创建一个`ostream_iterator`时，我们可以提供 **（可选的）第二参数，它是一个字符串，在输出每个元素后都会打印此字符串。** 此字符串必须是一个[[CPP#C风格字符串]]（即，一个字符串字面常量或者一个指向以空字符结尾的字符数组的指针）。

与`istream_iterator`不同的是在创建时必须绑定一个指定的流，不允许空的或表示尾后位置的`ostream_iterator`
![[Pasted image 20230127174315.png]]

使用ostream_iterator输出值的序列：
```c++
ostrea_miterator<int> out_iter(cout,"");
for(auto e:vec)
	*out_iter++=e；//赋值语句实际上将元素写到cout
cout<<endl;
```
每次向`out_iter`赋值时，写操作就会被提交。当我们向`out_iter`赋值时，可以忽略解引用和递增运算。即，循环可以重写成下面的样子：
```c++
for(auto e:vec)
	out_iter=e，//赋值语句将元素写到cout
cout<<endl;
```
运算符`*`和`++`实际上对`ostream_iterator`对象不做任何事情，因此忽略它们对我们的
程序没有任何影响。但是，推荐第一种形式。在这种写法中，流迭代器的使用与其他迭代器的使用保持一致。

使用`copy`打印`vec`中的元素：
```c++
copy(vec.begin(),vec.end(),out_iter)
cout<<endl;
//out_iter是ostream_iterator绑定了cout。
```

#### 使用流迭代器处理类类型
- 可以为任何定义了输入运算符（`>>`）的类型创建`istream_iterator`对象。
- 可以为任何定义了输出运算符（`<<`）的类型创建`ostream_iterator`对象。
![[Pasted image 20230128133253.png]]

### 反向迭代器
**反向迭代器**：反向迭代器就是在容器中从尾元素向首元素反向移动的迭代器。
对于反向迭代器，递增（以及递减）操作的含义会颠倒过来。递增一个反向迭代器（`++it`）会移动到前一个。

**除了`forward_list`之外，其他容器都支持反向迭代器。**

- 调用反向迭代器：`rbegin`、`rend`、`crbegin`、`crend`
![[Pasted image 20230128133729.png]]
```c++
vector<int> vec = {0,1,2,3};
for(auto riter = vec.crbegin();riter!=vec.crend();++iter){
	cout<<*riter<<endl;//打印3，2，1，0
	}

//利用sort排序为递减序列
sort(vec.begin(),vec.end());
sort(vec.rbegin(),vec.rend());//逆序：从大到小
```

#### 不适用反向迭代器的情况
除了`forward_list`之外，标准容器上的其他迭代器都既支持递增运算又支持递减运算。
但是，**流迭代器不支持递减运算**，因为不可能在一个流中反向移动。
因此，**不可能**从一个`forward_list`或一个**流迭代器**创建反向迭代器。

#### 反向迭代器和其他迭代器间的关系
**反向迭代器可以转化为普通迭代器**，使用调用`reverse_iterator`的`base`成员进行转换。
![[Pasted image 20230128135745.png]]
- **反向迭代器`rcomma`** 通过调用`base()`方法之后转变为普通迭代器，其递增递减的方向发生改变，与普通迭代器一致。
- `rcomma`和`rcomma.base()`指向不同元素，`line.crbegin`和`line.cend`也是如此。这些不同保证了元素范围无论是正向处理还是方向处理都是相同的。
- 从技术上讲，普通迭代器与反向迭代器的关系反映了**左闭合区间**的特性。键点在于$[line.crbegin(),rcomma)$和$[rcomma.base(),line.cend())$指向`line`中相同元素范围。都是单词`LAST`。
>[!note]
>反向迭代器的目的是表示元素范围，而这些范围是不对称的，这导致一个重要的结果：
>当我们从一个普通迭代器初始化一个反向迭代器，或是给一个反向迭代器赋值时，结果迭代器与原迭代器指向的并不是相同的元素。


## 10.5 泛型算法结构
C++ STL 提供的泛型算法不是直接操作容器的，而是通过迭代器操作数据的，这些算法所需要的迭代器操作可以分为5个迭代器类别：
![[Pasted image 20230128140453.png]]
>每个算法都会对它的每个迭代器参数指明须提供哪类迭代器。

### 10.5.1 5类迭代器
- 类似容器，迭代器也定义了一组公共操作。一些操作所有迭代器都支持，另外一些只有特定类别的迭代器才支持。
- 迭代器是按他们所提供的操作来分类的，而这种分类形成了一种层次。**除了输出迭代器之外，一个高层次类别的迭代器支持低层类别迭代器的所有操作(向下兼容)**。
- C++ 标准指明了泛型和数值算法的每个迭代器参数的最小类别。例如， `find` 算法在一个序列上进行一遍扫描，对元素进行只读操作，因此至少需要输入迭代器。
>对于向一个算法传递错误类别的迭代器的问题，很多编译器不会给出任何警告或提示。

#### 迭代器类别
- **输入迭代器**：可以读取序列中的元素。**只用于顺序访问**。一个输入迭代器必须支持
	- 用于比较两个迭代器的相等和不相等运算符 (`==、!=`)  
	- 用于推进迭代器的前置和后置递增运算 (`++`)  
	- 用于读取元素的解引用运算符（`*`）；解引用只会出现在赋值运算符的右侧  
	- 箭头运算符（`->`），等价于 `(*it).member`，即，解引用迭代器，并提取对象的成员。
>算法`find`和`accumulate`要求输入迭代器：而`istream_iterator`是一种输入迭代器。

- **输出迭代器**：可以看作输入迭代器功能上的补集——**只写而不读元素**。输入迭代器必须支持
	- 用于推进迭代器的前置和后置递增运算 （`++`）  
	- 解引用运算符（`*`），只出现在赋值运算符的左侧（向一个已经解引用的输出迭代出赋值，就是将值写入它所指向的元素）
>只能向一个输出迭代器赋值一次。类似输入迭代器，输出迭代器只能用于单遍扫描算法。用作目的位置的迭代器通常都是输出迭代器。`copy`函数的第三个参数就是输出迭代器，`ostream_iterator`类型也是输出迭代器。

- **前向迭代器**：可以读写元素。这类迭代器只能在序列中沿一个方向移动。
>支持所有输入和输出迭代器的操作，而且可以多次读写同一个元素。算法`replace`要求前向迭代器，`forward_list`上的迭代器是前向迭代器。

- **双向迭代器**：可以正向/反向读写序列中的元素。
>支持所有前向迭代器的操作，还支持前置和后置递减运算符（`--`）。`reverse`要求双向迭代器，除`forward_list`之外，其他标准库都提供符合双向迭代器要求的迭代器。

- **随机访问迭代器**：提供在常量时间内访问序列中任意元素的能力。此外还支持以下操作：
	- 用于比较两个迭代器相对位置的关系运算符 （`<`、`<=`、`>` 和 `>=` ）  
	- 迭代器和一个整数值的加减运算（ `+` 、`+=` 、`-` 和 `-=`），计算结果是迭代器在序列中前进（或后退）给定整数个元素后的位置  
	- 用于两个迭代器上的减法运算符（`-`），得到两个迭代器的距离  
	- 下标运算符（`iter[n]`）于 `*(iter[n])` 等价。
![[Pasted image 20230128141933.png]]
>随机访问迭代器支持上表所有操作。算法`sort`要求随机访问迭代器。`array`、`deque`、`string`和`vector`的迭代器都是随机访问迭代器，用于访问内置数组元素的指针也是。


### 10.5.2 算法形参模式
大多数算法具有如下4中形式之一：
- `alg(beg, end, other args);`  
- `alg(beg, end, dest, other args);`  
- `alg(beg, end, beg2, other args);`  
- `alg(beg, end, beg2, end2, other args);`
>`alg`是算法的名字，`beg` 和 `end` 表示算法所操作的**输入范围**。几乎所有算法都接受一个输入范围，是否有其他参数依赖于要执行的操作。这里列出了常见的一种——`dest` 、`beg2` 和 `end2`，都是迭代器参数，分别指定目的位置和第二个范围。

#### 接受单个目标迭代器的算法
`dest` 参数是一个表示算法可以写入的目的位置迭代器。
**算法假定**：按其需要写入数据，不管写入多少个元素都是安全的。  
>**向输入迭代器写入数据的算法都假定目标空间足够容纳写入的数据**

#### 接受第二个输入序列的算法
接受单独的 `beg2` 或者接受`beg2` 和 `end2` 的算法用这些迭代器表示第二个输入范围。
- 如果一个算法接受`beg2`和`end2`，这两个迭代器表示第二个范围。
- 只接受单独的`beg2`（不接受`end2`）的算法将`beg2`作为第二个输入范围中的元素。此范围的结束位置未指定，这些算法假定从`beg2`开始的范围与`beg`和`end`所表示的范围**至少一样大。**

### 10.5.3 算法命名规范
除了参数规范，算法还遵循一套命名和重载规范。

#### 一些算法使用重载形式传递一个谓词
**接受谓词参数来代替 `<=` 或 `==` 运算符**的算法，以及那些不接受额外参数的算法，通常都是重载的函数。  
例如:
```c++
unique(beg, end); // 使用 == 运算符比较元素
unique(beg, end, comp); // 使用 comp 比较元素
```
两个调用都重新整理给定序列，将相邻的重复元素删除。

#### _ if 版本的算法
接受一个元素值的算法通常有另一个不同名的（**不是重载**的）版本，该版本接受一个谓词代替元素值。接受谓词参数的算法都有附加的 `_if` 前缀：
```c++
find(beg, end, val); // 查找输入范围中 val第一次出现的位置
find_if(beg, end, pred); // 查找第一个令 pred 为真的元素
```
这两个算法提供了命名上差异的版本，而非[[CPP#函数重载|重载]]，因为两个版本的算法都**接受相同数目的参数。**

#### 区分拷贝元素的版本和不拷贝的版本
默认情况下，重排元素的算法将重排后的元素写入给定的输入序列中。这些算法还提供另一个版本，将**元素写到一个指定的输出目的位置**。写到额外目的空间的算法都在名字后面附加一个 `_copy`
```c++
reverse(beg, end); // 将反转输入范围中元素的顺序
reverse_copy(beg, end, dest); // 将元素按逆序拷贝到 dest
```
一些算法同时提供 `_copy` 和 `_if` 版本。这些版本接受一个目的位置迭代器和一个谓词：
```c++
// 从 v1 中删除奇数元素
remove_if(v1.begin(), v1.end(),
[](int i){return i%2; });

// 将偶数元素从v1拷贝到v2; v1 不变
remove_copy_if(v1.begin(), v1.end(), back_inserter(v2),
[](int i) { return i % 2; });
```
>[[#介绍back_inserter]]


## 10.6 特定容器算法
**链表**类型 `list` 和 `forward_list` 定义了几个成员函数形式的算法，如下表
![[Pasted image 20230128144309.png]]
![[Pasted image 20230128144316.png]]
特别是，它们定义了独有的`sort`、`merge`、`remove`、`reverse`和`unique`。
>通用版本的`sort`要求随机访问迭代器，因此不能用于`list`和`forward_list`，因为这两个类型分别提供双向迭代器和前向迭代器。并且随机访问链表开销较大。

>[!note]
>对于`list`和`forward_list`，应该优先使用成员函数版本的算法而不是通用算法

#### splice成员
**链表类型**还定义了 `splice` 算法，此算法是链表数据结构所特有的，因此不需要通用版本
![[Pasted image 20230128144533.png]]
>可能用于链表中移动多个元素？

#### 链表特有的操作会改变容器
- 多数链表特有的算法都与其通用版本很相似，但不完全相同。
- 链表特有版本与通用版本间的一个至关重要的区别是**链表版本会改变底层容器。** 例如， `remove` 的链表版本会删除元素。`unique`的链表版本会删除第二个和后继重复元素，`merge`和`splice`会销毁参数。
>例如，通用版本的`merge`将合并的序列
写到一个给定的目的迭代器：两个输入序列是不变的。而链表版本的`merge`函数会销毁给定的链表——元素从参数指定的链表中删除，被合并到调用`merge`的链表对象中。在之后，来自两个链表中的元素仍然存在，但它们都己在同一个链表中。


# C++标准库：关联容器
关联容器和顺序容器有着根本的不同：关联容器中的元素是**按关键字来保存和访问
的。** 与之相对，顺序容器中的元素是按它们在容器中的位置来顺序保存和访问的。

关联容器**支持高效**的关键字**查找和访问**。两个主要的关联容器：
- `map`：`map`中的元素是一些关键字一值对：
	- 关键字：起索引的作用
	- 值：与索引相关联的数据
- `set`：每个元素只包含一个关键字。支持搞笑的关键字查询操作——检查某关键字是否在`set`中

#### 标准库提供的关联容器
标准库提供8个关联容器：
![[Pasted image 20230128150430.png]]

这8个容器间的**不同**体现在三个维度上：
1. 或者是一个`set`，或者是一个`map`;
2. 或者要求不重复的关键字，或者允许重复关键字：
3. 按顺序保存元素，或无序保存。
	- 允许重复关键字的容器的名字中都包含单词`multi`；
	- 不保持关键字按顺序存储的容器的名字都以单词`unordered`开头。

定义位置：
- 类型`map`和`multimap`定义在头文件`map`中；
- `set`和`multiset`定义在头文件`set`中
- 无序容器则定义在头文件`unordered_map`和`unordered_set`中。

## 11. 1 使用关联容器
- `map`和`set`的简单介绍：
	- `map`：`map`类型通常被称为**关联数组。** 关联数组与“正常”数组类似，不同之处在于其**下标不必是整数**。
	- `set`：`set`就是关键字的简单集合。当只是**想知道一个值是否存在**时，`set`是最有用的

#### 使用map
- 使用关键数组`map`进行单词计数。
```c++
map<string,size_t> word_cout;//string到size_t的空map
string word;
//统计
while(cin>>word){
	++word_count[word];//提取word的计数器并加1
}
//输出
for(const auto &w:word_count)//对map中每个元素
	cout<<w.first<<"occurs"<<w,second<<((w.second>1?"times":"time")<<endl;)
```
- 关联容器也是模板，定义时必须**指定关键字和值的类型**，上例中关键字是`string`，值是`size_t`

#### 使用set
- 对[[#使用map]]中的程序进行扩展：忽略常见单词，如“the”、“and”，“or”，我们使用`set`保存想忽略单词，只对不在集合中的单词统计出现次数。
```c++
//统计输入中每个单词出现的次数,忽略常见单词
map<string,size_t> word_count;//string到size_t的空map
set<string> exclude = {"the","but","and","or"};
string word;
while(cin>>word)
	//不存在set中则添加到map	if(exclude.find(word)==exclude.end)
	++word_count[word];//获取并递增word的计数器
```
- `set`也是模板。为了定义一个`set`，必须指定其元素类型,本例中`set`是`string`类型。

## 11.2 关联容器概述
- 关联容器（有序的和无序的）都支持[[CPP#^fdecfb|普通容器操作]]。
- 关联容器不支持顺序容器的位置相关的操作，例如`push_front`或`push_back`。原因是关联容器中元素是根据关键字存储的，这些操作对关联容器没有意义。
- 除了与顺序容器相同的操作之外，关联容器还支持一些顺序容器不支持的操作和类型别名。此外无序容器还提供一些用来调整哈希性能的操作。
- 关联容器的迭代器都是双向的.

### 定义关联容器
#### 初始化map和set
在新标准下，我们也可以对关联容器进行值初始化：
```c++
map<strin,size_t> word_count;//空容器
set<string> exclude={"1",'2'}
map<string,string> autors={
{"Joyce","James"},
{"HHH","heiheihei"}
};
```
- 当初始化一个`map`时，必须提供关键字类型和值类型。我们将每个关腱字一值对包围:
```c++
{key,value}
```

#### 初始化multimap或multiset
map和set中，关键字必须是唯一的。容器`multimap`和`multiset`的**关键字可以重复。**
```c++
vector<int> ivec{1,1,2,2,3,3};
//使用迭代器初始化
set<int> iset(ivec.cbegin(),ivec.cend);//包含3个元素
multiset<int> miset(ivec.cbegin(),ivec.cend);//包含六个元素
```

### 11.2.2 关键字类型的要求
- 对于**无序容器**中关键字的要求，[[#11.4 无序容器]]
- 对于**有序容器**——`map`、`multimap`、`set`、`multset`，关键字类型必须定义元素比较的方法。默认使用`<`

#### 有序容器的关键字类型
- 可以使用[[#10.3 定制操作|定制操作]]向算法提供一个自定义的比较操作，替换关键字上的`<`运算符。
- 所提供的操作必须在关键字类型上定义一个**严格弱序**，即是“**小于等于**”。
- 自定义比较操作的函数必须具备如下性质：
	- 两个关键字**不能同时“小于等于**”对方。
	- **比较传递性**：如果k1“小于等于”k2，且k2“小于等于”k3，那么k1必须“小于等于”k3。
	- “等价传递性”：如果存在两个关键字，任何一个都不“小于等于”另一个，那么我们称这两个关键字是“**等价**”的。如果k1“等价于”k2，且k2“等价于”k3，那么k1必须“等价于”k3。
- 如果两个关键字是**等价的**（即，任何一个都不“小于等于”另一个），那么容器将它们视作相等来处理。
>[!note]
>在实际编程中，重要的是，如果一个类型定义了“行为正常”的`<`运算符，则它可以用作关键字类型。

#### 使用关键字类型的比较函数
- 为`Sales_data`定义一个函数，使其有一个**严格弱序**，并且提供比较运算符。
```c++
bool compareIsbn(const Sales_data &lhs,const Sales_data &rhs)
{
	return lhs.isbn()<rhs.isbn();
}
```

- 使用定制操作，创建有序容器：必须提供两个类型：
	- 关键字类型
	- 比较操作类型——一种[[CPP#函数指针|函数指针]]类型。
```c++
//bookstore中多条记录可以有相同的ISBN
//bookstore中的元素以ISBN的顺序进行排列
multiset<Sa1es_data,decltype(compareIsbn)*> bookstore(compareIsbn)；
```
>使用`decltype`来指出自定义操作的类型，根据函数类型生成函数指针。
>用`compareIsbn`来初始化`bookstore`对象，这表示当我们向`bookstore`通过调用`compareIsbn`来为这些**元素排序**。

>[!note]
>- 自定义比较操作必须提供严格**弱序**
>- 对于没有`<`操作的类自定义类，在生成有序容器时就需要自定义比较操作
>- 创建自定义比较操作的有序容器：`容器<类型,函数指针>`,，如果初始化时把比较操作函数作为参数进行初始化，那么创建额容器将按照这个操作进行排序。


### 11.2.3 pair类型
- `pair`的标准库类型：它定义在头文件utility中。
- `pair`**作用**：用来生成特定类型的模板。
- **创建**`pair`：
	- 一个`pair`保存两个数据成员，当创建一个`pair`时，我们必须提供两个类型别名，两个类型不要求一样：

- 默认初始化：
```c++
pair<string,string> annon;
pair<string,size_t> word_count;
pair<string,vector<int>> line;
```
>	`pair`的默认构造函数对数据成员进行**值初始化**。因此初始化之后anno是一个包含两个空`string`的`pair`

- 提供初始化值：以下一个`pair`两个成员被初始化为对应的值
```c++
pair<string,string> author{"hello","C++"};
```

- `pair`的数据成员：
	- `pair`的数据成员是`public`，分别命名为`first`和`second`
```c++
author.first=="hello";
author.word=="C++";
```

- `pair`的操作：
![[Pasted image 20230128162354.png]]

#### 创建pair对象的函数
- 创建一个返回一个`pair`的函数：
```c++
pair<string, int> process(vector<string> &v){
	//处理v
	if(!v.empty())
		//不为空，返回一个由v中最后一个string机器大小组成的pair
		return {v.back(),v.back.size()};//列表初始化
	else
		//构造一个空pair
		return pair<string,int>()//隐式构造返回值
}
```

## 11.3 关联容器操操作
![[Pasted image 20230128162823.png]]
![[Pasted image 20230128162837.png]]
除了上表列出的类型，关联容器还定义了以下操作：
![[Pasted image 20230128162928.png]]

- 对于`set`类型：`key_type`和`value_type`是一样的。`set`中保存的就是关键字。
- 对一个`map`类型：每个元素是`pair`对象，包含一个关键字和一个关联的值，由于关键字不能改变，所以`pair`的关键字部分是`const`的。
- 可以使用作用域运算符提取一个类型的成员。例如：`map<string,int>::key_type`

### 11.3.1 关联容器迭代器
当解引用一个关联容器迭代器时，我们会得到一个类型为容器的`value_type`的值的引用。
	- 对于`map`：`value_type`是一个`pair`类型。
	- 对于`set`：`set`的迭代器是`const`的，虽然`set`类型同时定义了`iterator`和`const_iterator`类型，但两种类型都**只允许只读访问`set`中的元素。**

```c++
auto it = word_count.begin();
it->first;//关键字，是const的不能改变
it->second;//值，可以改变

set<int> iset={1,2,3};
set<int>::iterator set_it = iset.begin();
*set_it ==1;//由于set中const是只读，所以不能赋值。
```

#### 遍历关联容器
`map`和`set`类型都支持`begin`和`end`操作。
```c++
auto map_it = word_count.cbegin();
while(map_it!=word_count.cend()){
	cout<<map_it->first<<":"<map_it->second<<endl;
	++map_it;//递增这代器，移动到下一个元素
}
```

#### 关联容器和算法
- 我们**通常不对关联容器使用泛型算法。**
**关键字是`const`** 这一特性意味着不能将关联容器传递给修改或重排容器元素的算法。

- 关联容器可**用于只读取元素的算法**。但是，很多这类算法都要搜索序列。由于关联容器中的元素不能通过它们的关键字进行（快速）查找，所以**对其使用泛型搜索算法几乎总是个坏主意**。

- 在实际编程中，如果我们真要对一个关联容器使用算法，要么是将它当作一个源序列，要么当作一个目的位置。例如，可以用泛型`copy`算法将元素从一个关联容器拷贝到另一个序列。

### 11.3.2 添加元素
关联容器的`insert`成员向容器中添加一个元素或一个元素范围。
由于`map`和`set`包含不重复的关键字，因此插入一个已经存在的元素对容器没有任何影响。
![[Pasted image 20230129134708.png]]

#### 向map添加元素
`map`的一个元素的类型是`pair`，插入时，可以在`insert`的参数列表创建一个`pair`。
```c++
word_count.insert((word,1));
//调用make_pair显示构造
word_count.insert(make_pair(word,1));
word_count.insert(pair<string,size_t>(word,1));
//构造一个恰当的pair类型，并构造该类型的一个新对象，插入到map中。
word_count.insert(map<string,size_t>::value(word,1));
```

#### 检测insert的返回值
- `insert`的返回值：`insert`（或`emplace`)返回的值依赖于**容器类型**和**参数**。

- 添加单一元素的`insert`和`emplace`的返回值：返回一个`pair`，包含两个成员`first`和`second`。
	- `first`成员：是一个迭代器，指向具有给定关键字的元素。
	- `second`成员：是一个`bool`值，指出元素是否成功插入还是已存在。
		- `true`：元素被插入
		- `false`：元素已存在，该操作说明也不做。
![[Pasted image 20230129135249.png]]
>`if`语句检查返回值的`bool`部分，若为`false`，则表明插入操作未发生。在此情况下，`word`己存在于`word_count`中，因此必须递增此元素所关联的计数器。

#### 展开递增语句
```c++
//下面两个语句等价
++ret.first->second;
++((ret.first)->second);
```
- `ret`：保存`insert`返回的值，是一个`pair`
- `ret.first`：是一个`map`迭代器，指向具有给定关键字的元素。
- `(ret.first)->second`：`map`中元素的值部分。
- `++ret.first->second`：递增`map`中元素的值。

#### 向multiset或multimap添加元素
- `multimap`：一个关键字有多个值。关键字不必唯一
```c++
multimap<string,string> authors;
authors.insert({"hello","C++"});
authors.insert({"hello","world"});//关键字也是“hello”
```

- 对**允许重复关键字的容器**，接受**单个元素**的`insert`操作**返回**一个指向新元素的**迭代器**。因为`insert`总是向这类容器中添加元素。


### 删除元素
关键容器定义了三个版本的`erase`来删除容器。
- 第一个版本：参数是一个迭代器，用于删除一个元素。指定元素被删除返回`void`
- 第二个版本：参数是一对迭代器，用于删除一对元素。指定元素被删除返回`void`
- 第三个版本：接受一个`key_type`参数，用于删除所有匹配给定关键字的元素，
	- 如果元素存在返回实际删除元素的数量。
	- 如果元素不存在，返回0
```c++
word_count.erase(removal_word)
```
![[Pasted image 20230129141400.png]]


### 11.3.4 map的下标操作
- `map`和`unordered_map`容器提供了下标运算符和一个对应的[[CPP#访问元素|at函数]]。
![[Pasted image 20230129141939.png]]
>[!note]
>`map`下标运算符接受一个索引，**如果关键字不存在，将会为它创建一个元素并插入到`map`中，关联值将进行值初始化。** 如果不希望添加新元素只想查找，应该使用`find`，不能使用下标。
```c++
map<string,size_t> word_count;
//插入一个关鍵字为Anna的元素，关联值进行值初始化；然后将1赋予它
word_count["Anna"]=1
```
- 在`word_count`中搜索关键字为`Anna`的元素，未找到。将一个新的关键字一值对插入到`word_count`中。关腱字是一个`const string`
- 保存`Anna`值进行值初始化，向初始化为0；
- 提取出新插入的元素，并将值1赋予它。

对于**其他关联容器**：
- `set`类型不支持下标，因为`set`中没有与关键字相关联的“值”，`set`中保存的元素本身就是关键字。
- 不能对一个`multimap`或一个`unordered_multimap`进行下标操作，因为这两个容器中一个关键字对应着多个值。

#### 使用下标操作的返回值
- 对于`map`对象：
	- 进行下标操作，获取一个`mapped_type`对象,返回[[CPP#左值和右值|左值]]，可以进行读写操作。
	- 解引用一个`map`迭代器：一个`value_type`对象.
```c++
++word_count["Anna"];//提取元素，然后增1
```

>[!warning]
>与`vector`与`string`不同，`map`的下标运算符返回的类型与解引用`map`迭代器得到的类型不同。


### 11.3.5 访问元素
- `find`函数：用于不允许重复的容器时和`count`效果类似。如果不需要计数，最好使用`find`.
	- 元素存在，返回指向该元素的迭代器
	- 元素不存在，返回指向该容器的尾后迭代器
- `count`函数：用于不允许重复的容器时和`find`效果类似。对于允许重复关键字的容器：`count`还会统计有多少个元素相同的关键字。
	- 元素存在，返回该元素的数量
	- 元素不存在，返回0

```c++
set<int> iset = {0,1,2,3};
iset.find(1);//找到。返回一个迭代器，指向key==1的元素。
iset.find(11);//找不到。返回指向iset.end()的迭代器

iset.count(1);//，找到，返回1
iset.count(1);//找不到，返回0
```
![[Pasted image 20230129144518.png]]
![[Pasted image 20230129144525.png]]

#### 对map使用find代替下标操作

**好处：** 只查找元素是否存在，不改变`map`
```c++
if(word_count.find("Anna")==word_count.end()){
	cout<<"Anna is not in the map"<<endl;
}
```

#### 在multimap或multiset中查找元素
如果一个`multimap`或`multiset`中有**多个元素**具有给定关键字，则这些元素在容器中会**相邻存储**。
```c++
//利用该性质打印同一作者的所有图书
string search_item("Alain de Botton");//要查找的作者
auto count = author.count(search_item);
auto iter  =author.find(search_item);//此作者的第一本书
while(count){
	cout<<iter->second<<endl;
	++iter;
	--entries;
}
```

#### 一种不同的，面向迭代器的解决方法
- `lower_bound`：如果元素存在，返回的迭代器将指向第一个具有给定关键字的元素。
- `upper_bound`：如果元素存在，返回的迭代器则指向最后一个匹配给定关键字的元素之后的位置。
如果元素不存在，则返回以上两个函数会返回相等的迭代器，指向一个不影响排序的关键字插入位置。
因此，用相同的关键字调用`lower_bound`和`upper_bound`会得到一个迭代器范围。

```c++
//打印同一作者的所有图书
string search_item("Alain de Botton");//要查找的作者

auto iter  =author.find(search_item);//此作者的第一本书
for(auto beg = author.lower_bound(search_item),end=author.upper_bound(search_item);beg!=end;++beg;)
	cout<<beg->second<<endl;//打印每本图书
```
如果**没有**元素与给定关键字匹配，则`lower_bound`和`upper_bound`**会返回相等的迭代器**一一都指向给定关键字的插入点，能保持容器中元素顺序的插入位置。

#### equal_range函数
- `equal_rang`函数：此函数**接受一个关键字**，返回一个迭代器`pair`，即`pair`中两个元素都是迭代器。
	- 如果关键字存在：则第一个迭代器指向第一个与关键字匹配的元素。第二个得带起指向最后一个**匹配元素之后的位置**。
	- 如果关键字不存在，则两个迭代器都指向关键字可以插入的位置。
```c++
string search_item("Alain de Botton");//要查找的作者

auto iter  =author.find(search_item);//此作者的第一本书
for(auto pos = authors.equal_range(search_item);pos.first!pos.second;++pos.first)
	cout<<pos.first->second<<endl;//打印每个题目
```

调用`equal_range`返回的迭代器`pair`中的两个迭代器相当于调用`lower_bound`和`upper_bound`返回的迭代器。


### 11.3.6 例子：一个单词转换的map
- **功能：** 将第二个文本文件，按照第一个文件中的规则进行转换。例如u转换成you，pic转换成picture。

- **定义函数：** 
	- `word_transform`：接受两个`ifstream`参数，读取第一个文件中的转换规则和第二个文件中待转换的文本。
		- 第一个参数：绑定第一个文件
		- 第二个参数，绑定第二个文件
	- `build_Map`：会读取转换规则文件，并创建一个map，用于保存每个单词到其转换内容的映射。
	- `transform`：接受一个`string`，如果存在转换规则，返回转换后的内容。

```c++
map<string,string> build_Map(ifstream &rule){
	map<string,string> trans_map ;//保存转换规则
	string key;
	string value;
	//这里因为我们假设规则都是由一个单词和一个短语构成，所以利用>>字节读取单词，再利用getline读取这一行中余下的内容
	while(rule>>key&&getline(rule，value)){
		if(value.size()>1)
			//存在转换规则，去掉最前面的空格
			trans_map[key] = value.substr(1);
		else
			throw runtime_error("no rule"+key);
	return trans_map
	
	}
}
const &string transform(const string &word,map<string,string> rule){
	//利用count
	auto count = rule.count(word);
	if(count!=0)
		return map[word];
	else
		return word;
		
	//利用find
	auto it = rule.find(word);
	if(it!=word.cend())
		return it->second;
	else
		return word;
	
}

void word_transform(ifstream &rule,&input){
	auto mrule = build_Map(rule);
	string text;
	while(getline(input,text)){//读取文件中的一行保存在text
		istringstream stream(text);//创建一个流，读取每个string
		string word;
		bool firstword = true;
	while(stream>>word){
		
		if(firstword)//使得第一个单词不打印空格
			firstword = false;
		else
			cout<<"";//在单词间打印一个空格
		cout<<transform(word,trans_map);
	}
	cout<<endl;//完成一行的转换
	}
}

```
>函数`substr`参见[[CPP#构造string的其他方法]]

## 11.4 无序容器
新标准定义了4个**无序关联容器**。`unordered_map`、`unordered_set`
- **无序关联容器：** 不是使用比较运算符来组织元素，而是使用一个**哈希函数**和关键字类型的`==`运算符。
- **适用范围**：在关键字类型的元素没有明显的序关系的情况下。
- **关键字是否可重复**：无序容器也有允许重复关键字的版本。

#### 使用无序容器
- 一些有序容器的操作也能应用于无序容器，
使用无序容器读取单词时，不太可能按字典序输出。

#### 管理桶
- **无序容器在存储上组织为一组桶，每个桶保存零个或多个元素。**
- 无序容器使用一个**哈希函数**将元素映射到桶。
- 访问元素时，容器首先计算元素的哈希值，它指出应该搜索哪个桶。
- 容器将具有一个特定哈希值的所有元素都保存在相同的桶中。
- 如果元素允许关键字重复，那么具有相同关键字的元素都会在一个桶中。
- 无需容器的性能依赖于哈希函数的质量和桶数量和大小。

- **哈希函数：** 对于相同的参数产生同样的结果。理想情况下，哈希函数还能将每个特定的值映射到唯一桶。但是，将不同关键字的元素映射到相同的桶也是允许的。
- 当**一个桶**保存多个元素时，需要**顺序搜索**这些元素来查找我们想要的那个。当一个桶中保存了很多元素，那么查找一个特定元素就需要大量的比较操作。

- 无序容器管理桶的函数：这些成员函数允许我们查询容器的状态以及在必要时强制容器进行重组。
![[Pasted image 20230129162852.png]]

#### 无序容器对关键字类型的要求
- 默认情况下，无序容器使用关键字类型的`==`运算符来比较元素，它们还使用一个`hash<key_type>`类型的对象来**生成每个元素的哈希值**。

- 标准库为内置类型（包括指针）提供了`hash`模板。还为一些标准库类型，包括`string`和智能指针类型定义了`hash`，因此，我们**可以直接定义关键字是内置类型（包括指针类型）、`string`还是智能指针类型的无序容器。**

- 自定义类类型需要我们自己定义`hash`模板以及`==`操作之后才可以使用该类型的无序容器。
```c++
//自定义哈希
size_t hasher(const Sales_data &sd){
	return hash<string>()(sd.isbn());
}
//自定义==
bool eqOp(const Sales_data &lhs,const Sales_data &rhs){
	return lhs.isbn()==rhs.isbn();
}

//定义一个无序可重复的set
using SD_multiset = unordered_multiset<Salas_data,decltype(hasher)*,decltype(eqOp) *>;
//参数是桶大小，哈希函数指针和相等性符指针
	SD_multiset(42,hasher,eqOp);
```
>有关函数指针，参见[[CPP#函数指针]]

>[!note]
>- 有序容器的迭代器通过关键字有序访问容器中的元素。
>- 无论在**有序容器**中还是在**无序容器**中，具有相同关键字的元素都是**相邻存储**的。



# C++标准库：动态内存

栈内存和静态内存中的对象由编译器自动创建和销毁。
- **静态内存** ：静态内存用来保存局部`static`对象、类`static`数据成员以及定义在任何函数之外的变量。`static`对象在使用之前分配，在程序结束是时销毁。
- **栈内存**：栈内存用来保存定义在函数内的非`static`对象。对于栈对象，仅在其定义的程序块运行时才存在。

动态分配的对象的生存期与它们在哪里创建是无关的，只有当显式地被释放时，这些对象才会销毁。
- **内存池：** 也叫**自由空间**或者对**堆**。程序用堆来存储动态分配的对象，即那些在程序运行时分配的对象。动态内存的生存期由程序来控制，不再使用时，我们的代码必须显式地销毁它们。 ^491a30

## 12.1 动态内存和智能指针
- 管理动态内存：
	- 运算符`new`，在动态内存中为对象分配空间并返回一个指向该对象地指针。
	- 运算符`delete`：接受一个动态对象地指针，销毁该对象，并释放与之关联地内存。

- **两种智能指针**：用于管理动态对象，行为类似常规指针。负责自动释放所指向地对象。
	- `shared_ptr`：允许多个指针指向同一个对象。
	- `unique_ptr`：独占所指对象。
	- `weak_ptr`：伴随类，它是一种弱引用，指向`shared_ptr`所管理的对象。
>以上三种类型都定义在头文件`memory`中 ^07af99

### 12.1.1 shared_ptr类
智能指针也是[[CPP#^575c7a|模板]]。因此，创建时必须提供其可以指向的类型，默认初始化的智能指针中保存着一个[[CPP#空指针|空指针]]。
```c++
shared_ptr<string> p1;//p1，可以指向string
shared_ptr<list<int>> p2;//p2可以指向int的list

if(p1&&p1->empty()){
	*p1="hi";//如果pl指向一个空string，解引用pl，将一个新值赋予string
}
```

![[Pasted image 20230130153452.png]]
![[Pasted image 20230130153500.png]]

#### make_shared函数
- `make_shared`函数：最安全的分配和使用动态内存的方法。该函数在动态内存中分配一个对象并初始化它，返回指向此对象的`shared_ptr`。存在于头文件memory中
```c++
//创建一个指向值42的int的shared_ptr
shared_ptr<int> p3=make_shared<int>(42);

//p4指向一个值为“999999999的string
shared_ptr<string> p3=make_shared<string>(10,9);

//p5指向一个值初始化的int,值为0
shared_ptr<int> p3=make_shared<int>;

//通常使用auto保存
auto p6 = make_shared<vector<int>>();
``` 

类似于[[CPP#使用emplace操作|顺序容器的emplace]]成员，`make_shared`用其参数来构造给定类型的对象。调用`make_shared`时传递的参数必须能用来初始化一个`int`。

#### shared_ptr的拷贝和赋值
当进行拷贝或赋值操作时，每个`shared_ptr`都会**记录有多少个其他`shared_ptr`指向相同的对象**：
```c++
auto p = make_shared<int>(42);//P指向的对象只有p一个引用者

//拷贝初始化
auto q(p);//p和q指向相同对象，此对象有两个引用者
```
每个`shared_ptr`都有一个关联的计数器，通常称其为**引用计数**,。
- 递增：利用一个`shared_ptr`拷贝初始化另一个`shared_ptr`，或者将它作为参数传递给一个函数以及作为函数的返回值时，计数器都会递增。
- 递减：给`shared_ptr`赋一个新值，或者被销毁时，计数器就会递减。

一旦一个shared_ptr的计数器变成0，**它就会自动释放自己所管理的对象。**
```c++
auto r = make_ptr<int>(42);//r指向的int只有r本身一个引用着。
r=q;//给r赋值，令它指向另一个地址
//递增q指向的对象的引用计数
//递减r原来指向的对象的引用计数
//r原来指向的对象已没有引用者，会自动释放
```

#### shared_ptr自动销毁所管理的对象
当指向一个对象的最后一个`shared_ptr`被销毁（通过**析构函数**）时，`shared_ptr`类会自动销毁此对象。

**析构函数**：类似构造函数，每个类都有一个析构函数，用于控制此类型对象销毁时做什么操作。

**析构函数用途**：一般用来释放对象所分配的资源。销毁对象，释放该对象的占用内存。

`shared_ptr`的析构函数会递减它所指向的对象的引用计数。
如果引用计数变为0，`shared_ptr`的析构函数就会销毁对象，并释放它占用的内存。

#### shared_ptr还会自动释放相关联的内存
![[Pasted image 20230130160747.png]]
![[Pasted image 20230130160753.png]]
- `p`离开作用域，`p`被销毁，由于`p`是唯一因为`factory`返回的内存的对象（`shared_ptr`指针），所以`p`指向的对象也销毁。

![[Pasted image 20230130160923.png]]
- `p`离开作用域，被销毁。但是`factory`返回的内存的对象（`shared_ptr`指针）通过返回`p`的拷贝，计数增加，所以不会被销毁。

>[!note]
>如果你将`shared_ptr`存放于一个容器中，而后不再需要全部元素，而只使用其中一部分，要记得用`erase`删除不再需要的那些元素.

#### 使用了动态生存期的资源的类
- 程序使用动态内存出于以下三种原因之一:
1. 程序不知道自己需要使用多少对象。（容器类）
2. 程序不知道所需对象的准确类型
3. 程序需要在**多个对象间共享数据**(常见原因)

对应第一种情况，我们使用容器类管理动态内存。该例子中，**资源**与**对象**有**相同的生存期。**
```c++
vector<int> v1
{
	vector<int> v2={1,2,3};
	v1=v2;//从v2拷贝元素到vl中
}//从v2被销毁，其中的元素也被销毁。
//v1有三个元素，是原来v2中元素的拷贝。
```
>由一个`vector`分配的元素只有当这个`vector`存在时才存在。

某些类分配的**资源**具有与原**对象**相**独立的生存期**。
```c++
//Bob类对象的不同拷贝之间，共享相同数据。
Bob<int> b1;
{
	Bob<int> b2={1,2,3,4};
	b1=b2;//bl和b2共享相同的元素
}//b2被销毁了，但b2中的元素不能销毁
//bl指向最初由b2创建的元素
```
>因为两个对象共享底层数据，所以当一个对象销毁时，它的底层数据还在。

#### 定义StrBIob类
看书去吧P404，这部分主要是使用智能指针管理一个类，类数据对象只有一个智能指针`shared_ptr<vector<string>>`，当最后一个引用对象被销毁时，其指向的`vector<string>`也会自动销毁。

### 12.1.2 直接管理内存
C++语言定义了**两个运算符**来分配和释放动态内存。
- 运算符`new`分配内存，
- `delete`释放`new`分配的内存。
>[!warning]
>相对于智能指针，使用这两个运算符管理内存非常容易出错。

#### 使用new动态分配和初始化对象
- 使用`new`：
```c++
int*pi= new int;//在自由空间构造一个int型对象，并返回指向该对象的指针
```
>[!note]
>在**自由空间(堆)** *分配的内存是无名的*，因此无法为其分配的对象命名，而是返回一个指向该对象的指针

`new`的默认初始化和直接初始化和值初始化：
- 默认情况下，**动态分配的对象是[[CPP#默认初始化|默认初始化]]的**，内置类型或组合类型的对象的值将是未定义的，自定义的类将使用默认构造方法。
```c++
string *ps=new string;//初始化为空string
int *pi=new int;//pi指向一个未初始化的int
```

- 利用[[CPP#直接初始化和拷贝初始化|直接初始化]]来初始化一个动态分配的对象
```c++
int *pi = new int(1024);//pi指向的对象值为1024
string *ps = new string(10,'9');//*ps为“9999999999”
``` 

- 值初始化一个动态分配的的对象：类型名之后紧跟一对**空括号**
```c++
string *ps1 = new string//默认初始化为空string
string *ps=news tring();//值初始化为空string
```
>[!note]
>- 对于定义了自己的构造函数的类，不管采用什么形式，对象都会通过**默认构造函数来初始化**。
>- 对于内置类型
>	- 值初始化：有着良好定义的值
>	- 默认初始化：值是未定义的

- 使用`auto`推断要分配的对象的类型：提供了一个括号包围的初始化器
```c++
auto pl = new auto(obj);//p指向一个与obj类型相同的对象,该对象用obj进行初始化
auto  p2 = new auto{a,b,c};//错误：括号中只能有单个初始化器
```
>`p1`是一个指针，指向`auto`推断出来的类型，如果`obj`是`int`那么`p1`就是`int *`，新分配的对象使用obj进行初始化。

#### 动态分配的const对象
```c++
//分配并初始化一个const int
const int *pci = new const int(1024);
//分配并默认初始化一个const的空string
const string *pcs=new const string;
```
类似其他任何`const`对象，一个**动态分配的`const`对象必须进行初始化。** 由于分配的对象是`const`的，`new`返回的指针是指向**const**的指针（底层`const`）
>对于一个定义定义了默认构造函数的类类型，其const对台对象可以隐式初始化，其他类型必须显式初始化。


#### 内存耗尽
**一旦一个程序用光了它所有可用的内存，`new`表达式就会失败。** 默认情况下，如果`new`不能分配所要求的内存空间，它会抛出一个类型为`bad_alloc`的异常。

- 改变使用new的方式来阻止它抛出异常：
```c++
//如果分配失败，new返回一个空指针
int *p1 = new int;//如果分配失败，new抛出std::bad_a11oc
int *p2 = new (nothrow) int//如果分配失败，new返回一个空指针
```
这种形式的`new`为**定位`new`**。

- 定位`new`：允许我们向`new`传递额外的参数。如上例的`nothrow`对象。
	- 如果程序内存满了，这种告诉`new`不能抛出异常。并且返回一个**空指针**。

>`bad_alloc`和`nothrow`都定义在头文件`new`中。

#### 释放动态内存
- 作用：为了防止内存耗尽
- **`delete`表达式**：接受一个指针，指向我们想要释放的对象。**销毁**给定的指针指向的**对象**，**释放**对应的**内存**。
```c++
delete p;//p必须指向一个动态分配的对象或是一个空指针
```

#### 指针值和delete
- 传递给`delete`的指针必须是以下两种：
	- 动态分配（`new`）的内存
	- [[CPP#空指针|空指针]]
如果释放一块并非`new`分配的内存，或者将相同的指针值释放多次，其行为是未定义的。
通常情况下，编译器**不能分辨**一个指针指向的是**静态**还是**动态分配**的对象。

#### 动态对象的生存期直到被释放时为止
- 两种动态对象的区别：
	- `shared_ptr`：由其管理的内存在最后一个`shared_ptr`销毁时会自动释放。
	- `new`：直到被显式释放之前它都是存在的。[[#使用new动态分配和初始化对象]]
```c++
//factory返回一个指针，指向一个动态分配的对象。
Foo *factory(T arg){
	//处理arg
	return new Foo(arg);//调用者负责释放此内存
}
```
>在作用域中使用该函数的返回值，当离开作用域时，保存该返回值的变量将被销毁，但是这个被销毁对象指向的动态内存并没有被释放，需要程序员手动使用`delete`

>[!warning]
>由内置指针（`new`返回的对象）管理的动态内存在被显式释放前一直都会存在。

>[!note]
>使用`new`和`delete`**管理动态内存**存在三个常见问题：
>1. 忘记`delete`内存。忘记释放动态内存会导致人们常说的“内存泄漏"问题。查找内存泄露错误是非常困难的，因为通常应用程序运行很长时间后，真正耗尽内存时，才能检测到这种错误。
>2. 使用已经释放掉的对象。
>3. 同一块内存释放两次。当有两个指针指向相同的动态分配对象时，可能发生这种错误。如果对其中一个指针进行了`delete`操作，对象的内存就被归还给[[#^491a30|自由空间]]了。如果我们随后又`delete`第二个指针，自由空间就可能被破坏。

#### delete之后重置指针值
当我们`delete`一个指针后，指针值就变为无效了。但是有可能会该指针仍保留着（已经释放了）动态内存的地址，即使该地址中值可能已经改变或者不存在。

- **空悬指针**：指针指向已经被释放的动态内存的地址。
- **未初始化指针**的所有缺点**空悬指针**都有。

- 避免**空悬指针**问题：在指针即将要离开其作用域之前释放掉它所关联的内存。如果我们需要保留指针，可以在`delete`之后将`nullptr`赋予指针，这样就清楚地指出指针不指向任何对象。

##### 可能存在的问题
动态内存的一个基本问题是可能**有多个指针指向相同的内存。**
在`delete`内存之后重置指针的方法只对这个指针有效，对其他任何仍指向（己释放的）内存的指针是没有作用的。
![[Pasted image 20230130173900.png]]


### 12.1.3 shared_ptr和new结合使用
如果我们不初始化一个智能指针，它就会被初始化为一个空指针。
![[Pasted image 20230130174745.png]]
![[Pasted image 20230130174752.png]]

- 如上表所示，我们可以使用`new`返回的指针来初始化智能指针。
```c++
shared_ptr<double> p1;//shared_ptr可以指向一个double
shared_ptr<int> p2(new int(42));//p2指向一个值为42的int
```
接受指针参数的智能指针构造函数是[[CPP#explicit：抑制构造函数定义的隐式转换|explicit]]的，因此我们不能将一个内置指针隐式转换为一个智能指针，必须使用[[CPP#直接初始化和拷贝初始化|直接初始化]]。
```c++
shared_ptr<int> p1=new int(1024);//错误：右边是int*指针左边是智能指针，不能隐式转换，必须使用直接初始化形式

shared_ptr<int> fun(int p){
	return new int(p);//错误，不能将int指针隐式转换为智能指针。
}
shared_ptr<int> p2(new int(42));
//正确：使用了直接初始化形式
shared_ptr<int> fun(int p){
	return shared_ptr<int>(new int(p));//正确，显示进行值初始化之后返回
}
```

- 初始化智能指针的对象可以是以下几种：
	- 默认情况下，一个**用来初始化智能指针的普通指针必须指向动态内存**，因为智能指针默认使用`delete`释放它所关联的对象。
	- 也可以是指向其他类型资源的指针，但是必须提供自己的操作来替代`delete`。

#### 不要混合使用普通指针和智能指针
**`shared_ptr`可以协调对象的析构**，但这仅限于其自身的拷贝（也是`shared_ptr`）之间。

**不要使用内置指针访问智能指针指向的对象，** 因为我们无法知道对象何时会被销毁
```c++
int *x(new int(1024));//危险：x是一个普通指针，不是一个智能指针,这个时候的释放需要我们调用delete
process(x);//错误：不能将int*转换为一个shared_ptr<int>
process(shared_ptr<int>(x));//合法的，但内存会被释放！因为shared_ptr是在调用函数时创建的临时变量，在函数中计数为1，函数调用结束之后计数0，释放该对象以及指向的动态内存
int j = *x;//定义的：x是一个空悬指针！因为x在调用process之后指向的动态内存已经被释放了。
```
>[!warning]
>访问已经释放了的内存将造成内存泄漏！

#### 也不要使用get初始化另一个智能指针或为智能指针赋值

![[Pasted image 20230130192334.png]]
- **`get`函数**：返回一个内置指针，指向智能指针管理的对象。
	- **作用**：用于向不能使用智能指针的代码传递一个内置指针。

![[Pasted image 20230130194314.png]]

>[!warning]
>特别是，永远不要用`get`初始化另一个智能指针或者为另一个智能指针赋值。因为这样会让两个独立的智能指针指向同一个动态内存。会对一块动态内存`delete`两次。

#### 其他shared_ptr操作
- `reset`函数：用于将一个新的指针赋予一个`shared_ptr`。`reset`会更新引用计数，如果需要的话，会释放原来指向的对象。
```c++
p = new int(1024);//错误：不能将一个指针赋予shared_ptr
p.reset(new int(1024));//正确：p指向一个新对象
```

- `unique`函数：检查当前`shared_ptr`的指向是否唯一，如果是则需要进行备份。

- `reset`经常与`unique`一起使用，用来控制多个`shared_ptr`共享的对象。
例如：在改变底层对象之前，我们检查自己是否是当前对象仅有的用户。如果不是，在改变之前要制作一份新的拷贝：
```c++
if(!p.unique())
	p.reset(new string(*p));//我们不是唯一用户，新申请一块内存空间p指向它
*p+=newVal;//
```

### 12.1.4 智能指针异常
一个简单的确保资源被释放的方法是使用智能指针。
![[Pasted image 20230131145943.png]]
![[Pasted image 20230131145950.png]]

#### 智能指针和哑类

**析构函数：** 负责清理对象使用的资源。
包括所有标准库类在内的很多c++类都定义了析构函数。

一个没有定义析构函数，但是又为其分配了资源，可能会遇到与使用动态内存相同的错误，即资源没有被释放，如果再资源分配和释放之间发生了异常，程序也会发生资源泄露。

如何管理不具有良好定义的析构函数的类？
使用 `shared_ptr`进行管理，在程序结束时来保证内存能被正确释放。

#### 使用我们自己的释放操作
当一个`shared_ptr`被销毁时，它默认对它管理的指针进行`delete`操作，对于没有定义`delete`操作的类，我们必须定义一个函数来代替`delete`。这个**删除器**函数必须能够完成对`shared_ptr`中保存的指针进行释放的操作。

- 当我们创建一个`shared_ptr`时，可以传递一个（可选的）指向删除器函数的参数。
![[Pasted image 20230131151151.png]]
- 如果`f`正常退出：那么`p`的销毁会作为结束处理的一部分。
- 如果`f`发生了异常：`p`同样会被销毁，从而连接被关闭。

>[!note]
>智能指针使用的规范：
>- 不使用相同的内置指针值初始化（或`reset`）多个智能指针。否则将会把多个独立的智能指针绑定到相同的内存空间上，可能会发生内存泄漏。
>- 不`delete get()`返回的指针。因为`get`返回的是智能指针指向的内存空间。如果智能指针销毁，我们在使用`get`返回的对象时也会发生内存泄漏
>- 如果你使用`get()`返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效了。
>- 如果你使用智能指针管理的资源不是`new`分配的内存，记住传递给它一个删除器。也就是要自定义`delete`函数。


### 12.1.5 unique_ptr

`unique_ptr`“**拥有**”它所指向的对象。与`shared_ptr`不同，某个时刻只能有一个`unique_ptr`指向一个给定对象。当`unique_ptr`被销毁时，它所指向的对象也被销毁。

- 定义：`unique_ptr`需要将其绑定到一个`new`返回的指针上，初始化时必须采用[[CPP#直接初始化和拷贝初始化|直接初始化]]。
```c++
unique_ptr<double> p1;//可以指向一个double的unique_ptr
unique_ptr<int> p2(new int(42));//p2指向一个值为42的int

```
**`unique_ptr`不支持普通的拷贝或赋值**。
![[Pasted image 20230131154302.png]]

- 但是**可以转移**
	- `release`：返回`unique_ptr`当前保存的指针并将调用者`unique_ptr`其置空。切断`unique_ptr`与它原来管理的对象之间的关系。
	- `reset`：接受一个可选的指针参数，令`unique_ptr`重新指向给定的指针。如果`unique_ptr`不为空，那么它原来指向的对象被释放。
利用`release`或`reset`将指针的所有权从一个`unique_ptr`（非`const`）**转移**给另一个`unique_ptr`
```c++
////将所有权从pl（指向string Stegosaurus）转移给p2
unique_ptr<string> p2(p1.release());//release将p1置为空
unique_ptr<string> p3(new string)
//将所有权从p3转移给p2
p2.reset(p3.release())
```
>`release`返回的指针通常被用来初始化另一个智能指针或给另一个智能指针赋值。如果不用一个智能指针来管理`unique_ptr`返回的对象，那么我们需要手动释放。

#### 传递unique_ptr参数和返回unique_ptr

**例外：** **可以拷贝或赋值一个将要被销毁的`unique_ptr`。** 例如拷贝函数返回的`unique_ptr`。
```c++
unique_ptr<int> clone(int p){
	//正确：从int*创建一个unique_ptr<int>
	return unique_ptr<int>(new int(p));
}

unique_ptr<int> clone(int p){
	//正确：返回局部对象的拷贝
	unique_ptr<int> ret(new int (p));
	return ret;
}
```
编译器都知道要返回的对象将要被销毁。在此情况下，编译器执行一种**特殊的“拷贝”**.

>标准库的较早版本包含了一个名为`auto_ptr`的类，它具有`unique_ptr`部分特性。

#### 向unique_ptr传递删除器

- 重载`unique_ptr`默认的删除器：
>[!warning]
>`unique_ptr`管理删除器的方式与`shared_ptr`不同。
>重载一个`unique_ptr`中的删除器会影响到`unique_ptr`类型以及如何构造该类型对象。
```c++
//在创建或reset一个unique_ptr对象时，提供一个可调用对象（删除器）

//p指向一个类型为objT的对象，并使用一个类型为delT的对象释放objT对象
//它会调用一个名为fcn的delT类型对象
unique_ptr<objT,delT>p(new objT,fcn);
```

![[Pasted image 20230131160118.png]]

>函数指针，参见[[CPP#将auto和decltype用于函数指针类型|decltype用于函数指针]]


### 12.1.6 weak_ptr
前面已经简单介绍过[[#^07af99|weak_ptr]]，`weak_ptr`是一种**不控制所指向对象生存期**的智能指针，它指向一个由`shared_ptr`管理的对象，但是不会改变`shared_ptr`的引用计数。一旦`shared_ptr`被销毁，`weak_ptr`还是会被释放。
![[Pasted image 20230131160956.png]]

- 创建`weak_ptr`：用`shared_ptr`来初始化它
```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);//wp弱共享p；p的引用计数未改变
```

- `weak_ptr`访问对象：调用`lock`，不能直接访问，因为指向的对象可能不存在。
	- `lock`：存在则返回一个指向共享对象的`shared_ptr`。不存在则返回一个空指针。
```c++
//如果np不为空则条件成立
if(shared_ptr<int> np=wp.lock()){
	//在lf中，np与p共享对象
}
```

#### 核查指针类
- 利用`weak_ptr`和`shread_ptr`指针共享指针指向对象的资源，调用时，可以调用`weak_ptr`的`lock`，在确保指向对象存在时再添加数据。
- 
P421有一个例子


## 12.2 动态数组
- 另一种`new`表达式语法，可以分配并初始化一个对象数组。

- 使用容器的类可以使用默认版本的拷贝、赋值和析构操作。**分配动态数组的类型必须定义自己版本的操作**，在拷贝、复制以及销毁对象时管理所关联的内存。

### 12.2.1 new和数组
- 使用`new`分配一个对象数组：
```c++
//调用get_size确定分配多少个int
int *pia = new int[get_size()];//pia指向第一个int

typedef int arrT[42];//arrT表示42个int的数组类型
int *p = new arrT;//分配一个42个int的数组；p指向第一个int
```
>[]内的大小必须是整型，但不必是常量。

#### 分配一个数组会得到一个元素类型的指针
- 使用`new`分配对象数组，返回的是一个**元素类型的指针**。
- 由于由于分配的内存并不是一个数组类型，因此不能对动态数组调用`begin`或`end`。因为这些函数使用数组为度来返回指向首元素和尾后元素的指针。类似的，也不能用范围`for`。

>[!warning]
>动态数组并不是数组类型！！！

#### 初始化动态分配对象的数组
默认情况下，`new`**分配的对象**，不管是单个分配的还是数组中的，**都是默认初始化的**。

- 值初始化：大小后面跟一对`()`
```c++
int *pia = new int[20];//10个未初始化的int
int *pia2 = new int[10]();//10个值初始化为0的int

//新标准中，可以用{}
int *pia3=new int[3]{0,1,2};
//10个string，前4个用给定的初始化器初始化，剩余的进行值初始化剩余值都是xxx
string *psa3 = news tring[10]{"a","an","the",string(3,'x')};
```
>使用`{}`，相当于列表初始化，如果如果初始化器数目小于元素数目，剩余元素将进行值初始化。如果初始化器数目大于元素数目，则`new`表达式失败，不会分配任何内存，抛出一个`bad_array_new_length`异常，定义在`new`头文件中。

虽然我们用**空括号**`()`对数组中元素进行值初始化，但不能在括号中给出初始化器，这意味着不能用`auto`分配数组.

#### 动态分配一个空数组是合法的
```c++
char arr[0];//错误：不能定义长度为0的数组
char *cp=new char[0];//正确：但cp不能解引用
```
- 当我们用`new`分配一个大小为0的数组时，`new`返回一个合法的非空指针。
- 此指针保证与`new`返回的其他任何指针都不相同。
- 对于零长度的数组来说，此指针就像**尾后指针**一样，我们可以像使用尾后迭代器一样使用这个指针。可以用此指针进行比较操作。
- 可以向此指针加上（或从此指针减去）0，也可以从此指针减去自身从而得到0。但此指针**不能解引用一一毕竟它不指向任何元素。**

#### 释放动态数组
为了释放动态数组，我们使用一种特殊形式的`delete`：在指针前加上一个空方括号对。
```c++
delete p;//p必须指向一个动态分配的对象或为空
delete [] pa;//pa必须指向一个动态分配的数组或为空
```
第二条语句销毁`pa`指向的数组中的元素，并释放对应的内存。数组中的元素**按逆序销毁**，即，最后一个元素首先被销毁，然后是倒数第二个，依此类推。

- `[]`的作用，空方括号对是**必需**的，它指示编译器此指针指向一个对象数组的第一个元素。错用`[]`会产生未定义行为。
**使用类型别名定义的动态数组**，`delete`时也必须使用`[]`
```c++
type int arrT[];
int *p = new arrT;//分配一个42个int的数组；p指向第一个元素
delete [] p;//方括号是必需的，因为我们当初分配的是一个数组
```

>[!warning]
>如果我们在`delete`一个数组指针时忘记了方括号，或者在`delete`一个单一对象的指针时使用了方括号，编译器很可能不会给出警告。我们的程序可能在执行过程中在没有任何警告的情况下行为异常。

#### 智能指针和动态数组
标准库提供了一个可以管理`new`分配的数组的`unique_ptr`版本。
- 使用方法：必须在对象类型后曲跟一对空方括号`[]`.
```c++
unique_ptr<int[]> up(new int[10]);
up.release();//自动用de1ete[]销毁其指针
```
类型说明符中的方括号(`<int []>`)指出`up`指向一个`int`数组而不是一个主`int`，销毁时，会自动调用`delete []`

- 指向数组的`unique_ptr`提供的操作与我们在[[#12.1.5 unique_ptr]]中有一些不同。如下：
	- 不能使用`.`和`->`。
	- 可以使用下标访问数组中元素。
![[Pasted image 20230131171014.png]]
```c++
//up是指向动态数组的unique_ptr
up[0]=123;
```

- 与`unique_ptr`不同，`shared_ptr`**不直接支持**管理动态数组。
如果希望使用`shared_ptr`管理一个动态数组，**必须提供自己定义的删除器**：
```c++
//为了使用shared_ptr,必须提供一个删除器
shared_ptr<int> sp(new int[10],
[](int(*p){delete []P;}
//使用我们提供的lambda释放数组，它使用delete[]
sp.reset();//使用我们提供的lambda释放数组，它使用delete []
```
>如果未提供删除器，这段代码将是未定义的。默认情况下，`shared_ptr`使用`delete`来销毁对象，由于没有`[]`，所以会报错

- `shared_ptr`未定义下标运算符(`unique_ptr`可以用下标)。
- 而且**智能指针类型**不支持指针算术运算。因此，为了访问数组中的元素，必须用`get`获取一个内置指针，然后用它来访问数组元素。
>智能指针支持算术运算，所以用`get`转换为普通类型指针。


### 12.2.2 allocator类
当**分配一大块内存**时，我们通常计划在这块内存上按需构造对象。在此情况下，我们希望将**内存分配和对象构造分离**。这意味着我们可以**分配大块内存，但只在真正需要时才真正执行对象创建操作**（同时付出一定开销）。

- 使用`new`分配大内存的弊端：
1. 分配过多，创建一些永远用不到的对象
2. 对于需要用到的对象，在默认初始化时赋值了一次，在后面赋值的时候又赋值了一次。
3. 没有默认构造函数的类不能奉陪动态数组

#### allocator类
- `allocator`类：定义在头文件`memory`中，是一个模板，定义时必须指令可以分配的对象类型。当一个`allocator`对象分配内存时，它会根据给定的对象类型来确定恰当的内存大小和对齐位位置。
```c++
allocator<string> alloc;//可以分配string的allocator对象
auto const p=alloc.allocate(n);//分配n个未初始化的string
```

- `allocator`类作用：帮助我们**将内存分配**和**对象构造**进行**分离**。

- `allocator`类操作：
![[Pasted image 20230131172820.png]]

#### allocator分配未构造的内存
- `allocator`分配的**内存是未构造的**。我们可以按需在内存中构造对象。

- `construct`成员函数：接受一个指针和零个或多个额外参数，**在给定位置构造一个元素。**
	- 指针参数：表示构造的位置
	- 额外参数：额外参数用来初始化构造的对象。**必须是与构造的对象的类型相匹配的合法的初始化器。**
```c++
auto q=p;//q指向最后构造的元素之后的位置
alloc.construct(q++);//*q为空字符串
alloc.construct(q++,10,'c');//*q为cccccccccc
alloc.construct(q++,"hello");//*q为hello
```
>还未构造对象的情况下就使用原始内存是错误的。**使用`allocate`返回的内存，必须使用`construct`构造对象**。

- `destory`成员函数：用于**销毁构造的元素对象**。接受一个指针，对指向的对象执行析构函数。
```c++
alloc.destory(q);//释放我们真正构造的string
```
>我们只能对真正构造了的元素进行`destroy`操作

- `deallocate`函数：**释放内存**。接受一个指针，和一个大小参数。
	- *指针参数*：不能为空，必须指向由`allocate`分配的内存
	- *大小参数*：必须与调用`allocate`分配内存时提供的大小参数具有相同的值。
```c++
alloc.deallocate(p,n);
```


#### 拷贝和填充未初始化内存的算法

**两个伴随算法**：可以在未初始化内存中创建对象。定义在头文件`memory`中。
![[Pasted image 20230131174159.png]]

- 例子：假定有一个`int`的`vector`，我们希望将其内容拷贝到动态内存中。我们将分配一块比`vector`中元素所占用空间**大一倍**的动态内存，然后将原`vector`中的元素拷贝到**前一半**空间，对**后一半**空间用一个给定值进行填充：
```c++
////分配比vi中元素所占用空间大一倍的动态内存
auto p = alloc.allocate(vi.size()*2);
//通过拷贝vi中的元素来构造从p开始的元素
auto q=uninitialized_copy(vi.begin(),vi.end(),P);
//将剩余元素初始化为42
uninitialized_fill_n(q,vi.size(),42);
```

- `uninitialized_copy`算法：接受三个迭代器爹数。前两个表示输入序列，第三个表示这些元素将要拷贝到的目的空间。**目的位置迭代器必须指向未构造内存。** 
	- **返回值**：返回递增后的目的位置迭代器。
>与`copy`的区别，`uninitialized_copy`在给定目的位置构造元素
 ^4f020c
- `uninitialized_fill_n`算法：类似`fill_n`接受一个指向目的位置的指针、一个计数和一个值。
	- 作用：它会在目的位置指针指向的内存中创建给定数目个对象，用给定值对它们进初始化。
>`fill_n`参见[[#10.2.2 写容器元素的算法]]


## 12.3 使用标准库：文本查询程序
P430

# 第13章：拷贝控制
- **拷贝控制操作**：对象的拷贝、移动、赋值和销毁。
- **物种特殊的成员函数**：
	- 拷贝构造函数
	- 拷贝赋值运算符
	- 移动构造函数
	- 移动赋值运算符
	- 析构函数
>拷贝和移动构造函数定义了当用同类型的另一个对象初始化本对象时做什么。
>拷贝和移动赋值定义了将一个对象赋予同类型的另一个对象时做什么。
>析构函数定义了当此类型对象销时做什么


## 13.1 拷贝、赋值与销毁
### 13.1.1 拷贝构造函数
- **拷贝构造函数**定义：第一个参数是**自身类类型的引用**，切任何额外参数都有**默认值**的**构造函数**。
```c++
class Foo{
public:
	Foo();//默认构造函数
	Foo(const Foo&);//拷贝构造函数
}
```
>拷贝构造函数通常不应该是[[CPP#explicit：抑制构造函数定义的隐式转换|explicit]]

#### 合成拷贝构造函数
如果我们没有为一个类定义拷贝构造函数，编译器会为我们定义一个。
**即便我们定义了其他构造函数，编译器也会为我们合成一个拷贝构造函数。**
>但是不会为我们合成默认构造函数，需要显示指定。

合成的拷贝构造函数会将其参数逐个拷贝到正在创建的对象中：
- 对于类类型的成员，会使用其拷贝构造函数来拷贝
- 对于数组，合成拷贝构造函数会逐元素的拷贝一个数组类型的成员
- 内置成员则直接进行拷贝
- `static`成员：不进行拷贝，因为`static`成员一旦被赋值，那么它的值就是一个类所共享的。

#### 拷贝初始化
- **拷贝初始化**和**直接初始化**的区别：
	- 直接初始化：编译器使用普通的函数匹配调用构造函数
	- 拷贝初始化：编译器将右侧运算对象拷贝到正在创建的对象中，如果需要的话还要进行类型转换。通常是调用拷贝构造函数来完成。
>如果一个类有一个移动构造函数，则拷贝初始化**有时会使用移动构造函数**而非拷贝构造函数来完成。
```c++
string dots(10,',');//拷贝初始化
string s(dots);//拷贝初始化
string s2=s;//拷贝初始化
```

- 拷贝初始化一般使用`=`时发生，在下列情况也会发生：
	- 将一个对象作为实参传递给一个非引用类型的形参
	- 从一个返回类型为非引用类型的函数返回一个对象
	- 用花括号列表初始化一个数组中的元素或一个[[CPP#聚合类|聚合类]]中的成员。
>初始化标准容器库或者调用`insert`或者`push`时容器对元素进行拷贝初始化。用`emplace`成员创建的元素都进行直接初始化

[[CPP#直接初始化和拷贝初始化|拷贝初始化和直接初始化]]

#### 参数和返回值
- 在函数调用过程中，具有**非引用类型的参数要进行拷贝初始化**。
- 当一个函数具有非引用的返回类型时，返回值会被用来初始化调用方的结果。

为何拷贝构造函数的参数必须是引用类型？
- 如果不是其参数不是引用类型，则调用永远也不会成功．一一为了调用拷贝构造函数，我们必须拷贝它的实参，但为了拷贝实参，我们又需要调用拷贝构造函数，如此无限循环。

#### 拷贝初始化的限制
如果我们使用的初始化值要求通过一个`explicit`的构造函数来进行类型转换，那么使用拷贝初始化还是直接初始化就不是无关紧要：
```c++
vector<int> v1(10);//直接初始化
vector<int> v2=10;//错误，接受大小参数的构造函数是explicit的，所以10不能默认转换成大小为10的vector

void f(vector<int> v);//不是引用，参数将进行拷贝初始化
f(10);//错误，因为10默认转换成vector
f(vector<int>(10));//正确：从一个int直接构造一个临时vector
```
>[!note]
>从**函数返回**一个值和**实参的传递**是通过**拷贝初始化**的，调用`=`运算符，会进行**隐式转换**，不能隐式的使用`explicit`函数。

#### 编译器可以绕过拷贝构造函数
在拷贝初始化过程中，**编译器可以（但不是必须）跳过拷贝/移动构造函数**，直接创建对象。
```c++
string null_book="9-999-99999-9";//拷贝初始化
string null_book("9-999-99999-9");//编译器略过了拷贝构造函数
```
>拷贝/移动构造函数必须是存在且可访问的


### 13.1.2 拷贝赋值运算符
- 与拷贝构造函数一样，如果类未定义自己的拷贝赋值运算符，编译器会为它合成一个。

#### 重载赋值运算符
- **重载运算符**：重载运算符**本质上是函数**，其名字由`operator`关键字后接表示要定义的运算符的符号组成。因此，赋值运算符就是一个**名为`operator=`的函数**，有一个返回类型和一个参数列表。
	- **参数**：表示运算符的**运算对象**。
	- **返回值**：通常应该返回一个**指向其左侧运算对象的引用**

如果重载运算符是一个成员函数，那么其左侧对象就绑定到隐式的[[CPP#引入this|this]]参数。
```c++
class Foo{
public:
	Foo &operator=(const Foo&);//值运真符,左侧运算对象是this，右侧是函数的参数
}
```

#### 合成拷贝赋值运算符
如果一个类**未定义自己的拷贝赋值运算符**，**编译器**会为它生成一个**合成拷贝赋值运算符**。
- 作用：将右侧运算对象的每个非`static`成员赋予左侧运算对象的对应成员。对于数组成员，，逐个赋值数组元素。
>对于某些类，合成拷贝赋值运算符用来禁止该类型对象的赋值。

- 返回值：返回一个指向其左侧运算对象的引用。
![[Pasted image 20230201145001.png]]

### 13.1.3 析构函数
- **析构函数**与**构造函数**的区别：
	- **构造函数**：构造函数**初始化对象的非`static`数据成员**，可能还会有其他工作。
	- **析构函数**：析构函数**释放对象使用的资源**，并销毁对象的非`static`数据成员。

- **析构函数**形式：
```c++
class Foo{
Public:
	~Foo();//析构函数
};

```

- **重载**：由于析构函数不接受参数，因此它**不能被重载**。对一个给定类，只会有**唯一**一个析构函数。

#### 析构函数完成什么工作
- **构造函数**：成员的初始化是在函数体执行之前完成的，且**按照它们在类中出现的顺序进行初始化。**
- **析构函数**：首先执行函数体，然后销毁成员。**成员按初始化顺序的逆序销毁。** 通常，析构函数释放对象在生存期分配的所有资源。

成员销毁时发生什么完全依赖于成员的类型。销毁类类型的成员需要执行成员自己的析构函数。内置类型没有析构函数，因此销毁内置类型成员什么也不需要做。

>[!note]
>隐式销毁一个内置普通指针类型的成员不会`delete`它所指向的对象。智能指针是类类型，有析构函数，析构时会自动销毁。

#### 什么时候会调用析构函数
无论何时一个对象被销毁，就会**自动调用**其析构函数：
- 变量在离开其作用域时被销毁。
- 当一个对象被销毁时，其成员被销毁。
- 容器（无论是标准库容器还是数组）被销毁时，其元素被销毁。对于**动态分配**的对象，当对指向它的指针应用`delete`运算符时被销毁。
- 对于临时对象，当创建它的完整表达式结束时被销毁。

>[!note]
>当指向一个**对象的引用或指针**离开作用域时，析构函数不会执行。

#### 合成析构函数
当一个类未定义自己的析构函数时，编译器会为它定义一个**合成析构函数**。

- 类似拷贝构造函数和拷贝赋值运算符，对于**某些类**，**合成析构函数被用来阻止该类型的对象被销毁**，如果不是这种情况，那么析构函数的函数体为空。
![[Pasted image 20230201150845.png]]

>[!note]
>**析构函数体自身并不直接销毁成员**，成员是在析构函数体之后隐含的析构阶段中被销毁的。
>析构函数体和成员销毁是不同部分。


### 13.1.4 三/五法则
有三个基本操作可以控制类的拷贝操作，也叫**拷贝控制成员**：
- 拷贝构造函数
- 拷贝赋值运算符
- 析构函数
>新标准中还有移动构造函数和移动赋值运算符。

C++语言并不要求我们定义所有这些操作：可以只定义其中一个或两个，而不必定义所有操作。

#### 需要析构函数的类也需要拷贝和赋值操作
当我们决定一个类是否要定义它自己版本的**拷贝控制成员**时，一个基本原则是首先**确定**这个类**是否需要一个析构函数**。

- 一个需要**析构函数**的类，一般也需要一个**拷贝构造函数**和**一个拷贝赋值运算符**

- 一个小例子：
	- 一个类中包含动态内存成员指针，需要**重写析构函数**，在析构的时候释放动态内存。
	- 如果我们**拷贝**该类的一个对象，那么将会有两个指针指向相同的动态内存。在这两个对象析构的时候将会对`delete`两次！这是明显错误的。
具体的可以看书P447

>[!note]
>如果一个类需要**自定义析构函数**，几乎可以肯定它**也需要自定义拷贝斌值运算符和拷贝构造函数**。

#### 需要拷贝操作的类也需要赋值操作，反之亦然
- **拷贝控制成员**：拷贝、赋值、析构
虽然很多类需要定义所有（或是不需要定义任何）**拷贝控制成员**，但某些类所要完成
的工作，*只需要拷贝或赋值操作，不需要析构函数。*
>例如一个需要需要的类，每个对象序号不同，我们在自定义赋值操作，来避免将序号赋值。定义拷贝操作，来生成一个独一无二对象。

>[!note]
>拷贝和赋值和析构一般三者都需要或者都不用。
>**但是有拷贝一定要有赋值，有赋值也一定要有拷贝！**

#### 使用=default
- **构造函数**：如果我们重载了构造函数，把编译器将不提供默认合成版本，如果我们需要默认合成版本这需要显式调用`=default`
- **拷贝控制成员**：我们也可以通过可以通过将拷贝控制成员定义为`=defaul`**来显式地要求编译器生成合成的版本**
![[Pasted image 20230201153150.png]]
>调用`=defaul`会将合成的函数隐式声明为内联函数。如果**不需要内联**，那么我们应该只对成员的**类外定义使用**`=deault`


### 13.1.6 阻止拷贝
>[!note]
>大多数类应该**定义默认构造函数、拷贝构造函数和拷贝赋值运算符**，无论是隐
式地还是显式地。

`iostream`**类阻止了拷贝**，以避免多个对象写入或读取相同的IO。

如何阻止拷贝？
- 不定义拷贝控制操作？那样编译器也会自动合成默认版本，这是行不通的。

#### 定义删除的函数
- **删除的函数**：指特殊定义的*拷贝构造函数*和*拷贝赋值运算符*。虽然定义了但是不能使用。
 ^88d7e2
- 定义**删除函数**：在函数的参数列表后曲加上`=delete`，通知编译器，不定义这些成员。

- **删除函数作用**：用于**阻止拷贝**。也可以用于函数匹配过程中的引导。
![[Pasted image 20230201160042.png]]
![[Pasted image 20230201160048.png]]

- `=delete`与`=defaul`的区别：
	- `=delete`必须出现在函数第一次声明的时候。`=defaul`是知道编译器生成代码时才需要
	- 可以对任何函数指定`=delete`；只能对编译器可以合成的默认构造函数或拷贝控制成员使用`=defaul`。

#### 析构函数不能是删除的成员
- **不能删除析构函数**。如果析构函数被删除，就无法销毁此类型的对象了。类中一个成员的析构函数被删除，那么该成员也无法被销毁，对象整体也无法销毁。

- 对于**删除了析构函数**的类型：
	- 编译器将不允许定义该类型的变量或创建该类的临时对象。
	- 可以动态分配该类型的对象，但是**不能释放**。
![[Pasted image 20230201160939.png]]
>[!warning]
>对于析构函数已删除的类型，
>- 不能定义该类型的变量
>定义之后，无法销毁。
>- 不能释放指向该类型动态分配对象的指针。
> 释放`delete`是调用析构函数。

#### 合成的拷贝控制成员可能是删除的
对某些类来说，编译器将这些合成的成员定义为删除的函数：
- 如果类的某个成员的**析构函数**是删除的或不可访问的（例如，是private的），则类的**合成析构函数**被定义为删除的。
- 如果类的某个成员的**拷贝构造函**数是删除的或不可访问的，则类的合成拷贝构造函数被定义为删除的。 如果类的某个成员的**析构函数**是删除的或不可访问的，则类**合成的拷贝构造函数**也被定义为删除的。
- 如果类的某个成员的**拷贝赋值运算符**是删除的或不可访问的，或是类有一个`const`的或引用成员，则类的**合成拷贝赋值运算符**被定义为删除的。
- 如果类的某个成员的**析构函数**是删除的或不可访问的，或是类有一个引用成员，它没有类内初始化器，或是类有一个`const`成员，它没有*类内初始化器*且其类型未显式定义默认构造函数，则该类的**默认构造函数**被定义为删除的。


本质上，这些规则的含义是：**如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则对应的成员函数将被定义为删除的。**
- 一个成员有删除的或不可访问的析构函数会导致合成的默认拷贝构造函数被定义删除的。为了防止创建的对象无法销毁。
- 对于具有引用成员或无法默认构造的`const`成员的类，编译器不会为其合成默认构造函数。
- 一个类有`const`成员，那么不能使用合成的拷贝赋值运算符。因为把一个新值赋值给`const`是不可能。
- 一个有引用成员的类，合成拷贝赋值运算符被定义为删除的。因为赋值无法改变引用的对象，只能改变引用对象的值，赋值之后对象仍然保持不变。

>[!note]
>**本质上，当不可能拷贝、赋值或销毁类的成员时，类的合成拷贝控制成员就被定义为删除的**

#### private拷贝控制
在新标准发布之前，类是通过将其拷贝构造函数和拷贝赋值运算符声明为`private`的来阻止拷贝。

>[!note]
>希望阻止拷贝的类应该使用：`delete`来定义它们自己的拷贝构造函数和拷贝赋值运算符，而不应该将它们声明为`private`的。例如 `iostream`对象


## 13.2 拷贝控制和资源管理
通常，管理类外资源的类必须定义拷贝控制成员。**一旦一个类需要析构函数，那么它几乎肯定也需要一个拷贝构造函数和一个拷贝赋值运算符。**

- 拷贝语义：一般来说，有两种选择：
	- 使类行像一个值：副本和原对象是**独立**的。
	- 使类行为像一个指针。副本和原对象共享底层数据，改变副本会改变原来对象。
>- 内置类型（除了指针）标准库容器和`string`类的行为像一个值。
>- `shared_ptr`类提供类似指针的行为。
>- `IO`和`unique_ptr`不允许拷贝和赋值，它们的行为不像值也不像指针。

### 13.2.1 行为像值的类
每个对象都应该拥有一份自己的拷贝。
![[Pasted image 20230201165150.png]]

#### 类值拷贝赋值运算符
**赋值运算符**通常组合了析构函数和构造函数的操作。
- 类似析构函数，赋值操作会销毁左侧运算对象的资源。
- 类似拷贝构造函数，赋值操作会从右侧运算对象拷贝数据。
>通过先拷贝右侧运算对象，再对左侧对象进行销毁。即使将一个对象赋予它自身，也保证正确。
```c++
HasPtr& HasPtr::operator=(const HasPtr &rhs)
{
	auto newp = new string(*rhs.ps);//拷贝底层string
	delete ps;//释放旧内存。
	ps = newp;//从右侧运算对象拷贝数据到本对象
	i = rhs.i;
	return *this;//返回本对象
}
```

>[!note]
>当你编写一个赋值运算符时，先将右侧对象保存到一个局部变量中，在进行销毁左侧对象，最后再将临时对象拷贝到左侧运算对象成员中。
>当你编写赋值运算符时，有两点需要记住：
>- 如果将一个对象赋予它自身，赋值运算符必须能正确工作。
>- 大多数赋值运算符组合了析构函数和拷贝构造函数的工作。


### 13.2.2 定义行为像指针的类
对于行为类似指针的类，我们需要为其定义**拷贝构造函数**和**拷贝赋值运算符**，来拷贝指针成员本身而不是它指向的`string`。

- 使用`shared_ptr`：使用用`shared_ptr`来管理类中的资源。拷贝（或赋值）一个`shared_ptr`会拷贝（赋值）`shared_ptr`所指向的指针。当没有用户使用对象时，`shared_ptr`类负责释放资源。（最好的方法）

- 不使用`shared_ptr`，我们也可以使用**引用计数**，直接管理资源。

#### 引用计数
- **引用计数**的工作方式：
- 除了初始化对象外，每个构造函数（拷贝构造函数除外）还要创建一个引用计数，用来记录有多少对象与正在创建的对象共享状态。当我们创建一个对象时，只有一个对象共享状态，因此将计数器初始化为0
- 拷贝构造函数不分配新的计数器，而是拷贝给定对象的数据成员，包括计数器。拷贝构造函数递增共享的计数器，指出给定对象的状态又被一个新用户所共享。
- 析构函数递减计数器，指出共享状态的用户少了一个。如果计数器变为0，则析构函数释放状态。
- 拷贝赋值运算符递增右侧运算对象的计数器，递减左侧运算对象的计数器。如果左侧运算对象的计数器变为0，意味着它的共享状态没有用户了，拷贝赋值运算符就必须销毁状态。

- **计数器**的保存：
，计数器保存在动态内存中。当创建一个对象时，我们也分配一个新的计数器。当拷贝或赋值对象时，我们拷贝指向计数器的指针。

#### 定义一个使用引用计数的类
![[Pasted image 20230201172641.png]]
- `use`数据成员：记录有多少对象共享相同的`string`。接受`string`参数的构造函数分配新的计数器，初始化为1，指出当前有一个用户使用本对象的`string`成员。

#### 类指针的拷贝成员“篡改引用计数
当拷贝一个`HasPtr`时，我们将拷贝`ps`本身，而不是`ps`指向的`string`。这使得两个对象共用`string`。当我们进行拷贝时，还会递增该`string`关联的计数器。

- **析构函数**不能无条件地`delete`，因为可能有多个对象指向这块内存。析构函数应该先递减引用计数，把计数器变成0才可以`delete`
![[Pasted image 20230201173356.png]]

- **拷贝赋值运算符**与往常一样执行类似拷贝构造函数和析构函数的工作。
	- 类似拷贝构造函数：赋值运算符必须递增右侧运算对象的引用计数。
	- 类似析构函数：递减左侧运算对象的引用计数，在必要时释放使用的内存。

>为了保证自赋值正常，先递增右侧对象的引用计数，再判断递减左侧对象计数是否为0，0就销毁。
![[Pasted image 20230201174040.png]]
![[Pasted image 20230201174054.png]]
>这里一开始对右侧对象递增计数值，一是左右对象相同时，为了保证自赋值正常进行；二是即便左右对象不同，我们再赋值之后也需要递增右侧对象的计数值。


## 13.3 交换操作
除了定义**拷贝控制成员**，管理资源的类通常还定义一个名为`swap`的函数。
>拷贝控制成员一共有三个，不记得看前面几节去

- 对于使用[[#10.2.3 重排容器元素的算法|重排元素算法]]的类，定义`swap`是非常重要的。因为这类算法在需要交换两个元素时会调用`swap`。

- 如果一个类定义了自己的`swap`，那么算法将使用类自定义版本。否则，算法将使用标准库的`swap`。

- 标准库`swap`的实现：
![[Pasted image 20230201180005.png]]
>`v1`中的`string`拷贝了两次，`v2`的`string`拷贝了一次，这样存在不必要的内存分配。

- 交换指针的`swap`：
![[Pasted image 20230201175204.png]]

#### 编写我们自己的swap函数
可以在我们的类上定义一个自己版本的`swap`来重载`swap`的默认行为
![[Pasted image 20230201175253.png]]

>[!note]
>与拷贝控制成员不同，`swap`并不是必要的。但是，对于分配了资源的类，定义`swap`可能是一种很重要的**优化手段**。

#### swap函数应该调用swap，而不是std::swap
![[Pasted image 20230201175253.png]]
`swap`函数中调用的`swap`不是`std::swap`
- 对于没有特定版本`swap`的内置成员，调用`swap`就是调用`std::swap`
- 对于有自己定义的`swap`函数的类，调用`std`会先查找是否存在特定类型的`swap`，其匹配程度会优于`std`中定义的版本。

- 为什么`using::swap`声明没有隐藏特定版本的`swap`?
[[#18.2.3]]节会说

#### 在赋值运算符中使用swap
定义`swap`的类通常用`swap`来定义它们的赋值运算符。这些运算符使用了一种名为**拷贝并交换**的技术。

- **拷贝并交换**：将**左侧**运算对象与**右侧**运算对象的一个**副本**进行**交换**。
![[Pasted image 20230201180728.png]]
- **参数并不是一个引用**，我们将右侧运算对象以传值方式传递给赋值运算符，因此`rhs`**是右侧对象的一个拷贝**。
- 调用`swap`：将左侧运算对象中原来保存的指针存入`rhs`中，并将`rhs`中原来的指针存入`*this`。因此交换之后`*this`中指针成员将指向（上一步中）新分配的`string`，即右侧运算对象中`string`的一个副本
- 当赋值运算符结束时，`rhs`被销毁，`HasPtr`的析构函数将执行。此析构函数`delete` `rhs`现在的内存，即释放掉左侧对象原来的内存。

这个技术的有趣之处是它**自动处理了自赋值情况**且天然就是异常安全的。它**通过在改变左侧运算对象之前拷贝右侧运算对象**（此例子中是`rhs`）保证了自赋值的正确。
>在拷贝右侧对象，调用拷贝构造函数`new`的时候可能会出错，但是这个发生在交换之前。

>[!note]
>使用**拷贝和交换**的赋值运算符自动就是异常安全的，且能**正确处理自赋值**。


## 13.4 拷贝控制示例

- `Message`类：分别表示电子邮件
- `Folder`类：表示消息目录
- 两者关系：`Message`可以出现在多个Floder中。但是任意给定的`Message`的内容只有一个副本。
- `Message`类`set`指针：记录`Message`保存在哪些`Folder`中。
- `Folder`类`set`指针：记录`Folder`包含的`set`
![[Pasted image 20230202154122.png]]

这是一个例子在P460，大概就是一个`Message`包含多个 `Folder`，而一个 `Folder`也包含多个 `Folder`。

## 13.5 动态内存管理类
- 某些类需要在**运行时分配可变大小的内存**空间。
- 这种类通常可以**使用标准库容器**来保存它们的数据。
- 对于某些需要自己及逆行你内存分配的类，一般必须定义自己拷贝控制成员来管理所分配的内存。

#### Str类
`Str`类实现标准库`vector`类的一个简化版本，无需模板，只能用于`string`。

- 类似于[[CPP#vector对象是如何增长的|vector类的增长]]，我们使用一个[[#allocator类]]来获取原始内存。由于`allocator`分配的内存是未构造的。
	- 在需要添加新元素时用`allocator`的`construct`成员在原始内存中创建对象。
	- 在删除元素时，我们使用`destory`来销毁元素。

- `StrVec`的成员：
	- `elements`：指向分配的内存中的首元素
	- `first_free`：指向最后一个实际元素之后的位置
	- `cap`：指向分配内存末尾之后的位置。
	- `alloc`：静态成员，类型为`allocator<string>`。用于分配`strVec`使用的内存
![[Pasted image 20230202161615.png]]

- `StrVec`的函数：
	- `alloc_n_copy`：会分配内存，并拷贝一个给定范围中的元素。
	- `free`：销毁构造元素并释放内存
	- `chk_n_alloc`：调用`reallocate`来分配更多内存
	- `reallocate`：在内存用完时为`StrVec`分配新内存。

#### StrVec类定义
![[Pasted image 20230202161942.png]]
![[Pasted image 20230202161953.png]]
![[Pasted image 20230202162200.png]]

#### 使用construct
函数`push_back`调用`chk_n_alloc`确保有空间容纳新元素。
![[Pasted image 20230202162326.png]]
- `chk_n_alloc`：有需要时会调用`reallocate`来分配更多内存。
- `construct`：第一个参数必须是一个指向调用`allocate`所分配的未构造的内存空间。
	- 调用之后会递增`first_free`，表示已经构造了一个新元素。
	- 递增是[[CPP#^e11b24|前置递增]]。

#### alloc_n_copy成员
我们在拷贝或赋值`StrVec`时，可能会调用`alloc_n_copy`成员。
- `alloc_n_copy`成员:会分配内存，并拷贝一个给定范围中的元素。返回一个指针`pair`
	- 第一个指针：指向新空间开始位置
	- 第二个指针：指向拷贝的尾后位置
![[Pasted image 20230202163249.png]]
- `uninitialized_copy`函数：[[#^4f020c|uninitialized_copy]]返回指向最后一个构造元素之后位置的指针。

#### free成员
- `free`成员：
	- 首先`destroy`元素，然后释放`StrVec`自己分配的内存空间。
	- `for`循环调用`allocator`的`destory`成员，从构造的尾元素开始，到首元素为止，**逆序销毁所有元素**。
>释放动态数组`delete []`也是**逆序销毁元素。**
![[Pasted image 20230202163654.png]]
- **销毁**元素：`destory`释放内存空间上构造的元素所占用的空间
	- `destory`函数：运行`string`的析构函数。析构函数会释放`string`自己分配的内存空间
- **释放**空间：把`allocato`分配的空间释放给回自由空间。
	- `deallocate`函数：释放本`StrVec`对象分配的内存空间。`deallocate`的指针必须是之前某次`allocate`调用所返回的指针。

#### 拷贝控制成员
- 拷贝构造函数：调用`alloc_n_copy`	
	- `alloc_n_copy`：会分配内存，并拷贝一个给定范围中的元素。
![[Pasted image 20230202164441.png]]
由于`allocn_copy`分配的空间恰好容纳给定的元素，`cap`也指向最后一个构造的元素之后
的位置。

- 析构函数：调用`free`
	- 首先`destroy`元素，然后释放`StrVec`自己分配的内存空间。
	- `for`循环调用`allocator`的`destory`成员，从构造的尾元素开始，到首元素为止，逆序销毁所有元素。
![[Pasted image 20230202164510.png]]

- 拷贝赋值运算符：释放己有元素之前调用`alloc_n_copy`，这样就可以**正确处理自赋值**
	- `alloc_n_copy`：会分配内存，并拷贝一个给定范围中的元素。
![[Pasted image 20230202164711.png]]
>[!note]
>为何能够正确处理**自赋值**？
>因为左值和右值可能相等，我们在free掉左值的时候有可能free右值，导致程序出错。但是这里我们在free之前调用了`alloc_n_copy`为右值分配空间保存，所以对就算是自赋值也没影响。

#### 在重新分配内存的过程中移动而不是拷贝元素
- `reallocate`：在内存用完时为`StrVec`分配新内存。
	- 为一个新的更大的`string`数组分配内存
	- 在内存空间的前一部分构造对象保存现有元素
	- **销毁原内存空间中的元素，并释放这块内存。**
观察这个操作步骤，我们可以看出，为一个`StrVec`重新分配内存空间会引起从旧内存空间逐个拷贝`string`。拷贝之后原来的`string`空间也会被销毁，相当于**将原来的`string`移动到新的内存空间**去，那么我们**拷贝这些`string`中的数据**是多余的。在重新分配内存空间时，如果我们能避免分配和释放`string`的额外开销，`StrVec`的性能会好得多。

#### 移动构造函数和std::move
通过使用新标准库引入的**两种机制**，我们就可以**避免`string`的拷贝**。


- 新标准库提供的**两种机制**：
	- 移动构造函数：移动构造函数通常是将资源从给定对象**“移动”而不是拷贝** 到正在创建的对象。 而且我们知道知道标准库保证“移后源”`string`仍然保持一个有效的、可析构的状态。
	- `move`：标准库函数，定义在头文件utility头文件中。
>对于`string`，我们可以想象每个`string`都有一个指向`char`数组的指针。可以假定`string`的**移动构造函数**进行了**指针的拷贝**，而不是为字符分配内存空间然后拷贝字符。

- 有关`move`函数：
1. 当`reallocate`在新内存中构构造`string`时，必须调用`move`来表示希望使用`string`的**移动构造函数**，如果不使用`move`将会使用`string`的拷贝构造函数。
2. `move`的使用通常不用`using`声明，而是直接使用`std::move`

>[!FAQ] 为何要避免拷贝？
>因为拷贝之后会有销毁原空间的操作，但是我们在重新分配内存的时候实际上只需要把数据**移动**到更大的内存空间而已。

#### reallocate成员
![[Pasted image 20230202170901.png]]
- 每次重新配内存时都会将`StrVec`的容量加倍。如果为空则分配容纳一个元素的空间。
- 调用`move`返回的结果会令`construct`使用`string`的**移动构造函数**。这些`string`管理的内存将不会被拷贝。相反，我们构造的每个`string`都会从`elem`指向的`string`那里接管内存的所有权。

>但是这里移动之后也要进行原空间的释放，我们拷贝的话申请空间然后把元素直接拷贝到新空间，然后释放原空间。好像没什么差别？
如果对象较大，或者是对象本身要求分配内存空间（如`string`)，行不必要的拷贝代价非常高。

>[!FAQ] 怎么理解`string`对象本身需要分配内存空间？
>我们在[[#free成员]]中可以看到，`free`的时候会消除每个元素分配的内存空间，之后再释放类占有的内存空间，也就是所在类分配了内存空间之后，`string`对象在此基础上也占有了该类的空间。
所以如果进行拷贝的话，在拷贝的时候会为`string`在类空间的基础上重新分配内存空间。





## 13.6 对象移动
新标准的一个最主要的特性是**可以移动而非拷贝对象**的能力。
>某些类不能被拷贝但是可以移动，例如IO类，或者`unnique_ptr`类别。`unnique_ptr`在特殊情况可以拷贝。

移动操作，对容器的影响：
- 在旧版本的标准库中，容器中所保存的类必须是可拷贝的。
- 在新版本中，我们可以用容器保存不可拷贝的类型，只要它们能够移动即可。

>[!note]
>标准库容器、`string`和`shared_ptr`类既支持移动也支持拷贝。
>IO类和`unique_ptr`类**可以移动但不能拷贝**

### 13.6.1 右值引用
为了支持移动操作，新标准引入了一种新的引用类型一一一**右值引用**.

- **右值引用**：所谓右值引用就是必须绑定到右值的引用。通过`&&`获取
- **右值引用的性质**：只能绑定到一个**将要销毁的对象**。

- 左值和右值是表达式的属性。
	- 一个左值表达式表示的是一个对象的身份。
	- 一个右值表达式表示的是对象的值。

- 常规引用我们称为**左值引用**，**右值引用**跟左值引用的特性完全相反。
	- 可以其绑定到要求转换的表达式、字面值常量或者是返回右值的表达式。
```c++
int i=42;
int &r = i;//正确：r引用i
int &&rr = i;//错误：不能将一个右值引用绑定到一个左值上
int &r2 = i*42;//错误：42是一个右值
const int &r3 = i*42;//正确：我们可以将一个const的引用绑定到一个右值上
int &&rr2 = i*42;//正确：将rr2绑定到乘法结果上
```

#### 左值持久：右值短暂

- 左值和右值的区别：
	- 左值有持久状态
	- 右值要么是字面常量，要么是在表达式求值过程中创建的临时对象。

- **右值引用的特点**：
	- 引用的对象将要被销毁
	- 该对象没有其他用户

>[!note]
>使用右值引用的代码可以自由地接管所引用的对象的资源。

#### 变量是左值
变量是左值，声明为右值引用类型的变量也是左值，因此不能被右值引用。

```c++
int &&rr1 = 42;//正确：字面常量是右值
int &&rr2=rr1;//错误：表达式rrl是左值！
```
>[!note]
>变量是左值，因此我们不能将一个右值引用直接绑定到一个变量上，即使这个变量是右值引用类型也不行。

#### 标准库move函数
虽然不能将一个右值引用直接绑定到一个左值上，但我们可以**显式地将一个左值转换为对应的右值引用类型。**

- `move`函数：**获取绑定到左值上的右值引用**。定义在头文件utility中。
```c++
int &&rr1 = 42;//rr1是一个左值，类型是右值引用
int &&rr3 = std::move(rr1);//ok
```
调用`move`就意味着承诺：**除了对`rr1`赋值或销毁它外，我们将不再使用它。** 在调用`move`之后，我们不能对移后源对象的值做任何假设。
>调用`move`就相当于调用源对象的**移动构造函数**?

>[!note]
>- **移后源对象：** 调用`move`函数转换后的原左值对象。
>- 我们可以销毁一个移后源对象，也可以赋予它新值，但不能使用一个移后源对象的值。
>- 使用`move`不提供`using`声明，而是直接调用`std::move`，可以避免潜在的名字冲突


### 13.6.2 移动构造函数和移动赋值运算符
- 为类定义移动构造函数和移动赋值运算符使得类支持移动操作。
- **移动构造函数**：第一个参数是该类类型的一个右值引用，额外的超速都需要有默认实参。
- **移动构造函数的作用**：完成资源的移动，同时保证移后源对象处于销毁无害状态。特别是，一旦资源完成移动，源对象必须不再指向被移动的资源，资源的所有权已经归属新创建的对象。
```c++
//移动构造函数，第一个参数是该类型的右值引用
StrVec::StrVec(StrVec&&s) noexcepect//移动操作不应抛出任何异常
:elements(s.elements),first_free(s.first_free),cap(s.cap){
	//令s进入这样的状态对其运行析构函数是安全的
	s.elements = s.first_free=s.cap=nullptr;
}
```
- 拷贝构造函数与移动构造函数的**区别**：
	- 移动构造函数**不分配任何新内存**，只是接管内存之后将来给定对象中的指针都设置为`nulptr`

- `noexcepect`：承诺函数不抛出异常的一种方法。在函数的列表之后指定，在构造函数的参数列表和初始化列表开始的冒号之间。
![[Pasted image 20230203144136.png]]
![[Pasted image 20230203144142.png]]

>[!note]
>不抛出异常的移动构造函数和移动赋值运算符必须标记为`noexcept`。

- 移动构造过程中可能会发生异常使得旧空间中的元素已经被改变而形空间中未构造的元素可能尚不存在。
- 如果是在使用拷贝构造过程中发生的异常则只需要释放新分配的空间，旧空间中的元素仍然存在。
- 除非`vector`知道元素类型的移动构造函数不会抛出异常否则在重新分配内存的过程中，必须使用拷贝构造函数而不是移动构造函数，如果需要**指定**使用移动构造函数进行自定义类型对象的**移动**而不是拷贝，我们这需要将移动构造函数及移动赋值运算符**标记为`noexcept`**。
>[!note]
>`vector`进行`push_back`的时候会有重新分配内存空间的情况发生，此时默认使用拷贝构造函数，除非能够确认移动构造函数不会发生异常，才会使用移动构造函数进行移动。

#### 移动赋值运算符
- 移动赋值运算符执行与析构函数和移动构造函数**相同的工作**。
- 与移动构造函数一样，如果我们的移动赋值运算符不抛出任何异常，我们就应该将它标记为`noexcept`。
- 移动赋值运算符必须正确处理自赋值
![[Pasted image 20230203145156.png]]
![[Pasted image 20230203145203.png]]
- 首先判断`this`和`rhs`的地址是否相同
	- 如果地址相同，那么什么都不做
	- 如果不同，那么我们需要释放`this`的内存，并接管`rhs`对象的内存，并且将`rhs`中的指针置为`nullptr`
>如果不把`rhs`中的指针置为`nullptr`，那么`rhs`在销毁过程中会释放左侧内存对象接管的内存空间。

>[!FAQ] 为何移动赋值运算符也要处理自赋值问题
>如上代码，如果不处理左右对象是否相同，就进行释放左对象，就会把右侧对象的内存空间也给释放掉。

#### 移后源对象必须可析构
`StrVec`的移动操作满足这一要求，这是通过将移后源对象的指针成员置为`nullptr`来实现**移后源对**象可析构。

- 移后源对象除了是**可析构**状态之外还必须是**有效的**对象。

- **有效对象**：指可以安全地为其赋予新值或者可以安全地使用而不依赖其当前值。

我们的`StrVec`类的移动操作将移后源对象置于与默认初始化的对象相同的状态。因此，我们可以继续对移后源对象执行所有的`StrVec`操作，与任何其他默认初始化的对象一样。而其他内部结构更为复杂的类，可能表现出完全不同的行为。

>[!note]
>在移动操作之后，移后源对象必须保持有效的、可析构的状态

#### 合成的移动操作
- 对于拷贝构造函数和拷贝赋值运算符，如果我们不声明自己的，编译器会为我们合成。

- **拷贝操作**要么被定义为逐成员拷贝，要么被定义为对象赋值，要么被定义为删除的函数。

- **合成移动操作**：编译器根本不会为某些类合成移动操作。
	- 如果一个类定义了自己的拷贝构造函数、拷贝赋值运算符或析构函数，编译器则不会合成移动构造函数和移动赋值运算符。
	- 当一个类**没有定义任何自己版本的拷贝控制成员**，且类的每个非`static`数据成员都可以移动时，编译器才会**合成拷贝移动操作**。
>如果一个类没有移动操作，通过正常的函数匹配，类会使用对应的拷贝操作来代替移动操作。

>[!FAQ] 何为可移动成员？
>内置类型成员和有对应移动操作的类，都是可移动成员

![[Pasted image 20230203151421.png]]

>[!note]
>只有当一个类**没有定义**任何自己版本的**拷贝控制成员**，且它的**所有数据成员**都能移动构造或移动赋值时，编译器才会为它**合成移动构造函数或移动赋值运算符**。

- 与拷贝操作不同，**移动操作永远不会隐式定义为删除的函数**。**但是**，如果我们显式地要求编译器生成`=default`的移动操作，且编译器不能移动所有成员，则编译器会将移动操作定义为**删除的函数**。
>在某些情况下，拷贝控制操作会被默认合称为[[#^88d7e2|删除的函数]]

- 除了**一个重要的例外**（上面的但是），什么时候将合成的移动操作定义为删除的函数遵循与定义删除的合成拷贝操作类似的原则：
1. 与**拷贝构造函数不同**，移动构造函数被定义为删除的函数的条件是：有类**成员定义了自己的拷贝构造函数且未定义移动构造函数**，或者是**有类成员未定义自己的拷贝构造函数且编译器不能为其合成移动构造函数**。移动赋值运算符的情况类似。
2. 如果**成员**的移动构造函数或移动赋值运算符被定义为**删除的**或是不可访问的，则**类**的移动构造函数或移动赋值运算符被定义为**删除的**。
3. 类似拷贝构造函数，如果**类的析构函数被定义为删除的或不可访问的**，则类的**移动构造函数被定义为删除的**。
4. 类似拷贝赋值运算符，如果有**类成员**是 **`const`的或是引用**，则类的移动赋值运算符被定义为删除的。

- 一个例子：Y是一个类，定义了自己的拷贝构造函数但未定义自己的移动构造函数
![[Pasted image 20230203154647.png]]
- `hashY`的成员`Y`有一个删除的移动构造函数，所以`hashY`默认生成的移动构造函数也是删除的。

- **拷贝操作**和**合成的拷贝控制成员**之间的**相互作用关系**：一个类是否定义了自己的移动操作对拷贝操作如何合成有影响。
	- 如果**类定义了一个移动构造函数**和/或一个**移动赋值运算符**，则该类的**合成拷贝构造函数和拷贝赋值运算符会被定义为删除的。**

>[!note]
>**定义了一个移动构造函数或移动赋值运算符的类必须也定义自己的拷贝操作**。否则，这些成员默认地被定义为删除的。

#### 移动右值，拷贝左值
如果一个类**既有移动构造函数**，**也有拷贝构造函数**，编译器使用普通的[[CPP#函数匹配|函数匹配规则]]来确定使用哪个构造函数，赋值操作的情况类似。

- 例子：StrVec类：
	- 拷贝构造函数：接受一个`const StrVec`的引用，可用于任何可以可以转换为`StrVec`的类型
	- 移动构造函数：接受一个` StrVec&&`，因此只能用于实参是非`static`右值的情形。
![[Pasted image 20230203160601.png]]
- 第一个赋值语句：`v2`的类型是`StrVec`，表达式v2是一个左值。
- 第二个赋值语句：我们赋予`v2`的是`getVec`调用的结果。此表达式是一个右值，类型`getVec`。为对于该情况两个赋值运算符都是可行的
	- 对于拷贝赋值：需要进行一次到`const`的转换。
	- 移动赋值：而因为其是右值，参数`StrVec&&`是精确匹配。因此会**调用移动赋值运算符**

#### 但如果没有移动构造函数，右值也被拷贝
如果一个类有一个**拷贝构造函数**但**未定义移动构造函数**，编译器**不会**为其**合成**移动构造函数。
![[Pasted image 20230203161159.png]]
- 调用了`std:move`返回一个绑定到x的`Foo&&`，但是因为没有拷贝构造函数，则将该`Foo`类型的右值引用转换为 `const`然后调用拷贝构造函数。

>[!note]
>- 用拷贝构造函数代替移动构造函数几乎肯定是安全的（赋值运算符的情况类似），如果一个类**有**一个可用的**拷贝构造函数**而**没有移动构造函数**，则其对象是**通过拷贝构造函数来“移动的”**。
>- 拷贝构造函数满足对应移动构造函数的要求：
>	- 它会拷贝给定对象，并将原对象置于有效状态，实际上，拷贝都不会改变原对象的值。

#### 拷贝并交换赋值运算符和移动操作
![[Pasted image 20230203161805.png]]
- 单一的赋值运算符就实现了**拷贝赋值**运算符和**移动赋值**运算符两种功能。
![[Pasted image 20230203162221.png]]
- 赋值运算符的第一个参数是非应用对象，那么使用赋值运算符时候需要进行拷贝初始化，拷贝初始化可以使用拷贝构造函数或者移动构造函数。
	- 使用拷贝构造函数：第一个赋值。因为右边是一个左值，移动构造函数不可行`rhs`将使用拷贝构造函数来初始化。
	- 使用移动构造函数：第二个赋值。我们调用`std::move`返回一个右值引用绑定到`hp2`上。因为右边是一个右值引用，拷贝初始化时，调用移动构造函数是精确匹配。移动构造函数从`hp2`拷贝指针，而不会分配任何内存。

>[!note]
>函数的实参不是引用类型那么在调用函数时候就需要进行参数的拷贝初始化。而拷贝初始化可以使用拷贝构造函数和移动构造函数，主要看函数匹配的精确度，使用匹配精确度高的那个。

>[!FAQ] 为什么在赋值的时候还要拷贝初始化？
>因为使用赋值运算符就是调用了类中我们定义的`operator=`函数，而在该例子中形参不是引用类型，在调用函数之前就需要进行实参的拷贝初始化。

>[!note]
>更新[[#13.1.4 三/五法则|三/五法则]]，一般来说，**一个类定义任何一个拷贝操作就应该定义所有五个操作**！
>某些类必须有拷贝控制成员才能正确工作，这些类通常有一个资源，而拷贝成员必须拷贝此资源，可能会导致一些额外的开销，定义了移动构造函数和移动赋值运算符就可以避免此问题。

#### Message类的移动操作
`Message`类可以使用`string`和`set`的移动操作来避免拷贝`contents`和`folders`成员的额外开销。
>`string`和`set`均有移动赋值运算符和移动构造函数

除了移动`folders`成员，我们还必须更新每个指向原`Message`的`Folder`。必须删除日向旧`Message`的指针，并添加一个指向新`Message`的指针。
- 定义一个更新`Folder`指针的操作：将参数的`folders`中的自身（`m`）替换成`this`
![[Pasted image 20230203165039.png]]
 >通过调用`move`，我们使用了`set`的移动赋值运算符而不是它的拷贝运算符，调用之后原来旧对象指向右侧对象的资源，右侧对象的指针置空。
 
 - 定义`Message`的移动构造函数和移动赋值运算符：向容器添加元素可能会抛出`bad_alloc`异常，因此不标记为`noexcept`
 - 定义`Message`的移动构造函数：
![[Pasted image 20230203165545.png]]
	- 调用`move_Folders`来删除指向`m`的指针并插入本`Message`的指针。

- 定义`Message`的移动赋值运算符：
![[Pasted image 20230203165900.png]]
	- 移动赋值运算符必须销毁左侧运算对象的旧状态，即从现有`folders`中删除指向本`Message`的指针，调用`remove_from_Folders`完成。
	- 调用`move`从`rhs`将`contents`移动到`this`对象。
	- 调用`move_Messages`来更新Folder指针。

#### 移动迭代器
`StrVec`的[[#reallocate成员]]使用了一个`for`循环来调用`construct`从就内存将元素拷贝到新内存，我们尝试使用`uninitialized_copy`来分配新内存，但是`allocator`分配的内存是未构造的，`uninitialized_copy`会使用源迭代器的解引用操作，调用`construct`操作，将元素**拷贝**到未构造的内存中。
>未购找的内存不支持拷贝操作？

- **移动迭代器**适配器：一个移动迭代器通过**改变**给定迭代器的**解引用运算**符的行为来适配此迭代器。与其他迭代器不同，移动迭代器**解引用**运算符**生成**一个**右值引用**。

- `make_move_iterator`函数：一个普通迭代器转换为一个**移动迭代器**。
	- **参数**：一个迭代器参数
	- **返回值**：一个移动迭代器

- 原迭代器转换为移动迭代器之后，原迭代器的其他操作在移动迭代器中都照常工作。
![[Pasted image 20230203171643.png]]
![[Pasted image 20230203171654.png]]
- `uninitialized_copy`对输入序列中的每个元素调用`construct`来将元素“拷贝”到目的位置。**此算法使用迭代器的解引用运算符**从输入序列中提取元素。由于我们传递给它的是**移动迭代器**，解引用生成的是右值引用，这意味着`construct`将使用**移动构造函数**来构造元素。
>为什么`construct`可以使用移动构造函数来构造元素？
>**`construct`函数使用其第二个和随后的实参的类型来确定使用元素的哪个构造函数**
>也就是如果是右值引用则调用元素的移动构造函数，是左值就调用拷贝构造函数。

>[!note]
>标准库不保证哪些算法适用移动迭代器，**使用迭代器会调用移动构造函数**，可能会销毁源对象，所以只有在确信算法在为一个元素赋值或将其传递给一个用户定义的函数后不再访问时，才将移动迭代器传给算法。

>[!warning]
>**移动构造函数**和**移动赋值运算符**这些类实现代码之外的地方，只有当你确信需要进行移动操作且移动操作是安全的，才可以使用std::move。


### 13.6.3 右值引用和成员函数
- 一个**成员函数**也允许同时提供拷贝和移动版本。
	- **移动**版本：指向一个非`const`的**右值引用**
	- **拷贝**版本：指向一个`const`的**左值引用**

- 例子：定义了`push_back`的标准库容器提供了拷贝和移动版本。
	- 第一个版本：可以将能转换为类型`X`的任何对象传递给第一个版本的`push_back`。
	- 第二个版本：**只**可以传递给它非`const`的**右值**。此版本对于非`const`的右值是**精确匹配**，当我们传递一个可修改的右值，编译器会选择运行这个版本。
```c++
void push_back(const X&);//拷贝：绑定到任意类型的x
void push_back(X&&);//移动：只能绑定到类型x的可修改的右值
```

>[!note]
>区分移动和拷贝的重载函数通常有一个版本接受一个`const T&`，而另一个版本接受一个`T&&`。
>通常不定义`const T&&`或`T&`，因为前者右值传递，必须是可以改变数据的，不能是`const`，后者拷贝操作不能改变源对象，所以必须要`const`。

- 一个例子：`StrVec`类定义另一个版本的`push_back`。
![[Pasted image 20230203175127.png]]
- 差别：差别在于右值引用版本调用`move`来将其参数传递给`construct`。由于`move`返回一个**右值引用**，传递给`construct`的实参类型是`string&&`。因此，会使用`string`的移动构造函数来构造新元素。
>`construct`函数使用其第二个和随后的实参的类型来确定使用元素的哪个构造函数

- 当我们调用`push_back`时，**实参类型决定了新元素是拷贝还是移动到容器中**
![[Pasted image 20230203175448.png]]
![[Pasted image 20230203175502.png]]
* 这些调用的差别在于实参是一个左值还是一个右值（从`"done"`创建的临时`string`)，具体调用哪个版本据此来决定。

#### 右值和左值引用成员函数
- 通常，我们在一个对象上调用成员函数，而不管该对象是一个左值还是一个右值。
```c++
string s1 ="a value",s2 = "another";
//string右值上调用find成
auto n = (s1+s2).find('a');
```

- 右值有时可以赋值，但是一般左侧运算对象都要为一个左值
```c++
s1+s2="wow!";
```

- 指出`this`的左值/右值属性的方式与定义[[CPP#引入const成员函数（常量成员函数）|const成员函数]]相同，即**在参数列表后放置引用限定符**`&`
![[Pasted image 20230203193431.png]]
- **引用限定符**：可以是`&`或`&&`，分别指出`this`可以指向一个**左值**或**右值**
	- 对于`&`限定的函数，我们只能将它用于左值；
	- 对于`&&`限定的函数，只能用于右值：
![[Pasted image 20230203193734.png]]

- 一个函数可以同时用`const`和**引用限定**。引用限定符必须跟随在`const`后面
![[Pasted image 20230203194050.png]]

#### 重载和引用函数
- 成员函数可以根据参数有无const来区别重载版本。[[CPP#从`const`成员函数返回`this`|重载const成员函数]]。
- **引用限定符**也可以区分**重载**版本。
![[Pasted image 20230203195222.png]]
	- 当我们对一个右值执行`sorted`时，可以直接对data成员进行排序，**对象是右值意味着没有其他用户，因此我们可以改变对象。**
	- 当对一个`const`右值或一个左值执行`sorted`时，我们不能改变对象，因此需要拷贝。

- 编译器会根据调用`sorted`的对象的左值/右值属性来确定使用哪个sorted版本
![[Pasted image 20230203195507.png]]

- 定义`const`成员函数，可以利用有无`const`进行函数重载。
- 定义引用限定函数，如果我们定义两个或两个以上具有相同名就**必须**对**所有函数都加上引用限定符**，或者**所有都不加：**
![[Pasted image 20230203195947.png]]

>[!note]
>- 如果一个成员函数有引用限定符，则具有相同参数列表的所有版本都必须有引用限定符。
>- 对象是**右值意味着没有其他用户**，因此我们可以改变对象。


# 类设计者工具：重载运算与类型转换

## 14.1 基本概念
- **重载运算符**：重载的运算符是具有特殊名字的函数，包含**返回类型**、**参数列表**以及**函数体**。由关键字`operator`和其后的运算符号共同组成。
	- 参数列表：参数数量与该运算符作用的运算对象一样多。一元运算符有一个参数，二元运算符有两个。

- **一元运算符与二元运算符**：
	- 一元运算符：一个参数
	- 二元运算符：两个参数，左侧对象传递给第一个参数，右侧对象传递给第二个参数。

- 除了重载的函数调用运算符`operator()`之外，其他重载运算符不能含有[[CPP#默认实参|默认实参]]

- **重载运算符和成员函数**：如果重载运算符函数是船员函数，则第一个（左侧）运算对象绑定到隐式的`this`指针上。成员函数的（显式参数）比运算符的运算对象总数少一个。

>[!note]
>当一个**重载的运算符是成员函数**时，`this`绑定到左侧运算对象。成员运算符函数的（显式）**参数数量比运算对象的数量少一个**。

- **运算符函数**：它或者是类的成员或者至少含有一个类类型的参数。当运算符作用于内置类型对象（如`int`）时，无法改变运算符的含义。
```c++
int operator+(int,int);//错误：不能为int重定义内置的运算符
```

- **可以被重载**的运算符：大多数可以被重载，少数不能重载。并且**只能重载已有的运算符**，而无权发明新的运算符号。
![[Pasted image 20230205144209.png]]

- **四个特殊符号**：它们既是一元运算符也是二元运算符，都能被重载，我们通过参数的数量推断出到底是一元还是二元。
	- `+`
	- `-`
	- `*`
	- `&`

- **重载运算符的优先级和结合律**：与对应的内置运算符保持一致。
```c++
x==y+z;
//等价于
x==(y+z);
```

#### 直接调用一个重载的运算符函数
- 调用重载运算符的两种方式；
	- 运算符调用：*通常情况*下，我们将运算符作用于类型正确的实参，从而以这种间接方式“调用”重载的运算符函数。
	- 直接调用：像调用普通函数一样直接调用运算符函数
```c++
data1+data2;//普通的表达式
operator+(data1,data2);//等价的函数调用
```

- 调用**成员**重载函数运算符函数：使用成员的`.`运算符。
```c++
data2+=data2;//基于“调用”的表达式
data1.operator+=(data2);//对成员运算符函数的等价调用
```
>这两条语句都调用了成员函数`operator+=`，将`this`绑定到`data1`的地址、将`data2`作为实参传入了函数。

#### 某些运算符不应该被重载
某些运算符指定了运算对象求值的顺序,因为使用重载的运算符本质上是一次**函数调用**，所以这些关于运算对象**求值顺序的规则无法应用到重载的运算符**
>逻辑与运算符、逻辑或运算符、逗号运算的求值顺序和`&&`和`||`的短路特性均无法保留

因为上述运算符的重载版本无法保留求值顺序和/或短路求值属性，因此**不建议重载它们**

- 不重载逗号运算符和取地址运算符的另外原因：C++语言己经定义了这两种运算符用于类类型对象时的特殊含义，因此一般不被重载。

>[!note]
>通常情况下，不应该重载逗号(`,`)、取地址(`&`)、逻辑与(`&&`)和逻辑或(`||`)运算符。

#### 使用与内置类型一致的含义
- 如果某些操作在逻辑上与运算符有关，那么它们适合定义成重载的运算符函数
	- 如果类执行IO操作，则定义移位运算符使其与内置类型的IO保持一致。
	- 如果类的某个操作是检查相等性，则定义`operator==`；如果类有了`operator==`意味着它通常也应该有`operator!=`
	- 如果类包含一个内在的单序比较操作，则定义`operator<`；如果类有了`operator<`，则它也应该含有其他关系操作。
	- 重载运算符的返回类型通常情况下应该与其内置版本的返回类型兼容：
		- 逻辑运算符和关系运算符应该返回`bool`，
		- 算术运算符应该返回一个类类型的值，
		- 赋值运算符和复合赋值运算符则应该返回左侧运算对象的一个引用。

>[!note]
>只有当操作的含义对于用户来说清晰明了时才使用运算符。

#### 赋值和复合赋值运算符
赋值运算符的行为与复合版本的类似：赋值之后，左侧运算对象和右侧运算对象的值相等，并且运算符应该返回它左侧运算对象的一个引用。重载的赋值运算应该继承而非违背其内置版本的含义。

- **复合运算符**：`+=`，`-=`
- 赋值运算符：`=`
>[!note]
>如果类含有算术运算符，或者位运算符，则最好也提供对应的复合赋值运算符。

#### 选择作为成员或者非成员
当我们定义重载的运算符时，必须首先决定是将其声明为类的成员函数还是声明为一个普通的非成员函数。

- 判断重载运算符义为成员函数还是普通的非成员函数的**准则**：
- 赋值`=`、下标`[]`、调用`()`和成员访问箭头`->`运算符必须是成员。
- 复合赋值运算符一般来说应该是成员，但并非必须，这一点与赋值运略不同
- 改变对象状态的运算符或者与给定类型密切相关的运算符，如递增`++`、递减`--`和解引用运算符`*`，通常应该是成员。
- 具有对称性的运算符可能转换任意一端的运算对象，例如算术、相等性、关系和位运算符等，因此它们通常应该是普通的非成员函数。

当我们把运算符定义成**成员函数**时，它的**左侧运算对象必须是运算符所属类的一个对象**。例如：
```c++
string s="world";
string t=s+"!";//正确：我们能把一个const char*加到一个string对象中
string u="hi"+s;//如果+是string的成员，则产生错误
```
- 如果是成员函数，第一个加法等价于`s.operator+("!")`，第二个等价于`"hi".operator+(s)`
- 如果是非成员函数：第二个语句等价于`operator+("hi",s)`，每个实参都能被转换成形参类型。且有一个是类类型，正确。


## 14.2 输入和输出运算符

### 14.2.1 重载输出运算符
- 通常情况下的输出运算符的两个形参
	- 一个是非常量`ostream`对象的引用。
	- 一个是常量的引用，该常量是我们想要打印的类类型。

>第一个形参为什么是非常量的`ostream`？
>因为`ostream`是因为写入内容会改变状态，且流对象无法复制。
>第二个形参为何是引用？
>因为我们希望避免复制实参。

>[!note]
>为了与其他输出运算符保持一致，`operator<<`一般要返回它的形参。

#### Sales_data的输出运算符
![[Pasted image 20230205154547.png]]
打印一个`Sales_data`对象意味着要分别打印它的三个数据成员以及通过计算得到的平均销售价格，每个元素以空格隔开。完成输出后，运算符返回刚刚使用的`ostream`的引用。

#### 输出运算符尽量减少格式化操作
>[!note]
>通常，输出运算符应该主要负责打印对象的内容而非控制格式，**输出运算符不应该打印换行符**，如果运算符打印了换行符，则用户就无法在对象的同一行内接着打印描述性文本。

#### 输入输出运算符必须是非成员函数
与`iostream`标准库兼容的**输入输出运算符必须**是普通的**非成员函数**，而不能是类的成员函数，否则，它们的左侧运算对象将是我们的类的一个对象：
```c++
Sales_data data;
data<<cout;//如果operator<<是Sa1es_data的成员
```
假设输入输出运算符是某个类的成员，则它们也必须是`istream`或`ostream`的成员。然而，这两个类属于标准库，并且我们无法给标准库中的类添加任何成员。
>因为iostream对象是可以操作运算符的，如果是成员函数就必须在此类中定义为成员函数。但是我们无法改变标准库的类。

>[!note]
>因此，如果我们希望为类自定义IO运算符，则**必须将其定义成非成员函数。**


### 14.2.2 重载输入运算符>>
- 通常情况下的输入运算符的两个形参
	- 一个是运算符将要读取的**流的引世**
	- 一个是将要读入到的（非常量）对象的引用。
>第二个形参之所以必须是个非常量是因为输入运算符本身的目的就是将数据读入到这个对象中。

#### Sales_data的输入运算符
![[Pasted image 20230205160504.png]]
`if`语句检查读取操作是否成功，如果发生了错误，则运算符将给定的对象重置为空`Sales_data`，这样可以确保对象处于正确的状态。

>[!note]
>输入运算符必须处理输入可能失败的情况，而输出运算符不需要。
>

#### 输入时的错误
在执行输入运算符时可能发生下列错误：
- 当流含有错误类型的数据时读取操作可能失败。例如在读取完`bookNo`后，输入运算符假定接下来读入的是两个数字数据，一旦输入的不是数字数据，则读取操作及后续对流的其他使用都将失败。
- 当读取操作到达文件末尾或者遇到输入流的其他错误时也会失败。

- 检查输入是否成功：在所有数据读取后使用前进行检查
![[Pasted image 20230205160740.png]]
在错误发生时，我们将对象置为初始状态，避免输入错误的影响。

>[!note]
>当读取操作发生错误时，**输入运算符应该负责从错误中恢复。**
>

#### 标示错误
一些输入运算符需要做更多数据验证的工作。如果流验证为错误，那么则需要设置流的[[CPP#条件状态|条件状态]]标示出失败信息。

通常情况下，输入运算符只设置`failbit`。除此之外，设置`eofbit`表示文件耗尽，而设置`badbit`表示流被破坏。最好的方式是由IO标准库自己来标示这些错误。


## 14.3 算数和关系运算符
通常情况下，我们把**算术和关系运算符定义成非成员函数**以允许对左侧或右侧的运算对象进行转换。因为这些运算符一般不需要改变运算对象的状态，所以形参都是常量的引用。

- 算术运算符：算术运算符通常会计算它的两个运算对象并得到一个新值，这个值常常位于一个局部变量之内，操作完成后**返回**该局部变量的**副本**作为其结果。

- **使用复合赋值来定义算术运算符**：类如果定义了算术运算符，一般也会定义一个对应的复合赋值运算符。
![[Pasted image 20230205162207.png]]
>[!note]
>如果类**同时定义了算术运算符和相关的复合赋值运算符**，则通常情况下应该使**用复合赋值来实现算术运算符。**

### 14.3.1 相等运算符
通常情况下，C++中的类通过**定义相等运算符来检验两个对象是否相等。**

- 如果是类对象，则每一个数据成员都应该进行比较，都相等才算两个对象相等。
![[Pasted image 20230205162351.png]]
![[Pasted image 20230205162358.png]]

- 相等运算符（`==`）的设计准则：
- 如果一个类含有判断两个对象是否相等的操作，则它显然应该把函数定义成`operator==`。
- 如果类定义了`operator==`，则该运算符应该能**判断一组给定的对象中是否含有重复数据**。
- 通常情况下，相等运算符应该具有传递性，换句话说，如果`a==b`和`b==c`都为真，则`a==c`也应该为真。
- 如果类定义了`operator==`，则这个类也应该定义`operator!=`。对于用户来说，当他们能使用`==`时肯定也希望能使用`!=`，反之亦然。
- 相等运算符和不相等运算符中的一个应该把工作委托给另外一个，这意味着其中一个运算符应该负责实际比较对象的工作，而另一个运算符则只是调用那个真正工作的运算符。

>[!note]
>如果某个类在逻辑上有相等性的含义，则该类应该定义`operator==`，这样做可以使得用户更容易使用标准库算法来处理这个类。

### 14.3.2 关系运算符
- **定义了相等运算符的类也常常（但不总是）包含关系运算符。**
>特别是，因为关联容器和一些算法要用到小于运算符，所以定义`operator<`会比较有用。比如，`sort`

通常情况下关系运算符应该：
1. 定义顺序关系，令其与关联容器中对关键字的要求一致。
2. 如果类同时也含有`==`运算符的话，则定义一种关系令其与`==`保持一致。特别是，如果两个对象是`!=`的，那么一个对象应该`<`另外一个。
>`Sales_data`类不支持关系运算符。

>[!note]
>如果存在唯一一种逻辑可靠的`<`定义，则应该考虑为这个类定义`<`运算符。如果同时还包含`==`，则当且仅当`<`的定义和`==`：产生的结果一致时才定义`<`运算符。

这里有解释 看P499


## 14.4 赋值运算符

- **拷贝赋值和移动赋值运算符**：可以把类的一个对象赋值给该类的另一个对象。
- 类还可以定义**其他赋值运算符**以使用别的类型作为右侧运算对象。
```c++
vector<string> v;
v={"a","an","the"};
```
![[Pasted image 20230205174709.png]]
![[Pasted image 20230205174715.png]]

- 这个函数接受可变参数列表，因此在释放当前对象空间时无需检查是否向自身赋值，因为[[CPP#含有可变形参的函数|可变形参]]和`this`不是同个对象。

>[!note]
>我们可以**重载赋值运算符**。**不论形参的类型是什么，赋值运算符都必须定义为成员函数。** 就算接受的是可变形参列表，也必须是成员函数。

#### 复合賦值运算符
**复合赋值运算符**不非得是类的成员，不过我们还是**倾向于把包括复合赋值在内的所有赋值运算都定义在类的内部。**

- 类的复合赋值运算符返回左侧对象的引用，保持与内置类型的复合赋值运算符一样。
![[Pasted image 20230205175131.png]]

>[!note]
>- 赋值运算符必须定义成**类的成员**，复合赋值运算符通常情况下也应该这样做。
-**赋值运算符和复合赋值运算符**这两类运算符都应该**返回左侧**运算对象的**引用**。


## 14.5 下标运算符
- **下标运算符`[]`**：表示容器的类通常可以通过元素在容器中的位置访问元素。

- 下标运算符的**返回值**：常以所访问**元素的引用**作为返回值。

>[!note]
>1. 下标运算符必须是**成员函数。** 
>2. 最好同时定义**常量版本**和**非常量版本**，确保下标运算符返回常量引用以确保我们不会给返回的对象赋值。
>3. 如果一个类包含**下标运算符**，则它通常会定义**两个版本**：
>	1. 一个是返回普通引用
>	2. 一个是类的常量成员并且返回常量引用。

![[Pasted image 20230205175944.png]]
![[Pasted image 20230205180036.png]]
- 当左侧容器是非常量时，可以使用下标运算符对其赋值。容器是常量时，则不能赋值。

>[!note]
>**常量成员只能调用常量函数**，所以我们需要定义下标运算符的常量版本供其调用，并且返回一个对象的常量引用，确保返回的对象不被修改。


## 14.6 递增和递减运算符
- **递增`++`和递减运算符`--`**：使得类可以在元素的序列中**前后移动**。
>[!note]
>递增递减该改变的很好使操作元素的状态，因此建议设置为成员函数。

- **两个版本**：对于内置类型来说，递增和递减运算符既有**前置**版本也有**后置**版本。
>前置和后置指的是递增和递减符号在前面和在后面，不是操作符啊(#`O ′)

#### 定义前置递增/递减运算符
- **前置版本**：返回递增或递减后对象的引用。
![[Pasted image 20230205192816.png]]
- 前置递增：
![[Pasted image 20230205192927.png]]
- 前置递减：
![[Pasted image 20230205192955.png]]

#### 区分前置和后置运算符
前置和后置版本使用的是同一个符号，意味着其重载版本所用的名字将是相同的，并且运算对象的数量和类型也相同。所以直接根据函数区分。

- **解决办法**：后置版本接受一个额外的（不被使用）`int`类型的形参。
![[Pasted image 20230205193351.png]]


- 对于后置版本来说，在递增对象之前需要首先记录对象的状态，然后对原始对象进行加一，返回对象的原始状态。
![[Pasted image 20230205193421.png]]
- 由上可知，我们的**后置运算符调用各自的前置版本来完成实际的工作**。
- 对象本身向前移动了一个元素，而**返回的结果**仍然反映对象在未递增之前**原始的值**。

>[!note]
>区分后置版本和前置版本的递增递减运算符是后置版本加入形参`int`，因为不会用到，所以不用命名。

#### 显式地调用后置运算符
- **成员访问运算符**：解引用运算符`*`，箭头运算符`->`

- **重写运算符**：
	- 解引用运算符：首先检查`curr`是否仍在作用范围内，如果是，则返回`curr`所指元素的一个**引用**。
	- 箭头运算符：不执行任何自己的操作，而是调用解引用运算符并返回**解引用结果**元素的**地址**。
![[Pasted image 20230205194345.png]]
- 箭头运算符返回类型是指针，指针实际上就是某个对象的地址，所以我们返回解引用结果元素的地址。

>[!note]
>**箭头运算符必须是类的成员**。解引用运算符通常也是类的成员，尽管并非必须如此。

![[Pasted image 20230205195402.png]]

#### 对箭头运算符返回值的限定
我们能令`operator*`，完成任何我们指定的操作。而重载箭头运算符时，只能该百年箭头从哪个对象当中获取成员。

- 根据`point`类型不同，`point->mem`分别等价于
```c++
(*point).mem;//point是一个内置的指针类型
point.operator()->mem;//point是类的一个对象
```
- `point->mem`的执行过程如下：
	- 如果`point`是指针，则我们应用内置的箭头运算符，表达式等价于`(*point)·mem`。首先解引用该指针，然后从所得的对象中获取指定的成员。如果`point`所指的类型没有名为`mem`的成员，程序会发生错误。
	- 如果`point`定义了`operator->`的类的一个对象，则我们使用`point.operator->()`的结果来获取`mem`。
		- 如果该结果是一个指针，则执行第上一步
		- 如果该结果本身含有重载的`operator->()`，则重复调用当前步骤。最终，当这一过程结束时程序或者返回了所需的内容，或者返回一些表示程序错误的信息。

>[!note]
>重载的箭头运算符必须返回类的指针或者自定义了箭头运算符的某个类的对象。
>为什么是需要自定义箭头运算符的类对象，因为如果不是指针的话会继续调用该类的重载的箭头运算符，否则会报错。

## 14.8 函数调用运算符
- **重载函数调用运算符**：可以像使用函数一样使用该类的对象。
![[Pasted image 20230206132701.png]]
![[Pasted image 20230206132706.png]]
- `absInt`类对象重载了函数调用运算符，使得我们可以像调用函数一样使用该类对象，所以我们调用函数重载运算符`absObj()`调用类中重载的函数调用运算符函数。

>[!note]
>**函数调用运算符必须是成员函数**。一个类可以定义多个不同版本的调用运算符，相互之间应该在参数数量或类型上有所区别。

- **函数对象**：如果类定义了调用运算符，则该类的对象称作**函数对象**。

#### 含有状态的函数对象类
函数对象类除了`operator()`之外也可以包含其他成员。函数对象类通常含有一些数据成员，这些成员被用于定制调用运算符中的操作。

![[Pasted image 20230206160737.png]]

![[Pasted image 20230206161039.png]]

### 14.8.1 lambda是函数对象
