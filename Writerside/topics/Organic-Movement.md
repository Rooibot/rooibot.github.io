# GMC Organic Movement

GMC's replication logic is in the Replication Component (a child of the Pawn Navigation Component), and much of the 
useful logic to handle movement generically is in the Movement Utility Component; if you are trying to design something
wildly different than the normal human movement systems -- a flight simulator, spaceship combat, a top-down 2D spaceship
bullet-hell game, whatever -- then the Movement Utility Component is what you'll likely be extending.

That, however, is something of an advanced use case.

For normal human movement -- the equivalent of Unreal's default Character Movement Component -- you'll almost
certainly be working with either the **Organic Movement Component** or its child, the **GoldSrc Movement Component**.
This document will focus on how the Organic Movement Component works; some of this is true of the Movement Utility
Component in general, but some is specific to the organic movement setup.

Reading the existing [GMC Organic Movement Guide](https://grimtec.net/gmcv2-doc-beginners-guide-organic-movement-overview/)
on the actual GMC website is recommended. In particular, the section labeled "Entry Points" is relevant to everything
that will be discussed here. Go do that now; we'll wait.

## Tick() tock...

One of the first things people notice about the Organic Movement Component is that there are three `Tick` functions:
`GenPredictionTick`, `GenSimulationTick`, and `GenAncillaryTick`. And one of the things they often immediately ask is 
"wait, what does each of these tick functions mean?" 

The answer is fairly simple:

* **GenPredictionTick** is the general purpose tick for predicted movement. Generally this will be on autonomous proxies 
  and authorities (e.g. the client which 'owns' a character, and the server), though it _may_ run on simulated proxies
  in some specific smoothing scenarios (where prediction is required). In single-player scenarios, this is the
  equivalent to the normal Tick and run each frame; in multiplayer, it will handle replays and sub-stepping as needed.
* **GenSimulationTick** is called _only_ on simulated proxies, and is used when what's replicated isn't sufficient to
  accurately represent the pawn's state.
* **GenAncillaryTick** is called _only_ on autonomous proxies or the authority, and always after GenPredictionTick.
  However, unlike GenPredictionTick, it will not be called during replay or substepping. This is generally used for
  predicted non-movement logic.

Which Tick should you put something in? Ask yourself a few questions:

1. **Is this a single-player game (and one where multiplayer will probably never be a consideration)?** Then just use 
   **GenPredictionTick**.
2. **Is this logic which will need to be run during rollback/replay?** Then **GenPredictionTick** is probably the 
   right place.
3. **Is this logic which needs to run on a simulated proxy?** Things that might touch off specific animations or such 
   on a simulated proxy should probably be in **GenSimulationTick** as well as **GenPredictionTick**.
4. **Are none of the above true?** If it's not movement, doesn't need to be handled during replay, and doesn't need to 
   be on a simulated proxy, then **GenAncillaryTick** is probably your... **tick**et. (I'm not sorry.)

