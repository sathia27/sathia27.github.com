---
layout: post
title: Rust cargo resolve authentication issue
date: '2021-08-19'
categories: posts
published: true
tags: [rust, cargo, programming, basics, installation, setup]
---

Have you faced authentication errors while installing dependencies using `cargo`

```bash
✗ cargo build
    Updating crates.io index
error: failed to get `async-std` as a dependency of package `app-name v0.1.0 (/Users/sathia/Developer/rust-codebase/app-name)`
Caused by:
  failed to load source for dependency `async-std`
Caused by:
  Unable to update registry `https://github.com/rust-lang/crates.io-index`
Caused by:
  failed to fetch `https://github.com/rust-lang/crates.io-index`
Caused by:
  failed to authenticate when downloading repository: git@github.com:rust-lang/crates.io-index
  * attempted ssh-agent authentication, but no usernames succeeded: `git`
  if the git CLI succeeds then `net.git-fetch-with-cli` may help here
  https://doc.rust-lang.org/cargo/reference/config.html#netgit-fetch-with-cli
Caused by:
  no authentication available
```

`cargo` requires to do some sort of authentication when using dependencies from git or any registries like `crates`

### Using SSH authentication

  ```bash
➜  eval `ssh-agent -s`
Agent pid 27236
➜  ssh-add
Identity added: /Users/sathia/.ssh/id_rsa
➜  uzhaippu git:(master) ✗ cargo build
```

### Using cargo net-git-fetch

If the above authentication method fails, cargo provides a config that executes the git executable to handle fetching remote repositories instead of using the built-in support

```bash
 ➜  CARGO_NET_GIT_FETCH_WITH_CLI=true cargo build
  Updating crates.io index
  Downloaded async-attributes v1.1.2
  Downloaded async-mutex v1.4.0
  Downloaded async-executor v1.4.1
  Downloaded async-global-executor v2.0.2
  Downloaded async-lock v2.4.0
```

It should work after the above step.

```bash
✗ cargo build
    Updating crates.io index
```

### Reference

[Rust doc reference](https://doc.rust-lang.org/nightly/cargo/appendix/git-authentication.html#git-authentication)