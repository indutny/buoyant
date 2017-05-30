# Buoyant

**WORK IN PROGRESS**
**PRs are welcome**

A Universe in VM, or VM in the Universe?

## Idea

Fully adjustable language-agnostic bytecode VM with:

* Non-conservative Garbage Collector with configurable types
* Customizable bytecode
* Inline caches (i.o.w., capable of rewriting bytecode on the fly)

## Goals

* Running on large selection of hardware from servers to memory-constrained
  embedded devices
* Execution performance
* Multi-Language interoperability

### Garbage Collector

GC is not conservative (i.e. moves data), the layout is the same for all objects
(each slot is pointer sized):

```
[ heap slot #1         ]
...
[ binary slot          ]
...
```

TBD

### Customizable Bytecode

All bytecode opcodes are described in the source module. Each opcode
is implemented in terms of internal bytecode. Internal bytecode is subset of
[wasm] instructions, and thus is very low-level. It interacts with VM by
reading/writing VM's registers.

Trivial Buoyant implementation can emulate internal bytecode. Advanced should
generate machine code for opcodes implemented in terms of internal bytecode.
This way a dispatch loop becomes similar to the one in [luajit][1].

### Types

TBD

### Inline Cache (IC)

Internal bytecode provides following instructions for facilitating use of ICs:

* `ic-fork reg_obj, imm_off`
* `ic-join`

NOTE: Only one use of `ic-fork` per opcode implementation is allowed.

Let's work out an example implementation of fictitious
`obj-prop obj, str` opcode:

```
load-arg r0, 0  # load `obj` into `r0`
load-arg r1, 1  # load `str` into `r1`

# Let's say `r[0]` holds object's type
ic-fork r0, 0

# Call C function to get property offset into `r1`
runtime-call <get_property_offset>, r0, r1

ic-join

# Load property from r0[r1] into r2
mem-load r0, r1, r2
ret r2
```

TBD

## Bytecode

Buoyant is a register-based VM.
Specific implementation of VM MAY place the limit of number
of registers, minimum number of TBD registers MUST be supported by all
implementations.

TBD

## LICENSE

This software is licensed under the MIT License.

Copyright Fedor Indutny, 2017.

Permission is hereby granted, free of charge, to any person obtaining a
copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to permit
persons to whom the Software is furnished to do so, subject to the
following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN
NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM,
DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR
OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE
USE OR OTHER DEALINGS IN THE SOFTWARE.

[0]: http://webassembly.org/
[1]: http://luajit.org/
