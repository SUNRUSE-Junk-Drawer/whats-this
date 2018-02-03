An in-memory sort-of-actor-system (more similar to Grains from Microsoft's 
Orleans) which is intended for building games.

Message handlers can be added and removed individually, which allows for "mods";
they are text files defining what to do when an actor matching a specific
pattern receives a message matching a specific pattern.