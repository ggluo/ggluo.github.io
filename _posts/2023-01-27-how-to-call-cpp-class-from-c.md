---
layout: distill
title: How to Call C++ Class from C
permalink: /blog/call-cpp-class-from-c
date: 2023-01-27 10:54 +0100
toc:
  - name: Define a C++ class
  - name: Define a C wrapper
  - name: Use the wrapper in C
tag:
    - c++
---

Calling a C++ class from C can be a bit tricky because C does not support classes and objects like C++. However, you can still interact with C++ code from C by creating a C interface for your C++ class. In this example, we define a C++ class MyClass with a method doSomething(). Then, we create a C wrapper MyClassWrapper that provides functions to create, manipulate, and destroy instances of MyClass. Finally, we can use these functions in a C program to interact with the MyClass object. See [github](https://github.com/ggluo/Call-cpp-class-from-c).

## Define a C++ class

```cpp
#ifndef MYCLASS_H
#define MYCLASS_H

class MyClass {
public:
    MyClass();
    void doSomething();
private:
    int data;
};
#endif // MYCLASS_H
```

```cpp
#include "MyClass.h"
#include <iostream>

MyClass::MyClass() : data(0) {}

void MyClass::doSomething() {
    std::cout << "Doing something in MyClass" << std::endl;
}
```

## Define a C wrapper

```cpp
// MyClassWrapper.h
#ifndef MYCLASSWRAPPER_H
#define MYCLASSWRAPPER_H

#ifdef __cplusplus
extern "C" {
#endif

typedef void* MyClassHandle;

MyClassHandle createMyClass();
void destroyMyClass(MyClassHandle obj);
void doSomethingInMyClass(MyClassHandle obj);

#ifdef __cplusplus
}
#endif

#endif // MYCLASSWRAPPER_H

```

```cpp
#include "MyClassWrapper.h"
#include "MyClass.h"

extern "C" {

MyClassHandle createMyClass() {
    return reinterpret_cast<MyClassHandle>(new MyClass());
}

void destroyMyClass(MyClassHandle obj) {
    delete reinterpret_cast<MyClass*>(obj);
}

void doSomethingInMyClass(MyClassHandle obj) {
    reinterpret_cast<MyClass*>(obj)->doSomething();
}
}

```

## Use the wrapper in C

```cpp
#include "MyClassWrapper.h"

int main() {
    MyClassHandle obj = createMyClass();
    doSomethingInMyClass(obj);
    destroyMyClass(obj);
    return 0;
}

```
