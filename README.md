# simply-cpp-geometry

Points, sizes and shapes (`sc::rect`, `sc::polygon`, `sc::circle`, `sc::line`, `sc::path`, ...), affine
transforms, distances in real units, percentages, small matrices, dual numbers and DBSCAN clustering:
the geometry the simply-cpp drawing modules (sc-graphics, sc-image, sc-pdflite) share. Only the
standard library.

**Documentation:** the [simply-cpp wiki](https://github.com/roelofrossouw/simply-cpp/wiki) covers the whole suite.

Until sc-core 4.0.0 this was part of [sc-core](https://github.com/roelofrossouw/simply-cpp) (as
`<sc/core/x.h>`); the names are unchanged and still in `sc::`, only the headers moved to
`<sc/geometry/x.h>`.

## Install

### apt (Ubuntu)

```bash
curl -fsSL https://apt.roelof.co.za/setup.sh | bash # registers the apt repo
sudo apt -y install simply-cpp-geometry-dev
```

## Dependencies

The library uses only the standard library. Building it (its helpers, tests and demos) needs sc-core.

## Usage

```cmake
find_package(sc-geometry CONFIG REQUIRED)

add_executable(myapp main.cpp)
target_link_libraries(myapp PRIVATE sc::sc-geometry)
```

## Geometry

`<sc/geometry/geometry.h>` has points, sizes and the shapes: `sc::rect`, `sc::rounded_rect`,
`sc::rotated_rect`, `sc::polygon`, `sc::circle`, `sc::line` and `sc::path` (lines and cubic
Bezier curves, as in SVG). Each but `path` also comes in whole pixels (`sc::rect_i`, ...); all
but `rotated_rect` convert int to double implicitly and back explicitly, rounding. Coordinates
are y down. It also has DBSCAN clustering (`sc::dbscan`, `sc::dbscan_by`).

The `sc::geometry` concept is `bounds()` (a path's exact, curves included) plus a free
`sc::to_path(shape)` giving the outline; a shape of your own with both counts too. `sc::shape`
holds any one shape, and `sc::group` is a collection of them that is a shape itself (groups
nest). Shapes are only geometry: sc-graphics' `sc::graphics::canvas` draws them, implemented by
sc-image's `sc::image::picture` and sc-pdflite's `sc::pdflite::writer`.

```cpp
const sc::rounded_rect button{sc::rect{10, 10, 80, 30}, 6};
const sc::path outline = sc::to_path(button);   // straight sides, a curve per corner
const sc::rect box = outline.bounds();          // 10, 10, 80 x 30

sc::group marks{button, sc::circle{{50, 60}, 5}, sc::line{{0, 0}, {9, 9}}};
marks.bounds();                                 // all three
marks.flip_horizontal(sc::rect{0, 0, 595, 842}); // mirrored across an A4 page, as a whole
```

`<sc/geometry/distance.h>` has `sc::distance`, a length kept in points (1/72 inch, the unit PDFs are drawn
in), and the units in `sc::dist` deriving from it: `dist::mm`, `dist::cm`, `dist::inch` and
`dist::point` (a namespace of their own, so they keep their plain names without clashing with
`sc::point` or each other). Each converts to its number of points as a double, so every shape
and drawing call takes them as they are, mixed with plain numbers (points). `points()`,
`millimetres()`, `centimetres()` and `inches()` read one back, `<<` writes it with its unit
(`595pt`, `210mm`), and `width.in<sc::dist::mm>()` (or `static_cast<sc::dist::mm>(width)`) gives the same length in
another unit. Points and sizes can hold distances too, keeping their units:
`sc::size_<sc::distance>` prints `[595.276pt,841.89pt]`, and its `in<sc::dist::mm>()` is
`[210mm,297mm]`; with `using namespace sc::dist;` that's `size.in<mm>()`.

```cpp
const sc::rect box{sc::dist::mm(10), sc::dist::inch(1.1), sc::dist::point(72), 72}; // 28.35, 79.2, 72 x 72
const double margin = sc::dist::mm(15) + sc::dist::inch(0.5);                      // 78.52 points
const sc::size a4{sc::dist::mm(210), sc::dist::mm(297)};                           // 595.28 x 841.89
```

`<sc/geometry/transform.h>` has `sc::transform`, an affine transform (SVG's `matrix(a b c d e f)`):
`translate`, `scale`, `rotate` (degrees, clockwise on screen) and `flip_horizontal`/`flip_vertical`,
about any point or line, composed with `then()`. `apply()` keeps a shape's kind where it can: a
rect stays a rect when moved, scaled, flipped or quarter-turned, becomes a `rotated_rect` when
turned and a polygon when sheared; a circle stays a circle unless stretched (then a path). Lines,
polygons and paths always keep theirs, and a group applies it to each shape (`group::apply()`).

```cpp
const auto turn = sc::transform::rotate(30, {50, 50}).then(sc::transform::translate(10, 0));
const sc::shape turned = turn.apply(sc::rect{20, 40, 60, 20});   // a rotated_rect
const sc::point moved = turn.apply(sc::point{50, 50});           // 60, 50
```


`<sc/geometry/percent.h>` has `sc::percent` (a fraction or a number of percent, rounded for display),
`<sc/geometry/matrix.h>` small fixed-size matrices (`sc::matrix<T, rows, columns>`), `<sc/geometry/dual.h>`
dual numbers that work out derivatives as they calculate, and `<sc/geometry/dbscan.h>` DBSCAN
clustering of numbers:

```cpp
const auto clusters = sc::dbscan(std::vector<double>{1, 1.1, 20}, 0.5, 2); // {0, 0, -1}
```


## Demo

| Demo | Shows |
|---|---|
| `sc-geometry-demo` | points and sizes, rectangles (overlap, distance, grouping), rotated rectangles and polygons, DBSCAN clustering |
| `sc-geometry-numbers` | `sc::percent`, `sc::matrix`, dual numbers for derivatives |


## Requirements

- CMake 3.22 or newer
- A C++20 compiler

## Building and testing

```bash
cmake -B build -S .
cmake --build build -j
ctest --test-dir build --output-on-failure
```
