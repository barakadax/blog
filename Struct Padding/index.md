# Memory Alignment and Struct Padding
- [Struct Alignment Mechanics](#struct-alignment-mechanics)
- [Cache Line Density and False Sharing](#cache-line-density-and-false-sharing)
- [Compiler Directives and Hazards](#compiler-directives-and-hazards)
- [Sources](#sources)

---

## Struct Alignment Mechanics

Struct alignment equals the maximum alignment of its scalar members.
Field offsets must be divisible by member alignment, and trailing padding pads the entire struct to a multiple of its overall alignment (guaranteeing array element alignment).

### Memory Layout Comparison

```text
Struct A: struct { char a; int b; char c; } -> 12 Bytes (align 4)
[ Offset: 0  | 1       2       3 | 4       5       6       7 | 8  | 9      10      11 ]
[ a (1B)     | Padding (3B)      | b: int (4B)               | c  | Trailing Pad (3B) ]

Struct B: struct { char a; char c; int b; } -> 8 Bytes (align 4)
[ Offset: 0  | 1       | 2       3       | 4       5       6       7 ]
[ a (1B)     | c (1B)  | Padding (2B)    | b: int (4B)               ]
```

- **Struct A**: `char (1B) + padding (3B) + int (4B) + char (1B) + padding (3B) = 12 Bytes` (due to total struct alignment matching largest member).
- **Struct B**: `char (1B) + char (1B) + padding (2B) + int (4B) = 8 Bytes`.

---

## Cache Line Density and False Sharing

CPU L1/L2/L3 caches operate strictly on **64-byte cache lines**:

1. **Array Packing Density**:
   - Array of `Struct A` (12B): Holds at most 5 elements per 64-byte cache line (60B used, 4B split across lines).
   - Array of `Struct B` (8B): Holds exactly 8 elements per 64-byte cache line (100% line saturation, zero split reads).
2. **False Sharing**:
   When multiple cores write to independent fields located inside the same 64-byte line, cache coherency protocols (MESI/MOESI) repeatedly invalidate the entire line across L1 caches, destroying multi-threaded scalability, mitigate via explicit alignment: `alignas(64)` in C/C++ or `#[repr(align(64))]` in Rust.

---

## Compiler Directives and Hazards

Compilers enforce or disable padding via attributes:

```c
// C: Eliminate padding
#pragma pack(push, 1)
struct PackedC { char a; int b; char c; }; // sizeof = 6
#pragma pack(pop)
// GCC/Clang alternative: struct __attribute__((packed)) PackedC { ... };
```

```rust
// Rust: Default layout can reorder fields (repr(Rust)).
// repr(C) guarantees declaration order; repr(packed) forces 1-byte alignment.
#[repr(C)]
struct OrderMatters { a: u8, b: u32, c: u8 } // 12 bytes

#[repr(C, packed)]
struct PackedRust { a: u8, b: u32, c: u8 }   // 6 bytes
```

### Hazards of Unaligned Pointer Dereferencing

Taking references to unaligned packed fields (`&packed.b`) generates undefined behavior (UB) in Rust and triggers hardware traps or severe instruction pipeline stalls in C.
Always copy packed fields by value (`std::ptr::read_unaligned` in Rust or `memcpy` in C) rather than creating raw references or pointers.

---

## Sources

- [Intel 64 and IA-32 Architectures Software Developer's Manual (Volume 1, Chapter 4: Data Types and Memory Alignment)](https://www.intel.com/content/www/us/en/developer/articles/technical/intel-sdm.html)
- [The Rust Reference: Type Layout and repr Attributes](https://doc.rust-lang.org/reference/type-layout.html)
- [C11 Standard (ISO/IEC 9899:2011) - Alignment specification](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
- [Ulrich Drepper - What Every Programmer Should Know About Memory (Red Hat)](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)
