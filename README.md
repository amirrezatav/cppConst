# Const Correctness  
## What is “const correctness”?
+ A good thing.
+ It means using the keyword const to prevent const objects from getting mutated.

For example, if you wanted to create a function f() that accepted a std::string, plus you want to promise callers not to change the caller’s std::string that gets passed to f(), you can have f() receive its std::string parameter…

+ `void f1(const std::string& s);      // Pass by reference-to-const`
+ `void f2(const std::string* sptr);   // Pass by pointer-to-const`
+ `void f3(std::string s);             // Pass by value`
In the pass by reference-to-const and pass by pointer-to-const cases, any attempts to change the caller’s std::string within the f() functions would be flagged by the compiler as an error at compile-time. This check is done entirely at compile-time: there is no run-time space or speed cost for the const. In the pass by value case (f3()), the called function gets a copy of the caller’s std::string. This means that f3() can change its local copy, but the copy is destroyed when f3() returns. In particular f3() cannot change the caller’s std::string object.

As an opposite example, suppose you wanted to create a function g() that accepted a std::string, but you want to let callers know that g() might change the caller’s std::string object. In this case you can have g() receive its std::string parameter…

+ `void g1(std::string& s);      // Pass by reference-to-non-const`
+ `void g2(std::string* sptr);   // Pass by pointer-to-non-const`

The lack of const in these functions tells the compiler that they are allowed to (but are not required to) change the caller’s std::string object. Thus they can pass their std::string to any of the f() functions, but only f3() (the one that receives its parameter “by value”) can pass its std::string to g1() or g2(). If f1() or f2() need to call either g() function, a local copy of the std::string object must be passed to the g() function; the parameter to f1() or f2() cannot be directly passed to either g() function. E.g.,
+ Error : 
```cpp
#include <iostream>
using namespace std;

int change(int &  a)
{
    int b = a;
    return b;
}
int main()
{
    const int a = 5;
    int b = 5;
    cout << change(a); // error
    cout << change(b);
}
 ```
 + Compiled :
```cpp
#include <iostream>
using namespace std;

int change(const int &  a)
{
    int b = a;
    return b;
}
int main()
{
    const int a = 5;
    int b = 5;
    cout << change(a);
    cout << change(b);
}
```
+ Error : 
```cpp
#include <iostream>
using namespace std;

int change(const int &  a)
{
    int b = a;
    a++; // error
    return b;
}
int main()
{
    int b = 5;
    cout << change(a);
    cout << change(b);
}
 ```
+ Conplied :
```cpp
#include <iostream>
using namespace std;

int change(const int &  a)
{
    int b = a;
    b++; 
    return b;
}
int main()
{
    int b = 5;
    cout << change(a);
    cout << change(b);
}
```
## How is “const correctness” related to ordinary type safety?
Declaring the const-ness of a parameter is just another form of type safety.

If you find ordinary type safety helps you get systems correct (it does; especially in large systems), you’ll find const correctness helps also.

The benefit of const correctness is that it prevents you from inadvertently modifying something you didn’t expect would be modified. You end up needing to decorate your code with a few extra keystrokes (the const keyword), with the benefit that you’re telling the compiler and other programmers some additional piece of important semantic information — information that the compiler uses to prevent mistakes and other programmers use as documentation.

Conceptually you can imagine that const std::string, for example, is a different class than ordinary std::string, since the const variant is conceptually missing the various mutative operations that are available in the non-const variant. For example, you can conceptually imagine that a const std::string simply doesn’t have an assignment operator += or any other mutative operations.
## What is a “const member function”?
A member function that inspects (rather than mutates) its object.

A const member function is indicated by a const suffix just after the member function’s parameter list. Member functions with a const suffix are called “const member functions” or “inspectors.” Member functions without a const suffix are called “non-const member functions” or “mutators.”
```cpp
class Fred {
public:
  void inspect() const;   // This member promises NOT to change *this
  void mutate();          // This member function might change *this
};
void userCode(Fred& changeable, const Fred& unchangeable)
{
  changeable.inspect();   // Okay: doesn't change a changeable object
  changeable.mutate();    // Okay: changes a changeable object
  unchangeable.inspect(); // Okay: doesn't change an unchangeable object
  unchangeable.mutate();  // ERROR: attempt to change unchangeable object
}
```
The attempt to call unchangeable.mutate() is an error caught at compile time. There is no runtime space or speed penalty for const, and you don’t need to write test-cases to check it at runtime.

The trailing const on inspect() member function should be used to mean the method won’t change the object’s abstract (client-visible) state. That is slightly different from saying the method won’t change the “raw bits” of the object’s struct. C++ compilers aren’t allowed to take the “bitwise” interpretation unless they can solve the aliasing problem, which normally can’t be solved (i.e., a non-const alias could exist which could modify the state of the object). Another (important) insight from this aliasing issue: pointing at an object with a pointer-to-const doesn’t guarantee that the object won’t change; it merely promises that the object won’t change via that pointer.
## What does “const X* p” mean?
It means p points to an object of class X, but p can’t be used to change that X object (naturally p could also be NULL).

Read it right-to-left: “p is a pointer to an X that is constant.”

For example, if class X has a const member function such as inspect() const, it is okay to say p->inspect(). But if class X has a non-const member function called mutate(), it is an error if you say p->mutate().

Significantly, this error is caught by the compiler at compile-time — no run-time tests are done. That means const doesn’t slow down your program and doesn’t require you to write extra test-cases to check things at runtime — the compiler does the work at compile-time.
## What’s the difference between “const X* p”, “X* const p” and “const X* const p”? 
+ Read the pointer declarations right-to-left.
   + const X* p means “p points to an X that is const”: the X object can’t be changed via p.
   + X* const p means “p is a const pointer to an X that is non-const”: you can’t change the pointer p itself, but you can change the X object via p.
   + const X* const p means “p is a const pointer to an X that is const”: you can’t change the pointer p itself, nor can you change the X object via p.
And, oh yea, did I mention to read your pointer declarations right-to-left?
## What does “const X& x” mean?
It means x aliases an X object, but you can’t change that X object via x.

Read it right-to-left: “x is a reference to an X that is const.”

For example, if class X has a const member function such as inspect() const, it is okay to say x.inspect(). But if class X has a non-const member function called mutate(), it is an error if you say x.mutate().

This is entirely symmetric with pointers to const, including the fact that the compiler does all the checking at compile-time, which means const doesn’t slow down your program and doesn’t require you to write extra test-cases to check things at runtime.
