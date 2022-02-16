---
layout: post
title: "What's up with Rust's trait objects?"
date: 2022-02-15 14:10:00 +1000
categories: rust
---
If I had to describe Rust in just one word today I think it would be "quirky". I love Rust, but it's sure got a _lot_ of quirks. This post is a quick rundown of some of those quirks in a specific feature. If you've worked with that feature yourself they'll be all too familiar to you. There's no new ground covered here.

What's a quirk? It's a kind of inconsistency where the intuition you've built from what you've seen before suddenly leads you astray. You end up in a place that doesn't make sense and forces you to add funny little exceptions to your mental model to avoid going back there in the future. Quirks could come in the unexpected crossing from one concept to another, or could just represent gaps that haven't been filled yet. There are good reasons for a lot of the quirks you find in Rust, but some features are certainly quirkier than others. Maybe the quirkiest of all is the humble trait object, or `dyn Trait`.

With `dyn Trait` you can abstract over traits. A `dyn Trait` is itself a single concrete type that represents an erased value with methods dispatched at runtime. Syntactically they look a lot like `impl Trait`:

```rust
fn impl_trait(i: impl Input) -> impl Output {}

fn dyn_trait(i: &dyn Input) -> Box<dyn Output> {}
```

...except are totally different (`impl Trait` is a source of its own quirks). Your intuition about how `impl Trait` works doesn't apply to `dyn Trait`.

Firstly, if you want a `dyn Trait` you have to perform a cast first. You might not even know this is happening. It could look as simple as:

```rust
let concrete: &str = "my string";

let erased: &dyn Debug = concrete;
```

That doesn't actually compile though. In order to cast to a `dyn Trait` you need a reference to a sized (`where Self: Sized`) value. That unfortunately rules out casting `str`, which is one of the most common Rust types you're likely to encounter. You can't take a `&'a str` and cast it to a `&'a dyn Trait`.

Since trait objects are themselves unsized you need to pass them using a suitable container. Since containers tend to carry ownership semantics with them you now need to make additional decisions on behalf of your callers. That's why our example `dyn_trait` accepts `&dyn Input` instead of just `dyn Input`, and produces `Box<dyn Output>` instead of just `dyn Output`. It's also why the issue with casting `str` is so significant. That `'a` lifetime might be _really_ important, and not being able to retain it could bring your whole lifetime sandcastle crashing down.

While we're also looking at `impl Trait`, another quirk comes from adding additional bounds to a `dyn Trait`. With `impl Trait` you could write:

```rust
fn impl_trait(i: impl Input + Debug) -> impl Output {}
```

With `dyn Trait` you might also expect to write:

```rust
fn dyn_trait(i: &(dyn Input + Debug)) -> Box<dyn Output> {}
```

That doesn't actually compile though. You can't create a trait object with multiple trait bounds. You need to work around it by defining your own composite frankentrait.

There are others around `impl Trait` and `dyn Trait` specifically (leaking of auto-traits is an honorable mention) but the last quirk I want to look at is object-safety. `dyn Trait` only supports a subset of all the things you can define using normal traits. Not all traits can actually be represented as `dyn Trait`. For example, this is object-safe:

```rust
trait ObjectSafe {
    fn by_mut(&mut self, v: i32);
}
```

But this is not:

```rust
trait NotObjectSafe {
    fn by_value<T>(&mut self, v: %);
}
```

> Note: a previous version of this example used `&self` vs `self`, but this example with generics is more direct.

Further restrictions are still incoming with `async`/`await` support in traits.

These quirks all taken together always make me nervous about exposing `dyn Trait` directly in public APIs. They feel too raw, too opinionated, too... quirky. So I always end up wrapping the `dyn Trait` in a newtype to try keep some wiggle room:

```rust
pub struct Wrapper<'a>(&'a dyn ErasedTrait);

impl<'a> Trait for Wrapper<'a> {}

pub fn dyn_trait(i: Wrapper);
```

It's not that `dyn Trait` is bad, just that it's quirky. To me, it feels a lot like `struct`s in .NET used to (and still do sometimes). They're not a foundational tool. They're a niche tool for solving a niche problem. There's nothing wrong with that, but they give a glimpse of much greater untapped potential. If only we could iron out a few of the quirks. For `dyn Trait` at least, there is active and ongoing work in that area.

It's also not even that quirks themselves are bad. In some sense these gaps you find in Rust that need filling in are a positive sign of its methodical march forwards. I for one though am super excited for that future Rust with even just a few less quirks.
