+++
title = '什么是多态'
date = 2025-03-27
+++
维基百科
通常来说主要的多态类型如下
- _[特设多态](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism "Ad hoc polymorphism")_: 为个体的特定类型的任意集合定义一个共同接口
- _[参数化多态](https://en.wikipedia.org/wiki/Parametric_polymorphism "Parametric polymorphism")_: 指定一个或多个类型不靠名字而是靠可以标识任何类型的抽象符号
- _[子类型多态](https://en.wikipedia.org/wiki/Subtyping "Subtyping")_ :一个名字指称很多不同的类的实例，这些类有某个共同的超类
## 特设多态(ad hoc polymorphism)
为什么叫特设多态，主要是相对于参数化多态，特设多态针对的是特定类型（类型组合）的多态
### 函数重载是特设多态
>什么是函数签名？
>通常意义上函数签名包括函数名以及输入的参数类型的组合，一般不包括函数的返回值类型
```cpp
int add(int x, int y);
float add(float x, float y);
```
编译到二进制的时候，符号表需要有唯一的标识，所以c++会对函数名进行[**Name Mangling**](https://www.ibm.com/docs/en/i/7.5?topic=linkage-name-mangling-c-only)。不过为了和c语言FFI，对函数添加`extern C`则不会进行name mangling，相应的函数也没法重载

odin的[producer group](https://odin-lang.org/docs/overview/#rationale-behind-explicit-overloading)也是特设多态，和其他语言的区别是需要用户手动指定哪些函数可以使用同样的名字，而非隐式通过同样的函数名实现函数重载

### 大部分语言的四则运算和比较运算都是特设多态
比如加法可以同时支持 `1+1` 和 `1.1 + 1.1`，此处加法支持各种整数和各种浮点数，通常情况下加法两边的类型是一致的比如都是i32或者f32
不过根据语言的不同可能会发生type coercion（type coercion也是**特设多态** ：）,或者叫[隐式类型转换](https://stackoverflow.com/questions/66388288/what-is-the-difference-between-conversion-casting-and-coercion)
ps. 新的语言在做隐式类型转换时一般都会验证转换是否类型安全，否则会强制用户使用显示转换
[Rust的type coercions](https://doc.rust-lang.org/reference/type-coercions.html)
[Zig的type coercion](https://ziglang.org/documentation/master/#Type-Coercion)
### OCaml没有特设多态
OCaml不支持特设多态，针对不同类型的同样行为无法使用同名函数,比如Ocaml的打印函数:
print_bytes,print_char,print_endline,print_float,print_int,print_newline,print_string 

整数加法是`+`：`1+ 1`
浮点数加法是`+.`: `1.1 +. 1.1`
### rust的trait（非dyn trait）和haskell的typeclass是特设多态
比如将一个Person类型在ghci中直接显示，在Haskell需要实现对应的Show instance，通过show函数将Person转成成字符串，通常情况下出于debug考虑可以使用`derive Show`来生成类似的模板代码
```haskell
data Person = Person String Int  -- Name, Age
instance Show Person where
    show (Person name age) = "Person: " ++ name ++ ", Age: " ++ show age
```
Rust中有类似的`fmt::Display`,`fmt::Debug` trait
```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}
// 为 Point 实现 Display
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}

```

## 参数化多态
大部分语言把叫作泛型，通常来讲参数化多态是不考虑泛型约束的，不过从实用的角度看，大部分语言的参数化多态都有泛型约束能力
### Haskell的map
```haskell
map :: (a->b) -> List a -> List b
map f [] = []
map f x:xs = f x : map f xs
```
此处的map不需要考虑a和b是什么类型
### rust的max函数
```rust
fn max<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}
```
此处的T需要支持比较操作(PartialOrd trait)

### 单态化
https://en.wikipedia.org/wiki/Monomorphization

c++ 模板单态化与linker 优化
https://lobste.rs/s/0jknbl/roc_rewrites_compiler_zig#c_siki17
https://www.airs.com/blog/archives/53

## 子类型多态
面向对象通过继承实现的多态
具体实现一般是vtable

# 动态多态与静态多态
## 静态多态
发生在编译期，包括参数化多态和特设多态
## 运行时多态
子类型多态是运行时多态
rust的trait object
java或者c++通过继承/接口实现的函数多态
### 对于rust什么样的trait可以转成trait object?
我自己的理解：这个trait可以实现成一个vtable，具体的类型可以被擦除，函数的入参都有self指针，如果有函数返回Self这种是不行的
https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility

不支持面向对象甚至没有接口的语言可以通过手写vtable实现运行时多态

#### 通过sum type/tagged union实现运行时多态
通过模式匹配手动分发到相应的函数，相比较vtable方案在存储上应该会有更大的开销,而且扩展必须要修改原来的sum type,具体可参考[表达式问题](https://en.wikipedia.org/wiki/Expression_problem)