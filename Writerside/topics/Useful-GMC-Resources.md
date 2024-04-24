# Useful GMC Resources

The following frameworks and documents can be very useful when building things atop the General Movement Component. 
Most (if not all) of these should be linked on the GMC licensee/support Discord as well, but they're presented here as 
they can be useful to reference if you're merely considering whether or not to take the plunge on GMCv2 and so are not 
yet _on_ the GMC licensee Discord.

### GMC Ability System (GMAS)
[https://github.com/Reznok/GMCAbilitySystem](https://github.com/Reznok/GMCAbilitySystem)

Reznok (along with several other licensees) has been implementing a GMC-focused equivalent to Epic's Gameplay Ability
System, called the GMC Ability System (or GMAS for short). This provides Abilities, Effects, and Attributes in a
similar fashion to GAS, but utilizes the GMC for replication and prediction.

This means that abilities which affect movement speed will be properly replicated and preserved in the movement history,
avoiding the pitfalls described in Reznok's [how not to use GAS + GMC](https://reznok.com/how-not-to-use-gas-gmc)
article.

### GMC Extended (GMCEx)
[https://github.com/Rooibot/GMCExtended](https://github.com/Rooibot/GMCExtended)

Rooibot's own GMCEx provides various extensions to GMC, such as trajectory prediction and motion warping, which
otherwise rely on behaviors specific to the Character Movement Component. Basically, if we have a generally-useful
reimplementation of something atop GMC which feasibly _can_ be put into GMCEx, it probably will be.

### GMAS + GMCEx Project Template
[https://github.com/Rooibot/GMASExTemplate](https://github.com/Rooibot/GMASExTemplate)

A simple template for integrating GMCv2, GMAS, and GMCEx all in one project is also available.

### GMC Shooting Component
[https://github.com/Candescence/GMCv2-Shooting-Component](https://github.com/Candescence/GMCv2-Shooting-Component)

Candescence has provided an example implementation of how to use the GMC's replication and prediction to implement
a shooting component, thus keeping weapon logic within the GMC replication ecosystem.

### GMC+ALS
[https://github.com/Dixon6/GMC-ALS](https://github.com/Dixon6/GMC-ALS)

Dixon has worked to port the `ALS-Replicated` implementation of the venerable Advanced Locomotion System (or ALS) to run
on top of GMC; this provides basic mantling support as well as ragdolling and the other usual ALS-powered goodies.

### GMC+Lyra
[https://github.com/Dr-Rank/GMC-Lyra](https://github.com/Dr-Rank/GMC-Lyra)

Dr.Rank has worked to port the animation system from Epic's "Lyra" example game and framework onto a GMC-powered base.