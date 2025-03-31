+++
title = '什么是多态'
date = 2025-03-30
+++
# 什么是多态？

来自维基百科，通常来说，多态主要分为以下几种类型：

- [**特设多态（Ad-hoc Polymorphism）**](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism)：为特定类型的不同集合定义一个统一的接口。  
- [**参数化多态（Parametric Polymorphism）**](https://en.wikipedia.org/wiki/Parametric_polymorphism)：使用类型参数代替具体类型，使代码适用于任意类型。  
- [**子类型多态（Subtyping）**](https://en.wikipedia.org/wiki/Subtyping)：允许同一个名称指代多个不同类的实例，这些类共享一个共同的超类。  

---

## **特设多态（Ad-hoc Polymorphism）**

### **为什么叫特设多态？**
特设多态与参数化多态相对，它针对的是**特定类型或类型组合**的行为，而不是对任意类型抽象。例如，函数重载和操作符重载就是特设多态的典型应用。

### **函数重载是特设多态**
#### **什么是函数签名？**
通常，函数签名包括**函数名**以及**参数类型**的组合，一般不包含返回值类型。

```cpp
int add(int x, int y);
float add(float x, float y);
```

在 C++ 中，为了在链接阶段唯一标识不同的函数，编译器会对函数名进行 [Name Mangling](https://www.ibm.com/docs/en/i/7.5?topic=linkage-name-mangling-c-only)。然而，为了与 C 语言进行 FFI（外部函数接口），如果在 C++ 代码中使用 `extern "C"` 关键字声明函数，编译器就不会对其进行 Name Mangling，因此这类函数**无法重载**。

Odin 语言的 [Procedure Group](https://odin-lang.org/docs/overview/#rationale-behind-explicit-overloading) 也是特设多态的一种表现形式。与大多数语言不同，Odin 需要**手动**指定哪些函数可以使用相同的名称，而不是通过**隐式的**函数重载机制。

### **大多数语言的四则运算和比较运算都是特设多态**
例如，加法运算 `+` 既可以用于整数 (`1 + 1`)，也可以用于浮点数 (`1.1 + 1.1`)。  
通常，加法两侧的操作数类型需要一致（如 `i32` 或 `f32`）。  
但某些语言支持 **隐式类型转换（Type Coercion）**，例如 `1 + 1.1` 可能会将 `1` 转换为 `1.0` 以适配浮点数计算(因为此处的`1`是字面量，所以可能编译器会直接将它作为浮点数处理)


Type Coercion 也是一种 **特设多态**，它允许不同但兼容的类型自动转换，在新的语言中一般会保证此时的转换是安全的，否则会要求改为显示转换的形式：
- [Rust 的 Type Coercion](https://doc.rust-lang.org/reference/type-coercions.html)
- [Zig 的 Type Coercion](https://ziglang.org/documentation/master/#Type-Coercion)

### **OCaml 不支持特设多态**
在 OCaml 中，相同运算对于不同的类型需要不同的运算符。例如：
```ocaml
1 + 1      (* 整数加法 *)
1.1 +. 1.1 (* 浮点数加法 *)
```
此外，OCaml 的打印函数也都是独立的，如 `print_int`, `print_float`, `print_string` 等，而不像许多语言那样使用一个统一的 `print` 函数。

### **Rust 的 Trait（非 `dyn`）和 Haskell 的 Typeclass 是特设多态**
在 Haskell 中，如果 `Person` 类型的变量要支持在ghci显示，则需要实现 `Show` 类型类：
```haskell
data Person = Person String Int  -- Name, Age
instance Show Person where
    show (Person name age) = "Person: " ++ name ++ ", Age: " ++ show age
```
Rust 也有类似的 `fmt::Display`Trait：
```rust
use std::fmt;

struct Point {
    x: i32,
    y: i32,
}
impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "({}, {})", self.x, self.y)
    }
}
```

---

## **参数化多态（Parametric Polymorphism）**
参数化多态也被称为 **泛型**，它允许代码独立于具体类型进行抽象。 通常来讲参数化多态是不考虑泛型约束的，不过从实用的角度看，大部分语言的参数化多态都有泛型约束能力

### **Haskell 的 `map`**
```haskell
map :: (a -> b) -> [a] -> [b]
map f [] = []
map f (x:xs) = f x : map f xs
```
`map` 函数无需关心 `a` 和 `b` 的具体类型

### **Rust 的 `max` 函数**
```rust
fn max<T: PartialOrd>(a: T, b: T) -> T {
    if a > b { a } else { b }
}
```
这里 `T` 需要实现 `PartialOrd` Trait，否则无法进行比较。

### 单态化（Monomorphization）
在某些语言（如 C++ 和 Rust）中，参数化多态会在编译期生成具体的类型实例，这称为 **单态化**：
- [C++ 模板和 Linker](https://www.airs.com/blog/archives/53)
- [Matklad 讨论 Zig 处理泛型的优势](https://lobste.rs/s/0jknbl/roc_rewrites_compiler_zig#c_siki17)

---

## **子类型多态（Subtyping Polymorphism）**
子类型多态通常通过 **继承** 机制实现，比如C++, Java, C#

### **Rust 的 Trait Object**
Trait 只有在满足一定条件时才能转换为 Trait Object，例如：
```rust
trait Animal {
    fn make_sound(&self);
}

struct Dog;
impl Animal for Dog {
    fn make_sound(&self) {
        println!("Woof!");
    }
}

let dog: Box<dyn Animal> = Box::new(Dog);
```
并非所有 Trait 都能成为 `dyn Trait`，更多细节可参考 [dyn-compatibility](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility)。

---

## **动态多态 vs 静态多态**
### **静态多态（Static Polymorphism）**
发生在**编译期**，包括：
- **特设多态**（函数重载、运算符重载）
- **参数化多态**（泛型）

### **动态（运行时）多态（Dynamic Polymorphism）**
发生在**运行时**，包括：
- **子类型多态**（继承/接口）
- **Trait Object（Rust）**
- **手写 VTable（如 Zig）**，可参考[Zig Interfaces](https://www.openmymind.net/Zig-Interfaces/)

#### **通过Sum Type / Tagged Union 实现运行时多态**
除了 VTable 方案，某些语言（如 Rust、OCaml）可以使用 **Sum Type / Tagged Union** 来手动分发到不同的行为，不过这种方法存在一定的问题，比如如果variant大小相差比较大的时候，会有空间的浪费，还有就是扩展是需要改变原来sum type的定义，具体可参考表达式问题
- [表达式问题（Expression Problem）](https://en.wikipedia.org/wiki/Expression_problem)
- [Rust: 使用 Trait Object vs Enum](https://users.rust-lang.org/t/using-trait-vs-wrapping-object-in-enum/92120)

---

## **Row Polymorphism**
Row Polymorphism 是一种更灵活的多态机制，允许**扩展对象的字段**：
- [介绍 Row Polymorphism](https://jadon.io/blog/row-polymorphism/)
