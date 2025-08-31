# PR Guidelines
This page contains guidelines that you should follow when making pull requests to the Atmospherics game area.

## Documentation
Atmospherics is a game area that has historically suffered from poor documentation.
The maintainers that have previously maintained on Atmospherics have long since left, leaving an information void in a fairly large section of the codebase.
There is now a large maintainer effort to re-learn Atmospherics and improve it.

In order for this to not be repeated, we ask that you extensively document your changes and what your code does, even private methods that you create.

If you perform any derivations for important calculations, you should bake the derivation into the code using comments.

## Units of Measurement
All Atmospherics units and calculations are done in metric (Liters, Kelvin, Joules, etc.).
Any measurements displayed to the user through UI/UX should be displayed in metric, with temperature measurements both displayed in Kelvin.

## Configuration and CVARs
Any major Atmospherics features that you add (like Monstermos, Active Groups, etc.) should be toggleable via a CVAR.
This allows forks that aren't majorly focused on Atmospherics gameplay to turn off features to save on tick time.
Additionally, some forks prefer to forgo certain features in the name of focusing on alternative gameplay allowed by that disabled feature.

Any magic numbers you do use to configure your feature (like constants for controlling limiting behavior) should be CVARs instead.

## Thermodynamics and Energy Conservation
Implementations should respect the [laws of thermodynamics](https://en.wikipedia.org/wiki/Laws_of_thermodynamics).

Respecting thermodynamics generally leads to cleaner and more robust solutions and avoids having to use magic numbers to tune or balance behavior.

It is ok to make concessions to ignore these laws in the name of more engaging and interesting gameplay (for example, we have CVARs to control the scaling of heat transfer in Atmospherics).

## Performance
Atmospherics is a highly performant section of the codebase, considering the various operations and math it has to do.
This is due to a variety of reasons:
- Atmospherics is a sleeping giant, with it only doing calculations when it needs to.
  - A still atmosphere is an atmosphere that doesn't need to be re-calculated.
  - A lot of data is cached and only updated when it needs to be updated (Airtightness/Tile Invalidation).
- When Atmospherics needs to perform calculations, the calculations done are usually highly abstracted in a way that still delivers the same effect, but without crunching a lot of numbers.
  - A small group of tiles in a larger tile mixture is processed instead of all tiles (Active Groups).
  - Large tile equalizations are processed in bulk instead of one at a time (Monstermos).
- Any array math calculations are usually done using SIMD-accelerated math methods in the NumericsHelpers class.

Generally, any large implementation should avoid degrading the current performance of Atmospherics.
You should run profiling or create a benchmark to test your feature.
This also helps yourself (and future contributors) optimize your feature, as you can get a reliable measurement of how much time your feature is using.

Atmospherics shouldn't be allocating large amounts of memory every processing run—avoid creating large lists or other objects, only to dispose of them at the end of the method.

Instead, Atmospherics allocates arrays up-front and reuses them across the lifetime of the system.

## Tests
Atmospherics is a system that has extremely low test coverage, in the single digits.
This is bad, as things can break silently and nobody will notice until further down the line.

For example, Airtightness was refactored to be cached and updated using an event system, instead of being recomputed each Atmospherics tick.
This silently broke firelocks for 1–2 years, with them being unable to stop spacing and triggering across the station when unnecessary.
It was fixed recently, however, it took around a week straight of debugging.

As such, any major Atmospherics feature should have tests written for it.