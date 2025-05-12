### C++测时模板

```cpp
#include<iostream>
#include<chrono>
typedef std::chrono::high_resolution_clock Clock;

int main()
{
    auto start = Clock::now();
    //your test program
    auto end = Clock::now();
    std::cout << "Time taken: "
		<< std::chrono::duration_cast<std::chrono::nanoseconds>(end - start).count()
		<< " nanoseconds" << std::endl;
    //其他时间单位：std::chrono::seconds, std::chrono::milliseconds,
}
```

可以去https://ideone.com/ (geeks for geeks同款)运行程序，排除机器变量干扰。

### C++矩阵运算(运算符重载)

[Link](https://mp.weixin.qq.com/s/C1c5XChVyVT4OSeNvpmaqQ)

```cpp
#include<iostream>

#include<vector>

class Matrix{

    private:

        int rows , cols ;

        std::vector<std::vector<int>> data ;

    public:

        Matrix(int rows, int cols , std::vector<std::vector<int>> data): rows(rows), cols(cols) , data(data){}



        int get_elem(int row , int col){

            return data[row][col];

        }

        void display(){

            for(int i = 0 ; i < rows ; i++){

                printf("\n");

                for(int j = 0 ; j < cols ; j++){

                    printf("%d  " , data[i][j]);

                }

            }

        }

        Matrix operator+(const Matrix& A) const{

            Matrix result(rows , cols , data);

            for(int i = 0 ; i < rows ; i++)

                for(int j = 0 ; j < cols ; j++){

                    result.data[i][j] += A.data[i][j];

                }

            return result;

        }



        Matrix operator-(const Matrix& A) const{

            Matrix result(rows , cols , data);

            for(int i = 0 ; i < rows ; i++)

                for(int j = 0 ; j < cols ; j++){

                    result.data[i][j] -= A.data[i][j];

                }

            return result;

        }



        Matrix operator*(const Matrix& A) const{

            if(cols != A.rows){

                std::cout << "Invalid matrix dimensions" << std::endl;

                exit(-1);

            }

            Matrix result(rows , A.cols , std::vector<std::vector<int>> (rows , std::vector<int>(A.cols,0)));

            for(int i = 0 ; i < rows ; i++)

                for(int j = 0 ; j < A.cols ; j++)

                    for(int k = 0 ; k < cols ; k++){

                        result.data[i][j] += data[i][k] * A.data[k][j];

                    }

            return result;

        }



        Matrix transpose()const{

            Matrix res(cols , rows , data);

            for(int i = 0 ; i < cols ; i++)

                for(int j = 0 ; j < rows ; j++){

                    res.data[i][j] = data[j][i];

                }

                return res;

        }

};



int main()

{

    std::vector<std::vector<int>> data = { {1,2,3} , {4,5,6} , {7,8,9}};

    std::vector<std::vector<int>> data1 = { {1,2,3} , {4,5,6} , {7,8,9} , {6,6,6}};

    std::vector<std::vector<int>> data2 = { {1,2,3,4} , {4,5,6,7} , {7,8,9,10}};

    Matrix A(3,4,data2);

    Matrix B(4,3,data1);

    Matrix C(3,3,data);

    // C = A + B;

    C = A * B;

    A.display();

    B.display();

    C.display();

    C.transpose().display();

}
```

### C++11 tuple

[Link](https://mp.weixin.qq.com/s/oPOqr5J0ES0NGO-L3eKvzA)

```cpp
#include<iostream>
#include<tuple>
#include<string>

int main()
{
    std::string name = "Alice";
    int age = 21;
    double grade = 100.0;
    auto student_tuple = std::make_tuple(name , age , grade);

    std::string name_from_student = std::get<0>(student_tuple);
    int age_of_student = std::get<1>(student_tuple);
    double grade_of_student = std::get<2>(student_tuple);

    std::cout << "name : " << name_from_student << std::endl;
    std::cout << "age  : " << age_of_student    << std::endl;
    std::cout << "grade: " << grade_of_student  << std::endl;
    return 0;
}
```

### 自动补全背后的算法(字典树)

[Link](https://mp.weixin.qq.com/s/L42BCgw-WczbWsiY9C0MeA)

Trie树（又称前缀树或字典树）是一种用于快速检索字符串数据集中的键的树形数据结构。这种结构非常适合处理字符串集合，特别是实现自动补全或拼写检查等功能。下面，我将通过一个具体的例子来解释Trie树的结构和工作原理。

假设我们有一个Trie树，我们要在其中存储单词“CAT”，“CAN”，和“BAT”。Trie树的结构将如下所示：

```
       Root
      /    \
     B      C
     |      |
     A      A
    / \    / \
   T   *  T   N
  /         /
 *         *  
```

在这个图形化表示中：

- 每个节点代表一个字符。
- 根节点（Root）不包含字符，它是所有单词的起始点。
- 每条从根节点到某个标记为“*”的节点的路径代表一个单词。例如，从根节点到第一个“*”的路径是“BAT”。
- 分支表示单词的不同字符。例如，“BAT”和“CAT”共享第一个字符，但第二个字符不同，因此在第二层分开。

现在，如果我们要添加另一个单词比如“CAR”，Trie树将会更新为：

```
       Root
      /    \
     B      C
     |      |
     A      A
    / \    / \
   T   *  T   R
  /       /   |
 *       *    *
              |
              *  
```

在这个新的结构中，“CAT”和“CAR”共享前两个字符“CA”，但第三个字符不同，在“CAR”中为“R”，因此在第三层分叉。

**操作说明**

1. **插入**：当插入一个新单词时，从根节点开始，为单词的每个字符创建一个新的子节点（除非该字符已经存在）。到达单词的最后一个字符时，标记这个节点表示一个单词的结束。
2. **搜索**：为了查找一个单词，从根节点开始，沿着单词的字符移动。如果能够在每一步都找到相应的字符，并且最后一个字符被标记为一个单词的结尾，那么该单词存在于Trie树中。
3. **前缀搜索**：前缀搜索与全词搜索类似，但不需要最后一个字符被标记为单词的结尾。如果能够顺着前缀的字符在树中移动到最后一个字符，那么这个前缀存在于树中。

Trie树在处理大量字符串，尤其是进行快速前缀查找和词汇自动补全时非常有效。由于它基于共享前缀来存储单词，因此相比于其他数据结构，它在空间效率上通常更优。

**TrieNode 设计**

`TrieNode` 用于表示Trie树中的每个节点。接下来详细解释这个代码片段，包括`TrieNode`的设计以及每个成员的作用。

```cpp
class trie_node{
    public:
        bool is_end_of_word;
        std::unordered_map<char , trie_node*> children;

        trie_node(): is_end_of_word(false){}
};
```

1. `bool isEndOfWord;`：这是一个布尔型成员变量，用于标记当前节点是否代表一个单词的结束。如果`isEndOfWord`为`true`，则表示从根节点到当前节点的路径构成一个完整的单词。
2. `unordered_map<char, TrieNode*> children;`：这是一个无序映射（`unordered_map`），用于存储当前节点的子节点。Trie树的一个关键特性是每个节点可以有多个子节点，每个子节点对应一个字符。这个`unordered_map`允许我们将字符映射到相应的子节点。
3. `TrieNode() : isEndOfWord(false) {}`：这是`TrieNode`类的构造函数。构造函数用于初始化新创建的`TrieNode`对象。在这里，构造函数将`isEndOfWord`初始化为`false`，表示节点默认不是单词的结束。

现在让我们通过一个示例来解释如何使用这个`TrieNode`类来构建Trie树：

假设我们要在Trie树中插入单词"CAT"，首先我们创建一个根节点。然后，我们从根节点开始，在每个字符位置上创建一个新的`TrieNode`对象，将`isEndOfWord`设置为`false`。在这个过程中，我们将字符映射到相应的子节点，如下所示：

1. 创建根节点，`isEndOfWord`为`false`。
2. 在根节点下创建字符'C'对应的子节点，`isEndOfWord`为`false`。
3. 在字符'C'对应的子节点下创建字符'A'对应的子节点，`isEndOfWord`为`false`。
4. 在字符'A'对应的子节点下创建字符'T'对应的子节点，`isEndOfWord`为`true`，因为"CAT"的最后一个字符表示单词的结束。

这就是如何使用`TrieNode`类来构建Trie树的一部分。这个类的设计允许我们轻松地在每个节点上附加字符和标记单词的结束。随着插入更多的单词，Trie树的结构会不断扩展。

**完整代码**

这段代码实现了一个称为“Trie”（也被称为前缀树或字典树）的数据结构:

```cpp
#include<iostream>
#include<unordered_map>
#include<string>
class trie_node{
    public:
        bool is_end_of_word;
        std::unordered_map<char , trie_node*> children;

        trie_node(): is_end_of_word(false){}
};

class trie{
    private:
        trie_node* root;
        //Recursively cleans up memory
        void clear_memory(trie_node* node){
            for(auto pair : node->children)
                clear_memory(pair.second);
            delete node;
        }
    public:
        trie(){
            root = new trie_node;
        }

        void insert(std::string word){
            trie_node* current = root;
            for(char ch : word){
                if(current->children.find(ch) == current->children.end()){
                    current->children[ch] = new trie_node;
                }
                current = current->children[ch];
            }
            current->is_end_of_word = true;
        }
        //search whether word is in trie tree or not
        bool search(std::string word){
            trie_node* current = root;
            for(char ch : word){
                if(current->children.find(ch) == current->children.end())
                    return false;
                current = current->children[ch]; 
            }
            return current != nullptr && current->is_end_of_word;
        }

        //search whether prefix is the prefix of the word in trie tree or not
        bool start_with(std::string prefix){
            trie_node* current = root;
            for(char ch : prefix){
                if(current->children.find(ch) == current->children.end()){
                    return false;
                }
                current = current->children[ch];
            }
            return true;
        }


        ~trie(){
            clear_memory(root);
        }

};

int main()
{
    trie trie_tree;
    trie_tree.insert("nudt");
    trie_tree.insert("hust");
    std::cout << trie_tree.search("hust") << std::endl;
    std::cout << trie_tree.start_with("nud") << std::endl;
    return 0;
}
```

### RAII and smart pointers

[reference1](https://stackoverflow.com/questions/2321511/what-is-meant-by-resource-acquisition-is-initialization-raii)

[reference2](https://www.geeksforgeeks.org/smart-pointers-cpp/)

`RAII`即`Resource Acquisition is Initialization` , 中文翻译叫`资源获取即初始化` .其实这种叫法很容易让人疑惑，摘一个stackoverflow上的回答:

> It's a really terrible name for an incredibly powerful concept, and perhaps one of the number 1 things that C++ developers miss when they switch to other languages. There has been a bit of a movement to try to rename this concept as **Scope-Bound Resource Management**, though it doesn't seem to have caught on just yet.

叫`Scope-Bound Resource Management`可能更合适。(即范围限制资源管理)

> When we say 'Resource' we don't just mean memory - it could be file handles, network sockets, database handles, GDI objects... In short, things that we have a finite supply of and so we need to be able to control their usage. The 'Scope-bound' aspect means that the lifetime of the object is bound to the scope of a variable, so when the variable goes out of scope then the destructor will release the resource. A very useful property of this is that it makes for greater exception-safety. For instance, compare this:

资源的使用一般经历三个步骤a.获取资源 b.使用资源 c.销毁资源，但是资源的销毁往往是程序员经常忘记的一个环节，所以程序界就想如何在程序员中让资源自动销毁呢？c++之父给出了解决问题的方案：RAII，它充分的利用了C++语言局部对象自动销毁的特性来控制资源的生命周期。`RAII`就是一种编程思想，获取资源(例如声明指针,打开文件)必须在一个范围(生命周期)内，离开范围(生命周期)后需要释放资源。

举一个例子(虽然很生硬):

```cpp
#include<iostream>
#include<cstdlib>
#include<random>
void fun(){
    int* array = new int[100];
    for(int i = 0 ; i < 100 ; i++)
        array[i] = rand();
    //use the numbers in array to do something.....
    //forget to delete array: delete[] array;
}
int main(){
    for(int i = 0 ; i < 1e6 ; i++)
        fun();
}
```

以上程序可能会发生内存泄露，原因是`fun()`内的指针`array`没有被释放，应该在`fun()`最后加一句`delete array;`

实际上，忘记释放资源的事在大规模程序中经常发生(在一些调用过程中甚至都意识不到)。如何去避免这种情况，一种便是如履薄冰，每次申请资源时便配套释放资源;一种则是让资源自动释放，即资源出了生命周期后便自动释放。

这种情景就类似于类的析构函数了，因此`RAII`的实现经常是借助类完成的，即构建一个资源类，离开生命周期后会自动调用析构函数释放资源。用`RAII`思想去重写以上代码:

```cpp
#include<iostream>
#include<cstdlib>
#include<random>
void fun(){
    int* array = new int[100];
    for(int i = 0 ; i < 100 ; i++)
        array[i] = rand();
    //use the numbers in array to do something.....

}
class array{
    public:
        int* array_;
        array(){
            array_ = new int[100];
            for(int i = 0 ; i < 100 ; i++)
                array_[i] = rand();
        }

        void fun(){
            //use array_ do something

        }

        ~array(){
            if(array_ != nullptr){
                delete[] array_;
                array_ = nullptr;
            }
        }
};
int main(){
    for(int i = 0 ; i < 1e6 ; i++){
        //fun();//memory leak
        array _array;
        _array.fun();
    }

    return 0;  
}
```

**smart pointer**

c++11引入的智能指针就是`RAII`思想的最好体现

`shared_ptr`  `unique_ptr`  `weak_ptr`

以上示例函数`fun()`可以用智能指针重写为:

```cpp
void _fun(){
    std::shared_ptr<int> arr(new int[100]);
    for(int i = 0 ; i < 100 ; i++){
        arr.get()[i] = rand();
    }
    //use arr to do something

    //delete arr automatically
}
```

* 智能指针需要用`.get()`得到原来的指针(retrieve the raw pointer stored within a `shared_ptr`)
* 智能指针包含在`<memory>`库中
* 初次使用智能指针时Vscode解释器可能会检出错误，可能是Vscode检查器用的c++版本低于c++11,可以重新生成一下`c_cpp_properties.json`,检查其中的`cppStandard`

```cpp
#include<iostream>
#include<memory>
class recangle{
    public:
        int length , breadth;
        recangle(int l , int b): length(l) , breadth(b){}

        int area(){
            return this->length * this->breadth ;
        }
};

class circle{
    public:
        int radius;
        circle(int r):radius(r){}

        int area(){ return 3.14 * this->radius * this->radius;}
};

int main(){
    std::shared_ptr<recangle> p1(new recangle(6,8));
    std::shared_ptr<recangle> p2(p1);
    std::weak_ptr<recangle> p2_(p2);
    std::cout << p2->area() << std::endl;
    std::cout << p2.use_count() << std::endl;

    std::unique_ptr<recangle> p3(new recangle(10,10));
    // std::unique_ptr<recangle> p4 = p3;//error:include\c++\bits\unique_ptr.h") cannot be referenced

    std::unique_ptr<recangle> p4 = move(p3);
    std::cout << p4->area() << std::endl;
    // std::shared_ptr<int> p5 = move(p4);//error:no suitable user-defined conversion from "std::unique_ptr<recangle, std::default_delete<recangle>>" to "std::shared_ptr<int>" existsC/C++(312)

}
```

* `unique_ptr`声明的指针独自占有一段资源，不允许其他指针指向这段资源
* 不能将`unique_ptr`的使用权转交给`shared_ptr` , 只能转给`unique_ptr`
* 能用`.use_count()`得出资源被指向的指针数，即有几根`shared_ptr`指向该资源
* `weak_ptr`提出的原因:

> Weak_ptr is a smart pointer that holds a non-owning reference to an object. It’s much more similar to shared_ptr except it’ll not maintain a **Reference Counter**. In this case, a pointer will not have a stronghold on the object. The reason is if suppose pointers are holding the object and requesting for other objects then they may form a **Deadlock.** 

`weak_ptr`指向资源而不占用资源，不增加`reference counter`   (`.use_count()`)

这是为了防止`shared_ptr`发生死锁(`deadlock`)

**Deadlock**:

> 死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去。这种情况在多线程编程中尤为常见，当两个或更多的线程**互相等待对方释放资源**时，就可能发生死锁。
> 
> 在C++中，shared_ptr智能指针可能会导致死锁，特别是当两个shared_ptr对象互相引用时。例如，假设我们有两个类A和B，它们互相持有**对方的shared_ptr**，这就可能导致死锁。因为每个shared_ptr都在等待对方释放资源，但由于它们都在等待，所以没有一个shared_ptr会释放资源，从而导致死锁

```cpp
class A;
class B;

class A {
public:
    shared_ptr<B> pb_;
    //应修改成: weak_ptr<B> pb_;
};

class B {
public:
    shared_ptr<A> pa_;
};

void test() {
    shared_ptr<A> pa(new A());
    shared_ptr<B> pb(new B());
    pa->pb_ = pb;
    pb->pa_ = pa;
}
```

学了`RAII`后，想着去重构以前的项目，尽量用智能指针，但发现还是不能完全替代:

* 智能指针一旦指向了某块资源，该智能指针也无法再修改。这意味着不能像裸指针一样可传入函数、返回参数。例如在写链表时用智能指针反而会有很多麻烦，因为你不知道什么时候智能指针就把你的结点释放了:joy: 基于此，智能指针**不适合传入函数、被函数(或类)反复调用**，因为你不知道什么时候他会被释放。
* 尝试过后，仍然觉得C++的指针还是比较难控制的。所以尽量少用指针(尽量不用`new` )吧，用一些成熟的符合`RAII`规范的容器(`link`,`vecter`,`string`.et)与配套的迭代器代替.如果实在要用指针，一定慎之又慎。:cry:  **C++不喜欢菜鸟**

PS:尝试用`list`实现链表(造链表轮子是内存泄漏的重灾区)

```cpp
#include<iostream>
#include<list>
using namespace std;
template<typename T>
class linked_list{
    public:
        list<T> L;

        T operator[] (int idx){
            auto it = L.begin();
            advance(it,idx);
            return *it;
        }

        int _insert(T elem , int idx){
            if(idx < 0 || idx > L.size()){cout << "please input a valid idx\n"; return -1;}
            auto it = L.begin();
            advance(it,idx);
            L.insert(it,elem);
            return 0;
        }

        void push_back(T elem){
            L.push_back(elem);
        }

        bool is_empty(){
            return !L.size();
        }

        void delete_elem(T elem){
            L.remove(elem);
        }

        void clear(){
            L.clear();
        }

        void show(){
            for(auto i : L){
                cout << i << " ";
            }
            cout << endl;
        }
};
int main(){
    linked_list<int> _L;
    for(int i = 0 ; i < 30 ; ++i){
        _L.push_back(i);
    }

    _L._insert(666,16);
    _L.delete_elem(19);
    _L.show();
    cout << _L[6];

    return 0;
}
```





### Virtual function & 多态

虚函数是实现类多态的一种方法：

```cpp
// C++ program to illustrate
// concept of Virtual Functions

#include <iostream>
using namespace std;

class base {
public:
	virtual void print() { cout << "print base class\n"; }

	void show() { cout << "show base class\n"; }
};

class derived : public base {
public:
	void print() { cout << "print derived class\n"; }//对base类中的print方法进行重载

	void show() { cout << "show derived class\n"; }//base中show()方法不是虚函数，无法重载
};

int main()
{
	base* bptr = new derived;

	// Virtual function, binded at runtime
	bptr->print();//print derived class\n

	// Non-virtual function, binded at compile time
	bptr->show();//show base class\n

	return 0;
}
```

* 每个包含虚函数的类都有一个虚函数表（vtable），这是为了支持运行时多态。虚函数表是一个指向函数指针的数组，这些函数指针对应于类中的每个虚函数。通过这个机制，程序可以在运行时根据对象的实际类型来调用正确的函数。
* **VTable & VPtr :** 
  1. **虚拟函数表（VTable）**：当一个类中声明了虚函数（virtual method），编译器会为这个类创建一个虚拟函数表（VTable）。这个表中存储了类中所有虚函数的地址。
  2. **虚拟指针（VPtr）**：编译器还会为每个类实例创建一个虚拟指针（VPtr），并将其初始化为指向该类的VTable。
  3. **VTable的共享**：一个类的VTable是所有该类实例共享的，也就是说，编译器只为这个类创建一个VTable实例，然后所有该类的对象都共享这个VTable。
  4. **VPtr的独立性**：尽管VTable是共享的，但每个类实例都有自己的VPtr。
  5. **类对象大小**：如果一个类对象至少包含一个虚函数，那么这个对象的大小将是该类数据成员的大小加上VPtr的大小。在C++中，可以通过sizeof操作符来获取这个大小
* 多态函数的解析是在运行时(runtime)完成的。(不像一般函数是在编译时完成解析。)
* 虚函数不能是静态函数(static)
* 应该使用基类类型的**指针或引用**来**访问虚函数**，以实现运行时多态性。
* 指向`derived`的指针可以是`base`(基类)类型 (`base* bptr = new derived;`) , 在执行`bptr->print()`时首先会尝试执行`base类`中的`print`方法，当发觉`base::print()`函数是虚函数时，会从虚函数地址表中查找`derived::print()`的地址去执行`derived::print()`。而`base::show()`不是虚函数，编译器不会从虚函数表中查找`derived::show()`(虚函数表中也没有该地址)，因此`bptr->show`执行的是`base::show()`。
* 下面是一个体现运行时多态的便利性的例子：

```cpp
#include <iostream>

// 基类 Shape
class Shape {
public:
    // 虚析构函数确保派生类的资源被正确释放
    virtual ~Shape() {}

    // 纯虚函数 Draw，定义了一个接口
    virtual void Draw() const = 0;
};

// 派生类 Circle
class Circle : public Shape {
public:
    void Draw() const override {
        std::cout << "Drawing a circle." << std::endl;
    }
};

// 派生类 Rectangle
class Rectangle : public Shape {
public:
    void Draw() const override {
        std::cout << "Drawing a rectangle." << std::endl;
    }
};
int main() {
    // 创建一个 Shape 指针数组
    Shape* shapes[2];
    shapes[0] = new Circle();
    shapes[1] = new Rectangle();

    // 通过 Shape 指针调用 Draw 函数
    for (auto shape : shapes) {
        shape->Draw();  // 动态绑定到正确的 Draw 实现
    }

    // 清理资源
    for (auto shape : shapes) {
        delete shape;
    }

    return 0;
}
```

- 在 `main` 函数中，我们创建了一个 `Shape` 指针数组 `shapes`，并将 `Circle` 和 `Rectangle` 对象的地址存入其中。
- 当我们通过 `Shape` 指针调用 `Draw()` 时，实际上会调用 `Circle` 或 `Rectangle` 中的 `Draw()` 方法。这是因为 `Draw()` 是虚函数，编译器会在运行时根据对象的实际类型来决定调用哪个版本的 `Draw()`
- 如果将来需要添加新的图形类型（例如 `Triangle`），只需要创建一个新的派生类并实现 `Draw()` 方法即可。不需要修改现有的代码，就可以将新的图形类型加入到系统中。这种设计使得系统更加灵活和易于维护。

**但虚函数带来的查表开销也是较大的，因此可以通过CRTP这种设计模式来避免**

### CRTP(Curiously recurring template pattern)

将派生类作为模板参数传入基类

[reference](https://www.geeksforgeeks.org/curiously-recurring-template-pattern-crtp-2/)

以下的例子展示了当虚函数被大量调用时所带来的开销：

```cpp
// A simple C++ program to demonstrate run-time
// polymorphism
#include <chrono>
#include <iostream>
using namespace std;

typedef std::chrono::high_resolution_clock Clock;

// To store dimensions of an image
class Dimension {
public:
	Dimension(int _X, int _Y)
	{
		mX = _X;
		mY = _Y;
	}

private:
	int mX, mY;
};

// Base class for all image types
class Image {
public:
	virtual void Draw() = 0;//虚函数等于0，表示该类的任何派生类都要重新定义Draw()
	virtual Dimension GetDimensionInPixels() = 0;

protected:
	int dimensionX;
	int dimensionY;
};

// For Tiff Images
class TiffImage : public Image {
public:
	void Draw() {}
	Dimension GetDimensionInPixels()
	{
		return Dimension(dimensionX, dimensionY);
	}
};

// There can be more derived classes like PngImage,
// BitmapImage, etc

// Driver code that calls virtual function
int main()
{
	// An image type
	Image* pImage = new TiffImage;

	// Store time before virtual function calls
	auto then = Clock::now();

	// Call Draw 1000 times to make sure performance
	// is visible
	for (int i = 0; i < 1000; ++i)
		pImage->Draw();

	// Store time after virtual function calls
	auto now = Clock::now();

	cout << "Time taken: "
		<< std::chrono::duration_cast<std::chrono::nanoseconds>(now - then).count()
		<< " nanoseconds" << endl;

	return 0;
}

```

Output : 

```
Time taken: 2613 nanoseconds
```

When a method is declared virtual, compiler secretly does two things for us: 

1. Defines a VPtr in first 4 bytes of the class object
2. Inserts code in constructor to initialize VPtr to point to the VTable



**采用CRTP方法来实现静态多态，减去虚函数表的动态开销** ：

```cpp
// Image program (similar to above) to demonstrate
// working of CRTP
#include <chrono>
#include <iostream>
using namespace std;

typedef std::chrono::high_resolution_clock Clock;

// To store dimensions of an image
class Dimension {
public:
	Dimension(int _X, int _Y)
	{
		mX = _X;
		mY = _Y;
	}

private:
	int mX, mY;
};

// Base class for all image types. The template
// parameter T is used to know type of derived
// class pointed by pointer.
template <class T>
class Image {
public:
	void Draw()
	{
		// Dispatch call to exact type
		static_cast<T*>(this)->Draw();
	}
	Dimension GetDimensionInPixels()
	{
		// Dispatch call to exact type
		static_cast<T*>(this)->GetDimensionInPixels();
	}

protected:
	int dimensionX, dimensionY;
};

// For Tiff Images
class TiffImage : public Image<TiffImage> {
public:
	void Draw()
	{
		// Uncomment this to check method dispatch
		// cout << "TiffImage::Draw() called" << endl;
	}
	Dimension GetDimensionInPixels()
	{
		return Dimension(dimensionX, dimensionY);
	}
};

// There can be more derived classes like PngImage,
// BitmapImage, etc

// Driver code
int main()
{
	// An Image type pointer pointing to Tiffimage
	Image<TiffImage>* pImage = new TiffImage;

	// Store time before virtual function calls
	auto then = Clock::now();

	// Call Draw 1000 times to make sure performance
	// is visible
	for (int i = 0; i < 1000; ++i)
		pImage->Draw();

	// Store time after virtual function calls
	auto now = Clock::now();

	cout << "Time taken: "
		<< std::chrono::duration_cast<std::chrono::nanoseconds>(now - then).count()
		<< " nanoseconds" << endl;

	return 0;
}
```

Output :

```
Time taken: 732 nanoseconds
```

* `Image<TiffImage>* pImage = new TiffImage;`  
* 传入基类`Image`的模板参数为`TiffImage`，那么`pImage`的`draw()`函数为`static_cast<TiffImage*>(this)->Draw();`

执行`pImage->Draw();`时，编译器会先找`Image`类，执行`Image::Draw()`  : `static_cast<TiffImage*>(this)->GetDimensionInPixels();`将`this`指针转成`TiffImage*`型，再去访问`TiffImage::GetDimensionInPixels()`

* CRTP是静态绑定，它通过模板实现多态，在编译时就已经确定了要调用的具体函数，而不是在运行时。

**CRTP实现类计数**

```cpp
#include<iostream>
template<typename CountedType>
class objectCounter{
public:
    inline static std::size_t count = 0;//必须加上inline才能在类中对静态变量赋值

    objectCounter(){
        ++count;
    }

    objectCounter(objectCounter<CountedType> const&){
        ++count;
    }

    objectCounter(objectCounter<CountedType> const&& ){
        ++count;
    }

    ~objectCounter(){
        --count;
    }

public:
    static std::size_t live(){
        return count;
    }
};

template<typename CharT>
class MyString : public objectCounter< MyString<CharT> >{
public:
    MyString(){}

    ~MyString(){}
};
int main(){
    MyString<char> s1, s2;

    MyString<char> s3(s2);

    MyString<wchar_t> ws1, ws2;
    
    std::cout << "num of MyString<char>: " << MyString<char>::live() << '\n';
    std::cout << "num of MyString<wchar_t>: " << MyString<wchar_t>::live() << '\n';
}
```

`objectCounter`是接受一个模板参数`CountedType`的计数器。它用于记录`CountedType`这个类的实例个数。

`MyString`是一个自定义的字符串class，接受一个字符类型`CharT`作为模板参数。它继承自`objectCounter< MyString<CharT> >`，使得`objectCounter`可以对`MyString<CharT>`进行计数。

模板的实例化：理解这个问题时， 可以想象编译器会实例化哪些模板。`MyString<char>`会实例化两个模板：`objectCounter<MyString<char>>` and `MyString<char>`。调用`MyString<char>`的构造函数后，会隐式调用`objectCounter< MyString<char>>`的构造函数，从而使`MyString<char>>::count`加一。

类的实例化：`MyString<char>`继承自`objectCounter< MyString<char>>` , 因此`MyString<char>`的构造函数、拷贝构造函数、移动构造函数、析构函数都会隐式调用父类相应的函数，从而实现类的实例计数。







