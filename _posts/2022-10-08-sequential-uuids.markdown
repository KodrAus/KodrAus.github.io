---
layout: post
title: "Sortable identifiers in `uuid`"
date: 2022-10-08 14:00:00 +1000
categories: rust
---
Before diving in, I'd like to give a huge thank you to [@rrichardson](https://github.com/rrichardson) who implemented the new functionality this post talks about.

[The `uuid` crate](https://github.com/uuid-rs/uuid) lets you generate unique identifiers across systems without them needing to coordinate. The representations of those ids and the approaches `uuid` takes to generate them are all specified as "versions" in [RFC 4122](https://www.rfc-editor.org/rfc/rfc4122). The most commonly used version is 4, which simply fills the identifier with random data. This is great for ensuring uniqueness and makes ids near impossible to guess, but isn't so great for database indexes or other scenarios that need sequential ids. Other specifications like [ULID](https://github.com/ulid/spec) have been serving these cases successfully for a while now independently of UUIDs, so there's a [new draft RFC](https://github.com/uuid6/uuid6-ietf-draft) in the works to bring sequential ids to UUIDs too. The RFC has details on how these new versions work.

As of `1.2.0`, the `uuid` crate supports the new draft UUID versions so you can start using them in your applications. Since they're still draft and subject to change we require a `RUSTFLAGS` config to opt-in to them. That makes them unsuitable for libraries right now, but once the RFC reaches a steady state we'll commit to them fully.

## Generating a sortable UUID

If you'd like your UUIDs to be sortable, the version you want is 7. It combines a Unix timestamp in millisecond precision for sortability with random data for uniqueness.

First, add the `v7` feature of `uuid` to your `Cargo.toml`:

```toml
[dependencies.uuid]
version = "1"
features = ["v7"]
```

Use the `Uuid::now_v7()` function to generate a UUID that embeds the current system time:

```rust
let uuid = uuid::Uuid::now_v7();
println!("{}", uuid);
```

Make sure you also pass the `uuid_unstable` configuration flag when running, otherwise the `now_v7()` function won't be available:

```shell
RUSTFLAGS="--cfg uuid_unstable" cargo run
```

That's it! If you give these new versions a try, or if you'd like to once they're stable please leave any feedback [on our GitHub issue tracker](https://github.com/uuid-rs/uuid/issues/523).
