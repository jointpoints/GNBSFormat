# GNBS file format

GNBS (**G**raph description with **N**o **B**ull**S**hit) is a file format for storing graphs (networks) in a compact and human-readable way.

Nowadays, most of the popular file formats for graphs use either XML (GraphML, GEXF, etc.) or JSON (GraphSON, JGF, etc.) serialisation, which often makes the structure of these files unnecessarily complicated and bulky, especially for relatively small graphs with relatively small number of attributes.

This is exactly where GNBS comes into play.

Main features of GNBS are:
* Minimalistic and straightforward structure.
* Easy to parse.
* Possibility to define both directed and undirected edges.
* Possibility to define **statically typed** attributes for both vertices and edges with wide variety of acceptable types.

Main disadvantages of GNBS are:
* Stores only one graph per file.
* No support for dynamic graphs.





# Minimal example

Here we demonstrate how the same graph taken from https://gexf.net/index.html can be stored in GEXF and in GNBS formats.



#### GEXF format

```
<?xml version="1.0" encoding="UTF-8"?>
<gexf xmlns="http://gexf.net/1.2" version="1.2">
    <meta lastmodifieddate="2009-03-20">
        <creator>Gexf.net</creator>
        <description>A hello world! file</description>
    </meta>
    <graph mode="static" defaultedgetype="directed">
        <nodes>
            <node id="0" label="Hello" />
            <node id="1" label="World" />
        </nodes>
        <edges>
            <edge id="0" source="0" target="1" />
        </edges>
    </graph>
</gexf>
```



#### GNBS format

```
AV S my vertex label
V 0 "Hello"
V 1 "World"
A 0 1
```





# Format specification

GNBS is a plain-text declarative format used to define graphs/networks.

Each GNBS file is a sequence of _declarations_. Each declaration takes one line of text. Each declaration is a sequence of _tokens_ separated by whitespace characters. Tokens in GNBS language can be _atomic_ and _complex_.



#### 1. Atomic tokens

An _atomic token_ is the primitive unit of the GNBS language that cannot be broken down into any further components. Tokens can be of different kinds:
1. Declaration specifiers
2. Type specifiers
3. Boolean literals
4. Integer literals
5. Rational literals
6. String literals
7. Void literal

Boolean, integer, rational and string literals will be collectively called _proper literals_. Proper literals and the void literal will be called _literals_.

###### 1.1. Declaration specifiers

_Declaration specifiers_ determine what exactly each declaration is declaring. See [Declarations](#3-declarations) for more details. There're in total 6 possible declaration specifiers:

| Declaration specifier |  Type of declaration |
| --------------------- | -------------------- |
| `AV` | Declaration of a new vertex attribute |
| `AE` | Declaration of a new edge attribute   |
|  `V` | Declaration of a new vertex           |
|  `A` | Declaration of a new directed edge    |
|  `E` | Declaration of a new undirected edge  |
|  `#` | Declaration of a comment              |

###### 1.2. Type specifiers

_Type specifiers_ name different types that can be assigned to vertex/edge attributes. There're in total 34 type specifiers. Among them there're 12 _primitive type specifiers_ and 22 _compound type specifiers_.

| Primitive type specifier | Type |
|--------------------------|------|
| `B` | Boolean |
| `S` | String |
| `U1` | A 1-byte (8-bit) unsigned integer |
| `U2` | A 2-byte (16-bit) unsigned integer |
| `U4` | A 4-byte (32-bit) unsigned integer |
| `U8` | An 8-byte (64-bit) unsigned integer |
| `I1` | A 1-byte (8-bit) signed integer |
| `I2` | A 2-byte (16-bit) signed integer |
| `I4` | A 4-byte (32-bit) signed integer |
| `I8` | An 8-byte (64-bit) signed integer |
| `F4` | A 4-byte (32-bit) floating-point number |
| `F8` | An 8-byte (64-bit) floating-point number |

| Compound type specifier |  Type  |
|-------------------------|--------|
| `L*` | A list (array) of `*`     |
| `C*` | A collection (set) of `*` |

For the compound type specifiers, `*` must be replaced with a primitive type specifier, e.g. `LU8` is an array of 8-byte unsigned integers. Exceptions: `CF4` and `CF8`. These two types would be ill-defined due to the problematic assessment of equality of floating-point numbers. Therefore, these two types don't exist in GNBS. Note also that nested compound types like `LLI8` (an array of arrays of `I8`) or `CLCLB` (a set of arrays of sets of Boolean arrays) are currently not supported as well.

###### 1.3. Boolean literals

There're two Boolean literals representing two logical states: `T` (true) and `F` (false).

###### 1.4. Integer literals

An integer literal represents an integer and is defined by the regular expression `(+|-)?0|([1-9][0-9]*)`.

E.g. `0`, `5`, `-10`.

###### 1.5. Rational literals

A rational literal represents a floating-point number and is defined by the regular expression `(+|-)?0|([1-9][0-9]*)(\.[0-9]+)?(e|E(+|-)?0|([1-9][0-9]*))?`.

E.g. `0`, `-3.1415926535`, `1.123456e+18`.

###### 1.6. String literals

A string literal represents a string of characters and is defined by the regular expression `".*?"`. Note that escape sequences aren't supported yet.

E.g. `""`, `"123"`, `"Hello, world!"`.

###### 1.7. Void literal

The void literal `X` marks that a vertex/edge attribute doesn't have any value for a particular vertex/edge.



#### 2. Complex tokens

_Complex tokens_ are made up of atomic tokens. There're 2 kinds of complex tokens: _list tokens_ and _collection tokens_.

###### 2.1. List tokens

List tokens represent arrays. They're made of multiple proper literals of the same type separated by commas inside square brackets.

E.g. `[]`, `[1, 2, 3]`, `[T,F,F,   F,T  ]`, `[ "Hello, world!" ]`.

###### 2.2. Collection tokens

Collection tokens represent sets. They're made of multiple proper literals of the same type separated by commas inside curly brackets.

E.g. `{}`, `{1, 2, 3}`, `{T,F,F,   F,T  }`, `{ "Hello, world!" }`.



#### 3. Declarations

Each line of a GNBS file is a _declaration_. All declarations have the same general structure:
```
{declaration specifier} {arguments separated by whitespace characters...}
```

###### 3.1. Declaration of a new vertex attribute

A vertex attribute is defined with the following line:
```
AV {type specifier} {type name}
```
Here, the `{type specifier}` determines the type of the new vertex attribute, the `{type name}` determines the name of the new vertex attribute. `{type name}` is a string literal without quotation marks.

###### 3.2. Declaration of a new edge attribute

A vertex attribute is defined with the following line:
```
AE {type specifier} {type name}
```
Here, the `{type specifier}` determines the type of the new edge attribute, the `{type name}` determines the name of the new edge attribute. `{type name}` is a string literal without quotation marks.

###### 3.3. Declaration of a new vertex

A vertex is defined with the following line:
```
V {unsigned integer literal} {literals...}
```
Here, the `{unsigned integer literal}` determines the ID of the new vertex, `{literals...}` define values for the vertex attributes in the order of their introduction with [`AV` declarations](#3-1-declaration-of-a-new-vertex-attribute). Use the void literal `X` if the new vertex must not have any value for the respective vertex attribute.

###### 3.4. Declaration of a new directed edge

A directed edge is defined with the following line:
```
A {unsigned integer literal 1} {unsigned integer literal 2} {literals...}
```
Here, the `{unsigned integer literal 1}` determines the ID of the source vertex, the `{unsigned integer literal 2}` determines the ID of the target vertex, `{literals...}` define values for the edge attributes in the order of their introduction with [`AE` declarations](#3-2-declaration-of-a-new-edge-attribute). Use the void literal `X` if the new edge must not have any value for the respective edge attribute.

###### 3.5. Declaration of a new undirected edge

An undirected edge is defined with the following line:
```
E {unsigned integer literal 1} {unsigned integer literal 2} {literals...}
```
Here, the `{unsigned integer literal 1}` and the `{unsigned integer literal 2}` determine the IDs of the incident vertices, `{literals...}` define values for the edge attributes in the order of their introduction with [`AE` declarations](#3-2-declaration-of-a-new-edge-attribute). Use the void literal `X` if the new edge must not have any value for the respective edge attribute.



#### 4. Additional structural requirements

* All declarations of vertex attributes must go **before** the first vertex is declared.
* All declarations of edge attributes must go **before** the first edge (whether directed or undirected) is declared.
* Declarations of vertices must go **before** the declarations of edges.
* All vertex IDs must be unique.
* Both vertex IDs used in a declaration of an edge must correspond to a declared vertex.
