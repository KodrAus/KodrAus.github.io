---
layout: post
title: "47 seconds of fuzzing in CI"
date: 2026-06-09 09:00:00 +1000
categories: rust
---

Anytime I fuzz a piece of code for the first time I'm always surprised by how trivially broken it is. The first fuzzing run always finds bugs within a few seconds of starting. Maybe I'm just bad at programming, but I suspect that experience is common. In case you're not familiar, fuzzing is a testing technique where your program is fed (purposefully) random inputs. The goal is to remove our own biases in deciding what to test and automatically exercise all possible paths through our program. It's especially good for programs that start from unstructured input, like parsers.

I use fuzzing a lot in my dayjob, but not so much in my open source projects. The reason is mostly because a fuzzer is usually run for a long time (hours, days) to exercise complex code. Google has been running [OSS Fuzz](https://github.com/google/oss-fuzz) for a number of years that continuously fuzzes projects. I should make better use of it, but what I really want is another layer in my safety net that catches issues early in CI. That would augment any continuous fuzzing running out-of-band.

I'd actually just started playing with fuzzing in CI more in one of my less used projects when [this parser panic](https://github.com/uuid-rs/uuid/issues/885) was logged against `uuid`. Oops. We did have fuzz tests for `uuid`'s parser but they hadn't been run in a long time so this bug went unnoticed. Now we fuzz `uuid`'s parser in CI. The infrastructure amounts to [a little shell script](https://github.com/uuid-rs/uuid/tree/main/fuzz) driving [`cargo fuzz`](https://github.com/rust-fuzz/cargo-fuzz). I mentioned before that fuzzing code for the first time finds bugs within seconds, so we only run our fuzz cases for 47 seconds on each CI run. That was ample time to [find the parser bug](https://github.com/uuid-rs/uuid/pull/886#issuecomment-4657805090), plus another unreported one too.

For you other maintainers out there, I would definitely recommend looking into some kind of similar CI setup for your projects too. If you've never tried fuzzing before, there [a `rust-fuzz` book](https://rust-fuzz.github.io/book/introduction.html) that guides you through the moving parts and how to use them. You'll be amazed at what you find.
