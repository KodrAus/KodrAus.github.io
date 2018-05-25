---
layout: post
title: "A shared, mutable ecosystem"
date: 2018-05-24 07:49:12 +1000
categories: rust
---
Ownership is a fundamental piece of Rust's story. It amounts to a tight set of rules about who owns a value in a program, how that value can be aliased and mutated, and when that value is dropped. It prevents shared mutable state, which is the root cause of major bugs in software written without the same guarantees.

In this post I'd like to talk about a different kind ownership in though. I'd like to talk about ownership of libraries in the Rust ecosystem and the problem of sustainable maintainership.

I think the key to unravelling this problem is accepting change as a feature of our open source ecosystem and working through the implications of change instead of trying to prevent it. One of those implications is that, unlike our programs, we want shared, mutable state in our ecosystem to achieve more sustainable consumption as changes unfold.

With that in mind, I’d like to outline a plan the [Ecosystem Working Group] has to support a shared, mutable ecosystem. We'll start with some specific libraries in the nursery, but encourage branching out into the rest of the ecosystem. Before we get into that process I’d like to explore the problem in detail first.

# Library ownership

Unsustainable consumption is a problem that's common and pervasive to all open source communities. As consumers, when we participate in an open source ecosystem we tend to do so through some big assumptions. We assume that maintainership is constant. We assume that effort is invested indefinitely into open source projects in a steady fashion. We assume the person who helped us solve a problem yesterday will help us solve a problem tomorrow. We assume things don’t change.

These assumptions don’t align with reality. Something that happens in open source is that folks get burned out, priorities shift, obscure libraries become popular and popular libraries fade into obscurity. In reality, everything changes.

We don't want people to burn out or feel like the waves of enthusiasm that help their creations set sail eventually also scuttle them on sharp rocks.

We want to help consumers of Rust libraries understand the dynamics of its open source ecosystem and what it means to use software provided _as-is_. We want to help maintainers of Rust libraries feel confident in their personal involvement and the capacity of the ecosystem to absorb change.

We want to help libraries in the Rust ecosystem mature their maintainership. We want to create a shared, mutable ecosystem by distributing ownership, trust and collaboration. Taking time out from swathes of open source should feel natural, rather than a last-ditch effort to claw back sanity.

# Distributing ownership

Our open source maintainership is not sustainable when we assume things don’t change. How might we organize our ecosystem if we accept its fluidity? What impact would that acceptance have on the pressure maintainers feel to keep a consistent pace? What impact would that acceptance have on the way consumers interact with maintainers of open source projects?

The process outlined below is a concrete attempt to incorporate change into project structures that we can apply to some libraries in the Rust nursery. The idea is to take a library that's currently in the nursery and set up an independent GitHub organization around it with a shepherd and team of new maintainers. We make this all scale by sharing experience in a set of resources to encourage other library maintainers to restructure their projects into a more sustainable form that suits their needs.

## The process

We take:

- A library.
- A common goal. It could be something like stabilize `1.0`.
- A shepherd. That’s an experienced maintainer who can help weigh in on feature requests, review pull requests and occasionally touch base with the rest of the team. It’s a lot like leading a team in other software projects.
- A team of more than one person that’s interested in maintaining the library. The more the merrier.

With these things:

- We set up an independent GitHub organization with the library and team.
- We set up a Gitter hub for the team to communicate.
- We set up a code of conduct and contributing docs.
- We offer a path for new maintainers to get on board.
- We maintain!

## Why not collect libraries under a single umbrella organization?

Why do we want to set up individual organizations instead of collecting libraries under a single one? Essentially, separate organizations seem like the right initial granularity because it gives libraries room to move.

The larger and less cohesive a set of libraries gets, the harder it is to give all of them the individual attention they need on an on-going basis. The Rust nursery is a current example of this issue.

Distribution offers a framework for maintainership that doesn't create a clique or deep hierarchies. Keeping teams shallow and localized makes growing contributors easier.

Focusing on independent organizations makes the process more adaptable to different circumstances. If we bake our solutions to sustainable maintainership into a single organization then we don’t have anything to offer maintainers in the ecosystem at large besides shifting their libraries into that one organization. That might not always be appropriate, especially for larger projects.

Distributing ownership can have some drawbacks too. A single organization of high-quality libraries offers broad visibility for both users and contributors that makes the libraries more discoverable. We can increase discoverability through other centralized resources, like the crates.io package repository and the Cookbook. Visibility isn't just a driver of discovery though, it also has implications for trust. I'd like to dig into these more in the following section.

# Distributing trust

Trust is hard. What do you do if someone socially engineers themselves access to an organization with permission to push a large number of crates and publishes versions with a malicious build script? What do you do if someone takes on maintenance for a popular library and _ruins it_?

Let’s dig into these issues in a bit more detail before discussing some of the tools we have to mitigate them.

## Trusting development

What if someone takes a popular library and _ruins it_? Most folks don’t want to _ruin_ libraries. They’re cautious about making changes and afraid of being called out for making mistakes. The risk is some amount of ecosystem churn and spreading a sense of unreliability. That's not ideal, but it's also not the-end-of-the-world.

Rust deals with trust by creating multiple opportunities for all changes to be reviewed by multiple people. Getting more viewpoints at the right time in the development process through RFCs, implementation, and stabilization helps make trade-offs explicit.

Once the design space has been explored though there’s still the challenging task of actually making a decision. Without the scars of previous bad decisions it can be immobilizing to be confronted with multiple equally valid paths forward.

Maintaining libraries over time requires a wide range of skills, but they can all be taught. Experienced maintainers aren’t born that way. We can build trust in new maintainers by supporting them.

## Trusting packages

What if someone with access to popular crates publishes malicious versions? If privileged access is shared too easily then it’s possible for someone to socially engineer themselves into a position to cause a lot of damage to the Rust ecosystem. That’s a legitimate security concern.

I don't want to dig into the problem here, because it's a problem regardless of how we deal with ownership in the ecosystem. It's an ongoing conversation.

# Distributing collaboration

We’ve discussed trust as an issue that becomes more significant as ownership becomes more distributed. Sharing trust doesn’t have to be a risk though. We can achieve distributed trust through a sprinkling of additional collaboration. While organizations are independent, collaborative tools link the organizations’ teams and the wider community.

I’d like to discuss two such collaborative measures we can use to maintain trust. From inside the team, shepherds fosters more experienced maintainers in the community. From outside the team, library evaluations offer the wider community a chance to participate in steering the project’s direction.

## Shepherds

People with experience maintaining libraries who can help mentor others. Each organization should include a shepherd. The Rust community already has a pretty strong mentoring culture which we should continue to support.

The effort invested on a shepherd’s part will depend on what they’re comfortable with. They might want to actively work on the library, just respond to mentions or just check in occasionally.

## Library evaluations

Library evaluations are a process that came from the Rust Libs teams 2017 initiative: the [libz blitz]. It involves collaborating with the Rust community to run a fine-tooth comb over the public API of a library and evaluating it against current idioms and resilience to future changes. It’s also a chance to talk about where a library lives within its domain.

Evaluations give the community at large a chance to offer new perspectives on a library and drive contributions towards polishing documentation.

Using library evaluations as a gate to stabilization makes them a safety net for both the maintainers and the community.

# Next steps

I’ve spent some time discussing change as an inherent property of an open source ecosystem and the problem of not working through its implications. I’ve outlined a process we’d like to apply to some libraries in the Rust nursery that attempts to set them up to naturally cope with change. Now I’d like to invite you to get on board and help make that happen.

---

**Let’s start with [`bitflags`], a very widely depended on library that needs a new home. Reach out [here][bitflags-maintainer] if you're interested in shepherding and/or maintaining `bitflags`.** Note that `bitflags` has already been through a library evaluation and stabilized. As such a fundamental crate, on-going stability and compatibility is really important but there's still scope for new features and docs to make working with flags better.

**Do you have a library you think needs more support growing its maintainership? Open a GitHub issue [here][other-libs] if you've got any thoughts.**

**Also see the [open set of issues][ecosystem-wg-issues] and leave your thoughts on anything there or open new issues for other ideas.**

---

There’s not a lot that’s novel in this post or process. Maintainers have been creating organizations around their libraries for a long time. As far as sustainability goes, ultimately it’s a combination of the innumerable tactical initiatives that make building and maintaining Rust libraries easier and a shared understanding of how people participate in open source that makes it sustainable.

One of the goals of describing this process is so that other folks can take and adapt it to suit their own needs. There’s a lot here we should talk about more. There’s no one-size-fits-all approach. I’d love to hear from our maintainers, contributors, and users in the community! Let’s all work towards a mature and sustainable ecosystem.

[Discuss on reddit][reddit-post]

[`bitflags`]: https://crates.io/crates/bitflags
[bitflags-maintainer]: https://github.com/rust-lang-nursery/ecosystem-wg/issues/2
[ecosystem-wg-issues]: https://github.com/rust-lang-nursery/ecosystem-wg/issues
[other-libs]: https://github.com/rust-lang-nursery/ecosystem-wg/issues
[libz blitz]: https://blog.rust-lang.org/2017/05/05/libz-blitz.html
[Ecosystem Working Group]: https://github.com/rust-lang-nursery/ecosystem-wg
[reddit-post]: https://www.reddit.com/r/rust/comments/8lwvc0/a_shared_mutable_ecosystem_ft_help_maintain/
