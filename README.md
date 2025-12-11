## 🌟 Overview

`search_engine_compressors` is a Rust library providing fast, efficient implementations of various **Integer Compression Algorithms** commonly used in **Inverted Indexes** and **Search Engine** technologies.

These schemes are optimized for compressing arrays of sorted, positive integers (like posting lists and document IDs) by encoding the differences (deltas) between successive numbers.



### Implemented Algorithms

This crate currently supports the following schemes, available as individual modules:

| Algorithm | Module | Description |
| :--- | :--- | :--- |
| **Simple9** | `simple9` | A variable-byte scheme that packs up to nine integers into a single 32-bit word. Excellent balance of speed and compression for moderate-to-sparse deltas. |
| **Simple16** | `simple16` | Similar to Simple9, packing up to 16 integers per word. |
| **Simple8b** | `simple8b` | Packs up to 8 integers into a 64-bit word using a selector and bit lengths. |
| **PForDelta** | `p_for_delta` | Patched Frame of Reference Delta. A highly effective blocked compression scheme that is often the top performer in search engine use cases. |
| **Variable-Byte** | `var_byte` | The classic byte-oriented compression scheme. Simple and effective for very large numbers. |
| **Rice Coding** | `rice` | A form of entropy encoding often used for positive integers with a geometric distribution. |

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