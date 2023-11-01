# Confetti
Confetti is a generic data description language with semantics based on _Data Algebra_ and a syntax derived from _configuration files_. Confetti is an extension of the [K-V](https://github.com/rolancon/key-value) data format. The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is based entirely on [Zermelo-Fraenkel set theory with the axiom of choice (ZFC)](https://en.wikipedia.org/wiki/Zermelo-Fraenkel_set_theory).

The basic data structure in Data Algebra is a _Couplet_, called a _pair_ in Confetti. This is very similar to a key-value pair, where the key gives context for (describes) the value. See [K-V](https://github.com/rolancon/key-value) for more information.
Couplets can be grouped together in a _set_: these are called _Relations_ in Data Algebra, and _tuples_ in Confetti. Tuples are quite similar to records in a database. Tuples in turn are then grouped together in an enclosing set, know as a _Clan_, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a _Horde_, which is somewhat equivalent to a database.

These four data structures can generically model many different types of data stores and data formats, e.g.: key-value stores and [dynamically typed values](https://github.com/rolancon/key-value/blob/main/README.md#types) as in K-V, and configuration files, document stores, relational databases, RDF triple stores, graph databases, CSV files, spreadsheets, XML and JSON as in Confetti. The syntax to model these sets and pairs is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

Just as in K-V, the syntax of Confetti for data structures is limited to the character set and [whitespace rules](https://github.com/rolancon/lazycode-minicode/blob/main/README.md#whitespace) of [Lazycode](https://github.com/rolancon/lazycode-minicode/blob/main/README.md#lazycode), the syntax of values is extended to the character set of [Minicode](https://github.com/rolancon/lazycode-minicode/blob/main/README.md#minicode).

## Basics

A set consists of two or more terms, separated with a comma **,**:

    term-a, term-b

### Configuration files

Since the syntax of Confetti is directly derived from configuration files, the basic section headers and key-value pairs are equivalent to configs:

    [config]
    key = value

This means that key _key_ with value _value_ is part of _config_, i.e., there is a path _config.key_ which leads to the value. The path is actually a syntactic convenience for a combination of these two different pairs:

    config = key
    key = value

This is even more convenient when you need to store multiple pairs under the same config, so you don't need to keep repeating the path from config to its keys:

    [config]
    key1 = value1
    key2 = value2

### Document stores

Document stores have more deeply nested structures. To support them use a _section header_ like in a configuration file: a section header consists of a pair of square brackets **[ ]**. In between the square brackets you specific the _path_ from one key to the next key, separated by dots **.**. Under the section header you specify the pairs that are nested in the key path:

    [key1.key2]
    key3 = value-a
    key4 = value-b

So in the above section, we actually have two separate paths: _key1.key2.key3_ and _key1.key2.key4_. These paths are actually nested pairs, from one key (_key1_) to the next one (_key2_) which functions as its value, but also as the key for the next pair (_key2.key3_). The final pair is then a regular key-value pair.

To model _trees_ in the data, which consists of a root nodes, nested nodes and terminal leaves, use a relative path in a subsidiary section header below the main section header. The subsidiary section header starts with a dot **.** to indicate this is a relative path to the previous absolute path. For example, to turn key3 and key4 in the previous example into sibling nodes from root key1.key2 containing their own nodes:

    [key1.key2]

    [.key3]
    key5 = value-a

    [.key4]
    key6 = value-b

In this example, the root is key1, which is extended with key2, together forming the absolute path key1.key2, which then branches into sibling nodes key3 and key4, indicated with a relative path, which both contains their own nodes (resp. key5 and key6) and terminal leaves (resp. value-a and value-b). So, key2 is the parent of keys key3 and key4, key1 is their ancestor, and vice versa, key3 and key4 are children to key2, and descendants of key1. 

## Data stores

### Relational databases

Relational databases extend further on document stores. There are multiple tuples (often called relations, records or rows). The tuples have similar keys (known as columns), but different values (or fields). The tuples are also are numbered incrementally.  This can be achieved automatically as follows:

    [table,]
    /
    key1 = value-a
    key2 = value-b
    /
    key1 = value-c
    key2 = value-d

The table header ends with a comma **,** to indicate that this is a separate set (a clan in Data Algebra terminology). The _/_ operator automatically increases a counter, and uses this in a new pair, where the counter is the key, and the tuple is the value. So the first tuple would then be identical to:

    [1]
    key1 = value-a
    key2 = value-b

the second one would have a new section header _[2]_, etc. These numbers are not supported for explicit, external term names (see section Terms), they are considered implicit, internal term names.

There are no _null_ values in set theory, therefore if a field value is _null_ in one of the tuples, then just leave it out. Suppose _value-c_ of _key1_ is _null_ in the second row:

    /
    key2 = value-d

A table could optionally also contain a separate schema, which is a set of all the columns used in the tuples:

    [table1,]
    key1, key2
    /

Internally the schema is stored under number 0:

    [0]
    key1, key2

In order to add more than one table just add more table headers:

    [table1,]

    [table2,]

The Confetti file that contains these table definitions constitutes the database (or the horde in Data Algebra terminology). A file could be named _customer-db.cft_ to indicate that it contains the schema and data of a customer database, consisting of several tables that constitute the database.

### RDF triple stores

A _triple_ is a data structure which mimics a sentence consisting of a subject, a predicate and an object. In the sentence

   We eat food.

**we** is the subject, **eat** the predicate, and **food** is the object.

In Confetti this looks like three terms combined with dots **.**, which indicate the ordering from left to right:

    we.eat.food

This triple could be decomposed in the following two pairs

    we = eat
    eat = food

but then the ordering disappears; still, it is not possible to construct a different triple since the only way they compose is through the predicate eat.

Suppose we'd like to add another triple _We drink water_:

    we.eat.food
    we.drink.water

This causes some reduplication with the subject. If in the future we might like to add more triples based on the same subject. In that case we can use the following shorthand:

    [we]
    eat.food
    drink.water
    
The same principle applies if we have the same subject and predicate with different objects:

    we.eat.fruit
    we.eat.candy

This can be more consisely expressed as:

    [we.eat]
    fruit
    candy

Basically the object becomes a collection. The object can also be an empty collection (nil in RDF). This can be expressed by leaving out the collection:

   [we.eat]

The subject can left unspecified, which is called a _blank node_, using a hyphen **-** as placeholder:

    -.eats.food

meaning someone unspecified eats food.

A triple itself can be part of another triple by stringing them together with dots (preferable separated by spaces), or by breaking them up over multiple lines using the backslash **\\** newline escape and a section header, below which triples are strung together with slashes on separate lines. Using a combination of triples with blank nodes we can combine them in creative ways, such as:

    -.named.alice . knows . \
     [-]
     /
     named.bob
     /
     who-knows . -.named.eve

By replacing the hyphen with _someone_, and repeating the _[-]_, this reads as the following sentence:

    Someone named Alice knows someone named Bob, and someone who knows someone named Eve.

The inspiration for this functionality comes from a section in the [Turtle](https://www.w3.org/TR/turtle/#unlabeled-bnodes) manual - Turtle is a terse RDF triple language.

### Graph databases

Graph databases have many similarities with triple stores. They model relations between terms. But instead of only 3 (potentially nested) terms in a triple, they can be of arbitrary length. Furthermore, the terms (_nodes_ in graph terminology) can be labeled or annotated with key-value pairs. Also, the paths between nodes (_edges_ in graph terminology) can be labeled or annotated with key-value pairs. Lastly, the edges between the nodes can be either simple or directed.

A directed graph with three nodes, without labels and annotations, is very much like a triple:

    a.b.c

but they can go be extended beyond three nodes:

    a.b.c.d

The last node in a graph path can be labelled or annotated with a section header:
A graph where the c node has a label:

    [a.b.c]
    label

or an annotation for the c node:

    [a.b.c]
    key = value

To anotate a node which is not the end node, add the path to the node to be annotated to a section header:

    [a.b]
    key-for-b = some-value

If the above annotation appears in a Confetti file in addition to the previous annotation, then they are additive, meaning both b and c nodes will get an annotation. This process can be repeated for any node that needs a label or annotation.

Annotations are part of a set (tuples), therefore there can be more than one:

    [a.b]
    key-for-b = some-value
    another-key-for-b = another-value

Trees can also be constructed as graphs, just like in document stores:

    [a.b]

    [.c]
    e = value1

    [.d]
    f = value2

The difference with trees is that nodes in graphs can connect back to other nodes:

    [a.b]
    c.e
    d.e

Here nodes c and d branch off (as in a tree), but then connect back common node e.

Nodes can also connect to themselves:

    a.a
    
Next to directed edges, graphs also support simple edges (the nodes are connected without direction from one node to the other). These nodes are simply sets:

    a, b, c

Adding a label or annotation to a set will cause all nodes to have the same label. I.e.,

    [a, b, c]
    label

will result in:

    a = label
    b = label
    c = label

Since that is probably not what you intented, just add labels separately with a key-value pair, as shown above. Annotations are added with a section header:

    [a]
    key = value

and also in this case, using multiple nodes as in a set will add the same annotation to all nodes:

    [a, b]
    key = value

After this both nodes a and b have annotation key = value.

Edges can also get labels or annotations. For directed graphs, use the following syntax:

    [a/b]
    edge-label

The slash **/** indicates that the set operations should be done on the edge, not on the nodes. Again, like with nodes, labels or annotations on a directed graph are only added to the final edge:

    [a/b/c]
    edge-key = value

will only add the annotation to the final edge between nodes b and c. To add one on the first edge between nodes a and b, repeat the path in another section header:

    [a/b]
    edge-key-a-b = value

Adding labels and annotations to a simple graph uses the following syntax:

    [a\b\c]
    edge-key = value

which adds the annotation to both edges: from a to b, and from b to c.

To add a label or annotation to just one edge, only repeat that path in a separate section header:

    [a\b]
    edge-label-a-b

## Data formats
    
### CSV files

CSV files are quite similar to tables in relational databases. The following example is derived from an earlier example in the relational databases section:

    [csv-file,]
    column1, column2
    /
    column1 = field-value-a
    column2 = field-value-b
    /
    column1 = field-value-c
    column2 = field-value-d

which models a CSV file with a header column1, column2 and two rows with field values.

Headerless CSV files have no column names and depend on order, mandated with slashes:

    [csv-file,]
    /
    field-value-a/field-value-b
    /
    field-value-c/field-value-d

The slashes between the field values cause them to be added as a list (pairs in a set, with an autoincrementing number for the left and the actual field value for the right).

### Spreadsheets

Spreadsheets have some similarities with CSV files with a header. The fields in a spreadsheet are called _cells_, and their layout and naming is also dictated through named verticals columns and numbered horizontal rows.  Since one spreadsheet can contain multiple tabs, a tab is part of a set (a clan).

    [spreadheet-tab,]
    /
    aa = some-amount
    ac = cell-value-ac1
    /
    aa = some-other-amount
    ac = cell-value-ac2
    /
    /
    aa = total
    ac = sum-of-ac1-and-ac2

The aa columnn contains the labels: two rows with names in their cells, labelling the column next to it (ac) as containing different amounts. As you can see from the naming, the second column, ab, has been skipped. The third row is skipped, and the fourth row shows that the cell value in column ac contains the total: the sum of the two cell values of the first two two rows in the same column.
    
### XML

The hierarchical structure of XML, consisting of nested _tags_ and the possibility of _attributes_ (with or without value) on tags can be modelled straightforward way in Confetti. Text in a tag can only be modelled if it's contained in a leaf node, not in the case of mixed nodes (nodes containinig a mix of nodes and text):

    [xml-file,]
    [outer-tag]
    attribute1 = 
    attribute2 = value
     [.inner-tag]
     text

The outer tag contains two attributes, one empty and the other with a value, the inner tag (nested inside the outer tag) contains only text.

### JSON

JSON has two scalar data types: _objects_ and _arrays_. Objects are similar to trees, a collection of nested pairs, as modelled in document stores and graphs. The toplevel, however, can contain more than one node. An example of an object with one toplevel pair (a root):

    [object-root]
    object-child1 = value1
    object-child2 = value2

An example of an object containing two toplevel pairs:

    [object-root1, object-root2]
    
     [object-root1]
     object-child1 = value1
     object-child2 = value2
    
     [object-root2]
     object-child3 = value3
     object-child4 = value4

Confetti does not have a direct equivalent for arrays, which are ordered, indexed lists. A list nested at most one level deep can be modelled similar to CSV files without a header, with autogenerated numbers for the left-hand side of pairs, where the list values are then on the right. A single list looks like:

    list-value1/list-value2

and a list nested one level deep looks like:

    /
    list-value1/list-value2
    /
    list-value3/list-value4

## Types

Whereas K-V only supports pairs in one set (a tuple), with the atomic data types (boolean, number, Minicode character and empty) and one compound data type (string), Confetti also supports other compound data types: multiple sets and levels of sets (tuples, clans and hordes), maps (as a syntactic path notation for nested pairs), lists, plus their empty type.

For the support for atomic datatypes, see the [Types](https://github.com/rolancon/key-value/blob/main/README.md#types) section in K-V.

Clans (sets of tuples) are multiple section headers with separate sets:
    [set1,]

    [set2,]

Multiple sets with nested tuples (sets of pairs) are supported using section headers with bodies:

    [set1,]
    tuple1-key1=value-a
    tuple1-key2=value-b

    [set2,]
    tuple1-key1=value-c
    tuple1-key2=value-d

Hordes (sets of clans) are not directly expressed in Confetti, but are supported through multiple files in the same directory, where each file is a separate horde containing its own clans. The filename is then the name of the horde, and should therefore conform to the [terms naming convention](https://github.com/rolancon/key-value/tree/main#terms) as explained in K-V.

A set in Confetti is multiple values separated by commas **,**:

    set = a, b, c

A map is multiple values separated by dots **.**:

    map = a.b.c

A list in Confetti is multiple values separated by slashes **/**:

    list = a/b/c

Moreover, Confetti has special notations for an empty set

    empty-set = ,

and an empty map

    empty-list = .
    
and an empty list

    empty-list = /

which reuse the separators of sets, maps and lists.

An empty pair can only be denoted in the context of an empty map where 'the pair is empty': **.**. Empty tuples, clans and hordes, since they are also sets, are also denoted as an empty set: **,**.
