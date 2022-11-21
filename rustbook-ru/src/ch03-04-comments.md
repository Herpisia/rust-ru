## Комментарии

Все хорошие программисты, создавая программный код, стремятся сделать его простым для понимания. Бывают всё же случаи, когда дополнительное описание просто необходимо. В этих случаях программисты пишут заметки (или как их ещё называют, комментарии). Комментарии игнорируются компилятором, но для тех кто код читает — это очень важная часть документации.

Пример простого комментария:

```rust
// Hello, world.
```

В Rust комментарии должны начинаться двумя символами `//` и простираются до конца строки. Чтобы комментарии поместились на более чем одной строке, необходимо разместить `//` на каждой строке, как в примере:

```rust
// So we’re doing something complicated here, long enough that we need
// multiple lines of comments to do it! Whew! Hopefully, this comment will
// explain what’s going on.
```

Комментарии могут быть размещены в конце строки имеющей код:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Но чаще вы увидите их использование в следующем формате, когда комментарий размещён на отдельной строке над кодом, который комментируется:

<span class="filename">Файл: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Также в Rust есть другой тип комментариев — документирующие комментарии, которые мы обсудим в разделе "Публикация пакета на Crates.io" Главы 14.
