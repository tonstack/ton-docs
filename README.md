## TL-B Language

TL-B (Type Language - Binary) serves to describe the type system, constructors, and existing functions. For example we can use TL-B schemes to build binary structures associated with the TON blockchain.

## Overview

We refer to any set of TL-B constructs as TL-B documents. A TL-B document usually consists of two sections, which are separated by the `---functions---` keyword. The first section consists of declarations of built-in types and aggregate types (i.e. their constructors). The second section consists of the declared functions, i.e. functional combinators. We may not follow the separation principle in small documents with no more than one functional combinator.

The first and second sections consist of combinator declarations, each ending with a semicolon(`;`). However, the first section contains only constructors, while the second section only involves functions. Each combinator is declared using a “combinator declaration” in the format explained above. However, the combinator number and field names may be explicitly assigned.

If additional type declarations are required after functions have been declared, the keyword (section divider)  
`---types---` is used. Furthermore, a functional combinator may be declared in the type section if its result type begins with an exclamation point (in fact, when the function section is interpreted, this exclamation point is added automatically).

To explicitly define 32-bit names of combinators, a hash mark (`#`) is added immediately after the combinator’s name, followed by 8 hexadecimal digits. In this documentation we will look at an example of how to correctly define a 32-bit combinator name for an internal message in the TON blockchain.

### Comments

Comments are the same as in C++
```
/* 
This is
a comment 
*/

// This is one line comment
```

### Useful sources

- [Telegram Open Network Virtual Machine](https://newton-blockchain.github.io/docs/tvm.pdf)
- [A description of an older version of TL](https://core.telegram.org/mtproto/TL)
- [block.tlb](https://github.com/newton-blockchain/ton/blob/master/crypto/block/block.tlb)
