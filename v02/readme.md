
# overview

- All numbers are stored in big endian byte order.
- Tiles are organised in the XYZ scheme, and not the TMS scheme. So tiles with x=0,y=0 are in the top left corner.

The file is composed of four parts:
1. starting with a [**`file_header`**](#file_header)
2. followed by compressed [**`metadata`**](#metadata)
3. followed by several [**`block`s**](#block), where each block consists of:
   - concatenated [**`tile`s**](#tile) blobs
   - followed by [**`tile_index`**](#tile_index) as an index of these tiles
4. followed by [**`block_index`**](#block_index) as an index of all blocks

<p align="center"><img src="file_format.svg?raw=true"></p>

# `file_header`

- has a length of 66 bytes
- all offsets are relative to start of the file
  
| offset | length | type   | description             |
|--------|--------|--------|-------------------------|
| 0      | 14     | string | `"versatiles_v02"`      |
| 14     | 1      | u8     | `tile_format`           |
| 15     | 1      | u8     | `tile_precompression`   |
| 16     | 1      | u8     | min zoom level          |
| 17     | 1      | u8     | max zoom level          |
| 18     | 4      | f32    | bbox min x (longitude)  |
| 22     | 4      | f32    | bbox min y (latitude)   |
| 26     | 4      | f32    | bbox max x (longitude)  |
| 30     | 4      | f32    | bbox max y (latitude)   |
| 34     | 8      | u64    | offset of `metadata`    |
| 42     | 8      | u64    | length of `metadata`    |
| 50     | 8      | u64    | offset of `block_index` |
| 58     | 8      | u64    | length of `block_index` |

## `tile_format` values:

| value  | type     | mime                       |
|--------|----------|----------------------------|
| `0x00` | bin      | *application/octet-stream* |
| `0x10` | png      | *image/png*                |
| `0x11` | jpg      | *image/jpeg*               |
| `0x12` | webp     | *image/webp*               |
| `0x13` | avif     | *image/avif*               |
| `0x14` | svg      | *image/svg+xml*            |
| `0x20` | pbf      | *application/x-protobuf*   |
| `0x21` | geojson  | *application/geo+json*     |
| `0x22` | topojson | *application/topo+json*    |
| `0x23` | json     | *application/json*         |

## `tile_precompression`

allowed values:
  - `0`: uncompressed
  - `1`: gzip compressed
  - `2`: brotli compressed

# `metadata`

Content of `tiles.json`. Encoded in UTF-8, compressed with `$tile_precompression`.
If no metadata is specified, offset and length must be `0`.

# `block`

- Each `block` is like a "super tile" and contains data of up to 256x256 (= 65536) `tile`s.
- Levels 0-8 can be stored with one `block` each. Level 9 can contain up to 512x512 `tile`s so up to 4 `block`s are necessary.

<p align="center"><img src="level_blocks.svg?raw=true"></p>

- Each `block` contains the concatenated `tile` blobs and ends with a `tile_index`.
- Neither `tile`s in a `block` nor `block`s in a `file` have to be sorted in any kind of order, as long as their indexes are correct.
- But it is recommended to sort the `tile`s in a `block` in an ascending order (e.g. to improve caching efficiency when reading/converting the whole file).

## `tile`

- each tile is a PNG/PBF/... file as data blob
- compressed with `$tile_precompression`

## `tile_index`

- Brotli compressed data structure
- `tile`s are read horizontally then vertically
- `index = (row - row_min)*(col_max - col_min + 1) + (col - col_min)`
- (`col_min`, `row_min`, `col_max`, `row_max` are specified in `block_index`)
- identical `tile`s can be stored once and referenced multiple times to save storage space
- if a `tile` does not exist, the length of `tile` is `0`
- offsets of `tile_blob`s are relative to the beginning of the `block`. So the offset of the first `tile_blob` should always be `0`.

| offset | length | type | description                      |
|--------|--------|------|----------------------------------|
| 12*i   | 8      | u64  | offset of `tile_blob` in `block` |
| 12*i+8 | 4      | u32  | length of `tile_blob`            |

<p align="center"><img src="block_tiles.svg?raw=true"></p>

# `block_index`

- Brotli compressed data structure
- Empty `block`s are not stored
- For each block `block_index` contains a 33 bytes long record:

| offset    | length | type | description               |
|-----------|--------|------|---------------------------|
| 0 + 33*i  | 1      | u8   | `level`                   |
| 1 + 33*i  | 4      | u32  | `column`/256              |
| 5 + 33*i  | 4      | u32  | `row`/256                 |
| 9 + 33*i  | 1      | u8   | `col_min` (0..255)        |
| 10 + 33*i | 1      | u8   | `row_min` (0..255)        |
| 11 + 33*i | 1      | u8   | `col_max` (0..255)        |
| 12 + 33*i | 1      | u8   | `row_max` (0..255)        |
| 13 + 33*i | 8      | u64  | offset of `block` in file |
| 21 + 33*i | 8      | u64  | length of `tile_blobs`    |
| 29 + 33*i | 4      | u32  | length of `tile_index`    |

- Since a `block` only consists of `tile_blobs` appended by `tile_index`, the length of `block` must be the sum of the lengths of `tile_blobs` and `tile_index`.
- Note: To efficiently find the `block` that contains the `tile` you are looking for, use a data structure such as a "map", "dictionary" or "associative array" and fill it with the data from the `block_index`.
