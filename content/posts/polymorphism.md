+++
title = '什么是多态'
date = 2025-03-27
+++

来自维基百科,通常来说主要的多态类型如下
- [特设多态](https://en.wikipedia.org/wiki/Ad_hoc_polymorphism): 为个体的特定类型的任意集合定义一个共同接口
- [参数化多态](https://en.wikipedia.org/wiki/Parametric_polymorphism): 指定一个或多个类型不靠名字而是靠可以标识任何类型的抽象符号
- [子类型多态](https://en.wikipedia.org/wiki/Subtyping):一个名字指称很多不同的类的实例，这些类有某个共同的超类
## 特设多态(ad hoc polymorphism)
为什么叫特设多态，主要是相对于参数化多态，特设多态针对的是特定类型（类型组合）的行为
### 函数重载是特设多态
>什么是函数签名？
>通常意义上函数签名包括函数名以及输入的参数类型的组合，一般不包括函数的返回值类型
```cpp
int add(int x, int y);
float add(float x, float y);
```
编译到二进制的时候，符号表需要有唯一的标识，所以c++会对函数名进行[**Name Mangling**](https://www.ibm.com/docs/en/i/7.5?topic=linkage-name-mangling-c-only)。不过为了c语言FFI，对函数添加`extern C`则不会进行name mangling，相应的函数也没法重载

odin的[procedure group](https://odin-lang.org/docs/overview/#rationale-behind-explicit-overloading)也是特设多态，和其他语言的区别是需要用户手动指定哪些函数可以使用同样的名字，而非隐式通过同样的函数名实现函数重载

### 大部分语言的四则运算和比较运算都是特设多态
比如加法可以同时支持 `1+1` 和 `1.1 + 1.1`，此处加法支持各种整数和各种浮点数，通常情况下加法两边的类型是一致的比如都是i32或者f32
不过根据语言的不同可能会发生type coercion（type coercion也是**特设多态** ：）,或者叫[隐式类型转换](https://stackoverflow.com/questions/66388288/what-is-the-difference-between-conversion-casting-and-coercion)
ps. 新的语言在做隐式类型转换时一般会验证转换是否安全，否则会强制用户使用显示转换
[Rust的type coercions](https://doc.rust-lang.org/reference/type-coercions.html)
[Zig的type coercion](https://ziglang.org/documentation/master/#Type-Coercion)
### OCaml没有特设多态
OCaml不支持特设多态，针对不同类型的同样行为无法使用同名函数,比如Ocaml的打印函数:
print_bytes,print_char,print_endline,print_float,print_int,print_newline,print_string 

整数加法是`+`：`1+ 1`
浮点数加法是`+.`: `1.1 +. 1.1`
### rust的trait（非dyn trait）和haskell的typeclass是特设多态
如过需要Person类型的变量在ghci中可以直接打印出来，Person需要实现对应的Show instance，通过show函数将Person转成成字符串，通常情况下也可以使用`derive Show`来实现类似的功能
```haskell
data Person = Person String Int  -- Name, Age
instance Show Person where
    show (Person name age) = "Person: " ++ name ++ ", Age: " ++ show age
```
Rust中有类似的`fmt::Display`,`fmt::Debug` trait，同样也可以用derive宏来实现相应的trait
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

## 参数化多态
大部分语言叫作泛型，通常来讲参数化多态是不考虑泛型约束的，不过从实用的角度看，大部分语言的参数化多态都有泛型约束能力
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
[https://en.wikipedia.org/wiki/Monomorphization](https://en.wikipedia.org/wiki/Monomorphization)

单态化与linker
[linker和c++模板](https://www.airs.com/blog/archives/53)
[matklad谈zig编译泛型代码的优势](https://lobste.rs/s/0jknbl/roc_rewrites_compiler_zig#c_siki17)

## 子类型多态
C++/Java通过继承实现子类型多态
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

[dyn-compatibility](https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility), A dyn-compatible trait can be the base trait of a trait object. A trait is dyn compatible if it has the following qualities

不支持面向对象甚至没有接口的语言可以通过手写vtable实现运行时多态

zig通过VTable实现多态,更多的可以参考[Zig Interfaces](https://www.openmymind.net/Zig-Interfaces/)
```zig
const std = @import("std");
const VTable = struct {
    ptr: *const anyopaque,
    drawFn: *const fn (*const anyopaque) void,
    fn init(ptr: anytype) VTable {
        const T = @TypeOf(ptr);
        const ptr_info = @typeInfo(T);
        const Ty = ptr_info.pointer.child;
        const gen = struct {
            pub fn draw(pointer: *const anyopaque) void {
                const self: T = @ptrCast(@alignCast(pointer));
                // self.draw();
                @compileLog(Ty);
                Ty.draw(self);

                // return ptr_info.pointer.child.draw(self);
            }
        };
        return .{ .ptr = ptr, .drawFn = gen.draw };
    }
    fn draw(self: VTable) void {
        self.drawFn(self.ptr);
    }
};
const Rectangle = struct {
    width: u32,
    height: u32,
    fn draw(self: *const Rectangle) void {
        std.debug.print("Rectangle{{ width = {}, height = {} }}\n", .{ self.width, self.height });
    }
    fn shape(self: *const Rectangle) VTable {
        return .init(self);
    }
};
const Circle = struct {
    raidus: u32,
    fn draw(self: *const Circle) void {
        std.debug.print("Circle{{ radius = {} }}\n", .{self.raidus});
    }
    fn shape(self: *const Circle) VTable {
        return .init(self);
    }
};
pub fn main() !void {
    const rect = Rectangle{ .width = 10, .height = 20 };
    const circle = Circle{ .raidus = 2 };
    const shape1 = rect.shape();
    const shape2 = circle.shape();
    const shapes = [_]VTable{ shape1, shape2 };
    for (shapes) |shape| {
        shape.draw();
    }
}

```

#### 通过sum type/tagged union实现运行时多态
通过模式匹配手动分发到相应的函数，相比较vtable方案在存储上应该会有更大的开销,而且扩展必须要修改原来的sum type,具体可参考[表达式问题](https://en.wikipedia.org/wiki/Expression_problem)
rust社区有不少相关的讨论
https://users.rust-lang.org/t/using-trait-vs-wrapping-object-in-enum/92120
https://users.rust-lang.org/t/trait-object-or-enum-how-to-choice/100268

# Row Polymorphism
[https://jadon.io/blog/row-polymorphism/](https://jadon.io/blog/row-polymorphism/)
