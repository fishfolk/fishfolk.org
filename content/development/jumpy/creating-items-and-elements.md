---
title: Creating Items & Elements
template: docs/page.html
weight: 20
extra:
    toc: true
---

## Key Terms

- **Element:** Anything that can be placed on the map.
- **Item:** Something that can be picked up by the player and used.

Examples of items include grenades and swords. Examples of elements include seaweed and sproingers.
Each item also has an element that spawns it on the map.

As we talk about making an item or an element, the process is mostly the same, and each item's code
is also in the [elements][rust_elements] folder along with the other non-item elements.

## How Elements Are Made

There are a two major parts that make up an Element in Jumpy:

- **Rust code:** Each element has a corresponding Rust file, in the [elements
  module][rust_elements]. This has all the element's game logic.
  - **Element metadata:** Each new kind of element usually has a corresponding variant in the
    [`BuiltinElementKind`] enum.
- **Assets:** Each element has a corresponding asset folder in the [element asset
  folder][asset_elements].
  - **`[item].element.yaml`:** The key asset for any element is it's `.element.yaml` file.

[`BuiltinElementKind`]: https://github.com/fishfolk/jumpy/blob/a8399252747833f5f9d488dfcf523cdd1620a004/core/src/metadata/element.rs#L25
[rust_elements]: https://github.com/fishfolk/jumpy/tree/main/core/src/elements
[asset_elements]: https://github.com/fishfolk/jumpy/tree/main/assets/elements

### 🏗 Under construction

We will work on a more thorough guide for building items. If you want to help out, feel free to open
a PR! There's an edit button at the bottom of this page.

## Examples

Here are links to the code and assets for existing items. These can be useful references when
creating new items, and are roughly ordered by complication, with the simple ones presented first.

### Seaweed & Anemone Decorations

The seaweed and anemone decorations are the simplest possible elements. Reading through their code
can give you a good idea of how elements are put together in Jumpy.

Seaweed and anemones are examples of two elements that use the same Rust implementation for a
`decoration` element. The code is shared, but each has their own asset folder.

- [Rust Code](https://github.com/fishfolk/jumpy/blob/main/core/src/elements/decoration.rs)
- [`BuiltinElementKind`](https://github.com/fishfolk/jumpy/blob/main/core/src/metadata/element.rs#L58-L64)
- Asset folders:
  - [Seaweed](https://github.com/fishfolk/jumpy/tree/main/assets/elements/decoration/seaweed)
  - [Anemones](https://github.com/fishfolk/jumpy/tree/main/assets/elements/decoration/anemones)

### Sproingers

Sproingers are the next step up from decorations. They are simple, and just rest on the map, but
they can also effect players and other physics bodies such as items and critters.

- [Rust Code](https://github.com/fishfolk/jumpy/blob/main/core/src/elements/sproinger.rs)
- [`BuiltinElementKind`](https://github.com/fishfolk/jumpy/blob/main/core/src/metadata/element.rs#L80-L88)
- [Asset Folder](https://github.com/fishfolk/jumpy/tree/main/assets/elements/environment/sproinger)

### Grenade

Grenades demonstrate how item elements are implemented by having an element spawn grenade entities
with the `Item` component, that lets players pick up and use the grenades.

There's a lot of important basics demonstrated here, including how to get items to re-spawn if they
fall of the map, and other small things like that.

The grenade is a great base to use for things that can be thrown and armed/lit and it shows how to
make an explosion.

- [Rust Code](https://github.com/fishfolk/jumpy/blob/main/core/src/elements/grenade.rs)
- [`BuildinElementKind`](https://github.com/fishfolk/jumpy/blob/main/core/src/metadata/element.rs#L32)
- [Asset Folder](https://github.com/fishfolk/jumpy/tree/main/assets/elements/item/grenade)

### Other Examples

All the other elements are located in the folders as the elements above and can be useful reference
for different techniques or functionality. Check those out for more examples!
