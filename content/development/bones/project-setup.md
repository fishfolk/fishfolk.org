---
title: Project Setup
template: docs/page.html
weight: 11

extra:
    toc: true
    lead: Tips for setting up a Bones project
---

This doc is WIP -

### Configuring Git Repository

It is recommended to setup a [.gitattributes](https://docs.github.com/en/get-started/getting-started-with-git/configuring-git-to-handle-line-endings#per-repository-settings) file in project repo to force unix line endings.

Add file: `.gitattributes` to repository root with contents:
```
* text eol=lf
```

The `bones_asset` crate will compute "Content IDs" (or the `Cid` type in code) by hashing the contents of assets. Text based assets such as `.yaml` schema files risk having different line endings on different development platforms. This could result in divergent Content IDs, leading to issues replicating asset handles in online play using `bones_asset::NetworkHandle<T>`. Forcing all git checkouts to use Unix line endings with `.gitattributes` will resolve this issue.
