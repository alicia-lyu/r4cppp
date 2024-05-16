# Destructuring

In our previous discussion, we explored Rust's data types. Once you have some data inside a structure, you will often need to extract that data. For structs, Rust allows field access similar to C++. For tuples, tuple structs, and enums, you use destructuring. Although destructuring exists in C++ only since C++17, it may be more familiar to you from languages such as Python or various functional languages. The basic idea is that just as you can initialize a data structure by filling its fields with local variables, you can extract data from a structure into local variables. This feature, combining pattern matching with assignment, is one of Rust's most powerful tools.

Destructuring is primarily done through the `let` and `match` statements. The `match` statement is used when the structure being destructured can have different variants, such as an enum. A `let` expression extracts variables into the current scope, whereas `match` introduces a new scope. For example:

```rust
fn foo(pair: (i32, i32)) {
    let (x, y) = pair;
    // x and y can now be used anywhere in foo
}

enum En {
    Var1,
    Var2,
    Var3(i32),
    Var4(i32, i32),
}

fn bar(pair: En) {
    match pair {
        En::Var1 => println!("first variant"),
        En::Var2 => println!("second variant"),
        En::Var3(x) => {
            println!("third variant with number {}", x)
            // x can only be used in this block
        },
        En::Var4(x, y) => {
            println!("fourth variant with numbers {} and {}", x, y)
            // x and y can only be used in this block
        }
    }
}
```

The syntax for patterns (used after `let` and before `=>` in the above example) is similar in both cases. You can also use these patterns in function arguments:

```rust
fn foo((x, y): (i32, i32)) {
}
```

This is particularly useful for structs or tuple structs rather than tuples.

Most initialization expressions can appear in a destructuring pattern, and they can be arbitrarily complex. This includes references, primitive literals, and data structures. For example:

```rust
struct St {
    f1: i32,
    f2: f32,
}

enum En {
    Var1,
    Var2,
    Var3(i32),
    Var4(i32, St, i32),
}

fn foo(x: &En) {
    match x {
        En::Var1 => println!("first variant"),
        En::Var3(5) => println!("third variant with number 5"),
        En::Var3(x) => println!("third variant with number {} (not 5)", x),
        En::Var4(3, St { f1: 3, f2: x }, 45) => {
            println!("destructuring an embedded struct, found {} in f2", x)
        }
        En::Var4(_, ref s, _) => {
            println!("Some other Var4 with {} in f1 and {} in f2", s.f1, s.f2)
        }
        _ => println!("other (Var2)"),
    }
}
```

Note how we destructure through a reference by using `&` in the patterns and use a mix of literals (`5`, `3`, `St { ... }`), wildcards (`_`), and variables (`x`).

You can use `_` wherever a variable is expected if you want to ignore a single item in a pattern, so we could have used `En::Var3(_)` if we didn't care about the integer. In the first `En::Var4` arm, we destructure the embedded struct (a nested pattern), and in the second `En::Var4` arm, we bind the whole struct to a variable. You can also use `..` to stand in for all fields of a tuple or struct. For example, if you want to handle each enum variant but don't care about the content of the variants:

```rust
fn foo(x: En) {
    match x {
        En::Var1 => println!("first variant"),
        En::Var2 => println!("second variant"),
        En::Var3(..) => println!("third variant"),
        En::Var4(..) => println!("fourth variant"),
    }
}
```

When destructuring structs, the fields don't need to be in order, and you can use `..` to elide the remaining fields. For example:

```rust
struct Big {
    field1: i32,
    field2: i32,
    field3: i32,
    field4: i32,
    field5: i32,
    field6: i32,
    field7: i32,
    field8: i32,
    field9: i32,
}

fn foo(b: Big) {
    let Big { field6: x, field3: y, .. } = b;
    println!("pulled out {} and {}", x, y);
}
```

As a shorthand with structs, you can use just the field name, which creates a local variable with that name. The `let` statement in the above example created two new local variables `x` and `y`. Alternatively, you could write:

```rust
fn foo(b: Big) {
    let Big { field6, field3, .. } = b;
    println!("pulled out {} and {}", field3, field6);
}
```

Now we create local variables with the same names as the fields, in this case `field3` and `field6`.

There are a few more tricks to Rust's destructuring. If you want a reference to a variable in a pattern, you can't use `&` because that matches a reference rather than creates one (and thus has the effect of dereferencing the object). For example:

```rust
struct Foo {
    field: &'static i32,
}

fn foo(f: Foo) {
    let Foo { field: &y } = f;
    // &y on LHS: we are defining y using f, equivalent to 
    // let &y = x.field;
}

fn main() {
    let x = 5;
    let f = Foo { field: &x };
    // &x on RHS: We are defining f using &x, equivalent to 
    // let x_ref = &x;
    // let f = Foo { field: x_ref };
    foo(f);
}
```

Here, `y` has the type `i32` and is a copy of the field in `x`.

To create a reference to something in a pattern, you use the `ref` keyword. For example:

```rust
fn foo(b: Big) {
    let Big { field3: ref x, ref field6, .. } = b;
    // equivalent to
    // let Big { field3: x_no_ref, field6, .. } = b;
    // let x = &x_no_ref;
    // let field6 = &b.field6;
    println!("pulled out {} and {}", *x, *field6);
}
```

Here, `x` and `field6` both have the type `&i32` and are references to the fields in `b`.

One last trick when destructuring is that if you are destructuring a complex object, you might want to name intermediate objects as well as individual fields. Going back to an earlier example, we had the pattern `En::Var4(3, St { f1: 3, f2: x }, 45)`. In that pattern, we named one field of the struct, but you might also want to name the whole struct object. You could write `En::Var4(3, s, 45)` which would bind the struct object to `s`, but then you would have to use field access for the fields, or if you wanted to only match with a specific value in a field you would have to use a nested match. Rust lets you name parts of a pattern using `@` syntax. For example, `En::Var4(3, s @ St { f1: 3, f2: x }, 45)` lets us name both a field (`x`, for `f2`) and the whole struct (`s`).

That just about covers your options with Rust pattern matching. There are a few features I haven't covered, such as matching vectors, but hopefully, you now know how to use `match` and `let` and have seen some of the powerful things you can do. Next time, I'll cover some of the subtle interactions between `match` and borrowing which tripped me up a fair bit when learning Rust.