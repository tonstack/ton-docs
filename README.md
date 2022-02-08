## TL-B Language

TL-B (Type Language - Binary) serves to describe the type system, constructors, and existing functions. For example we can use TL-B schemes to build binary structures associated with the TON blockchain. Special TL-B parsers can read schemes to deserialize binary data into different objects.

### Table of contents
- [TL-B Language](#tl-b-language)
  - [Table of contents](#table-of-contents)
  - [Overview](#overview)
  - [Constructor rules](#constructor-rules)
  - [Namespaces](#namespaces)
  - [Comments](#comments)
  - [Library usage](#library-usage)
  - [Useful sources](#useful-sources)

### Overview

We refer to any set of TL-B constructs as TL-B documents. A TL-B document usually consists of two sections, which are separated by the `---functions---` keyword. The first section consists of declarations of types (i.e. their constructors). The second section consists of the declared functions, i.e. functional combinators. 

We may not follow the separation principle in small documents with no more than one functional combinator.

The first and second sections consist of combinator declarations, each ending with a semicolon(`;`). However, the first section contains only constructors, while the second section only involves functions. Each combinator is declared using a “combinator declaration” in the format explained above. However, the combinator number and field names may be explicitly assigned.

If additional type declarations are required after functions have been declared, the keyword (section divider)  
`---types---` is used. Furthermore, a functional combinator may be declared in the type section if its result type begins with an exclamation point (in fact, when the function section is interpreted, this exclamation point is added automatically).

To explicitly define 32-bit names of combinators, a hash mark (`#`) is added immediately after the combinator’s name, followed by 8 hexadecimal digits. In this documentation we will look at an example of how to correctly define a 32-bit combinator name for an internal message in the TON blockchain.

Here is an example of a possible TL-B document.    
<img alt="tlb structure" src="img/tlb.svg" width="100%">


### Constructor rules

```c++
// ....
hm_edge#_ {n:#} {X:Type} {l:#} {m:#} label:(HmLabel ~l n)
{n = (~m) + l} node:(HashmapNode m X) = Hashmap n X;

hmn_leaf#_ {X:Type} value:X = HashmapNode 0 X;
// ....
```

The left-hand side of each equation describes a way to define, or even to serialize, a value of the type indicated in the right-hand side. Such a description begins with the name of a constructor, such as `hm_edge` or `hml_long`, immediately followed by an optional constructor tag, such as `#_` or `$10`, which describes the bitstring used to encode (serialize) the constructor in question.

learn by examples!
| constructor           | serialization                         |
|-----------------------|---------------------------------------|
| `some#0x3f5476ca`     | 32-bit uint serialize from hex value  |
| `some$0101`           | serialize `0101` raw bits             |
| `some#`               | serialize crc32 of string `some`      |
| `some#_` or `some$_`  | serialize nothing                     |


Tags may be given in either binary (after a dollar sign) or hexadecimal notation (after a hash sign).If a tag is not explicitly provided, TL-B parser must computes a default 32-bit constructor tag by crc32 hashing the text of the “equation” defining this constructor in a certain fashion. Therefore, empty tags must be explicitly provided by `#_` or `$_`. 

All constructor names must be distinct, and constructor tags for the same type must constitute a prefix code (otherwise the deserialization would not be
unique).



### Namespaces

Composite constructions like `<namespace_identifier>.<constructor_identifier>` and `<namespace_identifier>.<Type_identifier>` can be used as constructor- or type identifiers. The portion of the identifier to the left of the period is called the namespace. Moreover, the rule about a first uppercase letter in type identifiers and lowercase letter in constructor identifiers applies to the part of the construction after the period. For example, `msg.Body` would be a type, while `data.std_message` would be a constructor.

Namespaces do not require a special declaration.

### Comments

Comments are the same as in C++
```
/* 
This is
a comment 
*/

// This is one line comment
```

### Library usage

You can use TL-B libraries to extend your documents and to avoid writing repetitive schemes. We have prepared a set of ready-made libraries that you can use. They are mostly based on block.tlb, but we have also added some combinators of our own.

- `tonstdlib.tlb`
- `tonextlib.tlb`
- `hashmap.tlb`

In TL-B libraries there is no concept of cyclic import. Just indicate the dependency on some other document (library) at the top of the document with the keyword `dependson`. For example:

file `mydoc.tlb`:
```c++
//
// dependson "libraries/tonstdlib.tlb"
//

op:uint32 data:Any = MsgBody;
something$0101 data:(Maybe ^MsgBody) = SomethingImportant;
```

In dependencies, you are required to specify the correct relative path. The example above is located in such a tree:

```
.
├── mydoc.tlb
├── libraries
│   ├── ...
│   └── tonstdlib.tlb
└── ...
```

### Useful sources

- [Telegram Open Network Virtual Machine](https://newton-blockchain.github.io/docs/tvm.pdf)
- [A description of an older version of TL](https://core.telegram.org/mtproto/TL)
- [block.tlb](https://github.com/newton-blockchain/ton/blob/master/crypto/block/block.tlb)
