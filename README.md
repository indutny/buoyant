# Buoyant

**WORK IN PROGRESS**
**PRs are welcome**

A Universe in VM, or VM in the Universe?

## Idea

Fully adjustable language-agnostic bytecode VM with:

* Non-conservative Garbage Collector with configurable types
* Customizable bytecode
* Inline caches (i.o.w., capable of rewriting bytecode on the fly)

Goals:

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

NOTE: Although object pointer starts with the first heap slot, VM implementation
may put some PRIVATE fields right before it. Implementation MUST NOT overwrite
these slots.

NOTE: If object inherits from non-empty Type - parent's heap and binary slots
will be added to the object. The object's slots are the last ones in the layout.

Heap implementation provides default opcode for allocating object
of specific type. Number of arguments is 1.

### Customizable Bytecode

All bytecode opcodes are described in the source module. Each opcode
MUST have default implementation in C (supplied through shared library) OR MUST
be implemented in terms of other opcodes declared previously.

Each opcode MUST specify number of arguments.

OPTIONALLY source module may contain specialized implementations of
opcodes.
Two specializations are available that can be combined at the same time:

* By Architecture
* By Input Types

### Types

Following types are provided by default:

* HeapObject
* int64 (which may be boxed or unboxed, depending on VM implementation)
* double (which may be boxed or unboxed, depending on VM implementation)

Each type is described in the source module. Type definition MUST include:

* Parent type (can't be `int64` and `double`)
* Number of heap slots
* Number of binary slots

OPTIONALLY type may include reference to destructor opcode. This
opcode MUST have an argument count set equal to 1.

### Inline Cache

At first all opcodes start in `UNINITIALIZED` state. Once they are executed
: appropriate opcode implementation will be invoked. If no specializations
are available, or none matches the types. Opcode switches to `INITIALIZED`
state, a counter internal to the opcode location is set to `0`.

In `INITIALIZATION` phase every input type combination that hasn't been
previously seen increments internal counter. If the counter value is bigger than
configurable limit, opcode switches to `MEGAMORPHIC` phase.

In `MEGAMORPHIC` phase type-specific implementations of opcode are never
called.

NOTE: all states are local to the particular opcode location.

## Bytecode

Buoyant is a register-based VM.
Specific implementation of VM MAY place the limit of number
of registers, minimum number of TBD registers MUST be supported by all
implementations.

There are default opcodes that MUST be provided by VM for allocating:

* Object of non-int64, non-double type
* int64 value
* double value

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
