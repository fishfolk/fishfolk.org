---
title: Motivation
template: docs/page.html

extra:
    toc: true
    lead: Why create a meta-engine layer and custom ECS on top of the Bevy game engine?
---

There are two key motivations for making Bones ECS: (1) Rollback networking and (2) Scriptable mods. We're hoping this architecture will also result in better ergonomics for general Fish-game development, but that has not been our focus for this initial iteration.

## Rollback Networking Support

For Bones games we've decided to go with a rollback networking solution, currently powered by [GGRS]. There are two major features you need to make rollback networking work: **snapshot/restore** and **deterministic gameplay**. Both of these features [present challenges in Bevy](https://github.com/fishfolk/jumpy/discussions/489).

[GGRS]: https://github.com/gschup/ggrs

### Snapshot/Restore

Snapshot/Restore in Bevy can work, but it is also impossible to snapshot several different aspects of Bevy. You must individually register all of the resources and components that you want to be able to snapshot, and you must also avoid anything that uses local system state, such as event readers.

### How `bones_ecs` helps 🦴✨

Everything in Bones ECS can be trivially snapshot/restored by simply `.clone()`-ing the `World`. If it can't be cloned, it can't go in the world, which makes it trivial to ensure that everything we do is compatible with snapshotting.

For things that don't change across the game execution, we can avoid costly `.clone()`-ing by wrapping the types in an `Arc` ( atomic ref-count ), which means the data itself doesn't get cloned, just the reference to it.

### Determinism

The Bevy ECS is primarily designed for performance, not determinism, though this may improve over time. Currently the major cause of non-determinism is the query iteration order.

Getting deterministic query iteration requires making a query, collecting the result into a `Vec` and sorting the result by some component in this query. This greatly reduces ergonomics and is also easy to forget to do.

Every time we miss a potential determinism issue, it creates a networking sync bug that can be extremely difficult to find the root cause of, and results in searching the code and guessing where the non-determinism might have been introduced. As the codebase grows, this becomes harder and harder to do.

This also makes things more difficult for new contributors. It's actually already happened that new contributors have added features that work totally fine in local play, but either depended on features that don't support snapshot/restore in Bevy or somehow added non-determinism which was only discovered after merge.

### How `bones_ecs` helps 🦴✨

Bones ECS is deterministic by default. Internally it's extremely simple and always iterates deterministically.

It's worth noting that Bones ECS also doesn't have multi-threading. Multi-threading is inherently non-deterministic, though there are still times that it could be leveraged without producing non-determinstic simulation results. Multi-threading is something our simple games probably will not need for performance, but the possibility has been left open for multi-threading in Bones, and it would not be a very difficult feature to add if we needed it.

## Modding/Scripting Support

### Previous Attempt: `bevy_mod_js_scripting`

Scripting in Bevy is quite difficult. One of the most flexible solutions, which we have made extensive contributions to is [`bevy_mod_js_scripting`].

It leverages Bevy's reflect functionality, and allows creating bindings into the JS scripting environment to perform arbitrary actions on the Bevy world through JS. When it's complete it will probably be one of the most usable and powerful scripting interfaces to Bevy, but it is still tricky to figure out how to make JS interact with the Bevy world, and when there isn't already a way to do something you need, you have to manually craft new bindings Rust functions on the Bevy world. This is not the fault of `bevy_mod_js_scripting`. It is simply a fact that Bevy's API is difficult to create a complete scripting API for.

We also ran into performance issues.

While the scripting speed was totally sufficient for local play, when running in rollback networking mode, we may need to run up to 8 frames in a 16ms time period. The script bindings were too slow to be able to do everything we needed in that short period of time.

[`bevy_mod_js_scripting`]: https://github.com/jakobhellermann/bevy_mod_js_scripting

### How `bones_ecs` helps 🦴✨

Bones ECS is [designed to be extremely simple](https://github.com/fishfolk/jumpy/discussions/510) in its API. This dramatically reduces the complication in creating a scripting API. [The plan](https://github.com/fishfolk/jumpy/discussions/489#discussioncomment-4326596) is to create a simple C API to Bones ECS that can be used to bind practically any scripting language to.

While this won't allow scripts to modify anything in Bevy, it will allow them to change anything that our core game logic can change. So if the game does it, a mod can do it to.
