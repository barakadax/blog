# Parquet Data Format
- [Physical Layout Hierarchy](#physical-layout-hierarchy)
- [Encoding Techniques & Dremel Record Shredding](#encoding-techniques--dremel-record-shredding)
- [Query Engine Optimizations](#query-engine-optimizations)
- [Row-Based vs. Columnar Storage](#row-based-vs-columnar-storage)
- [Working with Parquet in C# (ParquetSharp's 4 APIs)](#working-with-parquet-in-c-parquetsharps-4-apis)
- [Sources](#sources)

Parquet is an open-source columnar storage format engineered for analytical (OLAP) processing.
It decouples logical record structure from physical disk layout to maximize I/O throughput, SIMD vectorization, and data compression.

## Physical Layout Hierarchy

Parquet files follow a strict nested physical structure:

```text
+-------------------------------------------------------------------------+
| File: 4-byte Magic Number ("PAR1")                                      |
|  +--------------------------------------------------------------------+ |
|  | Row Group (horizontal partition, typically 128MB - 512MB)          | |
|  |  +---------------------------------+  +--------------------------+ | |
|  |  | Column Chunk (Column A)         |  | Column Chunk (Column B)  | | |
|  |  |  +----------------------------+ |  |  +---------------------+ | | |
|  |  |  | Dictionary Page (optional) | |  |  | Data Page           | | | |
|  |  |  | Data Page (Values + D/R)   | |  |  | Data Page           | | | |
|  |  |  +----------------------------+ |  |  +---------------------+ | | |
|  |  +---------------------------------+  +--------------------------+ | |
|  +--------------------------------------------------------------------+ |
| File Metadata Footer (Schema, Row Group metadata, Column min/max stats) |
| 4-byte Footer Length | 4-byte Magic Number ("PAR1")                     |
+-------------------------------------------------------------------------+
```

Readers parse Parquet files via an initial **reverse seek**: they inspect the trailing 8 bytes (`4-byte footer length + 4-byte magic PAR1`), seek backward to deserialize the Thrift `FileMetaData` footer, and resolve schema, row-group byte offsets, and column statistics upfront before issuing targeted, parallel range requests.

---

## Encoding Techniques & Dremel Record Shredding

Column chunks leverage type homogeneity and locality using column-level encodings before generic block compression (Snappy/ZSTD):
- **Dictionary Encoding**: Replaces distinct values with compact dictionary indices; falls back to plain encoding if cardinality exceeds limits.
- **RLE & Bit-Packing**: Compresses consecutive repeated runs and stores integers in minimum required bit widths (e.g., 2 bits for $[0, 3]$).

### Dremel Repetition & Definition Levels
Parquet shreds nested and nullable schemas into two auxiliary integer streams (Google Dremel) without storing null values or record markers:
- **Definition Level ($D$)**: Specifies how many optional ancestors are defined. $D = \text{max}$ indicates a present value; $D < \text{max}$ identifies the null ancestor level.
- **Repetition Level ($R$)**: Specifies the schema depth where a repeated field recurs. $R = 0$ marks a new top-level record; $R > 0$ continues a nested list.

$D$ and $R$ streams are bit-packed and RLE-compressed alongside values for zero-overhead lossless reconstruction.

---

## Query Engine Optimizations

Modern analytical engines (DuckDB, Trino, Spark, ClickHouse) utilize Parquet's internals for two primary performance gains:

1. **Predicate Push down & Block Pruning**: Footers and page indexes store scalar statistics (min/max, null counts, bloom filters) per Row Group and Page. Planners evaluate query `WHERE` predicates against these bounds and skip non-matching Row Groups entirely without reading them off disk or object storage (S3/GCS).
2. **Vectorized SIMD Execution**: Homogeneous data pages unpack into contiguous columnar memory arrays, CPUs stream these unrolled vectors directly into L1/L2 caches, allowing SIMD vector lanes to evaluate operations across 1024+ records per clock cycle without branching.

---

## Row-Based vs. Columnar Storage

Row formats store entire records contiguously, while columnar formats isolate field data:

| Dimension | Row-Based (CSV / JSON) | Row-Oriented Binary (Avro) | Columnar (Parquet) |
| :--- | :--- | :--- | :--- |
| **Projection Cost** | $O(N \times M)$ scan + text parse | $O(N \times M)$ deserialize full record | $O(N \times K)$ reads only projected $K$ columns |
| **Entropy & Compression** | Heterogeneous types (poor) | Interleaved rows limit repeat patterns | High value locality $\rightarrow$ 3x–10x (Snappy / ZSTD) |
| **Filter Push down** | None (full table scan) | Record-level sequential scanning | Row Group & Page-level min/max pruning |
| **Optimal Workload** | Data ingest, web APIs, logging | High-write OLTP, streaming RPCs | Read-intensive OLAP, ad-hoc aggregation, BI |

---

## Working with Parquet in C# (ParquetSharp's 4 APIs)

**ParquetSharp** (G-Research) is the official cross-platform .NET wrapper around `apache-parquet-cpp`. It exposes 4 distinct API layers:

```csharp
public record struct Metric(int DeviceId, float Val);
```

### 1. Row-Oriented API
*Object mapping via reflection. Ideal for standard entity collections.*
```csharp
using ParquetSharp.RowOriented;

using var writer = ParquetFile.CreateRowWriter<Metric>("metrics.parquet");
writer.WriteRows(new[] { new Metric(1, 42.5f) });

using var reader = ParquetFile.CreateRowReader<Metric>("metrics.parquet");
Metric[] rows = reader.ReadRows();
```

### 2. Logical Column API
*Columnar streaming mapped to .NET types. Prunes unused columns without manual level buffers.*
```csharp
using var fileReader = new ParquetFileReader("metrics.parquet");
using var rowGroup = fileReader.RowGroup(0);

// Read column 1 ("Val") directly into primitive array (skips column 0)
using var colReader = rowGroup.Column(1).LogicalReader<float>();
float[] values = colReader.ReadAll(rowGroup.MetaData.NumRows);
```

### 3. Low-Level (Physical) Column API
*Direct C++ engine parity. Requires explicit handling of Dremel D/R levels for raw primitives.*
```csharp
using var colReader = (FloatReader)rowGroup.Column(1);
long count = rowGroup.MetaData.GetColumnChunkMetaData(1).NumValues;

float[] values = new float[count];
short[] defLevels = new short[count], repLevels = new short[count];
colReader.ReadBatch(count, defLevels, repLevels, values, out _);
```

### 4. Apache Arrow & DataFrame API
*Zero-copy in-memory analytics via `ParquetSharp.DataFrame`.*
```csharp
using ParquetSharp.DataFrame;

// Project column directly into .NET DataFrame without allocating C# objects
var df = fileReader.ToDataFrame(columns: new[] { "Val" }, rowGroupIndices: new[] { 0 });
var avg = ((PrimitiveDataFrameColumn<float>)df.Columns["Val"]).Mean();
```

---

## Sources

- [Apache Parquet Documentation & Format Specification](https://parquet.apache.org/docs/file-format/)
- [Melnik et al. (2010) — Dremel: Interactive Analysis of Web-Scale Datasets (VLDB)](https://research.google/pubs/pub36632/)
- [Apache Parquet Format Thrift Definition](https://github.com/apache/parquet-format)
- [ParquetSharp Official Documentation & API Guide (G-Research)](https://g-research.github.io/ParquetSharp/)
- [ParquetSharp GitHub Repository](https://github.com/G-Research/ParquetSharp)
- [DuckDB: Querying Parquet Internals](https://duckdb.org/2021/06/25/querying-parquet.html)
