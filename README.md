### TL-B Language

TL-B (Type Language - Binary) serves to describe the type system, constructors, and existing functions. For example we can use TL-B schemes to build binary structures associated with the TON blockchain. Special TL-B parsers can read schemes to deserialize binary data into different objects.

### Table of contents
- [TL-B Language](#tl-b-language)
- [Table of contents](#table-of-contents)
- [Overview](#overview)
- [Constructors](#constructors)
- [Field definitions](#field-definitions)
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
<img alt="tlb structure" src="img/tlb.drawio.svg" width="100%">


### Constructors

Constructors are used to specify the type of combinator, including the state at serialization. For example constructors can also be used when you want to specify an `op` in query to a smart contract in the TON.

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
| `some$`               | serialize crc32 of string `some`      |
| `some#_` or `some$_`  | serialize nothing                     |


Tags may be given in either binary (after a dollar sign) or hexadecimal notation (after a hash sign). If a tag is not explicitly provided, TL-B parser must computes a default 32-bit constructor tag by hashing with crc32 algorithm the text of the “equation” defining this constructor in a certain fashion. Therefore, empty tags must be explicitly provided by `#_` or `$_`. 

All constructor names must be distinct, and constructor tags for the same type must constitute a prefix code (otherwise the deserialization would not be unique).

This is an example from the [TonToken](https://github.com/akifoq/TonToken/blob/master/func/token/scheme.tlb) repository that shows  
us how to implement an internal message TL-B scheme:

```c++
extra#_ amount:Grams = Extra;

addr_std$10 anycast:(## 1) {anycast = 0}
      workchain_id:int8 address:bits256 = MsgAddrSmpl;

transfer#4034a3c0 query_id:uint64
    reciever:MsgAddrSmpl amount:Extra body:Any = Request;
```

In this example `transfer#4034a3c0` will be serialized as a 32 bit unsigned integer from hex value after hash sign(`#`). This meets the standard declaration of an `op` in the [Smart contract guidelines](https://ton.org/docs/#/howto/smart-contract-guidelines).

To meet the standard described in paragraph 5 of the [Smart contract guidelines](https://ton.org/docs/#/howto/smart-contract-guidelines), it is not enough for us to calculate the crc32. You can follow the following examples to define an `op` in requests or responses from smart contracts in a TL-B scheme:

```python
import binascii


def main():
    req_text = "some_request"
    req = hex(binascii.crc32(bytes(req_text, "utf-8")) & 0x7fffffff)
    print(f"{req_text}#{req} = Request;")  # some_request#0x733d0d35 = Request;

    rsp_text = "some_response"
    rsp = hex(binascii.crc32(bytes(rsp_text, "utf-8")) | 0x80000000)
    print(f"{rsp_text}#{rsp} = Response;")  # some_response#0x88b0eb8f = Response;


if __name__ == "__main__":
    main()
```


### Field definitions

The constructor and its optional tag are followed by field definitions. Each field definition is of the form `ident:type-expr`, where ident is an identifier with the name of the field16 (replaced by an underscore for anonymous fields), and type-expr is the field’s type. The type provided here is a type expression, which may include simple types or parametrized types with suitable parameters. Variables — i.e., the (identifiers of the) previously defined fields of types `#` (natural numbers) or Type (type of types) — may be used as parameters for the parametrized types. The serialization process recursively serializes each field according to its type, and the serialization of a value ultimately consists of the concatenation of bitstrings representing the constructor (i.e., the constructor tag) and the field values.

Some fields may be implicit. Their definitions are surrounded by curly
braces(`{`, `}`), which indicate that the field is not actually present in the serialization, but that its value must be deduced from other data (usually the parameters of the type being serialized). Example:

```c++
nothing$0 {X:Type} = Maybe X;
just$1 {X:Type} value:X = Maybe X;
```

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
