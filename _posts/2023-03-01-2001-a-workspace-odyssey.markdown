---
layout: post
title:  "2001: A Workspace Odyssey"
date:   2023-10-22 12:46:00 +0300
categories: cargo
---

Experimentally, I've come to the conclusion that a dependency that is a Cargo workspace does _not_ read the Cargo.toml workspace file.  Instead, when a `[dependencies]` line is added that points to a Git repository, for example, which is a workspace, the repository is searched for the Cargo.toml file(s) of the workspace member.  To quote the documentation [specifying-dependencies](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html):

> "Cargo will fetch the git repository at this location then look for a Cargo.toml for the requested crate anywhere inside the git repository (not necessarily at the root - for example, specifying a member crate name of a workspace and setting git to the repository containing the workspace)"

You might wonder why this matters.  Well, for example, the other day I was trying to use a dependency at a specific commit for a project that was a Cargo workspace and the build would break.  After a frustrating amount of time, I realized the issue was that the workspace specified a `[patch]`; however, since the workspace Cargo.toml file is seemingly ignored, the `[patch]` line must also be added to my project's Cargo.toml file.

Here's an example project structure:

*External Git repo (repo_one)*
```toml
[workspace]
members = [A, B, C]

[patch.crates-io.C]
path = "C"
```

*My Git repo (repo_two):*
```toml
[dependencies]
A = { git="https://repo_one", rev="70cf45" }

[patch.crates-io.C]
git="https://repo_one"
rev="70cf45"
```

`repo_one` is the workspace that my project `repo_two` depends on.  Initially, I tried only specifying the dependencies line without the `[patch]`.  Even though I don't care to use anything from members `B` or `C`, when `A` is included, it compiles and then looks for dependencies in `C` since `A` has a Cargo.toml file that points to `C`.  If `A` depends on `C` and since `C` is on `crates-io`, without the `[patch]` Cargo tries to fetch `C` from Crates.io instead of the repository at the specified Git commit.  Therefore, to fix the build so that both `A` and `C` use the same commit, a `[patch]` line can be added to `repo_two` (my project) so that it will be sure to fetch `C` from the specified Git commit as well.

More about using `patch` can be found [here](https://doc.rust-lang.org/cargo/reference/overriding-dependencies.html).

