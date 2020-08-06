# Rust异步手册-简体中文翻译
Asynchronous Programming in Rust- Simplified Chinese

本仓库是对[Asynchronous Programming in Rust](https://github.com/rust-lang/async-book)的 __自用__ 简体中文翻译版本，暂未翻译代码注释

___[前往阅读本书]()___

## Requirements
The async book is built with [`mdbook`], you can install it using cargo.

```
cargo install mdbook
cargo install mdbook-linkcheck
```

[`mdbook`]: https://github.com/rust-lang/mdBook

## Building
To create a finished book, run `mdbook build` to generate it under the `book/` directory.
```
mdbook build
```

## Development
While writing it can be handy to see your changes, `mdbook serve` will launch a local web
server to serve the book.
```
mdbook serve
```
