---
title: 如何让你的C++代码变得更快
date: 2024-01-21
categories: c++
---

## 使用初始化列表（Initializer Lists）

```C++
std::vector<ModelObject> mos{mo1, mo2};
auto mos = std::vector<ModelObject>{mo1, mo2};
// Don't do this
std::vector<ModelObject> mos;
mos.push_back(mo1);
mos.push_back(mo2);
```

初始化列表能帮助减少对象拷贝和容器的大小的扩容。

## 减少临时对象的创建
```C++
// Instead of
auto mo1 = getSomeModelObject();
auto mo2 = getAnotherModelObject();
doSomething(mo1, mo2);
// consider:
doSomething(getSomeModelObject(), getAnotherModelObject());
```

这能减少编译器的`move`操作。

## 减少拷贝和赋值
```C++
// Bad Idea
std::string somevalue;
if (caseA) {
  somevalue = "Value A";
} else {
  somevalue = "Value B";
}
// Better Idea
const std::string somevalue = caseA ? "Value A" : "Value B";
```

```C++
// Bad Idea
std::string somevalue;
if (caseA) {
  somevalue = "Value A";
} else if(caseB) {
  somevalue = "Value B";
} else {
  somevalue = "Value C";
}
// Better Idea
const std::string somevalue = [&](){
    if (caseA) {
      return "Value A";
    } else if (caseB) {
      return "Value B";
    } else {
      return "Value C";
    }
}();
```

## 避免使用`new`操作
```C++
// require two heap allocation
std::shared_ptr<ModelObject_Impl>(new ModelObject_Impl());
// should become
std::make_shared<ModelObject_Impl>(); // (it's also more readable and concise)
```

## 优先使用`unique_ptr`
```C++
std::unique_ptr<ModelObject_Impl> factory();
auto shared = std::shared_ptr<ModelObject_Impl>(factory());
```
最佳实践是通过工厂函数返回`unique_ptr`，在需要的时候再转化为`shared_ptr`。

## 避免使用std::endl
`std::endl`包含`flush`的操作。

## 限制局部变量的作用域
```C++
// Good Idea
for (int i = 0; i < 15; ++i)
{
  MyObject obj(i);
  // do something with obj
}
// Bad Idea
MyObject obj; // meaningless object initialization
for (int i = 0; i < 15; ++i)
{
  obj = MyObject(i); // unnecessary assignment operation
  // do something with obj
}
// obj is still taking up memory for no reason
```

## 使用`init-statement`
```C++
if (MyObject obj(index); obj.good()) {
    // do something if obj is good
} else {
// do something if obj is not good
}
```

## Char和String
```C++
// Bad Idea
std::cout << someThing() << "\n";
// Good Idea
std::cout << someThing() << '\n';
```
`"\n"`会被解析为`const char *`并为其做范围检查。单个`\n`能避免多余的CPU指令执行。

## 避免使用`std::bind`
```C++
// Bad Idea
auto f = std::bind(&my_function, "hello", std::placeholders::_1);
f("world");
// Good Idea
auto f = [](const std::string &s) { return my_function("hello", s); };
f("world");
```
推荐使用`lambda`，而不是`std::bind`。

## 尽可能使用`vector`
在使用容器的时候多问问是否用一个`vector`就够？

## 定义虚拟析构函数
对于任何一个有虚函数的类添加虚拟析构函数，即时它什么都没做。

## 不要在构造函数和析构函数中调用虚函数
```C++
class Transaction {
    public:
    Transaction();
    virtual void log_transaction() const = 0;
};
Transaction:: Transaction() {
  ...
  // log the tranaction
  log_transaction();
}
class BuyTransaction: public Transaction {
  public:
    virtual void log_tranction() const;
    ... 
}
class SellTransaction: public Transaction {
  public:
    virtual void log_transaction() const;
}

BuyTransaction b;
```

## 引用
* [https://www.geeksforgeeks.org/move-constructors-in-c-with-examples/](https://www.geeksforgeeks.org/move-constructors-in-c-with-examples/)
* [https://blog.quasardb.net/using-c-containers-efficiently](https://blog.quasardb.net/using-c-containers-efficiently)
* [https://www.geeksforgeeks.org/virtual-destructor/](https://www.geeksforgeeks.org/virtual-destructor/)
* [https://lefticus.gitbooks.io/cpp-best-practices/content/](https://lefticus.gitbooks.io/cpp-best-practices/content/)
* [https://www.educative.io/edpresso/what-is-a-move-constructor-in-cpp](https://www.educative.io/edpresso/what-is-a-move-constructor-in-cpp)


