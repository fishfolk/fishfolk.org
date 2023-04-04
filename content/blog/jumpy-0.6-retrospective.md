---
title: Jumpy v0.6 Retrospective
description: "A retrospective on the current architecture of Jumpy: what I learned, what I like, and what I'd like to improve in the future."
template: blog/page.html
date: 2022-04-03

taxonomies:
  authors:
    - Zicklag

extra:
  lead: "A retrospective on the current architecture of Jumpy: what I learned, what I like, and what I'd like to improve in the future."
---

Yesterday we released [Jumpy v0.6.1][v0.6.1] not long after our [v0.6.0] release, which was a complete rewrite of Jumpy on top of [Bevy]. Now seems like a good time to think about how the effort went!

[v0.6.1]: https://github.com/fishfolk/jumpy/releases/tag/v0.6.1
[v0.6.0]: https://github.com/fishfolk/jumpy/releases/tag/v0.6.0
[bevy]: https://bevyengine.org

## Things I Learned

The list of things I learned working working on Jumpy is enormous, and I'll not cover everything here, but there are a few stand-out takeaways.

### You Can't Predict the Future — So You Have to Try Things!

A lesson that I've learned before, but definitely seemed highlighted in my work on Jumpy, is that you can't tell how something is going to work for sure, until you actually do it. This seems _especially_ true in the world of game development.

There are so many factors that go into whether or not an architecture, gameplay feature, etc. is going to work or not in a game. You can do all kinds of guessing and researching to figure out whether or not something is a good idea, and by all means you should do that, but at the end of the day you often have to try something to know whether or not it will work.

Sometimes, you not only have to try some specific detail out, but you have to try it out in combination with the rest of your game.

For instance, I spent a lot of time testing and contributing to [`bevy_mod_js_scripting`] in the hopes of enabling advanced modding for Punchy and Jumpy. This went rather well, until I had to do modding _and_ networking. After deciding on the rollback networking model, which requires running the game simulation up to 8 times per 16ms frame, the scripting solution just couldn't keep up with the performance required.

In this case, networking came first, and we'll pursue alternative modding solutions in the future.

Still, I don't regret any time I spent on `bevy_mod_js_scripting`. I learned a lot by working on it! And, even though I still can't predict the future, that first-hand experience will help me make more educated decisions about new ideas moving forward.

[`bevy_mod_js_scripting`]: https://github.com/jakobhellermann/bevy_mod_js_scripting

<!-- ### Organizing Takes Practice

Jumpy is the first Open Source project that I've taken any lead on that also has other people trying to contribute to it. I'm still getting used to having help!

I'm finding that organizing the project and figuring out how to best work on the project with other people at the same time is something that's going to take practice, and I'm glad I had the opportunity to do it.

I think I need to get better at getting help when it's available, and I'm still learning how to try and split work up so that different pieces can be done at the same time, instead of requiring one person to do a majority of the work before other people can help out.

I think sometimes that is inescapable, but I think there are probably opportunities to share work that I'm not fully taking advantage of all the time. -->

### Networking Can Be Difficult — So Maybe Think About it Early in Development!

Game networking was something that I'd never done before working on Jumpy, and there were a number of phases that I went through before finally settling on what we have today ( or more like tomorrow, we're almost finished with it! ).

Something that I hadn't fully anticipated was the impact that the need for networking could have on the design of the game itself. It's not an easy thing to integrate into all games, and if you know you're going to want it, you might want to figure out how you're going to make it work before you get too far into development.

For instance, we developed a custom, deterministic and snapshot-able ECS to use for the Jumpy core game loop, largely motivated by the need for network play.

Rollback networking is really nice for games if you have deterministic gameplay, and it frees you from having to think about networking code in your core game loop. But the determinism and snapshot capabilities were not easy to achieve with the Bevy ECS, so we made a small ECS tailored to that use-case. **Networking changed, in large part, the way we developed Jumpy core.**

So now I know that if I'm going to want networking in a game, I should be wary of that need early on, so that I don't have to rewrite my game for it later.

### Software Development is Hard — It's OK if it Isn't Perfect the First Time

Software development is difficult. I always want to make things as simple, elegant, and understandable as possible, but it often doesn't turn out as nice as I'd like it.

Still, because you can't predict the future, and you have to try things, that's actually OK! You can rarely make the perfectly elegant, simple, and understandable solution on the first try. You have try things before you find out the best way to do something, and if that means re-working, that's fine.

On of my core development principles is:

> You have to be bold enough to imagine the "perfect" solution and work towards it, but practical enough to realize that it's only helpful if you can actually _do_ it in the real world.

Sometimes you have to do something that isn't optimal because you just have to get it working, and that's OK. But also don't settle easily for something that you feel could be better. Always work at making it the best it can be.

Jumpy definitely isn't perfect, and there are things I'd try to do better if I did it again, but we can always try to improve it, and it _is_ working, which is important! Don't let perfect be the enemy of good.

## Things I Like

Now lets talk about some of the things I like about the current design for Jumpy.

### ECS Architecture For Gameplay

Jumpy was my first major use of the Entity Component System design model, and I feel like it's definitely got some advantages for designing gameplay.

Gameplay tends to involve a massive amount of interactions over the game world, and the ECS design pattern handles that pretty well. Way better, in my experience, than Object Oriented design, which becomes hard to add features to when plans change or as the game develops.

### Isolating the Core Game Loop

As touched on above, we use a custom ECS for the Jumpy core game loop, which is essentially everything that happens during match gameplay. This ECS is almost completely isolated from the outside Bevy world. This was motivated in large part by networking, but the design also has some elegant, non-networking advantages I like.

- There are a lot of things that go on outside of the core gameplay such as asset loading, UI, rendering, settings, etc. and it's nice to have a clean separation between all that "meta" stuff and the gameplay itself.
- This also makes it easier to do things like restarting matches and making sure you don't have leftover state from the previous match leaking into the world. You just clear it out and create a new one when you start a new match, without clearing out things like your menu state, etc. that are in the main Bevy world.

Also, subjectively, it just _feels_ right. If global mutable state is the number one cause of software bugs, isolating the gameplay global state from the rest of the global state should reduce bugs, and in my experience, I think it has helped.

### Built-in Editor

When thinking about how I might design my own games, I'd previously leaned toward the direction of using an external map editor like [LDtk] to avoid making my own. With Jumpy, though, the idea was to have a bult-in map editor, and after making it, I'm pretty convinced that having built-in editors is the way to go for me.

[ldtk]: https://ldtk.io/

When I started the new Jumpy editor, I wanted to do something that was a little different from before, and that was to let you edit the map _while playing it_. Honestly this was partially due to developer laziness. I didn't want to go through the effort of rending the map in the game, just to have to make another implementation of the map viewer in the editor.

So I thought I could just make the map editor essentially a layer on top of the normal game. Override the camera controller, add some overlays to indicated item selection, and voilà! Not only do I avoid making a separate "editor view" where I have to fake the item rendering, etc., now the user experience is _amazing_.

The big reason I'd make a built-in editor over using an external editor is because an external editor will never be able to support game-specific live rendering. It just feels so good to be able to place an item in the game, such as a fish school or seaweed, and see what it will actually look like with animated seaweed and moving fish.

Plus, it wasn't as hard as I thought it might be to make a nice editor. Granted, there is definitely some polishing to do, and it's not done yet, but it is still very usable and feels overall better than LDtk would, just because it's built-in with live feedback, so I'm very happy with it.

## Things I'd Like to Improve

I was recently thinking about one point in particular that I'd like to try and improve in future iterations of Jumpy.

### State Management Outside of the Core Gameplay

While I feel like the ECS model works really well for gameplay, I'm not a big fan of it so far for things like the menu and pre-match states.

The menu and pre-match states, especially when you involve network matchmaking, which is fundamentally asynchronous, don't seem well lent to the ECS design pattern. It's absolutely possible to make work, and I believe there are ways to improve the situation, that probably wouldn't take too much work, but it doesn't feel right to me.

I end up putting things like the current menu page, etc. into a bunch of global ECS resources. There's a tendency to let different parts of the menu logic, modify all these resources and the "global-ness" of it all makes it harder to reason about.

Again, you can make it work, but I feel like something more in the direction of an actor model would be better suited for the menu and pre-game states.

In an actor model, each different "actor", which could be things like the menu, the matchmaker, the player input mapper, etc, each own their own state, and each of the actors can communicate with the others by sending messages.

Something like this pattern was already necessary, using bi-directional Rust channels, in order to connect the synchronous UI code to the asynchronous networking matchmaking code.

I think having each actor with it's own private state would make the setup more predictable and easier to reason about and extend over time. While for gameplay it's really useful, we don't need the massive global-ness of the ECS world when designing these discrete portions of the pre-game interaction that could instead interact with each-other through messages.

I haven't done any experiments in this direction yet, but it's something I'd like to explore sometime possibly in the future.

## Summary

I feel like we've made amazing progress with Jumpy, and I've learned so much while working on it. Jumpy wouldn't be what it is today without _all_ its contributors, and it's been awesome to be a part of it. Thank you everybody!

There is so much that goes into game development, and like any software, Jumpy isn't perfect, and never will be, but the lessons learned while working on it will help make Jumpy, and the games and other projects of the future, even better!
