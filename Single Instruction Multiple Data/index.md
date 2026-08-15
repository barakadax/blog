# Single Instruction Multiple Data
- [What is SIMD](#what-is-simd)
- [Where to use it](#where-to-use-it)
- [Code examples](#code-examples)
- [Sources](#sources)

## What is SIMD

**Single Instruction, Multiple Data (SIMD)** is a hardware-level execution model that applies a single instruction to multiple pieces of data simultaneously in a single clock cycle,
First conceptualized in 1958 with the Lincoln Laboratory TX-2,
Then coming to consumer markets in 1994 PA-RISC 1.1,
vector operations are no longer exclusive to CPUs—they form the core foundation of GPUs, DSPs, and AI accelerators.

```text
Scalar: [ A0 ] + [ B0 ] ---> [ C0 ]  (Executed sequentially over multiple cycles)

SIMD: [ A0 | A1 | A2 | A3 ]
                +           ---> [ C0 | C1 | C2 | C3 ]  (Executed in 1 cycle)
      [ B0 | B1 | B2 | B3 ]
```

## Where to use it

SIMD excels at uniform, bulk-data operations where long streams of numbers undergo the exact same mathematical transformation:
 - **Audio & Video**: Processing pixels, image filters, video decoding.
 - **Game Engines & 3D**: Vector/matrix math, physics engines, vertex transformations.
 - **Data Science & ML**: Matrix multiplication, neural network inference, NumPy array math.
 - **Cryptography & Hashing**: Bulk data encryption/decryption routines.

## Code examples

> [!NOTE]
> Memory Bounds: Arrays A and B must match in size. Destination array C must be pre-allocated before running SIMD operations.
> Tail Cleanup: When array size (count) is not an exact multiple of the vector size (e.g., 9 elements), process full chunks with SIMD and use a scalar loop for the leftover tail.
> Python does not support this by default, Numpy overload the plus operator and run C/C++ code

C/C++:
```C
#include <immintrin.h>

void add_simd(float* A, float* B, float* C, int count) {
    for (int i = 0; i <= count - 8; i += 8) {
        __m256 a_vec = _mm256_loadu_ps(&A[i]); // Load 8 floats
        __m256 b_vec = _mm256_loadu_ps(&B[i]); // Load 8 floats
        __m256 c_vec = _mm256_add_ps(a_vec, b_vec); // 1 CPU instruction adds all 8 pairs
        _mm256_storeu_ps(&C[i], c_vec);        // Store 8 floats
    }

    // Scalar cleanup loop for leftover tail elements
    for (; i < count; i++) {
        C[i] = A[i] + B[i];
    }
}
/*
 * Register Bit Widths (Single-Precision Floats):
 * 128-bit (SSE): 4 floats per cycle (__m128)
 * 256-bit (AVX/AVX2): 8 floats per cycle (__m256)
 * 512-bit (AVX-512): 16 floats per cycle (__m512)
 *
 * Higher throughput (32/64 floats) is achieved via loop unrolling across multiple vector registers:
 * 4 times 256-bit (AVX/AVX2): 32 floats per cycle (__m256)
 * 4 times 512-bit (AVX-512): 64 floats per cycle (__m512)
*/
```

Rust:
```rust
use std::arch::x86_64::*;

pub unsafe fn add_simd_rust(a: &[f32], b: &[f32], c: &mut [f32]) {
    let mut i = 0;
    let len = a.len().min(b.len()).min(c.len());

    while i <= len.saturating_sub(8) {
        let a_vec = _mm256_loadu_ps(a.as_ptr().add(i));
        let b_vec = _mm256_loadu_ps(b.as_ptr().add(i));
        let c_vec = _mm256_add_ps(a_vec, b_vec);
        _mm256_storeu_ps(c.as_mut_ptr().add(i), c_vec);
        i += 8;
    }

    while i < len {
        c[i] = a[i] + b[i];
        i += 1;
    }
}

// At the time of writing this Rust has a safe version in nightly using:
// #![feature(portable_simd)]
// use std::simd::prelude::*;
```

C#:
```csharp
using System;
using System.Numerics;

public static void AddSimdCSharp(ReadOnlySpan<float> a, ReadOnlySpan<float> b, Span<float> c) {
    int vectorSize = Vector<float>.Count; // Auto-selects width (e.g., 8 for AVX2, 4 for SSE2)
    int i = 0;
    int minLen = Math.Min(a.Length, Math.Min(b.Length, c.Length));

    for (; i <= minLen - vectorSize; i += vectorSize) {
        var aVec = new Vector<float>(a.Slice(i));
        var bVec = new Vector<float>(b.Slice(i));
        var cVec = aVec + bVec; // Overloaded '+' maps directly to target hardware SIMD
        cVec.CopyTo(c.Slice(i));
    }

    // Scalar cleanup for remaining elements
    for (; i < minLen; i++) {
        c[i] = a[i] + b[i];
    }
}
```

## Sources

- [TX-2](https://en.wikipedia.org/wiki/TX-2)
- [PA-RISC 1.1](https://en.wikipedia.org/wiki/PA-RISC)
- [immintrin.h](https://www.intel.com/content/www/us/en/docs/intrinsics-guide/index.html#text=immintrin.h)
- [C# Vector<T>](https://learn.microsoft.com/en-us/dotnet/api/system.numerics.vector-1?view=net-10.0)
