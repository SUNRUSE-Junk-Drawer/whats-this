An in-memory sort-of-actor-system (more similar to Grains from Microsoft's 
Orleans) which is intended for building games.

Message handlers can be added and removed individually, which allows for "mods";
they are text files defining what to do when an actor matching a specific
pattern receives a message matching a specific pattern.

# Data

Messages and actor state build on the same data schema, which borrows most of
its general ideas from JSON.

## Boolean

Two values; `true` and `false`.

## String

A piece of text.  The exact format/encoding is implementation-defined, but
literals are in the form `"This is a string literal."`.  Quotation marks can be
escaped by doubling them, i.e. `"This: "" is a quotation mark."`.  Anything else
can be between the quotes as long as the quotes are closed by the end of the
file.

## Number

A 64-bit IEEE 754 "double" float.  Any number literal of the form `0`, `0.`,
`.0`, `0.0`, `+0`, `+0.`,`+.0`, `+0.0`, `-0`, `-0.`,`-.0` or `-0.0` is allowed.
Note that negative zero is distinct from positive zero, but otherwise, all of
the above have the same meaning.

## Dictionary

Maps strings to any other data type; similar to an anonymous object, i.e. 
`{ item-one: 30, item-two: true, item-three: "the item" }`.

## List

An ordered list of any other data type, even mixed, i.e. 
`[30, true, "the item"]`.

## Actor reference

128-bit GUID/UUIDs.  The algorithm used to generate these is implementation
-defined.  Generally, these would not be entered as literals, but can be, in the
form `@4575EF21-6D87-4758-9CEA-CDA8E7F17580`.