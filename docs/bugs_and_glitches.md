# Bugs and Glitches

These are known bugs and glitches in the original Pokémon Crystal game: code that clearly does not work as intended, or that only works in limited circumstances but has the possibility to fail or crash.

Fixes are written in the `diff` format. If you've used Git before, this should look familiar:

```diff
 this is some code
-delete red - lines
+add green + lines
```

Some fixes are mentioned as breaking compatibility with link battles. This can be avoided by writing more complicated fixes that only apply if the value at `[wLinkMode]` is not `LINK_COLOSSEUM`. That's how Crystal itself fixed two bugs in Gold and Silver regarding the moves [Reflect and Light Screen](#reflect-and-light-screen-can-make-special-defense-wrap-around-above-1024) and [Present](#present-damage-is-incorrect-in-link-battles).


## Contents

nothing yet

check out flaws, forever alone