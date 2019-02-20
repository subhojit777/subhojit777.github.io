---
title: My experience with Rust
description: My experience with Rust
date: '2019-02-20'
author: Subhojit Paul
#tags:
# - "rust language"
# - "programming"
---

I started learning [Rust](https://www.rust-lang.org) in 2018. I completed my work in an [open source project](https://github.com/subhojit777/dc_ajax_add_cart) and was thinking about learning a new programming language. My motive was to learn a language that allows you to control the lower level of a [high-level programming language](https://en.wikipedia.org/wiki/High-level_programming_language). I considered learning Golang, but, in most of the online articles I learned that Rust (being a system programming language) gives you more control than Go, however, the learning curve is far steeper than Go. I had no worries about deadlines or time, therefore I chose Rust.

## Learning Rust
I started learning Rust by following ["The Book"](https://doc.rust-lang.org/book). I found that there are certain concepts, (like, lifetime), which are explained later in the book but should have been explained before. I was working on an exercise and spent around a month battling with the compiler errors. I referred [Programming Rust](http://shop.oreilly.com/product/0636920040385.do) and found that it has focussed more on the concepts, and explained them very well. I soon got unblocked in the exercise. However, recently, I was referring to the 2018 version of the book and found that it has much improved. It is up to date with the recent developments in Rust. I finished the books, and the outcome was [minigrep](https://github.com/subhojit777/minigrep).

## Post-learning
I always wanted to write a PHP parser. After completing the books, I started looking into writing parsers in Rust. I came across [tagua-vm parser](https://github.com/tagua-vm/parser) and [nom](https://github.com/Geal/nom). I tried to parse a simple PHP script. But, it was too difficult. Parsers are completely new to me. I faced difficulty in installing tagua-vm parser, and it was not updated as per the recent development in Rust compiler and nom. I tried to fix them, but quickly got overwhelmed.

Instead of writing something which I don't know about, I thought about writing something in Rust which I am already good at - the web! I worked on a [hobby project](https://github.com/subhojit777/questionnaire) which is written in JavaScript. I decided to write the backend in Rust. I am still working on it. The progress is slow, but I am seeing improvements. I am getting better at analyzing and fixing the compiler errors. [diesel.rs](http://diesel.rs) and [actix.rs](https://actix.rs) are very well documented, and the community is active and helpful - that is helping me a lot.

## Choosing IDEs
I started writing Rust in Vim. I used the plugin [rust.vim](https://github.com/rust-lang/rust.vim). However, I found that looking up functions and `struct`s were slow. I tried VSCode with [rls](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust), it was better than rust.vim, but the autocompletion was not working well. I tried IntelliJ IDEA with [IntelliJ Rust](https://intellij-rust.github.io). It worked better than the other two. The lookups were fine, autocompletion was smoother than rls. I liked how it shows the inferred types, I found it quite helpful. I decided to stick with IntelliJ IDEA.

## Challenges and difficulties

### Lifetimes, traits, generic types

I am originally a PHP developer, working on a framework called [Drupal](https://drupal.org) full time. Lifetimes, generic types, traits, etc. (I can go on an on) are completely new to me. The Rust syntax looked intimidating at first, with all those lifetimes and generic types, but I am getting better at reading and understanding Rust code.

### Battle with compiler

Analyzing the compiler errors, and try to understand the core problem - I think this is an important skill you need if you are learning Rust.

```rust
pub fn authorize_get(req: &HttpRequest<AppState>) -> Box<Future<Item = HttpResponse, Error = Error>> {
    let state = req.state().clone();

    Box::new(req.oauth2()
        .authorization_code(handle_get)
        .and_then(move |request| state
            .endpoint
            .send(request)
            .map_err(|_| OAuthError::InvalidRequest)
            .and_then(|result| result.map(Into::into))
        )
        .or_else(|err| Ok(ResolvedResponse::response_or_error(err).actix_response()))
    )
}
```

The above code gave a compiler error, that it is unable to analyze the lifetime of the response and it is incorrect. I think I spent around a month chasing the error and trying to fix it. Finally, I managed to fix it with:

```rust
pub fn authorize_get(req: &HttpRequest<AppState>) -> Box<Future<Item = HttpResponse, Error = Error>> {
    let state: AppState = req.state().clone();

    Box::new(req.oauth2()
        .authorization_code(handle_get)
        .and_then(move |request| state
            .endpoint
            .send(request)
            .map_err(|_| OAuthError::InvalidRequest)
            .and_then(|result| result.map(Into::into))
        )
        .or_else(|err| Ok(ResolvedResponse::response_or_error(err).actix_response()))
    )
}
```

The only difference is:

```rust
let state: AppState = req.state().clone();
```

The compiler was incorrectly inferring the type of `state`, and explicitly mentioning the type fixed the problem. I believe the compiler was telling the high-level problem, and I was unable to understand the core issue. Rust is getting better at compiler error messages. Hopefully, I will get better at understanding compiler errors with time. The fully documented [error index](https://doc.rust-lang.org/error-index.html) also helped me many times.

### Wrong decision

When I started writing a parser in Rust, the concept about parsers was completely new to me, plus the whole new Rust syntax and things got complex very quickly. I had to abandon it. Instead, I have started writing the backend of one of my past [hobby project](https://github.com/subhojit777/questionnaire) in Rust. Earlier it was written in JavaScript, the project is web-based which I am already good at, and I can only focus on Rust.

## Things I ~~liked~~ loved in Rust

### Concept of lifetime

Rust being a low-level language, it does not supports [GC](https://en.wikipedia.org/wiki/Garbage_collection_(computer_science)), and you have to take care of the scope of variables. I have so far worked with high-level programming languages, which are GC enabled. I found this change difficult to understand at first, but slowly realized its usefulness. Taking care of the scope, might not be relevant in high-level languages, but it makes you understand how programming languages handle acquiring and releasing memory.

### Strict typing

This might not sound new to everyone. I am full time working with PHP, and in the major part of my career, I worked with PHP 5.x. PHP 5.x does not support strict typing, but things are improving in PHP 7.x. Rust follows strict typing discipline and I found it very useful. It makes sure that the data always has the correct type, and you don't have to worry about incorrect type coercions.

### Raise problems beforehand, rather than fixing it later

In brief, "Rust does not allow you to execute bad code".

The necessary checks like - lifetimes, `impl`-mentation of `trait`s if you have used any, strict typing, etc. - all of these prevent you to write bad code. In the example, I mentioned before, although the compiler error was cryptic, it pointed out that you have to explicitly mention the type. However, in most high-level languages, it automatically does the inference (for the sake of simplicity) and that's when shit happens.

TBH I am still fighting with such strict-ness of Rust, and sometimes find it frustrating - but I have found it interesting at the same time. Also, this has inspired me to write code in a similar way in my job.

### Helpful community

The community is full of smart and helpful people. The language is growing, it has a wide scope - from embedded to the web, and slowly getting accepted in [major organizations](https://www.rust-lang.org/production), there are many actively maintained libraries and I have gotten fast and helpful response for my queries.

## What's next

Due to its benefits, I would like to continue learning and writing code in Rust. It has a steep learning curve, battling with compiler sometimes becomes frustrating, but I believe that it will help me to become a better programmer. Currently, I am writing a [hobby project](https://github.com/subhojit777/questionnaire-rs) in Rust, but I would very much like to write Rust as part of my job.
