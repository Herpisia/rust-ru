## Обращение с умными указателями как с обычными ссылками с помощью `Deref` типажа

Implementing the `Deref` trait allows you to customize the behavior of the *dereference operator*, `*` (as opposed to the multiplication or glob operator). By implementing `Deref` in such a way that a smart pointer can be treated like a regular reference, you can write code that operates on references and use that code with smart pointers too.

Let’s first look at how the dereference operator works with regular references. Then we’ll try to define a custom type that behaves like `Box<T>`, and see why the dereference operator doesn’t work like a reference on our newly defined type. We’ll explore how implementing the `Deref` trait makes it possible for smart pointers to work in ways similar to references. Then we’ll look at Rust’s *deref coercion* feature and how it lets us work with either references or smart pointers.

> Note: there’s one big difference between the `MyBox<T>` type we’re about to build and the real `Box<T>`: our version will not store its data on the heap. We are focusing this example on `Deref`, so where the data is actually stored is less important than the pointer-like behavior.

### Следование по указателю к значению с помощью оператора разыменования

A regular reference is a type of pointer, and one way to think of a pointer is as an arrow to a value stored somewhere else. In Listing 15-6, we create a reference to an `i32` value and then use the dereference operator to follow the reference to the data:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

<span class="caption">Listing 15-6: Using the dereference operator to follow a reference to an <code>i32</code> value</span>

The variable `x` holds an `i32` value, `5`. We set `y` equal to a reference to `x`. We can assert that `x` is equal to `5`. However, if we want to make an assertion about the value in `y`, we have to use `*y` to follow the reference to the value it’s pointing to (hence *dereference*). Once we dereference `y`, we have access to the integer value `y` is pointing to that we can compare with `5`.

If we tried to write `assert_eq!(5, y);` instead, we would get this compilation error:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

Comparing a number and a reference to a number isn’t allowed because they’re different types. We must use the dereference operator to follow the reference to the value it’s pointing to.

### Использование `Box<T>` как ссылку

We can rewrite the code in Listing 15-6 to use a `Box<T>` instead of a reference; the dereference operator will work as shown in Listing 15-7:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

<span class="caption">Листинг 15-7: Использование оператора разыменования с типом <code>Box&lt;i32&gt;</code></span>

Единственная разница между листингом 15-7 и листингом 15-6 состоит в том, что здесь мы устанавливаем `y` на экземпляр box, указывающий на значение `x`, а не ссылкой, указывающей на значение `x` . В последнем утверждении мы можем использовать оператор разыменования, чтобы проследовать за указателем box-а так же, как мы это делали когда `y` была ссылкой. Далее мы рассмотрим, что особенного у типа `Box<T>`, что позволяет нам использовать оператор разыменования, определяя наш собственный тип `Box`.

### Определение собственного умного указателя

Let’s build a smart pointer similar to the `Box<T>` type provided by the standard library to experience how smart pointers behave differently from references by default. Then we’ll look at how to add the ability to use the dereference operator.

Тип `Box<T>` в конечном итоге определяется как структура кортежа с одним элементом, поэтому в листинге 15-8 аналогичным образом определяется `MyBox<T>`. Мы также определим функцию `new`, чтобы она соответствовала функции `new`, определённой в `Box<T>`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

<span class="caption">Листинг 15-8: Определение типа <code>MyBox&lt;T&gt;</code></span>

We define a struct named `MyBox` and declare a generic parameter `T`, because we want our type to hold values of any type. The `MyBox` type is a tuple struct with one element of type `T`. The `MyBox::new` function takes one parameter of type `T` and returns a `MyBox` instance that holds the value passed in.

Let’s try adding the `main` function in Listing 15-7 to Listing 15-8 and changing it to use the `MyBox<T>` type we’ve defined instead of `Box<T>`. The code in Listing 15-9 won’t compile because Rust doesn’t know how to dereference `MyBox`.

<span class="filename">Файл: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

<span class="caption">Листинг 15-9: Попытка использовать <code>MyBox&lt;T&gt;</code> таким же образом, как мы использовали ссылки и библиотечный <code>Box&lt;T&gt;</code></span>

Вот результат ошибки компиляции:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Our `MyBox<T>` type can’t be dereferenced because we haven’t implemented that ability on our type. To enable dereferencing with the `*` operator, we implement the `Deref` trait.

### Трактование типа как ссылки реализуя типаж `Deref`

As discussed in Chapter 10, to implement a trait, we need to provide implementations for the trait’s required methods. The `Deref` trait, provided by the standard library, requires us to implement one method named `deref` that borrows `self` and returns a reference to the inner data. Listing 15-10 contains an implementation of `Deref` to add to the definition of `MyBox`:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

<span class="caption">Листинг 15-10: Реализация <code>Deref</code> для типа <code>MyBox&lt;T&gt;</code></span>

The `type Target = T;` syntax defines an associated type for the `Deref` trait to use. Associated types are a slightly different way of declaring a generic parameter, but you don’t need to worry about them for now; we’ll cover them in more detail in Chapter 19.

We fill in the body of the `deref` method with `&self.0` so `deref` returns a reference to the value we want to access with the `*` operator. The `main` function in Listing 15-9 that calls `*` on the `MyBox<T>` value now compiles, and the assertions pass!

Without the `Deref` trait, the compiler can only dereference `&` references. The `deref` method gives the compiler the ability to take a value of any type that implements `Deref` and call the `deref` method to get a `&` reference that it knows how to dereference.

When we entered `*y` in Listing 15-9, behind the scenes Rust actually ran this code:

```rust,ignore
*(y.deref())
```

Rust substitutes the `*` operator with a call to the `deref` method and then a plain dereference so we don’t have to think about whether or not we need to call the `deref` method. This Rust feature lets us write code that functions identically whether we have a regular reference or a type that implements `Deref`.

The reason the `deref` method returns a reference to a value, and that the plain dereference outside the parentheses in `*(y.deref())` is still necessary, is the ownership system. If the `deref` method returned the value directly instead of a reference to the value, the value would be moved out of `self`. We don’t want to take ownership of the inner value inside `MyBox<T>` in this case or in most cases where we use the dereference operator.

Note that the `*` operator is replaced with a call to the `deref` method and then a call to the `*` operator just once, each time we use a `*` in our code. Because the substitution of the `*` operator does not recurse infinitely, we end up with data of type `i32`, which matches the `5` in `assert_eq!` in Listing 15-9.

### Неявные разыменованные приведения с функциями и методами

*Deref coercion* is a convenience that Rust performs on arguments to functions and methods. Deref coercion works only on types that implement the `Deref` trait. Deref coercion converts such a type into a reference to another type. For example, deref coercion can convert `&String` to `&str` because `String` implements the `Deref` trait such that it returns `str`. Deref coercion happens automatically when we pass a reference to a particular type’s value as an argument to a function or method that doesn’t match the parameter type in the function or method definition. A sequence of calls to the `deref` method converts the type we provided into the type the parameter needs.

Deref coercion was added to Rust so that programmers writing function and method calls don’t need to add as many explicit references and dereferences with `&` and `*`. The deref coercion feature also lets us write more code that can work for either references or smart pointers.

To see deref coercion in action, let’s use the `MyBox<T>` type we defined in Listing 15-8 as well as the implementation of `Deref` that we added in Listing 15-10. Listing 15-11 shows the definition of a function that has a string slice parameter:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

<span class="caption">Listing 15-11: A <code>hello</code> function that has the parameter <code>name</code> of type <code>&amp;str</code></span>

Можно вызвать функцию `hello` со срезом строки в качестве аргумента, например `hello("Rust");`. Разыменованное приведение делает возможным вызов `hello` со ссылкой на значение типа `MyBox<String>`, как показано в листинге 15-12.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

<span class="caption">Листинг 15-12: Вызов <code>hello</code> со ссылкой на значение <code>MyBox&lt;String&gt;</code>, которое работает из-за разыменованного приведения</span>

Here we’re calling the `hello` function with the argument `&m`, which is a reference to a `MyBox<String>` value. Because we implemented the `Deref` trait on `MyBox<T>` in Listing 15-10, Rust can turn `&MyBox<String>` into `&String` by calling `deref`. The standard library provides an implementation of `Deref` on `String` that returns a string slice, and this is in the API documentation for `Deref`. Rust calls `deref` again to turn the `&String` into `&str`, which matches the `hello` function’s definition.

Если бы Rust не реализовал разыменованное приведение, мы должны были бы написать код в листинге 15-13 вместо кода в листинге 15-12 для вызова метода `hello` со значением типа `&MyBox<String>`.

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

<span class="caption">Листинг 15-13: Код, который мы должны были бы написать, если бы в Rust не было разыменованного приведения.</span>

Код `(*m)` разыменовывает `MyBox<String>` в `String`. Затем `&` и `[..]` принимают строковый срез `String`, равный всей строке, чтобы соответствовать сигнатуре `hello`. Код без разыменованного приведения сложнее читать, писать и понимать со всеми этими символами. Разыменованное приведение позволяет Rust обрабатывать эти преобразования для нас автоматически.

When the `Deref` trait is defined for the types involved, Rust will analyze the types and use `Deref::deref` as many times as necessary to get a reference to match the parameter’s type. The number of times that `Deref::deref` needs to be inserted is resolved at compile time, so there is no runtime penalty for taking advantage of deref coercion!

### Как разыменованное приведение взаимодействует с изменяемостью

Similar to how you use the `Deref` trait to override the `*` operator on immutable references, you can use the `DerefMut` trait to override the `*` operator on mutable references.

Rust does deref coercion when it finds types and trait implementations in three cases:

- Из типа `&T` в тип `&U` когда верно `T: Deref<Target=U>`
- Из типа `&mut T` в тип `&mut U` когда верно `T: DerefMut<Target=U>`
- Из типа `&mut T` в тип `&U` когда верно `T: Deref<Target=U>`

The first two cases are the same except for mutability. The first case states that if you have a `&T`, and `T` implements `Deref` to some type `U`, you can get a `&U` transparently. The second case states that the same deref coercion happens for mutable references.

The third case is trickier: Rust will also coerce a mutable reference to an immutable one. But the reverse is *not* possible: immutable references will never coerce to mutable references. Because of the borrowing rules, if you have a mutable reference, that mutable reference must be the only reference to that data (otherwise, the program wouldn’t compile). Converting one mutable reference to one immutable reference will never break the borrowing rules. Converting an immutable reference to a mutable reference would require that the initial immutable reference is the only immutable reference to that data, but the borrowing rules don’t guarantee that. Therefore, Rust can’t make the assumption that converting an immutable reference to a mutable reference is possible.
