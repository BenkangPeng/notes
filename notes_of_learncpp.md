### 1.x——Chapter 1 summary and quiz

* When writing programs , add a few lines or a function , compile , resolve any errors , and make sure it works . Don't wait until you're written an entire program before compiling it for the first time !

### 

### 5.5—Numeral systems (decimal,binary,hexadecimal,and octal)

```cpp
int x{12} ; //x is a decimal number : 12
int x1{012} ;//x is a octal number : 12(10 in decimal)
int x2{0xF} ;//x is a hexadecimal number 
int x3{0b1} ;//x is a binary number
```

```cpp
	int bin{};    // assume 16-bit ints
    bin = 0x0001; // assign binary 0000 0000 0000 0001 to the variable
    bin = 0x0002; // assign binary 0000 0000 0000 0010 to the variable
    bin = 0x0004; // assign binary 0000 0000 0000 0100 to the variable
    bin = 0x0008; // assign binary 0000 0000 0000 1000 to the variable
    bin = 0x0010; // assign binary 0000 0000 0001 0000 to the variable
    bin = 0x0020; // assign binary 0000 0000 0010 0000 to the variable
    bin = 0x0040; // assign binary 0000 0000 0100 0000 to the variable
    bin = 0x0080; // assign binary 0000 0000 1000 0000 to the variable
    bin = 0x00FF; // assign binary 0000 0000 1111 1111 to the variable
    bin = 0x00B3; // assign binary 0000 0000 1011 0011 to the variable
    bin = 0xF770; // assign binary 1111 0111 0111 0000 to the variable
```

```cpp
#include <bitset> // for std::bitset
#include <iostream>

int main()
{
	// std::bitset<8> means we want to store 8 bits
	std::bitset<8> bin1{ 0b1100'0101 }; // binary literal for binary 1100 0101
	std::bitset<8> bin2{ 0xC5 }; // hexadecimal literal for binary 1100 0101

	std::cout << bin1 << '\n' << bin2 << '\n';
	

	return 0;
}
```

```cpp
#include <iostream>

int main()
{
    int x { 12 };
    std::cout << x << '\n'; // decimal (by default)
    std::cout << std::hex << x << '\n'; // hexadecimal
    std::cout << x << '\n'; // x is now hexadecimal
    std::cout << std::oct << x << '\n'; // x is turned into octal
    std::cout << std::dec << x << '\n'; // return to decimal
    std::cout << x << '\n'; // decimal

    return 0;
}
```

- `test()` allows us to query whether a bit is a 0 or 1.
- `set()` allows us to turn a bit on (this will do nothing if the bit is already on).
- `reset()` allows us to turn a bit off (this will do nothing if the bit is already off).
- `flip()`allows us to flip a bit value from a 0 to a 1 or vice versa.

# 7.2 — User-defined namespaces and the scope resolution operator

```cpp
#include<iostream>
namespace mynamespace{
    void cout(){
        std::cout << "*****" << std::endl ;
    }
}
using namespace mynamespace ;
int main()
{
    std::cout << "jfkdksf" << std::endl ;
    cout() ;
    return 0 ;
}
```

`namespace`多用于避免函数名冲突(`prevent naming collisions`)：在不同的`namespace`中定义名称相同的`function`，用`::` 区分`function`