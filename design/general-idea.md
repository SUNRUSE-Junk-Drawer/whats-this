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

## Map

Maps strings, numbers or actor references to any other data type; similar to an 
anonymous object, i.e. 
`{ string-key: 30, 123: true, @4575EF21-6D87-4758-9CEA-CDA8E7F17580: "the item" }`.

The following all generate two separate properties; numbers, strings and actor
references are kept separate:

- `{ A575EF21-6D87-4758-9CEA-CDA8E7F17580: 30, @A575EF21-6D87-4758-9CEA-CDA8E7F17580: true }`
- `{ message.an-actor-reference: 30, message.string-representation-of-same-actor-reference: true }`
- `{ message.a-number: 30, message.string-representation-of-same-number: true }`

You can optionally quote strings used as keys:
`{ "this is a key with spaces in it.": 3 }`.

## List

An ordered list of any other data type, even mixed, i.e. 
`[30, true, "the item"]`.

## Actor reference

128-bit GUID/UUIDs.  The algorithm used to generate these is implementation
-defined.  Generally, these would not be entered as literals, but can be, in the
form `@4575EF21-6D87-4758-9CEA-CDA8E7F17580`.

# Scripting language

The scripting language used by message handlers is heavily inspired by SQL. 
Each message handler describes:

- A set of requirements which must be met for the message handler to run.
- A set of messages to send.
- If the message handler changes the receiving actor's state, a new value
  describing that state.

## Example

For instance, we have an actor describing a character, who is carrying items, 
and if they have a specific item, it should be dropped.  This state can be
modelled as:

```
{
    character: {
        location: @8db14087-db46-48ee-9b54-3ea35142406d
        inventory: {
            item-name-one: 5
            item-name-two: 8
            item-name-three: 2
        }
    }
}
```

A message comes in:

```
{
    drop-item: item-name-two
}
```

The following message handler would match the above scenario:

```
when actor is {
    character: {
        location: reference
        inventory: {
            message.drop-item: > 0
        }
    }
}

# We don't need to check the message here as everything inside it will have
# matched the actor.

# Generates a new actor ID, and sends it a message which will be used by another
# message handler to initialize its state.
tell new {
    initialize-pickup: {
        location: actor.character.location
        type: message.drop-item
    }
}

set character.inventory[message.drop-item] to character.inventory[message.drop-item] - 1
```

This would trigger another message handler:

```
when message is {
    initialize-pickup: {
        location: reference
        type: string
    }
}

# Tell the location where the item pickup spawned that there is a new item,
# giving our actor reference.
tell message.initialize-pickup.location {
    add-pickup: @actor
}

set pickup to {
    location: message.initialize-pickup.location
    type: message.initialize-pickup.type
}
```

Now, a modification could be added which adds grenades.  If this file was added:

```
when message is {
    initialize-pickup: {
        location: reference
        type: "grenade"
    }
}

tell message.initialize-pickup.location "explosion"
```

This uses a specificity system similar to CSS; as this new message handler has
more specific requirements before it will run, it will always try to match
before the other.