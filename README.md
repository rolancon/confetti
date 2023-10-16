# Confetti
Confetti is a generic data description language with semantics based on data algebra and a syntax derived from configuration files.
The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is entirely based on [Zermelo-Fraenkel set theory with the axiom of choice (ZFC)](https://en.wikipedia.org/wiki/Zermelo-Fraenkel_set_theory).

The basic data structure is a Couplet. This is very similar to a key-value pair, where the key gives context for (describes) the value. Couplets can be grouped together in a set: these are called Relations. Relations are quite similar to records in a database. Relations in turn are then grouped together in an enclosing set, know as a Clan, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a Horde, which is somewhat equivalent to a database.

These four data structures can generically model many different types of data stores and data formats, e.g.: key-value stores, document stores, relational databases, RDF triple stores, graph databases, configuration files, CSV files, spreadsheets, XML, JSON. The syntax to model these sets and couplets is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

## Basics



## Data stores

### Key-value stores

Key-value stores are simply supported with a couplet as follows:

    key = value

Multiple key-value pairs are then simply:

    key1 = valueA
    key2 = valueB

Together these couplets form a set (a set containing couplets is called a relation).

### Document stores

Document stores have more deeply nested structures. To support them use a section header: a section header consists of a pair of square brackets (_[ ]_). In between the square brackets you specific the path from one key to the next key, separated by dots (_._). Under the section header you specify the key-value that is nested in the key path.

    [key1.key2]
    key3 = valueA
    key4 = valueB

So in the above section, we actually have two separate paths: _key1.key2.key3_ and _key1.key2.key4_. These paths are actually nested couplets, from one key (_key1_) to the next one (_key2_) which functions as its value, but also as the key for the next couplet (_key2.key3_). The final couplet is then a regular key-value pair.

### Relational databases

Relational databases extend further on document stores. There are multiple relations (often called records or rows). The relations have similar keys (known as columns), but different values (or fields). The relations are also are numbered incrementally.  This can be achieved automatically as follows:

    [schema.table]
    /
    key1 = valueA
    key2 = valueB
    /
    key1 = valueC
    key2 = valueD

The _/_ operator automatically increases a counter, and uses this in a new couplet, where the counter is the key, and relation is the value. So the first relation would then be identical to:

    [1]
    key1 = valueA
    key2 = valueB

the second one would have a new section header _[2]_, etc.

There are no _null_ values in set theory, therefore if a field value is _null_ in one of the relations, then just leave it out. Suppose _valueC_ of _key1_ is _null_ in the second row:

    /
    key2 = valueD

### RDF triple stores

### Graph databases

