---
title: Jumpy v0.7 Current Status
description: An overview of the current status of Jumpy development.
template: blog/page.html
date: 2023-06-04

taxonomies:
  authors:
    - Zicklag

---

[Jumpy 0.7][v0.7.0] added online and LAN play to Jumpy, but there's still lots to do, and plenty of future potential! Here's an overview of where were at on several of the technical fronts in Jumpy.

[v0.7.0]: https://github.com/fishfolk/jumpy/releases/tag/v0.7.0

## Networking

Network games with both LAN and the internet are technically working, but there are a couple important caveats to address.

### Performance

The performance of network games is not very good yet, and can be un-playable depending on network conditions. There seems to be something wrong with our network transport that is limiting how fast we can send network messages, but so far the exact cause is unclear. We have [a plan and a tracking issue](https://github.com/fishfolk/jumpy/issues/790#issuecomment-1574957511) for finding out what's wrong and fixing the performance issues.

Part of this plan is to abstract our network transport so that we can swap it out with different implementations. This will accomplish three important goals:

- It will be easier to test different network transport implementations without changing anything else in the game, so that we can see if the performance issue is related to our networking library or configuration.
- It will allow us to implement a network transport for the browser. This will enable you to play online games in browsers that support [WebTransport](https://caniuse.com/webtransport).
- It will allow us to implement a network transport for Steam, to take advantage of Steam's networking infrastructure when run from Steam.

There are also other [possibilities](https://github.com/fishfolk/jumpy/issues/790#issuecomment-1574735131) for improving networking performance, including substep game simulations, and tailoring our networking server to minimize the required network traffic.

### De-Syncs

There are still occasional de-syncs between network players. This means that one player is playing the game and seeing one thing, while another player is playing and seeing something different. This is caused by subtle non-determinism in the gameplay code.

If you've been following Jumpy development, you might know that we specifically designed the [Bones ECS][becs] used in Jumpy to prevent non-determinism issues, so where is the non-determinism coming from!

Even though the Bones ECS is deterministic, not everything in Rust is. For instance, simply storing items in a HashMap and iterating over them is non-deterministic. So it's very easy, if you're not careful, to accidentally make the game non-deterministic, just by using a HashMap! We've already fixed a couple issues like this one, and it just remains to track down what code is causing the remaining de-syncs.

This is an area where we might implement new features to help the debugging process. Possibilities include automatically detecting network de-syncs by hashing the game state, and network session record/playback tools to help isolate the causes of network de-syncs.

## Modding

Modding is important to us. By being Open Source, anybody can change Jumpy as much as they could ever want, but we want to go further, hoping for a powerful modding ecosystem in the future.

### Map Editing

The easiest thing to mod in Jumpy right now is maps! Jumpy has a built-in map editor, with live editing capability, meaning you can play the game _while_ you edit the maps. All of the maps in Jumpy were made using the editor, and you can edit and save your own maps without leaving the game.

We've also just completed a new map randomization feature, included in the latest nightly release, thanks to contributions from [@AustinHellerRepo]. This takes an existing map, splits it automatically into chunks, and shifts the chunks around randomly to create a new map.

This is the first foray into all kinds of different possibilities for map generation and editing tools, and it laid the groundwork for implementing other map randomizer and generation algorithms. For instance, we could attempt to integrate AI map generation from text prompts using something similar to [mario-gpt](https://github.com/shyamsn97/mario-gpt).

### Asset Editing

In addition to maps, assets in Jumpy can be edited by modifying simple image and YAMl files. You can create characters skins, map tilesets, modify weapon sprites and create your own modifications. This works works by modifying parameters such as fuse time, explosion radius, etc.

We haven't implemented mod packages yet, so there's no way to install or enable specific mods. The only way to mod the game is to modify the files in the built-in asset directory. This is absolutely something that we will improve later, with the intent to have an online mod directory.

### Scripting

Right now item types are limited to the ones already in the game, and modifying asset files can only change so much. If you want to add totally new kinds of items, you have to contribute to the core game and write [Rust](https://www.rust-lang.org/) code. Eventually we plan to change this by implementing scripting.

Implementing scripting will mean that mods can not only include new assets, but also new game functionality. Scripting is difficult to get right, though, and there are many trade-offs to consider in the many different ways that you could implement it.

We have designed the [Bones ECS][becs] used in Jumpy with scripting in mind, but there are still technical challenges to overcome and practical experimentation to undertake, before we will have an actual scripting solution. One [idea](https://github.com/fishfolk/jumpy/discussions/489#discussioncomment-5115291) was to support scripts through WebAssembly for maximum performance, but there are issues that must be overcome with WASM linking before that can be feasible. There's also another [idea](https://github.com/fishfolk/jumpy/discussions/799) for implementing scripting for some languages that have pure Rust implementations. This may be a better solution for the shorter-term, and might be worth creating a proof-of-concept to test the idea.

## AI Players

Right now our AI players are not very smart. They can only swing swords, they get stuck in certain places on different maps, and on top of that, they aren't very fun to play with because they are relentless and nearly unbeatable! This is a temporary state of affairs and we are going to fix it.

Already in the current nightly we have improved the AI to be more fun, by slowing them down a bit and lower their reflexes. This doesn't make them smarter, but it does make them more fun. This _temporary_ fix only took a day to implement.

Our plan is to implement a proper AI that will be able to use different weapons and play more like a player, instead of just spawning and swinging a sword. This will take some experimentation and research to get working. We'll probably investigate more traditional game AI techniques as well as Reinforcement Learning and Active Inference.

## Documentation

We want Jumpy to be very well documented. We are creating an Open Source, production game title, and we want people to be able to see and understand how we did it!

Making a game is difficult and having a well-documented example could help other people get going with their own games.

To that end, I'm going to be working on filling out the [Jumpy architecture docs](https://fishfolk.github.io/jumpy/book/developers/rustdoc/jumpy/index.html) and the [Bones API docs](https://fishfolk.github.io/bones/rustdoc/bones_lib/index.html) which together will constitute an in-depth explanation of how the different pieces of Jumpy fit together. This will also help contributors learn about our codebase and how they can improve the game.

## Framework

We designed the [Bones] library to support Jumpy's needs, and to serve as a foundation for future games. Right now, there is a lot of code in Jumpy for handling things like UI, localization, networking, etc., that we would love to start moving into Bones, so that other games can share the the work that was invested in Jumpy. This will also help simplify the Jumpy codebase.

We are still working out exactly how to organize Jumpy and integrate different pieces of the game, but as we develop and find out which pieces are re-usable and how to combine them, we hope to create a Bones App framework that can take care of a lot of the repetitive game boilerplate, so that games can focus on what makes them unique.

I have rough ideas for how the Bones App framework might work, and [@AustinHellerRepo] is actually working on a small game other than Jumpy with Bones. This will help us get a better idea of how a Bones App framework can support multiple different games. It's something that we'll be feeling out as development continues.

## Summary

We've made tons of progress and have great potential for the future. There's lots to figure out and learn, but it's all problem solving, just like we've been doing since we started, and I'm excited to see what happens next!

[becs]: https://fishfolk.org/development/bones/introduction/#bones-ecs
[Bones]: https://fishfolk.org/development/bones/introduction/
[@AustinHellerRepo]: https://github.com/AustinHellerRepo
