## Library usage

You can use TL-B libraries to extend your documents and to avoid writing repetitive schemes. We have prepared a set of ready-made libraries that you can use. They are mostly based on block.tlb, but we have also added some combinators of our own.

- `tonstdlib.tlb`
- `tonextlib.tlb`
- `hashmap.tlb`

In TL-B libraries there is no concept of cyclic import. Just indicate the dependency on some other document (library) at the top of the document with the keyword `dependson`. For example:

file `mydoc.tlb`:
```c
//
// `dependson "libraries/tonstdlib.tlb"`
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

