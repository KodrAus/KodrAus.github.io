---
layout: post
title: "What is the standard library?"
date: 2021-09-27 14:10:00 +1000
categories: rust
---
The year is 2021. We're 7 years on from Rust 1.0 and a few weeks out from its second edition since then. [Mara Bos has just posed this question](https://rust-lang.zulipchat.com/#narrow/stream/219381-t-libs/topic/What.20is.20the.20standard.20library.3F/near/254442108) of Rust's standard library:

> What is the standard library, or what should it be? That is, what kind of things should be part of it, and what shouldn't? What purpose does it have, and what purpose should it not have?

This post is my attempt to answer this question by exploring some of the wider context around it. These are just my own thoughts. They might have some overlap with other language ecosystems, but I'm not trying to make a statement on behalf of all standard libraries.

So to return to the original question, I think there's a key implication that I want to really unpack:

> What **is** the standard library, or what **should** it be?

It's not just a question of what the standard library is today, it's a question of what it should be tomorrow. I think change is an essential element of standard library design. The whole process is built around coping with change when you've only got limited tools to respond to them. Following this train of thought I've arrived at this as a starting point:

> The standard library is the changing realization of its unchanging goals.

That's pretty nebulous, but it's just a starting point. Let's return to change in a little bit, I'd like to address the unchanging goals first.

## Unchanging goals

What is Rust's standard library? [It's own documentation offers an answer](https://doc.rust-lang.org/1.55.0/std/index.html#the-rust-standard-library) in the very first sentence:

> The Rust Standard Library is the foundation of portable Rust software, a set of minimal and battle-tested shared abstractions for the broader Rust ecosystem.

[The last time this wording changed was in 2015](https://github.com/rust-lang/rust/pull/26977) along with the `1.3.0` release. So for almost as long as Rust has been stable its standard library has followed the same basic definition. This is what forms the unchanging basis of my definition of Rust's standard library.

### What belongs in the standard library?

Let's go back to the standard library's self-proclaimed definition. It tells us what the standard library _is_ rather than what it _is for_. Let's pull out a few key points:

> The Rust Standard Library is the **foundation** of **portable** Rust software, a set of **minimal** and battle-tested **shared** abstractions for the broader Rust ecosystem.

So, what the standard libary _is_ is the foundational, portable, minimal, shared abstractions available by default to all Rust programs. What are some of these abstractions? Let's look at a few:

We've got the [**`Sync` trait**](https://doc.rust-lang.org/1.55.0/std/marker/trait.Sync.html). It's what's called a _marker interface_ that tells the Rust compiler something about the type that implements it. In this case, implementing `Sync` tells the compiler that a type is safe to access concurrently. There's another trait called `Send` that plays into this as well, but for this exercise we're just going to keep things simple and ignore it.

We've got the [**`Arc<T>` type**](https://doc.rust-lang.org/1.55.0/std/sync/struct.Arc.html). It lets you take any `T` and share any number of independent references to it. When the last reference is dropped then `T` is too. If that `T` is thread-safe (implements `Sync`) then you can create cheap references through the `Arc` and send them to other threads.

We've got the [**`Mutex<T>` type**](https://doc.rust-lang.org/1.55.0/std/sync/struct.Mutex.html). It lets you take any `T` and get exclusive access to it through a shared reference. Only one shared reference can be upgraded to an exclusive one at a time, implementing the idea of _mutual exclusion_. It makes any type implement `Sync`.

`Arc<T>` and `Mutex<T>` **combine** together into **`Arc<Mutex<T>>`**. That combines their properties to give you a way to create independent shared references that can be safely upgraded into exclusive ones. We can even safely swap the `T` out for an entirely different one and the shared references will still only see the one-true-value.

What's something we might want to put in our `Arc<Mutex<T>>`? How about a [**`File` type**](https://doc.rust-lang.org/1.55.0/std/fs/struct.File.html)? It's kind of like a semantic shell over a raw OS handle (a file-descriptor on Unix and a handle on Windows) that gives us a Rust API to work with. We can technically interact with files without needing exclusive access, but the OS might have a different idea of what to do with concurrent writes than you do (if you've ever tried to write straight to a file from multiple threads you might see this difference in action). So if we stuff everything in a `Arc<Mutex<File>>` we've got a shared handle to a file that only a single referer can access at a time. We could be reading and writing configuration, logs, application state, temporary results, anything.

What's something all these types have in common? They're targeted independent concepts that are generally applicable but combine together into something domain-specific. They're examples of what I'll call _load-bearing abstractions_.

### Load-bearing abstractions

A load-bearing abstraction is a codified concept that composes with others to produce something specific and useful. It's not exactly revolutionary, that's pretty much just a definition of programming, but I think giving a purposeful name to a set of things makes it easier for us to talk about what's in and what's out. The thing that makes these abstractions load-bearing is that they have **broad applicability in a shallow implementation**. They have broad applicability because their concepts play a role in many others. They have a shallow implementation because they're targeted in scope. These are the kinds of abstractions that belong in the standard library.

To get a little more concrete, I think there are a few roles or layers of load-bearing abstraction in the standard library. I'll list them below from most targeted and essential to least:

1. **Fundamentals**: These are the things you need to have a language and its runtime. They're a mix of primitives like [`i32`](https://doc.rust-lang.org/1.55.0/std/primitive.i32.html), [`[T]`](https://doc.rust-lang.org/1.55.0/std/primitive.slice.html), and [`str`](https://doc.rust-lang.org/1.55.0/std/primitive.str.html), traits like [`Try`](https://doc.rust-lang.org/1.55.0/std/ops/trait.Try.html), [`Sync`](https://doc.rust-lang.org/1.55.0/std/marker/trait.Sync.html), [`Add`](https://doc.rust-lang.org/1.55.0/std/ops/trait.Add.html), [`Iterator`](https://doc.rust-lang.org/1.55.0/std/iter/trait.Iterator.html), and [`Future`](https://doc.rust-lang.org/1.55.0/std/future/trait.Future.html), and basic abstractions like [`alloc`](https://doc.rust-lang.org/1.55.0/std/alloc/index.html), [`panic`](https://doc.rust-lang.org/1.55.0/std/panic/index.html), [`mem`](https://doc.rust-lang.org/1.55.0/std/mem/index.html), and [`ptr`](https://doc.rust-lang.org/1.55.0/std/ptr/index.html). They're what _makes_ Rust.
2. **Interchange interfaces**: These are the core abstractions that link the ecosystem together. You'll usually see them in the argument and return types of public APIs. They're the things you really want a single canonical definition for so you can link otherwise independent libraries together in your own code. They're types like [`String`](https://doc.rust-lang.org/1.55.0/std/string/struct.String.html), [`Path`](https://doc.rust-lang.org/1.55.0/std/path/struct.Path.html), [`Option<T>`](https://doc.rust-lang.org/1.55.0/std/option/enum.Option.html), [`Result<T, E>`](https://doc.rust-lang.org/1.55.0/std/result/enum.Result.html), [`ControlFlow<B, C>`](https://doc.rust-lang.org/1.55.0/std/ops/enum.ControlFlow.html), and traits like [`From`](https://doc.rust-lang.org/1.55.0/std/convert/trait.From.html), [`Debug`](https://doc.rust-lang.org/1.55.0/std/fmt/trait.Debug.html), [`Read`](https://doc.rust-lang.org/1.55.0/std/io/trait.Read.html), [`Future`](https://doc.rust-lang.org/1.55.0/std/future/trait.Future.html), and [`Error`](https://doc.rust-lang.org/1.55.0/std/error/trait.Error.html).
3. **Building blocks**: These are the codified concepts that combine together to give you the semantics you need in the things you build. They're typically generic over some `T` you plug in, like [`Box<T>`](https://doc.rust-lang.org/1.55.0/std/boxed/struct.Box.html), [`Vec<T>`](https://doc.rust-lang.org/1.55.0/std/vec/struct.Vec.html), [`HashMap<K, V, S>`](https://doc.rust-lang.org/1.55.0/std/collections/struct.HashMap.html), [`Arc<T>`](https://doc.rust-lang.org/1.55.0/std/sync/struct.Arc.html), [`Mutex<T>`](https://doc.rust-lang.org/1.55.0/std/sync/struct.Mutex.html), [`Cell<T>`](https://doc.rust-lang.org/1.55.0/std/cell/struct.Cell.html), and [`MaybeUninit<T>`](https://doc.rust-lang.org/1.55.0/std/mem/union.MaybeUninit.html).
4. **Consistent Capabilities**: These are bindings to "standard" platform capabilities that makes your Rust experience portable. They're types like [`Path`](https://doc.rust-lang.org/1.55.0/std/path/struct.Path.html), and modules like [`thread`](https://doc.rust-lang.org/1.55.0/std/thread/index.html), [`fs`](https://doc.rust-lang.org/1.55.0/std/fs/index.html), and [`net`](https://doc.rust-lang.org/1.55.0/std/net/index.html) that are usually a fairly thin veneer over platform APIs. These follow precedent in other standard libraries, but err towards minimalism.

Those roles aren't mutually exclusive, lots of APIs could be in multiple roles (particularly the first few). The fundamental and interchange traits have proven to be the hardest to change, while motivations for the capability APIs are fuzzy and open-ended. APIs in all roles take a lot of design work for different reasons. Identifying the roles a proposed API belongs to can help us evaluate it more effectively.

### Identifying load-bearing abstractions

How do you find the load-bearing abstractions that support some specific use-case? I think it's an exercise in decomposition: split bigger concepts into their composing parts until you arrive at some pieces that appear multiple times within that use-case, or within other (ideally independent) use-cases. The smaller these pieces are the better. That's when you've found something that could belong in the standard library. We already do this all the time when we're building software. The key is to keep digging until you arrive at those smallest-possible-units, and then prove those ideas out in real application.

### So what's in and what's out?

The standard library is a good place to ship the core load-bearing abstractions because they're orthogonal to the domain of any specific application. You'd feel like you're on some epic yak-shave if you had to build them yourself. They make _all_ Rust programmers more productive.

Note that I added another qualifier to these abstractions: they're **core** load-bearing abstractions. The standard library doesn't need to ship everything. One of Rust's great values is that the playing field between the standard library and the wider ecosystem is more balanced than some others. There's a standard and accessible package manager, automatic documentation publishing, community standards around permissive licensing, and lots of knowledge-sharing on what makes great libraries.

Now that we've spent some time looking at what's in-scope for the standard library, I'd like to explore some things I think are out-of-scope:

- **Not a one-stop-shop**: The standard library shouldn't be a place to find ready-made solutions to all common programming tasks. I don't think the standard library should ship a HTTP client for instance. What the standard library should ship are tools that can be used to build HTTP clients (and many other things).
- **Not an automatic home for stable APIs**: The standard library shouldn't be a place to elevate stable libraries so they're automatically available to all Rust programmers. A stable API should be a pre-requisite to wholly new additions, but that doesn't mean everything that's `1.0` should be a candidate for standardization (there are lots of bigger numbers than `1` afterall).
- **Not a place for "good-enough" alternatives**: The standard library shouldn't ship simplified implementations of external libraries, frameworks, or components. What the standard library should ship are APIs that standardize fundamental concepts exposed by those components that allow them to integrate with others.
- **Not the exclusive home of the highest-quality Rust code**: This one might sound a little strange. The standard library _must_ ship high-quality implementations by necessity, but we should discourage the idea that it's _the_ place to find Rust code you can depend on. That discounts the value of the whole ecosystem and creates pressure to put things in the standard library as a statement rather than because it belongs there.

I think it's also worth giving a special mention to APIs on [`Option<T>`](https://doc.rust-lang.org/1.55.0/std/option/enum.Option.html), [`Result<T, E>`](https://doc.rust-lang.org/1.55.0/std/result/enum.Result.html), [`Iterator`](https://doc.rust-lang.org/1.55.0/std/iter/trait.Iterator.html), and others that have evolved their own microcosms. They get a lot of attention and should probably have their own guidelines for what are considered in and out of scope that's consistent across all similar types.

## Changing realization

The unchanging goal of the standard library is to make all Rust programmers productive by shipping the core load-bearing abstractions they all depend on.

With that in place, now I'd like to return to the question of change. The standard library doesn't exist in a static context. As the needs of Rust programmers change, the way the standard library realizes its goals need to change too. The definition of the standard library is tied to the needs of Rust programmers, and those needs change. The problem is that the standard library has limited tools to make changes. Existing code can only break in very specific ways (read: in ways that let standard library authors blame the language) so coping with change is tricky.

### Coping with change

What if the standard library just refused to change? Just consider the whole thing done, reject any new APIs and just focus on performance and security patches. Imagine if we did this right... _now_. As of Rust `1.55.0` the standard library cannot move. What state would we be left in?

- We'd have IO APIs that don't work with uninitialized buffers.
- We'd have IO APIs that don't work with [`async`/`await`](https://rust-lang.github.io/async-book/).
- We'd have an error handling standard that doesn't capture backtraces or appear in embedded targets.
- We'd soon have an iterator API that doesn't let you return items that borrow from the iterator.

Just to name a few. Those are all changes coming in from the language and existing additions though. What if we accepted those as papercuts and froze the language now too? What state would we be left in?

- We'd have support for x86 hardware intrinsics in [`std::arch`](https://doc.rust-lang.org/1.55.0/core/arch/index.html), and limited support for WebAssembly, but none for ARM. Those are increasingly major platforms to be lacking support for.
- Embedded and custom targets would never get access to some platform features wrapped up in the standard library.

Those aren't changes coming from within Rust itself. They're coming from the wider context Rust exists in. That context continues to change whether or not Rust wants it to.

These are also just a handful of cherry-picked examples that are relevant today. If we just make a goal of solving today's papercuts and calling it done then we'd still have new ones emerge to consider tomorrow.

Stasis is one solution to the risks of change, but it's not a solution for the standard library. What we need is to come up with better tools and approaches to managing change where breakage must be avoided. We need to be able to evolve the standard library without creating a tangled mess of equivalent APIs or jarring strata where past idioms produce distinct layers.

### Yesterday's needs

If only the standard library had been split into separate crates with their own versioning policies when Rust `1.0` was released everything would be rosy. If only the standard library had just avoided stabilizing the [`Read`](https://doc.rust-lang.org/1.55.0/std/io/trait.Read.html) trait before we had [`MaybeUninit`](https://doc.rust-lang.org/1.55.0/std/mem/union.MaybeUninit.html) we'd all be in a better place now. These are all things we can conjure up today and trace back to some previous decision that could have been made differently.

Those things might clearly be decisions we'd make differently in today's context, but that doesn't mean they were the wrong decision to make at the time. Let's say Rust _did_ split the standard library into separate crates. You'd pull in [`fs`](https://doc.rust-lang.org/1.55.0/std/fs/index.html) if you wanted [`File`](https://doc.rust-lang.org/1.55.0/std/fs/struct.File.html), and you'd pull in [`process`](https://doc.rust-lang.org/1.55.0/std/process/index.html) if you wanted [`Command`](https://doc.rust-lang.org/1.55.0/std/process/struct.Command.html). That would have come with the temptation to limit the scope of Rust `1.0` by not shipping these libraries as `1.0`. If Rust announced its `1.0` with only an unstable interface for opening files or establishing TCP connections would it have encouraged innovation? Or would it have limited mindshare? Who knows. If we made that decision then, we might not have a language to apply hindsight to today.

There are of course things we've learned about what makes a time-tested standard library API in the last 7 years that we can apply today. But chances are there will be decisions being made now that are appropriate for today, but not for tomorrow.

### Tomorrow's needs

What kinds of changes will impact the standard library in the next 5, 10, 15 years? Unfortunately I seem to have bricked my crystal ball (it just flashes red if anybody knows how to fix that?) so can't say for sure. We can't predict the exact circumstances that will come up, but we can give ourselves the best opportunity to respond in the right time to needs as they arise.

This is where I think we are with the standard library now. We have an idea of what its unchanging goals are, and a sense for what belongs in and out, but we don't yet have all the tools we need to keep satisfying those goals into the future.

## What do you think?

I've put forward some of my opinions on what Rust's standard library should and shouldn't be based on my own experiences and intuitions. While writing this all down I've discovered a lot of what I really think in the process, and some differences in the way I thought about it before. Your perspective might be totally different! I'd love to hear from others in both the Rust community and in other communities about what _they_ think their standard library should be.
