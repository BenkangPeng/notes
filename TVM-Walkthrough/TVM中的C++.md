### 类型转换 Conversion Operators of class
[ref](https://www.geeksforgeeks.org/conversion-operators-in-cpp/)
> Conversion operators are special **member functions** in a class that enable **implicit or explicit** conversion of objects of that class to **another type**. They help in situations where you need to convert an object of a user-defined type to a basic or another user-defined type.

**本质是将数据类型当成一种运算符进行重载**，但没有形参和返回值类型。
见`include/tvm/runtime/packed_fuc.h`中`TVMPODValue_`的实现.
`TVMPODValue_`能根据赋值语句的类型将成员变量`value_`隐式转换成对应类型返回。

定义隐式转换的语法：

```cpp
class ClassName {
public:
    operator TargetType() const;
};
```

```cpp
#include<iostream>
#include<cmath>
class Vector2D {
public:
    Vector2D(double x, double y) : x(x), y(y) {}

    // 类型转换：向量模长
    operator double() const {
	    std::cout << "call the `operator double()`" << std::endl;
        return std::sqrt(x*x + y*y);
    }
    
    operator int() const{
        std::cout << "call the `operator int()`" << std::endl;
        return std::sqrt(x*x + y*y);
    }

    // 操作符重载：向量加法
    Vector2D operator+(const Vector2D& other) const {
        return Vector2D(x + other.x, y + other.y);
    }

private:
    double x, y;
};

int main(){
	Vector2D v1(3, 4), v2(1, 2);
	double len1 = v1;       // 隐式转换：5.0
    int len2 = v2;
    std::cout << len1 << " " << len2 << std::endl;
	Vector2D v3 = v1 + v2;    // 操作符重载
    
}
```
> ```shell
> call the `operator double()`
> call the `operator int()`
> 5 2
> ```

#### Constructor serves as conversion operator

If a class has a constructor which can be called with a single argument, then this constructor becomes a conversion constructor because such a constructor allows conversion of the single argument to the class being constructed.

eg:

```cpp

#include <iostream>
#include<cmath>
class Complex {
private:
	double real;
	double imag;
public:
	Complex(double r = 0.0, double i = 0.0) : real(r), imag(i){}
	
	bool operator == (Complex rhs){
		return (real == rhs.real && imag == rhs.imag);
	}
};
// Driver Code
int main()
{
	Complex com1(3.0, 0.0);
	if (com1 == 3.0)//3.0 -> Complex(3.0, 0.0);
    //equal to : if(com1 == Complex(3.0,0.0))
		std::cout << "Same";
	else
		std::cout << "Not Same";
	return 0;
}
```

> Same

As the constructor of `Complex` has the default parameter `r=0,i=0`, it's implicitly called as `Complex(3.0,0.0)` when meets `com1 == 3.0`

#### Implicit vs Explicit Conversion

Above code use `implicit type conversion`, which means the compiler will automatically use them when needed. But this may lead to ambiguity:

```cpp

#include <iostream>
#include<cmath>
class Complex {
private:
	double real;
	double imag;
public:
	Complex(double r = 0.0, double i = 0.0) : real(r), imag(i){}
	
	bool operator == (Complex rhs){
		return (real == rhs.real && imag == rhs.imag);
	}
};
int main()
{
	Complex com1(3.0, 0.0);
    std::cout << "com1 == 3.0?  ";
	if (com1 == 3.0)
		std::cout << "Yes";
	else
		std::cout << "No";
	return 0;
}
```

> ```
> com1 == 3.0?  Yes
> ```

We can use `explicit` to decorate the constructor, banning the implicit call of constructor.

```cpp
explicit Complex(double r = 0.0, double i = 0.0) : real(r), imag(i){}
```



### constexpr

`constexpr`用于声明**编译时常量表达式(compile-time constant expressions)**，被`constexpr`修饰的变量或者函数在**编译期**完成计算，从而提高性能。

### static
[ref](https://www.geeksforgeeks.org/static-keyword-cpp/)

#### 函数中的`static variable`
`static`修饰变量时，该变量会被分配一个**固定的内存空间**。即使函数被调用多次，该变量的声明(变量空间的分配)也只会进行一次。
```cpp
#include <iostream>
using namespace std;
void f() {
    // Static variable
    static int count = 0;
    // The value will be updated and carried over
  	// to the next function call
    count++;
  	cout << count << " ";
}

int main() {
  	// Calling function f() 5 times
    for (int i = 0; i < 5; i++)
        f();
    return 0;
}
```


> 1 2 3 4 5

#### Static Data Member in a Class

类中的静态变量独立于类，多个实例公用一个静态变量。
```cpp
#include <iostream>
using namespace std;
class GfG {
public:
  	// Static data member
    static int i;

    GfG(){
        // Do nothing
    };
};
int GfG::i = 1;//声明必须放在main()以外


int main() {
	cout << GfG::i << endl;
    GfG obj1;
    GfG obj2;
    obj1.i = 2;
    obj2.i = 3;
    cout << obj1.i << " " << obj2.i << endl;
    cout << GfG::i << endl;
}

```
> 1
   3 3
   3

#### Static Member Functions in a Class
与`python`中的`@staticmethod`一样，函数不依赖于类的实例，可以直接调用。
```cpp
#include <iostream>
using namespace std;
class GfG {
public:
    // Static member function
    static void printMsg() { cout << "Welcome to GfG!"; }
};
int main() {
    // Invoking a static member function
    GfG::printMsg();
}
```

> Welcome to GfG!



### 三种类的继承方式 & protected private

* `protected` vs `private`

`private`仅允许类内和友元访问，`protected`允许类内和友元访问外，还可以在**子类**中访问。

“私密（private）仅自己，保护（protected）传子女”

| 基类成员原权限 | public继承后权限(保持不变) | protected继承后权限(全变成protected) | private继承后权限(全变成private) |
| -------------- | -------------------------- | ------------------------------------ | -------------------------------- |
| public         | public                     | protected                            | private                          |
| protected      | protected                  | protected                            | private                          |
| private        | 不可见                     | 不可见                               | 不可见                           |

 ### 单例指针调用非静态函数

```cpp
#include<iostream>
class solution{
  
public: 
  static void static_print(){
    std::cout << "static method" << std::endl;   
  }
  
  void instance_print(){
      std::cout << "instance method" << std::endl;
  }
  
  static solution* Global(){
    static solution inst;
    return &inst;
  }
};

int main(){
    solution::static_print();
    // solution::instance_print();//error: cannot call it without object
    
    solution::Global()->instance_print();
}
```

`Global`成员函数返回了一个单例的指针，从而通过指针调用非静态函数。

在`TVM`的`src/runtime/object.cc`中的`class TypeContext`中就有这种写法。



### Functor

`functor`是一个**重载了 `operator()` 的类或结构体的实例**

```cpp
#include<iostream>
struct MyFunctor {
    inline static int cnt = 0;
    void operator()() {
        ++cnt;
        std::cout << "Hello from functor!" << cnt << std::endl;
    }
};

int main() {
    MyFunctor f;
    f();
    
    MyFunctor()();  //先MyFunctor()实例化，再()调用operator()函数
    
    MyFunctor f2;
    f2();
}
```

> ```
> Hello from functor!1
> Hello from functor!2
> Hello from functor!3
> ```

**常用于作为回调函数传入STL算法中**

### clangd配置

* `.clangd`文件配置

```
# Read `compile_commands.json`.
CompileFlags:
  CompilationDatabase: /path/to/complier_commands.json/
```

同时编译`clangd`，下载`Vscode clangd`插件，禁掉`Microsoft C/C++`的代码跳转(会与`clangd`冲突)

在`.vscode/settings.json`中设置

```json
    // forbid microsoft C/Cpp extension, and use clangd
    "C_Cpp.intelliSenseEngine": "disabled",

    "C_Cpp.errorSquiggles": "disabled",    // error squiggles
    "C_Cpp.intelliSenseCachePath": "",     // clean the cache
    "C_Cpp.intelliSenseCacheSize": 0
```



### deprecated

可以使用`deprecated`语法声明弃用的代码：

```cpp
#include<iostream>
#include<cstdint>

[[deprecated("Deprecated and will not be maintained, please use test2() instead.")]]
void test1(){
    // do some work
    std::cout << "Test1" << std::endl;
}

void test2(){
    // do some work
    std::cout << "Test2" << std::endl;
}

int main(){
    test1();
    test2();
}
```

> ```
> main.cpp: In function 'int main()':
> main.cpp:18:10: warning: 'void test1()' is deprecated: Deprecated and will not be maintained, please use test2() instead. [-Wdeprecated-declarations]
>    18 |     test1();
>       |     ~~~~~^~
> main.cpp:6:6: note: declared here
>     6 | void test1(){
>       |      ^~~~~
> Test1
> Test2
> ```

### Macro trick

#### `#`字符串化运算符，Stringizing Operator

将宏参数展开为字符串字面量

```cpp
#define STRINGIZE(x) #x
printf("%s", STRINGIZE(hello));  // 展开为 printf("%s", "hello");
```

若参数本身是宏，则阻止宏的展开：

```cpp
#define FOO 123
printf("%s", STRINGIZE(FOO));  // 输出 "FOO"，而非 "123"
```

#### `##`标记连接运算符，`Token Pasting Operator`

将两个标记连接成一个新标记

```cpp
#define CONCAT(a, b) a##b

int xy = 10;
printf("%d", CONCAT(x, y));  // 展开为 printf("%d", xy); 输出 10
```

但参数为宏时，会报错：

它不会先展开宏，再拼接

```cpp
#define A 1
#define B 2
#define CONCAT(a, b) a##b
#include<stdio.h>
int main(){  
    printf("%d", CONCAT(A, B));
}
```

> ```
> main.cpp: In function 'int main()':
> main.cpp:8:25: error: 'AB' was not declared in this scope; did you mean 'A'?
>     8 |     printf("%d", CONCAT(A, B));  // 展开为 printf("%d", 12);（若 A 和 B 是变量名则为AB）
>       |                         ^
> main.cpp:3:22: note: in definition of macro 'CONCAT'
>     3 | #define CONCAT(a, b) a##b
> ```

使用嵌套宏来解决这个问题

#### 嵌套宏

```cpp
#define STR_CONCAT_(__x, __y) __x##__y
#define STR_CONCAT(__x, __y) STR_CONCAT_(__x, __y)
```

`#define STR_CONCAT(__x, __y) STR_CONCAT_(__x, __y)`会先将宏展开，再交给`STR_CONCAT_`拼接



```cpp
#define A 1
#define B 2
#define CONCAT(a, b) CONCAT_(a, b)
#define CONCAT_(a, b) a##b
#include<stdio.h>
int main(){ 
    printf("%d", CONCAT(A, B));
}
```

> 12



### Registry 注册机

注册机通过使用以下变量在`main()`之前初始化的性质，实现自动注册(写入全局表中)：

- 全局变量（如 `int x;`）
- `static` 变量（如 `static int x;`）
- 匿名命名空间内的变量（默认具有静态存储期）

挖坑以后再填



```python
irmodule2: # from tvm.script import ir as I
# from tvm.script import relax as R

@I.ir_module
class Module:
    @R.function(private=True)
    def globalvar() -> R.Tensor((-1,), dtype="int32"):
        # from tvm.script import relax as R
        
        @R.function
        def f(ipt: R.Tensor((-1,), dtype="int32")) -> R.Tensor((-1,), dtype="int32"):
            x0: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][0]
            y: R.Tensor((-1,), dtype="int32") = f(x0)
            return y

        x1: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][1]
        x2: R.Tensor((-1,), dtype="int32") = R.add(x1, f(x1))
        gv0: R.Tensor((-1,), dtype="int32") = x2
        return gv0
```



```python
CallTIRRewrite()(irmodule2): # from tvm.script import ir as I
# from tvm.script import relax as R

@I.ir_module
class Module:
    @R.function(private=True)
    def globalvar() -> R.Tensor((-1,), dtype="int32"):
        # from tvm.script import relax as R
        
        @R.function
        def f(ipt: R.Tensor((-1,), dtype="int32")) -> R.Tensor((-1,), dtype="int32"):
            x0: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][0]
            y: R.Tensor((-1,), dtype="int32") = f(x0)
            return y

        x1: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][1]
        gv: R.Tensor((-1,), dtype="int32") = f(x1)
        x2: R.Tensor((-1,), dtype="int32") = R.add(x1, gv)
        gv0: R.Tensor((-1,), dtype="int32") = x2
        return gv0
```

```python
LowerTVMBuiltin()(irmodule2): # from tvm.script import ir as I
# from tvm.script import relax as R

@I.ir_module
class Module:
    @R.function(private=True)
    def globalvar() -> R.Tensor((-1,), dtype="int32"):
        # from tvm.script import relax as R
        
        @R.function
        def f(ipt: R.Tensor((-1,), dtype="int32")) -> R.Tensor((-1,), dtype="int32"):
            x0: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][0]
            y: R.Tensor((-1,), dtype="int32") = f(x0)
            return y

        x1: R.Tensor((-1,), dtype="int32") = metadata["relax.expr.Constant"][1]
        x2: R.Tensor((-1,), dtype="int32") = R.add(x1, f(x1))
        gv0: R.Tensor((-1,), dtype="int32") = x2
        return gv0
```





