

#### 隐式类型转换
见`include/tvm/runtime/packed_fuc.h`中`TVMPODValue_`的实现.
`TVMPODValue_`能根据赋值语句的类型将成员变量`value_`隐式转换成对应类型返回。

定义隐式转换的语法：

```cpp
operator TargetType() const;
```

```cpp
class MyClass {
private:
    int value;
public:
    MyClass(int v) : value(v) {}

    // 重载转换为 int 的运算符
    operator int() const {
        return value;
    }

    // 重载转换为 double 的运算符
    operator double() const {
        return static_cast<double>(value);
    }
};

int main() {
    MyClass obj(42);
    int x = obj;       // 隐式调用 operator int()
    double y = obj;    // 隐式调用 operator double()
    std::cout << x << ", " << y;  // 输出: 42, 42
}
```

#### 可变参数模板
