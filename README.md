# Confetti
Confetti is a generic data description language with semantics based on data algebra and a syntax derived from configuration files.
The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is entirely based on set theory.
The basic data structure is a Couplet. This is very similar to a key-value pair, where the key gives context for (describes) the value. Couplets can be grouped together in a set: these are called Relations. Relations are quite similar to records in a database. Relations in turn are then grouped together in an enclosing set, know as a Clan, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a Horde, which is somewhat equivalent to a database.

These four data structures can generically model many different types of databases and data formats, e.g.: key-value stores, document stores, relational databases, graph databases, spreadsheets, configuration files, JSON, XML. The syntax to model these sets and couplets is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

## Key-value stores

Key-value stores are supported with a couplet as follows:

    key = value

Multiple key-value pairs are then simply:

    key1 = valueA
    key2 = valueB

## Document stores

Document stores have more deeply nested structures. To support them use a section header: a section header consists of a pair of square brackets (_[ ]_). In between the square brackets you specific the path from one key to the next key, separated by dots (_._). Under the section header you specify the key-value that is nested in the key path.

    [key1.key2]
    key3 = valueA
    key4 = valueB

So in the above section, we actually have two paths: _key1.key2.key3_ and _key1.key2.key4_, which are both separate sets. These paths are actually nested couplets, from one key (_key1_) to the next one (_key2_) which functions as its value, but also as the key for the next couplet (_key2.key3_). The final couplet is then a regular key-value pair.



