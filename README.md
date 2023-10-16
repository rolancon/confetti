# Confetti
Confetti is a generic data description language with semantics based on data algebra and a syntax derived from configuration files.
The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is based entirely on [Zermelo-Fraenkel set theory with the axiom of choice (ZFC)](https://en.wikipedia.org/wiki/Zermelo-Fraenkel_set_theory).

The basic data structure is a Couplet. This is very similar to a key-value pair, where the key gives context for (describes) the value. Couplets can be grouped together in a _set_: these are called Relations. Relations are quite similar to records in a database. Relations in turn are then grouped together in an enclosing set, know as a Clan, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a Horde, which is somewhat equivalent to a database.

These four data structures can generically model many different types of data stores and data formats, e.g.: key-value stores, document stores, relational databases, RDF triple stores, graph databases, configuration files, CSV files, spreadsheets, XML, JSON. The syntax to model these sets and couplets is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

## Terms

A single _term_ is called an _atom_. 

    term

The naming convention for a term is as follows: a _base term_ should start with a lower-case letter [a-z], potentially followed by one ore more lower case letters or digits [0-9]. More than one base term can be combined into a _compound term_ with a hyphen **-**. Examples of terms:

    t
    term1
    term-a
    term-a1

A couplet contains of left-hand side term and a right-hand side term, connected with an equal sign **=**. E.g.:

    left-term = right-term

A set consists of two or more terms, separated with a comma **,**:

    term-a, term-b

## Data stores

### Key-value stores

Key-value stores are directly supported with a couplet as follows, which models a key-value _pair_:

    key = value

Multiple key-value pairs are then simply added one below the other:

    key1 = value-a
    key2 = value-b

Together these couplets form a set (a set containing couplets is called a relation).

### Document stores

Document stores have more deeply nested structures. To support them use a _section header_: a section header consists of a pair of square brackets **[ ]**. In between the square brackets you specific the _path_ from one key to the next key, separated by dots **.**. Under the section header you specify the key-value that is nested in the key path.

    [key1.key2]
    key3 = value-a
    key4 = value-b

So in the above section, we actually have two separate paths: _key1.key2.key3_ and _key1.key2.key4_. These paths are actually nested couplets, from one key (_key1_) to the next one (_key2_) which functions as its value, but also as the key for the next couplet (_key2.key3_). The final couplet is then a regular key-value pair.

To model _trees_ in the data, which consists of a root nodes, nested nodes and terminal leaves, use a relative path in a subsidiary section header below the main section header. The subsidiary section header starts with a dot **.** to indicate this is a relative path to the previous absolute path. For example, to turn key3 and key4 in the previous example into sibling nodes from root key1.key2 containing their own nodes:

    [key1.key2]

    [.key3]
    key5 = value-a

    [.key4]
    key6 = value-b

In this example, the root is key1, which is extended with key2, together forming the absolute path key1.key2, which then branches into sibling nodes key3 and key4, indicated with a relative path, which both contains their own nodes (resp. key5 and key6) and terminal leaves (resp. value-a and value-b). So, key2 is the parent of keys key3 and key4, key1 is their ancestor, and vice versa, key3 and key4 are children to key2, and descendants of key1. 

### Relational databases

Relational databases extend further on document stores. There are multiple relations (often called records or rows). The relations have similar keys (known as columns), but different values (or fields). The relations are also are numbered incrementally.  This can be achieved automatically as follows:

    [table,]
    /
    key1 = value-a
    key2 = value-b
    /
    key1 = value-c
    key2 = value-d

The table header ends with a comma **,** to indicate that this is a separate set (a clan in Data Algebra terminology). The _/_ operator automatically increases a counter, and uses this in a new couplet, where the counter is the key, and relation is the value. So the first relation would then be identical to:

    [.1]
    key1 = value-a
    key2 = value-b

the second one would have a new section header _[.2]_, etc. These numbers are not supported for explicit, external term names (see section Terms), they are considered implicit, internal term names.

There are no _null_ values in set theory, therefore if a field value is _null_ in one of the relations, then just leave it out. Suppose _value-c_ of _key1_ is _null_ in the second row:

    /
    key2 = value-d

A table could optionally also contain a separate schema, which is a set of all the columns used in the relations:

    [table1,]
    key1, key2
    /

Internally the schema is stored under number 0:

    [.0]
    key1, key2

In order to add more than one table just add more table headers:

    [table1,]

    [table2,]

The Confetti file that contains these table definitions constitutes the database schema (or the horde in Data Algebra terminology). A file could be named _customer-db.cft_ to indicate that it contains the schema and data of a customer database, consisting of several tables that constitute the database.

### RDF triple stores

### Graph databases

