+++
title = "Migrating from avra to avr-as"
date = 2019-02-04T15:02:26+01:00
draft = false
tags = []
categories = ["programming", "AVR"]
+++

{{< figure src="/images/migrating-avra-title.png" caption="assemble avra projects with avr-as" >}}

The main advantage of avr-as over avra is the possibility to generate
linkable .o object files that can be mixed with C files.

Unfortunatly, the syntax differs slightly between these two assemblers. And
it turns out that the syntax differences are big enough to make the
conversion of existing source code a non-trivial task that tends to take way
longer than expected.

To save others some troubles in similar situations, here are my findings
about [porting the SMC3](https://github.com/tenbaht/servo-motor-controller)
project from avra to avr-as/gas.

<!--more-->

Most of the difficulties boil down to the point that avr-as has no direct
replacement for avra's `.def` statement. `#define` looks quite similar, but
it doesn't behave in the same way. The seemingly subtle differences can lead
to very unexpected effects that are hard to debug.



## Macro parameter syntax

avr-as requires to specify the number of parmeters and refers to them by
`\name` while avra simply refers by numbers `@0`.


avra:

```text
	.macro  sti
	        ldi     r16,@1
	        st      @0,r16
	.endm
```

avr-as:

```text
	.macro  sti     adr, val
	        ldi     r16, \val
	        st      \adr, r16
	.endm
```

#### Conclusion
Easy.


## Macros and preprocessor constants

With avr-as it is very important to keep in mind that the preprocessing step
is really **pre**processing. There is a fundamental difference between a
`.macro` definition and using a '#define`. Referencing each other can be
tricky.

avr-as has no access to assembler values at the time the preprocessor runs
and `.macro` definitions can only access constant preprocessor definitions,
but they can't construct preprocessor references.

For avra both, `.def` and `.macro` are part of the assembling step and they
can refer to each other freely. This allows for things like this:

```text
	; for avra

	.def    AL      = r16
	.def    AH      = r17

	.macro  addiw
	        subi    @0L,low(-(@1))
	        sbci    @0H,high(-(@1))
	.endm

	start:
		addiw	A, 0x1234
```

The macro is expanded at assembly time into:

	start:
	        subi    AL,low(-(0x1234))
	        sbci    AH,high(-(0x1234))

And now the assembly-time `.def`'s are replaced:

	start:
	        subi    r16,low(-(0x1234))
	        sbci    r17,high(-(0x1234))

Doing the same with avr-as by using `#define` for the register names is not
possible, though. Just adding a suffix to a macro parameter is supported by
avr-as using the `\()` syntax, but that doesn't help in this case:

```text
	; for avr-as, does not work

	#define AL      r16
	#define AH      r17

	.macro  addiw   reg,val
	        subi    \reg\()L,lo8(-(\val))
	        sbci    \reg\()H,hi8(-(\val))
	.endm

	start:
		addiw	A, 0x1234
```

This would be expanded as:

	start:
	        subi    AL,lo8(-(0x1234))
	        sbci    AH,hi8(-(0x1234))

But AL and AH are unknown at this point. They are preprocessor
defines and the preprocessing step happened a long time ago. An
assembler declaration like `AL = r16` wouldn't help, as these are only
reference by value and `r16` is not a valid value but an identifier.


## Register handling

For this specific case there is a small loophole. avr-as allows simple
integers as register specifiers. So it is possible to do this:

```text
	; for avr-as

	#define A	16

	.macro  addiw   reg,val
	        subi    \reg,lo8(-(\val))
	        sbci    \reg+1,hi8(-(\val))
	.endm

	start:
		addiw	A, 0x1234
```

This would expand as:

```text
	start:
	        subi    16,lo8(-(0x1234))
	        sbci    17,hi8(-(0x1234))
```

This is less robust, though. If you use it with I/O registers it implicitly
assumes that the high byte it at the following address (and that is exists
at all). The avra way is more fool-prof. If there is no '*L' and '*H'
definition for that specific I/O register, it will trigger an assembler
error. avr-as can't detect that.



## Defining values

avra uses

	.def name	= replacement	; simple avra define and asm comment

and avr-as uses

	#define name	replacement	// avr-as define and C comment

Note the different comment tags. avr-as uses the C-preprocessor, so it only
detects (and removes) C-comments at this point. An assembler comment would
become part of the replacement string and would be inserted every time the
define is used, most probably breaking things.

See how bad it can get:

```text
; this is for avr-as, but it doesn't work as intended

#define	buffer	(RAMSTART+0x100)	; buffer area

	ld	r16, buffer+32		; use this
```

The C proprocessor has no idea about assembler syntax. All it does is to
replace some strings. This would come out:

```text
; this is for avr-as, but it doesn't work as intended

	ld	r16, (RAMSTART+0x100)	; buffer area+32		; use this
```

It will assemble just fine without any warning, but you will have a hell of
a time debugging that. Using an assembler variable instead of a `#define`
would haved saved your day in this case:[^1]

```text
; this is for avr-as and it will work fine

buffer	= RAMSTART+0x100		; buffer area

	ld	r16, buffer+32		; use this
```

#### Conclusion
Be **very** aware of the different stages of assembly when
working with avr-as!



## .ifdef and #ifdef

Again, preprocessor and assembler. They both access a different set of
defined values.

Use `.ifdef` for assembler defines, because the preprocessor has no idea
about them.

In most cases, use `#ifdef` for values defined by `#define` like the
register definitions from avr/io.h. 

In general, `.ifdef` is more flexible because it can access all assembler
defines and most of the preprocessor defines (except for concatenated label
names like in the macro example).

But since there is only `.ifdef` and no `.if defined(NAME)`, it is limited
to single tests only. The usual CPU type tests are easier to implement by
using `#if defined()||defined()...`.

Again, always be aware at which stage of assembly you would like the test to
happen.



## Pre-defined variables

avr-as allows for the full zoo of gcc predefines. The classic device type
test is as usual:

```text
#if defined(__AVR_ATtiny2313__) || defined(__AVR_ATtiny2313A__)
	; do something
#elif defined(__AVR_ATmega328__) || defined(__AVR_ATmega328P__)
	; do something else
#else
# error "no valid device chosen"
#endif
```

The syntax for avra is slightly different. Please notice the slightly
different device type name without the `AVR_` part in the middle:

```text
.if (__DEVICE__ == __ATtiny2313__) || (__DEVICE__ == __ATtiny2313A__)
	; do something
.elif (__DEVICE__ == __ATmega328P__)
	; do something else
.endif
```


## Program counter

avra refers to the current program counter address as `PC`, avr-as uses `.`.
But there is a more subtle and much more important difference:

avra counts words, but avr-as counts the bytes. But it is more complex than
just multiplying everything by two.

**relative jump with avra**: `PC` refers to the address of the current
command and counts the distance in words:

```text
	breq	PC+3
	nop
	inc	r0
	ret		; the branch lands here
```

**relative jump with avr-as**: `.` refers already to the address of the
following command and counts the distance in bytes:

```text
	breq	.+4
	nop
	inc	r0
	ret		; the branch lands here
```

### Conclusion
Stop counting. Use labels.


## Flash addresses

avra counts flash addresses in words, but avr-as counts the bytes.

**Refering to a string in the flash area with avra**: Data blocks are
automatically aligned to even addresses and padded as required.

```text
	ldi	r30, low(string*2)	; multiply by two for
	ldi	r31, high(string*2)	; the byte address

	string:	.db	"Hello",13,10,0
```

**Refering to a string in the flash area with avr-as**: avr-as does not
require the factor two for data addresses in the flash area. No need for
even addresses here.

```text
	ldi	r30, lo8(string)
	ldi	r31, high(string)

	string:	.db	"Hello",13,10,0
```




## Pseudo-Opcodes and operators

avra		|avr-as
------		|-------
low(val)	|lo8(val) (for RAM) or pm_lo8(val) (for flash)
high(val)	|hi8(val) (for RAM) or pm_hi8(val) (for flash)


This word/byte problem with flash addresses strikes again.


## Relocating negative pointer values

The AVRs offer a subi command, but no addi. So `subi reg, -val` is commonly
used instead of `addi reg, val`. If _val_ refers to an address, it needs to
be relocated by the linker. Unfortunately, _lo8()_ and _hi8()_ are very
picky about the syntax of their argument to choose the negative relocation
type.

This is _test.sx_:

```text
.global main

.section .bss
var:	.fill	1

.text
main:
	subi    r16, lo8(main)		; ok
	subi    r16, lo8(-main)		; ok
	subi    r16, hi8(var)		; ok
;	subi    r16, hi8(-var)		; linker error message
	subi    r16, hi8(-(var))	; ok
```

Compile it and check the result:

	avr-gcc -mmcu=atmega328 test.sx -o test.elf && avr-objdump -d test.elf

Uncommenting the second last line yields to surprising results:

	test.sx: Assembler messages:
	test.sx:13: Error: can't resolve `0' {.bss section} - `var' {.bss section}
	test.sx:13: Error: expression too complex

Negative relocations work within the .text segment, but when crossing
segment borders it needs an additional pair of brackets. Odd. _Very_ odd.



----
Footnotes:

[^1]: Yes, I am aware that hard-coding references to RAMSTART is a bad idea in any case. Just use a label with the `.fill` statement and let the assembler handle the details. But I need a simple example to illustrate the effect.
