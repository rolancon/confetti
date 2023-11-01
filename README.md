# Confetti
Confetti is a generic data description language with semantics based on data algebra and a syntax derived from configuration files. Confetti is an extension of the [K-V](https://github.com/rolancon/key-value) data format. The file extension for Confetti files is _.cft_.

The data structures in Confetti mirror those found in [Data Algebra](https://algebraixlib.readthedocs.io/en/latest/intro.html), which is based entirely on [Zermelo-Fraenkel set theory with the axiom of choice (ZFC)](https://en.wikipedia.org/wiki/Zermelo-Fraenkel_set_theory).

The basic data structure in Data Algebra is a Couplet, called a pair in Confetti. This is very similar to a key-value pair, where the key gives context for (describes) the value. See [K-V](https://github.com/rolancon/key-value) for more information.
Couplets can be grouped together in a _set_: these are called Relations in Data Algebra, and tuples in Confetti. Tuples are quite similar to records in a database. Tuples in turn are then grouped together in an enclosing set, know as a Clan, which mimics a table in a database schema. Finally, Clans could be grouped together in a set called a Horde, which is somewhat equivalent to a database.

These four data structures can generically model many different types of data stores and data formats, e.g.: key-value stores, document stores, relational databases, RDF triple stores, graph databases, configuration files, CSV files, spreadsheets, XML, JSON. The syntax to model these sets and couplets is derived from [configuration files](https://github.com/madmurphy/libconfini/blob/master/MANUAL.md), also known as [INI files](https://en.wikipedia.org/wiki/INI_file) on Windows.

The syntax of Confetti for data structures is limited to the character set and whitespace rules of [Lazycode](https://github.com/rolancon/lazycode-minicode).

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

The backslash **\\** newline escape can be used to break up one line over two lines:

    term-a, \
    term-b

## Comments

Single-line comments start with one semicolon **;** on a separate line:

    ; I am an atom
    term

Multi-line comments start end with two consecutive semicolons **;;** on separate lines:

    ;;
      This is a set containing two atoms:
      term-a
      term-b
    ;;
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

    [1]
    key1 = value-a
    key2 = value-b

the second one would have a new section header _[2]_, etc. These numbers are not supported for explicit, external term names (see section Terms), they are considered implicit, internal term names.

There are no _null_ values in set theory, therefore if a field value is _null_ in one of the relations, then just leave it out. Suppose _value-c_ of _key1_ is _null_ in the second row:

    /
    key2 = value-d

A table could optionally also contain a separate schema, which is a set of all the columns used in the relations:

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

This triple could be decomposed in the following two couplets

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
    
The subject can left unspecified, which is called a _blank node_, using a hyphen **-** as placeholder:

    -.eats.food

meaning someone unspecified eats food.

A triple itself can be part of another triple by stringing them together with spaces, and broken up over multiple lines using the backslash **\\** newline escape. Using a combination of triples with blank nodes we can combine them in creative ways, such as:

    -.named.alice knows \
     [-]
     named.bob
     who-knows -.named.eve

By replacing the hyphen with _someone_, and repeating the _[-]_, this reads as the following sentence:

    Someone named Alice knows someone named Bob, (and someone) who knows someone named Eve.

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

Annotations are part of a set (relation), therefore there can be more than one:

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

### Configuration files

Since the syntax of Confetti is directly derived from configuration files, the basic section headers and key-value pairs are equivalent to configs:

    [config.path]
    config-key = config-value
    
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

The slashes between the field values cause them to be added as a list (couplets in a set, with an autoincrementing number for the left and the actual field value for the right).

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

JSON has two scalar data types: _objects_ and _arrays_. Objects are similar to trees, a collection of nested couplets, as modelled in document stores and graphs. The toplevel, however, can contain more than one node. An example of an object with one toplevel pair (a root):

    [object-root]
    object-child1 = value1
    object-child2 = value2

An example of an object containing two toplevel couplets:

    [object-root1, object-root2]
    
     [object-root1]
     object-child1 = value1
     object-child2 = value2
    
     [object-root2]
     object-child3 = value3
     object-child4 = value4

Confetti does not have a direct equivalent for arrays, which are ordered, indexed lists. A list nested at most one level deep can be modelled similar to CSV files without a header, with autogenerated numbers for the left-hand side of couplets, where the list values are then on the right. A single list looks like:

    list-value1/list-value2

and a list nested one level deep looks like:

    /
    list-value1/list-value2
    /
    list-value3/list-value4

For the support for JSON's atomic datatypes, see the Types section below.

### Types

So far we've only seen examples of the term type, which is a special case of the string type. For atoms and leaf nodes Confetti, however, also support types which are very similar to JSON types. They are:
- the **null** type
- the **boolean** type
- the **number** type
- the **string** type.

The null type has no syntactic encoding, but is basically denotated by an empty value:

    null = 

The boolean type contains two values:

    bool = yes

for **true** values and

    bool = no

for **false** values.

The number is very similar to the JSON number type. A number can optionally have a negative sign **-**, a decimal separator **.** and an exponent (lowercase **e**) with an optional negative sign **-**:

    num = 1
    num = 19
    num = -2
    num = 2.7
    num = 3.70
    num = -4.0
    num = 5e6
    num = -5e6
    num = 5e-6
    num = -5e-6
    num = 5.8e6
    num = -5.8e6
    num = 5.8e-6
    num = -5.8e-6

The string datatype can be denotated in several different ways. In all cases whitespace before and after the actual string is not included. 

It can be encoded as literal printable ASCII characters on a single line:
  
    str = this is 1 line with: numbers, letters and 2 symbols

The syntax of the string datatype is extended to support the character set of Minicode, which means uppercase letters and shifted symbols from the keyboard are also allowed in a value:

    str = These are just some lowercase and uppercase ASCII characters including number 1.1 and shifted symbol #. 

It can be split up over two different lines using the backslash **\\** newline separator:

    str = This line is split between this line \
     and the next line.

unless the backslash is doubled, which counts as an escape:

    str = This line is not split and the backslash is part of the string: \\

It can be surrounded by single quotes **'**. Two single quotes constitute an empty string:

    empty-str = ''

In the case that quoted string is not empty several other rules apply -->

The whitespace at the start and end of the string that is within the quotes is included.

    str = ' two extra spaces '

Several characters escaped with the backslash characters have special interpretation:

    single-quote = '\''
    backslash = '\\'
    newline = '\n'
    tab = '\t'
    carriage-return = '\r'
    vertical-tab = '\v'
    formfeed = '\f'
    ascii-control-character = '\x00'
    non-ascii-unicode-character = '\uffff'
    emoji-character = '\j10ffff'

A single newline separator cannot appear in the string, unless it is positioned immediately after the opening quote and followed by a newline, in that case the first newline is removed from the string:

    str = '\
     the backslash and newline from the previous line are not part of this string'

and character escapes are interpreted literally in such strings (raw strings). To be able to include single quotes in the text, the number of surrounding single quotes should be one higher than the number of consecutive enclosed single quotes:

    str = ''\
     This single quote ' and other characters like this backslash \ need no escape.''

And for characters that could be interpreted as other datatypes, string interpretation can be enforced by doubling the equals sign **=**:

    empty-str-not-null == 
    str-not-bool == yes

A set in Confetti is multiple values separated by commas **,**:

    set = a, b, c

A list in Confetti is multiple values separated by slashes **/**:

    list = a/b/c
    
Moreover, Confetti has special notations for an empty set

    empty-set = ,
    
and an empty list

    empty-list = /

which reuse the separators of sets and lists.
