---
title: C++ Type Casting
toc: true
description: C++的类型转换详解
date: 2018-03-12 00:00:00
tags:
- C++
---
## C Type Casting

`(type_name) expression`

```c
#include <stdio.h>
main() {
    double d = (double) 5;
    printf("%f\n", d);
    int i = (int) 5.4;
    printf("%d\n", i);
    int i2 = (int) (5.4);
    printf("%d\n", i2);
    int i3 = int (5.4);
    printf("%d\n", i3);
    double result = (double) 4 / 5;
    printf("%f\n", result);
}
```

## C++ Type Casting

Converting a value of a given type into another type is known as *type-casting*.

Conversion can be **implicit** or **explicit**.

## C++ Implicit Conversion

Implicit conversions do not  require any operator.

They are automatically performed when a value is copied to a compatible type.

```c++
#include <iostream>
class A {
    int x = 0;
public:
    A(int x) { this->x = x; }
    int getX() { return x; }
};
class B {
    int x = 0, y = 0;
public:
    B(A a) { x = a.getX(); y = a.getX(); }
    void getXY(int &x, int &y) { x = this->x; y = this->y; }
};
int main() {
    A a(10);
    B b = a;
    int x, y;
    b.getXY(x, y);
    std::cout << "x: " << x << " y: " << y << std::endl;
    return 0;
}
```

C++ keyword `explicit`

The explicit specifier specifies that a constructor or conversion function (since C++11) doesn't allow [implicit conversions](http://en.cppreference.com/w/cpp/language/implicit_cast) or [copy-initialization](http://en.cppreference.com/w/cpp/language/copy_initialization). It may only appear within the decl-specifier-seq of the declaration of such a function within its class definition[^1].

## C++ Explicit Conversion

Conversions that imply a different interpretation of the value, require an explicit conversion using type cast operator.

`B b = (B) a;` 

`B b = B (a);` 

Using type cast operators indiscriminately on classes and pointers to classes can lead to code that while being **syntactically correct** can cause **runtime errors**.

```c++
#include <iostream>
class Data { float i = 5, j = 6; };
class Addition {
   int x, y;
public:
   Addition(int x, int y) { this->x = x; this->y = y; }
   int result() { return x + y; }
};
int main() {
    Data d;
    Addition *add = (Addition *) &d;
    std::cout << add->result();
    return 0;
}
```

### Four Specific Casting Operators

Traditional explicit type-casting allows to convert any pointer into any other pointer type, independently of the types they point to.

The subsequent call to member function will produce either a run-time error or unexpected result.

In order to control these types of conversions between classes, there are four specific casting operators:

*static_cast*,  *const_cast*,  *dynamic_cast*,  *reinterpret_cast*.

#### static_cast

`static_cast` can perform conversions between pointers to related classes, not only from the derived class to its base, but also from a base class to its derived. Such conversions are not always safe.

```c++
class B {};    
class D : public B {};  
void f(B* pb, D* pd) {  
   D* pd2 = static_cast<D*>(pb);   // Not safe, D can have fields  
                                   // and methods that are not in B.  
   B* pb2 = static_cast<B*>(pd);   // Safe conversion, D always  
                                   // contains all of B.  
}  
```

The `static_cast` operator can also be used to perform any implicit conversion, including standard conversions and user-defined conversions. For example:

```c++
typedef unsigned char BYTE;  
void f() {  
   int i = 65;  
   float f = 2.5;  
   char ch = static_cast<char>(i);   // int to char  
   double dbl = static_cast<double>(f);   // float to double  
   i = static_cast<BYTE>(ch);  
}  
```

The `static_cast` operator can explicitly convert an integral value to an enumeration type. If the value of the integral type does not fall within the range of enumeration values, the resulting enumeration value is undefined.

The `static_cast` operator converts a null pointer value to the null pointer value of the destination type.

Any expression can be explicitly converted to type void by the `static_cast` operator. The destination void type can optionally include the `const`, `volatile`, or `__unaligned` attribute.

#### const_cast

`const_cast` manipulates the constness of an object, either to be set or to be removed.

```c++
#include <iostream>
class B {
public:
    int num = 5;
};
int main() {
    using namespace std;
    const B b1;

    b1.num = 10;
    cout << b1.num << endl;

    B b2 = const_cast<B>(b1);
    b2.num = 15;
    cout << b1.num << endl;

    B *b3 = const_cast<B *>(&b1);
    b3->num = 20;
    cout << b1.num << endl;

    B &b4 = const_cast<B &>(b1);
    b4.num = 25;
    cout << b1.num << endl;
}
```

another example at [MSDN](https://msdn.microsoft.com/en-us/library/bz6at95h.aspx):

```c++
#include <iostream>
using namespace std;
class CCTest {
public:
   void setNumber( int );
   void printNumber() const;
private:
   int number;
};

void CCTest::setNumber( int num ) { number = num; }

void CCTest::printNumber() const {
   cout << "\nBefore: " << number;
   const_cast< CCTest * >( this )->number--;
   cout << "\nAfter: " << number;
}

int main() {
   CCTest X;
   X.setNumber( 8 );
   X.printNumber();
}
```

you could use `mutable` keyword on last example.

#### dynamic_cast

`dynamic_cast` can be used only with pointers and references to objects.

Its purpose is to ensure that the result of the type conversion is a valid complete object of the requested class.

`dynamic_cast` is always successful when we cast a class to one of its base classes. 

```c++
class B { };  
class C : public B { };  
class D : public C { };  
  
void f(D* pd) {  
   C* pc = dynamic_cast<C*>(pd);   // ok: C is a direct base class  
                                   // pc points to C subobject of pd   
   B* pb = dynamic_cast<B*>(pd);   // ok: B is an indirect base class  
                                   // pb points to B subobject of pd  
} 
```

This type of conversion is called an "upcast" because it moves a pointer up a class hierarchy, from a derived class to a class it is derived from. *An upcast is an implicit conversion*.

```c++
#include <iostream>
class B {virtual void f(){}};
class D : public B {};

int main() {
    B* b = new D;
    B* b2 = new B;

    D* d = dynamic_cast<D*>(b);
    if (d == nullptr) {
        std::cout << "null pointer on first type-casting." << std::endl;
    }
    D* d2 = dynamic_cast<D*>(b2);
    if (d2 == nullptr) {
        std::cout << "null pointer on second type-casting." << std::endl;
    }
    return 0;
}
```

This type of conversion is called a "downcast" because it moves a pointer down a class hierarchy, from a given class to a class derived from it.

Another cross cast example:

```c++
#include <iostream>
class A {
public:
    int num = 5;
    virtual void f(){}
};

class B : public A {};

class D : public A {};

int main() {
    B *b = new B;
    b->num = 10;
   	D *d1 = static_cast<D *>(b);
    D *d2 = dynamic_cast<D *>(b);
    std::cout << (d2 == nullptr) << std::endl;
    delete b;
}
```

Another more complicated example: 

```c++
class A {virtual void f();};  
class B : public A {virtual void f();};  
class C : public A { };  
class D {virtual void f();};  
class E : public B, public C, public D {virtual void f();};  
  
void f(D* pd) {  
   E* pe = dynamic_cast<E*>(pd);  
   B* pb = pe;   // upcast, implicit conversion  
   A* pa = pb;   // upcast, implicit conversion  
}  
```

#### reinterpret_cast

`reinterpret_cast` converts any pointer type to any other pointer type, even of unrelated classes.

```c++
class A {};
class B {};
A *a = new A;
B *b = reinterpret_cast<B*>(a);
```

Misuse of the `reinterpret_cast` operator can easily be unsafe. Unless the desired conversion is inherently low-level, you should use one of the other cast operators.

## Questions

1. Is the function as below safe ? If not, what is the better way ?[^2]

   ```c++
   template <class T>
   unsigned char* alias(T& x) {
       return (unsigned char*)(&x);
   }
   ```

2. Which of the following lines will compile errors?[^3]

   ```c++
   class A {
   public:
       virtual void foo() { }
   };
   class B {
   public:
       virtual void foo() { }
   };
   class C : public A , public B {
   public:
       virtual void foo() { }
   };
   void bar1(A *pa) {
       B *pc = dynamic_cast<B*>(pa);
   }
   void bar2(A *pa) {
       B *pc = static_cast<B*>(pa);
   }
   void bar3() {
       C c;
       A *pa = &c;
       B *pb = static_cast<B*>(static_cast<C*>(pa));
   }
   int main() {
       return 0;
   }
   ```

   ​

## External links

1. [When should static_cast, dynamic_cast, const_cast and reinterpret_cast be used?](https://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used)
3. [C++ casting](http://www.cplusplus.com/articles/iG3hAqkS/)
4. [static_cast operator](https://msdn.microsoft.com/en-us//library/c36yw7x9.aspx)
4. [dynamic_cast operator](https://msdn.microsoft.com/en-us/library/cby9kycs.aspx)
5. [reinterpret_cast operator](https://msdn.microsoft.com/en-us/library/e0w9f63b.aspx)
6. [TypeCasting](http://teacher.buet.ac.bd/mhkabir/cse201/slides/TypeCasting.pdf)




[^1]: [explicit specifier](http://en.cppreference.com/w/cpp/language/explicit)
[^2]: [How do you explain the differences among static_cast, reinterpret_cast, const_cast, and dynamic_cast to a new C++ programmer?](https://www.quora.com/How-do-you-explain-the-differences-among-static_cast-reinterpret_cast-const_cast-and-dynamic_cast-to-a-new-C++-programmer)
[^3]: http://blog.jobbole.com/108817/

