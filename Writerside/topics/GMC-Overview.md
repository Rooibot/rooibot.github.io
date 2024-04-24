# Is GMC Right For Your Project?

Rooibot -- or, at least, the lead developer -- has been using GRIMTEC's Unreal 
[General Movement Component](https://www.unrealengine.com/marketplace/en-US/product/general-movement-component) for quite
some time, and in that time we've seen many people have misconceptions and misunderstandings about what the General 
Movement Component actually _is_, and what it can provide. This is unfortunate, because while the GMC is an incredibly
powerful tool, it is also an incredibly _expensive_ tool.

So this document is getting written up in hopes of having a single easily-linked-to document about the GMC which can
answer a few of those common questions. Foremost among them, "Is the General Movement Component right for me?"

## Unreal and Movement Components

To understand where the General Movement Component fits into things, it's worth first understanding how movement 
components in Unreal work at a higher level.

Unreal has a concept of an `Actor`, which is anything you can place into a world. This ranges from the more obvious
actors who have a physical presence -- things like rocks, or trees, or NPCs -- to actors who may not have a physical
location but nonetheless still have a presence in the world, such as weather systems and AI management systems.

A `Pawn` is a special subtype of an `Actor`, with two main traits:

1. Pawns are not static; they can move around the level at runtime.
2. Pawns can be 'possessed' by a Controller (be it a player controller, an AI controller, or whatever).

Point #1 is provided by a `PawnMovementComponent`, which will determine _how_ a `Pawn` moves around when a controller 
provides either input (such as when a player says "go forward") or a specific destination (such as when an AI controller 
says "I want to move to this spot in the world").

Unreal also provides a specialized type of `Pawn` called a `Character`; this has a special `PawnMovementComponent` 
called a `CharacterMovementComponent`, also known as "the CMC" for short. The CMC provides some basic movement logic
to handle human-like movement and replicate it across the network; it makes for a very convenient starting place for
new Unreal developers, and as such many things are built atop the CMC.

However, it does have a few limitations.

Chief among those is the fact that extending the CMC with new types of movement often becomes a bit of a struggle;
while the CMC provides a lot of convenient logic already implemented for you, it also has very specific assumptions
about how things will be done. This means if you want to add a simple "sprint" function to the CMC in a multiplayer
scenario, you either need to extend the CMC's own implementation of saved moves and networking, or do something like
utilize Unreal's Gameplay Ability System to add a replicated "sprint" ability.

This is where the General Movement Component comes in.

## What is the General Movement Component?

The General Movement Component, or "GMC", is an implementation of a `PawnMovementComponent`. Like the CMC, it has a 
special pawn that goes with it -- the `GMC_Pawn` -- and takes player input or AI movement goals. However, it works in
a very different fashion than the CMC.

The GMC has a very precise movement lifecycle, with the GMC's Organic Movement Component (the one most GMC licensees 
use) built around "Pre Movement", "Movement", and "Post Movement" logic. It has special tick functions for both 
prediction _and_ simulation in multiplayer scenarios. It provides functionality to bind arbitrary variables for 
replication, and then will preserve those variables automatically as part of its movement record, allowing _everything_
relating to movement to be rolled back and replayed if a client and server get out of sync. It has a number of 
convenient bits of network logic which help to adapt to (and better conceal) the quirks of network latency in
multiplayer scenarios.

It is, in short, an extremely powerful tool for anyone who wants to build a custom movement system -- _especially_ if
you want to build a custom movement system with really solid networking.

But in order to do all these things better than the Character Movement Component does, the GMC needs to do these things
very _differently_ than the character movement component does. This means that any code out there which expects to find
a `Character` or a `CharacterMovementComponent` and directly hook into its guts _will not work_ with the GMC, at least
not right out-of-box. And there are _plenty_ of things which assume you're using a `Character` and 
`CharacterMovementComponent` when you might not expect it... some of which have caught people by surprise. 

As an example, Unreal's own Motion Matching and Motion Warping systems both expect a character movement component. The 
Motion Matching system expects to have a Trajectory Component (which itself wants a character movement component), 
while the Motion Warping system expects to be able to modify montage root motion in specific ways that the CMC 
facilitates.

Another big one is Epic's Gameplay Ability System. GAS is a very powerful tool for making extensible gameplay and 
handling abilities and effects in a multiplayer game. However, since GAS uses its own replication, anything which
affects movement -- like a Sprint ability, or a Stunned effect -- is getting controlled outside of the movement
component. For single-player scenarios, that's fine, but as soon as you get into multiplayer this has dire implications
for the GMC's ability to rollback and replay movement; if you have Sprint controlled by GAS externally to the GMC, and 
then things get out of sync and the GMC needs to rollback and replay networked movement, moves may replay with a 
different maximum movement speed than previously (due to you having started or stopped sprinting).

Obviously, it's possible to implement those same things atop the GMC. It's the whole reason we wrote 
[GMC Extended](https://github.com/Rooibot/GMCExtended), after all; it provides trajectory prediction for Motion Matching, and supports Motion Warping 
atop the GMC's replicated montage playback. And it's the purpose behind the nascent [GMC Ability System](https://github.com/Reznok/GMCAbilitySystem), 
or GMAS: providing a GAS equivalent which integrates directly with GMC's replication for purposes of multiplayer. 

But if you don't expect to need to reimplement those things atop the GMC, that requirement can blindside you.

So, in answering what the GMC _is_, it's also worth answering what it _isn't_: it is _not_ a drop-in replacement for the
Character Movement Component, in terms of utilizing things which expect a Character Movement Component to be present.

## Why _Should_ I Use the GMC?

While the GMC isn't a solution to every single Unreal movement and networking problem out there, it _is_ nonetheless
a solution to a great many of them. Especially for any scenario where you need to add custom logic.

Look, for instance, at the average tutorial on how to add sprinting functionality to a `Character`. The first thing
you'll notice is that half the time, you are required to go into C++ just to add sprinting functionality! That is 
inconvenient at the very least, and can be problematic for anyone who is trying to experiment with a first game
purely in Blueprint.

To add sprint functionality to a `GMC_Pawn`, you need only bind a new "Client Authoritative" variable such as a 
`Wants to Sprint` boolean. When the player presses the sprint button, set that variable to true on the client. When the 
variable is true, increase the maximum movement speed. Doing that is sufficient to not only implement sprinting in the 
GMC, but it will even handle prediction and rollback of the movement for you in a networked scenario!

This means that if movement is key to your game -- if you have things like parkour movement, wall-running, teleporting,
grappling/tether mechanics, etc. -- implementing that movement can be a great deal less painful atop the GMC, 
_especially_ in a multiplayer scenario. 

Moreover, the GMC is designed _specifically_ to allow it to be extended via Blueprint. While Blueprint code is never 
going to be quite as efficient as C++ code, there _have_ been games written predominantly in Blueprint logic, and 
even for C++ programmers (like over here at Rooibot) Blueprints can be extremely useful for prototyping things quickly
before moving them into C++.

In addition, the Character Movement Component makes a lot of assumptions about how movement works. These are relatively
reasonable assumptions for purposes of bipedal human movement, but they may not hold true for things like a flight
simulator or a spaceship.

While the GMC provides an "Organic Movement Component" which serves the same bipedal human movement purpose as the CMC
does, it also provides a general-purpose "Movement Utility Component", which gives you all the benefits of the GMC's
networking and prediction functionality, while allowing you to implement something wildly different.

So, scenarios where the GMC might well be the correct choice for you include:

* You are making something which has movement wildly different than the normal bipedal human movement types. A flight 
  simulator or spaceflight sim, for instance.
* Your game makes heavy use of based movement -- in other words, movement where the character is standing on or holding 
  onto something which is also moving. (This is a place the default Character Movement Component often falls short.)
* You have a great many ways in which you need to extend movement -- to add parkour functionality from scratch, for
  instance, or in a superhero game where Spider-Man-style grappling or some sort of flight or superspeed play a
  key role.
* You want really solid networking, and you _don't_ already have a ton of things built atop the CMC that you're going
  to need to utilize.
* You need custom movement and are doing most of your game in Blueprints; the CMC can be extremely painful to extend 
  when working entirely via Blueprints.

If those sort of things _are_ true for you, then the GMC might well be worthwhile (and could save you a lot of time, 
stress, and headaches).

## Why _Shouldn't_ I Use the GMC?

There are scenarios when the GMC is not the right choice.

Many first-person shooters don't have a core focus on _movement_, and so they don't need much functionality beyond what
the CMC provides. If Unreal's default First Person or Third Person templates provides 90% of the movement logic you
need, the GMC is almost certainly overkill!

Similarly, if you have a complex, fully-built-out ecosystem of things atop the CMC -- motion matching animation, a
combat system that relies on the CMC for melee functionality, and so on -- then the GMC may end up being more work than
it is worth, when you need to re-implement all those things atop the GMC (or at the very least port the logic over).

No one tool for Unreal is the correct choice 100% of the time; the CMC certainly isn't, as there are many scenarios
where it falls short. But the GMC isn't a silver bullet for every scenario, either... not even the ones where the CMC
falls short.

For instance, a purely physics-driven game akin to Rocket League is going to be difficult to implement
atop the CMC, but it's _also_ going to be difficult to implement atop the GMC. Unreal is, by nature, a non-deterministic
game engine, and a physics-driven multiplayer game needs to be deterministic to at least _some_ degree. And while the
GMC makes it far easier to extend movement logic with custom movement types than the CMC does, _neither_ is going to
provide the deterministic physics implementation needed.

Similarly, Unreal is not the best choice for multiplayer fighting games. Unreal does not inherently want to replicate
character animation -- for a variety of really good reasons -- but that nondeterministic nature of the engine means
that even while you might bring the _character position_ in sync across multiple clients, you're not going to bring the
exact character _pose_ in sync. Which, in most games, is fine; as long as you see the other player making coherent
moves, it doesn't matter whether I see my left foot forward on my screen but you see my right foot forward on yours.
But for fighting games, having the pose entirely in sync can be of crucial importance for purposes of hit detection!
And for all that the GMC provides many useful tools to extend movement, that is _not_ a scenario which the GMC aims
to solve.

In short, if any of the following are true for you, the GMC may _not_ be the solution to your woes:

* You are trying to do something which is fundamentally deterministic (a fighting game, a physics-driven game).
  Using the GMC won't be any _more_ difficult than the CMC for these, but it won't solve your woes, either. (Honestly,
  in this scenario _Unreal_ might be the wrong choice overall.)
* You are utilizing complex systems already built specifically atop the Character Movement Component, such as some
  pre-built combat system or parkour movement system.
* You have a mature set of abilities built out atop the Gameplay Ability System already, and your game is multiplayer
  (and thus needs abilities replicated). Reznok wrote up a good article on [why this is a problem](https://reznok.com/how-not-to-use-gas-gmc).

### Conclusion

Unless you're doing something very generic, no one thing out there is going to give you all of what you need right
out of the gate, and to some extent it's going to be a question of "which thing will take me the _least_ time to rewrite
my own version of." 

If movement is a core part of your game design, be it custom movement or based movement or whatever else is beyond
the baseline capabilities of the Character Movement Component, the answer might well be that the GMC will save you 
enough time and pain that it's worth writing your own implementation of things that would otherwise sit atop the CMC.

(I know that for me, GMC has _absolutely_ been a major benefit; it allows me to focus on my actual movement logic 
rather than bending the CMC into a pretzel trying to get the foundations of what I need to _start_ building!)

But in the end, what foundation you build your movement atop is going to be something you need to decide game-by-game;
there's no one right answer.