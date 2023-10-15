# Confetti
Confetti is a generic data description language with semantics based on data algebra and a syntax derived from configuration files.
The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is entirely based on set theory.
The basic data structure is a Couplet. This is very similar to a key-value pair, where the key gives context for (describes) the value. Couplets can be grouped together in a set: these are called Relations. Relations are quite similar to records in a database. Relations in turn are then grouped together in an enclosing set, know as a Clan, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a Horde, which is somewhat equivalent to a database.

These four data structures can generically model many different types of databases and data formats, e.g.: key-value stores, document stores, relational databases, graph databases, spreadsheets, configuration files, JSON, XML. The syntax to model these sets and couplets is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

