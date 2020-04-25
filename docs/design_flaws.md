# Design Flaws

These are parts of the code that do not work *incorrectly*, like [bugs and glitches](https://github.com/pret/pokecrystal/blob/master/docs/bugs_and_glitches.md), but that clearly exist just to work around a problem. In other words, with a slightly different design, the code would not need to exist at all. Design flaws may be exceptions to a usual rule, such as "tables of pointers in different banks use `dba`" ([one exception](#pic-banks-are-offset-by-pics_fix), [and another](#pokédex-entry-banks-are-derived-from-their-species-ids)) or "graphics used as a unit are stored and loaded contiguously" ([a notable exception](#footprints-are-split-into-top-and-bottom-halves)).


## Contents

- [Identical sine wave code and data is repeated five times](#identical-sine-wave-code-and-data-is-repeated-five-times)


## Identical sine wave code and data is repeated five times

`_Sine` in [engine/math/sine.asm](https://github.com/pret/pokecrystal/blob/master/engine/math/sine.asm):

```asm
_Sine::
; a = d * sin(e * pi/32)
	ld a, e
	calc_sine_wave
```

`Sprites_Cosine` and `Sprites_Sine` in [engine/gfx/sprites.asm](https://github.com/pret/pokecrystal/blob/master/engine/gfx/sprites.asm):

```asm
Sprites_Cosine:
; a = d * cos(a * pi/32)
	add %010000 ; cos(x) = sin(x + pi/2)
	; fallthrough
Sprites_Sine:
; a = d * sin(a * pi/32)
	calc_sine_wave
```

`BattleAnim_Cosine` and `BattleAnim_Sine` in [engine/battle_anims/functions.asm](https://github.com/pret/pokecrystal/blob/master/engine/battle_anims/functions.asm):

```asm
BattleAnim_Cosine:
; a = d * cos(a * pi/32)
	add %010000 ; cos(x) = sin(x + pi/2)
	; fallthrough
BattleAnim_Sine:
; a = d * sin(a * pi/32)
	calc_sine_wave BattleAnimSineWave

...

BattleAnimSineWave:
	sine_table 32
```

`StartTrainerBattle_DrawSineWave` in [engine/battle/battle_transition.asm](https://github.com/pret/pokecrystal/blob/master/engine/battle/battle_transition.asm):

```asm
StartTrainerBattle_DrawSineWave:
	calc_sine_wave
```

And `CelebiEvent_Cosine` in [engine/events/celebi.asm](https://github.com/pret/pokecrystal/blob/master/engine/events/celebi.asm):

```asm
CelebiEvent_Cosine:
; a = d * cos(a * pi/32)
	add %010000 ; cos(x) = sin(x + pi/2)
	calc_sine_wave
```

They all rely on `calc_sine_wave` in [macros/code.asm](https://github.com/pret/pokecrystal/blob/master/macros/code.asm):

```asm
calc_sine_wave: MACRO
; input: a = a signed 6-bit value
; output: a = d * sin(a * pi/32)
	and %111111
	cp  %100000
	jr nc, .negative\@
	call .apply\@
	ld a, h
	ret
.negative\@
	and %011111
	call .apply\@
	ld a, h
	xor $ff
	inc a
	ret
.apply\@
	ld e, a
	ld a, d
	ld d, 0
if _NARG == 1
	ld hl, \1
else
	ld hl, .sinetable\@
endc
	add hl, de
	add hl, de
	ld e, [hl]
	inc hl
	ld d, [hl]
	ld hl, 0
.multiply\@ ; factor amplitude
	srl a
	jr nc, .even\@
	add hl, de
.even\@
	sla e
	rl d
	and a
	jr nz, .multiply\@
	ret
if _NARG == 0
.sinetable\@
	sine_table 32
endc
ENDM
```

And on `sine_table` in [macros/data.asm](https://github.com/pret/pokecrystal/blob/master/macros/data.asm):

```asm
sine_table: MACRO
; \1 samples of sin(x) from x=0 to x<32768 (pi radians)
x = 0
rept \1
	dw (sin(x) + (sin(x) & $ff)) >> 8 ; round up
x = x + DIV(32768, \1) ; a circle has 65536 "degrees"
endr
ENDM
```

**Fix:** Edit [home/sine.asm](https://github.com/pret/pokecrystal/blob/master/home/sine.asm) to contain a single copy of the (co)sine code in bank 0, and call it from those five sites.
