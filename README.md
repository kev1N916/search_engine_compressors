## 🌟 Overview

`search_engine_compressors` is a Rust library providing fast, efficient implementations of various **Integer Compression Algorithms** commonly used in **Inverted Indexes** and **Search Engine** technologies.

These schemes are optimized for compressing arrays of sorted, positive integers (like posting lists and document IDs) by encoding the differences (deltas) between successive numbers.

# Compression Algorithms Overview

The algorithms implemented are Simple9,Simple16,Simple8b,PForDelta and Rice Coding.

## Algorithm Comparison

### Simple-9/16/8b

**Alignment:** Word (e.g., 32/64-bit)

**Core Principle:** Packs a variable number of small integers into a fixed-size word using a selector (scheme).

**Primary Use Cases:**
- Inverted indexes
- High-speed lookup tables

**Key Advantage:** Extremely fast decoding speed due to word alignment.

---

### PForDelta

**Alignment:** Block/Word

**Core Principle:** Compresses a block by fitting the majority of values into a fixed, small bit-width *B* and treating the few larger values as exceptions.

**Primary Use Cases:**
- Inverted indexes (postings lists)
- Time-series data

**Key Advantage:** Excellent balance between compression ratio and decoding speed.

---

### Variable-Byte (VByte)

**Alignment:** Byte

**Core Principle:** Uses a variable number of bytes (1 to 10) per integer, flagged by the Most Significant Bit (MSB).

**Primary Use Cases:**
- Database systems
- General data storage

**Key Advantage:** Simple, highly byte-aligned, and great for lists with many small numbers.

---

### Rice Coding

**Alignment:** Bit-wise

**Core Principle:** A specialized form of Golomb coding that uses a power-of-two parameter *k* to encode numbers into a quotient (unary) and a remainder (fixed-length).

**Primary Use Cases:**
- Compressing residuals
- Highly skewed distributions (e.g., post-delta-encoding)

**Key Advantage:** Optimal entropy coding for data following a geometric distribution; fast bitwise operations.

## 🛠️ Usage

### 1. Add to your project

Add the following to your `Cargo.toml` file:

```toml
[dependencies]
search_engine_compressors = "0.1"
```
## Examples

### Simple9 Compression

Simple9 is a lightweight integer compression algorithm ideal for small to medium-sized datasets.

```rust
use search_engine_compressors::simple9;

fn main() {
    // Input data is a vector of u32 (or u64 for some modules)
    let data: Vec<u32> = vec![1, 2, 3, 4, 5, 10, 20, 30];

    // Encode the data
    let encoded: Vec<u32> = simple9::compress(&data);
    println!("Encoded size (u32 words): {}", encoded.len());

    // Decode the data
    let decoded: Vec<u32> = simple9::decompress_from_bytes(&encoded);

    // Note: Decompression might return padding if the input size is not a multiple of the block size.
    // Always assert against the original data length.
    assert_eq!(&data, &decoded[..data.len()]);
}
```

### PForDelta Compression

PForDelta is a block-based compression algorithm optimized for large, ordered lists with delta encoding.

```rust
use search_engine_compressors::p_for_delta;

fn main() {
    // PForDelta is block-based, often used on deltas of large, ordered lists.
    let data: Vec<u32> = (0..128).map(|i| i as u32 * 10).collect();

    let encoded: Vec<u32> = p_for_delta::compress(&data);
    let decoded: Vec<u32> = p_for_delta::decompress(&encoded);

    assert_eq!(&data, &decoded[..data.len()]);
}
```