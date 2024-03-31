
# format

- all numbers are stored in big endian byte order

The file is composed of several parts:
1. A **header** with 66 bytes
2. compressed **metadata** (tiles.json)
3. several **blocks**, where each block consists of:
   - several **tiles**
   - **index** of these tiles
4. **index** of all blocks

<p align="center"><img src="file_format.svg"></p>

## file

### `file_header` (66 bytes)

- all `offset`s are relative to start of the file
  
| offset | length | type   | description               |
|--------|--------|--------|---------------------------|
| 0      | 14     | string | `"versatiles_v01"`        |
| 14     | 1      | u8     | `tile_format`             |
| 15     | 1      | u8     | `tile_precompression`     |
| 16     | 1      | u8     | min zoom level            |
| 17     | 1      | u8     | max zoom level            |
| 18     | 4      | f32    | bbox min x (longitude)    |
| 22     | 4      | f32    | bbox min y (latitude)     |
| 26     | 4      | f32    | bbox max x (longitude)    |
| 30     | 4      | f32    | bbox max y (latitude)     |
| 34     | 8      | u64    | `offset` of `meta`        |
| 42     | 8      | u64    | `length` of `meta`        |
| 50     | 8      | u64    | `offset` of `block_index` |
| 58     | 8      | u64    | `length` of `block_index` |

### `tile_format` values:
  - `0`: png
  - `1`: jpg
  - `2`: webp
  - `3`: pbf

### `tile_precompression` values:
  - `0`: uncompressed
  - `1`: gzip compressed
  - `2`: brotli compressed

### `meta`

- Content of `tiles.json`
- UTF-8
- compressed with `$tile_precompression`

### `block`

- Each `block` is like a "super tile" and contains data of up to 256x256 (= 65536) `tile`s.

### `block_index` (29 bytes per block)

- Brotli compressed data structure
- Empty `block`s are not stored
- For each block `block_index` contains a 29 bytes long record:

| offset    | length | type | description              |
|-----------|--------|------|--------------------------|
| 0 + 29*i  | 1      | u8   | `level`                  |
| 1 + 29*i  | 4      | u32  | `column`/256             |
| 5 + 29*i  | 4      | u32  | `row`/256                |
| 9 + 29*i  | 1      | u8   | `col_min` (0..255)       |
| 10 + 29*i | 1      | u8   | `row_min` (0..255)       |
| 11 + 29*i | 1      | u8   | `col_max` (0..255)       |
| 12 + 29*i | 1      | u8   | `row_max` (0..255)       |
| 13 + 29*i | 8      | u64  | `offset` of `tile_index` |
| 21 + 29*i | 8      | u64  | `length` of `tile_index` |

## `block`

- Each `block` contains data of up to 256x256 (= 65536) `tile`s.
- Levels 0-8 can be stored with one `block` each. level 9 might contain 512x512 `tile`s so 4 `block`s are necessary.

<p align="center"><img src="level_blocks.svg"></p>

- Each `block` contains the concatenated `tile` blobs and ends with a `tile_index`.
- Neither the order of `block`s in the `file` nor the order of `tile`s in a `block` matters as long as their indexes are correct.
- Note: To efficiently find the `block` that contains the `tile` you are looking for, use a data structure such as a "map", "dictionary", or "associative array" and fill it with the data from the `block_index`.

### `tile`

- each tile is a PNG/PBF/… file as data blob
- compressed with `$tile_precompression`

### `tile_index`

- brotli compressed data structure
- `tile`s are read horizontally then vertically
- `j = (row - row_min)*(col_max - col_min + 1) + (col - col_min)`

<p align="center"><img src="block_tiles.svg"></p>

- identical `tile`s can be stored once and referenced multiple times to save storage space
- if a `tile` does not exist, the length of `tile` is zero

| offset | length | type | description               |
|--------|--------|------|---------------------------|
| 12*j   | 8      | u64  | `offset` of `tile_blob` j |
| 12*j+8 | 4      | u32  | `length` of `tile_blob` j |
