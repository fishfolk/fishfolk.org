---
title: Jumpy's Migration to the Bones Framework
description: >
  We recently migrated Jumpy to our new Bones Framework.
  Here's all about what we did and what it means for Jumpy.
template: blog/page.html
date: 2023-11-25

taxonomies:
  authors:
    - Zicklag
---

Recently we migrated Jumpy from Bevy to our new [Bones Framework][bones].
Here we're going to go over what that means, and how it helps us.

## What is the Bones Framework?

Bones Framework is our new **game engine**, built on the Bones ECS. Before now, Bones was mostly just an ECS,
and some other related components, not it's own game engine. It was used to give Jumpy a deterministic game
core that supported snapshot and restore. The latest refactor, though, change **a lot**.

### Asset System

The thing that started this journey was the asset system. Before the refactor, Bones had a very clunky
integration with Bevy's asset system that did _just enough_ to make it usable in Jumpy. It was not optimal
and it was only meant to be temporary, until bones got it's own asset system.

As I started working on the asset system, I was keeping in mind some important things that we wanted to
support:

- **Asset Packs:** We wanted to have asset packs be a first-class feature, so that it would be trivial to
  make games that allows users to add their own content.
- **Network Synchronization:** If you have mods, and you have network games, then we want a way to for you
  to synchronize your mods over the network, without having to do a super-annoying manual transfer.
- **Convenient Metadata Assets:** Jumpy was already making heavy use of what we called metadata assets.
  These are inter-connected YAML files that make it easy to organize all of our game data, such as items,
  maps, player skins, etc. This pattern worked really well for making the game, but wasn't super convenient
  to accomplish in Bevy, so we wanted to make sure that this worked super well out-of-the box in Bones.
- **Modding:** We wanted to be able to have mods create their _own_ kind of metadata assets, so that if a
  mod added a "clam" element, it could load the information for it from `.clam.yaml` files.

The trickiest part was related to metadata assets and modding. We needed a way for YAML files to be loaded
into Rust structs, kind of like the Rust [`serde`] library does, but we also needed to be able to _describe_
custom asset types that might not come from Rust.

At some point, I finally realized that this _describability_ was actually the _same_ thing that we needed
for first-class scripting support in the Bones ECS.

### Bones Schema

This kicked off the creation of [`bones_schema`], which, for those who know about it, is similar in concept
to [`bevy_reflect`]. The idea is that we needed a way to describe data with schemas, that could be used in
multiple crucial places:

- **The ECS:** The schema gives us memory layout information needed by the ECS to allocate memory for the
  data.
- **The Asset System:** The schema tells us what fields and types the data has, so that we can load the data
  from YAML or JSON files.
- **The Scripting System:** The schema tells us about the fields and types the data has so that scripts can
  read and write the data.

The new schema system opened the door _all_ of these systems to cleanly interact with each-other, and it
helped simplify some of the existing code in the Bones ECS, too.

### A Unified Framework

While the schema system is fairly simple in concept and implementation, there was still just a _lot_ to do.
Even if something is simple, a lot of simple stuff is still a lot of stuff. While migrating all of Bones
to using the new schema system, _and_ creating a new asset system, it seemed to make sense that we might
as well start checking off the next big ticket item that we needed to get done: creating a **unified bones
framework**.

As it currently stood, Jumpy was made out of a strange combination of Bones and Bevy. There was `jumpy_core`,
which was built on Bones and it's clunky Bevy asset integration, and then there was `jumpy` proper, which was
a Bevy game that used `jumpy_core` for the gameplay section of the game.

Inside `jumpy` you had to deal with the Bones ECS _and_ the Bevy ECS and dealing with that juggle was not
super pretty. It was confusing, and it made it probably twice as confusing for contributors who, depending on
what part of the game they want to contribute to, have to learn two different game engines that are eerily
similar, but still very different.

On top of all of this, there were tons of things that we were doing in Jumpy that we would rather be done in
bones, so that other games could take advantage of it, and so that Jumpy could stay focused on what makes
Jumpy unique.

This called for the Bones Framework, so that we could make Jumpy a Bones game, not a "Bones + lots of glue +
Bevy" game.

So, while we migrated to the new schema design, and made a new asset system, we _also_ started moving parts
of Jumpy out of Jumpy and into Bones. We migrated things like the localization system and, importantly, our
egui components. Migrating egui to Bones was a big win because, previously, there was no way to do UI from
inside of the Bones world. All of the UI was trapped outside in the Bevy part of Jumpy. Now though, Bones
has access to the full power of egui.

### Becoming Renderer Agnostic

As we shaped the Bones Framework we also made another useful accomplishment by making Bones **renderer
agnostic**. After we had our own asset system, Bones was no longer tied to Bevy. Today we still use Bevy
for rendering, but there's nothing that forces us to. Bones Framework games can be moved to any renderer
that can render sprites, tilemaps, egui, and debug lines.

This is a great freedom for us to have, not that there is anything wrong with Bevy, but we can also see
that Bevy is putting a great deal of effort into 3D rendering and other things that we don't necessarily
need. Being agnostic means that we take our project into our own hands a bit more, and allow us to focus
on what is important to us when necessary.

Also, by focusing rendering on _simple_ primtives, such as sprites, tilemaps, lines, and UI, we actually
keep the game more scriptable and moddable, because we are not depending on the ability to do arbitrarily
complex rendering directly from the ECS.

### Maintaining Rendering Power and Flexibility

Limiting our rendering primitives, though, does mean that the Bones Framework might not do all the
different rendering things that some games might want or need. That's totally fine, though, because we
allow you to make custom rendering integrations.

For example, say you are making a game that uses our official `bones_bevy_renderer`. This is the easiest
way to get started with Bones, and it uses Bevy to render your game, without you having to think about it
or do anything other than write your game for Bones.

But then you see that there's a really cool [Bevy Magic Light 2D][bml2d] plugin that can make 2D
lighting effects, and you want to use that in your bones game. Bones allows you to create your own
integration between it and the Bevy Magic Light plugin, so that you can control the lighting plugin using
Bones ECS components.

Once you create your integration you get the ability to get cool lighting effects in your Bones game, _and_
it will be compatible with Bones's asset pack and scripting system!

Or, in another scenario, immagine you wanted to make a "falling sand", such as [Sandspiel]. This is
actually a use-case [@Zac8668] is exploring with their [Astratomic] project! For a game like this, rendering
is pretty much 100% custom. You're trying to put all your little sand particles on the screen in a way
that is most efficient for that kind of game. Bevy isn't going to help you much at all with this
kind of rendering, so in this case, a custom rendering backend for bones would be appropriate. This lets
you get rendering efficiency while still hooking into Bones's modding features!

While Bones emphasises simplicity over the powerhouse rendering features that Bevy offers, it will still let
you stretch your rendering powers where necessary, with a little extra work, and we'll also be trying to
come up with more simple interfaces for other common rendering tasks that aren't supported out-of-the-box
in Bones yet.

### Multi-World Paradigm

As we migrated Bones to be a fully fledged framework, and not just a core piece of Jumpy, it became clear
that we were going to need a way to organize the "outside" part of Jumpy that was currently done in Bevy.
This part of the game was responsible for reading raw game inputs and handling the networking pieces
for online play. It "orchestrated" the core game, which was isolated and deterministic, and had no
knowledge of the networking.

This clear division between the core match, and the larger game and networking was something that
still made sense, but now we needed a way for Bones to be both on the "inside" and the "outside" of this
division. The solution was to make **multi-world** a first-class feature.

When using almost any ECS there is a concept of the "World" that stores all your data. All your game systems
interact with the data in this world. With our new multi-world setup, you are able to keep a useful isolation
between different parts of your game by putting them in different worlds. In Bones, we called these worlds,
along with their related systems, "Sessions".

This new pattern of game sessions helped unlock a few really useful patterns.

#### Game Start and Reset

It's a common task in a game to show a main menu of sorts, and have a button that
starts the game when clicked. This usually triggers a completely different set of systems and logic than
what is present on the main menu. And when you need to exit back to the main menu, you need to delete all
of the stuff that was used for the gameplay and re-setup the menu. This is actually a pretty annoying and
tricky thing to do when all of your data is in one world, because you can't just delete it all without
deleting the menu, too.

With sessions, though, this becomes trivial. You simply make one session for the menu, and another session
for the game. When the game starts, you hide the menu session, and create a new game session. When the game
is over, you delete the game session, and show the menu session again. Simple!

#### Session Runners

Along with sessions, we have a concept of "session runners". These runners are responsible for running the
session game loop. Having custom runners allows us to create runners for things like fixed updates, to make
the game run with a fixed simulation rate. We can also use runners to encapsulate networking or interpolation
logic, so that you could have one session runner for rollback networking, or maybe another session runner
for client-server networking, or other use-cases.

#### Game States

Sessions also turned out useful where we used to use Bevy states to enable and disable different systems.

For instance, for the pause menu, we used to use game states to control whether or not the game systems
should run and the game should play, or the pause menu systems should play. In this case, we can just
use sessions. Once session for the game and one for the pause menu, and the pause menu can stop and start
the game session based on interaction with the UI it renders.

Since we added sessions, we haven't found a need for game states again, and the pattern feels nice so far.

## Finishing the First Pass Jumpy Migration

Finally we had migrated the core Jumpy gameplay to the Bones framework. It was a _ton_ of work, but I'm
convinced that it was the right direction, and we've gained a lot with the migration. That said, we
haven't finished migrating the whole game. Notably the map editor and network play hasn't been ported
to the latest version of Jumpy yet, but we'll get there!

First, there were some developments in the wider Rust community that we wanted to take advantage of. 😉

## Implementing Scripting

Recently, deveopment was picked back up on the [Piccolo] project. Piccolo is a [Lua] implementation
written in Rust, and that was something we were interested in.

In order to get seamless web support, it was important that whatever scripting language we integrated
with Bones be implemented in pure Rust. While it's absolutely possible to us langauges not written
in Rust on native platforms, using those languages the web becomes much more difficult. Lua is also
a very common and popular language for game modding, so being able to support Lua was a big deal for us.

While it's still under development, we are happy to be some of the first users of Piccolo and [@kyren],
the author, has been very helpful to us as we've gotten started with it.

Since all of the recent updates to Bones, the Bones ECS was, as far as I could tell, had all of the
features it needed to be able to be accessed from a scripting language, and now it was time to test it
out!

In reality, there were lots of fixes and updates I needed to make to get bones _actually_ ready, but the
foundation that we had set was holding. Step by step we resolved all of the blockers until finally
we were able to actually implement a Jumpy item with a Lua plugin! See my [blog post][luainjumpy]
from yesterday for more details about that.

And that catches us up to the present day.

## Retrospect

_Phew_, now feels like a good time to catch a breather. Despite taking longer than immagined, so
far things have developed roughly according to plan. Since it's conception, Bones has been planned
to be fundamentally moddable and simple, and it seems that the vision is coming to fruiton.

I'm going to be glad to get out of the massive refactor and be able to focus on smaller tasks again
that can easily considered "done" and also show some more user-visible improvements. Working on
huge refactors without a lot of visible value-add in the short term can be difficult, but I do
think it was all worth it. I think our attempt at being pragmatic and pursuing our vision has payed off
so far, and will continue to do so.

While we're setting out to make a game, the Bones engine has grown from the, actual needs of that game,
and it is a means to fulfilling the vision not just of Jumpy, but of the larger ecosystem of Open Source,
moddable games that we want to create.

I'm personally not one of those developers that just wants to write everything themselves. In fact
I have a history of years of trying to avoid writing a game engine! Bones was not really a part
of my initial plan. But, surprisingly enough, we have needs that only Bones has been able to fill
for us.

I think that having complete control of our engine is going to be, and already has been, a benefit
for us. We have been able to shape our developer and modder experience into exactly what we want
it to be, and we are able to save ourselves development work by tailoring our engine to our needs.

It's been a rather enlightening project to work on. This is the first time I've ever written a game
_or_ an engine. I've learned an enormous amount and I know there is still so much more to learn and
I'm excited about that.

## The Future

So, what's next?

The next feature that I want to get done is asset pack loading. We just added scripting, which
is a huge moddability boost, and we already have a tutorial for making Jumpy
skins, but we don't have a way for users to conveniently install custom content! As mentioned above,
this is something that our new asset system was designed for, but we haven't "plugged it in" to
Jumpy yet. This shouldn't be too much work, and the value-add will be great, so I want to get
this done as soon as possible.

Documentation is the next big TODO, on my List. Jumpy has changed a lot, and Bones has 
probably doubled in size at least. Good documentation will help other people contribute and make
Bones more ready for other people to use in their own games.

Next I want to try to iron out some wrinkles in Jumpy's item system. We don't have events in Bones
yet, which has led to some experimentation with different design patterns that haven't been as clean
or as scripting friendly as I'd like. I'm investigating a simple reactivity system that might help
with this, and might be useful for more UI-like programming patterns.

Finally, we've got to finish porting some of the things that Jumpy lost in the refactor, like
debugging tools, networking, and the map editor.

That should get us in a good position to start working on more Jumpy game features.

[luainjumpy]: @/blog/introducing-lua-scripting-in-jumpy/index.md
[@kyren]: https://github.com/kyren
[Lua]: https://www.lua.org/
[piccolo]: https://github.com/kyren/piccolo/
[bones]: https://github.com/fishfolk/bones/
[`bones_schema`]: https://fishfolk.github.io/bones/rustdoc/bones_schema/index.html
[`bevy_reflect`]: https://docs.rs/bevy_reflect/latest/bevy_reflect/
[`serde`]: https://serde.rs
[bml2d]: https://github.com/zaycev/bevy-magic-light-2d
[sandspiel]: https://github.com/MaxBittker/sandspiel
[astratomic]: https://github.com/Zac8668/astratomic
[@Zac8668]: https://github.com/Zac8668

