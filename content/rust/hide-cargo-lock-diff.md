+++
title = "Hide Cargo.lock in git Diff"
date = 2020-06-28
author = "Mat"
draft = true

[taxonomies]
tags = ["rust"]
+++

Whenever you add or update dependencies in `Cargo.toml`, the diff output in `Cargo.lock` can be verbose and distracting when I'm looking at `git diff` output

<!-- more -->