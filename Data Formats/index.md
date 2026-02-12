# Data Formats
- [What is a Data Format?](#what-is-a-data-format?)
- [JSON](#json)
- [YAML](#yaml)
- [Protobuf](#protobuf)
- [Parquet](#parquet)
- [CSV](#csv)
- [XML](#xml)
- [TOON](#toon)
- [TOML](#toml)
- [Avro](#avro)
- [MessagePack](#messagepack)
- [BSON](#bson)
- [CBOR](#cbor)
- [ORC](#orc)
- [FlatBuffers](#flatbuffers)
- [Arrow](#arrow)
- [Pickle](#pickle)
- [Summary](#summary)
- [Sources](#sources)

## What are Data Formats?

Data formats are the standardized rules for structuring information so different systems can exchange or store it accurately,
They act as a "language contract" between a sender and a receiver or storage system,
Ensuring that data is encoded into a format that can later be reconstructed into its original form.

## JSON

- **Created:** 2001 by Douglas Crockford.
- **Ownership & License:** Public Domain (standardized by ECMA and ISO). The original "JSON License" had a "do no evil" clause, but modern standards are open and non-restrictive.
- **Is it type safe:** No (it's dynamically typed; requires JSON Schema for validation).
- **Is it serializable:** Yes.
- **Human Readable:** Yes.
- **Format:** Text.
- **Structure:** Hierarchical (Objects and Arrays).
- **Best Use Case:** Web APIs, configuration files, and data exchange between frontend and backend.
- **Example:**
```json
{
  "name": "John Doe",
  "age": 30,
  "isEmployee": true
}
```
- **Security Risk:** JSON Injection, Cross-Site Request Forgery (CSRF) if not handled correctly, and insecure deserialization.
- **Variants:** Variants exist because the original JSON specification is intentionally strict, while that strictness makes JSON great for sending data between computers, it makes it frustrating for certain human tasks or high-performance tasks, examples of variants: JSONC, JSONL, JSON5 and HJSON.

## YAML

- **Created:** 2001 by Clark Evans, Ingy döt Net, and Oren Ben-Kiki.
- **Ownership & License:** Open standard (YAML.org).
- **Is it type safe:** No (dynamically typed; supports tags for explicit typing).
- **Is it serializable:** Yes.
- **Human Readable:** Yes (designed to be very readable).
- **Format:** Text.
- **Structure:** Hierarchical (using indentation).
- **Best Use Case:** Configuration files (e.g., Docker, Kubernetes, CI/CD pipelines).
- **Example:**
```yaml
name: John Doe
age: 30
isEmployee: true
```
- **Security Risk:** Arbitrary code execution during deserialization (especially with complex tags/anchors) if using unsafe loaders.

## Protobuf

- **Created:** 2001 (internal at Google), released publicly in 2008.
- **Ownership & License:** Google; BSD-3-Clause license.
- **Is it type safe:** Yes (requires a `.proto` schema and generates code).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format; requires a tool to decode).
- **Format:** Binary.
- **Structure:** Schema-based (Hierarchical).
- **Best Use Case:** High-performance microservices communication (gRPC), internal data storage.
- **Example:**
```protobuf
message Person {
  string name = 1;
  int32 age = 2;
  bool is_employee = 3;
}
```
- **Security Risk:** Buffer overflows in older parsers; potential for resource exhaustion (Denial of Service) if not carefully constrained during decoding.

## Parquet

- **Created:** 2013 by Twitter and Cloudera.
- **Ownership & License:** Apache Software Foundation; Apache License 2.0.
- **Is it type safe:** Yes (schema is embedded in the file metadata).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Columnar.
- **Best Use Case:** Big data processing and analytical queries (e.g., Spark, Hive, Presto) due to efficient columnar storage and compression.
- **Example:** (Binary format representation)
```
[Header]
[Record Batch 1: Column A, Column B...]
[Record Batch 2: Column A, Column B...]
[Footer with Metadata]
```
- **Security Risk:** Potential for malicious metadata triggering vulnerabilities in parsers; data exposure if encryption (Parquet Modular Encryption) is not used.

## CSV

- **Created:** 1972 (IBM Fortran), standardized in 2005 (RFC 4180).
- **Ownership & License:** Public Domain / Open Standard.
- **Is it type safe:** No (everything is a string; requires manual parsing).
- **Is it serializable:** Yes.
- **Human Readable:** Yes.
- **Format:** Text.
- **Structure:** Tabular / Row-based.
- **Best Use Case:** Simple data exchange, spreadsheets, and legacy system integration.
- **Example:**
```csv
name,age,isEmployee
John Doe,30,true
```
- **Security Risk:** CSV Injection (Excel Formula Injection) where malicious strings can execute code when opened in spreadsheet software.

## XML

- **Created:** 1996 by the W3C.
- **Ownership & License:** W3C Open Standard.
- **Is it type safe:** No (requires XSD or DTD for validation).
- **Is it serializable:** Yes.
- **Human Readable:** Yes (though verbose).
- **Format:** Text.
- **Structure:** Hierarchical (Tree-based).
- **Best Use Case:** Documents, configuration, and legacy web services (SOAP).
- **Example:**
```xml
<person>
  <name>John Doe</name>
  <age>30</age>
  <isEmployee>true</isEmployee>
</person>
```
- **Security Risk:** XML External Entity (XXE) attacks, XML Bomb (Billion Laughs attack), and insecure transformation (XSLT).

## TOON

- **Created:** 2023 by the TOON project.
- **Ownership & License:** Open source (MIT/Apache 2.0).
- **Is it type safe:** Yes (statically typed).
- **Is it serializable:** Yes.
- **Human Readable:** Yes.
- **Format:** Text (but optimized for performance).
- **Structure:** Hierarchical / Object-oriented.
- **Best Use Case:** High-performance configuration and data modeling where type safety is critical.
- **Example:**
```toon
Person {
  name: "John Doe"
  age: 30
  isEmployee: true
}
```
- **Security Risk:** Relatively new, so potential undiscovered parser vulnerabilities; typical deserialization risks if types are not constrained.

## TOML

- **Created:** 2013 by Tom Preston-Werner (co-founder of GitHub).
- **Ownership & License:** Open Standard; MIT License.
- **Is it type safe:** No (dynamically typed, but supports explicit types like integers, floats, booleans, dates).
- **Is it serializable:** Yes.
- **Human Readable:** Yes (designed to be "minimal" and easy to read).
- **Format:** Text.
- **Structure:** Hierarchical (using headings and key-value pairs).
- **Best Use Case:** Configuration files for projects (e.g., Cargo for Rust, pyproject.toml for Python).
- **Example:**
```toml
name = "John Doe"
age = 30
isEmployee = true
```
- **Security Risk:** Parser vulnerabilities (e.g., resource exhaustion via deeply nested tables) or "TOML Injection" if data is not correctly escaped when written.

## Avro

- **Created:** 2009 by Doug Cutting (creator of Hadoop).
- **Ownership & License:** Apache Software Foundation; Apache License 2.0.
- **Is it type safe:** Yes (schema-based; schema is stored with the data).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Row-based (but supports sophisticated schemas).
- **Best Use Case:** Big data pipelines (Apache Kafka) where data schemas evolve over time.
- **Example:** (JSON representation of the schema)
```json
{
  "type": "record",
  "name": "Person",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "age", "type": "int"}
  ]
}
```
- **Security Risk:** Deserialization of untrusted data; schema mismatch vulnerabilities if the reader and writer schemas are inconsistent.

## MessagePack

- **Created:** 2008 by Sadayuki Furuhashi.
- **Ownership & License:** Open source; Apache License 2.0.
- **Is it type safe:** No (dynamically typed like JSON, but more compact).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Hierarchical (Objects and Arrays).
- **Best Use Case:** Compact data exchange in web applications or IoT when JSON is too large.
- **Example:** (Hexadecimal representation)
```
83 a4 6e 61 6d 65 a8 4a 6f 68 6e 20 44 6f 65 ...
```
- **Security Risk:** Buffer overflows in decoders; resource exhaustion if processing maliciously crafted messages.

## BSON

- **Created:** 2009 by 10gen (now MongoDB, Inc.).
- **Ownership & License:** MongoDB, Inc.; Apache License 2.0.
- **Is it type safe:** Partially (supports more types than JSON, like Date and Binary).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Hierarchical (Objects and Arrays).
- **Best Use Case:** Internal data storage for MongoDB and applications needing rapid traversal.
- **Example:** (Binary representation)
```
\x16\x00\x00\x00\x02hello\x00\x06\x00\x00\x00world\x00\x00
```
- **Security Risk:** Similar to JSON; vulnerabilities in the BSON parser can lead to remote code execution or denial of service.

## CBOR

- **Created:** 2013 by Carsten Bormann.
- **Ownership & License:** IETF Standard (RFC 8949); Open Standard.
- **Is it type safe:** No (dynamically typed, but more robust than JSON).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Hierarchical (Objects and Arrays).
- **Best Use Case:** IoT and constrained environments where low overhead is essential.
- **Example:** (Hexadecimal representation)
```
bf 63 46 6f 6f 63 42 61 72 ff
```
- **Security Risk:** Vulnerabilities in decoding logic; potential for resource exhaustion if cyclic references are supported and not handled.

## ORC

- **Created:** 2013 by Hortonworks (now Cloudera).
- **Ownership & License:** Apache Software Foundation; Apache License 2.0.
- **Is it type safe:** Yes (schema is embedded in the file).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Columnar.
- **Best Use Case:** Highly optimized storage and analysis for Hive and big data stacks.
- **Example:** (Binary structure representation)
```
[Postscript]
[Stripe 1]
[Stripe 2]
[File Footer]
```
- **Security Risk:** Similar to Parquet; vulnerabilities in the complex reader logic or metadata handling.

## FlatBuffers

- **Created:** 2014 by Wouter van Oortmerssen (Google).
- **Ownership & License:** Google; Apache License 2.0.
- **Is it type safe:** Yes (schema-based).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary (optimized for zero-copy).
- **Structure:** Schema-based (Hierarchical).
- **Best Use Case:** High-performance games and low-latency applications where memory access speed is critical.
- **Example:** (Hexadecimal representation of a buffer)
```
0c 00 00 00 08 00 0c 00 04 00 08 00 08 00 00 00 ...
```
- **Security Risk:** Maliciously crafted buffers causing out-of-bounds reads if validation is skipped (though encouraged by design).

## Arrow

- **Created:** 2016 by the Apache Arrow community.
- **Ownership & License:** Apache Software Foundation; Apache License 2.0.
- **Is it type safe:** Yes (schema-based).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary (In-memory columnar format).
- **Structure:** Columnar.
- **Best Use Case:** In-memory analytics and cross-language data sharing without serialization overhead.
- **Example:** (In-memory layout representation)
```
[Validity Bitmap]
[Offsets Buffer]
[Data Buffer]
```
- **Security Risk:** Memory safety issues if the underlying buffers are manipulated or if the memory management logic has bugs.

## Pickle

- **Created:** 1994 by Jim Fulton (part of Python).
- **Ownership & License:** Python Software Foundation; PSF License (Open Source).
- **Is it type safe:** No (Python-specific; handles arbitrary objects).
- **Is it serializable:** Yes.
- **Human Readable:** No (binary format).
- **Format:** Binary.
- **Structure:** Hierarchical / Object-oriented.
- **Best Use Case:** Serializing Python objects for persistence or IPC between Python processes.
- **Example:** (Binary representation)
```python
import pickle
data = {'name': 'John', 'age': 30}
pickle.dumps(data)
# Output: b'\x80\x04\x95\x1e\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x04name\x94\x8c\x04John\x94\x8c\x03age\x94K\x1eu.'
```
- **Security Risk:** **Extremely high.** Unpickling untrusted data can lead to arbitrary code execution. Never unpickle data from untrusted sources.

## Summary

Data formats vary based on their focus: **human readability**, **performance/efficiency** and **large-scale analytics**,
Choosing the right format depends on the specific trade-offs between speed, size, and ease of debugging.

### Comparison Table

| Format | Readability | Complexity Perf | Scale Perf | Memory Usage |
| :--- | :--- | :--- | :--- | :--- |
| **JSON** | High | Medium | Low | Medium |
| **YAML** | High | Medium | Low | High |
| **Protobuf** | Low | High | High | Low |
| **Parquet** | Low | Medium | High | Medium |
| **CSV** | High | Low | Medium | Low |
| **XML** | Medium | Medium | Low | High |
| **TOON** | High | High | Medium | Medium |
| **TOML** | High | Medium | Low | Medium |
| **Avro** | Low | High | High | Low |
| **MessagePack** | Low | High | Medium | Low |
| **BSON** | Low | Medium | Medium | Medium |
| **CBOR** | Low | High | Medium | Low |
| **ORC** | Low | Medium | High | Medium |
| **FlatBuffers** | Low | High | High | Very Low |
| **Arrow** | Low | High | High | Low (Zero-copy) |
| **Pickle** | Low | High | Medium | Medium |

## Sources

- [JSON](https://www.json.org/)
- [YAML](https://yaml.org/)
- [Protobuf](https://protobuf.dev/)
- [Parquet](https://parquet.apache.org/)
- [CSV](https://www.rfc-editor.org/rfc/rfc4180)
- [XML](https://www.w3.org/XML/)
- [TOON](https://toon.dev/)
- [TOML](https://toml.io/)
- [Avro](https://avro.apache.org/)
- [MessagePack](https://msgpack.org/)
- [BSON](https://bsonspec.org/)
- [CBOR](https://cbor.io/)
- [FlatBuffers](https://google.github.io/flatbuffers/)
- [Arrow](https://arrow.apache.org/)
- [Pickle](https://docs.python.org/3/library/pickle.html)
- [Clean code](http://cleancoder.com/products)
- [Standards organizations](https://barakadax.github.io/blog?article=Standards%20organizations)
