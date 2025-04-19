### Why `cudaMalloc` Requires `void**` as a Parameter

Why should we use `void**` instead of `void*` to pass into `cudaMalloc`?

I used to think that when passing a pointer into a function, what was actually passed was the pointer **itself** instead of the **copy** of it, because I just saw that I can modify the memory in the function.

```cpp
#include <iostream>
using namespace std;

void func(int* a_copy){
	//`a_copy` is a copy of the pointer `a` from the main function.
	//Both point to the same memory location, but `a_copy` itself resides at a different address.
    //Thus, `a` and `a_copy` are distinct pointer variab
	
	cout << "a_copy points to " << a_copy << endl;  //the same address where `a` points
	
	//address `a_copy` itself, differ from address of `a` in main function
	cout << "the pointer `a_copy`'s address is " << &a_copy << endl;
	
	
	*a_copy += 1;//We can use a_copy to modify the memory where it points
}

void ChangeAddressPointing1(int* a_copy){
	a_copy = new int(88);
	//Cause a_copy is just the copy of `a`, changing address where a_copy points
	//don't affect the address where `a` points in the main function.
	
	//However, passing the address of the pointer or a reference to it allows the function to modify the memory address the original pointer points to.↓
}

void ChangeAddressPointing2(int** a_copy){
	*a_copy = new int(88);
}

int main() {
	int *a = new int(8);
	cout << "a in main function points to " << a << endl;  //address where `a` points
	cout << "the pointer `a`'s address is " << &a << endl; //address `a` itself
	func(a);
	cout << *a << endl;//9
	
	//But if I want to change the address where `a` points in function, how do I do?
	ChangeAddressPointing1(a);
	cout << "a still point to " << a << " with " << *a << endl;
	
	ChangeAddressPointing2(&a);
	cout << "a point to " << a << " with " << *a << endl;
	
	return 0;
}
```

> ```
> a in main function points to 0x5589283cde70
> the pointer `a`'s address is 0x7ffccf668d40
> a_copy points to 0x5589283cde70
> the pointer `a_copy`'s address is 0x7ffccf668d18
> 9
> a still point to 0x5589283cde70 with 9
> a point to 0x5589283ceec0 with 88
> ```



The demo above shows that:

* While passing a pointer into the function as a parameter, it actually passes the copy of the pointer.
* To modify the memory address a pointer points to within a function, we must pass either a `pointer-to-pointer` or a `reference to a pointer`.

So for `cudaMalloc` function:

```c
cudaError_t cudaMalloc ( void** ptr, size_t size );
```

* `ptr`'s type should be `void**` due to the function prototype's requirement.
* `ptr` should be a double pointer, as cuda will change the memory address where `ptr` points in `CPU(Host)` into a new memory address in `GPU(Device)` in the function `cudaMalloc`.

:rocket: However, when the formal parameter is a reference, what is passed is the variable itself, or in other words, an alias of the variable. 

```cpp
#include <iostream>
using namespace std;
void f(int& x){
	x = 12;
	cout << &x << endl;
}
int main() {
	int x = 10;
	cout << &x << endl;
	f(x);
	return 0;
}
```

> ```
> 0x7ffd1e75d744
> 0x7ffd1e75d744
> ```

### threadIdx, blockIdx, gridDim, blockDim

:link:[reference](https://zhuanlan.zhihu.com/p/544864997)



### Reference

[MLSys 入门向读书笔记\] CUDA by Example: An Introduction to General-Purpose GPU Programming（下） - 知乎](https://zhuanlan.zhihu.com/p/718988880)



$\sqrt(\frac{32ln\frac{em}{2}+8ln\frac{4}{\delta}}{m})$

$m \geq \frac{4}{\epsilon}ln\frac{4}{\delta}$
