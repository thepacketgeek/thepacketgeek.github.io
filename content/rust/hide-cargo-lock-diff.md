+++
title = "Hide Cargo.lock in git Diff"
date = 2020-06-28
author = "Mat"

[taxonomies]
tags = ["rust"]
+++

Whenever you add or update dependencies in `Cargo.toml`, the diff output in `Cargo.lock` can be verbose and distracting when I'm looking at `git diff` output:

```diff
$ git diff --stat
 Cargo.lock | 54 ++++++++++++++++++++++++++++++++++++++++++++++++++++++
 Cargo.toml |  3 +++
 2 files changed, 57 insertions(+)
```

```diff
$ git diff
diff --git a/Cargo.lock b/Cargo.lock
index c287579..a41401e 100644
--- a/Cargo.lock
+++ b/Cargo.lock
@@ -151,6 +151,7 @@ dependencies = [
  "jsonrpsee",
  "log",
  "net2",
+ "pcap-file",
  "prettytable-rs",
  "serde",
  "serde_json",
@@ -344,6 +345,16 @@ dependencies = [
  "memchr",
 ]

+[[package]]
+ ... pages of [[package]] updates
```

<!-- more -->

# Hiding via Git CLI
You can temporarily hide the `Cargo.lock` diff output by adding a path filter to the `git diff` command:

```diff
$ git diff ':!*lock'
diff --git a/Cargo.toml b/Cargo.toml
index a3cd170..3d68210 100644
--- a/Cargo.toml
+++ b/Cargo.toml
@@ -41,3 +41,6 @@ tokio = {version = "^0.2.11", features=["macros", "tcp", "time", "stream"]}
 tokio-util = { version = "^0.2", features=["codec"]}
 toml = "0.5"
 twoway = "0.2.0"
+
+[dev-dependencies]
+pcap-file = "*"
```

You can also add this as an alias in your Git configuration for quick recall:

#### **`~/.gitconfig`**
```
...
[alias]
    diff-no-lock = "!git diff ':!*lock'"
```

# Hiding via a Custom Hunk Header
For a more permanent per-project or global fix, you can add special handling for `*.lock` (or any file you want) when using `git diff` by adding a [hunk header](https://git-scm.com/docs/gitattributes#_defining_a_custom_hunk_header) to customize the output.

## Add a Hunk Header
Since our goal is to not show any output for `Cargo.lock` files, our hunk header command will be `/bin/true` (or `/usr/bin/true` for MacOS):

#### **`~/.gitconfig`**
```
...
[diff "hidden"]
     command = /bin/true
```

## Target .lock Files in Git Attributes
Now we need to associate `.lock` files to use our custom hunk header. Create or edit your `.gitattributes` file:

#### **`~/.gitattributes`**
```
*.lock  diff=hidden
```

If you don't already have a `.gitattributes` file, you will also need to point your Git config at this new file:

#### **`~/.gitconfig`**
```sh
...
[core]
    attributesfile = ~/.gitattributes
```

## Success!
Now enjoy your `Cargo.update` free diff output:

```diff
$ git diff --stat
 Cargo.lock | 54 ++++++++++++++++++++++++++++++++++++++++++++++++++++++
 Cargo.toml |  3 +++
 2 files changed, 57 insertions(+)
$ git diff
diff --git a/Cargo.toml b/Cargo.toml
index a3cd170..3d68210 100644
--- a/Cargo.toml
+++ b/Cargo.toml
@@ -41,3 +41,6 @@ tokio = {version = "^0.2.11", features=["macros", "tcp", "time", "stream"]}
 tokio-util = { version = "^0.2", features=["codec"]}
 toml = "0.5"
 twoway = "0.2.0"
+
+[dev-dependencies]
+pcap-file = "*"
```

## Caveat
Adding a hunk header will affect Git patches (E.g. `git diff > ~/mychange.patch`) since it will not include any of the `Cargo.lock` changes. If you send someone a patch, that person can run `cargo update` to re-build the `Cargo.lock` file, or you can temporarily disable the hunk header in your `.gitconfig`.

