My second post, and another one on Rust!  
I've been spending a lot of time playing with programming languages and thinking about how I might use them in practical situations.

Rust is an excellent candidate for an embedded language. In a larger project, it provides safety, which helps protect against memory issues, and speed, which keeps the host platform responsive by letting Rust do the heavy lifting.

# Creating A Rust Dynamic Library
So, let's jump right in to it with our new, revolutionary, Rust library:
```rust
// mylib.rs
#![crate_type = "dylib"]

#![feature(libc)]
extern crate libc;
use std::ffi::CStr;

#[no_mangle]
pub extern "C" fn hello(name: *const libc::c_char) {
    let buf_name = unsafe { CStr::from_ptr(name).to_bytes() };
    let str_name = String::from_utf8(buf_name.to_vec()).unwrap();
    println!("Hello {}!", str_name);
}
```

We can compile this with:
```bash
$ rustc mylib.rs
```
And out comes The Miracle: our dynamic library, `libmylib.so`.

Let's go through everything that's happening:
```rust
#![create_type = "dylib"]
```
We are setting our [create type](http://doc.crates.io/manifest.html#building-dynamic-or-static-libraries) to be a dynamic library (opposed to `lib`, `rlib`, or `staticlib`), so we can link to it from our dynamically running code.

```rust
#![feature(libc)]
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
I spend most of my time working in JavaScript/nodejs for my day job, so let's go over how we can call in to our Magnificent Rust Library.

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

This post and the generator was inspired by [Zbigniew Siciarz's blog post on calling Rust from other languages](http://localhost:9000/articles/embedded-rust).

If you have any questions/suggestions feel free to comment, [open an issue](https://github.com/oppenlander/generator-rust-ffi/issues), or [make a pull request](https://github.com/oppenlander/generator-rust-ffi/pulls).
