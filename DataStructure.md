---
title: DataStructure
date: 2023-10-21 00:09:08
tags:
---

<p style = "color : #f03752 ; font-size : 24px">It's the note I made when learned data structure . </p>

Ref : [数据结构与算法基础（青岛大学-王卓）_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1nJ411V7bd/?spm_id_from=333.337.search-card.all.click&vd_source=da5120fea3f8bb8d2fe1984a02a9a745)

Source code : [BenkangPeng/datastructure](https://github.com/BenkangPeng/datastructure) 

<!-- more -->

# Representation and Implementation of ADT

**EX :  Arithmetic Operations on Complex Numbers**

```cpp
#include <iostream>
using namespace std ;
typedef struct 
{
    float real;
    float imag;
} Complex;
void assign(Complex &A, float real, float imag) {
    A.real = real;
    A.imag = imag;
}
void print(Complex A){
    cout << A.real << '+' << A.imag << 'i' << endl ;
}
void add(Complex &A , Complex B , Complex C){
    A.real = B.real + C.real ;
    A.imag = B.imag + C.imag ;
}
void minus(Complex &A , Complex B , Complex C){
    A.real = B.real - C.real ;
    A.imag = B.imag - C.imag ;
}
void multiply(Complex &A , Complex B , Complex C){
    A.real = B.real * C.real - B.imag * C.imag ;
    A.imag = B.real * C.imag + B.imag * C.real ;
}
void divide(Complex &A , Complex B , Complex C){
    A.real = (B.real * C.real + B.imag * C.imag) / (C.real * C.real + C.imag * C.imag) ;
    A.imag = ( - B.real * C.imag + B.imag * C.real ) / (C.real * C.real + C.imag * C.imag) ;
}
int main() {
    Complex A , B , C ;
    assign(A , 1 , 2);
    assign(B , 3 , 4) ;
    assign(C , 5 , 6) ;
    divide(A , B , C) ;
    print(A) ;
    return 0;
}
```

# Linear List

## Squence Linear List

### **EX1. Horner's rules or Qin Jiushao(秦九韶)'s algorithm**

```cpp
#include<iostream>
#include<cmath>
#include<ctime>
#define N 9 
#define K 1e7 // numbers of loop
double a[10] = { 2.1 , 3.4 , 2 , 1.3 , 3 , 8.8 , 4.3 , 5.3 , 5.3 , 4.3} ;
double func1(double x , double *a ){
    double res = a[0] ;
    for(int i = 1 ; i <= N ; i++){
        res += a[i] * pow(x , i) ;
    }
    return res ;
    //std:: cout << "the result of polynominal by func1 is " << res << std::endl ; 
}
double func2(double x , double *a){
    // Qin Jiushao's algorithm
    double res = a[9] ;
    for(int i = N ; i > 0 ; i--){
        res = a[i - 1] + res * x ;
    }
    return res ;
    //std::cout << "the resulr of polynominal by func2 is " << res << std::endl ;
}
void time_func1(){
    clock_t start , stop ;
    start = clock() ;
    for(int i = 0 ; i < K ; i++){
        func1( 2.4 , a ) ;   
    }
    stop = clock() ;
    std::cout << "func1 cost " << (double(stop - start)) / CLK_TCK / K << " s per time" << std::endl ;
}
void time_func2(){
    clock_t start , stop ;
    start = clock() ;
    for(int i = 0 ; i < K ; i++){
        func2( 2.4 , a ) ;   
    }
    stop = clock() ;
    std::cout << "func2 cost " << (double(stop - start)) / CLK_TCK / K  << " s per time" << std::endl ;
}
int main(){
    double x ; 
    std::cin >> x ; 
    std::cout << "the result of func1 is " << func1(x , a) << std::endl ;    
    std::cout << "the result of func2 is " << func2(x , a) << std::endl ;    
    func2(x , a) ;
    time_func1() ;
    time_func2() ;
}
```

### C++ dynamic memory allocation

```cpp
int* p = new int ;
int* p2 = new int[10] ; //allocate 10 int memory to p2
delete p , p2;
```

### Convenient way of passing values in C++

```cpp
void swap_1(int& a , int& b){
    int temp = a ;
    a = b ;
    b = temp ;
}
int main(){
    int a = 2 , b = 9 ;
    swap_1(a , b) ;
    cout << "a = " << a << " b = " << b << endl ;
    return 0 ;
}
```

### Representation of LinearList

```cpp
#include<iostream>
#define ElementType int 
#define MaxSize 20 
using namespace std ;
typedef struct SqList{
    ElementType  *element ;
    int length ;
}SqList ;


void InitList_Sq(SqList &L){
    //Initial the Squence Linear List
    L.element = new ElementType[MaxSize] ;
    if(!L.element)  {
        exit(0);
        L.length = 0 ;
    }
}
void DestroyList(SqList &L){
    if(L.element)   delete L.element ;
}
void ClearList(SqList &L){
    L.length = 0 ;
}
int GetLength(SqList &L){
    return  L.length ;
}
int IsEmpty(SqList &L){
    if(L.length)    return 0 ;
    else    return 1 ;
}
int GetEle(SqList &L , int i , ElementType &e){
    if(i < 1 || i > L.length)   return 0 ;
    else e = L.element[i] ;
    return 1 ;
}
int LocateEle (SqList &L , ElementType e){
    for(int i = 0 ; i < L.length ; i++){
        if(L.element[i] == e)   return i ;
    }
    return 0 ;
}
int ListInsert_Sq(SqList &L , ElementType e , int i ){
    if(i < 0 || i > L.length || (L.length + 1) > MaxSize)   return 0 ;
    for(int j = L.element[L.length - 1] ; j >= i ; j--){
        L.element[j + 1] = L.element[j] ;
    }
    L.element[i] = e ;
    L.length++ ;
    return 1 ;
}
void PrintList_Sq(SqList &L){
    for(int i = 0 ; i < L.length ; i++){
        cout << L.element[i] << " " ;
    }
    cout << endl ;
}
int ListDelete_Sq(SqList &L , int i ){
    if(i < 0 || i > L.length - 1) return 0 ;
    for(int j = i + 1 ; j < L.length ; j++){
        L.element[j - 1] = L.element[j] ; 
    }
    L.length-- ;
    return 1 ;
}
void union_list(SqList &La , SqList Lb){
    int length = GetLength(Lb) ;
    for(int i = 0 ; i < GetLength(Lb) ; i++){
        if(LocateEle(La , Lb.element[i]) == -1) {
            ListInsert_Sq(La , Lb.element[i] , GetLength(La)) ;
        }
    }
}
void merge_list_Sq(SqList &Lc , SqList La , SqList Lb ){
    int length_a = GetLength(La) , length_b = GetLength(Lb) ;
    Lc.length = length_a + length_b ;
    Lc.element = new ElementType[Lc.length] ;
    int i = 0 , j = 0 , k = 0 ;
    while(i < length_a && j < length_b){
        if(La.element[i] < Lb.element[j]){
            Lc.element[k++] = La.element[i++] ;
        }
        else Lc.element[k++] = Lb.element[j++] ;
    }
    while(i < length_a) Lc.element[k++] = La.element[i++] ;
    while(j < length_b) Lc.element[k++] = Lb.element[j++] ;
}
int main(){
    SqList L ;
    InitList_Sq(L) ; 
    int a[5] = {1 , 3 , 5 , 7 , 9} ;
    for(int i = 0 ; i < 5 ; i++){
        L.element[i] = a[i] ;
    }
    L.length = 5 ;
    ListDelete_Sq(L , 4) ;
    PrintList_Sq(L) ;
}
```

**Advantages and Disadvantages of Squence Linear List :**

**Advantages :**

* **Storing physical location relationships represents logical relationships .**
* **Can get any elements quickly .**

**Disadvantage:**

* **Too many steps to move elements  in the option of Insert and Delete elements .**

## Linked Linear List

I write a new article about linked linear list [HERE](https://benkangpeng.github.io/posts/14fa845f/)

# Stack and Queue

```cpp
#include<iostream>
#define MAXSIZE 20
using namespace std ;
typedef struct stack{
    int *base , *top ;
    int stack_size ;
}stack ;
void init_stack(stack &s){
    s.base = new int[MAXSIZE] ;
    if(!s.base) exit(-1) ;//overflow
    s.top = s.base ;
    s.stack_size = MAXSIZE ;
}
int is_empty(stack s){
    return s.top == s.base ;
}
int get_stack_length(stack s){
    return s.top - s.base ;
}
void clear_stack(stack s){
    if(s.base)  s.top = s.base ;
}
void destroy_stack(stack s){
    if(s.base){
        delete s.base ;
        s.stack_size = 0 ;
        s.top = s.base = NULL ;
    }
}
void push(stack &s , int e){
    if(s.top - s.base == s.stack_size)  exit(-1) ; // stack is filled .
    *s.top++ = e ; 
}
int pop(stack &s){
    if(s.top == s.base) exit(-1) ;
    return *--s.top ;
}
int main(){
    stack s ;
    init_stack(s) ;
    for(int i = 0 ; i < 10 ; i++){
        push(s , i) ;
    }
    for(int i = 0 ; i < 10 ; i++){
        cout << pop(s) << "  " ;
    }
    return  0 ;
}
```

循环队列：

* `rear = (rear + 1) % MAXSIZE` 
* 空出一个位置来区分队列满or空 ： 空 ： `front == rear`  ,  满 ： `(rear + 1)%MAXSIZE == top`

```cpp
#include<iostream>
#define MAXSIZE 20
using namespace std ;
typedef struct queue{
    int *elem ; 
    int front , rear ;
}queue ;
void init_queue(queue &q){
    q.elem = new int[MAXSIZE] ;
    q.front = q.rear = 0 ;
}
int get_length(queue q){
    return (q.rear - q.front + MAXSIZE) % MAXSIZE ;
}
void enter_queue(queue &q , int e){
    if((q.rear + 1)%MAXSIZE == q.front) {cout << "queue is filled" << endl ; exit(-1) ;} 
    q.elem[q.rear] = e ;
    q.rear = (q.rear + 1) % MAXSIZE ;

}
int exit_queue(queue &q){
    if(q.rear == q.front)   {cout << "queue is empty" << endl ; exit(-1) ; }
    int e = q.elem[q.front] ;
    q.front = (q.front + 1) % MAXSIZE ;
    return e ;
}
int main(){
    //循环队列
    queue q ;
    init_queue(q) ;
}
```

**LInked queue**:

```cpp
#include<iostream>
using namespace std ;
typedef struct Qnode{
    int data ;
    struct Qnode *next ;
}Qnode , *Qnode_pointer ;
typedef struct linked_queue{
    Qnode_pointer front ;
    Qnode_pointer rear ;
}linked_queue ;
void init_queue(linked_queue &queue){
    queue.front = queue.rear = new Qnode ;
    queue.front->next = NULL ;
}
void destroy_queue(linked_queue &queue){
    while(queue.front){
    Qnode_pointer p = queue.front;
    queue.front = queue.front->next ; 
    delete queue.front ;
    }
}
void insert(linked_queue &queue , int x ){
    Qnode *p = new Qnode ;
    p->data = x ;
    queue.rear->next = p ;
    p->next = NULL ;
    queue.rear = p ;
}
int pop(linked_queue &queue){
    if(queue.front == queue.rear)   {cout << "the queue is empty" << endl ; return -1 ;}
    int x = queue.front->next->data ;
    Qnode_pointer p = queue.front ;
    queue.front = queue.front->next ;
    delete p ;
    return x ;
}
void print_queue(linked_queue queue){
    Qnode_pointer p = queue.front->next ;
    while (p)
    {
        cout << p->data << ' ' ;
        p = p->next ;
    }

}
int main(){
    linked_queue queue ;
    init_queue(queue) ;
    for(int i = 0 ; i < 10 ; i++){
        insert(queue , i) ;
    }
    print_queue(queue) ;
    for(int i = 0 ; i < 5 ; i++){
        pop(queue) ;
    }
    print_queue(queue) ;
}
```

## 

**String , Array and** 

```cpp
int index_BF(string s , string t){
    // Brute force 
    int s_length = s.length() , t_length = t.length() ;
    int i = 0 , j = 0 ;
    while(i < s_length && j < t_length ){
        if(s[i] == t[j]){
            ++i ; 
            ++j ;
        }
        else{
            i = i - j + 1 ;
            j = 0 ;
        }
    }
    if(j >= t_length) return i - j ;
    return -1 ;

}
```

```cpp
//KMP
int* get_next(string t){
    int len = t.length() ;
    int *next = new int[len] ;
    next[1] = 0 ;
    int i = 1 , j = 0 ;
    while (i < len){
        if(j == 0 || t[i-1] == t[j-1])  next[++i] = ++j  ;
        else j = next[j] ;
    }
    return next ;
}
int index_KMP(string s , string t){
    int i = 1 , j = 1 ;
    int s_len = s.length() , t_len = t.length() ;
    int *next = get_next(t) ;
    while(i <= s_len && j <= t_len){
        if(j == 0 || s[i-1] == t[j-1]){
            i++ ;
            j++ ;
        }
        else{
            j = next[j] ;
        }
    }
    if(j > t_len)  return i - t_len - 1;
    return -1 ;
}
```

**It's a little difficult to understand `KMP` , especially getting the `next` array . There are some links maybe help us get it .**:

[KMP算法之求next数组代码讲解_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV16X4y137qw/?spm_id_from=333.337.search-card.all.click&vd_source=da5120fea3f8bb8d2fe1984a02a9a745)

[KMP算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/105629613)
