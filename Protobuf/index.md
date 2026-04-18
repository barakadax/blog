# Protobuf
- [What is Protobuf](#what-is-protobuf)
- [How does it work](#how-does-it-work)
  - [Define a schema](#define-a-schema)
  - [Nesting and composition](#nesting-and-composition)
  - [Compile](#compile)
  - [Serialize and deserialize](#serialize-and-deserialize)
  - [Wire format](#wire-format)
- [Where is it used](#where-is-it-used)
- [Compare](#compare)
- [Sources](#sources)

## What is Protobuf

Protocol Buffers (Protobuf) is a **language-neutral, platform-neutral binary serialization format** developed by Google.
Data structures are defined in a `.proto` schema file, which is compiled into code for any supported language.
Released publicly in 2008 under a BSD-3-Clause license; the current standard is **proto3**.

Unlike text formats, Protobuf encodes data in a compact binary representation — fields are identified by **numbers**, not names, making the wire format smaller and faster to parse than JSON or XML.

## How does it work

### Define a schema

All data structures are declared in `.proto` files using the `message` keyword.
Each field has a **scalar type**, a **name**, and a unique **field number** (the number is what appears in the binary output, not the name).

```shell
syntax = "proto3";

message Person {
  string name    = 1;
  int32  age     = 2;
  bool   active  = 3;
  repeated string emails = 4;
}
```

Supported scalar types: `int32`, `int64`, `uint32`, `uint64`, `sint32`, `sint64`, `float`, `double`, `bool`, `string`, `bytes`.
Fields can also be nested `message` types, `enum`, `map<K, V>`, or wrapped in `oneof` (at most one field set at a time).

> [!NOTE]
> Field numbers must never be reused or changed once data is in production — they are the only link between the schema and the binary data.
> Deleting a field requires reserving its number to prevent accidental reuse: `reserved 2;`

### Nesting and composition

Messages can contain other messages as field types (**composition**) or be defined inside other messages (**nesting**):

```shell
message Address {
  string street = 1;
  string city   = 2;
}

message Company {
  string name = 1;

  message Department {             // nested — scoped as Company.Department
    string           title   = 1;
    repeated string  members = 2;
  }

  Address             address     = 2;  // composition
  repeated Department departments = 3;
}
```

Nested messages are scoped to their parent (`Company.Department`) but otherwise behave identically to top-level messages.

**Protobuf has no inheritance.** The idiomatic substitute is `oneof`, which lets a field hold exactly one of several message types — a discriminated union:

```shell
message Shape {
  oneof kind {
    Circle    circle    = 1;
    Rectangle rectangle = 2;
    Triangle  triangle  = 3;
  }
}

message Circle    { float radius = 1; }
message Rectangle { float width  = 1; float height = 2; }
message Triangle  { float base   = 1; float height = 2; }
```

The reader checks which `oneof` field is populated and handles each case explicitly — all possible types are visible directly in the schema.

### Compile

The `.proto` file is compiled with **`protoc`** (the Protocol Buffer Compiler) to generate source code in your target language.

```shell
protoc --python_out=. person.proto
# generates person_pb2.py
```

Official code generators exist for `C++`, `Java`, `Python`, `Go`, `C#`, `Ruby`, `Kotlin`, `Dart`, and more.
Third-party plugins cover additional languages.

### Serialize and deserialize

```python
from person_pb2 import Person

person = Person(name="John Doe", age=30, active=True)

serialized = person.SerializeToString()
# b'\n\x08John Doe\x10\x1e\x18\x01'  →  14 bytes

restored = Person()
restored.ParseFromString(serialized)
print(restored.name)  # John Doe
```

The equivalent JSON (`{"name":"John Doe","age":30,"active":true}`) is 42 bytes — exactly **3× larger**.

### Wire format

Each field is encoded as a **tag–value pair**:

```
tag = (field_number << 3) | wire_type
```

| Wire type | Value | Used for |
| :--- | :---: | :--- |
| Varint | 0 | `int32`, `int64`, `bool`, `enum` |
| 64-bit | 1 | `fixed64`, `double` |
| Length-delimited | 2 | `string`, `bytes`, nested messages, repeated |
| 32-bit | 5 | `fixed32`, `float` |

![Binary breakdown](binary.png)

```
0a 08 4a 6f 68 6e 20 44 6f 65 10 1e 18 01
│  │  └──────────────────────┘  │  │  │  └─ active = true
│  │       "John Doe" (8 B)     │  │  └──── field 3, varint
│  └─ length = 8                │  └─────── age = 30
└─── field 1, length-delimited  └────────── field 2, varint
```

Integers use **varint encoding** — small values (0–127) occupy just 1 byte regardless of the declared type (`int32` or `int64`).
Unknown fields (tags the reader's schema doesn't recognize) are preserved, enabling **forward compatibility**.

## Where is it used

- **gRPC**: Protobuf is the default serialization layer for Google's RPC framework, used widely in microservices.
- **Kubernetes**: API machinery and etcd storage use Protobuf for internal serialization.
- **Mobile and IoT**: Compact size and fast parsing make it practical on bandwidth- or CPU-constrained devices.
- **Big data**: Used as a storage and transport format in pipelines alongside Avro and Parquet.

## Compare

| Feature | Protobuf | JSON |
| :--- | :--- | :--- |
| **Format** | Binary | Text |
| **Human readable** | No | Yes |
| **Schema required** | Yes (`.proto`) | Optional |
| **Type safety** | Yes | No |
| **Schema evolution** | Yes (field numbers) | Manual |
| **Code generation** | Yes (`protoc`) | No |
| **Size** | Small | Large |
| **Performance** | High | Medium |
| **Language support** | Wide | Universal |

Protobuf's main trade-off is **debuggability** — you cannot read a serialized payload without the matching `.proto` schema and a decoder tool (`protoc --decode` or tools like `grpcurl`).
JSON's human readability makes it preferable for public APIs and configuration; Protobuf is preferable for high-throughput internal communication.

## Sources

- [Wikipedia](https://en.wikipedia.org/wiki/Protocol_Buffers)
- [Protobuf dev pages](https://protobuf.dev/)
- [Protobuf GitHub repo](https://github.com/protocolbuffers/protobuf)
