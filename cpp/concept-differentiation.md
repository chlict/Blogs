# C++中几个概念的区分
C++中有一些概念名词，不仔细区分的话经常会觉得迷惑，不清楚它们的具体所指。本文做一些区分与解释，一是弄清这些相近的概念本身比较有趣，二是对我们日常工作也有帮助，比如，C++编译器的报错信息就经常出现一些概念，准确理解它们的含义有助于快速诊断错误，修复代码，提高工作效率。
## 1. 'parameter' vs 'argument'
### 1.1. 函数的'paramter'和'argument'
`parameter`和`argrment`在中文里通常都称为"参数"，区分度不大，但在英文语境下，两者有比较明显的区别：在函数的声明或定义中，参数列表中出现的参数被称为`parameter`，而函数调用时传递的参数被称为`argument`。也就是说，`parameter`对应我们通常说的"形参"，而`argument`对应我们通常说的"实参"。
```cpp
void foo(int x) {
    // x is called parameter
}

int main() {
    int a = 1;
    foo(a); // a is called argument
}
```
### 1.2. 模板的'parameter'和'argument'
另一组相近的概念是`template parameter`与`template argument`。`template parameter`是指定义一个template时，紧跟在`template`声明后的`<>`中出现的模板参数，而`template argument`是指实例化或特例化一个template时，出现在模板名称后的`<>`里的类型参数。举例来说：
```cpp
// T and U are template parameters
template <typename T, typename U>
struct A {};

void Foo() {
    A<int, char> a;  // int, char are template arguments
}
```
cppreference[2]上对`template parameters`的解释：
> **Template parameters**
> Every template is parameterized by one or more template parameters, indicated in the parameter-list of the template declaration syntax:
> template < parameter-list > declaration	

其中的parameter又分为*non-type template parameter*, *type template parameter*和*template template parameter*，这里不再赘述。

对`template arguments`的解释：
> **Template arguments**
> In order for a template to be instantiated, every template parameter (type, non-type, or template) must be replaced by a corresponding template argument. 

C++中有一种*partial template specialization*的概念，它的语法定义是这样的[3]：
```
template < parameter-list > class-key class-head-name < argument-list > declaration                   (1)
template < parameter-list > decl-specifier-seq declarator < argument-list > initializer(optional)     (2) since c++14
```
从这个定义里也很容易看出哪部分是`template parameter`，哪部分是`template argument`。举例说明：
```cpp
// primary template
template <class T, T t> // parameter list
struct C {};

// partial template specialization
template <class T> // parameter list
struct C<T, 1>;    // argument list. error: type of the argument 1 is T,
                   // which depends on the parameter T
```
这个例子中，由于特例化的参数`1`的类型是另一个模板参数`T`，是一个*dependent type*，不允许被*partial specialization*，因此编译器会报告错误：
```
test.cpp:7:13: error: non-type template argument specializes a template parameter with dependent type 'T'
struct C<T, 1>;
            ^
test.cpp:2:22: note: template parameter is declared here
template <class T, T t>
                   ~ ^
```
理解了`template parameter`与`template argument`的区别，也更容易读懂编译器的报错信息了。

### 1.3. 'template template parameter' vs 'template template argument'
前文说过，`template template parameter`是`template paramter`的一种，是指用模板来做模板参数。cppreference[2]中对其定义如下：

> **Template template parameter**
> template < parameter-list > typename(C++17)|class name(optional)	(1)	
> template < parameter-list > typename(C++17)|class name(optional) = default	(2)	
> template < parameter-list > typename(C++17)|class ... name(optional)	(3)	(since C++11)

一般的`template parameter`也就是简单的'**typename|class** name'。

举例来说：
```cpp
template <typename T, template <typename> X>
struct A {};

template <typename U>
struct B {};

A<B> a;
```
模板A中的`template <typename> X`就是一个`template template parameter`，B是一个可以匹配X的模板，在`A<B>`表达式中作为`template template argument`实例化了模板A。

参考：
[1] https://stackoverflow.com/questions/1788923/parameter-vs-argument
[2] https://en.cppreference.com/w/cpp/language/template_parameters
[3] https://en.cppreference.com/w/cpp/language/partial_specialization

## 2. 'class template' vs 'template class'
理论上说，只有'class template'，没有'template class'。通常定义一个模板类即是定义了一个'class template'，意思是当用实际的类(arguments)替换掉里面的模板参数(parameters)时，这个模板就能生成一个类定义。有时候人们会称从模板实例化出来的类为'template class'，但标准的说法仍然是instantiation。

C++的作者Bjarne Stroustrup对此持不较真的态度，他在其著作*The C++ Programming Language 4th edition*中说道：
> There are people who make semantic distinctions between the terms class template and template class. I don't; that would be too subtle: please consider those terms interchangeable. Similarly, I consider function template interchangeable with template function.

参考：
[1] https://stackoverflow.com/questions/879535/what-is-the-difference-between-a-template-class-and-a-class-template
