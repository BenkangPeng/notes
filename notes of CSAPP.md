# 第一章 计算机系统漫游

* 信息就是位＋上下文
* 预处理→编译→汇编→链接
* 存储器层次结构：

<img src = "https://pic3.zhimg.com/v2-37cd14433f0a64844ccd435f3b48b236_b.jpg" style = "width : 400px ; height : 400px">

* 进程是操作系统对一个正在运行的程序的抽象，一个系统可以同时运行多个进程，这种看上去多进程是并发进行的现象，实则是由**上下文切换**实现的。
* 进程间的切换是由内核(kernel)管理的，内核是操作系统常驻主存的部分。
* 一个进程可以由多个线程组成。由于多线程之间比多进程之间更容易共享数据，因此线程并行更容易实现且更高效。

# 第二章 信息的表示和处理

* `size_t`即`max——type(int , char , long ……)` ，即所有数据类型中字节数最大的一个。一般地，`short`占2字节，`int`占4字节，`double`占8字节，所以一般的机器中，**`size_t`是字节数为8的``unsigned int ``** . `size_t`的作用就是设置一个当前机器最大的、无符号的整数数据类型。

```cpp
    size_t x = 114514 ;
    std::cout << "x = " << x << "  sizeof(size_t) = " << sizeof(size_t) << std::endl ;
```

> x = 114514  sizeof(size_t) = 8

* IEEE 浮点表示

$V = (-1)^s×M×2^E$  尾数(significand)、阶码(exponent)

单精度浮点：32位，其中s占1位，exp占8位，frac占23位。

双精度浮点：64位，其中s占1位，exp占11位，frac占52位。

```cpp
#include<iostream>

typedef unsigned char *byte_pointer ;

void show_bytes(byte_pointer start , size_t len){

    size_t i ;

    for(i = 0 ; i < len ; i++){

        std::printf("%.2x  " , start[i]) ;

    }

    std::cout << std::endl ;

}

void show_int(int x){

    show_bytes((byte_pointer)&x , sizeof(int)) ;

}

void show_float(float x){

    show_bytes((byte_pointer)&x , sizeof(float)) ;

}

void show_pointer(int* x){

    show_bytes((byte_pointer)&x , sizeof(int)) ;

}

void test_show_bytes(int val){

    int ival = val ;

    float fval = (float)val ;

    int *pval = &ival ;

    show_int(ival) ;

    show_float(fval) ;

    show_pointer(pval) ;

}

int main()

{
    test_show_bytes(-12345) ;
    int x = 12345 ;
    byte_pointer p = (byte_pointer) &x ;
    for(int i = 0 ; i < 4 ; i++){
        std::printf("address %0x 's content is %0x\n", p+i , *(p+i) );
    }
    for(int i = 0 ; i < 4 ; i++){
        std::printf("address %0x 's content is %0x\n" , &x + i , *( (&x)+i ) );
    }

}
```

* `(unsigned char *)&x`是将`x`的地址截下低位的一个字节，转为`unsigned char`指针。为什么要进行以上转换？可以看到：

> address 61fe0c 's content is 39
> address 61fe0d 's content is 30
> address 61fe0e 's content is 0
> address 61fe0f 's content is 0
> address 61fe0c 's content is 3039
> address 61fe10 's content is 61fe0c
> address 61fe14 's content is 0
> address 61fe18 's content is 3

`&x`与`(&x) + 1`是相差4个字节的(`int`占4个字节)。`p`与`p + 1`相差一个字节(`unsigned char`占1个字节)。

`x`十进制为12345，十六进制则为`00003039` , 若`&x = 61fe0c` , 则小端模式下`*(61fe0c) = 39,*(61fe0d) = 30 , *(61fe0e)=00 , *(61fe0e)=00`

只有用`unsigned char`才能做到每自增1同时实现地址加1 .

* 为什么不用`char`，`char`也是一个字节。测试:

```cpp
char *p_char = (char*) &x ;
    for(int i = 0 ; i < 4 ; i++){
        std::printf("address %0x 's content is %0x\n", p_char+i , *(p_char+i) );
    }
```

> address 61fe08 's content is 61fdfc
> address 61fdfc 's content is ffffffc7
> address 61fdfd 's content is ffffffcf
> address 61fdfe 's content is ffffffff
> address 61fdff 's content is ffffffff

因为`char`会输出负数前缀`ffffff`
