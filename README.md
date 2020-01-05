# Godplay

> WIP; at this point I'm noting down the plan, with time I'll start implementing

Virtual environment in which fake life tries to thrive.

This is a first iteration of the model, so it describes a world as a plane (its 
as easy to walk on land and water and there are no hills, so you move everywhere 
as easy), where some herbivorous creatures live. No specimen ever kills another,
but they are pretty dumb and their planning facilities are almost algorithmic (
they are not truly algorithmic, since there is a random factor involved, to 
simulate irrationality, panic and general absurdity on life at large. Besides,
genome model is pretty trivial and birth happens instantly after copulation. 
These (and some more) are simplifications, but they should be easily extendable
and replacable.

## Overview

There is a map that has sectors. These sectors have characteristics (some are
full of water, some spawn food, some are toxic, warm, cold, safe, etc). In these
sectors additional objects (like food) may be also spawned.

On that map there is also an initial population of randomized (well, almost, 
there should be a pre-run that makes initial population survivable) `specimen`.

Each specimen has a `genotype`. That genotype transcribes to `fenotype`, which is
description of that single specimen. It contains its sex, aging process (health, 
eyesight, speed, etc over time but also expected time of life if dying of old 
age), daily needs, resiliency to unsatisfied needs, and decision-making process 
(when replanning occurs, which urges are more important, etc).

Specimen also has state, that contains current position, health, mental state 
and physical condition (which impacts things like speed or inter-specimen conflict 
resolution), level of satisfaction of each need (both on daily and long-term basis).

Atomic time intervals are gonna conventionally be called `days`. It doesn't 
really work, since in this model specimen doesn't really have to sleep or eat 
daily, but it's quite easy to use when modelling.

Each specimen has a set of `need`s (e.g. need for food, sleep, etc). Its fenotype 
describes daily levels of these needs. If a need won't be satisfied during a day,
it accumulates for long-term and long-term needs can be satisfied by providing
more than daily need level. Long-term needs are usually "debt", but "surplus"
can occur as well (though surplus will gradually decrease with time, even if daily
levels are satisfied).

Needs are satisfied by performing `action`s (eating, sleeping, etc). Actions 
have requirements (specimen must be in watery area to drink, next to food to eat,
 etc) and satisfy number of needs at some level (e.g. sleeping in 'normal' place 
 satisfies need for sleep, but sleeping in safe lace satisfies need for sleep 
 better and also satisfies need for safety).
 
 Unsatisfied needs cause `urges` (not neceserarily any unsatisfied need, more 
 probably these that have debt above some threshold which was described as 
 'resiliency to unsatisfied needs' above) to take some actions that will satisfy
 them. Urges have levels based on short- and long-term needs, but other factors
 may come into play as well (e.g. attractiveness of some other specimen may boost
 level of sexual urge).
 
 Specimen always have a `plan`. Plan is supposed to satisfy an urge and consists
 of series of steps (usually several "move" and "perform action" at the end).
 From these steps we can derive `cost` of a plan (how much effort, that loosely
 translates to time required, is this gonna take?). 
 
 On most days specimen will perform next step of current plan. In some cases
 (I've finished previous plan yesterday, food I wanted was eaten by someone else,
 my wanted partner died or ran, etc, but also some urges suddenly overwhelmed 
 other urges, together with the one we're trying to satisfy by executing current 
 plan) it may start the day with replanning, which just discards current plan and 
 chooses a new one.
 
 Planning (choosing an action to take and a series of steps to satisfy that 
 action requirements) is based on several factors: current urge levels (how 
 much do I require that?), plan costs (can I have similiar or not much worse 
 effects with less effort?), action results (if I can eat more or less for similiar 
 effort, I should eat more), personal preferences (these map to resiliency to unsatisfied
 needs; some specimen may have higher need for water than for food, so when both 
 these needs are similarly satisfied, it will probably choose plan that brings water)
 and some random factor for good luck and some fun.
 
 ## Details
 
 > For the sake of initial values, we assume that average daily need of average
 > specimen over the whole simulation is 10. All the numbers will be tweaked by
 > trial and error once implemented.
 
 ### World
 
 World is represented as a cyclic n/m grid ((x, 0) is just above (x, m), 
 and (0, y) is just to the right from (n, y)) of cells called `region`. Each 
 region has any (0-n) number of modifiers:
 
 - watery
 - food-spawning
 - toxic
 - dangerous
 - safe
 - warm
 - cold
 
 There are some combinations that cannot go together:
 
 - safe region cannot be toxic or dangerous
 - warm region cannot be cold
 
 > and obviously vice-versa; though a region can be simultaneously toxic and dangerous
 
 #### Evolution
 
 At this point the only thing that changes in the world is that food is spawned.
 
 > Future extensions: carnivores, other non-violent species, maybe seasons of 
 > the year that tweak regions modifiers or their probabilities.
 
 New food appears in food-spawning regions with some probability. This probability
 is boosted if a region is additionally watery.
 
 > Initial proposed values:
 >
 > food spawn probability = 1%
 >
 > watery region boost = 0.5%
 
 Food is described with (gain = how much hunger is satisfies):
 
- initial gain (random from 30-50)
- final gain (35-65% of initial gain)
- lifespan (random from 60-100 days)

If a specimen consumes food at the same day as that food was spawned, he satisfies his 
need by initial gain. Every day through the lifespan (measured in days) gain
of that food object decreases linearily to final gain (at (spawn time + lifespan) 
moment, objects gain = final gain). Next day the object disappears.

> Future extenion: uneaten food fertilizes ground, increasing its spawn probability.

### Specimen

#### Genetics

> Unless explicitly stated, things like `daily`, `value`, `weight`, etc are `float`s.

##### Genotype

    enum Needs = ...  # see section below
    enum Feature = several (lets starts with 10) values without clear semantics (they represent the magic that we're trying to figure out in DNA)
    enum Sex = Male, Female

    class Genome:
        class Chromosome:
            daily[Need] needs
            value[Feature] features

        # size at least 3 - one male, one female, one general; 
        weight[Chromosome] genes
        # never the same
        Sex sex

        def weightedChromosome() -> Chromosome: # this is more like RNA, final level before transcription to fenotype
            first chromosome is related to being on one sex, last of the other
            when an entity is male, male-related chromose weight is doubled and female-related weight is cut in half
            reverse happens for females

            return chromosome obtained by weighing chromosomes from genes by their weights

##### Fenotype

    class StateOverLife<Unit>: # utility object
        Unit base
        double ageParam

        def value(age: (0, 1]) -> Unit:
            something that simulates aging, with a base value that is multiplied by some function controlled by age param
            (e.g. when reaching half an age, that property may be almost twice the base, while infants and elders are
            down to 75% of base)

    class Fenotype:
        Sex sex
        int diesAt
        //this impacts thresholds of satisfactionLevels
        [0.9, 1.1][Need] character
        StateOverLife<Health> healthState # Health is int
        StateOverLife<radius> eyesight
        StateOverLife<radius> speed
        StateOverLife<daily>[Need] urges
        StateOverLife<(0, 0.5]> mentalStrength # controls happiness (psychical condition) threshold
        StateOverLife<(0, 0.5]> physicalResiliency # controls physical condition threshold
        StateOverLife<low-int range> replanningPeriod

> We're using term `happiness` for mental state and `(physical) condition` to avoid confusion between `psychical` and `physical`

##### Transcription

##### Mutation

> daily mutation
> breeding mutation

##### Crossover

> details on when it happens - later, at reproduction needs

#### State

> fenotype, age, position, plan, daily and long-term needs saitsfation, timestamp, derivable urges, actions to satisfy needs and to proceed through they day (satisfy() will probably be passed as a callback to action)

#### Needs


