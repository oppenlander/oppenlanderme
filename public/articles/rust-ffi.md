I've been spending a lot of time playing with programming languages and thinking about how I might use them in practical situations.

Rust's speed and safety make it an excellent candidate for an embedded language. By calling into Rust for expensive operations, you can help keep your host platform responsive.

# Creating A Rust Dynamic Library
So, let's jump right in to it with our new, revolutionary, Rust library.  
Start by starting a new Rust library with cargo:
```bash
$ cargo new mylib && cd mylib
```

Replace the contents of your `Cargo.toml` with:
```toml
# Cargo.toml
[package]
name = "mylib"
version = "0.1.0"
authors = ["First Last <your@email.com>"]

[dependencies]
libc = "0.1"

[lib]
name = "mylib"
crate_type = ["dylib"]
```

And replace the contents of `src/lib.rs` with:
```rust
// src/lib.rs
#![crate_type = "dylib"]

extern crate libc;
use std::ffi::CStr;

#[no_mangle]
pub extern "C" fn hello(name: *const libc::c_char) {
    let buf_name = unsafe { CStr::from_ptr(name).to_bytes() };
    let str_name = String::from_utf8(buf_name.to_vec()).unwrap();
    println!("Hello {}!", str_name);
}
```

We can now build with:
```bash
$ cargo build
```
And out comes The Miracle: our dynamic library, `target/debug/libmylib.so`.

Let's go through everything that's happening in `Cargo.toml` and `src/lib.rs`:

## Cargo.toml
First we have basic information about this crate:
```toml
[package]
name = "mylib"
version = "0.1.0"
authors = ["First Last <your@email.com>"]
```

The `libc` pacakge is currently flagged as unstable, but we can still use it through the cargo library.  
Luckily, the [libc crate](https://crates.io/crates/libc) is available in lue of a bundled module:
```toml
[dependencies]
libc = "0.1"
```
Later, when we build, cargo will pull in our dependencies and build our library for us.

We also want to tell cargo we are making a dynamic library, and want to call it with the ever-unique "mylib": 
```toml
[lib]
name = "mylib"
crate_type = ["dylib"]
```
Rust has multiple types of libraries it can build: `rlib`/`lib` (default), `dylib`, and `staticlib`.  
If we want to dynamically load the library in an interpreted langague, we want to build with `dylib`.

## src/lib.rs
```rust
extern crate libc;
use std::ffi::CStr;
```
Here is where we are importing some useful external libraries: [libc](http://doc.rust-lang.org/libc/libc/index.html) and [ffi](http://doc.rust-lang.org/std/ffi/index.html).
- `libc` helps us abstract platform-dependent C types to useful Rust types. For example, on my platform `c_char` is a type definition for `i8`.
- `ffi` helps with the actual conversion between C types and Rust types.

```rust
#[no_mangle]
pub extern "C" fn hello(name: *const libc::c_char) { ... }
```
This is our C-compatible function declaration:
- `#[no_mangle]` ensures the exported function signature won't be mangled by the Rust compiler.
- `extern "C"` specifies the ABI we are exporting for.
- `name: *const libc::c_char` a type signature for a CString

```rust
let buf = unsafe { CStr::from_ptr(name).to_bytes() };
let str_name = String::from_utf8(buf.to_vec()).unwrap();
println!("Hello {}!", str_name);
```
Here we get into some unsafe code so we can extract bytes out of the CString's raw pointer. We then convert the bytes into a String, and print out our _very_ important message.

# Calling Rust From Node
I spend most of my time working in JavaScript/nodejs, so let's go over how we can call in to our Magnificent Rust Library.

```js
// mylib.js
var FFI = require('ffi');

var lib = FFI.Library('./libmylib', {
  'hello': [ 'void', [ 'string' ] ]
});

lib.hello('Rust');
```
The JavaScript code is fairly straightforward, with the help of [node-ffi](https://github.com/node-ffi/node-ffi). We just need to tell it where our dynamic library is and stub out what the C ABI function signature look like. The signature uses the C types from [ref](http://tootallnate.github.io/ref/), which is used internally in `node-ffi`.

# Generator Rust-FFI
To ease the process of creating Node -> Rust libraries, I created a yeoman generator, [Rust FFI](https://github.com/oppenlander/generator-rust-ffi).

To use, simply install yeoman and generator-rust-ffi through NPM:
```bash
$ npm install -g yeoman generator-rust-ffi
```
And run the generator:
```bash
$ yo rust-ffi
```

The idea of this project is to have Rust libraries with bindings into multiple languages. These bindings should follow good practices in the module/package structures and testing strategies, specific to whatever language the binding is for. The same repository should also be a valid package for the binding language's native package manager.

I've implemented this boilerplate and example bindings for JavaScript/nodejs in *Rust FFI*. All JavaScript library and testing code is stored in the `js` folder and it is a valid NPM module, which will compile the Rust dylib on install.

The decision to use [Yeoman](http://yeoman.io/) was simply because I haven't seen a better tool for bootstrapping project structures.

This post and the generator was inspired by [Zbigniew Siciarz's blog post on calling Rust from other languages](https://siciarz.net/24-days-of-rust-calling-rust-from-other-languages/).

If you have any questions/suggestions feel free to comment, [open an issue](https://github.com/oppenlander/generator-rust-ffi/issues), or [make a pull request](https://github.com/oppenlander/generator-rust-ffi/pulls).

__Update 2015-05-28__  
Thank you to both [JavaScript Jabber](http://devchat.tv/js-jabber/161-jsj-rust-with-david-herman) and [DailyJS](http://dailyjs.com/2015/04/14/1419-node-roundup/) for the mentions.  
I've updated the article to work with Rust 1.0 and to use cargo. If you want to see the old (broken) guide, I make the [source to my site available on github](https://github.com/oppenlander/oppenlanderme/blob/3459b59c2ea117c2a9b99a154fd1c468382136a0/public/articles/rust-ffi.md). 
