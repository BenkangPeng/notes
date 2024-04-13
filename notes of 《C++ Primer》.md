# Section 1.5 Introducing Classes

* Headers from the standard library are enclosed in angle **brackets (<>)**. Those that are not part of the library(code by youself) are enclosed in double **quotes ("")**.

* **USING FILE REDIRECTION** 

  It can be tedious to repeatedly type these transactions as input to the programs you are testing. 

  Most operating systems support file redirection, which lets us associate a named file with the standard input and the standard output: **$ addItems <infile >outfile$**

   Assuming $ is the system prompt and our addition program has been compiled into an executable file named addItems.exe (or addItems on UNIX systems), this command will read transactions from a file named infile and write its **output to a file** named outfile in the current directory.

* Processing Every Character? Use Range-Based for 

```cpp
for (declaration : expression) 
    statement
```

```cpp
string str("some string") ;
for(auto c : str)
    cout << c << endl ;
```

**Using a Range for to Change the Characters in a string**

```cpp
string str("some string") ;
for(auto &c : str)
	c = toupper(c) ;
cout << str << endl ;
```

* The type of return variable of `<string>.size()` is `string::size_type` , like `size_t` .

**Subscripting Does Not Add Elements** 

```cpp
vector<int> ivec; // empty vector 
for (decltype(ivec.size()) ix = 0; ix != 10; ++ix) ivec[ix] = ix; // disaster: ivechas no elements
```

# Iterators

```cpp
auto b = v.begin() , e = v.end() ;
```

```cpp
string s("some string"); 
if (s.begin() != s.end()){  // make sure sis not empty 
    auto it = s.begin(); // itdenotes the first character in s 
	*it = toupper(*it); // make that character uppercase 
}
```

```cpp
for(auto it = s.begin() ; it != s.end() ; it++)
    	cout << *it ;
```

**KEY CONCEPT:GENERIC PROGRAMMING** 

​	Programmers coming to C++ from C or Java might be surprised that we used != rather than < in our for loops such as the one above and in the one on page 94. C++ programmers use != as a matter of habit. They do so for the same reason that they use iterators rather than subscripts: This coding style applies equally well to various kinds of containers provided by the library. 

​	As we’ve seen, only a few library types, vector and string being among them, have the subscript operator. Similarly, all of the library containers have iterators that define the == and != operators. Most of those iterators do not have the < operator. By routinely using iterators and !=, we don’t have to worry about the precise type of container we’re processing.

## Iterator Types

The library types that have iterators define types named `iterator` and `const_iterator` that represent actual iterator types:

```cpp
vector<int>::iterator it; // itcan read and write vector<int>elements 
string::iterator it2; // it2can read and write characters in a string 
vector<int>::const_iterator it3; // it3can read but not write elements 
string::const_iterator it4; // it4can read but not write characters
vector<int> v; 
const vector<int> cv; 
auto it1 = v.begin(); // it1has type vector<int>::iterator 
auto it2 = cv.begin(); 
// it2 has typevector<int>::const_iterator
```

```cpp
auto it3 = v.cbegin(); 
// it3has type vector<int>::const_iterator
```

## Combing Dereference and Member Access

When we dereference an iterator, we get the object that the iterator denotes. If that object has a class type, we may want to access a member of that object. For example, we might have a vector of strings and we might need to know whether a given element is empty. Assuming it is an iterator into this vector,we can check whether the string that it denotes is empty as follows: 

```cpp
(*it).empty() ;
(*it).empty() // dereferences itand calls the member emptyon the resulting object 
*it.empty() // error: attempts to fetch the member named emptyfrom it // but it is an iterator and has no member named empty
```

To simplify expressions such as this one, the language defines the arrow operator (the -> operator). The arrow operator combines dereference and member access into a single operation. That is, `it->mem` is a synonym for `(*it).mem`.



## begin and end function in array

Although we can compute an off-the-end pointer, doing so is error-prone. To make it easier and safer to use pointers, the new library includes two functions, named **begin** and **end**. These functions act like the similarly named container members (§ 3.4.1, p. 106). However, **arrays are not class types**, so these functions **are not member functions**. Instead, they **take an argument** that is an array:

```cpp
int ia[] = {0,1,2,3,4,5,6,7,8,9}; // iais an array of ten ints 

int *beg = begin(ia); // pointer to the first element in ia 
int *last = end(ia); // pointer one past the last element in ia
```



```cpp
string s1 = "a string", 
*p = &s1; auto n = s1.size(); // run the sizemember of the strings1 
n=(*p).size(); // run sizeon the object to which ppoints 
n = p->size(); // equivalent to (*p).size()
```

# Inline Functions Avoid Function Call Overhead

On most machines, a function call does a lot of work: Registers are saved before the call and restored after the return; arguments may be copied; and the program branches to a new location.

A function specified as inline (usually) is expanded “in line” at each call

# The assert Preprocessor Macro

assert is a preprocessor macro , which is defined in the `<cassert>`header .

> assert(expr)

evaluates expr and if the expression is false (i.e., zero), then assert writes a message and terminates the program. If the expression is true (i.e., is nonzero), then assert does nothing.

# Template

我们常在**函数功能相同、数据类型不同**的场景下应用**函数重载**。例如：

```cpp
void print(vector<int>& a){
    for(auto it = a.begin() ; it != a.end() ; it++)
        cout << *it << ' ' ;
    cout << endl ;
}
void print(vector<float>& a){
    for(auto it = a.begin() ; it != a.end() ; it++)
        cout << *it << ' ' ;
    cout << endl ;
}
void print(vector<double>& a){
    for(auto it = a.begin() ; it != a.end() ; it++)
        cout << *it << ' ' ;
    cout << endl ;
}
```

但这样看起来很冗余，换用模板：

```cpp
template <typename T>
void print(T a){
    for(auto it = a.begin() ; it != a.end() ; it++)
        cout << *it << ' ' ;
    cout << endl ;
}
```

[[notes of 《C++ Primer》]]
#cpp