# gridcoarsefine
gridcoarsefine is a single-header c++ library for 2d vectors. Features conversions for different grid sizes. `namespace gcf`

It's basically a fancy wrapper for
```cpp
struct V2D
{
  int x;
  int y;
};
```

## Example Usage
```cpp
#include <cstdint>
#include <unordered_map>
using namespace gcf;

const int chunk_size = 64;

using pixelV2D = intV2D<int64_t, 16>;
using tileV2D = intV2D<int32_t, 16 * 16>;
using chunkV2D = intV2D<int32_t, 16 * 16 * chunk_size>;

struct tile {};
struct Chunk
{
    tile tiles[chunk_size][chunk_size];
};

const Chunk * get_chunk(chunkV2D); //maybe an unordered_map with chunkV2D key

tile get_tile_at_pixel(pixelV2D pixel_grid_position)
{
    tileV2D tile_grid_position = tileV2D(pixel_grid_position); //these conversions will use divPos_binary, which compiles to a bit shift
    chunkV2D chunk_grid_position = chunkV2D(tile_grid_position);
    const Chunk * chunk = get_chunk(chunk_grid_position);

    tileV2D tile_within_chunk = tileV2D(modPos_binary<chunk_size>(tile_grid_position.x),
                                        modPos_binary<chunk_size>(tile_grid_position.y)); //modPos_binary is just a binary &
    return chunk->tiles[tile_within_chunk.y][tile_within_chunk.x];
}
```

## Types

* `genericV2D <type>`
  Basic {x, y} struct with mathematical operations. `type` can be int, float, or `class my_custom_rational_type` as long as it has numeric operators.

* `genericV2D_r`
  Grandparent of other types. Implements mathematical operations expected of a 2d vector.

  Exists because some children types need to stay the same type when returned (thats the `_r`) from basic mathimatical operations

* `intV2D <std::integral, int es>`
  integer {x, y} type with a tag, `es`, that defines the courseness of a grid, with 1 being the tiniest squares.

  `intV2D<int, 10>{5, 6}` can be easily converted to `intV2D<int, 1>{50, 60}` with a conversion constructor. If both `es` are a power of 2, this ends up just being a few bit shifts.

  Supports converting between different `std::integral` types

## Functions

* `dot`, `cross`, and mathematical operators

* `divPos(a, b)` and `modPos(a, b)` - does a / b division with a remainder that is always positive.

  `divPos(-7,3)` = -3

  `modPos(-7,3)` = 2

* `divPos_binary<b>(a)` and `modPos_binary<b>(a)` - binary optimisation. denominator of division must be a positive power of two, and is passed as a template parameter

* `static intV2D intV2D::trim_cast(intV2D)` - used to convert `intV2D` from a large `std::integral` to a smaller `std::integral`, possibly trimming bits
