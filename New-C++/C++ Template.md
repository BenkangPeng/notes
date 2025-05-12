

### 模板参数不会被推导的情况

```cpp
#include <iostream>
template<typename RT, typename T1, typename T2>
RT max(T1 a, T2 b){
	return a > b ? a : b;
}
int main() {
	int a = 2;
	double b = 3.14;
	std::cout << ::max(a, b) << std::endl;
}
```

> ```
> prog.cpp:3:4: note:   template argument deduction/substitution failed:
> prog.cpp:9:25: note:   couldn't deduce template parameter ‘RT’
>   std::cout << ::max(a, b) << std::endl;
>                          ^
> ```

上面的例子中，`RT`的类型是`T1` or `T2`，编译器无法推断：

>  In cases when there is **no connection** between **template parameters(e.g. RT)** and **call parameters(e.g. a and b)**,  the compiler can't take the template parameters into account and also can't deduce it.

`RT`与函数形参无关时，编译器**不会去推断**`RT`的类型。

编译器不是推断不出`RT`，而是根本不会去推断`RT`。把代码改成这样，编译器也不会去推断`RT`:

```cpp
#include <iostream>
template<typename RT, typename T>
RT max(T a, T b){
	return a > b ? a : b;
}
int main() {
	int a = 2;
	int b = 3;
	std::cout << ::max(a, b) << std::endl;
}
```

尽管我们一眼可以看出`RT`肯定等于`T`，但编译器不会去推断。

**正确示范**

**使用`auto`**

```cpp
#include <iostream>
template<typename T1, typename T2>
auto max(T1 a, T2 b){
	return a > b ? a : b;
}
int main() {
	int a = 2;
	double b = 3.14;
	std::cout << ::max(a, b) << std::endl;
}
```

`auto`会在编译期进行推断，例如`a为int,b为double`，在进行`a>b`比较时，`a`会隐式转换为`double`，那么`auto`会被推断为`double`.



**显示指定**

```cpp
#include <iostream>
template<typename RT, typename T1, typename T2>
RT max(T1 a, T2 b){
	return a > b ? a : b;
}
int main() {
	int a = 2;
	double b = 3.14;
	std::cout << ::max<double>(a, b) << std::endl;
}
```

**type trait 类型萃取**

```cpp
#include <iostream>
#include<type_traits>
template<typename T1, typename T2, typename RT = std::common_type_t<T1,T2>>
RT max(T1 a, T2 b){
	return a > b ? a : b;
}
int main() {
	int a = 2;
	double b = 3.14;
	std::cout << ::max<double>(a, b) << std::endl;
}
```

`std::common_type_t<>`会返回多个类型的公共类型。例如`std::common_type_t<int,double>`等于`double`，因为可以隐式转换`int`为`double`从而统一类型。





### Concepts

在模板下添加`requires`

```cpp
#include<concepts>

template <typename T>
requires std::totally_ordered<T> // 要求类型T可排序
T my_max(T a, T b) {
    return a > b ? a : b;
}
```

可以尝试去掉以下代码中的`requires`，看看报错信息有多么令人头疼。加上`requires`，报错信息更精确。

```cpp
#include <iostream>
#include<vector>
#include<string>
#include <concepts>

class Person{
private:
    std::string name;
public:
    template<typename STR>
    requires std::is_convertible_v<STR,std::string>
    
    explicit Person(STR&& n): name(std::forward<STR>(n)){
        std::cout << "TMPL-CONSTR for " << name << "\n";
    }
};
int main(){
    Person p(123);
}
```

常见的约束：

|        概念（Concept）        |                   用途                   |                   示例                    |
| :---------------------------: | :--------------------------------------: | :---------------------------------------: |
|      `std::integral<T>`       | `T` 必须是整数类型（`int`, `short` 等）  |        `requires std::integral<T>`        |
|   `std::floating_point<T>`    | `T` 必须是浮点类型（`float`, `double`）  |     `requires std::floating_point<T>`     |
|      `std::copyable<T>`       | `T` 可拷贝（有拷贝构造函数和赋值运算符） |        `requires std::copyable<T>`        |
|       `std::movable<T>`       | `T` 可移动（有移动构造函数和赋值运算符） |        `requires std::movable<T>`         |
| `std::equality_comparable<T>` |          `T` 支持 `==` 和 `!=`           |  `requires std::equality_comparable<T>`   |
|   `std::totally_ordered<T>`   |      `T` 支持 `<`, `<=`, `>`, `>=`       |    `requires std::totally_ordered<T>`     |
|    `std::ranges::range<T>`    |  `T` 是范围（有 `begin()` 和 `end()`）   |     `requires std::ranges::range<T>`      |
|   `std::input_iterator<T>`    |             `T` 是输入迭代器             |     `requires std::input_iterator<T>`     |
| `std::invocable<F, Args...>`  |       `F` 可调用，参数为 `Args...`       | `requires std::invocable<F, int, double>` |

**Defining Concepts**

**检查成员函数是否存在**

```cpp
#include<concepts>
#include<vector>
#include<string>
#include<iostream>
template <typename T>
requires requires(T obj){
    obj.size();
    obj.clear();
}
void reset(T& obj){
    obj.clear();   
}

int main(){
    std::vector<int> v1{1,2};
    reset(v1);
    std::cout << v1.size() << std::endl;
    
    int a = 1;
    reset(a);//constraints not satisfied  
    return 0;
}
```

**检查运算符**

```cpp
template<typename T>
requires requires(T a, T b) {
    { a + b } -> std::convertible_to<T>;  // 必须支持 + 运算且返回 T
}
T add(T a, T b) {
    return a + b;
}
```



### Alias Template

```cpp
template <typename T>
class MyContainer{
	......
}

template<typename T>
using MyIterator = typename MyContainer<T>::iterator;//需要指明typename

MyIterator<int> pos;
```



### Nontype Template Parameters

模板参数不一定是`type`，也可以是`constant integral values`

```cpp
#include<iostream>
#include<array>
#include<cassert>

template <typename T, int MaxSize>
class myStack{
private:
    std::array<T,MaxSize> elem;
    int num;

public:
    myStack();
    
    void push(const T& _elem);

    void pop();

    T top();

    int size();


};

template<typename T, int MaxSize>
myStack<T,MaxSize>::myStack(): num(0){}

template<typename T, int MaxSize>
void myStack<T,MaxSize>::push(const T& _elem){
    assert(num < MaxSize);
    elem[num++] = _elem;
}


template<typename T, int MaxSize>
void myStack<T, MaxSize>::pop(){
    assert(num>0);
    --num;
}

template<typename T, int MaxSize>
int myStack<T, MaxSize>::size(){
    return num;
}

template<typename T, int MaxSize>
T myStack<T, MaxSize>::top(){
    assert(num > 0);
    return elem[num-1];
}
```



### Variadic Templates

通过`variadic Templates`实现可变参数：

```cpp
#include<iostream>

void print(){}

template <typename T, typename... Types>
void print(T first, Types... args){
    
    std::cout << first << std::endl;
    print(args...);
}

int main(){
    print(1, 2.0, "Hello");
}
```

注意：要在模板前定义一个终止函数`void print(){}`，否则模板展开时会反复调用`print(arg...)`导致爆栈。

实现`and_all`:

```cpp
#include<iostream>


bool and_all(){
    return true;
}

template<typename T>
bool and_all(T one){
    return one;
}

template <typename T, typename... Types>
bool and_all(T first, Types... args){
    return first && and_all(args...);   
}

int main(){
    std::cout << and_all(1,1,1,1,1,0) << std::endl;
    std::cout << and_all(1) << std::endl;
}
```



### Fold Expressions

|    折叠类型    |           语法            | 展开示例（参数包为 `a, b, c`） |
| :------------: | :-----------------------: | :----------------------------: |
| **一元左折叠** |     `( ... op args )`     |       `((a op b) op c)`        |
| **一元右折叠** |     `( args op ... )`     |       `(a op (b op c))`        |
| **二元左折叠** | `( init op ... op args )` |  `(((init op a) op b) op c)`   |
| **二元右折叠** | `( args op ... op init )` |  `(a op (b op (c op init)))`   |

```cpp
#include<iostream>

template <typename... Types>
bool and_all(Types... args){
    return (args && ...);   
}

template<typename... Types>
auto add_all(Types... args){
    return (0 + ... + args);// 而且括号不能少，不能写成0 + ... + args
}
int main(){
    std::cout << and_all(1,1,1,1,1,0) << std::endl;   
    std::cout << add_all(1, 3.14, 2u) << std::endl;
}
```

* 不能写成 `return (args + ...)`; 防止无参数调用,例如`add_all()`
* 括号不能少，不能写成`return 0 + ... + args`

* `(0 + ... + args)` 就是``( init op ... op args )`` 的形式



```cpp
#include<iostream>

template <typename... Types>
void print_all(Types... args){
    
    ( (std::cout << args << "\n"), ... );   
}
int main(){
    print_all(1, "hello", 3.14, "world");
    
}
```

> ```
> 1
> hello
> 3.14
> world
> ```

* `( (std::cout << args << "\n"), ... )` 是一元右折叠表达式.

`args`拼成了一个新的`new_args` = `(std::cout << args << "\n")`，`new_args`中有四个变量，分别是`(std::cout << 1 << "\n")`, `(std::cout << "hello" << "\n")`, `(std::cout << 3.14 << "\n")`, `(std::cout << "world" << "\n")`

* `(new_args , ...)` 一元右折叠，`op` = `,`  



```cpp
#include<iostream>

template <typename T1, typename... TN>
constexpr bool isHomogeneous(T1, TN...){//可以省略参数名
    return (std::is_same<T1,TN>::value && ...);
}
int main(){    
    std::cout << isHomogeneous(1, 2 , "hello") << std::endl;
    
    std::cout << isHomogeneous(1, 2 , 3) << std::endl;
}
```



### Generic Template Parameter & Reference Collasing & Perfect Forwarding

`add_all`这样写更好：

```cpp
#include<iostream>

template <typename... Args>
auto add_all(Args&&... args){
       return (0 + ... + std::forward<Args>(args));
}

int main(){ 
    int x = 6;
    std::cout << add_all(1, x, 2, 3.14, 'a') << std::endl;
}
```

* `Generic Template Parameter`：使用`Args&&...`能够同时处理`lvalue`和`rvalue`
* `Reference Collasing`: `int&&&`等价于`int&`, `int&&&&` -> `int&&`
* `Perfect Forwarding`: 完美转发，不改变左值右值属性

当`add_all`被传入`rvalue`例如`3.14`时，`Args`推断为`float`, 完美转发后为`float&& 3.14`

当`add_all`被传入`lvalue`例如`x`时，`Args`推断为`int&`, 完美转发后为`int&&& x` = `int& x`

一个使用完美转发前后的性能对比实验：[link](https://ideone.com/Nt5DzC)



### `typename `keyword

```cpp
#include<iostream>
#include<string>
#include<vector>
#include<tuple>
// print elements of an STL container

template<typename T>
void printcoll (T const& coll){
    typename T::const_iterator pos;
    typename T::const_iterator end(coll.end());// end position

    for(pos = coll.begin(); pos != end; ++pos){
        std::cout << *pos << std::endl;
    }
}
int main(){
    std::vector<int> v1{1, 2, 3};
    std::vector<std::string> v2{"hello", "world"};   
    
    printcoll(v1);
    printcoll(v2);
}
```



### Disable Templates with `enable_if_t<>`

语法：

```cpp
template <typename T>
std::enable_if_v< bool_expr, type >  foo(){
    
}
```

* 如果`bool_expr`为真，函数`foo`的返回值为`type`
* 如果`bool_expr`为假，则`foo`不完成实例

* `type`缺省为`void`

```cpp
#include <iostream>
#include <type_traits>

// 整数打印
template<typename T>
std::enable_if_t<std::is_integral_v<T>, void>
    print(T value) {
        std::cout << "Integer: " << value << std::endl;
    }

// 浮点数打印
template<typename T>
    std::enable_if_t<std::is_floating_point_v<T>, void>
    print(T value) {
        std::cout << "Floating: " << value << std::endl;
    }

int main() {
    print(42);       // Integer
    print(3.14f);    // Floating
    print("hello"); // no matching function
    return 0;
}
```

可以使用默认模板参数+`enable_if_t`来对模板参数进行约束：

```cpp
#include <type_traits>

template<typename T, typename = std::enable_if_t<std::is_integral_v<T>>>
void foo() {
    std::cout << "T is integral" << std::endl;
}
int main() {
    foo<int>();    // ✅ 编译通过
    foo<double>(); // ❌ 编译失败（SFINAE 排除此版本）
    return 0;
}
```

* 第一个模板参数是`T`，第二个模板参数`typename = std::enable_if_t<std::is_integral_v<T>>` , 通过第二个模板参数的推导来限制第一个模板参数`T`的类型
* 如果`T`是整数类型，模板展开为

```cpp
tempalte<typename T, typename = void>
```

第二个模板参数用不上，只是为了去约束`T`的。

### concept

个人觉得`concept`相较于`enable_if`更加简洁清晰。但是`concept`在`C++20`进入标准，可能老代码还是得用`enable_if`.

```cpp
template<typename STR>
requires std::is_convertible_v<STR,std::string>

PerSon(STR&& n):name(std::forward<STR>(n)) {
......
}
```



