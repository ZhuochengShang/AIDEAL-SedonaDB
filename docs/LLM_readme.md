# SedonaDB configured SQL Original R0 — LLM_readme

LLM-facing API documentation. Generated from the API surface, project profile, and distilled docs.

## API Test: `RS_AsGeoTiff`

### Signature
```python
RS_AsGeoTiff(arg1: raster) -> binary | RS_AsGeoTiff(arg1: raster, tile_size: integer) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double, tile_size: integer) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double, tile_width: integer, tile_height: integer) -> binary
```
_Source: source/docs/reference/sql/rs_asgeotiff.qmd:19_

### Goal
Encodes a SedonaDB `raster` object into a binary GeoTIFF file format, supporting optional compression and tiling configurations.

### Parameters
- `arg1` (`raster`): The input raster object to be encoded.
- `compression` (`string`): The codec to use (case-insensitive): 'None', 'PackBits', 'Deflate', 'Huffman', 'LZW', or 'JPEG'.
- `quality` (`double`): A value between 0.0 and 1.0 for JPEG lossy compression; ignored by other codecs.
- `tile_width` (`integer`): The width of the GeoTIFF tiles in pixels; must be a multiple of 16.
- `tile_height` (`integer`): The height of the GeoTIFF tiles in pixels; must be a multiple of 16.

### Input
The function accepts a `raster` object (e.g., created via `RS_FromPath` or `RS_MakeEmptyRaster`). If the input is `NULL`, the result is `NULL`. For tiled output, `tile_width` and `tile_height` must be provided as multiples of 16.

### Output
Returns `binary` containing the raw bytes of the encoded GeoTIFF file.

### Valid Call Patterns
```python
# Basic encoding
sd.sql("SELECT RS_AsGeoTiff(RS_Example())").to_arrow_table()

# Tiled encoding
sd.sql("SELECT RS_AsGeoTiff(RS_Example(), 16)").to_arrow_table()

# Compressed encoding
sd.sql("SELECT RS_AsGeoTiff(RS_Example(), 'DEFLATE', 0.85)").to_arrow_table()

# Fully specified tiled and compressed encoding
sd.sql("SELECT RS_AsGeoTiff(RS_Example(), 'LZW', 0.85, 16, 16)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_AsGeoTiff` to convert internal raster objects to binary GeoTIFF format.
- Always ensure `tile_width` and `tile_height` are multiples of 16 if tiling is requested.
- Use the appropriate overload based on whether you need compression, quality settings, or specific tile dimensions.

### Prompt Snippet
```text
RS_AsGeoTiff(raster_col, 'LZW', 0.9, 32, 32)
```

### Common Failure Modes
- **Invalid Tile Dimensions:** Providing `tile_width` or `tile_height` that are not multiples of 16 will cause an error.
- **Invalid Quality Range:** Providing a `quality` value outside the 0.0–1.0 range for JPEG compression will raise an error.
- **Unsupported Codec:** Using a codec string not in the supported list (e.g., 'None', 'Deflate', 'LZW') will fail.
- **Huffman Errors:** Using 'Huffman' (CCITT RLE) on multi-band or non-1-bit data will result in an error.

### Fix Code Hint
```python
# Ensure tile dimensions are multiples of 16
# Example: 32 is a valid multiple of 16
sd.sql("SELECT RS_AsGeoTiff(my_raster, 'LZW', 0.8, 32, 32) FROM my_table").to_arrow_table()
```

__CHECK__ witness:
```python
# Verify the function returns binary data for a valid raster
result = sd.sql("SELECT RS_AsGeoTiff(RS_Example()) IS NOT NULL as is_valid").to_arrow_table()
assert result["is_valid"][0] == True
print("__CHECK__")
```

## API Test: `RS_AsRaster`

### Signature
```python
RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double, use_geometry_extent: boolean) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double, use_geometry_extent: boolean) -> raster
```

### Goal
Creates a single-band in-db raster by burning a geometry into the grid defined by a reference raster.

### Parameters
- `geometry` (`geometry`): The input vector geometry to be rasterized.
- `reference_raster` (`raster`): The template raster defining the grid, CRS, and spatial extent.
- `pixel_type` (`string`): The data type for the output raster pixels (e.g., 'uint8', 'float64').
- `all_touched` (`boolean`): If true, all pixels touched by the geometry are burned; if false, only pixels whose centers are covered by the geometry are burned.
- `burn_value` (`double`): The value to assign to pixels covered by the geometry.
- `nodata_value` (`double`): The value used to represent "no data" in the output raster.
- `use_geometry_extent` (`boolean`): If true, the output raster extent is cropped to the geometry; if false, it matches the reference raster extent.

### Input
- **Data:** A valid `geometry` object and a `raster` object.
- **Preconditions:** If both inputs have a CRS, they must match or be reprojectable; if only one has a CRS, the operation will return an error.

### Output
Returns a `raster` object representing the rasterized geometry on the reference grid.

### Valid Call Patterns
```python
# Example using the full parameter set
query = """
SELECT RS_AsRaster(
  ST_SetSRID(ST_GeomFromText('POLYGON ((10 10, 50 10, 50 50, 10 50, 10 10))'), 4326),
  RS_MakeEmptyRaster(1, 'uint8', 400, 400, ST_MakeEnvelope(0, 0, 400, 400, 4326)),
  'uint8', false, 1.0, 0.0, false
)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure the `reference_raster` is provided as a valid raster object (e.g., from `RS_FromPath` or `RS_MakeEmptyRaster`).
- Ensure the `pixel_type` is a valid string supported by the engine.
- If the geometry and raster have different CRSs, ensure they are compatible for reprojection.

### Prompt Snippet
```text
RS_AsRaster(geometry_col, reference_raster_col, 'float64', false, 1.0, 0.0, false)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and a reference raster where only one has a defined CRS will cause an error.
- **Invalid Pixel Type:** Providing an unsupported string for `pixel_type`.
- **Memory Constraints:** Attempting to rasterize extremely large geometries or using very high-resolution reference rasters may exceed memory limits.

### Fix Code Hint
```python
# Ensure both inputs have the same CRS before calling
sd.sql("SELECT RS_AsRaster(ST_Transform(geom, 4326), raster, 'float64') FROM table")
```

## API Test: `RS_BandIsNoData`

### Signature
```python
RS_BandIsNoData(arg1: raster) -> boolean | RS_BandIsNoData(arg1: raster, band: integer) -> boolean
```
_Source: source/docs/reference/sql/rs_bandisnodata.qmd:19_

### Goal
Determines if every pixel within a specified raster band matches the band's defined nodata value.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `band` (`integer`): The 1-based index of the band to check. Defaults to 1 if not specified.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must contain a defined nodata value. If a band has no nodata value, the function will return `false` even if all pixels are 0. Only 2-D rasters are supported.

### Output
Returns `boolean`: `true` if every pixel in the band equals the nodata value (or if the pixel is NaN and the nodata value is NaN), otherwise `false`.

### Valid Call Patterns
```python
# Check the default (first) band
sd.sql("SELECT RS_BandIsNoData(RS_Example())").to_arrow_table()

# Check a specific band (e.g., band 1)
sd.sql("SELECT RS_BandIsNoData(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_BandIsNoData` to verify if a raster band is effectively empty based on its nodata metadata.
- Always specify the band index as an integer if checking a band other than the first.
- Remember that a band without a nodata value will always return `false`.

### Prompt Snippet
```text
RS_BandIsNoData(raster_column, band_index)
```

### Common Failure Modes
- **Missing Nodata Value:** If the raster band metadata does not explicitly define a nodata value, the function returns `false` regardless of pixel content.
- **Unsupported Dimensions:** Attempting to use this function on non-2-D rasters will result in an error.
- **Invalid Band Index:** Providing a band index that does not exist in the raster (e.g., 0 or a number greater than `RS_NumBands`) will cause a runtime error.

### Fix Code Hint
```python
# Ensure the raster has a nodata value set before checking
# If the band index is unknown, verify with RS_NumBands(raster) first
result = sd.sql("SELECT RS_BandIsNoData(my_raster, 1) FROM my_table").to_arrow_table()
print(f"__CHECK__: {result}")
```

## API Test: `RS_BandNoDataValue`

### Signature
```python
RS_BandNoDataValue(arg1: raster) -> float64 | RS_BandNoDataValue(arg1: raster, band: int) -> float64
```
_Source: source/docs/reference/sql/rs_bandnodatavalue.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_BandNoDataValue
description: >
  Returns the nodata value of the specified band as a double. Returns null if the
  band has no nodata value defined.
kernels:
  - returns: float64
    args: [raster]
  - returns: float64
    args:
    - raster
    - name: band
      type: int
      description: Band index (1-based). Defaults to 1 if not specified.
---

## Examples

```sql
SELECT RS_BandNoDataValue(RS_Example());
```

```sql
SELECT RS_BandNoDataValue(RS_Example(), 1);
```


### Goal
Retrieves the defined "NoData" value for a specific band within a raster object to facilitate accurate masking and statistical analysis.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the NoData value.
- `band` (`int`): The 1-based index of the band to query. Defaults to 1 if not provided.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must contain the specified band index. If the band index is out of range, the query will fail.

### Output
Returns `float64` — The NoData value associated with the specified band. Returns `null` if no NoData value is defined for that band.

### Valid Call Patterns
```python
# Verify NoData value for the first band
result = sd.sql("SELECT RS_BandNoDataValue(RS_Example())").to_arrow_table()
print(result)
# __CHECK__

# Verify NoData value for a specific band (e.g., band 1)
result = sd.sql("SELECT RS_BandNoDataValue(RS_Example(), 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_BandNoDataValue` to inspect raster metadata before performing operations that rely on NoData masking.
- Remember that band indexing is 1-based.
- If the result is `null`, the raster band does not have a NoData value explicitly set.

### Prompt Snippet
```text
RS_BandNoDataValue(raster_column, band_index)
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index that does not exist in the raster (e.g., 0 or a number greater than `RS_NumBands`).
- **Null Handling:** Failing to account for `null` returns when a raster band lacks a defined NoData value.

### Fix Code Hint
```python
# Ensure the band index is within the valid range [1, RS_NumBands(raster)]
# Use COALESCE if you need a default value when NoData is not defined
sd.sql("SELECT COALESCE(RS_BandNoDataValue(r, 1), -9999.0) FROM my_raster_table")
```

## API Test: `RS_BandPath`

### Signature
```python
RS_BandPath(arg1: raster) -> utf8
```
_Source: source/docs/reference/sql/rs_bandpath.qmd:19_

### Goal
Retrieves the external file path associated with an out-of-database (out-db) raster band.

### Parameters
- `arg1` (`raster`): The raster object from which to extract the file path.

### Input
- **Data:** A raster object, typically created via `RS_FromPath` or similar functions that reference external files.
- **Preconditions:** The raster must be an "out-db" raster. If the raster is stored entirely in-database (e.g., created via `RS_FromGDALRaster` without external referencing), the function may return an empty string or null, as no external file path exists.

### Output
Returns `utf8` — The absolute or relative file system path to the external raster file referenced by the input raster.

### Valid Call Patterns
```python
# Using the registered SQL surface
result = sd.sql("SELECT RS_BandPath(RS_FromPath('/path/to/raster.tiff'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_BandPath` only when inspecting the metadata of out-db rasters.
- Always ensure the input is a valid `raster` type.
- Do not assume the returned path is valid on the local file system if the raster was moved or if the database is running in a containerized environment where paths differ.

### Prompt Snippet
```text
SELECT RS_BandPath(raster_column) FROM raster_table;
```

### Common Failure Modes
- **In-DB Rasters:** Calling this on a raster that is fully materialized in the database will result in an empty or null return value, as there is no external file path.
- **Invalid Raster Input:** Passing a non-raster type (e.g., geometry or string) will result in a SQL execution error.
- **Path Accessibility:** If the raster was registered with a path that is not accessible to the current SedonaDB process, the path will still be returned, but subsequent attempts to read the file will fail.

### Fix Code Hint
```python
# Ensure the raster is an out-db raster before calling
# If the result is empty, the raster is likely in-db
df = sd.sql("SELECT RS_BandPath(my_raster) as path FROM my_table")
```

## API Test: `RS_BandPixelType`

### Signature
```python
RS_BandPixelType(arg1: raster) -> utf8 | RS_BandPixelType(arg1: raster, band: int) -> utf8
```
_Source: source/docs/reference/sql/rs_bandpixeltype.qmd:19_

### Goal
Returns the pixel data type (e.g., '8BUI', '16BSI', '32BF') of a specified raster band as a string.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): The 1-based index of the band to query. Defaults to 1 if not specified.

### Input
- **Data:** A valid raster object, typically loaded via `RS_FromPath` or `RS_FromGDALRaster`.
- **Preconditions:** The raster must be correctly initialized and contain the requested band index. If the band index is out of range, the function will return an error.

### Output
Returns `utf8` — A string representing the pixel data type of the specified band.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify default band (1) pixel type
result = sd.sql("SELECT RS_BandPixelType(RS_Example())").to_arrow_table()
print(result)
# __CHECK__

# Verify specific band pixel type
result = sd.sql("SELECT RS_BandPixelType(RS_Example(), 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_BandPixelType` to inspect raster metadata before performing operations that require specific pixel types (e.g., casting for resampling).
- Remember that band indices are 1-based.
- If the raster has multiple bands, explicitly provide the `band` integer argument to avoid ambiguity.

### Prompt Snippet
```text
RS_BandPixelType(raster_column, band_index)
```

### Common Failure Modes
- **Invalid Band Index:** Providing a `band` index that does not exist in the raster (e.g., 0 or a number greater than `RS_NumBands`) will cause a runtime error.
- **Null Input:** Passing a `NULL` raster object will result in a `NULL` output.
- **Type Mismatch:** Attempting to call this on a non-raster geometry type will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the band index is within the valid range [1, RS_NumBands(raster)]
# Example:
sd.sql("SELECT RS_BandPixelType(my_raster, 1) FROM my_table WHERE RS_NumBands(my_raster) >= 1")
```

## API Test: `RS_BandToDim`

### Signature
```python
RS_BandToDim(arg1: raster, dim_name: utf8) -> raster
```
_Source: source/docs/reference/sql/rs_bandtodim.qmd:19_

### Goal
Collapses all bands of a multi-band raster into a single band by prepending a new dimension, facilitating N-dimensional data structures like Zarr.

### Parameters
- `arg1` (`raster`): The input multi-band raster to be transformed.
- `dim_name` (`utf8`): The name for the newly created dimension (e.g., 'time', 'band').

### Input
- **Data:** A multi-band raster.
- **Preconditions:** All bands within the input raster must share identical dimension names, shapes, data types, and nodata values.
- **Constraint:** The `dim_name` must not already exist as a dimension name within the input raster.

### Output
Returns a `raster` where all original bands are concatenated into a single band, with the new dimension prepended to the existing dimensions.

### Valid Call Patterns
```python
# Example using the SedonaDB SQL interface
result = sd.sql("SELECT RS_BandToDim(RS_Example(), 'band')").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `RS_BandToDim` to convert multi-band rasters into a single-band N-dimensional format.
- Ensure the `dim_name` provided is unique and does not conflict with existing dimensions in the raster.
- Remember that this function is the inverse of `RS_DimToBand`.

### Prompt Snippet
```text
RS_BandToDim(raster_column, 'new_dimension_name')
```

### Common Failure Modes
- **Dimension Conflict:** Providing a `dim_name` that already exists in the input raster will cause an error.
- **Heterogeneous Bands:** If input bands have differing shapes, data types, or nodata values, the function will return an error to prevent data loss.
- **Experimental Status:** As an experimental feature, behavior may change; ensure your workflow accounts for potential updates in future SedonaDB versions.

### Fix Code Hint
```python
# Ensure all bands are uniform before calling
# If bands differ, you may need to resample or cast them first
sd.sql("SELECT RS_BandToDim(my_raster, 'time')").to_arrow_table()
```

## API Test: `RS_CRS`

### Signature
```python
RS_CRS(arg1: raster) -> string
```
_Source: source/docs/reference/sql/rs_crs.qmd:19_

### Goal
Retrieves the Coordinate Reference System (CRS) definition string associated with a given raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the CRS metadata.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must have a defined CRS. If the raster was created without a CRS (e.g., via certain `RS_MakeEmptyRaster` configurations), the function may return an empty string or null depending on the underlying implementation.

### Output
Returns `string` — The CRS definition, typically represented as a WKT (Well-Known Text) string or an EPSG code string, depending on how the raster's CRS was initialized.

### Valid Call Patterns
```python
# Using the registered SQL surface via Python
result = sd.sql("SELECT RS_CRS(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `sd.sql("SELECT RS_CRS(...)")` when calling this function from Python.
- Ensure the input is a valid raster type; do not pass geometry types or raw file paths directly to this function.

### Prompt Snippet
```text
SELECT RS_CRS(raster_column) FROM table_name
```

### Common Failure Modes
- **Null/Empty CRS:** If the raster has no CRS assigned, the function may return an empty string, which can cause downstream errors in spatial operations that require a defined CRS.
- **Type Mismatch:** Passing a geometry object instead of a raster object will result in a SQL execution error.
- **Invalid Raster:** Passing a null or uninitialized raster object will result in a runtime error.

### Fix Code Hint
```python
# Ensure the raster has a CRS before calling RS_CRS
# If missing, you may need to define it using RS_SetSRID if supported, 
# or verify the source file's metadata.
sd.sql("SELECT RS_CRS(my_raster) FROM my_table WHERE my_raster IS NOT NULL")
```

## API Test: `RS_Clip`

### Signature
```python
RS_Clip(arg1: raster, band: integer, geom: geometry) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double, crop: boolean) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double, crop: boolean, lenient: boolean) -> raster
```

### Goal
`RS_Clip` performs spatial clipping of a raster by a geometry, masking pixels outside the geometry to a nodata value and optionally cropping the raster extent to the geometry's bounding box.

### Parameters
- `arg1` (`raster`): The input raster to be clipped.
- `band` (`integer`): 1-based index of the band to clip, or 0 to clip every band. A NULL band yields a NULL result.
- `geom` (`geometry`): The clipping geometry. Reprojected into the raster's CRS if both carry one.
- `all_touched` (`boolean`): If true, keeps every pixel the geometry touches; otherwise, keeps only pixels whose center falls inside. Defaults to false.
- `no_data_value` (`double`): The value assigned to pixels outside the geometry. Defaults to the band's nodata value or the data type's minimum.
- `crop` (`boolean`): If true, crops the output to the geometry's bounding box. Defaults to true.
- `lenient` (`boolean`): If true, a geometry that does not intersect the raster yields NULL; if false, it raises an error. Defaults to true.

### Input
- **Data:** A `raster` object (e.g., from `RS_FromGDALRaster` or `RS_Example`) and a `geometry` object (e.g., from `ST_GeomFromText`).
- **Preconditions:** If both the raster and geometry have a CRS, they must match or be reprojectable. If exactly one has a CRS, the operation will error.

### Output
Returns a `raster` object representing the clipped (and optionally cropped) subset of the original input.

### Valid Call Patterns
```python
# Basic clip
sd.sql("SELECT RS_Clip(RS_Example(), 1, ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'))").to_arrow_table()

# Full parameter clip
sd.sql("SELECT RS_Clip(RS_Example(), 1, ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'), false, 0.0, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the `band` index is 1-based or 0.
- Verify that the `geom` and `raster` share a CRS or lack one entirely to avoid errors.
- Use `all_touched=True` if the geometry is small or thin to ensure pixels are captured.
- Remember that `RS_Clip` is experimental and behavior may change.

### Prompt Snippet
```text
RS_Clip(raster_col, 1, geom_col, all_touched=True, no_data_value=0.0, crop=True, lenient=True)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and raster where only one has a defined CRS will raise an error.
- **Data Type Incompatibility:** Providing a `no_data_value` that does not fit the band's data type (e.g., a float value for an integer band) will cause an error.
- **Empty Intersection:** If `lenient` is false and the geometry does not intersect the raster, the query will fail.

### Fix Code Hint
```python
# Ensure CRS alignment before clipping
sd.sql("""
    SELECT RS_Clip(
        raster_col, 
        1, 
        ST_Transform(geom_col, RS_CRS(raster_col))
    ) FROM my_table
""").to_arrow_table()
```

## API Test: `RS_Contains`

### Signature
```python
RS_Contains(arg1: raster, arg2: geometry) -> boolean | RS_Contains(arg1: geometry, arg2: raster) -> boolean | RS_Contains(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_contains.qmd:19_

### Goal
Determines if the convex hull of the first spatial argument completely contains the second spatial argument (raster or geometry).

### Parameters
- `arg1` (`raster` | `geometry`): The primary spatial object whose extent is checked for containment.
- `arg2` (`geometry` | `raster`): The secondary spatial object to be checked against the extent of the first.

### Input
- **Data Types:** Accepts `raster` and `geometry` types.
- **Preconditions:** If both arguments have defined CRSs, they must be compatible for transformation. If CRSs differ, the second argument is transformed to the CRS of the first. If transformation fails, both are transformed to WGS 84 as a fallback. If either lacks a CRS, no transformation is performed.
- **Formats:** Rasters should be loaded via `RS_FromGDALRaster` or `RS_Example()`; geometries should be standard SedonaDB types (e.g., `ST_Point`, `ST_GeomFromText`).

### Output
Returns `boolean` — `true` if the convex hull of the first argument contains the second argument, otherwise `false`.

### Valid Call Patterns
```python
# Checking raster against geometry
result = sd.sql("SELECT RS_Contains(RS_Example(), ST_Point(0.5, 0.5, 4326))").to_arrow_table()
print(result)
# __CHECK__

# Checking raster against raster
result = sd.sql("SELECT RS_Contains(RS_Example(), RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_Contains` to perform spatial containment checks between rasters and geometries.
- Always ensure that if you are comparing two objects with different CRSs, the system will attempt an automatic transformation; if you are unsure of the CRS, verify it using `RS_CRS(raster_col)`.
- When comparing two rasters, remember that the function evaluates the containment of their convex hulls.

### Prompt Snippet
```text
RS_Contains(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** If the CRS transformation fails and no fallback to WGS 84 is possible, the query may error.
- **Invalid Geometry:** Passing a null or malformed geometry as `arg2` will result in a null or error.
- **Memory Constraints:** While `RS_Contains` is efficient, performing this on massive datasets without prior filtering may impact performance; ensure spatial indexes are utilized if available.

### Fix Code Hint
```python
# Ensure CRS consistency if the function fails due to transformation errors
sd.sql("SELECT RS_Contains(RS_Reproject(raster_col, 'EPSG:4326'), geometry_col) FROM my_table")
```

## API Test: `RS_ConvexHull`

### Signature
```python
RS_ConvexHull(arg1: raster) -> geometry
```
_Source: source/docs/reference/sql/rs_convexhull.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_ConvexHull
description: Returns the convex hull geometry of a raster.
kernels:
  - returns: geometry
    args: [raster]
---

## Examples

```sql
SELECT RS_ConvexHull(RS_Example());
```


### Goal
Computes the minimum convex polygon that encloses the spatial extent of the provided raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which the convex hull geometry is derived.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must be properly initialized and contain valid spatial metadata (extent and CRS).

### Output
Returns `geometry` — A polygon representing the convex hull of the raster's spatial footprint.

### Valid Call Patterns
```python
# Example using the Python SQL interface
result = sd.sql("SELECT RS_ConvexHull(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_ConvexHull` as a standalone function within a SQL `SELECT` statement.
- Ensure the input is a valid raster object; if using a file path, wrap it in `RS_FromPath` first.

### Prompt Snippet
```text
SELECT RS_ConvexHull(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Raster:** Passing a null or malformed raster object will result in a query execution error.
- **CRS Mismatch:** While `RS_ConvexHull` operates on the raster's extent, ensure that subsequent operations involving this geometry and other spatial objects account for potential CRS differences.
- **Memory Constraints:** For extremely large or complex raster datasets, ensure the system has sufficient memory to process the geometry generation.

### Fix Code Hint
```python
# Ensure the raster is valid before processing
sd.sql("SELECT RS_ConvexHull(my_raster) FROM my_table WHERE my_raster IS NOT NULL").to_arrow_table()
```

## API Test: `RS_DimNames`

### Signature
```python
RS_DimNames(arg1: raster) -> list<utf8> | RS_DimNames(arg1: raster, band: int) -> list<utf8>
```
_Source: source/docs/reference/sql/rs_dimnames.qmd:19_

### Goal
`RS_DimNames` retrieves the ordered list of dimension names (e.g., "y", "x", "time") associated with a specific raster band or the entire raster.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): The 1-based index of the band to query. If omitted, the function checks all bands for consistent dimension names.

### Input
- **Data:** A valid `raster` object (e.g., created via `RS_FromPath` or `RS_Example`).
- **Preconditions:** If the `band` index is omitted, all bands in the raster must share identical dimension names; otherwise, the function will raise an error.

### Output
Returns `list<utf8>` — An ordered list of strings representing the dimension names, where the i-th name corresponds to the i-th dimension size returned by `RS_Shape`.

### Valid Call Patterns
```python
# Retrieve dimension names for all bands (must be consistent)
sd.sql("SELECT RS_DimNames(RS_Example())").to_arrow_table()

# Retrieve dimension names for a specific band
sd.sql("SELECT RS_DimNames(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_DimNames` to inspect the dimensionality of a raster.
- Always prefer the 1-based `band` index if the raster contains multiple bands with potentially different dimensions to avoid ambiguity errors.
- Note that this function is **Experimental** and subject to change.

### Prompt Snippet
```text
RS_DimNames(raster_col, band_index)
```

### Common Failure Modes
- **Inconsistent Dimensions:** Calling `RS_DimNames(raster)` without a band index on a multi-band raster where bands have different dimension names will trigger an error.
- **Invalid Index:** Providing a `band` index that is null or out-of-range (e.g., 0 or greater than the number of bands) will return `null`.

### Fix Code Hint
```python
# If RS_DimNames fails due to inconsistent band dimensions, specify the band explicitly:
sd.sql("SELECT RS_DimNames(my_raster, 1) FROM my_table")
```

## API Test: `RS_DimSize`

### Signature
```python
RS_DimSize(arg1: raster, dim_name: utf8) -> int64 | RS_DimSize(arg1: raster, dim_name: utf8, band: int) -> int64
```
_Source: source/docs/reference/sql/rs_dimsize.qmd:19_

### Goal
Retrieves the size of a specific named dimension (such as 'x', 'y', or 'time') from a raster or a specific band within that raster.

### Parameters
- `arg1` (`raster`): The input raster object to query.
- `dim_name` (`utf8`): The name of the dimension to retrieve (e.g., 'x', 'y', 'time').
- `band` (`int`): Optional 1-based index of the band to query. If omitted, the function attempts to resolve the dimension across all bands.

### Input
- **Data:** A valid raster object (e.g., created via `RS_Example`, `RS_FromPath`, or `RS_FromGDALRaster`).
- **Preconditions:** The dimension name must exist within the raster's metadata. Pixel coordinates and band indices are 1-based.

### Output
Returns `int64` — The integer size of the requested dimension. Returns `NULL` if a specific band index is provided but the band does not contain the requested dimension.

### Valid Call Patterns
```python
# Querying dimension size without specifying a band
result = sd.sql("SELECT RS_DimSize(RS_Example(), 'x')").to_arrow_table()
print(result) # __CHECK__

# Querying dimension size for a specific band
result = sd.sql("SELECT RS_DimSize(RS_Example(), 'y', 1)").to_arrow_table()
print(result) # __CHECK__
```

### LLM Instruction Prompt
- Use `RS_DimSize` to inspect raster dimensions when the shape is unknown or heterogeneous.
- Always prefer the 2-argument form unless the raster is known to be heterogeneous (where bands have conflicting dimension sizes), in which case use the 3-argument form with an explicit band index.
- Remember that this function is **Experimental** and may change.

### Prompt Snippet
```text
RS_DimSize(raster_column, 'dimension_name')
RS_DimSize(raster_column, 'dimension_name', band_index)
```

### Common Failure Modes
- **Dimension Mismatch:** If bands have different sizes for the same dimension and no band index is provided, the function will return an error.
- **Missing Dimension:** If the `dim_name` does not exist in the raster, the function will return an error (often indicating a typo).
- **Invalid Band Index:** Providing a null or out-of-range band index will return `NULL`.

### Fix Code Hint
```python
# If RS_DimSize fails due to heterogeneous band sizes, specify the band index:
sd.sql("SELECT RS_DimSize(my_raster, 'x', 1) FROM my_table")
```

## API Test: `RS_DimToBand`

### Signature
```python
RS_DimToBand(arg1: raster, dim_name: utf8) -> raster
```
_Source: source/docs/reference/sql/rs_dimtoband.qmd:19_

### Goal
Promotes a named non-spatial dimension within a raster into separate, individual bands to enable standard band-based analytical operations.

### Parameters
- `arg1` (`raster`): The input raster containing a non-spatial dimension (e.g., 'time' or 'wavelength') that needs to be expanded into separate bands.
- `dim_name` (`utf8`): The name of the dimension to promote. This must match a dimension existing within the raster's metadata.

### Input
- **Data:** A `raster` object, typically loaded via `RS_FromPath` or `RS_FromGDALRaster`.
- **Preconditions:** The raster must contain the specified `dim_name` as a non-spatial dimension. Spatial dimensions (`x_dim`, `y_dim`) cannot be promoted.
- **Compatibility:** If the raster was previously transformed using `RS_BandToDim`, this function acts as the inverse operation.

### Output
Returns `raster` — A new raster where each index of the promoted dimension has been converted into a distinct band, with the original dimension removed.

### Valid Call Patterns
```python
# Example using the SedonaDB SQL interface
query = "SELECT RS_DimToBand(RS_BandToDim(RS_Example(), 'band'), 'band')"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `RS_DimToBand` when you need to perform band-specific math on data that is currently stored as a dimension within a single band.
- Always ensure the `dim_name` provided matches the dimension present in the raster metadata; otherwise, the function will raise an error.
- Remember that this is an **experimental** feature and behavior may change.

### Prompt Snippet
```text
RS_DimToBand(raster_column, 'dimension_name')
```

### Common Failure Modes
- **Missing Dimension:** If the `dim_name` does not exist in the raster, the function will error. This is often caused by a typo in the dimension name.
- **Spatial Dimension Promotion:** Attempting to promote `x_dim` or `y_dim` is not supported and will fail.
- **Experimental Status:** As an experimental function, unexpected behavior may occur with complex or non-standard raster metadata structures.

### Fix Code Hint
```python
# Verify the dimension name exists in the metadata before calling
# If the error persists, ensure the raster was correctly prepared via RS_BandToDim
result = sd.sql("SELECT RS_DimToBand(my_raster, 'correct_dim_name')").to_arrow_table()
```

## API Test: `RS_Envelope`

### Signature
```python
RS_Envelope(arg1: raster) -> geometry
```
_Source: source/docs/reference/sql/rs_envelope.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_Envelope
description: Returns the envelope (bounding box) of a raster as a geometry.
kernels:
  - returns: geometry
    args: [raster]
---

## Examples

```sql
SELECT RS_Envelope(RS_Example());
```


### Goal
Calculates the spatial extent of a raster object and returns it as a standard geometry (bounding box).

### Parameters
- `arg1` (`raster`): The input raster object from which the bounding box geometry is derived.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must be correctly initialized within the SedonaDB session.

### Output
Returns `geometry` — A polygon representing the rectangular bounding box (envelope) of the input raster, defined by its spatial extent.

### Valid Call Patterns
```python
# Verify the envelope of an example raster
result = sd.sql("SELECT RS_Envelope(RS_Example())").to_arrow_table()
print(result)
# __CHECK__ witness: The result is a geometry object representing the raster bounds.

# Use the envelope to define a new empty raster
sd.sql("SELECT RS_Width(RS_MakeEmptyRaster(1, 16, 8, RS_Envelope(RS_Example())))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_Envelope` to extract the spatial boundary of a raster when performing operations that require a geometry input, such as `RS_MakeEmptyRaster` or spatial joins.
- Ensure the input is a valid `raster` type; do not pass geometry types directly to this function.

### Prompt Snippet
```text
RS_Envelope(raster_column)
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type (e.g., a geometry or string) will result in a SQL execution error.
- **Null Input:** If the input raster is null, the function will return null.
- **CRS Mismatch:** While `RS_Envelope` returns the geometry of the raster, subsequent operations using this geometry (like `RS_Clip`) may fail if the CRS of the envelope does not match the CRS of the target raster.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling
sd.sql("SELECT RS_Envelope(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL")
```

## API Test: `RS_Example`

### Signature
```python
RS_Example() -> raster
```
_Source: source/docs/reference/sql/rs_example.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_Example
description: Creates a simple example raster for testing and demos.
kernels:
  - returns: raster
    args: []
---


### Goal
Generates a synthetic, pre-defined raster object within the SedonaDB engine for the purpose of testing, demonstration, and verifying raster processing workflows.

### Parameters
_None._

### Input
No input parameters are required. This function is self-contained and generates its own internal raster data.

### Output
Returns `raster` — A materialized raster object containing sample pixel data, which can be used as an input for other SedonaDB raster functions like `RS_Clip`, `RS_Resample`, or `RS_AsGeoTiff`.

### Valid Call Patterns
```python
# Verify the raster can be generated and exported
result = sd.sql("SELECT RS_AsGeoTiff(RS_Example()) IS NOT NULL").to_arrow_table()
print(result)
# __CHECK__ witness: True
```

### LLM Instruction Prompt
- Use `RS_Example()` as a standalone function in a `SELECT` statement when a raster input is required for testing or demonstrating SedonaDB raster functions.
- Do not attempt to pass arguments to this function, as it takes none.

### Prompt Snippet
```text
SELECT RS_Example()
```

### Common Failure Modes
- **Type Mismatch:** Attempting to use the output of `RS_Example()` in functions that strictly require a specific raster band type (e.g., `Int64` or `UInt64` rasters are not supported for resampling/warping; cast to `Int32` or `Float64` first).
- **Contextual Errors:** If used in a complex query involving other rasters, ensure that the CRS of the generated `RS_Example()` raster is compatible with other inputs, as SedonaDB requires matching CRSs for operations like `RS_Clip` or `RS_ReprojectMatch`.

### Fix Code Hint
```python
# If resampling fails due to type, cast the example raster first
sd.sql("SELECT RS_Resample(CAST(RS_Example() AS 'Float64'), 32, 16, false, 'NearestNeighbor')")
```

## API Test: `RS_FromGDALRaster`

### Signature
```python
RS_FromGDALRaster(content: binary) -> raster
```
_Source: source/docs/reference/sql/rs_fromgdalraster.qmd:19_

### Goal
`RS_FromGDALRaster` decodes binary-encoded raster data into an in-database raster object with all band data materialized inline.

### Parameters
- `content` (`binary`): The raw byte sequence of a raster file in any format supported by GDAL (e.g., GeoTIFF, PNG).

### Input
- **Data:** Binary data representing a raster image.
- **Preconditions:** The input must be a valid, GDAL-readable binary stream. If the input is `NULL`, the function returns `NULL`.
- **Formats:** Supports common formats like GeoTIFF and PNG; specific format support depends on the underlying GDAL installation.

### Output
Returns `raster` — An in-database raster object containing the materialized band data, suitable for further spatial analysis or map algebra within SedonaDB.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Round-trip verification using an example raster
query = "SELECT RS_Width(RS_FromGDALRaster(RS_AsGeoTiff(RS_Example())));"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Use `RS_FromGDALRaster` when you have binary raster data (e.g., from a database column or file read) that needs to be converted into a SedonaDB-compatible raster object for processing.
- Always ensure the input is a `binary` type.
- Remember that this function materializes data inline; for large datasets, consider if `RS_FromPath` is more appropriate for out-db referencing.

### Prompt Snippet
```text
RS_FromGDALRaster(binary_column_name)
```

### Common Failure Modes
- **Invalid Binary:** Passing non-raster binary data or corrupted file bytes will result in a runtime error during parsing.
- **NULL Input:** Passing a `NULL` value is valid and will return `NULL`, which may cause downstream errors if not handled with `COALESCE` or `WHERE` clauses.
- **Memory Constraints:** Because this function materializes the entire raster inline, processing extremely large binary blobs may lead to memory pressure on the single-node engine.

### Fix Code Hint
```python
# Ensure the input is not NULL before processing
query = """
SELECT RS_Width(RS_FromGDALRaster(raster_binary)) 
FROM my_table 
WHERE raster_binary IS NOT NULL
"""
```

## API Test: `RS_FromPath`

### Signature
```python
RS_FromPath(path: string) -> raster
```
_Source: source/docs/reference/sql/rs_frompath.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_FromPath
description: Creates an out-of-database raster from a raster file path.
kernels:
  - returns: raster
    args:
    - name: path
      type: string
---

## Description

Loads raster metadata from the file at `path` and returns a raster whose bands
reference the source file as out-db bands.

This is useful when you want to work with rasters stored on disk without copying
their pixel data into the raster value itself.

## Examples

```sql
SELECT RS_BandPath(RS_FromPath('../../../submodules/sedona-testing/data/raster/test4.tiff'));
```


### Goal
Creates an out-of-database raster object by referencing a file on disk, allowing for efficient analysis without loading full pixel data into memory.

### Parameters
- `path` (`string`): A file system path or URI pointing to a GDAL-readable raster file (e.g., GeoTIFF, PNG).

### Input
- **Data:** A valid path to a raster file accessible by the SedonaDB engine.
- **Preconditions:** The file must be in a format supported by GDAL. Because this creates an "out-db" raster, the file must remain accessible at the provided path for the duration of any subsequent operations that require reading pixel values.

### Output
Returns `raster` — A raster object containing metadata and references to the source file bands.

### Valid Call Patterns
```python
# Example using the Python SQL connection
query = "SELECT RS_BandPath(RS_FromPath('../../../submodules/sedona-testing/data/raster/test4.tiff'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_FromPath` when the goal is to reference large raster files on disk to avoid memory overhead.
- Ensure the path provided is a string literal or a valid column reference.
- Do not use this function if the raster data is transient or if the file will be deleted before the query executes.

### Prompt Snippet
```text
Use RS_FromPath(path) to create an out-of-database raster reference.
```

### Common Failure Modes
- **File Not Found:** The engine cannot locate the file at the provided path, resulting in a runtime error.
- **Unsupported Format:** The file at the path is not a GDAL-readable format.
- **Permission Denied:** The SedonaDB process lacks read permissions for the specified file path.
- **Broken Reference:** If the file is moved or deleted after `RS_FromPath` is called, subsequent operations (like `RS_Value` or `RS_Clip`) will fail when they attempt to access the pixel data.

### Fix Code Hint
```python
# Ensure the path is absolute or relative to the SedonaDB working directory
# and verify file existence before execution.
import os
path = "../../../submodules/sedona-testing/data/raster/test4.tiff"
assert os.path.exists(path), "Raster file not found"
sd.sql(f"SELECT RS_FromPath('{path}')").to_arrow_table()
```

## API Test: `RS_GeoReference`

### Signature
```python
RS_GeoReference(arg1: raster) -> utf8 | RS_GeoReference(arg1: raster, format: utf8) -> utf8
```
_Source: source/docs/reference/sql/rs_georeference.qmd:19_

### Goal
Returns the georeference metadata of a raster as a six-line string in either GDAL or ESRI world file format.

### Parameters
- `arg1` (`raster`): The input raster object to extract georeferencing metadata from.
- `format` (`utf8`): The output format style. Use 'GDAL' (or 'PIXEL') for the upper-left corner of the upper-left pixel, or 'ESRI' (or 'NODE') to shift coordinates to the center of the upper-left pixel. Defaults to 'GDAL'.

### Input
The function accepts a `raster` object created via SedonaDB raster functions (e.g., `RS_FromPath`, `RS_MakeEmptyRaster`). The input must be a valid, initialized raster.

### Output
Returns `utf8` — A string containing six lines representing the affine transformation parameters: `scalex`, `skewy`, `skewx`, `scaley`, `upperleftx`, and `upperlefty`.

### Valid Call Patterns
```python
# Basic usage with default GDAL format
result = sd.sql("SELECT RS_GeoReference(RS_Example())").to_arrow_table()
print(result)
# __CHECK__

# Explicitly specifying ESRI format
result = sd.sql("SELECT RS_GeoReference(RS_Example(), 'ESRI')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_GeoReference` to extract affine transformation metadata from a raster.
- Always specify the `format` argument as 'GDAL' or 'ESRI' if a specific coordinate alignment (corner vs. center) is required.
- Ensure the input is a valid `raster` type; if the raster is not yet loaded, use `RS_FromPath` or `RS_MakeEmptyRaster` first.

### Prompt Snippet
```text
RS_GeoReference(raster_column, 'ESRI')
```

### Common Failure Modes
- **Invalid Raster Input:** Passing a non-raster type or a null value will result in a SQL execution error.
- **Unsupported Format String:** Providing a format string other than 'GDAL', 'PIXEL', 'ESRI', or 'NODE' may result in unexpected behavior or errors.
- **Memory Constraints:** While `RS_GeoReference` is metadata-focused, ensure the input raster is properly registered in the SedonaDB session.

### Fix Code Hint
```python
# Ensure the raster is valid before calling
sd.sql("SELECT RS_GeoReference(r) FROM (SELECT RS_MakeEmptyRaster(...) as r)").to_arrow_table()
```

## API Test: `RS_GeoTransform`

### Signature
```python
RS_GeoTransform(arg1: raster) -> struct
```
_Source: source/docs/reference/sql/rs_geotransform.qmd:19_

### Goal
Decomposes a raster's affine geotransformation matrix into a structured set of six human-readable spatial parameters.

### Parameters
- `arg1` (`raster`): The input raster object for which the geotransform decomposition is requested.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`).
- **Preconditions:** The raster must be properly initialized with spatial metadata. If the raster lacks a defined geotransform, the function may return default or null values depending on the underlying GDAL metadata state.

### Output
Returns `struct` — A structure containing six double-precision values:
- `magnitudeI`, `magnitudeJ`: Pixel sizes along the transformed i and j axes.
- `thetaI`: Rotation of the raster in radians (positive clockwise).
- `thetaIJ`: Angle from the transformed i axis to the transformed j axis in radians (positive counter-clockwise).
- `offsetX`, `offsetY`: The coordinates of the upper-left corner of the upper-left pixel.

### Valid Call Patterns
```python
# Using the Python SQL interface to verify the geotransform structure
result = sd.sql("SELECT RS_GeoTransform(RS_Example())").to_arrow_table()
print(result)
# __CHECK__ witness: The result should contain a struct column with 6 fields.
```

### LLM Instruction Prompt
- Use `RS_GeoTransform` when you need to extract the spatial orientation and pixel dimensions of a raster in a decomposed format.
- Do not use this function if you require raw affine coefficients; use `RS_ScaleX`, `RS_SkewX`, `RS_UpperLeftX`, or `RS_GeoReference` instead.
- Always ensure the input is a valid `raster` type.

### Prompt Snippet
```text
SELECT RS_GeoTransform(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input:** Passing a geometry or non-raster type will result in a SQL execution error.
- **Missing Metadata:** If the raster was created without spatial reference information, the returned struct may contain zeroed or default values.
- **Type Mismatch:** Attempting to use this on a null raster input will return null.

### Fix Code Hint
```python
# Ensure the raster is valid before calling
sd.sql("SELECT RS_GeoTransform(my_raster) FROM my_table WHERE my_raster IS NOT NULL")
```

## API Test: `RS_Height`

### Signature
```python
RS_Height(arg1: raster) -> bigint
```
_Source: source/docs/reference/sql/rs_height.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_Height
description: Returns the height of a raster in pixels.
kernels:
  - returns: bigint
    args: [raster]
---

## Examples

```sql
SELECT RS_Height(RS_Example());
```


### Goal
Returns the vertical dimension of a raster object measured in pixels.

### Parameters
- `arg1` (`raster`): The input raster object for which the height is to be calculated.

### Input
The input must be a valid `raster` object. This can be generated via functions like `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`. No specific CRS preconditions are required for this metadata function.

### Output
Returns `bigint` — The number of pixels along the vertical axis (height) of the raster.

### Valid Call Patterns
```python
# Verify the height of a sample raster
result = sd.sql("SELECT RS_Height(RS_Example()) as height").to_arrow_table()
assert result["height"][0] > 0
print(f"__CHECK__ Height: {result['height'][0]}")
```

### LLM Instruction Prompt
- Use `RS_Height(raster_expression)` to retrieve the vertical pixel count.
- Ensure the input is a valid raster type; do not pass geometry types or raw binary data directly without conversion.

### Prompt Snippet
```text
SELECT RS_Height(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-raster type (e.g., a geometry or string) will result in a SQL execution error.
- Passing a null value will return null.
- If the raster object is malformed or failed to load (e.g., invalid file path in `RS_FromPath`), the function will fail.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling RS_Height
sd.sql("SELECT RS_Height(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `RS_Intersects`

### Signature
```python
RS_Intersects(arg1: raster, arg2: geometry) -> boolean | RS_Intersects(arg1: geometry, arg2: raster) -> boolean | RS_Intersects(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_intersects.qmd:19_

### Goal
Determines if the convex hull of a raster intersects with a geometry or another raster's convex hull to perform spatial filtering.

### Parameters
- `arg1` (`raster`): The primary raster object to evaluate.
- `arg2` (`geometry`): The geometry object (or a second raster) to check for intersection against the first argument.

### Input
- **Data:** Raster objects (e.g., from `RS_FromGDALRaster` or `RS_Example()`) and geometry objects (e.g., `ST_Point`, `ST_Polygon`).
- **Preconditions:** If both inputs have a CRS, they must be compatible for transformation. If only one has a CRS, the operation proceeds without transformation. If neither has a CRS, the comparison is performed directly.

### Output
Returns `boolean` — `true` if the convex hulls of the two inputs intersect, `false` otherwise.

### Valid Call Patterns
```python
# Raster and Geometry intersection
sd.sql("SELECT RS_Intersects(RS_Example(), ST_Point(0.5, 0.5, 4326))").to_arrow_table()

# Raster and Raster intersection
sd.sql("SELECT RS_Intersects(RS_Example(), RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Intersects` to perform spatial predicate filtering between rasters and geometries.
- Always ensure that if you are comparing across different CRSs, the inputs are valid for transformation; otherwise, ensure they share a common CRS or have no CRS defined.
- When comparing two rasters, the function implicitly uses their convex hulls.

### Prompt Snippet
```text
RS_Intersects(raster_input, geometry_input)
```

### Common Failure Modes
- **CRS Mismatch:** If both inputs have defined CRSs that cannot be transformed, the query may fail.
- **Invalid Raster:** Passing a non-raster object as the raster argument will result in a SQL execution error.
- **Memory Constraints:** While `RS_Intersects` is a predicate, ensure the input rasters are properly registered in the SedonaDB session.

### Fix Code Hint
```python
# Ensure CRS consistency if intersection returns unexpected results
sd.sql("SELECT RS_Intersects(RS_Reproject(my_raster, 'EPSG:4326'), my_geometry)").to_arrow_table()
```

## API Test: `RS_IsEmpty`

### Signature
```python
RS_IsEmpty(arg1: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_isempty.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_IsEmpty
description: |
  Returns true when the raster has zero visible volume — some dimension has
  length 0, so it holds no cells. This is visible-volume emptiness, not
  byte-emptiness: a lazy raster whose pixels are resolved on demand carries
  no bytes but is not empty.
kernels:
  - returns: boolean
    args: [raster]
---

## Examples

```sql
SELECT RS_IsEmpty(RS_Example());
```


### Goal
Determines if a raster object has zero visible volume by checking if any of its dimensions have a length of zero.

### Parameters
- `arg1` (`raster`): The input raster object to evaluate for visible-volume emptiness.

### Input
- **Data:** A valid raster object (e.g., created via `RS_MakeEmptyRaster`, `RS_FromPath`, or `RS_FromGDALRaster`).
- **Preconditions:** The input must be a valid raster type. Note that this function checks for dimensional emptiness, not whether the raster contains data bytes (a lazy raster with no loaded bytes is not considered empty if its dimensions are non-zero).

### Output
Returns `boolean` — `true` if the raster has at least one dimension with length 0 (no cells), otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify emptiness of a raster
result = sd.sql("SELECT RS_IsEmpty(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_IsEmpty` to validate if a raster has valid dimensions before performing operations like `RS_Clip` or `RS_ZonalStats`.
- Do not confuse this with byte-level emptiness; a raster can be "empty" in terms of volume but still be a valid object in the SedonaDB engine.

### Prompt Snippet
```text
RS_IsEmpty(raster_column)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a geometry or non-raster type to the function will result in a SQL execution error.
- **Null Input:** Passing a `NULL` raster will return `NULL` rather than a boolean.
- **Misinterpretation:** Expecting `RS_IsEmpty` to return `true` for rasters that are initialized but contain no pixel data (e.g., a raster with dimensions 10x10 but no assigned values); this function only checks if the dimensions themselves are zero.

### Fix Code Hint
```python
# Ensure the raster is not empty before processing
df = sd.sql("SELECT * FROM raster_table WHERE NOT RS_IsEmpty(raster_col)")
```

## API Test: `RS_MakeEmptyRaster`

### Signature
```python
RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, extent: geometry) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, extent: geometry) -> raster | RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, upper_left_x: double, upper_left_y: double, cell_size: double) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, upper_left_x: double, upper_left_y: double, cell_size: double) -> raster | RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, upper_left_x: double, upper_left_y: double, scale_x: double, scale_y: double, skew_x: double, skew_y: double, srid: integer) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, upper_left_x: double, upper_left_y: double, scale_x: double, scale_y: double, skew_x: double, skew_y: double, srid: integer) -> raster
```

### Goal
Creates an in-database, zero-filled raster template from a grid definition, useful as a target for rasterization or as a spatial reference template.

### Parameters
- `num_bands` (`integer`): Number of zero-filled bands to create; use `0` for a bandless grid template.
- `band_data_type` (`string`): Pixel type (e.g., 'uint8', 'float64', 'I', 'D'). Defaults to 'float64'.
- `width` (`integer`): Grid width in pixels.
- `height` (`integer`): Grid height in pixels.
- `upper_left_x` (`double`): X coordinate of the upper-left corner of the upper-left pixel.
- `upper_left_y` (`double`): Y coordinate of the upper-left corner of the upper-left pixel.
- `scale_x` (`double`): Pixel width.
- `scale_y` (`double`): Pixel height (negative for north-up rasters).
- `skew_x` (`double`): X-axis skew.
- `skew_y` (`double`): Y-axis skew.
- `srid` (`integer`): EPSG code of the raster's CRS; `0` for no CRS.

### Input
Requires a grid definition provided via an extent geometry (SedonaDB extension) or explicit affine transform parameters (Sedona Spark parity). The extent geometry must have a positive width and height.

### Output
Returns a `raster` object containing the specified number of zero-filled bands.

### Valid Call Patterns
```python
# Create a 400x400 uint8 raster from an extent
query = """
SELECT RS_MakeEmptyRaster(1, 'uint8', 400, 400, ST_MakeEnvelope(0, 0, 400, 400, 4326))
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `RS_MakeEmptyRaster` to initialize grids for `RS_AsRaster`.
- If using the extent-based signature, ensure the geometry is valid and has a defined CRS.
- When using the affine transform signature, ensure `scale_y` is negative for standard north-up rasters.

### Prompt Snippet
```text
RS_MakeEmptyRaster(num_bands, width, height, extent)
```

### Common Failure Modes
- **Invalid Data Type:** SedonaDB raises an error for unrecognized `band_data_type` strings, unlike Sedona Spark which defaults to `float64`.
- **Memory Limits:** Creating very large rasters (e.g., high resolution) can exceed available memory as each band is materialized in-database.
- **CRS Mismatch:** If using an extent geometry, ensure the CRS is compatible with other spatial operations performed on the resulting raster.

### Fix Code Hint
```python
# Ensure band_data_type is a valid string if provided
# Use 0 for num_bands if only a spatial template is needed
raster = sd.sql("SELECT RS_MakeEmptyRaster(1, 'float64', 100, 100, ST_MakeEnvelope(0,0,10,10,4326))").to_arrow_table()
```

## API Test: `RS_MetaData`

### Signature
```python
RS_MetaData(arg1: raster) -> struct
```
_Source: source/docs/reference/sql/rs_metadata.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_MetaData
description: Returns raster metadata as a struct.
kernels:
  - returns: struct
    args: [raster]
---

## Description

`RS_MetaData()` returns a struct containing:

- `upperLeftX`, `upperLeftY`: origin of the raster geotransform
- `gridWidth`, `gridHeight`: raster dimensions in pixels
- `scaleX`, `scaleY`: pixel scale
- `skewX`, `skewY`: geotransform skew terms
- `srid`: raster SRID if available, otherwise `0`
- `numSampleDimensions`: band count
- `tileWidth`, `tileHeight`: GDAL block size derived tile dimensions

For rasters with no bands, `tileWidth` and `tileHeight` are `0`.

## Examples

```sql
SELECT RS_MetaData(RS_Example());
```

```sql
SELECT meta."gridWidth", meta."gridHeight"
FROM (
  SELECT RS_MetaData(
    RS_FromPath('https://download.osgeo.org/geotiff/samples/gdal_eg/cea.tif')
  ) AS meta
) q;
```


### Goal
Extracts comprehensive georeferencing, dimensional, and tiling metadata from a raster object into a structured format for inspection.

### Parameters
- `arg1` (`raster`): The input raster object to be inspected.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`).
- **Preconditions:** The raster must be successfully loaded into the SedonaDB engine.

### Output
Returns `struct` — A structured object containing the raster's geotransform parameters (origin, scale, skew), dimensions, SRID, band count, and GDAL-derived tile dimensions.

### Valid Call Patterns
```python
# Example using the Python SQL connection
import sedona.db
sd = sedona.db.connect()

# Verify metadata extraction
result = sd.sql("SELECT RS_MetaData(RS_Example()) as meta").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_MetaData(raster_expression)` within a SQL query context via `sd.sql()`.
- Access individual fields of the returned struct using standard SQL dot notation or alias selection (e.g., `meta."gridWidth"`).
- Ensure the input is a valid raster type; do not pass geometry types to this function.

### Prompt Snippet
```text
SELECT RS_MetaData(my_raster_column) FROM my_table;
```

### Common Failure Modes
- **Invalid Input:** Passing a geometry or non-raster type will result in a SQL execution error.
- **Null Input:** Passing a `NULL` raster will return `NULL`.
- **Accessing Non-existent Fields:** Attempting to access fields not defined in the struct (e.g., `meta."invalidField"`) will cause a query failure.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling metadata
sd.sql("SELECT RS_MetaData(raster_col) FROM my_table WHERE raster_col IS NOT NULL")
```

## API Test: `RS_MinConvexHull`

### Signature
```python
RS_MinConvexHull(arg1: raster) -> geometry | RS_MinConvexHull(arg1: raster, band: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_minconvexhull.qmd:19_

### Goal
Calculates the footprint of a raster by returning a polygon representing the smallest pixel-aligned rectangle that encompasses all pixels containing valid data (excluding nodata values).

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `band` (`integer`): The 1-based index of the band to evaluate. If omitted, the function considers a pixel to contain data if any band contains valid data at that location.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must have defined nodata values if trimming is desired; otherwise, the function returns the full grid extent. Only 2-D rasters are supported.

### Output
Returns `geometry` — A polygon representing the bounding box of the valid data pixels, aligned to the raster's grid axes. Returns `null` if no pixels contain valid data.

### Valid Call Patterns
```python
# Basic usage on a raster
sd.sql("SELECT RS_MinConvexHull(RS_Example())").to_arrow_table()

# Usage with a specific band and nodata configuration
sd.sql("SELECT RS_MinConvexHull(RS_SetBandNoDataValue(RS_Example(), 1, 1), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_MinConvexHull` when you need to determine the spatial extent of valid data within a raster grid.
- Always specify the band index if you need to isolate the footprint of a specific data layer.
- Remember that the result is always axis-aligned to the raster grid, not necessarily the minimum bounding box of the data values themselves.

### Prompt Snippet
```text
RS_MinConvexHull(raster_column)
RS_MinConvexHull(raster_column, band_index)
```

### Common Failure Modes
- **Unsupported Dimensions:** Passing a multi-dimensional (non 2-D) raster will result in an error.
- **Missing Nodata:** If a band does not have a nodata value defined, the function will treat all pixels as valid data, returning the full extent of the raster.
- **Null Results:** If the raster is entirely composed of nodata values, the function returns `null`.

### Fix Code Hint
```python
# Ensure the raster has a nodata value set if you want to trim the footprint
# Use RS_SetBandNoDataValue before calling RS_MinConvexHull
query = "SELECT RS_MinConvexHull(RS_SetBandNoDataValue(my_raster, 1, 0), 1)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `RS_NumBands`

### Signature
```python
RS_NumBands(arg1: raster) -> uint32
```
_Source: source/docs/reference/sql/rs_numbands.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_NumBands
description: Returns the number of bands in the raster.
kernels:
  - returns: uint32
    args: [raster]
---

## Examples

```sql
SELECT RS_NumBands(RS_Example());
```


### Goal
Returns the total count of spectral or data bands contained within a given raster object.

### Parameters
- `arg1` (`raster`): The input raster object to be inspected.

### Input
The input must be a valid raster object, typically created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`. No specific file format constraints exist beyond the requirement that the input is a valid SedonaDB raster type.

### Output
Returns `uint32` — An unsigned 32-bit integer representing the number of bands present in the raster.

### Valid Call Patterns
```python
# Verify the number of bands in a sample raster
result = sd.sql("SELECT RS_NumBands(RS_Example())").to_arrow_table()
assert result.column(0)[0].as_py() > 0
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `RS_NumBands(raster_expression)` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster type; if the input is a path, wrap it in a raster-loading function first.

### Prompt Snippet
```text
SELECT RS_NumBands(<raster_column_or_function>)
```

### Common Failure Modes
- Passing a non-raster type (e.g., a geometry or string) to the function will result in a SQL execution error.
- Attempting to call the function on a null raster input may return null or cause an evaluation error depending on the query context.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
sd.sql("SELECT RS_NumBands(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `RS_NumDimensions`

### Signature
```python
RS_NumDimensions(arg1: raster) -> int32 | RS_NumDimensions(arg1: raster, band: int) -> int32
```
_Source: source/docs/reference/sql/rs_numdimensions.qmd:19_

### Goal
Returns the number of dimensions (e.g., x, y, time, pressure) associated with a specific raster band or the entire raster.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): Optional 1-based index of the specific band to query.

### Input
- **Data:** A valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Preconditions:** If the `band` index is omitted, all bands in the raster must share the same number of dimensions; otherwise, the function will return an error.

### Output
Returns `int32` — The count of dimensions present in the specified band or the raster. Returns `null` if the provided band index is out of range or null.

### Valid Call Patterns
```python
# Querying dimensions for the default band configuration
sd.sql("SELECT RS_NumDimensions(RS_Example())").to_arrow_table()

# Querying dimensions for a specific band (1-based index)
sd.sql("SELECT RS_NumDimensions(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_NumDimensions` to determine the dimensionality of a raster.
- If the raster contains multiple bands with potentially different dimensions, always specify the `band` index to avoid ambiguity errors.
- Note that this function is **Experimental** and subject to change.

### Prompt Snippet
```text
RS_NumDimensions(raster_column)
RS_NumDimensions(raster_column, band_index)
```

### Common Failure Modes
- **Ambiguity Error:** Calling `RS_NumDimensions(raster)` on a multi-band raster where bands have different dimension counts. **Fix:** Specify the band index explicitly.
- **Out-of-Range Index:** Providing a `band` index that does not exist in the raster returns `null`.
- **Experimental Status:** As an experimental feature, behavior may vary across versions; ensure the environment is updated to the latest SedonaDB release.

### Fix Code Hint
```python
# If RS_NumDimensions(raster) fails due to inconsistent band dimensions:
# Explicitly query the dimension of the first band
sd.sql("SELECT RS_NumDimensions(my_raster, 1) FROM my_table").to_arrow_table()
```

__CHECK__

## API Test: `RS_PixelAsCentroid`

### Signature
```python
RS_PixelAsCentroid(arg1: raster, colX: integer, rowY: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_pixelascentroid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_PixelAsCentroid
description: Returns the centroid of the specified pixel as a Point geometry.
kernels:
  - returns: geometry
    args:
    - raster
    - name: colX
      type: integer
    - name: rowY
      type: integer
---

## Examples

```sql
SELECT RS_PixelAsCentroid(RS_Example(), 1, 1);
```


### Goal
Calculates the geographic coordinate of the center of a specific pixel within a raster, returning it as a `Point` geometry.

### Parameters
- `arg1` (`raster`): The source raster object from which to extract the pixel location.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
The function requires a valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`). The `colX` and `rowY` parameters must be within the bounds of the raster's dimensions (as defined by `RS_Width` and `RS_Height`). Note that SedonaDB uses 1-based indexing for pixel coordinates.

### Output
Returns a `geometry` object representing the `Point` at the center of the specified pixel, projected in the coordinate system of the input raster.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the centroid of the top-left pixel (1, 1)
query = "SELECT RS_PixelAsCentroid(RS_Example(), 1, 1) as geom"
result = sd.sql(query).to_arrow_table()

print("__CHECK__")
print(result)
```

### LLM Instruction Prompt
- Always use 1-based indexing for `colX` and `rowY`.
- Ensure the input `raster` is a valid SedonaDB raster object.
- The returned geometry will be in the same CRS as the input raster.

### Prompt Snippet
```text
RS_PixelAsCentroid(raster_column, col_index, row_index)
```

### Common Failure Modes
- **Out of Bounds:** Providing `colX` or `rowY` values that exceed the raster's width or height will result in an error or null geometry.
- **Indexing Error:** Using 0-based indexing (common in other libraries) will return the incorrect pixel or fail, as SedonaDB expects 1-based indexing.
- **Invalid Raster:** Passing a non-raster type or a null raster object will cause the query to fail.

### Fix Code Hint
```python
# Ensure indices are 1-based and within bounds
# Check dimensions first if unsure:
# SELECT RS_Width(raster), RS_Height(raster) FROM table;
```

## API Test: `RS_PixelAsPoint`

### Signature
```python
RS_PixelAsPoint(arg1: raster, colX: integer, rowY: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_pixelaspoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_PixelAsPoint
description: Returns the upper-left corner of the specified pixel as a Point geometry.
kernels:
  - returns: geometry
    args:
    - raster
    - name: colX
      type: integer
    - name: rowY
      type: integer
---

## Examples

```sql
SELECT RS_PixelAsPoint(RS_Example(), 1, 1);
```


### Goal
`RS_PixelAsPoint` calculates the spatial coordinate of the upper-left corner of a specific pixel within a raster, returning it as a Point geometry.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the pixel coordinate.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
The function requires a valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`). Pixel coordinates are 1-based; providing indices outside the raster dimensions will result in an error or undefined behavior.

### Output
Returns a `geometry` object representing the Point at the upper-left corner of the specified pixel in the raster's coordinate reference system.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the upper-left corner of the first pixel
query = "SELECT RS_PixelAsPoint(RS_Example(), 1, 1) as geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always use 1-based indexing for `colX` and `rowY`.
- Ensure the input `arg1` is a valid raster object.
- The returned geometry will inherit the CRS of the input raster.

### Prompt Snippet
```text
RS_PixelAsPoint(raster_object, column_index, row_index)
```

### Common Failure Modes
- **Out-of-bounds indices:** Providing `colX` or `rowY` values that exceed the raster's width or height.
- **Invalid Raster:** Passing a null or non-raster object as the first argument.
- **Indexing Errors:** Attempting to use 0-based indexing, which is not supported by SedonaDB raster functions.

### Fix Code Hint
```python
# Ensure indices are within bounds (1 to width/height)
# Check raster dimensions first if indices are dynamic
query = "SELECT RS_PixelAsPoint(my_raster, 1, 1) FROM my_table"
```

## API Test: `RS_PixelAsPolygon`

### Signature
```python
RS_PixelAsPolygon(arg1: raster, colX: integer, rowY: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_pixelaspolygon.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_PixelAsPolygon
description: Returns the bounding polygon of the specified pixel.
kernels:
  - returns: geometry
    args:
    - raster
    - name: colX
      type: integer
    - name: rowY
      type: integer
---

## Examples

```sql
SELECT RS_PixelAsPolygon(RS_Example(), 1, 1);
```


### Goal
Returns the spatial bounding polygon geometry corresponding to the extent of a specific pixel identified by its column and row index within a raster.

### Parameters
- `arg1` (`raster`): The source raster object from which the pixel geometry is derived.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
The input must be a valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`). The `colX` and `rowY` parameters must be within the bounds of the raster's dimensions (1-based indexing).

### Output
Returns a `geometry` object representing the polygon boundary of the specified pixel in the coordinate reference system of the input raster.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the geometry of the top-left pixel
query = "SELECT RS_PixelAsPolygon(RS_Example(), 1, 1) as geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always use 1-based indexing for `colX` and `rowY`.
- Ensure the input is a valid raster object.
- The function returns a geometry; ensure downstream operations are compatible with polygon types.

### Prompt Snippet
```text
RS_PixelAsPolygon(raster_column, col_index, row_index)
```

### Common Failure Modes
- **Out of Bounds:** Providing `colX` or `rowY` values that exceed the raster's width or height will result in an error or undefined behavior.
- **Invalid Raster:** Passing a null or non-raster object as `arg1`.
- **Indexing Error:** Attempting to use 0-based indexing (SedonaDB raster functions use 1-based indexing).

### Fix Code Hint
```python
# Ensure indices are within bounds using RS_Width and RS_Height
# Example: 1 <= colX <= RS_Width(raster)
```

## API Test: `RS_Polygonize`

### Signature
```python
RS_Polygonize(arg1: raster, band: integer) -> list
```
_Source: source/docs/reference/sql/rs_polygonize.qmd:19_

### Goal
Converts a specific band of a raster into a set of vector polygons, where each polygon represents a contiguous region of pixels sharing the same value.

### Parameters
- `arg1` (`raster`): The input raster object to be processed.
- `band` (`integer`): The 1-based index of the raster band to polygonize.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromGDALRaster` or `RS_FromPath`).
- **Preconditions:** The raster must contain the specified band index. Pixel coordinates are 1-based.

### Output
Returns `list` — A list of objects where each item contains:
- `geom`: The polygon geometry encoded as WKB (Well-Known Binary).
- `value`: The pixel value associated with that specific connected region.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Execute RS_Polygonize via SQL
result = sd.sql("SELECT RS_Polygonize(RS_Example(), 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_Polygonize` when converting raster-based categorical data (like land cover classes) into vector format for spatial analysis.
- Always specify the band index as an integer.
- Ensure the input raster is valid and accessible within the SedonaDB session.

### Prompt Snippet
```text
SELECT RS_Polygonize(raster_column, 1) FROM my_raster_table;
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index that does not exist in the raster (e.g., 0 or a number greater than `RS_NumBands`).
- **Memory Constraints:** Polygonizing very large, high-resolution rasters with many unique pixel regions can consume significant memory.
- **Unsupported Raster Types:** Ensure the raster data is in a format compatible with SedonaDB's raster processing engine.

### Fix Code Hint
```python
# Verify band count before polygonizing
sd.sql("SELECT RS_NumBands(raster_col) FROM my_raster_table").to_arrow_table()

# Ensure the band index is within range [1, RS_NumBands]
sd.sql("SELECT RS_Polygonize(raster_col, 1) FROM my_raster_table").to_arrow_table()
```

## API Test: `RS_RasterToWorldCoord`

### Signature
```python
RS_RasterToWorldCoord(arg1: raster, x: integer, y: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_rastertoworldcoord.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_RasterToWorldCoord
description: Converts raster pixel coordinates to world coordinates as a point.
kernels:
  - returns: geometry
    args:
    - raster
    - name: x
      type: integer
    - name: "y"
      type: integer
---

## Description

Pixel coordinates are 1-based, as in PostGIS `ST_RasterToWorldCoord`: `(1, 1)`
is the upper-left pixel, and the result is the world coordinate of that pixel's
upper-left corner. Coordinates outside the grid, including `0`, extrapolate
along the geotransform.

## Examples

```sql
SELECT RS_RasterToWorldCoord(RS_Example(), 10, 20);
```


### Goal
Converts specific 1-based pixel coordinates of a raster into their corresponding geospatial world coordinates represented as a `geometry` point.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform information.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
The caller must provide a valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example()`). Pixel coordinates are 1-based; `(1, 1)` corresponds to the upper-left corner of the raster.

### Output
Returns a `geometry` object representing the world coordinate (Point) of the specified pixel's upper-left corner.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using the built-in RS_Example raster
query = "SELECT RS_RasterToWorldCoord(RS_Example(), 10, 20) as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use 1-based indexing for `x` and `y`.
- Ensure the input `arg1` is a valid raster object.
- Note that coordinates outside the grid boundaries are valid and will be extrapolated based on the raster's geotransform.

### Prompt Snippet
```text
RS_RasterToWorldCoord(raster_column, x_int, y_int)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-raster object as the first argument will result in a SQL execution error.
- **Coordinate Misinterpretation:** Using 0-based indexing instead of 1-based indexing will result in an offset coordinate (e.g., `(0,0)` will extrapolate outside the expected range).
- **Missing Raster:** Attempting to call the function on a null or uninitialized raster column.

### Fix Code Hint
```python
# Ensure 1-based indexing is used
# If you have 0-based indices (col, row), use (col + 1, row + 1)
query = "SELECT RS_RasterToWorldCoord(raster_col, col + 1, row + 1) FROM my_raster_table"
```

## API Test: `RS_RasterToWorldCoordX`

### Signature
```python
RS_RasterToWorldCoordX(arg1: raster, x: integer, y: integer) -> double
```
_Source: source/docs/reference/sql/rs_rastertoworldcoordx.qmd:19_

## Description

Pixel coordinates are 1-based, as in PostGIS `ST_RasterToWorldCoord`: `(1, 1)`
is the upper-left pixel, and the result is the world coordinate of that pixel's
upper-left corner. Coordinates outside the grid, including `0`, extrapolate
along the geotransform.

## Examples

```sql
SELECT RS_RasterToWorldCoordX(RS_Example(), 10, 20);
```

### Goal
Converts a specific 1-based pixel coordinate (x, y) of a raster into its corresponding world X coordinate based on the raster's geotransform.

### Parameters
- `arg1` (`raster`): The input raster object to query.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
The function requires a valid `raster` object (e.g., created via `RS_FromPath` or `RS_FromGDALRaster`) and two integer coordinates. Pixel coordinates are 1-based; `(1, 1)` represents the upper-left corner of the raster.

### Output
Returns `double` — The world X coordinate corresponding to the upper-left corner of the specified pixel.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the world X coordinate for pixel (10, 20) of the example raster
result = sd.sql("SELECT RS_RasterToWorldCoordX(RS_Example(), 10, 20) as world_x").to_arrow_table()
print(result)
# __CHECK__ witness: result column 'world_x' contains a double value
```

### LLM Instruction Prompt
- Always use 1-based indexing for the `x` and `y` parameters.
- Ensure the input `arg1` is a valid raster object.
- Note that coordinates outside the grid boundaries are valid and will be extrapolated based on the raster's geotransform.

### Prompt Snippet
```text
RS_RasterToWorldCoordX(raster_object, x_pixel, y_pixel)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a geometry or non-raster object as `arg1` will result in a SQL execution error.
- **Coordinate Misinterpretation:** Using 0-based indexing (common in other libraries) will result in an offset coordinate; remember that SedonaDB uses 1-based indexing.
- **Null Raster:** If the input raster is `NULL`, the function will return `NULL`.

### Fix Code Hint
```python
# Ensure 1-based indexing is used
# If you have 0-based coordinates (x, y), use (x + 1, y + 1)
query = "SELECT RS_RasterToWorldCoordX(my_raster, 1, 1) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `RS_RasterToWorldCoordY`

### Signature
```python
RS_RasterToWorldCoordY(arg1: raster, x: integer, y: integer) -> double
```
_Source: source/docs/reference/sql/rs_rastertoworldcoordy.qmd:19_

## Description

Pixel coordinates are 1-based, as in PostGIS `ST_RasterToWorldCoord`: `(1, 1)`
is the upper-left pixel, and the result is the world coordinate of that pixel's
upper-left corner. Coordinates outside the grid, including `0`, extrapolate
along the geotransform.

## Examples

```sql
SELECT RS_RasterToWorldCoordY(RS_Example(), 10, 20);
```

### Goal
Converts a specific raster pixel coordinate (x, y) into its corresponding world Y coordinate based on the raster's geotransform.

### Parameters
- `arg1` (`raster`): The source raster object from which to derive the world coordinate.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
- **Data:** A valid raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must have a valid geotransform defined. Pixel coordinates are 1-based; values outside the grid dimensions are permitted and will be extrapolated based on the geotransform.

### Output
Returns `double` — The world Y coordinate corresponding to the upper-left corner of the specified pixel.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the world Y coordinate for pixel (10, 20) of the example raster
query = "SELECT RS_RasterToWorldCoordY(RS_Example(), 10, 20) as world_y"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always use 1-based indexing for `x` and `y` parameters.
- Ensure the input raster is a valid SedonaDB raster object.
- The function returns a single `double` value; ensure the SQL query selects this as a column.

### Prompt Snippet
```text
RS_RasterToWorldCoordY(raster_column, x_int, y_int)
```

### Common Failure Modes
- **Invalid Raster:** Passing a non-raster type or a null value will result in a query execution error.
- **Coordinate Out of Bounds:** While the function supports extrapolation, providing non-integer values for `x` or `y` will cause a type mismatch error.
- **Missing Geotransform:** If the raster lacks a valid geotransform (e.g., an improperly initialized empty raster), the result may be undefined or return an error.

### Fix Code Hint
```python
# Ensure coordinates are integers
x_coord = int(10)
y_coord = int(20)
query = f"SELECT RS_RasterToWorldCoordY(my_raster, {x_coord}, {y_coord}) FROM my_table"
```

## API Test: `RS_ReplaceBandNoDataValue`

### Signature
```python
RS_ReplaceBandNoDataValue(arg1: raster, band: int, nodata: float64) -> raster
```
_Source: source/docs/reference/sql/rs_replacebandnodatavalue.qmd:19_

### Goal
Updates a raster band's nodata value by remapping all existing nodata pixels to the new value, ensuring that pixels previously identified as nodata remain identified as nodata after the operation.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`int`): The 1-based index of the band to modify.
- `nodata` (`float64`): The new nodata value to assign. This value must be compatible with the band's underlying data type.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The specified band must already have a defined nodata value; if no nodata value is set, the function will return an error. The `nodata` value must fit the band's data type exactly.
- **Null Handling:** If the input raster, band index, or nodata value is null, the function returns a null raster.

### Output
Returns `raster` — A new raster object where the specified band's nodata value is updated and all pixels previously marked as nodata are rewritten to the new value.

### Valid Call Patterns
```python
# Example using the SedonaDB SQL interface
query = "SELECT RS_BandNoDataValue(RS_ReplaceBandNoDataValue(RS_Example(), 1, 200), 1)"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_ReplaceBandNoDataValue` when the goal is to preserve the "nodata" status of pixels while changing the underlying nodata value.
- Ensure the `band` index is 1-based.
- Verify that the input raster is 2-D.
- Do not use this function if the band does not already have a nodata value; use `RS_SetBandNoDataValue` if you intend to define a nodata value for a band that currently lacks one.

### Prompt Snippet
```text
RS_ReplaceBandNoDataValue(raster_object, band_index, new_nodata_value)
```

### Common Failure Modes
- **Missing Nodata:** Calling the function on a band that does not have an existing nodata value will result in an error.
- **Type Mismatch:** Providing a `nodata` value that cannot be represented by the band's data type (e.g., a large float for an integer band) may cause unexpected behavior or errors.
- **Invalid Band Index:** Providing a band index outside the range of available bands (1-based) will result in an error.
- **Unsupported Dimensions:** The function only supports 2-D rasters; attempting to use it on higher-dimensional data will fail.

### Fix Code Hint
```python
# Ensure the band has a nodata value before replacing
# If unsure, check with RS_BandNoDataValue first
sd.sql("SELECT RS_ReplaceBandNoDataValue(my_raster, 1, 255.0)").to_arrow_table()
```

## API Test: `RS_ReprojectMatch`

### Signature
```python
RS_ReprojectMatch(raster: raster, reference: raster) -> raster | RS_ReprojectMatch(raster: raster, reference: raster, algorithm: string) -> raster
```
_Source: source/docs/reference/sql/rs_reprojectmatch.qmd:19_

### Goal
`RS_ReprojectMatch` reprojects a source raster onto the CRS, grid, and extent of a reference raster to ensure spatial alignment for subsequent raster analysis.

### Parameters
- `raster` (`raster`): The input raster to be reprojected.
- `reference` (`raster`): The target raster whose CRS, grid, and extent define the output geometry.
- `algorithm` (`string`): Optional resampling algorithm (case-insensitive): `NearestNeighbor` (default), `Bilinear`, `Cubic` (alias `Bicubic`), `CubicSpline`, `Lanczos`, `Average`, or `Mode`.

### Input
- **Data:** GDAL-readable raster formats (e.g., GeoTIFF, PNG).
- **Preconditions:** 
    - Both rasters must either both have a CRS or both lack one; mixed CRS states will result in an error.
    - `Int64` and `UInt64` rasters are not supported; cast to `Int32` or `Float64` before processing.
    - The function reads the entire input raster into memory.

### Output
Returns a `raster` object that matches the CRS, resolution, dimensions, and extent of the `reference` raster, with pixel values recomputed from the input `raster`.

### Valid Call Patterns
```python
# Identity reprojection using default NearestNeighbor
sd.sql("SELECT RS_Width(RS_ReprojectMatch(RS_Example(), RS_Example())).to_arrow_table()")

# Reprojection with explicit Bilinear resampling
sd.sql("SELECT RS_Height(RS_ReprojectMatch(RS_Example(), RS_Example(), 'Bilinear')).to_arrow_table()")
```

### LLM Instruction Prompt
- Always use `RS_ReprojectMatch(raster, reference)` or `RS_ReprojectMatch(raster, reference, algorithm)` as a standalone SQL function call.
- Ensure the input raster is not `Int64` or `UInt64`.
- Verify that both rasters have consistent CRS definitions (both defined or both undefined).

### Prompt Snippet
```text
RS_ReprojectMatch(source_raster, ref_raster, 'Bilinear')
```

### Common Failure Modes
- **CRS Mismatch:** Providing one raster with a CRS and one without will trigger an error.
- **Unsupported Types:** Attempting to reproject `Int64` or `UInt64` rasters will fail due to GDAL warp limitations.
- **Memory Exhaustion:** Large rasters may cause memory issues as the entire input is loaded into memory.

### Fix Code Hint
```python
# Cast unsupported 64-bit integer rasters to Float64 before reprojecting
sd.sql("SELECT RS_ReprojectMatch(RS_CastTo(my_raster, 'Float64'), reference_raster)")
```

__CHECK__

## API Test: `RS_Resample`

### Signature
```python
RS_Resample(raster: raster, reference_raster: raster, use_scale: boolean, algorithm: string) -> raster | RS_Resample(raster: raster, width_or_scale: double, height_or_scale: double, use_scale: boolean, algorithm: string) -> raster | RS_Resample(raster: raster, width_or_scale: double, height_or_scale: double, grid_x: double, grid_y: double, use_scale: boolean, algorithm: string) -> raster
```

### Goal
`RS_Resample` modifies a raster's pixel grid (dimensions, pixel size, or origin) while preserving its CRS, band count, and per-band nodata values.

### Parameters
- `raster` (`raster`): The input raster to be resampled.
- `width_or_scale` (`double`): Output width in pixels (if `use_scale` is false) or target x pixel size (if `use_scale` is true).
- `height_or_scale` (`double`): Output height in pixels (if `use_scale` is false) or target y pixel size (if `use_scale` is true; negative for north-up).
- `grid_x` (`double`): X coordinate of the grid to which the output origin is snapped.
- `grid_y` (`double`): Y coordinate of the grid to which the output origin is snapped.
- `use_scale` (`boolean`): If false, treats dimensions as pixel counts; if true, treats them as target pixel sizes.
- `algorithm` (`string`): Resampling method: `NearestNeighbor`, `Bilinear`, `Cubic`, `CubicSpline`, `Lanczos`, `Average`, or `Mode`.

### Input
- **Data:** GDAL-readable raster formats (e.g., GeoTIFF).
- **Preconditions:** 
    - Input raster must not be `Int64` or `UInt64` (cast to `Int32` or `Float64` first).
    - Skewed or rotated rasters do not support scale changes or origin snaps.
    - CRS must match if using a `reference_raster`.

### Output
Returns a new `raster` with the modified grid dimensions or pixel size.

### Valid Call Patterns
```python
# Downsample example raster
sd.sql("SELECT RS_Width(RS_Resample(RS_Example(), 32, 16, false, 'NearestNeighbor'))").to_arrow_table()

# Upsample example raster
sd.sql("SELECT RS_Height(RS_Resample(RS_Example(), 128, 64, false, 'Bilinear'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always specify the `algorithm` string (case-insensitive).
- Ensure `use_scale` is correctly set based on whether you are providing pixel dimensions or target pixel sizes.
- Remember that `RS_Resample` does not perform reprojection; ensure the CRS is already correct.

### Prompt Snippet
```text
RS_Resample(raster_col, 32.0, 32.0, false, 'Bilinear')
```

### Common Failure Modes
- **Memory Exhaustion:** Extent-growing scale changes and origin snaps load the entire raster into memory.
- **Type Errors:** Attempting to resample `Int64` or `UInt64` rasters will fail; cast them first.
- **Invalid Dimensions:** Providing fractional values for `width_or_scale` or `height_or_scale` when `use_scale` is false will cause an error.
- **CRS Mismatch:** Using a `reference_raster` with a different CRS than the input raster will result in an error.

### Fix Code Hint
```python
# Cast to Float64 before resampling if the raster type is unsupported
sd.sql("SELECT RS_Resample(CAST(raster_col AS Float64), 100, 100, false, 'Cubic') FROM my_table")
```

## API Test: `RS_Rotation`

### Signature
```python
RS_Rotation(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_rotation.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_Rotation
description: Returns the raster rotation in radians based on skew parameters.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_Rotation(RS_Example());
```


### Goal
Calculates the rotation angle of a raster in radians by analyzing its internal skew parameters.

### Parameters
- `arg1` (`raster`): The input raster object for which the rotation angle is to be determined.

### Input
- **Data:** A valid raster object loaded into the SedonaDB environment (e.g., via `RS_FromGDALRaster` or `RS_Example`).
- **Preconditions:** The raster must contain valid affine transformation metadata (skew parameters) to compute a non-zero rotation.

### Output
Returns `double` — The rotation angle of the raster expressed in radians.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify rotation of an example raster
result = sd.sql("SELECT RS_Rotation(RS_Example()) as rotation").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_Rotation` to extract the orientation of a raster.
- Always pass a single `raster` type argument.
- The function returns a `double` representing radians; ensure downstream calculations account for this unit.

### Prompt Snippet
```text
RS_Rotation(raster_column)
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type (e.g., geometry or string) will result in a SQL execution error.
- **Missing Metadata:** If the raster lacks skew parameters or has an identity affine transform, the function may return `0.0` or an undefined value depending on the underlying GDAL metadata state.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling
sd.sql("SELECT RS_Rotation(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL")
```

## API Test: `RS_SRID`

### Signature
```python
RS_SRID(arg1: raster) -> integer
```
_Source: source/docs/reference/sql/rs_srid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_SRID
description: Returns the SRID of a raster.
kernels:
  - returns: integer
    args: [raster]
---

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT RS_SRID(RS_Example());
```


### Goal
Returns the integer Spatial Reference System Identifier (SRID) associated with the provided raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the coordinate reference system identifier.

### Input
- **Data:** A valid raster object (e.g., created via `RS_MakeEmptyRaster`, `RS_FromGDALRaster`, or loaded from a supported format like GeoTIFF).
- **Preconditions:** The raster must have a defined Coordinate Reference System (CRS). If the raster was created without an explicit CRS, the behavior depends on the underlying engine's default handling of undefined spatial references.

### Output
Returns `integer` — The SRID code (e.g., 4326 for WGS 84) representing the spatial reference system of the raster.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify SRID of an example raster
result = sd.sql("SELECT RS_SRID(RS_Example())").to_arrow_table()
print(result)
# __CHECK__ witness: The result should contain an integer column representing the SRID.
```

### LLM Instruction Prompt
- Always use `RS_SRID` as a standalone function within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster object.
- Do not attempt to cast the output to other types; it is strictly an integer.

### Prompt Snippet
```text
SELECT RS_SRID(raster_column) FROM table_name;
```

### Common Failure Modes
- **Null Input:** Passing a null or non-raster object will result in a query execution error.
- **Undefined CRS:** If the raster does not have an associated CRS, the function may return 0 or an error depending on the specific raster metadata state.
- **Type Mismatch:** Attempting to pass a geometry object instead of a raster object will cause a type mismatch error.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling RS_SRID
sd.sql("SELECT RS_SRID(r) FROM (SELECT RS_MakeEmptyRaster(1, 'uint8', 10, 10, ST_MakeEnvelope(0,0,10,10,4326)) as r)").to_arrow_table()
```

## API Test: `RS_ScaleX`

### Signature
```python
RS_ScaleX(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_scalex.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_ScaleX
description: Returns the pixel width (scale X) of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_ScaleX(RS_Example());
```


### Goal
Retrieves the horizontal pixel dimension (scale X) of a given raster object, representing the width of a single pixel in the units of the raster's coordinate reference system.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the horizontal pixel scale.

### Input
The function accepts a valid `raster` object. Rasters can be generated via `RS_MakeEmptyRaster`, `RS_FromPath`, or `RS_FromGDALRaster`. No specific CRS preconditions are required for this metadata retrieval, provided the input is a valid raster.

### Output
Returns `double` — The width of a pixel along the X-axis in the units defined by the raster's CRS.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the pixel width of an example raster
result = sd.sql("SELECT RS_ScaleX(RS_Example()) as scale_x").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_ScaleX(raster_expression)` as a standalone SQL function call.
- Ensure the input is a valid raster object; if the input is a path or binary, ensure it has been converted to a raster type first using appropriate constructors.

### Prompt Snippet
```text
RS_ScaleX(raster_column)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a geometry or string instead of a `raster` object will result in a SQL execution error.
- **Null Input:** If the input raster column contains `NULL` values, the function will return `NULL` for those rows.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling RS_ScaleX
query = "SELECT RS_ScaleX(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL"
sd.sql(query).to_arrow_table()
```

## API Test: `RS_ScaleY`

### Signature
```python
RS_ScaleY(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_scaley.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_ScaleY
description: Returns the pixel height (scale Y) of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_ScaleY(RS_Example());
```


### Goal
Returns the vertical dimension (pixel height) of a raster's grid cells as defined in its affine transformation.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the vertical scale component.

### Input
- **Data:** A valid raster object created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`.
- **Preconditions:** The raster must be correctly initialized with an affine transform that defines its spatial resolution.

### Output
Returns `double` — The pixel height (scale Y) value of the raster.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the pixel height of the example raster
result = sd.sql("SELECT RS_ScaleY(RS_Example()) as scale_y").to_arrow_table()
print(result)
# __CHECK__ witness: scale_y
```

### LLM Instruction Prompt
- Use `RS_ScaleY` when you need to determine the vertical resolution of a raster grid.
- Always pass a valid `raster` type as the argument.
- The function returns a `double`, which can be used for further spatial calculations or metadata verification.

### Prompt Snippet
```text
RS_ScaleY(raster_column)
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type (e.g., a geometry or string) will result in a SQL execution error.
- **Null Input:** If the input raster column contains `NULL` values, the function will return `NULL` for those rows.
- **Uninitialized Raster:** Attempting to call this on a raster object that has not been properly constructed or loaded may result in an error.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling RS_ScaleY
sd.sql("SELECT RS_ScaleY(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL")
```

## API Test: `RS_SetBandNoDataValue`

### Signature
```python
RS_SetBandNoDataValue(arg1: raster, nodata: float64) -> raster | RS_SetBandNoDataValue(arg1: raster, band: int, nodata: float64) -> raster
```
_Source: source/docs/reference/sql/rs_setbandnodatavalue.qmd:19_

### Goal
Sets the nodata value metadata for a specific band of a raster, packing the value into the band's native data type without modifying the actual cell values.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`int`): The 1-based index of the band to update.
- `nodata` (`float64`): The nodata value to assign to the band, or `NULL` to clear the existing nodata value.

### Input
- **Data:** A valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Preconditions:** For single-band rasters, the two-argument form is sufficient. For multi-band rasters, the three-argument form must be used to specify the target band index.
- **Metadata:** This operation is metadata-only; it does not alter the underlying pixel data.

### Output
Returns a `raster` object with the updated nodata metadata for the specified band. A null input raster or invalid band index results in a null raster.

### Valid Call Patterns
```python
# Using the 3-argument form to set nodata for a specific band
result = sd.sql("SELECT RS_SetBandNoDataValue(RS_Example(), 1, 0.0)").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `RS_SetBandNoDataValue(raster, band, nodata)` for multi-band rasters.
- Use `RS_SetBandNoDataValue(raster, nodata)` only for single-band rasters.
- Pass `NULL` as the `nodata` argument to clear the existing nodata value.
- Remember that band indices are 1-based.

### Prompt Snippet
```text
RS_SetBandNoDataValue(raster_col, 1, 255.0)
```

### Common Failure Modes
- **Index Out of Bounds:** Providing a band index that does not exist in the raster (e.g., 0 or a number greater than the total band count).
- **Type Mismatch:** Passing a non-numeric value for `nodata` (unless passing `NULL`).
- **Multi-band Ambiguity:** Attempting to use the two-argument form on a multi-band raster, which may lead to unexpected behavior or errors.

### Fix Code Hint
```python
# Ensure the band index is valid (1-based)
# If the raster is multi-band, always use the 3-argument form:
sd.sql("SELECT RS_SetBandNoDataValue(my_raster, 1, 0.0) FROM my_table")
```

## API Test: `RS_SetCRS`

### Signature
```python
RS_SetCRS(arg1: raster, target_crs: string) -> raster
```
_Source: source/docs/reference/sql/rs_setcrs.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_SetCRS
description: Sets the Coordinate Reference System (CRS) for a raster.
kernels:
  - returns: raster
    args:
    - raster
    - name: target_crs
      type: string
---

## Description

Sets a CRS on a raster. This is metadata-only: raster cell values and
geotransform are not transformed.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT RS_SetCRS(RS_Example(), 'EPSG:4326');
```


### Goal
Updates the metadata of a raster object to associate it with a specific Coordinate Reference System (CRS) without modifying the underlying pixel data or geotransform.

### Parameters
- `arg1` (`raster`): The input raster object to be updated.
- `target_crs` (`string`): The CRS identifier string (e.g., 'EPSG:4326') to assign to the raster.

### Input
The function accepts a valid `raster` object (e.g., created via `RS_FromPath` or `RS_MakeEmptyRaster`) and a `string` representing a valid CRS. This is a metadata-only operation; no spatial transformation of the raster data occurs.

### Output
Returns a `raster` object with the updated CRS metadata.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Assigning EPSG:4326 to an example raster
query = "SELECT RS_SetCRS(RS_Example(), 'EPSG:4326') as raster_with_crs"
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness: CRS updated successfully")
```

### LLM Instruction Prompt
- Use `RS_SetCRS` only when you need to define or override the CRS metadata of a raster.
- Remember that this function does not perform reprojection; it only updates the metadata label.
- Ensure the `target_crs` string follows standard CRS naming conventions (e.g., 'EPSG:XXXX').

### Prompt Snippet
```text
RS_SetCRS(raster_column, 'EPSG:4326')
```

### Common Failure Modes
- Providing an invalid or unsupported CRS string format, which may result in runtime errors during subsequent spatial operations.
- Assuming that `RS_SetCRS` performs a coordinate transformation; if the raster data is in a different projection than the assigned CRS, spatial analysis (like `RS_Clip` or `RS_ZonalStats`) will produce incorrect results.

### Fix Code Hint
```python
# If you need to actually transform the data, use a reprojection function instead.
# RS_SetCRS is for metadata assignment only.
```

## API Test: `RS_SetGeoReference`

### Signature
```python
RS_SetGeoReference(arg1: raster, georeference: string) -> raster | RS_SetGeoReference(arg1: raster, georeference: string, format: string) -> raster | RS_SetGeoReference(arg1: raster, upperleftx: numeric, upperlefty: numeric, scalex: numeric, scaley: numeric, skewx: numeric, skewy: numeric) -> raster
```
_Source: source/docs/reference/sql/rs_setgeoreference.qmd:19_

### Goal
Updates the affine geotransform metadata of a raster without resampling the underlying pixel values.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `upperleftx` (`numeric`): X-coordinate of the upper-left pixel's corner.
- `upperlefty` (`numeric`): Y-coordinate of the upper-left pixel's corner.
- `scalex` (`numeric`): The width of a pixel.
- `scaley` (`numeric`): The height of a pixel (typically negative for north-up rasters).
- `skewx` (`numeric`): The row rotation term.
- `skewy` (`numeric`): The column rotation term.

### Input
- **Data:** A valid `raster` object.
- **Georeference String:** A string containing six whitespace-separated numbers in the order: `scalex`, `skewy`, `skewx`, `scaley`, `upperleftx`, `upperlefty`.
- **Format:** Optional string ('GDAL' or 'ESRI') defining the coordinate convention for the upper-left values. 'GDAL' (default) treats coordinates as the pixel corner; 'ESRI' treats them as the pixel center.

### Output
Returns a new `raster` object with the updated affine geotransform metadata.

### Valid Call Patterns
```python
# Using a georeference string
sd.sql("SELECT RS_SetGeoReference(RS_Example(), '2 0 0 -3 100 200')").to_arrow_table()

# Using numeric arguments
sd.sql("SELECT RS_SetGeoReference(RS_Example(), 100, 200, 2, -3, 0, 0)").to_arrow_table()

# __CHECK__ witness:
# The result is a raster with updated georeference metadata.
```

### LLM Instruction Prompt
- Use `RS_SetGeoReference` to modify raster metadata only.
- When using the string format, ensure the six numbers follow the order: `scalex`, `skewy`, `skewx`, `scaley`, `upperleftx`, `upperlefty`.
- When using numeric arguments, ensure the order is: `upperleftx`, `upperlefty`, `scalex`, `scaley`, `skewx`, `skewy`.
- Remember that this operation does not resample pixel values.

### Prompt Snippet
```text
RS_SetGeoReference(raster_col, '2 0 0 -3 100 200')
```

### Common Failure Modes
- **Incorrect Argument Order:** Providing numbers in the wrong order for the string vs. numeric overloads will result in an invalid geotransform.
- **Format Mismatch:** Using 'ESRI' format when 'GDAL' is expected (or vice versa) will cause a shift in the raster's spatial location.
- **Invalid Raster:** Passing a non-raster type as `arg1` will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the georeference string matches the expected order:
# scalex, skewy, skewx, scaley, upperleftx, upperlefty
query = "SELECT RS_SetGeoReference(my_raster, '2 0 0 -3 100 200') FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `RS_SetSRID`

### Signature
```python
RS_SetSRID(arg1: raster, srid: integer) -> raster
```
_Source: source/docs/reference/sql/rs_setsrid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_SetSRID
description: Sets the SRID (spatial reference identifier) for a raster.
kernels:
  - returns: raster
    args:
    - raster
    - name: srid
      type: integer
---

## Description

Sets the SRID of a raster. This is metadata-only: raster cell values and
geotransform are not transformed.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT RS_SetSRID(RS_Example(), 4326);
```


### Goal
Updates the spatial reference identifier (SRID) metadata of a raster object without modifying the underlying pixel data or geotransform.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `srid` (`integer`): The integer identifier representing the target Coordinate Reference System (CRS).

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`).
- **Preconditions:** The raster must be a valid SedonaDB raster type. This operation is metadata-only; it does not perform coordinate reprojection.

### Output
Returns `raster` — A new raster object with the updated SRID metadata.

### Valid Call Patterns
```python
# Example using the SedonaDB SQL interface
query = "SELECT RS_SetSRID(RS_Example(), 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `RS_SetSRID(raster, srid)` as a standalone SQL function call within an `sd.sql()` query.
- Do not attempt to use this function for coordinate reprojection; it only updates the metadata tag.

### Prompt Snippet
```text
RS_SetSRID(raster_column, 4326)
```

### Common Failure Modes
- **Invalid SRID:** Providing an SRID that is not supported or recognized by the underlying CRS registry.
- **Non-Raster Input:** Passing a geometry or non-raster type as the first argument will result in a type mismatch error.
- **Misunderstanding Metadata:** Expecting the function to reproject the raster data; users must use appropriate reprojection functions if coordinate transformation is required.

### Fix Code Hint
```python
# Ensure the input is a valid raster and the SRID is an integer
sd.sql("SELECT RS_SetSRID(my_raster_col, 3857) FROM my_table").to_arrow_table()
```

## API Test: `RS_SetValue`

### Signature
```python
RS_SetValue(arg1: raster, col_x: integer, row_y: integer, new_value: double) -> raster | RS_SetValue(arg1: raster, band: integer, col_x: integer, row_y: integer, new_value: double) -> raster
```
_Source: source/docs/reference/sql/rs_setvalue.qmd:19_

## Description

`col_x` and `row_y` count from 1, as in PostGIS and Sedona Spark: `(1, 1)` is
the upper-left pixel. The pixel is overwritten even if it holds the band's
nodata value, and the band's nodata value is unchanged.

The value is stored in the band's pixel type: an integer band truncates it
toward zero (3.9 is stored as 3) and a float32 band rounds it to the nearest
float32. A value outside an integer band's range (for a 64-bit integer band,
beyond ±2^53, the largest integers a double holds exactly), NaN, a pixel
outside the grid, or a band the raster does not have is an error. Only 2-D
rasters are supported.

## Examples

Setting pixel (2, 2) of band 1 to 42 raises the band's maximum, which is
otherwise 1, to 42:

```sql
SELECT RS_SummaryStats(RS_SetValue(RS_Example(), 1, 2, 2, 42), 'max', 1);
```

### Goal
Updates the value of a specific pixel within a designated band of a 2-D raster.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`integer`): The 1-based index of the band to modify. Required for multiband rasters; optional for single-band rasters.
- `col_x` (`integer`): The 1-based column index of the pixel to update.
- `row_y` (`integer`): The 1-based row index of the pixel to update.
- `new_value` (`double`): The new value to assign to the specified pixel.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The specified `col_x` and `row_y` must be within the raster's grid dimensions. If a `band` is specified, it must exist within the raster.

### Output
Returns a new `raster` object with the specified pixel updated to `new_value`.

### Valid Call Patterns
```python
# Example using the Python interface
result = sd.sql("SELECT RS_SetValue(RS_Example(), 1, 2, 2, 42.0)").to_arrow_table()
print("__CHECK__", result)
```

### LLM Instruction Prompt
- Always use 1-based indexing for `col_x`, `row_y`, and `band`.
- If the raster is multiband, the `band` argument is mandatory.
- Ensure `new_value` is provided as a double; note that integer bands will truncate this value toward zero.

### Prompt Snippet
```text
RS_SetValue(raster_obj, band_index, col, row, value)
```

### Common Failure Modes
- **Out of Bounds:** Providing `col_x` or `row_y` values that fall outside the raster grid dimensions.
- **Invalid Band:** Specifying a `band` index that does not exist in the raster.
- **Type Mismatch/Range:** Providing a `new_value` that is NaN or exceeds the valid range of the target band's pixel type (e.g., integer overflow).
- **Unsupported Raster:** Attempting to use this function on non-2-D rasters.

### Fix Code Hint
```python
# Ensure indices are 1-based and within bounds
# Verify band existence before calling
query = "SELECT RS_SetValue(my_raster, 1, 1, 1, 10.5) FROM my_table"
```

## API Test: `RS_Shape`

### Signature
```python
RS_Shape(arg1: raster) -> list<int64> | RS_Shape(arg1: raster, band: int) -> list<int64>
```
_Source: source/docs/reference/sql/rs_shape.qmd:19_

### Goal
Returns the dimension sizes (shape) of a raster or a specific raster band as a list of integers.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `band` (`int`): The 1-based index of the specific band to query. If omitted, all bands must share the same shape.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromGDALRaster` or `RS_Example`).
- **Preconditions:** If the `band` index is omitted, the raster must have consistent dimensions across all bands; otherwise, an error is raised. Pixel coordinates and band indices are 1-based.

### Output
Returns `list<int64>` — A list of dimension sizes. For standard 2D rasters, this is typically `[height, width]`.

### Valid Call Patterns
```python
# Get shape of all bands (must be consistent)
sd.sql("SELECT RS_Shape(RS_Example())").to_arrow_table()

# Get shape of a specific band
sd.sql("SELECT RS_Shape(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Shape` to retrieve the dimensions of a raster.
- Always specify the `band` index if the raster contains multiple bands with potentially different dimensions.
- Note that this function is marked as **Experimental**.

### Prompt Snippet
```text
RS_Shape(raster_column)
RS_Shape(raster_column, band_index)
```

### Common Failure Modes
- **Inconsistent Band Shapes:** Calling `RS_Shape(raster)` without a band index on a multi-band raster where bands have different dimensions will trigger an error.
- **Out-of-range Index:** Providing a `band` index that does not exist in the raster returns `null`.
- **Null Input:** Passing a null raster object returns `null`.

### Fix Code Hint
```python
# If RS_Shape(raster) fails due to inconsistent band sizes, specify the band:
sd.sql("SELECT RS_Shape(my_raster, 1) FROM my_table").to_arrow_table()
```

## API Test: `RS_SkewX`

### Signature
```python
RS_SkewX(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_skewx.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_SkewX
description: Returns the X skew (rotation) parameter of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_SkewX(RS_Example());
```


### Goal
Retrieves the X-axis skew (rotation) component from the affine transformation matrix of a given raster.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the X skew metadata.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`).
- **Preconditions:** The raster must be properly initialized with an affine transformation matrix.

### Output
Returns `double` — The X skew value representing the rotation of the raster grid relative to the coordinate system axes.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify X skew of an example raster
result = sd.sql("SELECT RS_SkewX(RS_Example()) as skew_x").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_SkewX` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster type; if the input is a path, wrap it in a raster-loading function first.

### Prompt Snippet
```text
SELECT RS_SkewX(raster_column) FROM table_name;
```

### Common Failure Modes
- **Null Input:** Passing a null or invalid raster object will result in a null output or execution error.
- **Missing Metadata:** If the raster does not have a defined affine transformation matrix, the function may return a default value (typically 0.0) or fail depending on the raster source.

### Fix Code Hint
```python
# Ensure the raster is valid before querying metadata
sd.sql("SELECT RS_SkewX(my_raster) FROM my_table WHERE my_raster IS NOT NULL")
```

## API Test: `RS_SkewY`

### Signature
```python
RS_SkewY(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_skewy.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_SkewY
description: Returns the Y skew (rotation) parameter of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_SkewY(RS_Example());
```


### Goal
Retrieves the Y-axis skew (rotation) component from the affine transformation matrix of a given raster.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the Y skew metadata.

### Input
- **Data:** A valid raster object loaded into SedonaDB (e.g., via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** The raster must be correctly initialized with georeferencing metadata.

### Output
Returns `double` — The Y skew value representing the rotation of the raster grid relative to the coordinate system axes.

### Valid Call Patterns
```python
# Execute via the SedonaDB SQL connection
result = sd.sql("SELECT RS_SkewY(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_SkewY` when you need to inspect the georeferencing rotation of a raster.
- Always pass a valid raster object as the argument.
- The function returns a scalar `double`.

### Prompt Snippet
```text
RS_SkewY(raster_column)
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type or a null value will result in a query execution error.
- **Missing Metadata:** If the raster was created without georeferencing information (e.g., a raw pixel array), the skew value may be undefined or default to 0.0.

### Fix Code Hint
```python
# Ensure the raster is properly georeferenced before calling metadata functions
# If the raster is missing metadata, use RS_SetGeoReference to define the affine transform
sd.sql("SELECT RS_SkewY(my_raster_col) FROM my_table").to_arrow_table()
```

## API Test: `RS_Slice`

### Signature
```python
RS_Slice(arg1: raster, dim_name: utf8, index: int) -> raster
```
_Source: source/docs/reference/sql/rs_slice.qmd:19_

### Goal
Extracts a single slice along a named non-spatial dimension from a raster, removing that dimension from the output while preserving spatial dimensions.

### Parameters
- `arg1` (`raster`): The input raster object containing one or more bands with non-spatial dimensions.
- `dim_name` (`utf8`): The name of the non-spatial dimension to slice (e.g., 'time' or 'band').
- `index` (`int`): The zero-based index along the specified dimension to extract.

### Input
- **Data:** A raster object, typically created via `RS_FromPath`, `RS_FromGDALRaster`, or transformed via `RS_BandToDim`.
- **Preconditions:** The raster must contain at least one band that carries the specified `dim_name`. Spatial dimensions (`x_dim`, `y_dim`) cannot be sliced.

### Output
Returns a `raster` with the specified dimension removed from all bands that carried it. Bands that do not contain the dimension are passed through unchanged.

### Valid Call Patterns
```python
# Example using the SedonaDB SQL interface
query = "SELECT RS_Slice(RS_BandToDim(RS_Example(), 'band'), 'band', 0)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `RS_Slice` when you need to reduce the dimensionality of a multi-dimensional raster by selecting a specific index along a non-spatial axis.
- Always ensure the dimension name provided matches the metadata of the input raster.
- If the raster is 2D, use `RS_BandToDim` first to create a dimension that can be sliced.

### Prompt Snippet
```text
RS_Slice(raster_object, 'dimension_name', index_integer)
```

### Common Failure Modes
- **Dimension Not Found:** Errors if no band in the raster carries the specified `dim_name` (often due to a typo).
- **Index Out of Range:** Errors if the provided `index` is outside the valid range for any band that carries the dimension.
- **Experimental Status:** As an experimental feature, behavior may change; ensure the environment supports the current version of SedonaDB.

### Fix Code Hint
```python
# Ensure the dimension exists before slicing
# If slicing fails, verify the dimension name using metadata functions
# or check if the raster has been prepared with RS_BandToDim.
```

## API Test: `RS_SliceRange`

### Signature
```python
RS_SliceRange(arg1: raster, dim_name: utf8, start: int, end: int) -> raster
```
_Source: source/docs/reference/sql/rs_slicerange.qmd:19_

### Goal
`RS_SliceRange` narrows a specified non-spatial dimension of a raster to a half-open index range `[start, end)`, preserving the dimension in the output with a reduced size.

### Parameters
- `arg1` (`raster`): The input raster object containing one or more dimensions.
- `dim_name` (`utf8`): The name of the non-spatial dimension to narrow (e.g., 'time' or 'band').
- `start` (`int`): The zero-based start index (inclusive) of the range.
- `end` (`int`): The exclusive end index of the range.

### Input
The input must be a valid `raster` object. The `dim_name` must correspond to a dimension present in at least one band of the raster. Spatial dimensions (`x_dim` and `y_dim`) cannot be narrowed using this function.

### Output
Returns a `raster` with the specified dimension reduced to the size defined by the range `[start, end)`. Bands that do not contain the specified dimension are passed through to the output unchanged.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Narrowing a dimension on a raster
query = "SELECT RS_SliceRange(RS_BandToDim(RS_Example(), 'band'), 'band', 0, 2)"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print("Result materialized successfully.")
```

### LLM Instruction Prompt
When using `RS_SliceRange`, ensure the `dim_name` matches a dimension existing in the raster. The `start` index must be less than the `end` index, and both must be within the bounds of the specified dimension. Do not attempt to use this function to slice spatial dimensions (`x` or `y`).

### Prompt Snippet
```text
RS_SliceRange(raster_object, 'dimension_name', start_index, end_index)
```

### Common Failure Modes
- **Dimension Not Found:** Errors if no band in the raster carries the provided `dim_name`.
- **Invalid Range:** Errors if `start >= end` or if the indices are out of bounds for the specified dimension.
- **Spatial Dimension Usage:** Attempting to slice `x` or `y` dimensions will fail as they are not supported.

### Fix Code Hint
```python
# Ensure the dimension exists and indices are valid
# If slicing fails, verify the dimension name using RS_MetaData()
query = "SELECT RS_SliceRange(my_raster, 'time', 0, 5)"
```

## API Test: `RS_Stack`

### Signature
```python
RS_Stack(arg1: raster, rasters: raster) -> raster
```
_Source: source/docs/reference/sql/rs_stack.qmd:19_

### Goal
Combines multiple rasters into a single multi-band raster by stacking their bands in the order provided.

### Parameters
- `arg1` (`raster`): The base raster that defines the spatial grid (width, height, geotransform, and CRS) for the resulting stacked raster.
- `rasters` (`raster`): One or more additional rasters to be stacked onto the base raster.

### Input
- **Data:** Raster objects (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Preconditions:** All input rasters must share the exact same grid definition as `arg1` (same width, height, geotransform, and CRS).
- **Constraints:** A raster on a different grid will trigger an error. A band with a dimension already named 'y' or 'x' that conflicts with the base raster's naming convention will trigger an error.

### Output
Returns `raster` — A new raster object containing all bands from the input rasters, preserving individual pixel types, nodata values, and band names.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example: Stacking two example rasters and verifying the band count
query = "SELECT RS_NumBands(RS_Stack(RS_Example(), RS_Example())) as band_count"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Use `RS_Stack` to merge multiple rasters into one.
- Ensure all input rasters are aligned to the same grid (width, height, geotransform, CRS) as the first argument.
- Note that this is an experimental function and does not trigger pixel-level data loading (out-of-database operations are preserved).

### Prompt Snippet
```text
RS_Stack(base_raster, additional_raster)
```

### Common Failure Modes
- **Grid Mismatch:** Providing rasters with different dimensions, geotransforms, or CRS will result in an error.
- **Naming Conflicts:** Attempting to stack a band that has a dimension already named 'y' or 'x' when the base raster uses a different naming convention.
- **Experimental Status:** As an experimental feature, behavior may change; ensure the environment is updated to the latest SedonaDB version.

### Fix Code Hint
```python
# Ensure rasters are aligned before stacking using RS_ReprojectMatch if necessary
query = """
SELECT RS_Stack(r1, RS_ReprojectMatch(r2, r1)) 
FROM my_raster_table
"""
```

## API Test: `RS_SummaryStats`

### Signature
```python
RS_SummaryStats(arg1: raster, stat_type: string) -> double | RS_SummaryStats(arg1: raster, stat_type: string, band: integer) -> double | RS_SummaryStats(arg1: raster, stat_type: string, band: integer, exclude_nodata: boolean) -> double
```
_Source: source/docs/reference/sql/rs_summarystats.qmd:19_

### Goal
Calculates a specific summary statistic (count, sum, mean, stddev, min, or max) for the pixel values within a specified band of a 2-D raster.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `stat_type` (`string`): The statistic to compute. Must be one of `count`, `sum`, `mean`, `stddev`, `min`, or `max` (case-insensitive).
- `band` (`integer`): The 1-based index of the band to process. Defaults to 1 if not specified.
- `exclude_nodata` (`boolean`): If true, pixels matching the band's nodata value are excluded from the calculation. Defaults to true.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must contain the specified band index. 64-bit integer pixels must not exceed 2^53 in magnitude, as they cannot be represented exactly as a double.

### Output
Returns `double` — The calculated summary statistic for the specified band. Returns `NaN` for statistics other than `count` or `sum` if the band contains no valid pixels.

### Valid Call Patterns
```python
# Basic usage with default band and nodata exclusion
sd.sql("SELECT RS_SummaryStats(RS_Example(), 'mean')").to_arrow_table()

# Explicit band and nodata inclusion
sd.sql("SELECT RS_SummaryStats(RS_Example(), 'count', 1, false)").to_arrow_table()

# Using with modified raster
sd.sql("SELECT RS_SummaryStats(RS_SetValue(RS_Example(), 1, 2, 2, 42), 'max', 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_SummaryStats` within a `sd.sql()` query.
- Ensure the `stat_type` is one of the six supported strings.
- Remember that band indices are 1-based.
- If the raster has specific nodata requirements, explicitly set the `exclude_nodata` boolean.

### Prompt Snippet
```text
RS_SummaryStats(raster_column, 'mean', 1, true)
```

### Common Failure Modes
- **Invalid Band Index:** Requesting a band index that does not exist in the raster will result in an error.
- **Unsupported Stat Type:** Providing a string other than the six supported types will cause an error.
- **Precision Limits:** Attempting to process 64-bit integer pixels with values exceeding 2^53 will result in an error because the return type is a double.
- **Raster Dimensionality:** Attempting to use this function on non-2-D rasters will result in an error.

### Fix Code Hint
```python
# Ensure the band exists before calling
# Check raster metadata if unsure about band count
sd.sql("SELECT RS_NumBands(my_raster) FROM my_table").to_arrow_table()

# If dealing with large integers, cast to float64 first if necessary
```

## API Test: `RS_SummaryStatsAll`

### Signature
```python
RS_SummaryStatsAll(arg1: raster) -> struct | RS_SummaryStatsAll(arg1: raster, band: integer) -> struct | RS_SummaryStatsAll(arg1: raster, band: integer, exclude_nodata: boolean) -> struct
```
_Source: source/docs/reference/sql/rs_summarystatsall.qmd:19_

### Goal
Computes the complete set of summary statistics (count, sum, mean, stddev, min, and max) for a specified raster band in a single operation.

### Parameters
- `arg1` (`raster`): The input 2-D raster object to analyze.
- `band` (`integer`): The 1-based index of the band to process. Defaults to 1 if not specified.
- `exclude_nodata` (`boolean`): If true, pixels matching the band's nodata value are excluded from the calculation. Defaults to true.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The specified `band` must exist within the raster. 
- **Constraints:** 64-bit integer pixels with magnitudes exceeding $2^{53}$ will cause an error as they cannot be represented exactly as doubles. Only 2-D rasters are supported.

### Output
Returns a `struct` containing six double-precision values: `count`, `sum`, `mean`, `stddev` (population standard deviation), `min`, and `max`. If the band contains no valid pixels, `count` and `sum` are 0, and the remaining fields are `NaN`.

### Valid Call Patterns
```python
# Basic usage with default band and nodata exclusion
sd.sql("SELECT RS_SummaryStatsAll(RS_Example())").to_arrow_table()

# Explicit band and nodata inclusion
sd.sql("SELECT RS_SummaryStatsAll(RS_Example(), 1, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_SummaryStatsAll` when multiple statistics are required for a single band to optimize performance (scans the band three times instead of six).
- Ensure the `band` index is 1-based.
- If the raster contains nodata values, explicitly set `exclude_nodata` to `true` (default) or `false` based on the analytical requirement.

### Prompt Snippet
```text
RS_SummaryStatsAll(raster_col, band_index, exclude_nodata_bool)
```

### Common Failure Modes
- **Invalid Band Index:** Requesting a band index that does not exist in the raster will result in an error.
- **Precision Loss:** Attempting to process 64-bit integer rasters with values $> 2^{53}$ will trigger an error.
- **Unsupported Dimensions:** Attempting to process non-2-D rasters will fail.

### Fix Code Hint
```python
# Ensure the band exists before calling
# Cast 64-bit integer rasters to Float64 if values exceed 2^53
sd.sql("SELECT RS_SummaryStatsAll(CAST(my_raster AS Float64), 1, true)").to_arrow_table()
```

## API Test: `RS_Tile`

### Signature
```python
RS_Tile(arg1: raster, width: integer, height: integer, pad_with_no_data: boolean, no_data_value: double) -> list | RS_Tile(arg1: raster, band_indices: array, width: integer, height: integer, pad_with_no_data: boolean, no_data_value: double) -> list
```

### Goal
`RS_Tile` splits a single input raster into a grid of smaller, fixed-size raster tiles, returning a list of structs containing the grid coordinates and the tile data.

### Parameters
- `arg1` (`raster`): The input raster to be tiled.
- `band_indices` (`array`): Optional `Array[Int]` of 1-based band indices to include in the output tiles.
- `width` (`integer`): The desired width of each tile in pixels (must be >= 1).
- `height` (`integer`): The desired height of each tile in pixels (must be >= 1).
- `pad_with_no_data` (`boolean`): If true, pads partial edge tiles to the full tile size using a nodata value; if false, emits partial tiles as-is.
- `no_data_value` (`double`): The value used for padding pixels when `pad_with_no_data` is true.

### Input
The function accepts a `raster` object (e.g., from `RS_Example()` or loaded via `RS_FromGDALRaster`). The input must be a valid raster object within the SedonaDB session.

### Output
Returns a `list` of structs. Each struct contains:
- `x`: The 0-based grid column index.
- `y`: The 0-based grid row index.
- `tile`: The resulting raster tile with a geotransform shifted to its local origin.

### Valid Call Patterns
```python
# Tile every band into 32x16 tiles
sd.sql("SELECT UNNEST(RS_Tile(RS_Example(), 32, 16)) AS tile").to_arrow_table()

# Tile specific bands (1 and 3) into 32x16 tiles
sd.sql("SELECT UNNEST(RS_Tile(RS_Example(), make_array(1, 3), 32, 16)) AS tile").to_arrow_table()
```

### LLM Instruction Prompt
When using `RS_Tile`, always remember that it performs a full copy of the raster data. If the input raster is large or the tile size is very small, this will consume significant memory. Use `UNNEST` on the returned list to flatten the results into individual rows for further processing.

### Prompt Snippet
```sql
SELECT UNNEST(RS_Tile(raster_col, 32, 32)) FROM my_raster_table;
```

### Common Failure Modes
- **Memory Exhaustion:** Creating a very large number of small tiles (e.g., 1x1) will materialize a massive amount of data in memory.
- **Invalid Data Type:** Providing a `no_data_value` that does not fit the band's underlying data type will result in an error.
- **Experimental Status:** As an experimental feature, behavior or syntax may change; ensure the environment is updated to the latest SedonaDB version.

### Fix Code Hint
```python
# If memory issues occur, increase tile size or process in smaller chunks
# Ensure band_indices are 1-based
result = sd.sql("SELECT UNNEST(RS_Tile(my_raster, make_array(1), 128, 128))").to_arrow_table()
print("__CHECK__ witness: Successfully tiled raster into ArrowTable")
```

## API Test: `RS_UpperLeftX`

### Signature
```python
RS_UpperLeftX(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_upperleftx.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_UpperLeftX
description: Returns the upper-left X coordinate of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_UpperLeftX(RS_Example());
```


### Goal
Retrieves the X-coordinate of the upper-left corner of the provided raster object within the SedonaDB coordinate system.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the upper-left X coordinate.

### Input
The function accepts a valid `raster` object. Rasters can be created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`. Ensure the raster is properly initialized and contains valid spatial metadata before calling this function.

### Output
Returns `double` — The X-coordinate value corresponding to the upper-left corner of the raster's spatial extent.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the upper-left X coordinate of an example raster
result = sd.sql("SELECT RS_UpperLeftX(RS_Example()) as x_coord").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_UpperLeftX(raster_expression)` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster type; if the input is a path or file, use a constructor like `RS_FromPath` first.

### Prompt Snippet
```text
SELECT RS_UpperLeftX(<raster_column_or_expression>)
```

### Common Failure Modes
- **Null Input:** Passing a `NULL` raster will result in a `NULL` return value.
- **Invalid Raster:** Passing an object that is not a valid raster (e.g., a geometry or string) will cause a SQL execution error.
- **Uninitialized Raster:** Attempting to call this on a raster that failed to load or was improperly constructed via `RS_MakeEmptyRaster` may lead to unexpected results or errors.

### Fix Code Hint
```python
# Ensure the raster is valid before extraction
query = "SELECT RS_UpperLeftX(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL"
sd.sql(query).to_arrow_table()
```

## API Test: `RS_UpperLeftY`

### Signature
```python
RS_UpperLeftY(arg1: raster) -> double
```
_Source: source/docs/reference/sql/rs_upperlefty.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_UpperLeftY
description: Returns the upper-left Y coordinate of a raster.
kernels:
  - returns: double
    args: [raster]
---

## Examples

```sql
SELECT RS_UpperLeftY(RS_Example());
```


### Goal
Retrieves the Y-coordinate of the upper-left corner of the provided raster object, which is essential for understanding the spatial positioning of the raster within its coordinate reference system.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the upper-left Y coordinate.

### Input
- **Data:** A valid raster object created via functions like `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`.
- **Preconditions:** The raster must be correctly initialized and contain valid affine transformation metadata.

### Output
Returns `double` — The Y-coordinate value corresponding to the upper-left corner of the raster's extent.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the upper-left Y coordinate of an example raster
result = sd.sql("SELECT RS_UpperLeftY(RS_Example()) as y_coord").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_UpperLeftY` when you need to extract the vertical origin of a raster for spatial alignment or coordinate verification.
- Always pass a valid `raster` type as the argument.
- The function returns a `double`, which can be used in subsequent SQL calculations or filtering.

### Prompt Snippet
```text
RS_UpperLeftY(raster_column)
```

### Common Failure Modes
- **Invalid Input:** Passing a geometry or non-raster type will result in a SQL execution error.
- **Null Raster:** If the input raster column contains `NULL` values, the function will return `NULL` for those rows.
- **Missing Metadata:** If the raster was created without proper affine transformation parameters, the returned value may be undefined or default to 0.0.

### Fix Code Hint
```python
# Ensure the raster is valid before calling metadata functions
sd.sql("SELECT RS_UpperLeftY(r) FROM my_raster_table WHERE r IS NOT NULL")
```

## API Test: `RS_Value`

### Signature
```python
RS_Value(arg1: raster, point: geometry) -> float64 | RS_Value(arg1: raster, point: geometry, band: int) -> float64 | RS_Value(arg1: raster, colX: int, rowY: int, band: int) -> float64
```
_Source: source/docs/reference/sql/rs_value.qmd:19_

### Goal
`RS_Value` retrieves the pixel value of a raster at a specific location, defined either by a point geometry or by 1-based grid coordinates.

### Parameters
- `arg1` (`raster`): The input raster object to be sampled.
- `colX` (`int`): The 1-based column index of the pixel.
- `rowY` (`int`): The 1-based row index of the pixel.
- `band` (`int`): The 1-based index of the raster band to sample.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** If using a point geometry, the point is reprojected into the raster CRS if both carry one. If using grid coordinates, `(1, 1)` represents the upper-left pixel.
- **Constraints:** Only 2-D rasters are supported.

### Output
Returns `float64` representing the pixel value. Returns `NULL` if the location is outside the raster bounds or if the pixel value matches the band's nodata value.

### Valid Call Patterns
```python
# Sampling by point geometry and band
sd.sql("SELECT RS_Value(RS_Example(), ST_Point(74.58, 110.57, 'OGC:CRS84'), 1)").to_arrow_table()

# Sampling by grid coordinates
sd.sql("SELECT RS_Value(RS_Example(), 1, 1, 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for `colX`, `rowY`, and `band`.
- If the raster is multiband, the `band` parameter is mandatory.
- If the raster is single-band, the `band` parameter is optional and defaults to 1.
- Ensure the input geometry is a valid point.

### Prompt Snippet
```text
RS_Value(raster_column, ST_Point(x, y), band_index)
```

### Common Failure Modes
- **Ambiguity Error:** Omitting the `band` parameter when querying a multiband raster.
- **Out of Bounds:** Providing grid coordinates of `0` or values exceeding the raster dimensions, which returns `NULL`.
- **Unsupported Dimensions:** Attempting to use this function on non-2-D rasters.
- **CRS Mismatch:** Providing a point geometry in a CRS that cannot be reprojected to the raster's CRS.

### Fix Code Hint
```python
# Ensure band is explicitly provided for multiband rasters
result = sd.sql("SELECT RS_Value(my_raster, 1, 1, 1) FROM my_table").to_arrow_table()
print(result)
# __CHECK__
```

## API Test: `RS_Values`

### Signature
```python
RS_Values(arg1: raster, points: geometry) -> list | RS_Values(arg1: raster, points: geometry, band: int) -> list
```
_Source: source/docs/reference/sql/rs_values.qmd:19_

### Goal
`RS_Values` samples pixel values from a raster at the locations of a MultiPoint or Point geometry, returning the results as a list of doubles.

### Parameters
- `arg1` (`raster`): The source 2-D raster to sample.
- `points` (`geometry`): A MultiPoint or Point geometry defining the locations to sample. If the geometry and raster have defined CRSs, the points are reprojected into the raster's CRS.
- `band` (`int`): The 1-based index of the raster band to sample. Defaults to 1 for single-band rasters; required for multiband rasters.

### Input
- **Data:** A 2-D raster and a geometry (Point or MultiPoint).
- **Preconditions:** Only 2-D rasters are supported. If the raster is multiband, the `band` parameter must be explicitly provided. If the geometry and raster carry different CRSs, they must be compatible for reprojection.

### Output
Returns `list` — A list of doubles corresponding to the pixel values at each point in the input geometry, maintaining the input order. Elements are `NULL` if the point is outside the raster extent or if the sampled pixel matches the band's nodata value.

### Valid Call Patterns
```python
# Example using the Python SQL connection
query = """
SELECT RS_Values(
  RS_Example(),
  ST_GeomFromText('MULTIPOINT (74.58 110.57, 500 500)', 'OGC:CRS84'),
  1
)
"""
result = sd.sql(query).to_arrow_table()
# __CHECK__ witness: result contains a list of values
```

### LLM Instruction Prompt
- Use `RS_Values` when you need to extract multiple pixel values from a raster based on a set of spatial coordinates.
- Always specify the `band` index if the raster is known to be multiband to avoid ambiguity errors.
- Remember that the function performs no resampling; it samples the specific pixel containing the point.

### Prompt Snippet
```text
RS_Values(raster_column, geometry_column, band_index)
```

### Common Failure Modes
- **Ambiguity Error:** Calling `RS_Values` on a multiband raster without specifying the `band` parameter.
- **NULL Results:** Points falling outside the raster bounds or hitting pixels defined as "nodata" will return `NULL` for that specific list element.
- **CRS Mismatch:** Providing geometries and rasters with incompatible CRSs that cannot be reprojected.
- **Unsupported Dimensions:** Attempting to use this function on non-2-D rasters.

### Fix Code Hint
```python
# Ensure the band is specified for multiband rasters
sd.sql("SELECT RS_Values(my_raster, my_points, 1) FROM my_table").to_arrow_table()
```

## API Test: `RS_Width`

### Signature
```python
RS_Width(arg1: raster) -> bigint
```
_Source: source/docs/reference/sql/rs_width.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: RS_Width
description: Returns the width of a raster in pixels.
kernels:
  - returns: bigint
    args: [raster]
---

## Examples

```sql
SELECT RS_Width(RS_Example());
```


### Goal
Returns the horizontal dimension of a raster object measured in pixels.

### Parameters
- `arg1` (`raster`): The input raster object for which the pixel width is to be calculated.

### Input
The function accepts a `raster` object. This can be generated via `RS_FromPath`, `RS_FromGDALRaster`, `RS_MakeEmptyRaster`, or returned by other raster-processing functions like `RS_Clip` or `RS_Resample`.

### Output
Returns `bigint` — The total number of pixels along the x-axis (width) of the raster.

### Valid Call Patterns
```python
# Example using the official SQL interface
query = "SELECT RS_Width(RS_Example())"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a bigint column representing the raster width.
```

### LLM Instruction Prompt
- Always use `RS_Width` as a standalone function in the SQL `SELECT` clause, passing a valid `raster` object as the argument.
- Ensure the input is a valid raster type; if the input is a path, wrap it in `RS_FromPath`.

### Prompt Snippet
```text
RS_Width(raster_expression)
```

### Common Failure Modes
- Passing a non-raster type (e.g., a geometry or string) will result in a SQL execution error.
- Attempting to call `RS_Width` on a null raster input may return null or trigger an error depending on the query context.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
query = "SELECT RS_Width(RS_FromPath('/path/to/raster.tif'))"
```

## API Test: `RS_Within`

### Signature
```python
RS_Within(arg1: raster, arg2: geometry) -> boolean | RS_Within(arg1: geometry, arg2: raster) -> boolean | RS_Within(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_within.qmd:19_

### Goal
Determines if the convex hull of the first spatial object is completely contained within the extent of the second spatial object.

### Parameters
- `arg1` (`raster`): The primary raster object to evaluate.
- `arg2` (`geometry`): The geometry object to check against the extent of the first argument. Note: This parameter also accepts `raster` types as per the function overloads.

### Input
- **Data Types:** Accepts combinations of `raster` and `geometry` types.
- **Preconditions:** If both arguments have defined CRSes, they must be compatible for transformation. If CRSes differ, the geometry is transformed to the raster's CRS (or the second raster's extent is transformed to the first). If no CRS is set on either, the comparison proceeds without transformation.

### Output
Returns `boolean` — `true` if the convex hull of the first argument is entirely within the second argument, otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify geometry within raster
result = sd.sql("SELECT RS_Within(ST_Point(0.5, 0.5, 4326), RS_Example())").to_arrow_table()
print(result)
# __CHECK__

# Verify raster within raster
result = sd.sql("SELECT RS_Within(RS_Example(), RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_Within(A, B)` to check if the convex hull of `A` is contained by `B`.
- Remember that `RS_Within(A, B)` is equivalent to `RS_Contains(B, A)`.
- Always ensure CRS compatibility or acknowledge that SedonaDB will attempt a WGS 84 fallback if preferred transformations fail.

### Prompt Snippet
```text
RS_Within(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** If arguments have different CRSes and no valid transformation path exists, the operation may fail or produce unexpected results.
- **Type Constraints:** Passing unsupported types (e.g., non-spatial types) will result in a SQL execution error.
- **Empty Inputs:** Passing an empty raster or null geometry may result in a `null` or `false` return value depending on the engine state.

### Fix Code Hint
```python
# Ensure CRS alignment before calling if transformation fails
sd.sql("SELECT RS_Within(raster_col, ST_Transform(geom_col, 4326)) FROM my_table")
```

## API Test: `RS_WorldToRasterCoord`

### Signature
```python
RS_WorldToRasterCoord(arg1: raster, x: double, y: double) -> geometry | RS_WorldToRasterCoord(arg1: raster, point: geometry) -> geometry
```
_Source: source/docs/reference/sql/rs_worldtorastercoord.qmd:19_

### Goal
Converts world coordinates (geospatial) into 1-based pixel coordinates within the raster's grid system.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform metadata required for coordinate conversion.
- `x` (`double`): The world X-coordinate (e.g., longitude or easting).
- `y` (`double`): The world Y-coordinate (e.g., latitude or northing).

### Input
- **Data:** A valid `raster` object (e.g., created via `RS_FromGDALRaster` or `RS_Example()`).
- **Preconditions:** The world coordinates are interpreted within the raster's own coordinate space. This function does not perform reprojection; if the world coordinates are in a different CRS than the raster, they must be transformed prior to calling this function.

### Output
Returns `geometry` — A `Point` geometry representing the 1-based pixel coordinate `(column, row)`. Coordinates inside the upper-left pixel map to `(1, 1)`. Values outside the raster extent are extrapolated based on the geotransform.

### Valid Call Patterns
```python
# Using explicit x, y coordinates
sd.sql("SELECT RS_WorldToRasterCoord(RS_Example(), 10.0, 20.0)").to_arrow_table()

# Using a point geometry
sd.sql("SELECT RS_WorldToRasterCoord(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoord` to map geospatial locations to pixel indices.
- Always provide the raster as the first argument.
- Choose between the `(raster, x, y)` signature or the `(raster, point)` signature based on whether the input is provided as separate doubles or a pre-constructed `ST_Point`.
- Remember that the output is a 1-based coordinate system.

### Prompt Snippet
```text
RS_WorldToRasterCoord(raster_col, x_val, y_val)
RS_WorldToRasterCoord(raster_col, ST_Point(x, y))
```

### Common Failure Modes
- **CRS Mismatch:** Providing world coordinates in a CRS that does not match the raster's CRS will result in incorrect pixel mapping, as no automatic reprojection occurs.
- **Invalid Raster:** Passing a null or improperly initialized raster object will cause the query to fail.
- **Extrapolation:** Users may be surprised by 0 or negative results if the provided world coordinates fall outside the raster's spatial extent.

### Fix Code Hint
```python
# Ensure coordinates are in the same CRS as the raster before conversion
sd.sql("""
    SELECT RS_WorldToRasterCoord(
        my_raster, 
        ST_Transform(ST_Point(10.0, 20.0), RS_CRS(my_raster))
    ) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `RS_WorldToRasterCoordX`

### Signature
```python
RS_WorldToRasterCoordX(arg1: raster, x: double, y: double) -> integer | RS_WorldToRasterCoordX(arg1: raster, point: geometry) -> integer
```
_Source: source/docs/reference/sql/rs_worldtorastercoordx.qmd:19_

## Description

The result is a 1-based pixel coordinate, as in PostGIS
`ST_WorldToRasterCoord`: a world coordinate inside the upper-left pixel maps to
`(1, 1)`. A world coordinate outside the raster extrapolates along the
geotransform, so it can map to `0` or a negative coordinate.

## Examples

```sql
SELECT RS_WorldToRasterCoordX(RS_Example(), 10.0, 20.0);
```

The world coordinate can also be given as a point:

```sql
SELECT RS_WorldToRasterCoordX(RS_Example(), ST_Point(10.0, 20.0));
```


### Goal
Converts a given world coordinate (either as X/Y doubles or a geometry point) into the corresponding 1-based column index (X coordinate) within the provided raster's grid.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform metadata required for coordinate conversion.
- `x` (`double`): The world X-coordinate to convert.
- `y` (`double`): The world Y-coordinate to convert.
- `point` (`geometry`): A geometry point containing the world coordinates to convert; the point is used in the raster's own space without reprojection.

### Input
- **Data:** A valid `raster` object (e.g., from `RS_FromGDALRaster` or `RS_Example`).
- **Preconditions:** The world coordinates must be provided in the same coordinate reference system (CRS) as the raster. This function does not perform reprojection.

### Output
Returns `integer` — The 1-based column index (X-coordinate) corresponding to the provided world location. Values outside the raster bounds are extrapolated based on the raster's geotransform.

### Valid Call Patterns
```python
# Using X and Y coordinates
result = sd.sql("SELECT RS_WorldToRasterCoordX(RS_Example(), 10.0, 20.0)").to_arrow_table()
print(result) # __CHECK__

# Using a geometry point
result = sd.sql("SELECT RS_WorldToRasterCoordX(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
print(result) # __CHECK__
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoordX` to map spatial world coordinates to raster grid columns.
- Choose between the `(raster, x, y)` signature or the `(raster, point)` signature based on available input types.
- Remember that the output is a 1-based index; a result of 1 corresponds to the first column.

### Prompt Snippet
```text
RS_WorldToRasterCoordX(raster_col, x_val, y_val)
RS_WorldToRasterCoordX(raster_col, ST_Point(x, y))
```

### Common Failure Modes
- **CRS Mismatch:** Providing coordinates in a different CRS than the raster will result in incorrect pixel mapping, as no automatic reprojection occurs.
- **Invalid Raster:** Passing a null or improperly initialized raster object will cause the query to fail.
- **Extrapolation:** Coordinates far outside the raster extent will return valid integers (including 0 or negative values) based on the geotransform, which may be logically invalid for specific application requirements.

### Fix Code Hint
```python
# Ensure the point is in the same CRS as the raster before conversion
sd.sql("SELECT RS_WorldToRasterCoordX(my_raster, ST_Transform(my_point, RS_CRS(my_raster)))")
```

## API Test: `RS_WorldToRasterCoordY`

### Signature
```python
RS_WorldToRasterCoordY(arg1: raster, x: double, y: double) -> integer | RS_WorldToRasterCoordY(arg1: raster, point: geometry) -> integer
```
_Source: source/docs/reference/sql/rs_worldtorastercoordy.qmd:19_

## Description

The result is a 1-based pixel coordinate, as in PostGIS
`ST_WorldToRasterCoord`: a world coordinate inside the upper-left pixel maps to
`(1, 1)`. A world coordinate outside the raster extrapolates along the
geotransform, so it can map to `0` or a negative coordinate.

## Examples

```sql
SELECT RS_WorldToRasterCoordY(RS_Example(), 10.0, 20.0);
```

The world coordinate can also be given as a point:

```sql
SELECT RS_WorldToRasterCoordY(RS_Example(), ST_Point(10.0, 20.0));
```


### Goal
Converts a given world coordinate (either as X/Y doubles or a geometry point) into the corresponding 1-based Y-axis pixel coordinate of a raster.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform metadata required for coordinate conversion.
- `x` (`double`): The world X coordinate (used only in the 3-argument overload).
- `y` (`double`): The world Y coordinate (used only in the 3-argument overload).
- `point` (`geometry`): A geometry point representing the world coordinate (used in the 2-argument overload).

### Input
- **Data:** A valid `raster` object (e.g., created via `RS_FromGDALRaster` or `RS_Example`).
- **Preconditions:** The world coordinates provided must be in the same coordinate space as the raster. This function does not perform reprojection.

### Output
Returns `integer` — The 1-based Y-axis pixel coordinate corresponding to the provided world location. Values outside the raster bounds are extrapolated based on the raster's geotransform.

### Valid Call Patterns
```python
# Using X and Y coordinates
sd.sql("SELECT RS_WorldToRasterCoordY(RS_Example(), 10.0, 20.0)").to_arrow_table()

# Using a geometry point
sd.sql("SELECT RS_WorldToRasterCoordY(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoordY` to map spatial locations to pixel indices.
- Always ensure the input raster is valid and the coordinates are in the raster's native coordinate system.
- Prefer the `ST_Point` overload when working with existing geometry columns.

### Prompt Snippet
```text
RS_WorldToRasterCoordY(raster_col, x_val, y_val)
RS_WorldToRasterCoordY(raster_col, ST_Point(x, y))
```

### Common Failure Modes
- **Coordinate Mismatch:** Providing coordinates in a different CRS than the raster will result in incorrect pixel mapping, as no reprojection is performed.
- **Invalid Raster:** Passing a null or improperly initialized raster object will cause the function to fail.
- **Extrapolation:** Be aware that coordinates outside the raster extent will return values (0 or negative) that may not correspond to valid pixel data.

### Fix Code Hint
```python
# Ensure coordinates are in the raster's CRS before calling
# If necessary, use ST_Transform to align the point to the raster's CRS first
sd.sql("SELECT RS_WorldToRasterCoordY(my_raster, ST_Transform(my_point, RS_CRS(my_raster)))").to_arrow_table()
```

__CHECK__

## API Test: `RS_ZonalStats`

### Signature
```python
RS_ZonalStats(arg1: raster, roi: geometry, stat_type: string) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean, exclude_no_data: boolean) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean, exclude_no_data: boolean, lenient: boolean) -> double
```
_Source: source/docs/reference/sql/rs_zonalstats.qmd:19_

### Goal
Computes a single summary statistic (e.g., mean, count, min, max) for raster pixels contained within a specified region of interest (ROI) geometry.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `roi` (`geometry`): The region-of-interest geometry used to clip the raster area.
- `band` (`integer`): The 1-based index of the raster band to process. Required for multiband rasters.
- `stat_type` (`string`): The statistic to compute: `count`, `sum`, `mean`, `median`, `mode`, `stddev`, `variance`, `min`, or `max`.
- `all_touched` (`boolean`): If true, includes all pixels touched by the ROI; if false (default), only includes pixels whose center falls within the ROI.
- `exclude_no_data` (`boolean`): If true (default), ignores pixels matching the band's nodata value.
- `lenient` (`boolean`): If true (default), returns NULL if the ROI does not intersect the raster; if false, raises an error.

### Input
- **Raster:** Must be a valid SedonaDB raster object (e.g., loaded via `RS_FromPath` or `RS_FromGDALRaster`).
- **Geometry:** Must be a valid geometry. If both the raster and geometry have a CRS, they must match or be reprojectable; it is an error if exactly one has a CRS.
- **Preconditions:** The function operates on 2-D (y, x) bands.

### Output
Returns a `double` representing the calculated statistic. `count` is returned as a whole number; all other statistics are floating point.

### Valid Call Patterns
```python
# Example using the standard 4-argument overload
query = """
SELECT RS_ZonalStats(
  RS_Example(),
  ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'),
  1,
  'mean'
)
"""
result = sd.sql(query).to_arrow_table()
# __CHECK__ witness: result contains the mean value of the specified band
```

### LLM Instruction Prompt
- Always specify the `band` index for multiband rasters.
- Ensure the `stat_type` string is one of the supported values (case-insensitive).
- When using `all_touched`, `exclude_no_data`, or `lenient`, ensure the argument order matches the signature exactly.
- Verify that the raster and geometry share the same CRS or that neither has one.

### Prompt Snippet
```text
RS_ZonalStats(raster_col, roi_geom, band_index, 'stat_name', all_touched_bool, exclude_nodata_bool, lenient_bool)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and raster with mismatched CRS definitions will cause an error.
- **Missing Band Index:** Omitting the `band` argument on a multiband raster.
- **Invalid Statistic:** Providing a `stat_type` not in the supported list (e.g., 'median' is supported, but 'skewness' is not).
- **Non-intersecting ROI:** If `lenient` is set to `false` and the ROI does not intersect the raster, the query will fail.

### Fix Code Hint
```python
# Ensure CRS alignment before calling
query = """
SELECT RS_ZonalStats(
    raster_col, 
    ST_Transform(roi_geom, RS_CRS(raster_col)), 
    1, 
    'mean'
) FROM my_table
"""
```

## API Test: `RS_ZonalStatsAll`

### Signature
```python
RS_ZonalStatsAll(arg1: raster, roi: geometry) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean, exclude_no_data: boolean) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean, exclude_no_data: boolean, lenient: boolean) -> struct
```

### Goal
Computes a comprehensive set of summary statistics (count, sum, mean, median, mode, stddev, variance, min, max) for raster pixels covered by a specified geometry.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `roi` (`geometry`): The region-of-interest geometry used to clip the raster pixels.
- `band` (`integer`): The 1-based index of the raster band to process. Required for multiband rasters.
- `all_touched` (`boolean`): If true, includes every pixel the ROI touches; if false, only includes pixels whose center falls within the ROI. Defaults to `false`.
- `exclude_no_data` (`boolean`): If true, ignores pixels matching the band's nodata value. Defaults to `true`.
- `lenient` (`boolean`): If true, returns `NULL` when the ROI does not intersect the raster; if false, raises an error. Defaults to `true`.

### Input
- **Raster:** Must be a valid 2-D `(y, x)` raster.
- **Geometry:** Must be a valid geometry. If both the raster and geometry have a CRS, they must match or be reprojectable; it is an error if exactly one side has a CRS.
- **Preconditions:** Pixel coordinates are 1-based.

### Output
Returns a `struct` containing the fields: `count` (64-bit integer), `sum`, `mean`, `median`, `mode`, `stddev`, `variance`, `min`, and `max` (all floating point).

### Valid Call Patterns
```python
# Example using the band-specific overload
query = """
SELECT RS_ZonalStatsAll(
  RS_Example(),
  ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'),
  1
)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always specify the `band` index for multiband rasters.
- Ensure the `roi` geometry and the raster share the same CRS or lack one entirely to avoid projection errors.
- Use `all_touched=True` if the analysis requires including pixels partially covered by the ROI boundary.

### Prompt Snippet
```text
RS_ZonalStatsAll(raster_col, roi_geom, 1, True, True, True)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and raster where only one has a defined CRS will trigger an error.
- **Missing Band Index:** Omitting the `band` argument on a multiband raster will cause an error.
- **Non-Intersection:** If `lenient` is set to `false` and the ROI does not intersect the raster, the query will fail.
- **Experimental Status:** As an experimental feature, behavior may change; verify results in the current environment.

### Fix Code Hint
```python
# Ensure CRS alignment before calling
query = """
SELECT RS_ZonalStatsAll(
    raster_col, 
    ST_Transform(roi_geom, RS_CRS(raster_col)), 
    1
) FROM my_table
"""
```

## API Test: `S2_CellIdFromPoint`

### Signature
```python
S2_CellIdFromPoint(arg1: geography) -> int64
```
_Source: source/docs/reference/sql/s2_cellidfrompoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: S2_CellIdFromPoint
description: Returns the S2 cell ID containing the given point geography at the maximum S2 level (30).
kernels:
  - returns: int64
    args: [geography]
---

## Description

Returns the 64-bit S2 cell ID at level 30 (the finest resolution) that contains the given point geography. Returns NULL for empty point geometries. This is useful for spatial indexing and efficient spatial queries.

## Examples

```sql
SELECT S2_CellIdFromPoint(ST_GeogPoint(-64, 45));
```


### Goal
Computes the unique 64-bit S2 cell identifier for a given point geography at the maximum resolution (level 30) to facilitate spatial indexing and efficient query performance.

### Parameters
- `arg1` (`geography`): A point geography object representing the location for which the S2 cell ID is calculated.

### Input
- **Data Type:** Must be a `geography` type.
- **Preconditions:** The input must be a valid point geometry. If the input is an empty point geometry, the function returns `NULL`.

### Output
Returns `int64` — A 64-bit integer representing the S2 cell ID at level 30.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using the SQL interface
query = "SELECT S2_CellIdFromPoint(ST_GeogPoint(-64, 45)) as cell_id"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: cell_id is an int64
```

### LLM Instruction Prompt
- Always use `S2_CellIdFromPoint` with a `geography` type input.
- Ensure the input is a point; non-point geometries are not supported by this function.
- Use this function when generating SQL for spatial indexing or partitioning tasks.

### Prompt Snippet
```text
S2_CellIdFromPoint(geography_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` type instead of a `geography` type may result in execution errors depending on the session's implicit casting rules.
- **Empty Input:** Passing an empty point geometry will return `NULL` rather than an integer, which may cause downstream errors if the result is expected to be non-null.

### Fix Code Hint
```python
# Ensure input is cast to geography if it is currently geometry
sd.sql("SELECT S2_CellIdFromPoint(ST_GeographyFromText(ST_AsText(geom))) FROM my_table")
```

## API Test: `S2_CoveringCellIds`

### Signature
```python
S2_CoveringCellIds(arg1: geography) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64, max_level: int64) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64, max_level: int64, max_cells: int64) -> array
```
_Source: source/docs/reference/sql/s2_coveringcellids.qmd:19_

## Description

Returns an array of 64-bit S2 cell IDs that form a covering of the given geography. A covering is a set of cells that completely contains the geography with minimal overlap. For a point, returns an array with a single cell ID. For empty geometries, returns an empty array.

This is useful for spatial indexing and efficient spatial queries. The returned cells may be at different S2 levels depending on the size and shape of the geography.

### Goal
Generates a set of S2 cell IDs that provide a spatial covering for a given geography, enabling efficient indexing and spatial query optimization within SedonaDB.

### Parameters
- `arg1` (`geography`): The input geography object to be covered by S2 cells.
- `min_level` (`int64`): The minimum S2 cell level (0 to 30). Defaults to 0.
- `max_level` (`int64`): The maximum S2 cell level (0 to 30). Defaults to 30.
- `max_cells` (`int64`): The maximum number of cells to return. Must be greater than 0. Defaults to 8.

### Input
The function accepts a `geography` type. Input geometries should be valid; empty geometries will result in an empty array. Ensure the input is correctly formatted as a geography (e.g., via `ST_GeogFromWKT`).

### Output
Returns `array` — An array of 64-bit integers representing the S2 cell IDs that cover the input geography.

### Valid Call Patterns
```python
# Basic usage
sd.sql("SELECT S2_CoveringCellIds(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'))").to_arrow_table()

# Advanced usage with level and cell constraints
sd.sql("SELECT S2_CoveringCellIds(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'), 10, 12, 16)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_GeogFromWKT` or similar constructors to ensure the input is of type `geography`.
- When specifying levels, ensure `min_level` and `max_level` are within the valid range of 0 to 30.
- `max_cells` must be a positive integer.

### Prompt Snippet
```text
Use S2_CoveringCellIds(geography, min_level, max_level, max_cells) to generate spatial index keys for the provided geography.
```

### Common Failure Modes
- **Invalid Level Range:** Providing `min_level` or `max_level` outside the 0–30 range will cause an execution error.
- **Invalid `max_cells`:** Providing a value less than or equal to 0 for `max_cells` will result in an error.
- **Type Mismatch:** Passing a standard `geometry` type instead of a `geography` type may lead to unexpected behavior or errors depending on the session's implicit casting rules.

### Fix Code Hint
```python
# Ensure geography type and valid level constraints
result = sd.sql("""
    SELECT S2_CoveringCellIds(
        ST_GeogFromWKT('POINT(0 0)'), 
        0, 30, 8
    )
""").to_arrow_table()
print("__CHECK__")
```

## API Test: `SD_Format`

### Signature
```python
SD_Format(value: any, options: string) -> any
```
_Source: source/docs/reference/sql/sd_format.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: SD_Format
description: Formats values for display with optional JSON options.
kernels:
  - returns: any
    args:
    - name: value
      type: any
    - name: options
      type: string
---

## Description

Returns a formatted representation of the input value. When `options` is provided, it should be a JSON string containing formatting hints such as `width_hint`.

## Examples

```sql
SELECT SD_Format(ST_Point(1.0, 2.0), '{"width_hint": 16}');
```


### Goal
Provides a human-readable or display-optimized string representation of geospatial or scalar data types using JSON-based configuration hints.

### Parameters
- `value` (`any`): The input object (e.g., geometry, raster, or scalar) to be formatted.
- `options` (`string`): A JSON-formatted string containing formatting parameters, such as `width_hint` to control the display width.

### Input
- **Data:** Any valid SedonaDB data type (Geometry, Raster, or primitive).
- **Preconditions:** The `options` parameter must be a valid JSON string.

### Output
Returns `any` — A formatted representation of the input, typically as a string, adjusted according to the provided JSON hints.

### Valid Call Patterns
```python
# Example using sd.sql to format a point geometry
result = sd.sql("SELECT SD_Format(ST_Point(1.0, 2.0), '{\"width_hint\": 16}')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `SD_Format` when you need to generate a display-friendly string for a geometry or raster object.
- Always ensure the `options` argument is a valid JSON string.
- If no specific formatting is required, pass an empty JSON object `'{}'`.

### Prompt Snippet
```text
SD_Format(value, '{"width_hint": <integer>}')
```

### Common Failure Modes
- **Invalid JSON:** Passing a string that is not valid JSON in the `options` parameter will cause a query execution error.
- **Unsupported Types:** Passing complex objects that do not have a defined string representation may result in default or empty output.

### Fix Code Hint
```python
# Ensure options are properly escaped if using Python strings
options_json = '{"width_hint": 20}'
query = f"SELECT SD_Format(geom, '{options_json}') FROM my_table"
```

## API Test: `SD_Order`

### Signature
```python
SD_Order(value: any) -> any
```
_Source: source/docs/reference/sql/sd_order.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: SD_Order
description: Returns a value suitable for ordering or sorting.
kernels:
  - returns: any
    args:
    - name: value
      type: any
---

## Examples

```sql
SELECT city, geom, SD_Order(geom)
FROM (
  VALUES
    ('San Francisco', ST_Point(-122.4194, 37.7749)),
    ('Los Angeles',   ST_Point(-118.2437, 34.0522)),
    ('New York',      ST_Point(-73.9857, 40.7484)),
    ('Chicago',       ST_Point(-87.6298, 41.8781)),
    ('Houston',       ST_Point(-95.3698, 29.7604)),
    ('Phoenix',       ST_Point(-112.0740, 33.4484)),
    ('Portland',      ST_Point(-122.6765, 45.5231)),
    ('Miami',         ST_Point(-80.1918, 25.7617)),
    ('Denver',        ST_Point(-104.9903, 39.7392)),
    ('Boston',        ST_Point(-71.0589, 42.3601))
) AS my_table(city, geom);
```


### Goal
Generates a sortable representation of a geometry or other spatial object to facilitate deterministic ordering or grouping operations within SedonaDB.

### Parameters
- `value` (`any`): The input geometry or spatial object to be transformed into a sortable format.

### Input
The function accepts standard SedonaDB geometry types (e.g., Point, Polygon). No specific file format is required as input, provided the data is materialized as a geometry object within the SQL engine.

### Output
Returns `any` — A value that can be used as a key for `ORDER BY` or `GROUP BY` clauses in SQL queries.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using SD_Order to sort geometries
query = """
SELECT city, geom 
FROM (VALUES ('SF', ST_Point(-122.4, 37.7)), ('LA', ST_Point(-118.2, 34.0))) AS t(city, geom)
ORDER BY SD_Order(geom)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `SD_Order` when you need to perform an `ORDER BY` or `GROUP BY` on a column containing geometry objects.
- Do not assume the return type is a specific numeric or string type; treat it as an opaque sortable key.

### Prompt Snippet
```text
Use SD_Order(geometry_column) in the ORDER BY clause to sort spatial data.
```

### Common Failure Modes
- Attempting to use `SD_Order` on non-spatial types that do not have a defined ordering implementation in the underlying engine.
- Expecting the output to be a human-readable coordinate string; the output is an internal representation for sorting.

### Fix Code Hint
```python
# If sorting fails, ensure the input is a valid geometry object
# and not a null or unsupported type.
sd.sql("SELECT SD_Order(geom) FROM table WHERE geom IS NOT NULL")
```

## API Test: `ST_Affine`

### Signature
```python
ST_Affine(geom: geometry, a: double, b: double, d: double, e: double, xOff: double, yOff: double) -> geometry | ST_Affine(geom: geometry, a: double, b: double, c: double, d: double, e: double, f: double, g: double, h: double, i: double, xOff: double, yOff: double, zOff: double) -> geometry
```
_Source: source/docs/reference/sql/st_affine.qmd:19_

### Goal
Applies a 2D or 3D affine transformation (scaling, rotation, shearing, and translation) to a geometry object.

### Parameters
- `geom` (`geometry`): The input geometry to be transformed.
- `a` (`double`): The coefficient for the x-dimension (scale/rotation).
- `b` (`double`): The coefficient for the x-dimension (shear/rotation).
- `c` (`double`): The coefficient for the x-dimension (3D only).
- `d` (`double`): The coefficient for the y-dimension (shear/rotation).
- `e` (`double`): The coefficient for the y-dimension (scale/rotation).
- `f` (`double`): The coefficient for the y-dimension (3D only).
- `g` (`double`): The coefficient for the z-dimension (3D only).
- `h` (`double`): The coefficient for the z-dimension (3D only).
- `i` (`double`): The coefficient for the z-dimension (3D only).
- `xOff` (`double`): The translation offset for the x-axis.
- `yOff` (`double`): The translation offset for the y-axis.
- `zOff` (`double`): The translation offset for the z-axis (3D only).

### Input
The function accepts a `geometry` type (e.g., Point, Polygon) created via standard SedonaDB constructors like `ST_GeomFromText`. The transformation coefficients are provided as `double` values.

### Output
Returns a new `geometry` object representing the transformed input.

### Valid Call Patterns
```python
# 2D Affine Transformation
result = sd.sql("SELECT ST_AsText(ST_Affine(ST_GeomFromText('POINT (1 2)'), 1, 0, 0, 1, 10, 20))").to_arrow_table()
print(result)
# __CHECK__ witness: POINT (11 22)
```

### LLM Instruction Prompt
- Use `ST_Affine` to perform geometric transformations.
- Choose the 7-argument overload for 2D transformations and the 13-argument overload for 3D transformations.
- Ensure all numeric parameters are provided as `double` types.

### Prompt Snippet
```text
ST_Affine(geom, a, b, d, e, xOff, yOff)
ST_Affine(geom, a, b, c, d, e, f, g, h, i, xOff, yOff, zOff)
```

### Common Failure Modes
- Providing an incorrect number of arguments for the intended dimension (e.g., using 7 arguments when 3D coefficients are required).
- Passing non-numeric types for the transformation coefficients.
- Attempting to transform a geometry that is null or invalid.

### Fix Code Hint
```python
# Ensure the number of arguments matches the dimension of the geometry
# For 2D:
sd.sql("SELECT ST_Affine(geom, 1.0, 0.0, 0.0, 1.0, 10.0, 20.0) FROM table")
```

## API Test: `ST_Analyze_Agg`

### Signature
```python
ST_Analyze_Agg(arg1: geometry) -> struct | ST_Analyze_Agg(arg1: geography) -> struct
```
_Source: source/docs/reference/sql/st_analyze_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Analyze_Agg
description: Computes the statistics of geometries for the input geometry or geography.
kernels:
  - returns: struct
    args: [geometry]
  - returns: struct
    args: [geography]
---

## Description

`ST_Analyze_Agg()` provides a high-level summary of its input geometries. The fields of
its struct return type are:

- `count`: Number of input geometries
- `minx`, `miny`, `maxx`, `maxy`: Minimum bounding rectangle (envelope). For geography, this represents geographical bounds such that minx may be greater than maxx where this represents more compact bounds (e.g., a small feature crossing the antimeridian).
- `mean_size_in_bytes`
- `mean_points_per_geometry`
- `puntal_count`: Number of point or multipoint geometries
- `lineal_count`: Number of line or multilinestring geometries
- `polygonal_count`: Number of polygon or multipolygon geometries
- `geometrycollection_count`: Number of geometrycollection geometries
- `mean_envelope_width`
- `mean_envelope_height`
- `mean_envelope_area`

## Examples

```sql
SELECT ST_Analyze_Agg(
    ST_GeomFromText('MULTIPOINT(1.1 101.1,2.1 102.1,3.1 103.1,4.1 104.1,5.1 105.1,6.1 106.1,7.1 107.1,8.1 108.1,9.1 109.1,10.1 110.1)')
  ) AS analyze
```

```sql
SELECT ST_Analyze_Agg(
    ST_GeogFromText('MULTIPOINT(1.1 101.1,2.1 102.1,3.1 103.1,4.1 104.1,5.1 105.1,6.1 106.1,7.1 107.1,8.1 108.1,9.1 109.1,10.1 110.1)')
  ) AS analyze
```


### Goal
`ST_Analyze_Agg` computes a comprehensive statistical summary of a collection of geometries or geographies, providing insights into their spatial distribution, complexity, and type composition.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object or column containing geometries/geographies to be analyzed.

### Input
The function accepts standard SedonaDB geometry or geography types. Inputs can be derived from WKT, WKB, or loaded datasets (e.g., GeoParquet). Ensure the input column contains valid spatial objects before aggregation.

### Output
Returns a `struct` containing 12 fields: `count`, `minx`, `miny`, `maxx`, `maxy`, `mean_size_in_bytes`, `mean_points_per_geometry`, `puntal_count`, `lineal_count`, `polygonal_count`, `geometrycollection_count`, `mean_envelope_width`, `mean_envelope_height`, and `mean_envelope_area`.

### Valid Call Patterns
```python
# Example using sd.sql() to retrieve the struct result
result = sd.sql("""
    SELECT ST_Analyze_Agg(ST_GeomFromText('MULTIPOINT(1.1 101.1, 2.1 102.1)')) AS stats
""").to_arrow_table()
print(result)
# __CHECK__ witness: The result is an Arrow Table containing a struct column 'stats'.
```

### LLM Instruction Prompt
- Always use `ST_Analyze_Agg` as a standalone SQL function within an `sd.sql()` query.
- Ensure the input is a valid geometry or geography type.
- When interpreting the output, remember that `minx` may be greater than `maxx` for geography types crossing the antimeridian.

### Prompt Snippet
```text
SELECT ST_Analyze_Agg(geometry_column) FROM table_name
```

### Common Failure Modes
- Passing non-spatial types (e.g., strings or integers) directly without casting via `ST_GeomFromText` or similar constructors.
- Attempting to use the function on a null column without handling nulls, which may result in unexpected aggregate behavior.

### Fix Code Hint
```python
# Ensure input is cast to geometry if coming from a raw format
query = "SELECT ST_Analyze_Agg(ST_GeomFromText(geom_wkt)) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_Area`

### Signature
```python
ST_Area(arg1: geometry) -> double | ST_Area(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_area.qmd:19_

## Description

`ST_Area()` Returns the area of a polygon or multi-polygon. For geometry, returns the area in the units of the coordinate reference system (squared); for geography, returns the area in square meters, calculated on a spherical Earth model.

## Examples

```sql
SELECT ST_Area(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'));
```

```sql
SELECT ST_Area(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'));
```

### Goal
Calculates the surface area of a polygon or multi-polygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (Polygon or MultiPolygon) for which the area is to be computed.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** The input must be a valid polygon or multi-polygon. If using `geometry`, the result is in the squared units of the coordinate reference system. If using `geography`, the result is in square meters based on a spherical Earth model.

### Output
Returns `double` — The calculated area of the input geometry or geography.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Calculate area of a geometry
result = sd.sql("SELECT ST_Area(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print(result)
# __CHECK__ witness: 1.0

# Calculate area of a geography
result_geog = sd.sql("SELECT ST_Area(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print(result_geog)
```

### LLM Instruction Prompt
- Use `ST_Area` to compute the area of polygon-based geometries.
- Distinguish between `geometry` (units squared) and `geography` (square meters) inputs.
- Always wrap the function call within a `sd.sql("...").to_arrow_table()` execution pattern.

### Prompt Snippet
```text
ST_Area(ST_GeomFromWKT('POLYGON ((...))'))
```

### Common Failure Modes
- **Invalid Geometry:** Passing a non-polygon type (e.g., Point or LineString) may result in an area of 0 or an error depending on the engine state.
- **CRS Mismatch:** If performing calculations on geometries, ensure the CRS is appropriate for area measurements, as `ST_Area` returns units squared based on the CRS definition.
- **Null Inputs:** Passing a null geometry will return a null result.

### Fix Code Hint
```python
# Ensure the input is a valid polygon and cast if necessary
sd.sql("SELECT ST_Area(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
```

## API Test: `ST_AsBinary`

### Signature
```python
ST_AsBinary(arg1: geometry) -> binary | ST_AsBinary(arg1: geography) -> binary
```
_Source: source/docs/reference/sql/st_asbinary.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_AsBinary
description: Converts a geometry or geography to Well-Known Binary (WKB) format.
kernels:
  - returns: binary
    args: [geometry]
  - returns: binary
    args: [geography]
---

## Description

`ST_AsBinary()` Returns the Well-Known Binary representation of a geometry or geography. This function also has the alias ST_AsWKB.

`ST_AsBinary()` may preserve the underlying Arrow storage type of the input geometry. For example,
when the input geometry is backed by Arrow `BinaryView` storage, the WKB output may also use
`BinaryView` storage. If a downstream consumer requires simpler Arrow storage such as `Binary`,
wrap the result with `sd_simplifystorage()`.

## Examples

```sql
SELECT ST_AsBinary(ST_Point(1.0, 2.0));
```

```sql
SELECT ST_AsBinary(ST_GeogPoint(1.0, 2.0));
```


### Goal
Converts a spatial object (geometry or geography) into its Well-Known Binary (WKB) representation for serialization or storage.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be converted to WKB format.

### Input
The function accepts any valid geometry or geography object generated by SedonaDB spatial constructors (e.g., `ST_Point`, `ST_GeomFromText`). No specific file format is required as input, provided the object is materialized in the SedonaDB SQL context.

### Output
Returns `binary` — A byte array representing the Well-Known Binary (WKB) format of the input spatial object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify WKB conversion for a geometry
result = sd.sql("SELECT ST_AsBinary(ST_Point(1.0, 2.0)) as wkb").to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a binary column representing the point.
```

### LLM Instruction Prompt
- Always use `ST_AsBinary` to serialize geometries for external consumption.
- If the downstream consumer expects standard `Binary` storage rather than `BinaryView`, wrap the call as `sd_simplifystorage(ST_AsBinary(arg1))`.
- Use the alias `ST_AsWKB` if preferred for compatibility with other OGC-compliant systems.

### Prompt Snippet
```text
Use ST_AsBinary(geometry_column) to convert spatial data to WKB. Wrap with sd_simplifystorage() if binary storage compatibility is required.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., string or integer) directly into the function will result in a SQL execution error.
- **Storage Incompatibility:** Downstream applications expecting strict `Binary` Arrow types may fail if the function returns `BinaryView` (the default behavior for some inputs). Use `sd_simplifystorage()` to resolve this.

### Fix Code Hint
```python
# If downstream consumer fails due to BinaryView storage:
query = "SELECT sd_simplifystorage(ST_AsBinary(geom)) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_AsEWKB`

### Signature
```python
ST_AsEWKB(arg1: geometry) -> binary | ST_AsEWKB(arg1: geography) -> binary
```
_Source: source/docs/reference/sql/st_asewkb.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_AsEWKB
description: Returns the EWKB representation of a geometry or geography.
kernels:
  - returns: binary
    args: [geometry]
  - returns: binary
    args: [geography]
---

## Description

EWKB extends WKB to include SRID information in the binary header and is useful for PostGIS compatibility. This function preserves item-level CRSes or will repeat a type-level CRS for all elements if present.

## Examples

```sql
SELECT ST_AsEWKB(ST_Point(1.0, 2.0, 4326));
```


### Goal
Converts a geometry or geography object into its Extended Well-Known Binary (EWKB) representation, including SRID metadata for compatibility with external systems like PostGIS.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be serialized into binary format.

### Input
- **Data:** A valid geometry or geography object created via SedonaDB constructors (e.g., `ST_Point`, `ST_GeomFromText`).
- **Preconditions:** The input must be a valid spatial type. If the object has an associated CRS, it will be encoded into the EWKB header.

### Output
Returns `binary` — A byte array representing the EWKB format of the input geometry or geography, including the SRID header.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify EWKB conversion
result = sd.sql("SELECT ST_AsEWKB(ST_Point(1.0, 2.0, 4326)) as ewkb").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_AsEWKB` when the user requires binary serialization of spatial data for interoperability with PostGIS or other systems that require SRID-aware binary formats.
- Always ensure the input is a valid geometry or geography type.

### Prompt Snippet
```text
Use ST_AsEWKB(geometry_column) to export spatial data to EWKB format.
```

### Common Failure Modes
- **Invalid Input:** Passing a non-spatial type (e.g., string or integer) to the function will result in a SQL execution error.
- **Null Inputs:** If the input geometry is `NULL`, the function will return `NULL`.
- **CRS Mismatch:** While the function handles CRS encoding, ensure the input geometry has a valid SRID defined if PostGIS compatibility is the goal.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry if it is currently a string
sd.sql("SELECT ST_AsEWKB(ST_GeomFromText('POINT(1 2)', 4326))").to_arrow_table()
```

## API Test: `ST_AsGeoJSON`

### Signature
```python
ST_AsGeoJSON(arg1: geometry) -> string
```
_Source: source/docs/reference/sql/st_asgeojson.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_AsGeoJSON
description: Returns the GeoJSON representation of a geometry.
kernels:
  - returns: string
    args: [geometry]
---

## Examples

```sql
SELECT ST_AsGeoJSON(ST_Point(1.0, 2.0));
```


### Goal
Converts a SedonaDB geometry object into its corresponding GeoJSON string representation for serialization or interoperability.

### Parameters
- `arg1` (`geometry`): The input geometry object (e.g., Point, Polygon, LineString) to be serialized.

### Input
The function accepts any valid geometry type supported by SedonaDB. Geometries can be constructed via SQL constructors (e.g., `ST_Point`, `ST_GeomFromText`) or loaded from supported vector formats like GeoParquet.

### Output
Returns `string` — A GeoJSON-formatted string representing the geometry object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using a SQL query to serialize a point
result = sd.sql("SELECT ST_AsGeoJSON(ST_Point(1.0, 2.0)) as geojson").to_arrow_table()
print(result)
# __CHECK__ witness: {"type":"Point","coordinates":[1.0,2.0]}
```

### LLM Instruction Prompt
- Always use `ST_AsGeoJSON` to convert geometry objects to string format when exporting data to JSON-based formats.
- Ensure the input is a valid geometry type; if the input is a string (WKT), wrap it in a constructor like `ST_GeomFromText` first.

### Prompt Snippet
```text
ST_AsGeoJSON(geometry_column)
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or a raw string) directly to the function will result in a SQL execution error.
- Attempting to serialize a null geometry will return a null result.

### Fix Code Hint
```python
# If input is WKT string, cast it to geometry first:
sd.sql("SELECT ST_AsGeoJSON(ST_GeomFromText('POINT(1 2)'))")
```

## API Test: `ST_AsText`

### Signature
```python
ST_AsText(arg1: geometry) -> string | ST_AsText(arg1: geography) -> string
```
_Source: source/docs/reference/sql/st_astext.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_AsText
description: Returns the Well-Known Text string representation of a geometry or geography.
kernels:
  - returns: string
    args: [geometry]
  - returns: string
    args: [geography]
---

## Description

Alias: `ST_AsWKT`.

## Examples

```sql
SELECT ST_AsText(ST_Point(1.0, 2.0));
```

```sql
SELECT ST_AsText(ST_GeogPoint(1.0, 2.0));
```


### Goal
Converts a SedonaDB geometry or geography object into its standard Well-Known Text (WKT) string representation for human-readable output or serialization.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be serialized into a WKT string.

### Input
The function accepts any valid geometry or geography object generated by SedonaDB spatial constructors (e.g., `ST_Point`, `ST_GeomFromText`). No specific file format is required as input, as the function operates on materialized spatial objects within the SQL engine.

### Output
Returns `string` — A Well-Known Text (WKT) string representing the geometry or geography (e.g., `'POINT (1.0 2.0)'`).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify WKT conversion of a point
result = sd.sql("SELECT ST_AsText(ST_Point(1.0, 2.0)) as wkt").to_arrow_table()
assert result["wkt"][0] == "POINT (1.0 2.0)"
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_AsText` when you need to inspect the coordinates or structure of a geometry object in a human-readable format.
- Use the alias `ST_AsWKT` if preferred, as it is functionally identical.
- Ensure the input is a valid spatial type; passing non-spatial types will result in a SQL execution error.

### Prompt Snippet
```text
Use ST_AsText(geometry_column) to convert spatial data to WKT format.
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-spatial type (e.g., integer, string, or raster) to `ST_AsText` will cause a query execution failure.
- **Null Inputs:** If the input geometry is `NULL`, the function will return `NULL`.
- **Complex Geometries:** Extremely large or complex geometries may result in very long strings; ensure the downstream consumer can handle the resulting string length.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_AsText(ST_GeomFromText(my_wkt_col)) FROM my_table")
```

## API Test: `ST_Azimuth`

### Signature
```python
ST_Azimuth(arg1: geometry, arg2: geometry) -> double
```
_Source: source/docs/reference/sql/st_azimuth.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Azimuth
description: Returns the azimuth between two points in radians, or NULL if not available.
kernels:
  - returns: double
    args: [geometry, geometry]
---

## Description

Returns the azimuth (angle) from the first point to the second point, measured in
**radians** clockwise from north. Returns NULL if either input is not a point.

## Examples

```sql
SELECT ST_Azimuth(ST_Point(0.0, 25.0), ST_Point(0.0, 0.0));
```


### Goal
Calculates the clockwise angle in radians from the North direction between two specified geometry points.

### Parameters
- `arg1` (`geometry`): The starting point geometry.
- `arg2` (`geometry`): The ending point geometry.

### Input
Requires two valid `geometry` objects. Inputs must be of type `Point`. If non-point geometries are provided, the function returns `NULL`.

### Output
Returns `double` — The azimuth angle in radians, measured clockwise from North.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify azimuth calculation
query = "SELECT ST_Azimuth(ST_Point(0.0, 25.0), ST_Point(0.0, 0.0)) as azimuth"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: azimuth is 3.141592653589793 (pi radians)
```

### LLM Instruction Prompt
- Always use `ST_Azimuth(point1, point2)` syntax within a `sd.sql()` call.
- Ensure both arguments are valid geometry points.
- Remember that the output is in radians, not degrees.

### Prompt Snippet
```text
ST_Azimuth(ST_Point(x1, y1), ST_Point(x2, y2))
```

### Common Failure Modes
- **Non-Point Input:** Passing a `Polygon` or `LineString` instead of a `Point` will result in a `NULL` return value.
- **Coordinate Mismatch:** While the function calculates the angle, ensure the points are in the same coordinate system to maintain geometric accuracy.

### Fix Code Hint
```python
# Ensure inputs are points
sd.sql("SELECT ST_Azimuth(ST_Centroid(geom), ST_Point(0,0)) FROM my_table")
```

## API Test: `ST_Boundary`

### Signature
```python
ST_Boundary(arg1: geometry) -> geometry | ST_Boundary(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_boundary.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Boundary
description: Returns the closure of the combinatorial boundary of this geometry or geography.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

For a polygon, the boundary is the exterior and interior rings; for a linestring it is the endpoints.

## Examples

```sql
SELECT ST_Boundary(ST_GeomFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'));
```

```sql
SELECT ST_Boundary(ST_GeogFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'));
```


### Goal
Computes the combinatorial boundary of a given geometry or geography object, returning the exterior/interior rings for polygons or endpoints for linestrings.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to process.

### Input
The function accepts standard geometry or geography types. Inputs are typically provided via WKT or WKB constructors (e.g., `ST_GeomFromWKT`, `ST_GeogFromWKT`). Ensure the input is a valid spatial object supported by SedonaDB.

### Output
Returns `geography | geometry` representing the boundary of the input. The output type matches the input type (geometry returns geometry, geography returns geography).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Geometry boundary
result = sd.sql("SELECT ST_Boundary(ST_GeomFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'))").to_arrow_table()
print("__CHECK__")

# Geography boundary
result = sd.sql("SELECT ST_Boundary(ST_GeogFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'))").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_Boundary` as a standalone function call within the `sd.sql()` string.
- Ensure the input is explicitly cast or constructed using SedonaDB spatial constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`.
- Do not attempt to call this as a method on a Python object; it must be invoked within the SQL query string.

### Prompt Snippet
```text
Use ST_Boundary(arg1) to extract the boundary of a geometry or geography.
```

### Common Failure Modes
- Passing a non-spatial type or a null value to the function.
- Mismatched types: attempting to use geometry-specific constructors for geography inputs or vice versa.
- Syntax errors: calling `geometry.ST_Boundary()` instead of `ST_Boundary(geometry)` inside the SQL string.

### Fix Code Hint
```python
# Ensure the input is a valid spatial type
sd.sql("SELECT ST_Boundary(ST_GeomFromWKT('LINESTRING(0 0, 1 1)'))").to_arrow_table()
```

## API Test: `ST_Buffer`

### Signature
```python
ST_Buffer(arg1: geometry, distance: float64) -> geometry | ST_Buffer(arg1: geometry, distance: float64, params: string) -> geometry | ST_Buffer(arg1: geography, distance: float64) -> geography | ST_Buffer(arg1: geography, distance: float64, num_quad_segs: integer) -> geography | ST_Buffer(arg1: geography, distance: float64, params: string) -> geography
```
_Source: source/docs/reference/sql/st_buffer.qmd:19_

### Goal
Computes a geometry or geography representing all points within a specified distance from the input geometry or geography.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to be buffered.
- `distance` (`float64`): The radius of the buffer. For `geography` types, this value is specified in meters. Negative values are supported for polygon inputs to shrink the geometry.
- `params` (`string`): Optional space-separated `key=value` parameters (e.g., `quad_segs`, `endcap`, `join`, `mitre_limit`, `side`) identical to PostGIS buffer parameter strings.

### Input
- **Data:** Accepts `geometry` or `geography` types.
- **Preconditions:** For `geography` inputs, the distance is interpreted as meters on a sphere. Ensure the input geometry is valid for the intended operation.

### Output
Returns a `geometry` or `geography` object representing the buffered area.

### Valid Call Patterns
```python
# Basic geometry buffer
result = sd.sql("SELECT ST_Buffer(ST_GeomFromText('POINT(0 0)'), 1.0)").to_arrow_table()

# Geometry buffer with parameters
result = sd.sql("SELECT ST_Buffer(geom, 1.0, 'quad_segs=2') FROM input").to_arrow_table()

# Geography buffer (distance in meters)
result = sd.sql("SELECT ST_Buffer(ST_GeogFromText('POINT(0 0)'), 1000)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Buffer` to expand or shrink geometries/geographies.
- Always specify `distance` as a `float64`.
- When using `geography`, remember that `distance` is in meters.
- Use the `params` string argument for advanced configuration like `quad_segs` or `endcap` styles.

### Prompt Snippet
```text
ST_Buffer(geometry_col, distance_float)
ST_Buffer(geography_col, distance_float, 'quad_segs=8')
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` when a `geography` is expected, or vice versa, may lead to unexpected results or errors depending on the CRS context.
- **Invalid Distance:** Providing a non-numeric value for `distance`.
- **Invalid Params:** Providing malformed `key=value` strings in the `params` argument.

### Fix Code Hint
```python
# Ensure distance is a float and params are correctly formatted
df = sd.sql("SELECT ST_Buffer(geom, 5.0, 'quad_segs=4') as buffered_geom FROM my_table").to_arrow_table()
print("__CHECK__")
```

## API Test: `ST_BuildArea`

### Signature
```python
ST_BuildArea(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_buildarea.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_BuildArea
description: Returns a geometry that encloses the area formed by the given linework.
kernels:
  - returns: geometry
    args: [geometry]
---

## Description

Treats the input geometry's edges as linework and assembles them into a polygon or multipolygon.
Open rings are closed automatically before assembly. Empty input returns an empty polygon, and
polygon or multipolygon input is returned unchanged. If no closed ring can be formed, returns NULL.

## Examples

```sql
SELECT ST_BuildArea(ST_GeomFromWKT('LINESTRING (0 0, 1 0, 1 1, 0 1, 0 0)'));
```


### Goal
`ST_BuildArea` reconstructs a polygon or multipolygon geometry by treating the input geometry's edges as linework.

### Parameters
- `arg1` (`geometry`): The input geometry (typically a `LINESTRING` or `MULTILINESTRING`) whose edges are used to form the area.

### Input
The function accepts standard geometry types. If the input is already a `POLYGON` or `MULTIPOLYGON`, it is returned unchanged. Open rings in the input linework are automatically closed.

### Output
Returns a `geometry` object representing the polygon or multipolygon formed by the input edges, or an empty polygon for empty input.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = "SELECT ST_BuildArea(ST_GeomFromWKT('LINESTRING (0 0, 1 0, 1 1, 0 1, 0 0)')) as geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Use `ST_BuildArea` when you need to convert a set of lines into a closed polygon area.
- Always pass a valid geometry object as the argument.
- Note that if the input linework cannot form a closed ring, the function will return `NULL`.

### Prompt Snippet
```text
ST_BuildArea(geometry_column)
```

### Common Failure Modes
- **NULL Return:** If the provided linework is insufficient to form any closed rings, the function returns `NULL`.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT/WKB will result in a query execution error.

### Fix Code Hint
```python
# Ensure the input is a valid linework geometry
# If the result is NULL, verify the input linework connectivity
query = "SELECT ST_BuildArea(ST_GeomFromWKT('...'))"
```

## API Test: `ST_CRS`

### Signature
```python
ST_CRS(arg1: geometry) -> string | ST_CRS(arg1: geography) -> string
```
_Source: source/docs/reference/sql/st_crs.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_CRS
description: Returns the Coordinate Reference System (CRS) metadata associated with a geometry or geography object.
kernels:
  - returns: string
    args: [geometry]
  - returns: string
    args: [geography]
---

## Description

Returns the CRS (Coordinate Reference System) string associated with a geometry
or geography. Whether it was set as an authority code (e.g. `'EPSG:3857'`),
PROJJSON, or WKT, the CRS is returned in the same form it was set.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT ST_CRS(ST_Point(0.25, 0.25, 4326)) AS crs_info;
```


### Goal
Retrieves the Coordinate Reference System (CRS) metadata string associated with a specific geometry or geography object to verify its spatial reference.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the CRS metadata is requested.

### Input
The input must be a valid geometry or geography object. SedonaDB tracks CRS information internally; the function extracts the metadata string (e.g., EPSG code, WKT, or PROJJSON) that was assigned to the object upon creation or transformation.

### Output
Returns `string` — The CRS metadata string in the exact format (authority code, WKT, or PROJJSON) in which it was originally defined for the object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify CRS of a point created with an EPSG code
query = "SELECT ST_CRS(ST_Point(0.25, 0.25, 4326)) AS crs_info"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_CRS` to inspect the spatial reference of a geometry or geography object when debugging coordinate mismatches.
- Ensure the input is a valid spatial object; do not pass raw strings or numbers directly without a constructor (e.g., `ST_Point`, `ST_GeomFromText`).

### Prompt Snippet
```text
Use ST_CRS(geometry_column) to retrieve the CRS metadata string for verification.
```

### Common Failure Modes
- **Null Input:** Passing a `NULL` geometry will return `NULL` rather than a CRS string.
- **Missing CRS:** If a geometry was created without an explicit CRS, the function may return an empty string or indicate that no CRS is defined.
- **Type Mismatch:** Attempting to pass non-spatial types (e.g., integers or strings) will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the geometry has a CRS defined if ST_CRS returns an empty result
query = "SELECT ST_CRS(ST_SetSRID(ST_Point(0, 0), 4326))"
```

## API Test: `ST_Centroid`

### Signature
```python
ST_Centroid(arg1: geometry) -> geometry | ST_Centroid(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_centroid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Centroid
description: Returns the centroid of a geometry or geography.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

For geography, the centroid is calculated using spherical geometry.

## Examples


```sql
SELECT ST_Centroid(
  ST_GeomFromWKT('LINESTRING (-64 45, 52 8)')
);
```


```sql
SELECT ST_Centroid(
  ST_GeogFromWKT('LINESTRING (-64 45, 52 8)')
);
```


### Goal
Calculates the geometric center (centroid) of a given geometry or geography object.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object for which the centroid is to be computed.

### Input
- **Data:** Accepts standard geometry types (e.g., `LINESTRING`, `POLYGON`) or geography types.
- **Formats:** Supports WKT, WKB, and GeoParquet inputs registered as views or literals.
- **Preconditions:** Ensure the input is a valid spatial object. For geography types, the calculation is performed using spherical geometry.

### Output
Returns `geography | geometry` — A point representing the centroid of the input object.

### Valid Call Patterns
```python
# SQL execution via Python
result = sd.sql("SELECT ST_Centroid(ST_GeomFromWKT('LINESTRING (-64 45, 52 8)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Centroid(geometry_column)` to compute the center point of spatial features.
- Distinguish between `geometry` and `geography` inputs; use `ST_GeogFromWKT` if spherical calculation is required.

### Prompt Snippet
```text
ST_Centroid(geometry_expression)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or an invalid WKT string will result in a query execution error.
- **Empty Geometries:** Passing an empty geometry may return a null or empty point depending on the underlying engine implementation.

### Fix Code Hint
```python
# Ensure input is valid geometry
sd.sql("SELECT ST_Centroid(ST_MakeValid(geom)) FROM table").to_arrow_table()
```

## API Test: `ST_ClosestPoint`

### Signature
```python
ST_ClosestPoint(arg1: geometry, arg2: geometry) -> geometry | ST_ClosestPoint(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_closestpoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ClosestPoint
description: Returns the 2-dimensional point on geom1 that is closest to geom2.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Examples

```sql
SELECT ST_ClosestPoint(
    ST_GeogFromText('LINESTRING(0 0, 10 10)'),
    ST_GeogPoint(5, 0)
);
```


### Goal
Returns the 2-dimensional point on the first geometry (or geography) that is closest to the second geometry (or geography).

### Parameters
- `arg1` (`geometry`): The reference geometry or geography object from which the closest point is calculated.
- `arg2` (`geometry`): The target geometry or geography object to which the distance is minimized.

### Input
- **Data Formats:** Accepts WKT, WKB, or standard geometry/geography types registered in SedonaDB.
- **Preconditions:** Both `arg1` and `arg2` must share the same type (both `geometry` or both `geography`). If using `geometry`, ensure the Coordinate Reference Systems (CRS) match; mismatched CRSs may require explicit reprojection.

### Output
Returns a `geometry` or `geography` point representing the location on `arg1` that is nearest to `arg2`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geography types
query = """
SELECT ST_ClosestPoint(
    ST_GeogFromText('LINESTRING(0 0, 10 10)'),
    ST_GeogPoint(5, 0)
) as closest_point
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure `arg1` and `arg2` are of the same spatial type (geometry or geography).
- If inputs are in different CRSs, use a reprojection function before calling `ST_ClosestPoint`.
- The function returns a point; ensure the downstream consumer expects a geometry/geography type.

### Prompt Snippet
```text
ST_ClosestPoint(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in a runtime error.
- **CRS Mismatch:** Providing geometries with different coordinate systems without prior reprojection will cause the operation to fail or produce incorrect results.
- **Null Inputs:** Passing `NULL` as either argument will return `NULL`.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type and CRS
query = """
SELECT ST_ClosestPoint(
    ST_Transform(geom1, 4326),
    ST_Transform(geom2, 4326)
) FROM my_table
"""
```

## API Test: `ST_Collect_Agg`

### Signature
```python
ST_Collect_Agg(arg1: geometry) -> geometry | ST_Collect_Agg(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_collect_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Collect_Agg
description: Combines multiple geometries or geographies from a set of rows into a single collection.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

An aggregate function that collects multiple geometries or geographies into a
single GeometryCollection or the appropriate Multi-type if all inputs share the
same geometry type. Collecting geographies returns a geography.

## Examples

```sql
SELECT ST_Collect_Agg(geom) AS collected_points
FROM (
  VALUES
    (ST_Point(-122.4194, 37.7749)),
    (ST_Point(-118.2437, 34.0522)),
    (ST_Point(-122.6765, 45.5231))
) AS cities(geom);
```


### Goal
Aggregates multiple individual geometry or geography objects from a set of rows into a single, unified collection object.

### Parameters
- `arg1` (`geometry`): The column or expression containing the geometries or geographies to be aggregated.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** All inputs must share the same spatial type (geometry or geography). If all inputs share the same specific geometry type (e.g., all are Points), the output will be the corresponding Multi-type (e.g., MultiPoint); otherwise, it returns a GeometryCollection.

### Output
Returns `geography | geometry` — A single aggregated geometry or geography object representing the collection of all input rows.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

query = """
SELECT ST_Collect_Agg(geom) AS collected_points
FROM (
  VALUES
    (ST_Point(-122.4194, 37.7749)),
    (ST_Point(-118.2437, 34.0522)),
    (ST_Point(-122.6765, 45.5231))
) AS cities(geom)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Collect_Agg` only as an aggregate function in a `SELECT` statement.
- Ensure all inputs to the function are of the same spatial type (geometry or geography).
- Do not attempt to mix geometry and geography types in the same aggregation.

### Prompt Snippet
```text
Use ST_Collect_Agg(column_name) to aggregate multiple spatial rows into a single collection.
```

### Common Failure Modes
- **Type Mismatch:** Attempting to aggregate a mix of `geometry` and `geography` types in the same function call will result in an error.
- **Empty Input:** If the input set is empty, the function may return `NULL`.

### Fix Code Hint
```python
# Ensure all inputs are cast to the same type before aggregation
sd.sql("SELECT ST_Collect_Agg(ST_GeomFromText(wkt_col)) FROM my_table")
```

## API Test: `ST_ConcaveHull`

### Signature
```python
ST_ConcaveHull(geom: geometry, pct_convex: double) -> geometry
```
_Source: source/docs/reference/sql/st_concavehull.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ConcaveHull
description: Returns a concave hull enclosing the input geometry.
kernels:
  - returns: geometry
    args:
    - name: geom
      type: geometry
    - name: pct_convex
      type: double
---

## Description

Returns a concave hull enclosing the input geometry. The `pct_convex` parameter
controls how "tight" the hull is: 1.0 produces the convex hull, while smaller
values produce tighter, more concave shapes.

## Examples

```sql
SELECT ST_ConcaveHull(
    ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'),
    0.3
);
```


### Goal
Computes a concave hull polygon that encloses the input geometry, allowing for variable tightness based on the `pct_convex` parameter.

### Parameters
- `geom` (`geometry`): The input geometry (e.g., Point, LineString, Polygon) to be enclosed.
- `pct_convex` (`double`): A value between 0.0 and 1.0. A value of 1.0 results in the convex hull; lower values result in a tighter, more concave shape.

### Input
The function accepts standard geometry types (WKT, WKB, or GeoParquet-loaded geometries). Ensure the input geometry is valid before processing.

### Output
Returns a `geometry` object representing the concave hull polygon.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = """
SELECT ST_ConcaveHull(
    ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'),
    0.3
) as hull
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_ConcaveHull(geom, pct_convex)` as a standalone SQL function.
- Ensure `pct_convex` is a double between 0.0 and 1.0.
- If the input geometry is complex, ensure it is properly registered as a view or created via `ST_GeomFromText`.

### Prompt Snippet
```text
ST_ConcaveHull(geom_column, 0.5)
```

### Common Failure Modes
- **Invalid `pct_convex`:** Providing a value outside the expected range may lead to unexpected hull shapes or errors.
- **Invalid Geometry:** Passing an invalid or empty geometry will result in an error or an empty output.
- **CRS Mismatch:** If the input geometry is part of a larger spatial join, ensure all geometries share the same CRS to avoid projection errors.

### Fix Code Hint
```python
# Ensure geometry is valid and pct_convex is within [0, 1]
sd.sql("SELECT ST_ConcaveHull(ST_MakeValid(geom), 0.5) FROM my_table").to_arrow_table()
```

## API Test: `ST_Contains`

### Signature
```python
ST_Contains(arg1: geometry, arg2: geometry) -> boolean | ST_Contains(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_contains.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Contains
description: Returns true if geomA contains geomB.
kernels:
  - returns: boolean
    args: [geometry, geometry]
  - returns: boolean
    args: [geography, geography]
---

## Description

Returns `true` if no points of B lie outside A and at least one point of B lies inside A.

## Examples

```sql
SELECT ST_Contains(
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_Point(0.25, 0.25)
) AS val;
```

```sql
SELECT ST_Contains(
    ST_GeogFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_GeogPoint(0.25, 0.25)
) AS val;
```


### Goal
Determines if the first spatial object (geometry or geography) spatially contains the second spatial object.

### Parameters
- `arg1` (`geometry` or `geography`): The container object.
- `arg2` (`geometry` or `geography`): The object to be tested for containment within `arg1`.

### Input
- **Data Formats:** Accepts WKT, WKB, or geometries/geographies registered as views in SedonaDB.
- **Preconditions:** Both arguments must share the same coordinate reference system (CRS). If inputs have mismatched CRSs, the operation will fail or produce invalid results.

### Output
Returns `boolean` — `true` if no points of `arg2` lie outside `arg1` and at least one point of `arg2` lies inside `arg1`; otherwise `false`.

### Valid Call Patterns
```python
# Geometry containment check
result = sd.sql("SELECT ST_Contains(ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), ST_Point(0.25, 0.25)) as val").to_arrow_table()
print(result)
# __CHECK__ witness: val=True

# Geography containment check
result = sd.sql("SELECT ST_Contains(ST_GeogFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), ST_GeogPoint(0.25, 0.25)) as val").to_arrow_table()
print(result)
# __CHECK__ witness: val=True
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (both `geometry` or both `geography`).
- Use `ST_GeomFromText` for planar geometry operations and `ST_GeogFromText` for spherical geography operations.
- Do not mix geometry and geography types in the same function call.

### Prompt Snippet
```text
ST_Contains(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different coordinate systems will cause errors. Ensure all inputs are reprojected to a common CRS before calling.
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` is not supported.
- **Invalid Geometry:** Passing malformed WKT strings will result in a runtime exception.

### Fix Code Hint
```python
# Ensure CRS alignment if necessary
sd.sql("SELECT ST_Contains(geom_col, ST_Transform(other_geom, 'EPSG:4326')) FROM my_table")
```

## API Test: `ST_ConvexHull`

### Signature
```python
ST_ConvexHull(arg1: geometry) -> geometry | ST_ConvexHull(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_convexhull.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ConvexHull
description: Returns the Convex Hull of a geometry or geography.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

Returns the smallest convex polygon that encloses all points in the input geometry.

## Examples

```sql
SELECT ST_ConvexHull(
  ST_GeomFromText('MULTIPOINT ((0 0), (0 1), (1 0))')
);
```

```sql
SELECT ST_ConvexHull(
  ST_GeogFromWKT('MULTIPOINT ((0 0), (0 1), (1 0))')
);
```


### Goal
Computes the smallest convex polygon that encloses all points within a given geometry or geography object.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object for which the convex hull is to be calculated.

### Input
The function accepts standard geometry or geography types. Inputs are typically provided via WKT (Well-Known Text) or WKB (Well-Known Binary) formats, or as existing geometry columns in a registered SedonaDB view.

### Output
Returns a `geometry` or `geography` object representing the convex hull polygon.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_ConvexHull(ST_GeomFromText('MULTIPOINT ((0 0), (0 1), (1 0))'))").to_arrow_table()
print(result)
# __CHECK__ witness: The result contains the convex hull polygon geometry.
```

### LLM Instruction Prompt
- Always use `ST_ConvexHull` as a standalone function call in SQL queries.
- Ensure the input is a valid geometry or geography type; use `ST_GeomFromText` or `ST_GeogFromWKT` if the input is a string literal.

### Prompt Snippet
```text
ST_ConvexHull(geometry_expression)
```

### Common Failure Modes
- Passing an unsupported data type that cannot be cast to a geometry or geography.
- Providing an empty geometry, which may result in an empty geometry output or an error depending on the underlying engine state.
- Mismatched CRS: If performing operations involving multiple geometries, ensure they share the same CRS to avoid spatial calculation errors.

### Fix Code Hint
```python
# Ensure input is valid geometry
sd.sql("SELECT ST_ConvexHull(ST_GeomFromText('POINT(0 0)'))").to_arrow_table()
```

## API Test: `ST_ConvexHull_Agg`

### Signature
```python
ST_ConvexHull_Agg(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_convexhull_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ConvexHull_Agg
description: An aggregate function that returns the convex hull of all geometries in a group.
kernels:
  - returns: geometry
    args: [geometry]
---

## Description

An aggregate function that computes the smallest convex polygon enclosing all points across every geometry in a group.

## Examples

```sql
SELECT ST_ConvexHull_Agg(geom) FROM (
    VALUES
        (ST_Point(0, 0)),
        (ST_Point(0, 1)),
        (ST_Point(1, 0)),
        (ST_Point(1, 1))
) AS t(geom);
```


### Goal
Computes the smallest convex polygon that encloses all input geometries within a specified group.

### Parameters
- `arg1` (`geometry`): The input geometry column or expression to be aggregated.

### Input
- **Data:** Accepts standard geometry types (Point, LineString, Polygon, etc.).
- **Preconditions:** All geometries in the group must share the same Coordinate Reference System (CRS) if defined; otherwise, the operation may fail or produce undefined results.

### Output
Returns `geometry` — A single polygon representing the convex hull of the input set.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

query = """
SELECT ST_ConvexHull_Agg(geom) FROM (
    VALUES
        (ST_Point(0, 0)),
        (ST_Point(0, 1)),
        (ST_Point(1, 0)),
        (ST_Point(1, 1))
) AS t(geom)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_ConvexHull_Agg` only as an aggregate function within a `SELECT` statement.
- Ensure the input column contains valid geometry types.
- Do not attempt to use this function on raster types directly; raster data must be converted to geometry first if needed.

### Prompt Snippet
```text
ST_ConvexHull_Agg(geometry_column)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with conflicting CRS definitions in the same group.
- **Empty Input:** Calling the function on an empty group or a table with no rows may return `NULL`.
- **Invalid Geometry:** Passing malformed WKT or WKB that cannot be parsed into a valid geometry object.

### Fix Code Hint
```python
# Ensure all geometries are in the same CRS before aggregation
sd.sql("SELECT ST_ConvexHull_Agg(ST_Transform(geom, 'EPSG:4326')) FROM my_table")
```

## API Test: `ST_CoveredBy`

### Signature
```python
ST_CoveredBy(arg1: geometry, arg2: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_coveredby.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_CoveredBy
description: Returns true if geomA is covered by geomB.
kernels:
  - returns: boolean
    args: [geometry, geometry]
---

## Description

Returns `true` if no point in geometry A is outside geometry B. Inverse of
`ST_Covers`: `ST_CoveredBy(A, B)` is equivalent to `ST_Covers(B, A)`.

## Examples

```sql
SELECT ST_CoveredBy(
    ST_Point(0.25, 0.25),
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
);
```


### Goal
Determines if the entirety of a given geometry (arg1) is contained within the spatial extent of another geometry (arg2).

### Parameters
- `arg1` (`geometry`): The geometry to be tested for coverage.
- `arg2` (`geometry`): The geometry acting as the covering boundary.

### Input
Accepts standard SedonaDB geometry types (Point, Polygon, etc.) created via constructors like `ST_Point` or `ST_GeomFromText`. Inputs must share the same Coordinate Reference System (CRS) if defined; mismatched CRSs may result in errors or undefined behavior.

### Output
Returns `boolean` — `true` if every point of `arg1` lies within `arg2`, otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify coverage of a point within a polygon
query = """
SELECT ST_CoveredBy(
    ST_Point(0.25, 0.25),
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
) as is_covered
"""
result = sd.sql(query).to_arrow_table()
assert result["is_covered"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_CoveredBy(geometry, geometry)` as a standalone SQL function call.
- Ensure both arguments are valid geometry objects.
- Remember that `ST_CoveredBy(A, B)` is logically equivalent to `ST_Covers(B, A)`.

### Prompt Snippet
```text
ST_CoveredBy(geom_a, geom_b)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different, incompatible Coordinate Reference Systems.
- **Invalid Geometry:** Passing malformed WKT or null values as arguments.
- **Type Mismatch:** Attempting to pass raster objects directly; ensure rasters are converted to geometry (e.g., via `ST_ConvexHull` or similar) before use.

### Fix Code Hint
```python
# Ensure both geometries are in the same CRS before comparison
sd.sql("SELECT ST_CoveredBy(ST_Transform(geom1, 'EPSG:4326'), ST_Transform(geom2, 'EPSG:4326')) FROM table")
```

## API Test: `ST_Covers`

### Signature
```python
ST_Covers(arg1: geometry, arg2: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_covers.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Covers
description: Returns true if geomA covers geomB.
kernels:
  - returns: boolean
    args: [geometry, geometry]
---

## Description

Returns `true` if no point in geometry B is outside geometry A. Similar to
`ST_Contains` but does not require interior intersection.

## Examples

```sql
SELECT ST_Covers(
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_Point(0.25, 0.25)
) AS val;
```


### Goal
Determines if a geometry fully contains another geometry, returning true if no point in the second geometry lies outside the first.

### Parameters
- `arg1` (`geometry`): The covering geometry (e.g., Polygon, MultiPolygon).
- `arg2` (`geometry`): The geometry to be covered (e.g., Point, LineString, Polygon).

### Input
Requires two valid geometry objects. Inputs can be provided as WKT, WKB, or geometry columns registered in a SedonaDB view. Ensure both geometries share the same Coordinate Reference System (CRS) to avoid spatial mismatch errors.

### Output
Returns `boolean` — `true` if `arg1` covers `arg2`, otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify coverage of a point by a polygon
query = """
SELECT ST_Covers(
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_Point(0.25, 0.25)
) AS val
"""
result = sd.sql(query).to_arrow_table()
assert result["val"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_Covers(geomA, geomB)` to check if `geomA` covers `geomB`.
- Ensure both arguments are valid geometry types.
- Do not use this function for raster-to-geometry comparisons; use `RS_Intersects` or `RS_Contains` for raster-based spatial predicates.

### Prompt Snippet
```text
ST_Covers(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** If `arg1` and `arg2` have different CRSs, the operation may fail or produce incorrect results; ensure both are reprojected to a common CRS before comparison.
- **Invalid Geometry:** Passing malformed WKT or empty geometries will result in a runtime error.
- **Type Mismatch:** Passing a raster object instead of a geometry object will cause a type error; cast or convert raster data to geometry if necessary.

### Fix Code Hint
```python
# Ensure CRS alignment before calling ST_Covers
query = """
SELECT ST_Covers(
    ST_Transform(geom_col_a, 'EPSG:4326'),
    ST_Transform(geom_col_b, 'EPSG:4326')
) FROM my_table
"""
```

## API Test: `ST_Crosses`

### Signature
```python
ST_Crosses(arg1: geometry, arg2: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_crosses.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Crosses
description: Returns true if A crosses B.
kernels:
  - returns: boolean
    args: [geometry, geometry]
---

## Description

Returns `true` if the two geometries have some (but not all) interior points in
common and the dimension of the intersection is less than that of either input.

## Examples

```sql
SELECT ST_Crosses(
    ST_GeomFromWKT('LINESTRING(0 0, 2 2)'),
    ST_GeomFromWKT('LINESTRING(0 2, 2 0)')
);
```


### Goal
Determines if two geometries spatially cross each other by evaluating if their interior intersection has a lower dimension than the input geometries.

### Parameters
- `arg1` (`geometry`): The first geometry object to evaluate.
- `arg2` (`geometry`): The second geometry object to evaluate.

### Input
- **Data Formats:** Accepts standard geometry types (e.g., LineString, Polygon) created via constructors like `ST_GeomFromWKT` or loaded from GeoParquet.
- **Preconditions:** Both geometries must be in the same Coordinate Reference System (CRS). If CRSs are mismatched, the operation may fail or produce incorrect results.

### Output
Returns `boolean` — `true` if the geometries cross, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify the crossing condition
query = """
SELECT ST_Crosses(
    ST_GeomFromWKT('LINESTRING(0 0, 2 2)'),
    ST_GeomFromWKT('LINESTRING(0 2, 2 0)')
) as result
"""
result = sd.sql(query).to_arrow_table()
assert result["result"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_Crosses(geom1, geom2)` to check for spatial crossing.
- Ensure both inputs are valid geometry objects.
- Always verify that inputs share the same CRS before calling.

### Prompt Snippet
```text
ST_Crosses(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different coordinate systems will result in errors or invalid spatial logic.
- **Invalid Geometry:** Passing malformed WKT or null geometry objects will cause the query to fail.
- **Dimension Constraints:** `ST_Crosses` is typically used for LineString/LineString or LineString/Polygon intersections; it will return `false` if the intersection dimension does not meet the criteria (e.g., if the geometries are identical or disjoint).

### Fix Code Hint
```python
# Ensure geometries are in the same CRS before comparison
query = """
SELECT ST_Crosses(
    ST_Transform(geom1, 'EPSG:4326'),
    ST_Transform(geom2, 'EPSG:4326')
) FROM my_table
"""
```

## API Test: `ST_DWithin`

### Signature
```python
ST_DWithin(arg1: geometry, arg2: geometry, distance: double) -> boolean | ST_DWithin(arg1: geography, arg2: geography, distance: double) -> boolean
```
_Source: source/docs/reference/sql/st_dwithin.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_DWithin
description: Returns true if two geometries or geographies are within a specified distance of each other.
kernels:
  - returns: boolean
    args:
    - geometry
    - geometry
    - name: distance
      type: double
  - returns: boolean
    args:
    - geography
    - geography
    - name: distance
      type: double
---

## Description

For geography, the distance is specified in meters and represents the spherical approximation of the geodesic distance between the two geographies.

## Examples

```sql
SELECT ST_DWithin(
    ST_Point(0, 0),
    ST_Point(1, 1),
    2
);
```

```sql
SELECT ST_DWithin(
    ST_GeogPoint(0, 0),
    ST_GeogPoint(1, 1),
    200000
);
```


### Goal
Determines if two spatial objects (geometries or geographies) are within a specified distance of each other.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to compare.
- `arg2` (`geometry` or `geography`): The second spatial object to compare.
- `distance` (`double`): The threshold distance. For `geography` types, this value is interpreted in meters.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** Both inputs must be of the same type (both `geometry` or both `geography`). For `geometry` types, ensure the coordinate reference system (CRS) is consistent between inputs, as SedonaDB requires matching CRSs for spatial operations.

### Output
Returns `boolean` — `true` if the distance between the two objects is less than or equal to the specified `distance`, otherwise `false`.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_DWithin(ST_Point(0, 0), ST_Point(1, 1), 2.0)").to_arrow_table()
print(result)
# __CHECK__ witness: result contains True

# Example using geography
result = sd.sql("SELECT ST_DWithin(ST_GeogPoint(0, 0), ST_GeogPoint(1, 1), 200000.0)").to_arrow_table()
print(result)
# __CHECK__ witness: result contains True
```

### LLM Instruction Prompt
- Always ensure the distance parameter is a `double`.
- When using `geography` types, ensure the distance is provided in meters.
- Verify that both input arguments share the same spatial type (geometry vs geography).

### Prompt Snippet
```text
ST_DWithin(arg1, arg2, distance)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other will result in an execution error.
- **CRS Mismatch:** Providing geometries with different or undefined CRSs may cause the operation to fail or return unexpected results.
- **Unit Confusion:** Providing a distance in degrees for `geography` types (which expect meters) will lead to incorrect logical results.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_DWithin(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(1 1)'), 2.0)")
```

## API Test: `ST_DelaunayTriangles`

### Signature
```python
ST_DelaunayTriangles(arg1: geometry) -> geometry | ST_DelaunayTriangles(arg1: geometry, tolerance: float64) -> geometry | ST_DelaunayTriangles(arg1: geometry, tolerance: float64, flags: integer) -> geometry
```
_Source: source/docs/reference/sql/st_delaunaytriangles.qmd:19_

### Goal
Computes a Delaunay triangulation of the vertices of an input geometry to generate a set of non-overlapping triangles that cover the input's convex hull.

### Parameters
- `arg1` (`geometry`): The input geometry whose vertices will be used to construct the triangulation.
- `tolerance` (`float64`): Snap-rounding tolerance for vertices; use 0.0 for exact computation.
- `flags` (`integer`): Output format control: use 0 for a `GEOMETRYCOLLECTION` of `POLYGON` triangles, or 1 for `MULTILINESTRING` edge-only output.

### Input
- **Data:** Accepts standard geometry types (e.g., `MULTIPOINT`, `POLYGON`).
- **Formats:** Input geometries can be constructed via WKT, WKB, or loaded from GeoParquet.
- **Preconditions:** The input must be a valid geometry. If using `tolerance`, ensure the value is non-negative.

### Output
Returns `geometry` — A `GEOMETRYCOLLECTION` of `POLYGON` triangles by default, or a `MULTILINESTRING` if `flags` is set to 1.

### Valid Call Patterns
```python
# Basic triangulation
result = sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'))").to_arrow_table()
print("__CHECK__")

# Triangulation with tolerance
result = sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'), 0.0)").to_arrow_table()
print("__CHECK__")

# Triangulation with tolerance and flags
result = sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'), 0.0, 1)").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_DelaunayTriangles` as a standalone SQL function within `sd.sql()`.
- If specifying `flags`, you must also provide a `tolerance` value (e.g., 0.0).
- Ensure the input geometry is valid; use `ST_GeomFromWKT` for literal inputs.

### Prompt Snippet
```text
ST_DelaunayTriangles(geometry_expression, tolerance, flags)
```

### Common Failure Modes
- **Missing Arguments:** Providing `flags` without a `tolerance` value will result in a signature mismatch error.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT strings will cause the query to fail.
- **Negative Tolerance:** Providing a negative value for `tolerance` may lead to unexpected behavior or errors during the snap-rounding phase.

### Fix Code Hint
```python
# Ensure all required positional arguments are provided in order
# Correct: ST_DelaunayTriangles(geom, 0.0, 1)
# Incorrect: ST_DelaunayTriangles(geom, 1) 
```

## API Test: `ST_Difference`

### Signature
```python
ST_Difference(arg1: geometry, arg2: geometry) -> geometry | ST_Difference(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_difference.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Difference
description: Computes the difference between geomA and geomB.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Description

Returns the part of geometry A that does not intersect with geometry B.

## Examples

```sql
SELECT ST_Difference(
    ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'),
    ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
) AS val;
```

```sql
SELECT ST_Difference(
    ST_GeogFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'),
    ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
) AS val;
```


### Goal
Computes the spatial set difference, returning the portion of the first geometry that does not overlap with the second geometry.

### Parameters
- `arg1` (`geometry`): The primary geometry or geography from which the second geometry will be subtracted.
- `arg2` (`geometry`): The geometry or geography to be subtracted from the first.

### Input
- **Data Formats:** Accepts standard WKT, WKB, or geometries/geographies registered in the SedonaDB session.
- **Preconditions:** Both inputs must share the same coordinate reference system (CRS). If inputs have mismatched CRSs, the operation will fail.

### Output
Returns `geography | geometry` — A new geometry or geography object representing the area of `arg1` not covered by `arg2`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("""
    SELECT ST_Difference(
        ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'),
        ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
    ) AS val
""").to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (both `geometry` or both `geography`).
- Do not mix `geometry` and `geography` types in a single `ST_Difference` call.
- Ensure both inputs are in the same CRS before calling the function.

### Prompt Snippet
```text
ST_Difference(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** Attempting to compute the difference between geometries with different coordinate systems will result in an error.
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` is not supported.
- **Invalid Geometry:** Providing malformed WKT strings will cause the underlying parser to fail.

### Fix Code Hint
```python
# Ensure CRS alignment if necessary
sd.sql("SELECT ST_Difference(geom1, ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `ST_Dimension`

### Signature
```python
ST_Dimension(arg1: geometry) -> integer | ST_Dimension(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_dimension.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Dimension
description: Returns the dimension of the geometry or geography.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

## Description

Returns the inherent dimension of the geometry:

- 0 for points or multipoints
- 1 for linestring or multilinestrings
- 2 for polygons or multipolygons

For geometry collections, returns the largest dimension among the components.

## Examples

```sql
SELECT ST_Dimension(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'));
```

```sql
SELECT ST_Dimension(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'));
```


### Goal
Determines the topological dimension of a given geometry or geography object to classify it as a point (0), line (1), or surface (2).

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to evaluate.

### Input
The function accepts standard geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. No specific CRS preconditions are required for dimension calculation.

### Output
Returns `integer` — The inherent dimension: 0 for points/multipoints, 1 for linestrings/multilinestrings, and 2 for polygons/multipolygons. For geometry collections, it returns the maximum dimension found among its constituent parts.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify dimension of a polygon
result = sd.sql("SELECT ST_Dimension(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))')) as dim").to_arrow_table()
assert result["dim"][0] == 2
print("__CHECK__")
```

### LLM Instruction Prompt
When calling `ST_Dimension`, ensure the input is a valid geometry or geography object. If the input is a complex collection, remember that the result is determined by the highest-dimensional component.

### Prompt Snippet
```text
ST_Dimension(geometry_expression)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) directly without using a constructor like `ST_GeomFromWKT`.
- Expecting a specific dimension for an empty geometry (behavior is undefined/implementation-dependent).

### Fix Code Hint
```python
# Ensure input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_Dimension(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_Disjoint`

### Signature
```python
ST_Disjoint(arg1: geometry, arg2: geometry) -> boolean | ST_Disjoint(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_disjoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Disjoint
description: Returns true if geomA is disjoint from geomB.
kernels:
  - returns: boolean
    args: [geometry, geometry]
  - returns: boolean
    args: [geography, geography]
---

### Goal
Determines if two spatial objects (geometries or geographies) share no common points, serving as the logical inverse of `ST_Intersects`.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to evaluate.
- `arg2` (`geometry` or `geography`): The second spatial object to evaluate.

### Input
- **Data Formats:** Accepts standard WKT, WKB, or geometry/geography types registered in the SedonaDB session.
- **Preconditions:** Both arguments must be of the same type (both `geometry` or both `geography`). If using `geometry` types, they must share the same Coordinate Reference System (CRS) or be explicitly reprojected to match before comparison.

### Output
Returns `boolean`: `true` if the two inputs do not share any points, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify disjoint geometries
result = sd.sql("""
    SELECT ST_Disjoint(
        ST_GeomFromText('POLYGON ((0 0, 1 0, 0 1, 0 0))'),
        ST_GeomFromText('POLYGON ((2 2, 3 2, 2 3, 2 2))')
    ) as is_disjoint
""").to_arrow_table()

assert result["is_disjoint"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (geometry vs geography).
- If using geometries, verify that both inputs are in the same CRS.
- Use `ST_Disjoint` when you need to confirm that two spatial features have no intersection.

### Prompt Snippet
```text
ST_Disjoint(geom1, geom2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other will result in a runtime error.
- **CRS Mismatch:** Comparing geometries with different CRSs without prior reprojection will cause the operation to fail or return incorrect results.
- **Invalid Geometry:** Providing malformed WKT strings to the input arguments will cause the query to fail.

### Fix Code Hint
```python
# Ensure CRS alignment before calling ST_Disjoint
sd.sql("""
    SELECT ST_Disjoint(
        geom_col, 
        ST_Transform(other_geom_col, 'EPSG:4326')
    ) FROM my_table
""")
```

## API Test: `ST_Distance`

### Signature
```python
ST_Distance(arg1: geometry, arg2: geometry) -> double | ST_Distance(arg1: geography, arg2: geography) -> double
```
_Source: source/docs/reference/sql/st_distance.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Distance
description: Returns the distance between two geometries or geographies.
kernels:
  - returns: double
    args: [geometry, geometry]
  - returns: double
    args: [geography, geography]
---

## Description

For geometry, returns the 2D Cartesian (planar) distance between two geometries, in the units of the coordinate reference system. For geography, returns the spherical approximation of the geodesic distance between two geographies in meters.

## Examples

```sql
SELECT ST_Distance(ST_Point(0, 0), ST_Point(1, 1));
```

```sql
SELECT ST_Distance(ST_GeogPoint(0, 0), ST_GeogPoint(1, 1));
```


### Goal
Calculates the spatial separation between two vector objects, providing either planar distance for geometries or geodesic distance for geographies.

### Parameters
- `arg1` (`geometry`): The first geometry or geography object.
- `arg2` (`geometry`): The second geometry or geography object.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** Both inputs must share the same Coordinate Reference System (CRS). If CRSs are mismatched, the operation may fail or produce invalid results; explicit reprojection is required if inputs originate from different coordinate systems.

### Output
Returns `double` — The distance value. For `geometry`, the unit is determined by the CRS; for `geography`, the unit is always in meters.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_Distance(ST_Point(0, 0), ST_Point(1, 1)) as dist").to_arrow_table()
print(result)
# __CHECK__ witness: dist is 1.4142135623730951

# Example using geography
result_geog = sd.sql("SELECT ST_Distance(ST_GeogPoint(0, 0), ST_GeogPoint(1, 1)) as dist").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Distance(arg1, arg2)` as a standalone SQL function call within `sd.sql()`.
- Ensure both arguments are of the same type (both `geometry` or both `geography`).
- Do not mix geometry and geography types in a single call.

### Prompt Snippet
```text
Use ST_Distance(geom1, geom2) for planar distance or ST_Distance(geog1, geog2) for geodesic distance. Ensure both inputs share the same CRS.
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different CRSs will result in an error or incorrect calculation.
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other is not supported.
- **Null Inputs:** Passing `NULL` as an argument will result in a `NULL` return value.

### Fix Code Hint
```python
# Ensure CRS alignment before distance calculation
sd.sql("SELECT ST_Distance(geom1, ST_Transform(geom2, 'EPSG:4326')) FROM my_table")
```

## API Test: `ST_Dump`

### Signature
```python
ST_Dump(arg1: geometry) -> struct | ST_Dump(arg1: geography) -> struct
```
_Source: source/docs/reference/sql/st_dump.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Dump
description: Expands multi-part geometries into child parts
kernels:
  - returns: struct
    args: [geometry]
  - returns: struct
    args: [geography]
---

## Description

If the geometry is simple it returns the geometry itself, if the geometry is collection or multi it returns record for each of collection components.

## Examples

```sql
SELECT ST_Dump(ST_GeomFromText('MULTIPOINT ((10 40), (40 30), (20 20), (30 10))'));
```


### Goal
`ST_Dump` decomposes complex multi-part geometries or geometry collections into individual, simple component geometries.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be expanded.

### Input
- **Data:** Accepts standard geometry types (e.g., `MULTIPOINT`, `MULTIPOLYGON`, `GEOMETRYCOLLECTION`).
- **Formats:** Supports geometries derived from WKT, WKB, or GeoParquet.
- **Preconditions:** The input must be a valid geometry or geography type recognized by SedonaDB.

### Output
Returns `struct` — A set of records where each record contains a single component geometry derived from the original input.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example inferred from SQL signature
query = "SELECT ST_Dump(ST_GeomFromText('MULTIPOINT ((10 40), (40 30))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result should contain individual point geometries.
```

### LLM Instruction Prompt
- Always use `ST_Dump` when you need to process individual parts of a multi-part geometry independently.
- Ensure the input is a valid geometry type; if the input is a simple geometry, the function returns the geometry itself.

### Prompt Snippet
```text
Use ST_Dump(geometry_column) to expand multi-part geometries into individual rows for further spatial analysis.
```

### Common Failure Modes
- **Invalid Geometry:** Passing a non-geometry type or malformed WKT string will result in a query execution error.
- **Type Mismatch:** Attempting to pass raster types directly into `ST_Dump` will fail, as it is strictly for vector geometry/geography types.

### Fix Code Hint
```python
# Ensure input is cast to geometry if coming from an unknown source
sd.sql("SELECT ST_Dump(ST_GeomFromText(my_wkt_column)) FROM my_table")
```

## API Test: `ST_EndPoint`

### Signature
```python
ST_EndPoint(arg1: geometry) -> geometry | ST_EndPoint(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_endpoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_EndPoint
description: Returns last point of a linestring.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Examples

```sql
SELECT ST_EndPoint(ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'));
```


### Goal
`ST_EndPoint` extracts the final coordinate point from a provided `LINESTRING` geometry or geography object.

### Parameters
- `arg1` (`geometry`): A `LINESTRING` geometry or geography object from which the last vertex is to be extracted.

### Input
The input must be a valid `LINESTRING` type. If the input is a different geometry type (e.g., `POINT` or `POLYGON`), the function behavior is undefined or may result in an error. Ensure the input is properly registered as a view or passed as a WKT/WKB literal.

### Output
Returns a `geometry` or `geography` object representing the last point of the input linestring.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = "SELECT ST_EndPoint(ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)')) as end_point"
result = sd.sql(query).to_arrow_table()

# Verification
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_EndPoint` with a `LINESTRING` input.
- When writing SQL queries, ensure the input is wrapped in a constructor like `ST_GeomFromText` if the data is provided as a string literal.
- The function returns a spatial object; if you need the coordinates as numeric values, chain this with `ST_X` and `ST_Y`.

### Prompt Snippet
```text
ST_EndPoint(ST_GeomFromText('LINESTRING(...)'))
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-linestring geometry (e.g., a `POINT`) will result in an execution error.
- **Null Input:** Passing a `NULL` geometry will return `NULL`.
- **CRS Mismatch:** While `ST_EndPoint` is a scalar operation, ensure that if you are performing subsequent spatial joins with the result, the CRS of the resulting point matches the target dataset.

### Fix Code Hint
```python
# Ensure the input is a LINESTRING
query = "SELECT ST_EndPoint(geom) FROM my_table WHERE ST_GeometryType(geom) = 'ST_LineString'"
```

## API Test: `ST_Envelope`

### Signature
```python
ST_Envelope(arg1: geometry) -> geometry | ST_Envelope(arg1: geography) -> geometry
```
_Source: source/docs/reference/sql/st_envelope.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Envelope
description: Returns the bounding box (envelope) of a geometry or geography as a new geometry.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args: [geography]
---

## Description

Returns the minimum axis-aligned bounding box of a geometry. This is usually a polygon but can be a linestring for perfectly vertical or horizontal input and is a point if the input is also a point. For geography, the return value is still a rectangular polygon geometry that represents the latitude and longitude range of the input. For geographies that span the antimeridian, a multipolygon is returned (one on each side of the antimeridian).

Because of slight numerical inaccuracy of geographical calculations, the bounds are expanded slightly compared to the corresponding geometry envelope.

## Examples

```sql
SELECT ST_Envelope(ST_Point(1.0, 2.0));
```

```sql
SELECT ST_Envelope(ST_GeomFromWKT('LINESTRING(0 0, 3 4)'));
```

```sql
SELECT ST_Envelope(ST_GeogFromWKT('LINESTRING(0 0, 3 4)'));
```

```sql
SELECT ST_Envelope(ST_GeomFromWKT('LINESTRING(170 3, -170 4)'));
```

```sql
SELECT ST_Envelope(ST_GeogPoint(1.0, 2.0));
```

```sql
SELECT ST_Envelope(ST_GeogFromWKT('LINESTRING(0 0, 3 4)'));
```

```sql
SELECT ST_Envelope(ST_GeogFromWKT('LINESTRING(170 3, -170 4)'));
```


### Goal
Computes the minimum axis-aligned bounding box for a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the bounding box is calculated.

### Input
Accepts standard geometry or geography types. Input data can be provided via WKT, WKB, or existing spatial columns in a registered view. No specific preconditions are required, though geography inputs will account for antimeridian crossing.

### Output
Returns a `geometry` object representing the bounding box. For standard inputs, this is a polygon; for degenerate cases (points or perfectly vertical/horizontal lines), it may return a point or linestring. For geography spanning the antimeridian, it returns a multipolygon.

### Valid Call Patterns
```python
# Inferred from signature and SQL examples
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT ST_Envelope(ST_Point(1.0, 2.0))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Envelope` to calculate the spatial extent of a geometry or geography.
- Note that geography inputs may result in a `MULTIPOLYGON` if the input crosses the antimeridian.
- Remember that geography envelopes are slightly expanded compared to geometry envelopes due to numerical inaccuracies.

### Prompt Snippet
```text
ST_Envelope(geometry_column)
```

### Common Failure Modes
- Passing non-spatial types: The function expects valid geometry or geography types; passing strings or numbers directly without a constructor (e.g., `ST_Point`) will fail.
- Mismatched CRS: While `ST_Envelope` operates on the object, ensure the input geometry is in the expected coordinate system before processing.

### Fix Code Hint
```python
# Ensure input is cast to a geometry type if coming from raw WKT
sd.sql("SELECT ST_Envelope(ST_GeomFromWKT('POINT(1 1)'))").to_arrow_table()
```

## API Test: `ST_Envelope_Agg`

### Signature
```python
ST_Envelope_Agg(arg1: geometry) -> geometry | ST_Envelope_Agg(arg1: geography) -> geometry
```
_Source: source/docs/reference/sql/st_envelope_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Envelope_Agg
description: An aggregate function that returns the collective bounding box (envelope) of a set of geometries.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args: [geography]
---

## Description

An aggregate function that computes the collective bounding box (envelope) of all geometries in a group. Geography input is also accepted; the envelope is always returned as a geometry. For geographies that cross the antimeridian, a multi geometry is returned with one polygon each for the west and east component.

## Examples

```sql
SELECT ST_Envelope_Agg(geom) FROM (
    VALUES
        (ST_Point(0, 0)),
        (ST_Point(5, 5)),
        (ST_Point(10, 2))
) AS t(geom);
```


### Goal
Computes the collective bounding box (envelope) of a set of geometries or geographies within a group.

### Parameters
- `arg1` (`geometry`): The column containing geometry or geography objects to be aggregated.

### Input
- **Data:** A column of geometry or geography types.
- **Preconditions:** The input must be part of a grouped SQL query (e.g., using `GROUP BY`) or a global aggregation.

### Output
Returns `geometry` — A single polygon representing the minimum bounding box that encompasses all input geometries. For geographies crossing the antimeridian, it returns a multi-geometry.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and SQL example
query = "SELECT ST_Envelope_Agg(geom) FROM my_table"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Envelope_Agg` as an aggregate function in a `SELECT` statement.
- Ensure the input column contains valid geometry or geography types.
- Note that the output is always a `geometry` type, regardless of whether the input was `geography`.

### Prompt Snippet
```text
ST_Envelope_Agg(column_name)
```

### Common Failure Modes
- **Non-aggregated context:** Attempting to use `ST_Envelope_Agg` without a `GROUP BY` clause when other non-aggregated columns are present in the `SELECT` list.
- **Type Mismatch:** Passing non-spatial data types to the function.

### Fix Code Hint
```python
# Ensure the query aggregates correctly
sd.sql("SELECT ST_Envelope_Agg(geom) FROM table_name GROUP BY category_id").to_arrow_table()
```

## API Test: `ST_Equals`

### Signature
```python
ST_Equals(arg1: geometry, arg2: geometry) -> boolean | ST_Equals(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_equals.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Equals
description: Returns true if geomA equals geomB.
kernels:
  - returns: boolean
    args: [geometry, geometry]
  - returns: boolean
    args: [geography, geography]
---

## Description

Returns `true` if the two geometries are topologically equal (i.e., they represent the same point set, regardless of vertex order).

## Examples

```sql
SELECT ST_Equals(
    ST_GeomFromText('LINESTRING(0 0, 1 1)'),
    ST_GeomFromText('LINESTRING(1 1, 0 0)')
);
```

```sql
SELECT ST_Equals(
    ST_GeogFromText('LINESTRING(0 0, 1 1)'),
    ST_GeogFromText('LINESTRING(1 1, 0 0)')
);
```


### Goal
Determines if two spatial objects (geometry or geography) are topologically equivalent by verifying they represent the same point set.

### Parameters
- `arg1` (`geometry`): The first geometry or geography object to compare.
- `arg2` (`geometry`): The second geometry or geography object to compare.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Formats:** Inputs are typically generated via constructors like `ST_GeomFromText` or `ST_GeogFromText`.
- **Preconditions:** Both arguments must be of the same type (both `geometry` or both `geography`).

### Output
Returns `boolean` — `true` if the two inputs represent the same point set, `false` otherwise.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT ST_Equals(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(0 0)'))").to_arrow_table()
print(result)
# __CHECK__ witness: True
```

### LLM Instruction Prompt
- Use `ST_Equals` to check for topological equality between two spatial objects.
- Ensure both arguments are of the same spatial type (geometry or geography).
- Do not use this function for coordinate-based equality; it ignores vertex order.

### Prompt Snippet
```text
ST_Equals(geom1, geom2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other will result in a SQL execution error.
- **Null Inputs:** If either input is `NULL`, the result of the function will be `NULL`.
- **CRS Mismatch:** While `ST_Equals` checks point sets, ensure inputs are in the same coordinate reference system if they are expected to be compared spatially.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_Equals(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(0 0)'))")
```

## API Test: `ST_ExteriorRing`

### Signature
```python
ST_ExteriorRing(arg1: geometry) -> geometry | ST_ExteriorRing(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_exteriorring.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ExteriorRing
description: Returns the exterior ring of a polygon as a linestring geometry.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

Returns the exterior ring (outer boundary) of a polygon as a `LINESTRING`. Returns NULL for
non-polygon geometry types.

## Examples

```sql
SELECT ST_ExteriorRing(ST_GeomFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))'));
```

```sql
SELECT ST_ExteriorRing(ST_GeogFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))'));
```


### Goal
Extracts the outer boundary of a polygon as a `LINESTRING` geometry or geography object, effectively stripping away any interior rings (holes).

### Parameters
- `arg1` (`geometry`): A polygon geometry or geography object from which the exterior ring is to be extracted.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** The input must be a polygon. If the input is not a polygon (e.g., a `Point` or `LineString`), the function returns `NULL`.

### Output
Returns `geography | geometry` — A `LINESTRING` representing the outer boundary of the input polygon.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = "SELECT ST_ExteriorRing(ST_GeomFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))')) as exterior"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_ExteriorRing` with a valid polygon input.
- If the input geometry type is unknown, ensure it is cast or verified as a polygon to avoid `NULL` results.
- Use the SQL interface via `sd.sql(query).to_arrow_table()` for execution.

### Prompt Snippet
```text
ST_ExteriorRing(ST_GeomFromWKT('POLYGON ((...))'))
```

### Common Failure Modes
- **Non-Polygon Input:** Passing a `Point`, `LineString`, or `MultiPoint` will result in a `NULL` return value rather than an error.
- **Invalid WKT:** Providing malformed WKT strings to the geometry constructor will cause the query to fail.

### Fix Code Hint
```python
# Ensure input is a valid polygon before calling ST_ExteriorRing
query = "SELECT ST_ExteriorRing(geom) FROM my_table WHERE ST_GeometryType(geom) = 'ST_Polygon'"
```

## API Test: `ST_FlipCoordinates`

### Signature
```python
ST_FlipCoordinates(arg1: geometry) -> geometry | ST_FlipCoordinates(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_flipcoordinates.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_FlipCoordinates
description: Returns a new geometry with the X and Y coordinates of each vertex swapped.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Examples

```sql
SELECT ST_FlipCoordinates(ST_Point(1, 2));
```


### Goal
`ST_FlipCoordinates` transforms a geometry or geography object by swapping the X and Y coordinate values for every vertex in the input.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object whose vertices are to be flipped.

### Input
The function accepts standard geometry or geography types. Input data should be valid spatial objects (e.g., Points, LineStrings, Polygons) created via SedonaDB constructors like `ST_Point` or loaded from supported formats like GeoParquet or WKT.

### Output
Returns a new `geometry` or `geography` object where the X and Y coordinates of each vertex have been swapped.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example inferred from signature and SQL documentation
result = sd.sql("SELECT ST_FlipCoordinates(ST_Point(1, 2))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_FlipCoordinates(geometry_expression)` to swap X and Y coordinates.
- Ensure the input is a valid geometry or geography type.
- The function returns a new object; it does not modify the input in place.

### Prompt Snippet
```text
ST_FlipCoordinates(arg1)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) as `arg1` will result in a SQL execution error.
- Attempting to flip coordinates on a null geometry will return null.
- Mismatched CRS handling: Ensure the input geometry is in a valid state; while coordinate flipping is a structural operation, ensure the resulting geometry remains valid within its defined CRS.

### Fix Code Hint
```python
# Ensure the input is a valid geometry before flipping
sd.sql("SELECT ST_FlipCoordinates(ST_GeomFromText('POINT(1 2)'))").to_arrow_table()
```

## API Test: `ST_Force2D`

### Signature
```python
ST_Force2D(arg1: geometry) -> geometry | ST_Force2D(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_force2d.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Force2D
description: Forces a geometry or geography into a XY coordinate model.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

### Goal
`ST_Force2D` converts a 3D or 4D geometry or geography object into a 2D representation by discarding any existing Z (elevation) or M (measure) coordinate values.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object containing Z or M dimensions that need to be removed.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** The input must be a valid geometry or geography object. If the input is already 2D, the function returns the geometry unchanged.

### Output
Returns `geography | geometry` — A 2D version of the input object containing only X and Y coordinates.

### Valid Call Patterns
```python
# Inferred from signature and documentation examples
import sedona.db
sd = sedona.db.connect()

# Example for Geometry
result = sd.sql("SELECT ST_Force2D(ST_GeomFromWKT('POINT Z (1 2 3)'))").to_arrow_table()
print("__CHECK__", result)

# Example for Geography
result_geog = sd.sql("SELECT ST_Force2D(ST_GeogFromWKT('POINT Z (1 2 3)'))").to_arrow_table()
print("__CHECK__", result_geog)
```

### LLM Instruction Prompt
- Always use `ST_Force2D` when you need to normalize 3D/4D spatial data to 2D for operations that do not support Z or M coordinates.
- Ensure the input is a valid geometry or geography type; do not pass raw strings without wrapping them in a constructor like `ST_GeomFromWKT`.

### Prompt Snippet
```text
Use ST_Force2D(arg1) to strip Z and M dimensions from a geometry or geography object.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., integer or string) directly into the function without a constructor.
- **Null Inputs:** Passing a `NULL` geometry will result in a `NULL` output.

### Fix Code Hint
```python
# Ensure input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_Force2D(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_Force3D`

### Signature
```python
ST_Force3D(arg1: geometry) -> geometry | ST_Force3D(arg1: geometry, z: double) -> geometry | ST_Force3D(arg1: geography) -> geography | ST_Force3D(arg1: geography, z: double) -> geography
```
_Source: source/docs/reference/sql/st_force3d.qmd:19_

### Goal
Forces a geometry or geography object into a 3D (XYZ) coordinate model, optionally assigning a specific Z value if one is not already present.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be converted to 3D.
- `z` (`double`): The optional Z-coordinate value to assign if the input does not already contain Z-dimension data. Defaults to 0.0 if omitted.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** The input must be a valid geometry or geography object. If the input already contains Z-coordinates, those values are preserved and the `z` argument is ignored.

### Output
Returns a `geometry` or `geography` object (matching the input type) that is guaranteed to have a Z-dimension.

### Valid Call Patterns
```python
# Inferred from SQL examples
import sedona.db
sd = sedona.db.connect()

# Example 1: Force 3D with explicit Z value
result = sd.sql("SELECT ST_Force3D(ST_GeomFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()
print(result)
# __CHECK__

# Example 2: Force 3D with default Z value (0.0)
result = sd.sql("SELECT ST_Force3D(ST_GeomFromWKT('POINT (1 2)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Force3D` when a 3D coordinate model is required for spatial operations.
- If the input is a `geography` type, ensure the function is called with a `geography` input to maintain type consistency.
- Remember that `ST_Force3D` is aliased as `ST_Force3DZ`.

### Prompt Snippet
```text
ST_Force3D(geometry_column, z_value)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., string or integer) instead of a geometry or geography object will result in a SQL execution error.
- **Null Inputs:** Passing a `NULL` geometry/geography will return `NULL`.
- **Z-Preservation:** Users may mistakenly assume the `z` argument will overwrite existing Z-values; however, the function preserves existing Z-values if they are already present in the input.

### Fix Code Hint
```python
# Ensure input is cast to geometry if necessary
sd.sql("SELECT ST_Force3D(ST_GeomFromWKT(my_wkt_col), 5.0) FROM my_table")
```

## API Test: `ST_Force3DM`

### Signature
```python
ST_Force3DM(arg1: geometry) -> geometry | ST_Force3DM(arg1: geometry, m: double) -> geometry | ST_Force3DM(arg1: geography) -> geography | ST_Force3DM(arg1: geography, m: double) -> geography
```
_Source: source/docs/reference/sql/st_force3dm.qmd:19_

### Goal
Forces a geometry or geography object into an XYM coordinate model, ensuring the presence of an M (measure) dimension.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be converted to an XYM coordinate model.
- `m` (`double`): An optional measure value to assign to the geometry if it does not already contain M values. Defaults to 0.0 if not provided.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** The input must be a valid geometry or geography object. If the input already contains M values, they are preserved; otherwise, the provided `m` value (or default 0) is applied.

### Output
Returns `geography | geometry` — A new geometry or geography object in the XYM coordinate model.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_Force3DM(ST_GeomFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()
print("__CHECK__ witness:", result)

# Example using geography
result = sd.sql("SELECT ST_Force3DM(ST_GeogFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()
print("__CHECK__ witness:", result)
```

### LLM Instruction Prompt
- Always use `ST_Force3DM` within an `sd.sql()` query.
- Ensure the input is a valid geometry or geography object (e.g., created via `ST_GeomFromWKT` or `ST_GeogFromWKT`).
- If an M value is required, provide it as a `double` as the second argument.

### Prompt Snippet
```text
ST_Force3DM(geometry_column, 10.0)
```

### Common Failure Modes
- Passing an incompatible data type that is not a geometry or geography.
- Providing an `m` argument that cannot be cast to a `double`.
- Attempting to use the function on null inputs, which will return null.

### Fix Code Hint
```python
# Ensure input is cast to geometry if necessary
sd.sql("SELECT ST_Force3DM(ST_GeomFromWKT(geom_wkt), 5.0) FROM my_table")
```

## API Test: `ST_Force4D`

### Signature
```python
ST_Force4D(arg1: geometry) -> geometry | ST_Force4D(arg1: geometry, z: double) -> geometry | ST_Force4D(arg1: geometry, z: double, m: double) -> geometry | ST_Force4D(arg1: geography) -> geography | ST_Force4D(arg1: geography, z: double) -> geography | ST_Force4D(arg1: geography, z: double, m: double) -> geography
```
_Source: source/docs/reference/sql/st_force4d.qmd:19_

## Description

If the geometry already has Z and M values they are preserved; otherwise optional `z` and `m` arguments (both default 0) are used.

## Examples

```sql
SELECT ST_Force4D(ST_GeomFromWKT('POINT (1 2)'), 10.0, 20.0);
```

```sql
SELECT ST_Force4D(ST_GeomFromWKT('POINT (1 2)'), 10.0);
```

```sql
SELECT ST_Force4D(ST_GeomFromWKT('POINT (1 2)'));
```

```sql
SELECT ST_Force4D(ST_GeomFromWKT('POINT ZM (1 2 3 4)'), 10.0, 20.0);
```

```sql
SELECT ST_Force4D(ST_GeogFromWKT('POINT ZM (1 2 3 4)'), 10.0, 20.0);
```


### Goal
`ST_Force4D` converts a geometry or geography object into a 4D coordinate model (XYZM), ensuring Z and M dimensions are present by applying default values if they are missing.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to be transformed into a 4D coordinate model.
- `z` (`double`): The Z-coordinate value to assign if the input lacks a Z dimension. Defaults to 0.
- `m` (`double`): The M-coordinate value to assign if the input lacks an M dimension. Defaults to 0.

### Input
The function accepts `geometry` or `geography` types. Inputs can be constructed via standard SedonaDB constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. If the input already contains Z or M values, those values are preserved rather than overwritten.

### Output
Returns a `geometry` or `geography` object (matching the input type) that is guaranteed to have XYZM coordinates.

### Valid Call Patterns
```python
# Example using geometry with explicit Z and M
result = sd.sql("SELECT ST_Force4D(ST_GeomFromWKT('POINT (1 2)'), 10.0, 20.0) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Force4D` within a `sd.sql()` query string.
- Ensure the input `arg1` is a valid geometry or geography object.
- If providing `m`, you must also provide `z`.
- Do not attempt to use this function on raster types; it is strictly for vector/geography data.

### Prompt Snippet
```text
ST_Force4D(geometry_column, z_value, m_value)
```

### Common Failure Modes
- Providing only the `m` parameter without a `z` parameter (the signature requires `z` to be defined if `m` is provided).
- Passing a raster object instead of a geometry or geography object.
- Mismatched types between the input object and the expected return type in complex query chains.

### Fix Code Hint
```python
# Ensure Z is provided if M is provided
query = "SELECT ST_Force4D(geom, 0.0, 100.0) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_GeoHash`

### Signature
```python
ST_GeoHash(arg1: geometry) -> string | ST_GeoHash(arg1: geometry, precision: integer) -> string | ST_GeoHash(arg1: geography) -> string | ST_GeoHash(arg1: geography, precision: integer) -> string
```
_Source: source/docs/reference/sql/st_geohash.qmd:19_

### Goal
Encodes a geometry or geography object into a base-32 GeoHash string representation for spatial indexing and clustering.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to encode. Non-point geometries are encoded using the center of their bounding box.
- `precision` (`integer`): The number of base-32 characters in the returned GeoHash. Capped at 20; non-positive values return an empty string.

### Input
- **Data:** Geometry or Geography types.
- **Preconditions:** 
    - The input must use the WGS84 datum (`EPSG:4326` or `OGC:CRS84`) or have no CRS defined. Other CRSes will raise an error; use `ST_Transform()` to reproject first.
    - Geography arguments require the `s2geography` bounder to be registered on the session.
    - If `precision` is omitted, the input must be a `POINT`.

### Output
Returns `string` — A base-32 encoded GeoHash string representing the spatial location.

### Valid Call Patterns
```python
# Example 1: Encoding a point with specific precision
result = sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POINT (21.4234 52.0423)'), 10)").to_arrow_table()
print(result)
# __CHECK__

# Example 2: Encoding a polygon with specific precision
result = sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'), 8)").to_arrow_table()
print(result)
# __CHECK__

# Example 3: Encoding a point with default maximum precision (20)
result = sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POINT (21.4234 52.0423)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always verify that the input geometry is in WGS84 (`EPSG:4326`) before calling `ST_GeoHash`.
- If the input is not a `POINT` and `precision` is omitted, do not call the function as it will raise an error.
- If the input CRS is unknown or non-WGS84, suggest wrapping the input in `ST_Transform(geom, 4326)`.

### Prompt Snippet
```text
ST_GeoHash(geometry_column, precision_integer)
```

### Common Failure Modes
- **CRS Mismatch:** Passing a geometry with a non-WGS84 CRS (e.g., NAD83) will raise an error.
- **Missing Precision:** Calling `ST_GeoHash` without a `precision` argument on a non-point geometry (e.g., `POLYGON`) will raise an error.
- **Out of Bounds:** Latitudes outside the range [-90, 90] will return `NULL`.
- **Geography Bounder:** Failing to register the `s2geography` bounder when using `geography` types.

### Fix Code Hint
```python
# Reproject to WGS84 before encoding
query = "SELECT ST_GeoHash(ST_Transform(geom, 4326), 10) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_GeogFromWKB`

### Signature
```python
ST_GeogFromWKB(wkb: binary) -> geography
```
_Source: source/docs/reference/sql/st_geogfromwkb.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_GeogFromWKB
description: Constructs a Geography from WKB Binary.
kernels:
  - returns: geography
    args:
    - name: wkb
      type: binary
---

## Examples

```sql
SELECT ST_GeogFromWKB(
  x'010200000002000000000000000084d600c0000000000080b5d6bf00000060e1eff7bf00000080075de5bf'
);
```


### Goal
Constructs a `geography` object from a Well-Known Binary (WKB) representation to enable spherical-based geospatial analysis in SedonaDB.

### Parameters
- `wkb` (`binary`): A binary string representing the geometry in Well-Known Binary (WKB) format.

### Input
The input must be a valid WKB binary sequence. Ensure the binary data is correctly formatted as a hex literal (e.g., `x'...'`) when passed directly within a SQL query string.

### Output
Returns `geography` — A geography object representing the spatial feature, which allows for calculations based on the Earth's curvature rather than a flat Cartesian plane.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using a hex literal WKB string
query = "SELECT ST_GeogFromWKB(x'010200000002000000000000000084d600c0000000000080b5d6bf00000060e1eff7bf00000080075de5bf') as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a geography object materialized from the WKB input.
```

### LLM Instruction Prompt
- Always use `ST_GeogFromWKB` when the input is a binary WKB blob and the intended analysis requires spherical geography calculations.
- Ensure the input is provided as a valid binary type or hex literal.

### Prompt Snippet
```text
Use ST_GeogFromWKB(wkb_binary) to convert WKB data into a geography type for SedonaDB spatial operations.
```

### Common Failure Modes
- **Invalid WKB:** Passing a malformed or truncated binary string will result in a runtime error during query execution.
- **Type Mismatch:** Attempting to pass a WKT string instead of a binary WKB will cause a type error; use `ST_GeogFromText` for WKT inputs instead.

### Fix Code Hint
```python
# If you have a WKT string, use ST_GeogFromText instead:
sd.sql("SELECT ST_GeogFromText('POINT(0 0)')")

# If you have binary WKB, ensure it is properly escaped as a hex literal:
sd.sql("SELECT ST_GeogFromWKB(x'010100000000000000000000000000000000000000')")
```

## API Test: `ST_GeogFromWKT`

### Signature
```python
ST_GeogFromWKT(wkt: string) -> geography | ST_GeogFromWKT(wkt: string, srid: integer) -> geography
```
_Source: source/docs/reference/sql/st_geogfromwkt.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_GeogFromWKT
description: Constructs a Geography from WKT.
kernels:
  - returns: geography
    args:
    - name: wkt
      type: string
  - returns: geography
    args:
    - name: wkt
      type: string
    - name: srid
      type: integer
---

## Description

Alias: `ST_GeogFromText`.

## Examples

```sql
SELECT ST_GeogFromWKT('LINESTRING (1 2, 3 4, 5 6)');
```


### Goal
Constructs a `geography` object from a Well-Known Text (WKT) string representation for use in spherical geospatial analysis.

### Parameters
- `wkt` (`string`): A valid Well-Known Text string representing a spatial feature (e.g., 'POINT(0 0)').
- `srid` (`integer`): An optional Spatial Reference System Identifier to define the coordinate system of the geography.

### Input
The input must be a valid WKT string. If an SRID is provided, it must be a valid integer representing a supported coordinate reference system.

### Output
Returns a `geography` type, which is a SedonaDB object representing spatial data on a sphere, suitable for geodesic calculations.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_GeogFromWKT('LINESTRING (0 0, 1 1)')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_GeogFromWKT('POINT (0 0)', 4326)").to_arrow_table()

# __CHECK__
# result = sd.sql("SELECT ST_GeogFromWKT('POINT (0 0)')").to_arrow_table()
# assert result is not None
```

### LLM Instruction Prompt
- Always use `ST_GeogFromWKT` when the analysis requires spherical distance or area calculations rather than planar geometry.
- If the input WKT is malformed, the function will raise a SQL execution error.
- Use the alias `ST_GeogFromText` if preferred for compatibility with other spatial database standards.

### Prompt Snippet
```text
Use ST_GeogFromWKT(wkt_string) or ST_GeogFromWKT(wkt_string, srid) to create a geography object.
```

### Common Failure Modes
- **Invalid WKT:** Providing a string that does not conform to the WKT specification will cause the query to fail.
- **Unsupported SRID:** Providing an SRID that is not registered in the database's coordinate system registry.
- **Type Mismatch:** Attempting to use a `geography` object in functions that strictly require `geometry` (planar) types without explicit conversion.

### Fix Code Hint
```python
# Ensure WKT is well-formed and SRID is valid
query = "SELECT ST_GeogFromWKT('POINT(10 20)', 4326)"
result = sd.sql(query).to_arrow_table()
```

## API Test: `ST_GeogPoint`

### Signature
```python
ST_GeogPoint(longitude: double, latitude: double) -> geography
```
_Source: source/docs/reference/sql/st_geogpoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_GeogPoint
description: Creates a geography POINT from given longitude and latitude coordinates.
kernels:
  - returns: geography
    args:
    - name: longitude
      type: double
    - name: latitude
      type: double
---

## Description

Creates a geography Point from longitude and latitude coordinates.

## Examples

```sql
SELECT ST_GeogPoint(-64, 45);
```


### Goal
Constructs a `geography` type Point object from specified decimal degree coordinates for use in geodesic spatial operations.

### Parameters
- `longitude` (`double`): The longitudinal coordinate in decimal degrees.
- `latitude` (`double`): The latitudinal coordinate in decimal degrees.

### Input
Requires two `double` precision floating-point values representing coordinates. These inputs are typically used in contexts where geodesic distance or area calculations are required, as opposed to planar `geometry` types.

### Output
Returns `geography` — A geography-typed Point object representing the location on the Earth's surface.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify creation of a geography point
result = sd.sql("SELECT ST_AsBinary(ST_GeogPoint(1.0, 2.0))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_GeogPoint` when the downstream operation requires geodesic calculations (e.g., distance on a sphere/ellipsoid).
- Ensure inputs are provided as `double` types.
- Use this function within `sd.sql()` queries.

### Prompt Snippet
```text
ST_GeogPoint(longitude, latitude)
```

### Common Failure Modes
- Passing non-numeric types (e.g., strings) as arguments will result in a SQL execution error.
- Attempting to use the resulting `geography` object in functions strictly requiring `geometry` types may cause type mismatch errors depending on the specific spatial function's requirements.

### Fix Code Hint
```python
# Ensure coordinates are cast to double if coming from a different source
sd.sql("SELECT ST_GeogPoint(CAST(lon AS DOUBLE), CAST(lat AS DOUBLE)) FROM my_table")
```

## API Test: `ST_GeomCollFromText`

### Signature
```python
ST_GeomCollFromText(wkt: string) -> geometry | ST_GeomCollFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomcollfromtext.qmd:19_

### Goal
Constructs a `GeometryCollection` object from a Well-Known Text (WKT) string, ensuring the input strictly conforms to the `GEOMETRYCOLLECTION` type.

### Parameters
- `wkt` (`string`): A Well-Known Text representation of a geometry. Must represent a `GEOMETRYCOLLECTION`.
- `srid` (`crs`): An optional Coordinate Reference System identifier (e.g., 4326) to associate with the resulting geometry.

### Input
- **Data:** A valid WKT string starting with `GEOMETRYCOLLECTION`.
- **Preconditions:** The input string must be a valid WKT representation of a `GeometryCollection`. If the string represents a different geometry type (e.g., `POINT`, `POLYGON`), the function will raise an error.

### Output
Returns `geometry` — A materialized `GeometryCollection` object that can be used in subsequent spatial operations or stored in a table.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))')").to_arrow_table()
print("__CHECK__", result)

# Construction with SRID
result_srid = sd.sql("SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))', 4326)").to_arrow_table()
print("__CHECK__", result_srid)
```

### LLM Instruction Prompt
- Use `ST_GeomCollFromText` only when the input WKT is explicitly a `GEOMETRYCOLLECTION`.
- If an SRID is provided, ensure it is a valid integer CRS identifier.
- Always wrap the function call in `sd.sql("...").to_arrow_table()` when executing via the Python interface.

### Prompt Snippet
```text
ST_GeomCollFromText(wkt_string, [srid_integer])
```

### Common Failure Modes
- **Type Mismatch:** Passing a WKT string that is not a `GEOMETRYCOLLECTION` (e.g., `POINT(0 0)`) will cause the function to error.
- **Invalid WKT:** Providing a malformed WKT string that does not follow OGC standards will result in a parsing error.
- **CRS Mismatch:** If the resulting geometry is used in spatial operations with other geometries, ensure the SRIDs match, as SedonaDB requires consistent CRS for spatial predicates.

### Fix Code Hint
```python
# Ensure the WKT string is wrapped in GEOMETRYCOLLECTION
# Incorrect: ST_GeomCollFromText('POINT(0 0)')
# Correct: ST_GeomCollFromText('GEOMETRYCOLLECTION(POINT(0 0))')
```

## API Test: `ST_GeomFromEWKB`

### Signature
```python
ST_GeomFromEWKB(ewkb: binary) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromewkb.qmd:19_

### Goal
Constructs a geometry object from an Extended Well-Known Binary (EWKB) representation, enabling the import of geometries that include SRID information.

### Parameters
- `ewkb` (`binary`): A binary string representing the geometry in EWKB format, which includes the geometry type, coordinates, and optional SRID header.

### Input
The input must be a valid `binary` type containing EWKB data. This format is commonly used for PostGIS compatibility. Ensure the binary string is correctly formatted as a hex literal (e.g., `x'...'`) when passed directly in SQL queries.

### Output
Returns a `geometry` object that can be used in subsequent spatial operations such as spatial joins, predicates, or transformations.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using a hex literal for a Point(1.0, 2.0)
query = "SELECT ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040') as geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Use `ST_GeomFromEWKB` when the input data is provided as a binary EWKB string.
- Always wrap the binary input in a hex literal `x'...'` when writing raw SQL strings.
- Ensure the binary input is valid EWKB; malformed binary strings will result in a runtime error during query execution.

### Prompt Snippet
```text
ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040')
```

### Common Failure Modes
- **Invalid Binary Format:** Passing a string that is not a valid EWKB binary representation will cause the function to fail.
- **Type Mismatch:** Passing a non-binary type (e.g., a standard string or integer) will result in a type error.
- **Missing Hex Prefix:** Forgetting the `x` prefix in SQL (e.g., `'0101...'` instead of `x'0101...'`) may cause the engine to interpret the input as a standard string rather than binary data.

### Fix Code Hint
```python
# Ensure the input is a valid hex-encoded binary string
# Correct:
sd.sql("SELECT ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040')")

# Incorrect:
# sd.sql("SELECT ST_GeomFromEWKB('0101000000000000000000f03f0000000000000040')")
```

## API Test: `ST_GeomFromEWKT`

### Signature
```python
ST_GeomFromEWKT(ewkt: string) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromewkt.qmd:19_

### Goal
Constructs a geometry object from an Extended Well-Known Text (EWKT) string, allowing for the explicit definition of a Coordinate Reference System (CRS) via an embedded SRID.

### Parameters
- `ewkt` (`string`): A string formatted as `SRID=<srid>;<wkt>`, where `<srid>` is the integer identifier for the spatial reference system and `<wkt>` is the standard Well-Known Text representation of the geometry.

### Input
- **Data Format:** A string literal or column containing valid EWKT.
- **Preconditions:** The SRID provided in the string must be recognized by the SedonaDB environment. The WKT portion must follow standard OGC geometry definitions (e.g., `POINT`, `POLYGON`).

### Output
Returns `geometry` — A materialized geometry object with the specified SRID metadata attached, ready for use in spatial operations like joins, intersections, or raster clipping.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example: Constructing a point with SRID 4326
query = "SELECT ST_GeomFromEWKT('SRID=4326;POINT(40.7128 -74.0060)') as geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Use `ST_GeomFromEWKT` when you need to instantiate a geometry from a string that includes explicit CRS information.
- Ensure the input string strictly follows the `SRID=<srid>;<wkt>` format.
- Do not use this function for standard WKT without an SRID; use `ST_GeomFromText` instead if no SRID is required.

### Prompt Snippet
```text
ST_GeomFromEWKT('SRID=<srid>;<wkt>')
```

### Common Failure Modes
- **Malformed String:** Providing a string that lacks the `SRID=` prefix or the semicolon separator will result in a parsing error.
- **Unsupported SRID:** Providing an SRID that is not defined in the database's spatial reference system registry.
- **Invalid WKT:** Providing a WKT string that does not conform to standard geometry definitions (e.g., unclosed polygons).

### Fix Code Hint
```python
# Ensure the string is properly formatted
ewkt_input = "SRID=4326;POINT(40.7128 -74.0060)"
sd.sql(f"SELECT ST_GeomFromEWKT('{ewkt_input}')").to_arrow_table()
```

## API Test: `ST_GeomFromWKB`

### Signature
```python
ST_GeomFromWKB(wkb: binary) -> geometry | ST_GeomFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromwkb.qmd:19_

### Goal
Constructs a geometry object from a Well-Known Binary (WKB) representation, optionally assigning a specific Coordinate Reference System (CRS).

### Parameters
- `wkb` (`binary`): The binary representation of the geometry to be parsed.
- `srid` (`crs`): An optional identifier (integer or string) representing the Coordinate Reference System to associate with the geometry.

### Input
The input must be a valid `binary` type representing a WKB-encoded geometry. When using SQL, binary data is often provided via the `decode(string, 'hex')` function. If an `srid` is provided, it must be a valid CRS identifier recognized by the engine.

### Output
Returns a `geometry` object representing the spatial feature defined by the input WKB.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_GeomFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'))").to_arrow_table()
print("__CHECK__")

# Construction with SRID
result = sd.sql("SELECT ST_GeomFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326)").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `decode(..., 'hex')` when passing hex-encoded strings as WKB binary inputs in SQL.
- If the geometry requires a specific spatial reference, provide the `srid` as the second argument.
- Ensure the input binary is a valid WKB format; the function performs validation by default.

### Prompt Snippet
```text
ST_GeomFromWKB(binary_wkb, [srid])
```

### Common Failure Modes
- **Invalid WKB:** Providing a malformed binary string that does not conform to the WKB specification will result in a parsing error.
- **Type Mismatch:** Passing a raw string instead of a `binary` type (e.g., forgetting the `decode` function in SQL) will cause a type error.
- **CRS Mismatch:** If the geometry is used in subsequent operations (like `RS_Clip` or spatial joins) with other geometries or rasters, ensure the `srid` provided here matches the CRS of the other datasets to avoid runtime errors.

### Fix Code Hint
```python
# Ensure hex strings are decoded to binary
query = "SELECT ST_GeomFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326)"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_GeomFromWKT`

### Signature
```python
ST_GeomFromWKT(wkt: string) -> geometry | ST_GeomFromWKT(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromwkt.qmd:19_

### Goal
Constructs a SedonaDB `geometry` object from a Well-Known Text (WKT) string representation, optionally assigning a specific spatial reference system.

### Parameters
- `wkt` (`string`): A valid Well-Known Text string representing a geometry (e.g., 'POINT (30 10)', 'POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))').
- `srid` (`crs`): An optional spatial reference identifier (integer) or CRS string (e.g., 'OGC:CRS27') to associate with the geometry.

### Input
- **Data:** A string literal or column containing WKT-formatted geometry data.
- **Preconditions:** The WKT string must be syntactically correct according to OGC standards. If an SRID is provided, ensure it is supported by the engine's CRS registry.

### Output
Returns a `geometry` object that can be used in subsequent spatial operations (e.g., `ST_Area`, `ST_Boundary`, or spatial joins).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_GeomFromWKT('POINT (30 10)') as geom").to_arrow_table()
print("__CHECK__", result)

# Construction with SRID
result_srid = sd.sql("SELECT ST_GeomFromWKT('POINT (30 10)', 4326) as geom").to_arrow_table()
print("__CHECK__", result_srid)
```

### LLM Instruction Prompt
- Always use `ST_GeomFromWKT` to convert raw WKT strings into geometry objects before performing spatial analysis.
- If the geometry is intended for use with other datasets, ensure the `srid` parameter matches the CRS of the target data to avoid spatial mismatch errors.
- Use the aliases `ST_GeomFromText` or `ST_GeometryFromText` if required by existing SQL scripts.

### Prompt Snippet
```text
ST_GeomFromWKT(wkt_string, [srid])
```

### Common Failure Modes
- **Invalid WKT:** Passing a malformed string (e.g., missing parentheses or invalid coordinates) will result in a runtime SQL error.
- **CRS Mismatch:** Performing operations between geometries with different SRIDs (or one with an SRID and one without) may cause errors or incorrect results in spatial joins and predicates.
- **Type Mismatch:** Attempting to pass a non-string type as the first argument will fail.

### Fix Code Hint
```python
# Ensure WKT is a valid string and SRID is a supported integer or CRS string
query = "SELECT ST_GeomFromWKT('POINT (0 0)', 4326)"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_GeometryN`

### Signature
```python
ST_GeometryN(arg1: geometry, n: integer) -> geometry | ST_GeometryN(arg1: geography, n: integer) -> geography
```
_Source: source/docs/reference/sql/st_geometryn.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_GeometryN
description: Returns the 1-based Nth geometry if the geometry is a GEOMETRYCOLLECTION, (MULTI)POINT, (MULTI)LINESTRING, MULTICURVE or (MULTI)POLYGON.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: "n"
      type: integer
  - returns: geography
    args:
    - geography
    - name: "n"
      type: integer
---

## Description

Returns NULL if the index is out of range.

## Examples

```sql
SELECT ST_GeometryN(
    ST_GeomFromText('MULTIPOINT ((1 2), (3 4), (5 6))'),
    1
);
```

```sql
SELECT ST_GeometryN(
    ST_GeogFromWKT('MULTIPOINT ((1 2), (3 4), (5 6))'),
    1
);
```


### Goal
Extracts a specific component geometry from a collection or multi-geometry object using a 1-based index.

### Parameters
- `arg1` (`geometry`): The input geometry or geography collection (e.g., MULTIPOINT, MULTILINESTRING, MULTIPOLYGON, or GEOMETRYCOLLECTION).
- `n` (`integer`): The 1-based index of the geometry to retrieve from the collection.

### Input
The function accepts standard geometry or geography types. The input must be a collection type (e.g., MULTI* or GEOMETRYCOLLECTION). If the input is not a collection, the behavior is undefined or may return NULL.

### Output
Returns a `geometry` or `geography` object corresponding to the Nth element of the input collection. Returns `NULL` if the index `n` is out of range.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Extract the first point from a MULTIPOINT
query = "SELECT ST_GeometryN(ST_GeomFromText('MULTIPOINT ((1 2), (3 4), (5 6))'), 1) as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use 1-based indexing for the `n` parameter.
- Ensure the input `arg1` is a collection type (MULTIPOINT, MULTILINESTRING, MULTIPOLYGON, or GEOMETRYCOLLECTION).
- If the index `n` is less than 1 or greater than the number of elements in the collection, the function will return `NULL`.

### Prompt Snippet
```text
ST_GeometryN(geometry_column, index_integer)
```

### Common Failure Modes
- **Index Out of Range:** Providing an index `n` that is less than 1 or greater than the number of geometries in the collection results in a `NULL` return value.
- **Invalid Input Type:** Passing a simple geometry (e.g., a single POINT or POLYGON) instead of a collection type may result in unexpected behavior or `NULL`.

### Fix Code Hint
```python
# Ensure index is within bounds by checking the count first if necessary
# Example: SELECT ST_GeometryN(geom, 1) FROM table WHERE ST_NumGeometries(geom) >= 1
```

## API Test: `ST_GeometryType`

### Signature
```python
ST_GeometryType(arg1: geometry) -> string | ST_GeometryType(arg1: geography) -> string
```
_Source: source/docs/reference/sql/st_geometrytype.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_GeometryType
description: Returns the type of a geometry or geography.
kernels:
  - returns: string
    args: [geometry]
  - returns: string
    args: [geography]
---

## Description

Returns the OGC geometry type name prefixed with `ST_`. Possible return values
include: `'ST_Point'`, `'ST_LineString'`, `'ST_Polygon'`, `'ST_MultiPoint'`,
`'ST_MultiLineString'`, `'ST_MultiPolygon'`, and `'ST_GeometryCollection'`.

## Examples

```sql
SELECT ST_GeometryType(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'));
```

```sql
SELECT ST_GeometryType(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'));
```


### Goal
Identifies the OGC geometry type of a given spatial object (geometry or geography) by returning its name as a string prefixed with `ST_`.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object whose type is to be determined.

### Input
The function accepts valid geometry or geography objects. These are typically generated via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. No specific CRS preconditions are required for this metadata inspection.

### Output
Returns `string` — The OGC geometry type name (e.g., `'ST_Point'`, `'ST_Polygon'`).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify geometry type
result = sd.sql("SELECT ST_GeometryType(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))')) as type").to_arrow_table()
assert result["type"][0] == "ST_Polygon"
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_GeometryType(arg)` within a `sd.sql()` query.
- Ensure the input is a valid spatial object (geometry or geography).
- The return value is a string; do not expect a geometry object back.

### Prompt Snippet
```text
ST_GeometryType(geometry_column)
```

### Common Failure Modes
- Passing a non-spatial data type (e.g., integer or string) directly without a constructor like `ST_GeomFromWKT` will result in a SQL execution error.
- Expecting the function to return a geometry object instead of a string.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type first
sd.sql("SELECT ST_GeometryType(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_HasM`

### Signature
```python
ST_HasM(arg1: geometry) -> boolean | ST_HasM(arg1: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_hasm.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_HasM
description: Returns true if the geometry has a M dimension.
kernels:
  - returns: boolean
    args: [geometry]
  - returns: boolean
    args: [geography]
---

## Examples

```sql
SELECT ST_HasM(ST_GeomFromWKT('POINT M (1 2 3)'));
```

```sql
SELECT ST_HasM(ST_GeomFromWKT('POINT (1 2)'));
```


### Goal
Determines whether a given geometry or geography object contains an M (measure) dimension.

### Parameters
- `arg1` (`geometry` | `geography`): The spatial object to inspect for the presence of an M coordinate.

### Input
The function accepts standard geometry or geography types. Inputs are typically constructed via WKT (e.g., `ST_GeomFromWKT`) or other geometry-generating functions. No specific CRS preconditions are required for this check.

### Output
Returns a `boolean` value: `true` if the geometry has an M dimension, and `false` otherwise.

### Valid Call Patterns
```python
# Example inferred from signature and documentation
import sedona.db
sd = sedona.db.connect()

# Verify M dimension presence
result = sd.sql("SELECT ST_HasM(ST_GeomFromWKT('POINT M (1 2 3)')) as has_m").to_arrow_table()
assert result["has_m"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_HasM` to verify the dimensionality of a geometry before performing operations that might require or ignore M-coordinates.
- Always pass a valid geometry or geography object as the argument.

### Prompt Snippet
```text
ST_HasM(geometry_expression)
```

### Common Failure Modes
- Passing a `NULL` geometry will result in a `NULL` output rather than a boolean.
- Passing non-spatial types (e.g., strings or integers) that are not cast to geometry will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure input is cast to geometry if it is a raw string
sd.sql("SELECT ST_HasM(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_HasZ`

### Signature
```python
ST_HasZ(arg1: geometry) -> boolean | ST_HasZ(arg1: geography) -> boolean
```
_Source: source/docs/reference/sql/st_hasz.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_HasZ
description: Returns true if the geometry has a Z dimension.
kernels:
  - returns: boolean
    args: [geometry]
  - returns: boolean
    args: [geography]
---

## Examples

```sql
SELECT ST_HasZ(ST_GeomFromWKT('POINT Z (1 2 3)'));
```

```sql
SELECT ST_HasZ(ST_GeomFromWKT('POINT (1 2)'));
```


### Goal
Determines whether a given geometry or geography object contains a Z-coordinate (elevation) dimension.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to inspect for the presence of a Z-coordinate.

### Input
The function accepts standard geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or loaded from supported vector formats (GeoParquet, WKT, WKB).

### Output
Returns `boolean`: `true` if the geometry has a Z dimension, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify Z dimension presence
result = sd.sql("SELECT ST_HasZ(ST_GeomFromWKT('POINT Z (1 2 3)')) as has_z").to_arrow_table()
assert result["has_z"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_HasZ` to check for 3D coordinate data in geometry columns.
- The function is a scalar function and should be used within a `SELECT` statement.
- Ensure the input is a valid geometry or geography object.

### Prompt Snippet
```text
ST_HasZ(geometry_column)
```

### Common Failure Modes
- Passing a non-geometry/non-geography type (e.g., a string or integer) will result in a SQL execution error.
- Passing a null geometry will return `null` rather than `false`.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_HasZ(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_HausdorffDistance`

### Signature
```python
ST_HausdorffDistance(arg1: geometry, arg2: geometry) -> double | ST_HausdorffDistance(arg1: geometry, arg2: geometry, densifyFrac: double) -> double
```
_Source: source/docs/reference/sql/st_hausdorffdistance.qmd:19_

### Goal
Calculates the Hausdorff distance between two geometries, providing a measure of similarity by identifying the greatest distance from any point in one geometry to the nearest point in the other.

### Parameters
- `arg1` (`geometry`): The first input geometry (e.g., Point, LineString, Polygon).
- `arg2` (`geometry`): The second input geometry to compare against the first.
- `densifyFrac` (`double`): Optional fraction used to densify segments before calculation; a value of 0.25 adds vertices at 1/4 of the segment length to improve accuracy for complex shapes.

### Input
- **Data:** Geometries must be provided in standard formats supported by SedonaDB (WKT, WKB, or GeoParquet).
- **Preconditions:** Both geometries must be valid. If either geometry is `NULL`, the function returns `NULL`. If either geometry is empty, the function returns 0.

### Output
Returns `double` — The calculated Hausdorff distance representing the maximum distance between the two shapes.

### Valid Call Patterns
```python
# Basic usage with two geometries
result = sd.sql("SELECT ST_HausdorffDistance(ST_Point(0, 0), ST_Point(3, 4))").to_arrow_table()

# Usage with densification fraction
query = """
SELECT ST_HausdorffDistance(
    ST_GeomFromText('LINESTRING (130 0, 0 0, 0 150)'),
    ST_GeomFromText('LINESTRING (10 10, 10 150, 130 10)'),
    0.5
)
"""
result = sd.sql(query).to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` to execute the function.
- Ensure both `arg1` and `arg2` are valid geometry objects or WKT strings.
- Use the `densifyFrac` parameter only when dealing with geometries containing long segments where higher precision is required.

### Prompt Snippet
```text
ST_HausdorffDistance(geometry_a, geometry_b)
ST_HausdorffDistance(geometry_a, geometry_b, densify_fraction)
```

### Common Failure Modes
- **NULL Inputs:** Passing a `NULL` geometry will result in a `NULL` output rather than an error.
- **Empty Geometries:** Passing an empty geometry returns 0, which may be unexpected if the logic requires a non-zero distance.
- **Performance:** For very complex geometries, omitting `densifyFrac` may lead to less accurate results, while an excessively small `densifyFrac` may increase computation time.

### Fix Code Hint
```python
# Ensure geometries are not NULL before calling
query = "SELECT ST_HausdorffDistance(g1, g2) FROM table WHERE g1 IS NOT NULL AND g2 IS NOT NULL"
```

## API Test: `ST_InteriorRingN`

### Signature
```python
ST_InteriorRingN(arg1: geometry, n: integer) -> geometry | ST_InteriorRingN(arg1: geography, n: integer) -> geography
```
_Source: source/docs/reference/sql/st_interiorringn.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_InteriorRingN
description: Returns the Nth interior ring of a polygon.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: "n"
      type: integer
  - returns: geography
    args:
    - geography
    - name: "n"
      type: integer
---

### Goal
Extracts the Nth interior `LINESTRING` ring from a `POLYGON` geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The input polygon containing one or more interior rings.
- `n` (`integer`): The 1-based index of the interior ring to retrieve.

### Input
- **Data:** A `POLYGON` geometry or geography object.
- **Preconditions:** The input must be a valid polygon. If the input is not a polygon, or if `n` is out of range (e.g., `n` is less than 1 or greater than the number of interior rings), the function returns `NULL`.

### Output
Returns a `LINESTRING` representing the Nth interior ring of the input polygon, matching the input type (geometry or geography).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = """
SELECT ST_InteriorRingN(
    ST_GeomFromText('POLYGON((0 0, 0 5, 5 5, 5 0, 0 0), (1 1, 2 1, 2 2, 1 2, 1 1))'),
    1
) as ring
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_InteriorRingN(geometry, n)` or `ST_InteriorRingN(geography, n)` syntax.
- Remember that `n` is 1-based; `1` retrieves the first interior ring.
- Ensure the input is a polygon; non-polygon inputs or invalid indices result in `NULL`.

### Prompt Snippet
```text
ST_InteriorRingN(polygon_geom, 1)
```

### Common Failure Modes
- **Index Out of Range:** Providing an `n` value that exceeds the number of interior rings in the polygon returns `NULL`.
- **Invalid Geometry Type:** Passing a `POINT` or `LINESTRING` instead of a `POLYGON` returns `NULL`.
- **1-based Indexing Error:** Attempting to access the first ring using `0` will return `NULL` because the index is 1-based.

### Fix Code Hint
```python
# Ensure n is at least 1 and verify the geometry is a polygon
# If n is unknown, check the number of rings first using ST_NumInteriorRings(arg1)
query = "SELECT ST_InteriorRingN(geom, 1) FROM table WHERE ST_NumInteriorRings(geom) >= 1"
```

## API Test: `ST_Intersection`

### Signature
```python
ST_Intersection(arg1: geometry, arg2: geometry) -> geometry | ST_Intersection(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_intersection.qmd:19_

### Goal
Computes the spatial intersection of two geometries or geographies, returning the shared region as a new geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The first spatial object to intersect.
- `arg2` (`geometry` | `geography`): The second spatial object to intersect.

### Input
- **Data Formats:** Accepts standard SedonaDB geometry types (WKT, WKB, GeoParquet).
- **Preconditions:** Both inputs must share the same Coordinate Reference System (CRS). If inputs have mismatched CRSs, the operation will fail or produce undefined results. Ensure geometries are valid before performing intersection.

### Output
Returns `geography | geometry` representing the intersection of the two input objects. The output type matches the input type (geometry or geography).

### Valid Call Patterns
```python
# SQL execution via Python
result = sd.sql("""
    SELECT ST_Intersection(geom_a, geom_b) as intersection_result 
    FROM my_table
""").to_arrow_table()

# Literal WKT example
result = sd.sql("""
    SELECT ST_Intersection(
        ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'),
        ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
    )
""").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both arguments to `ST_Intersection` are of the same type (both `geometry` or both `geography`).
- If the inputs are in different coordinate systems, explicitly wrap them in a reprojection function (e.g., `ST_Transform`) before passing them to `ST_Intersection`.
- Use `ST_GeomFromText` or similar constructors to prepare literal inputs.

### Prompt Snippet
```text
ST_Intersection(geom_a, geom_b)
```

### Common Failure Modes
- **CRS Mismatch:** Attempting to intersect geometries with different SRIDs will result in an error.
- **Invalid Geometries:** Intersecting self-intersecting or otherwise invalid polygons may return unexpected results or errors.
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` is not supported by the function signature.

### Fix Code Hint
```python
# Ensure CRS alignment before intersection
sd.sql("""
    SELECT ST_Intersection(
        ST_Transform(geom_a, 4326), 
        ST_Transform(geom_b, 4326)
    ) FROM my_table
""").to_arrow_table()
```

## API Test: `ST_Intersection_Agg`

### Signature
```python
ST_Intersection_Agg(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_intersection_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Intersection_Agg
description: Returns the cumulative intersection of all geometries in the input.
kernels:
  - returns: geometry
    args: [geometry]
---

## Examples

```sql
SELECT ST_Intersection_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('POLYGON ((0 0, 0 4, 4 4, 4 0, 0 0))')),
    (ST_GeomFromText('POLYGON ((2 2, 2 6, 6 6, 6 2, 2 2))'))
) AS t(geom);
```


### Goal
Computes the cumulative spatial intersection of a set of geometry objects, returning a single geometry representing the overlapping region shared by all inputs.

### Parameters
- `arg1` (`geometry`): A column or expression containing the geometry objects to be intersected.

### Input
- **Data:** Accepts standard geometry types (e.g., Polygon, MultiPolygon) provided via SQL queries or registered views.
- **Preconditions:** All input geometries must share the same Coordinate Reference System (CRS). If geometries have mismatched CRSs, the operation may fail or produce undefined results.

### Output
Returns `geometry` — A single geometry object representing the intersection of all input geometries. If the intersection is empty, the result is an empty geometry.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using a SQL query to aggregate geometries
query = """
SELECT ST_Intersection_Agg(geom) 
FROM (
  VALUES 
    (ST_GeomFromText('POLYGON ((0 0, 0 4, 4 4, 4 0, 0 0))')), 
    (ST_GeomFromText('POLYGON ((2 2, 2 6, 6 6, 6 2, 2 2))'))
) AS t(geom)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a Polygon representing the intersection (2,2 to 4,4).
```

### LLM Instruction Prompt
- Use `ST_Intersection_Agg` as an aggregate function within a `SELECT` statement.
- Ensure all input geometries are valid and share the same CRS before aggregation.
- The function expects a single geometry column as input.

### Prompt Snippet
```text
ST_Intersection_Agg(geometry_column)
```

### Common Failure Modes
- **CRS Mismatch:** Attempting to aggregate geometries with different CRSs will result in an error or invalid spatial output.
- **Empty Intersection:** If the input geometries do not overlap, the function returns an empty geometry.
- **Invalid Geometry:** Providing non-geometry types or malformed WKT/WKB will cause the query to fail.

### Fix Code Hint
```python
# Ensure all geometries are in the same CRS before aggregation
query = """
SELECT ST_Intersection_Agg(ST_Transform(geom, 'EPSG:4326')) 
FROM my_table
"""
```

## API Test: `ST_Intersects`

### Signature
```python
ST_Intersects(arg1: geometry, arg2: geometry) -> boolean | ST_Intersects(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_intersects.qmd:19_

### Goal
Determines whether two spatial objects (geometries or geographies) share any common space, returning true if they intersect and false otherwise.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to evaluate for intersection.
- `arg2` (`geometry` or `geography`): The second spatial object to evaluate for intersection.

### Input
- **Data Formats:** Accepts standard spatial types including WKT, WKB, and registered views containing geometry or geography columns.
- **Preconditions:** Both arguments must share the same Coordinate Reference System (CRS). If inputs have mismatched CRSs, the operation will fail or produce undefined results; ensure both inputs are projected to the same CRS before comparison.

### Output
Returns `boolean` — `true` if the two spatial objects intersect, `false` if they do not.

### Valid Call Patterns
```python
# Using SQL via the Python connection
nyc_bbox_wkt = "POLYGON((-74.2591 40.4774, -74.2591 40.9176, -73.7004 40.9176, -73.7004 40.4774, -74.2591 40.4774))"

result = sd.sql(f"""
SELECT ST_Intersects(geometry, ST_GeomFromText('{nyc_bbox_wkt}', 4326))
FROM buildings
""").to_arrow_table()

print("__CHECK__ witness: Intersection check complete.")
```

### LLM Instruction Prompt
- Always use `ST_Intersects(arg1, arg2)` as a standalone function call within the `sd.sql()` query string.
- Ensure both arguments are explicitly cast or constructed using spatial constructors (e.g., `ST_GeomFromText`, `ST_Point`) if they are not already geometry/geography columns.
- Verify that both inputs are in the same CRS.

### Prompt Snippet
```text
ST_Intersects(geometry_column, ST_GeomFromText('POLYGON((...))', 4326))
```

### Common Failure Modes
- **CRS Mismatch:** Calling `ST_Intersects` with geometries in different coordinate systems will result in an error. Always reproject using `ST_Transform` if necessary.
- **Type Mixing:** Attempting to intersect a `geometry` type with a `geography` type directly may cause execution errors; ensure both arguments are of the same spatial type.
- **Invalid WKT:** Passing malformed WKT strings to `ST_GeomFromText` within the intersection call will cause the query to fail.

### Fix Code Hint
```python
# Ensure CRS alignment before intersection
sd.sql("""
SELECT ST_Intersects(
    geom_col, 
    ST_Transform(other_geom_col, 4326)
) FROM my_table
""").to_arrow_table()
```

## API Test: `ST_IsClosed`

### Signature
```python
ST_IsClosed(arg1: geometry) -> boolean | ST_IsClosed(arg1: geography) -> boolean
```
_Source: source/docs/reference/sql/st_isclosed.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsClosed
description: Returns true if the LINESTRING start and end point are the same.
kernels:
  - returns: boolean
    args: [geometry]
  - returns: boolean
    args: [geography]
---

## Description

Returns `true` if the LineString's start and end points are the same. For MultiLineStrings, all elements must be closed.

## Examples

```sql
SELECT ST_IsClosed(ST_GeomFromText('LINESTRING(0 0, 1 1, 1 0, 0 0)'));
```

```sql
SELECT ST_IsClosed(ST_GeogFromWKT('LINESTRING(0 0, 1 1, 1 0, 0 0)'));
```


### Goal
Determines if a spatial object (LineString or MultiLineString) is closed by verifying if its start and end coordinates are identical.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate. For `LineString` types, it checks the start and end points; for `MultiLineString` types, it returns true only if every constituent element is closed.

### Input
- **Data Formats:** Accepts WKT, WKB, or GeoParquet-loaded geometries.
- **Preconditions:** The input must be a `LineString` or `MultiLineString` type.

### Output
Returns `boolean` — `true` if the geometry is closed, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify a closed LineString
result = sd.sql("SELECT ST_IsClosed(ST_GeomFromText('LINESTRING(0 0, 1 1, 1 0, 0 0)')) as is_closed").to_arrow_table()
assert result["is_closed"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_IsClosed` to validate the topological closure of linear geometries.
- Ensure the input is a `LineString` or `MultiLineString` type.
- When using `geography` types, ensure the input is constructed via `ST_GeogFromWKT` or similar geography-aware constructors.

### Prompt Snippet
```text
ST_IsClosed(geometry_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `Point` or `Polygon` type may result in unexpected behavior or errors depending on the underlying engine implementation.
- **MultiLineString partial closure:** If one segment in a `MultiLineString` is not closed, the entire function returns `false`.
- **Null Inputs:** Passing a `NULL` geometry will return `NULL` rather than a boolean.

### Fix Code Hint
```python
# Ensure input is cast to the correct type if coming from raw text
sd.sql("SELECT ST_IsClosed(ST_GeomFromText('LINESTRING(0 0, 1 1, 0 0)'))")
```

## API Test: `ST_IsCollection`

### Signature
```python
ST_IsCollection(arg1: geometry) -> boolean | ST_IsCollection(arg1: geography) -> boolean
```
_Source: source/docs/reference/sql/st_iscollection.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsCollection
description: Returns true if the geometry or geography type of the input is a collection type.
kernels:
  - returns: boolean
    args: [geometry]
  - returns: boolean
    args: [geography]
---

## Description

Returns TRUE if the geometry type of the input is a geometry collection type. Collection types are the following:

- GEOMETRYCOLLECTION
- MULTIPOINT
- MULTIPOLYGON
- MULTILINESTRING

## Examples

```sql
SELECT ST_IsCollection(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'));
```

```sql
SELECT ST_IsCollection(ST_GeogFromWKT('MULTIPOINT ((0 0), (1 1))'));
```


### Goal
Determines if a given geometry or geography object is a collection type (e.g., MultiPoint, MultiPolygon, MultiLineString, or GeometryCollection).

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate.

### Input
The function accepts standard geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. No specific CRS preconditions are required for this type-check operation.

### Output
Returns `boolean`: `TRUE` if the input is a collection type, `FALSE` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify a MultiPoint geometry
result = sd.sql("SELECT ST_IsCollection(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))')) as is_coll").to_arrow_table()
assert result["is_coll"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_IsCollection` to validate if a spatial object is a collection type before performing operations that require singular geometries.
- The function accepts both `geometry` and `geography` types.

### Prompt Snippet
```text
ST_IsCollection(geometry_column)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) will result in a SQL execution error.
- Passing a singular geometry type (e.g., `POINT`, `POLYGON`) will correctly return `FALSE`, which may be unexpected if the logic requires a collection.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_IsCollection(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_IsEmpty`

### Signature
```python
ST_IsEmpty(arg1: geometry) -> boolean | ST_IsEmpty(arg1: geography) -> boolean
```
_Source: source/docs/reference/sql/st_isempty.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsEmpty
description: Returns true if the geometry or geography is empty.
kernels:
  - returns: boolean
    args: [geometry]
  - returns: boolean
    args: [geography]
---

## Examples

```sql
SELECT ST_IsEmpty(ST_GeomFromWKT('POINT EMPTY'));
```

```sql
SELECT ST_IsEmpty(ST_GeogFromWKT('POINT EMPTY'));
```


### Goal
Determines whether a given geometry or geography object contains no points or vertices (i.e., is empty).

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate. Expected values include standard WKT-derived geometries or geographies.

### Input
The function accepts geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. No specific CRS preconditions are required, as the emptiness check is a structural property of the object.

### Output
Returns `boolean` — `true` if the input object is empty (e.g., `POINT EMPTY`), and `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify geometry emptiness
result = sd.sql("SELECT ST_IsEmpty(ST_GeomFromWKT('POINT EMPTY')) as is_empty").to_arrow_table()
assert result["is_empty"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_IsEmpty` to validate if a geometry or geography object is empty.
- The function takes exactly one argument of type `geometry` or `geography`.
- Always wrap the call in `sd.sql("SELECT ST_IsEmpty(...)")` when using the Python interface.

### Prompt Snippet
```text
ST_IsEmpty(arg1)
```

### Common Failure Modes
- Passing a `NULL` value: The function will return `NULL` rather than a boolean.
- Passing non-spatial types: The function expects a valid geometry or geography object; passing strings or numbers directly without a constructor (e.g., `ST_GeomFromWKT`) will result in a type mismatch error.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_IsEmpty(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_IsRing`

### Signature
```python
ST_IsRing(arg1: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_isring.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsRing
description: Returns true if a linestring is ST_IsClosed and ST_IsSimple.
kernels:
  - returns: boolean
    args: [geometry]
---

## Description

Returns `true` if the linestring is both closed (`ST_IsClosed`) and simple (`ST_IsSimple`). In other words, the input forms a ring with no self-intersections.

## Examples

```sql
SELECT ST_IsRing(
  ST_GeomFromText('LINESTRING(0 0, 0 1, 1 1, 1 0, 0 0)')
);
```


### Goal
Determines if a given `geometry` (specifically a `LineString`) is both closed and simple, effectively verifying if it forms a valid ring without self-intersections.

### Parameters
- `arg1` (`geometry`): The input geometry to evaluate. Expected to be a `LineString`.

### Input
The input must be a valid `geometry` object. In SedonaDB, this is typically provided as a WKT string (e.g., via `ST_GeomFromText`) or a geometry column from a registered view.

### Output
Returns `boolean`: `true` if the geometry is a ring (closed and simple), `false` otherwise.

### Valid Call Patterns
```python
# Example using sd.sql to verify a ring geometry
query = "SELECT ST_IsRing(ST_GeomFromText('LINESTRING(0 0, 0 1, 1 1, 1 0, 0 0)')) as is_ring"
result = sd.sql(query).to_arrow_table()
assert result["is_ring"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_IsRing(geometry_expression)` within a `sd.sql()` query.
- Ensure the input is a valid geometry type; if providing WKT, wrap it in `ST_GeomFromText`.

### Prompt Snippet
```text
ST_IsRing(geometry_column)
```

### Common Failure Modes
- Passing a non-linestring geometry (e.g., `Point` or `Polygon`), which may result in unexpected behavior or errors depending on the underlying engine implementation.
- Passing a self-intersecting `LineString` (e.g., a figure-eight), which will return `false` because it is not simple.
- Passing an unclosed `LineString`, which will return `false` because it is not closed.

### Fix Code Hint
```python
# Ensure the geometry is a closed, simple LineString
# If the geometry is self-intersecting, use ST_MakeValid or simplify the geometry first.
query = "SELECT ST_IsRing(my_geom_col) FROM my_table"
```

## API Test: `ST_IsSimple`

### Signature
```python
ST_IsSimple(arg1: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_issimple.qmd:19_

## Description

Returns `true` if the geometry has no anomalous geometric points such as
self-intersections or self-tangency.

## Examples

```sql
SELECT ST_IsSimple(
  ST_GeomFromWKT('POLYGON((1 1, 3 1, 3 3, 1 3, 1 1))')
);
```

### Goal
Determines if a given geometry is "simple" by verifying that its only self-intersections occur at boundary points.

### Parameters
- `arg1` (`geometry`): The input geometry object (e.g., Point, LineString, Polygon) to be evaluated for self-intersections.

### Input
- **Data:** A valid geometry object.
- **Formats:** Accepts geometries created via WKT, WKB, or other SedonaDB-supported geometry constructors.
- **Preconditions:** The input must be a valid geometry type.

### Output
Returns `boolean` — `true` if the geometry is simple (no self-intersections or self-tangency), `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify a simple polygon
query = "SELECT ST_IsSimple(ST_GeomFromWKT('POLYGON((1 1, 3 1, 3 3, 1 3, 1 1))')) as is_simple"
result = sd.sql(query).to_arrow_table()

assert result["is_simple"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_IsSimple` to validate the topological integrity of vector geometries.
- Always pass a valid geometry object as the argument.
- The function returns a boolean; use it in `SELECT` clauses or `WHERE` filters to identify non-simple geometries.

### Prompt Snippet
```text
ST_IsSimple(geometry_column)
```

### Common Failure Modes
- **Invalid Geometry:** Passing a null or non-geometry type will result in a SQL execution error.
- **Complex Geometries:** Geometries like "bow-tie" polygons or self-intersecting lines will correctly return `false`, which may be unexpected if the user assumes all valid WKT inputs are simple.

### Fix Code Hint
```python
# If a geometry is not simple, use ST_MakeValid to attempt to fix it
query = "SELECT ST_IsSimple(ST_MakeValid(geom)) FROM my_table"
```

## API Test: `ST_IsValid`

### Signature
```python
ST_IsValid(arg1: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_isvalid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsValid
description: Checks whether a geometry meets the rules of a valid spatial object according to the OGC standard.
kernels:
  - returns: boolean
    args: [geometry]
---

## Description

Returns `true` if the geometry is well-formed according to OGC rules. Use
`ST_IsValidReason` to get a diagnostic message for invalid geometries and
`ST_MakeValid` to attempt to repair them.

## Examples

```sql
SELECT ST_IsValid(
    ST_GeomFromText('POLYGON ((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))')
) AS is_valid;
```


### Goal
Determines if a given geometry object conforms to OGC standards for well-formed spatial data.

### Parameters
- `arg1` (`geometry`): The geometry object to validate (e.g., Point, Polygon, LineString).

### Input
The function accepts any valid geometry type. Input geometries are typically constructed via WKT (e.g., `ST_GeomFromText`) or loaded from supported vector formats like GeoParquet.

### Output
Returns `boolean`: `true` if the geometry is well-formed according to OGC rules, otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify geometry validity
query = "SELECT ST_IsValid(ST_GeomFromText('POLYGON ((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))')) as is_valid"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
When using `ST_IsValid`, ensure the input is a valid geometry object. If the geometry is invalid, do not attempt to perform spatial operations that require validity; instead, suggest using `ST_MakeValid` to repair the geometry or `ST_IsValidReason` to diagnose the issue.

### Prompt Snippet
```text
ST_IsValid(geometry_column)
```

### Common Failure Modes
- **Invalid Geometry Input:** Passing a non-geometry type or a null value will result in an error.
- **Self-Intersecting Polygons:** Complex polygons that self-intersect are often invalid; `ST_IsValid` will return `false` for these, requiring a call to `ST_MakeValid` before further processing.

### Fix Code Hint
```python
# If ST_IsValid returns false, repair the geometry:
query = "SELECT ST_MakeValid(invalid_geom_col) FROM my_table"
```

## API Test: `ST_IsValidReason`

### Signature
```python
ST_IsValidReason(arg1: geometry) -> string
```
_Source: source/docs/reference/sql/st_isvalidreason.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_IsValidReason
description: Returns a text explanation describing why a geometry is invalid.
kernels:
  - returns: string
    args: [geometry]
---

## Description

Returns a text string explaining why a geometry is invalid, or `"Valid Geometry"` if the geometry is valid. Useful for debugging geometry construction.

## Examples

```sql
SELECT ST_IsValidReason(
    ST_GeomFromText('POLYGON((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))')
) AS reason;
```


### Goal
`ST_IsValidReason` provides diagnostic feedback by returning a descriptive string explaining the topological invalidity of a geometry object, or confirming it is valid.

### Parameters
- `arg1` (`geometry`): The geometry object to be validated.

### Input
- **Data:** A geometry object (e.g., Point, LineString, Polygon) created via constructors like `ST_GeomFromText` or loaded from supported vector formats like GeoParquet.
- **Preconditions:** The input must be a valid geometry type recognized by SedonaDB.

### Output
Returns `string` — A human-readable explanation of the geometry's invalidity, or the literal string `"Valid Geometry"` if the input satisfies topological requirements.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using a self-intersecting polygon
query = "SELECT ST_IsValidReason(ST_GeomFromText('POLYGON((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))')) as reason"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always use `ST_IsValidReason` to diagnose geometry issues when spatial operations fail or return unexpected results.
- Ensure the input is a valid geometry type; if the input is a string, wrap it in a constructor like `ST_GeomFromText`.

### Prompt Snippet
```text
ST_IsValidReason(geometry_column)
```

### Common Failure Modes
- **Null Input:** Passing a `NULL` geometry will result in a `NULL` output rather than a diagnostic string.
- **Unsupported Types:** Passing non-geometry types (e.g., raw raster objects) will result in a SQL execution error.

### Fix Code Hint
```python
# If the geometry is invalid, use ST_MakeValid to attempt a repair
query = "SELECT ST_MakeValid(geom) FROM table WHERE ST_IsValidReason(geom) != 'Valid Geometry'"
```

## API Test: `ST_KNN`

### Signature
```python
ST_KNN(geomA: geometry, geomB: geometry, k: integer, use_spheroid: boolean) -> boolean | ST_KNN(geomA: geometry, geomB: geometry, k: integer) -> boolean | ST_KNN(geomA: geometry, geomB: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_knn.qmd:19_

## Description

`ST_KNN` is used in a join condition to find the k nearest neighbors of one geometry from another set of geometries. The actual k-nearest neighbors logic is handled by the spatial join execution engine: the function itself is a stub that simply defines the expected signature.

## Examples

```sql
WITH table2 AS (
    SELECT geom1 FROM (VALUES
        (ST_Point(0, 1)),
        (ST_Point(2, 3)),
        (ST_Point(4, 5)),
        (ST_Point(6, 7)),
        (ST_Point(8, 9))
    ) AS t(geom1)
),
table1 AS (
    SELECT ST_Point(0, 0) AS geom2
)
SELECT * FROM table1 JOIN table2 ON ST_KNN(geom1, geom2, 3);
```

### Goal
`ST_KNN` acts as a join predicate to identify the `k` nearest neighbor geometries from a candidate set (`geomB`) relative to a reference geometry (`geomA`).

### Parameters
- `geomA` (`geometry`): The reference geometry around which to search for neighbors.
- `geomB` (`geometry`): The column containing candidate geometries to evaluate for proximity.
- `k` (`integer`): The number of nearest neighbors to return. Defaults to 1.
- `use_spheroid` (`boolean`): If `true`, uses spherical distance; if `false`, uses Euclidean distance. Defaults to `false`.

### Input
- **Data:** Requires two sets of geometries (typically provided via a SQL `JOIN` operation).
- **Preconditions:** The function is designed to be used exclusively within a SQL `JOIN` condition. It acts as a stub for the spatial join execution engine.

### Output
Returns `boolean` — Returns `true` if the geometry from the candidate set is among the `k` nearest neighbors of the reference geometry.

### Valid Call Patterns
```python
# Example of calling ST_KNN via the Python SQL interface
query = """
SELECT * FROM table1 
JOIN table2 ON ST_KNN(table2.geom, table1.geom, 3)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_KNN` within a `JOIN ... ON` clause.
- Do not attempt to use `ST_KNN` as a standalone scalar function in a `SELECT` list, as it is a stub for the join engine.
- Ensure `k` is a positive integer.

### Prompt Snippet
```text
Use ST_KNN(geomA, geomB, k) inside a SQL JOIN ON clause to filter for the k-nearest neighbors.
```

### Common Failure Modes
- **Standalone Usage:** Calling `ST_KNN` outside of a `JOIN` condition will fail as it is a stub for the spatial join engine.
- **Missing Join Context:** Attempting to use the function without a valid join relationship between two tables.
- **Type Mismatch:** Providing non-geometry types to `geomA` or `geomB`.

### Fix Code Hint
```python
# Ensure the function is part of the JOIN condition
sd.sql("SELECT * FROM A JOIN B ON ST_KNN(A.geom, B.geom, 5)").to_arrow_table()
```

## API Test: `ST_Length`

### Signature
```python
ST_Length(arg1: geometry) -> double | ST_Length(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_length.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Length
description: Returns the length of a geometry or geography.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

Returns the length of geom. This function only supports LineString, MultiLineString, and GeometryCollections containing linear geometries. Use ST_Perimeter for polygons. For geometry, returns the length in the units of the coordinate reference system; for geography, returns the spherical approximation of the geodesic length in meters.

## Examples

```sql
SELECT ST_Length(ST_GeomFromWKT('LINESTRING (0 0, 0 1)'));
```

```sql
SELECT ST_Length(ST_GeogFromWKT('LINESTRING (0 0, 0 1)'));
```


### Goal
Calculates the linear length of a geometry or the geodesic length of a geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object. Must be a `LineString`, `MultiLineString`, or a `GeometryCollection` containing linear geometries.

### Input
- **Data Types:** Supports `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** Input must be a linear geometry type. Polygons are not supported; use `ST_Perimeter` for polygons.
- **Units:** For `geometry`, the result is in the units of the coordinate reference system (CRS). For `geography`, the result is in meters.

### Output
Returns `double` — The calculated length of the input geometry or geography.

### Valid Call Patterns
```python
# Inferred from signature and documentation
result = sd.sql("SELECT ST_Length(ST_GeomFromWKT('LINESTRING (0 0, 0 1)'))").to_arrow_table()
print(result)
# __CHECK__ witness: 1.0
```

### LLM Instruction Prompt
- Use `ST_Length` only for `LineString`, `MultiLineString`, or `GeometryCollection` containing linear geometries.
- If the input is a `Polygon`, suggest `ST_Perimeter` instead.
- Specify whether the input is `geometry` (planar) or `geography` (spherical) to ensure the correct unit of measurement (CRS units vs. meters).

### Prompt Snippet
```text
ST_Length(geometry_column)
```

### Common Failure Modes
- **Unsupported Geometry Type:** Passing a `Polygon` or `Point` will result in an error or unexpected behavior; ensure the input is linear.
- **CRS Mismatch:** For `geometry` types, the length is returned in the units of the CRS; ensure the CRS is appropriate for the desired unit of measurement.
- **Null Inputs:** Passing a null geometry will return null.

### Fix Code Hint
```python
# If input is a polygon, use ST_Perimeter
sd.sql("SELECT ST_Perimeter(my_polygon_geom) FROM my_table").to_arrow_table()

# If input is geometry and you need meters, ensure it is cast to geography
sd.sql("SELECT ST_Length(ST_GeographyFromText('LINESTRING (0 0, 0 1)'))").to_arrow_table()
```

## API Test: `ST_LineFromText`

### Signature
```python
ST_LineFromText(wkt: string) -> geometry | ST_LineFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_linefromtext.qmd:19_

### Goal
Constructs a `LineString` geometry object from a Well-Known Text (WKT) string representation.

### Parameters
- `wkt` (`string`): A valid WKT string representing a LineString (e.g., `'LINESTRING (0 0, 1 1)'`).
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to associate with the geometry.

### Input
- **Data:** A string literal or column containing WKT-formatted geometry data.
- **Preconditions:** The input string must strictly represent a `LineString`. If the WKT represents a different geometry type (e.g., `POINT` or `POLYGON`), the function will raise an error.

### Output
Returns a `geometry` object representing the parsed LineString, optionally tagged with the provided SRID.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)')").to_arrow_table()
print("__CHECK__", result)

# Construction with SRID
result_srid = sd.sql("SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)', 4326)").to_arrow_table()
print("__CHECK__", result_srid)
```

### LLM Instruction Prompt
- Use `ST_LineFromText` when converting raw WKT strings into SedonaDB geometry types.
- Always verify that the input string is a valid `LINESTRING` WKT; otherwise, the query will fail.
- Use the alias `ST_LineStringFromText` if preferred for readability.

### Prompt Snippet
```text
ST_LineFromText(wkt_string)
ST_LineFromText(wkt_string, srid_integer)
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `LINESTRING` (e.g., `POINT(0 0)`) will cause the function to error.
- **Malformed WKT:** Passing a string that does not conform to the WKT specification will result in a parsing error.
- **CRS Mismatch:** If using the resulting geometry in subsequent operations (like `RS_Clip` or spatial joins) with other geometries, ensure the SRIDs match, as SedonaDB requires consistent CRS for spatial operations.

### Fix Code Hint
```python
# Ensure the input is a valid LineString WKT
# If you have a Point, use ST_Point instead
query = "SELECT ST_LineFromText('LINESTRING (0 0, 1 1)')"
```

## API Test: `ST_LineInterpolatePoint`

### Signature
```python
ST_LineInterpolatePoint(arg1: geometry, fraction: double) -> geometry | ST_LineInterpolatePoint(arg1: geography, fraction: double) -> geography
```
_Source: source/docs/reference/sql/st_lineinterpolatepoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_LineInterpolatePoint
description: Returns a point interpolated along a line.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: fraction
      type: double
  - returns: geography
    args:
    - geography
    - name: fraction
      type: double
---

## Description

Returns a point interpolated along a line. First argument must be a LINESTRING. Second argument is a double between 0 and 1 representing fraction of total linestring length the point has to be located. For geography, the interpolation is computed using spherical interpolation between vertices.

## Examples

```sql
SELECT ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (0 0, 1 1)'), 0.5);
```

```sql
SELECT ST_LineInterpolatePoint(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'), 0.5);
```


### Goal
Calculates a point at a specific fractional distance along a provided LINESTRING geometry or geography.

### Parameters
- `arg1` (`geometry`): The input LINESTRING geometry or geography object.
- `fraction` (`double`): A value between 0.0 and 1.0 representing the percentage of the total line length where the point should be placed.

### Input
The input must be a valid LINESTRING. If using `geometry`, the input is typically constructed via `ST_GeomFromWKT` or similar constructors. If using `geography`, the input is typically constructed via `ST_GeogFromWKT`.

### Output
Returns a `geometry` (Point) or `geography` (Point) representing the interpolated location.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (0 0, 1 1)'), 0.5)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure the first argument is a LINESTRING.
- Ensure the `fraction` parameter is a double between 0 and 1.
- Use `ST_GeomFromWKT` for geometry inputs and `ST_GeogFromWKT` for geography inputs to ensure correct type handling.

### Prompt Snippet
```text
ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (x1 y1, x2 y2)'), fraction)
```

### Common Failure Modes
- Providing a geometry type other than LINESTRING (e.g., Polygon or Point), which will result in an error.
- Providing a `fraction` value outside the [0, 1] range.
- Mismatched types: attempting to pass a `geography` object to a function expecting `geometry` without explicit conversion.

### Fix Code Hint
```python
# Ensure input is a LINESTRING and fraction is within [0, 1]
query = "SELECT ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (0 0, 10 10)'), 0.5)"
result = sd.sql(query).to_arrow_table()
```

## API Test: `ST_LineLocatePoint`

### Signature
```python
ST_LineLocatePoint(arg1: geometry, arg2: geometry) -> double | ST_LineLocatePoint(arg1: geography, arg2: geography) -> double
```
_Source: source/docs/reference/sql/st_linelocatepoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_LineLocatePoint
description: Returns the location of the closest point on a LineString as a fraction of its total length.
kernels:
  - returns: double
    args: [geometry, geometry]
  - returns: double
    args: [geography, geography]
---

## Description

Returns a double between 0 and 1, representing the location of the closest point on the LineString as a fraction of its total 2D length. The first argument must be a LineString. For geometry, the fraction is computed using 2D Cartesian distance. For geography, the fraction is computed based on spherical interpolation between vertices.

## Examples

```sql
SELECT ST_LineLocatePoint(
    ST_GeomFromText('LINESTRING (0 0, 0 1)'),
    ST_GeomFromText('POINT (0 0.5)')
);
```

```sql
SELECT ST_LineLocatePoint(
    ST_GeogFromWKT('LINESTRING (0 0, 1 1, 2 2)'),
    ST_GeogFromWKT('POINT (1 1)')
);
```


### Goal
Calculates the normalized distance (0.0 to 1.0) along a LineString to the point closest to the provided input geometry or geography.

### Parameters
- `arg1` (`geometry`): The reference LineString geometry or geography.
- `arg2` (`geometry`): The target point geometry or geography to locate on the line.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** The first argument (`arg1`) must be a `LineString`. Both arguments must share the same spatial type (both geometry or both geography).

### Output
Returns `double` — A value between 0.0 and 1.0 representing the fraction of the total 2D length of the LineString where the closest point is located.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = "SELECT ST_LineLocatePoint(ST_GeomFromText('LINESTRING (0 0, 0 1)'), ST_GeomFromText('POINT (0 0.5)')) as fraction"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: fraction=0.5
```

### LLM Instruction Prompt
- Always ensure the first argument is a `LineString`.
- Ensure both arguments are of the same type (geometry or geography).
- Use `ST_GeomFromText` or `ST_GeogFromWKT` to construct inputs if they are provided as WKT strings.

### Prompt Snippet
```text
ST_LineLocatePoint(ST_GeomFromText('LINESTRING(...)'), ST_GeomFromText('POINT(...)'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as the first argument and a `geography` as the second will result in an error.
- **Invalid Geometry:** Passing a non-LineString geometry as the first argument.
- **CRS Mismatch:** If using geography, ensure the coordinate systems are compatible as per SedonaDB's strict CRS propagation rules.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type
sd.sql("SELECT ST_LineLocatePoint(ST_GeomFromText('LINESTRING (0 0, 0 1)'), ST_GeomFromText('POINT (0 0.5)'))")
```

## API Test: `ST_LineMerge`

### Signature
```python
ST_LineMerge(arg1: geometry) -> geometry | ST_LineMerge(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_linemerge.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_LineMerge
description: Merges a collection of potentially connected line segments into the fewest possible LineStrings.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Examples


```sql
SELECT ST_LineMerge(
    ST_GeomFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))')
);
```

```sql
SELECT ST_GeogFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))')
);
```


### Goal
`ST_LineMerge` merges a collection of connected line segments (such as a `MultiLineString`) into the fewest possible `LineString` geometries.

### Parameters
- `arg1` (`geometry` or `geography`): A collection of line segments, typically provided as a `MultiLineString` or `GeometryCollection`.

### Input
- **Data Formats:** Accepts WKT, WKB, or GeoParquet-loaded geometries/geographies.
- **Preconditions:** The input must contain connected line segments. If segments do not share endpoints, they will remain separate in the output.

### Output
Returns `geography | geometry` — A `LineString` or `MultiLineString` representing the merged segments.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Merging geometry segments
query = "SELECT ST_LineMerge(ST_GeomFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))')) as merged"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a single LineString geometry.
```

### LLM Instruction Prompt
- Use `ST_LineMerge` when you need to simplify a fragmented `MultiLineString` into continuous paths.
- Ensure the input type (geometry vs geography) matches the coordinate system of your data.
- Always wrap the input in a constructor like `ST_GeomFromWKT` if passing raw strings.

### Prompt Snippet
```text
ST_LineMerge(ST_GeomFromWKT('MULTILINESTRING ((...))'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` type when the session expects `geography` (or vice versa) without explicit conversion.
- **Disconnected Segments:** If the input segments do not share endpoints, `ST_LineMerge` will return the original input unchanged, as no merging is possible.
- **Invalid Geometry:** Passing non-line geometries (e.g., `Point` or `Polygon`) may result in an error or unexpected behavior depending on the underlying engine implementation.

### Fix Code Hint
```python
# Ensure input is a valid MultiLineString before merging
query = "SELECT ST_LineMerge(ST_GeomFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))'))"
```

## API Test: `ST_LineStringFromWKB`

### Signature
```python
ST_LineStringFromWKB(wkb: binary) -> geometry | ST_LineStringFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_linestringfromwkb.qmd:19_

### Goal
Constructs a `LineString` geometry object from a Well-Known Binary (WKB) representation, optionally assigning a specific Coordinate Reference System (CRS).

### Parameters
- `wkb` (`binary`): The binary representation of the geometry.
- `srid` (`crs`): An optional integer or identifier representing the Coordinate Reference System to assign to the resulting geometry.

### Input
- **Data:** A `binary` value representing a WKB-encoded geometry.
- **Preconditions:** The input binary must represent a `LineString` geometry type. If the WKB represents a different geometry type (e.g., `Point` or `Polygon`), the function returns `NULL`.

### Output
Returns `geometry` — A SedonaDB geometry object representing the parsed `LineString`.

### Valid Call Patterns
```python
# Inferred from SQL examples provided in documentation
import sedona.db
sd = sedona.db.connect()

# Basic construction
query = "SELECT ST_LineStringFromWKB(decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex'))"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")

# Construction with SRID
query_srid = "SELECT ST_LineStringFromWKB(decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex'), 4326)"
result_srid = sd.sql(query_srid).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_LineStringFromWKB` when converting raw binary WKB data into usable geometry objects.
- Always use the `decode(..., 'hex')` function if the input is a hex-encoded string.
- If the geometry type is unknown, verify the input is a `LineString` to avoid `NULL` results.

### Prompt Snippet
```text
ST_LineStringFromWKB(decode('...', 'hex'), [srid])
```

### Common Failure Modes
- **Type Mismatch:** Providing WKB that represents a `Point` or `Polygon` will result in a `NULL` return value rather than an error.
- **Invalid Binary:** Providing malformed binary data that does not conform to the WKB specification will cause the query to fail.
- **CRS Mismatch:** If performing subsequent spatial operations (like `RS_Clip` or spatial joins) with the resulting geometry, ensure the `srid` provided matches the CRS of other datasets in the operation to avoid runtime errors.

### Fix Code Hint
```python
# Ensure the input is a LineString WKB
# If result is NULL, verify the input geometry type
result = sd.sql("SELECT ST_LineStringFromWKB(wkb_col) FROM my_table").to_arrow_table()
```

## API Test: `ST_LineSubstring`

### Signature
```python
ST_LineSubstring(arg1: geometry, start_fraction: double, end_fraction: double) -> geometry
```
_Source: source/docs/reference/sql/st_linesubstring.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_LineSubstring
description: Returns a linestring being a substring of the input one starting and ending at the given fractions of total 2d length.
kernels:
  - returns: geometry
    args:
      - geometry
      - name: start_fraction
        type: double
        description: The fraction from which the returned line will start (between 0 and 1)
      - name: end_fraction
        type: double
        description: The fraction from which the returned line will end (between 0 and 1)
---

## Examples

```sql
SELECT ST_AsText(
  ST_LineSubstring(
    ST_GeomFromWKT('LINESTRING(0 0, 10 10)'),
    0.3,
    0.6
  )
);
```


### Goal
Extracts a segment of a `LineString` geometry based on specified start and end fractions of its total 2D length.

### Parameters
- `arg1` (`geometry`): The input `LineString` geometry to be sliced.
- `start_fraction` (`double`): The starting point of the substring, expressed as a fraction of the total 2D length (must be between 0 and 1).
- `end_fraction` (`double`): The ending point of the substring, expressed as a fraction of the total 2D length (must be between 0 and 1).

### Input
The function accepts a `geometry` type, typically created via `ST_GeomFromWKT` or loaded from a GeoParquet source. The input must be a valid `LineString`.

### Output
Returns a `geometry` object representing the portion of the original `LineString` between the provided fractions.

### Valid Call Patterns
```python
# Inferred from SQL example
query = """
SELECT ST_AsText(
  ST_LineSubstring(
    ST_GeomFromWKT('LINESTRING(0 0, 10 10)'),
    0.3,
    0.6
  )
)
"""
result = sd.sql(query).to_arrow_table()
# __CHECK__ witness: result contains the WKT of the substring
```

### LLM Instruction Prompt
- Always ensure `start_fraction` and `end_fraction` are `double` values between 0.0 and 1.0.
- Ensure the input geometry is a `LineString`.
- Use `ST_AsText` if you need to verify the resulting geometry coordinates in a human-readable format.

### Prompt Snippet
```text
ST_LineSubstring(geometry_column, start_frac, end_frac)
```

### Common Failure Modes
- Providing fractions outside the [0, 1] range, which may result in errors or undefined behavior.
- Passing a non-linestring geometry type (e.g., `Point` or `Polygon`) as `arg1`.
- Mismatched CRS if the input geometry is part of a larger spatial join or transformation pipeline.

### Fix Code Hint
```python
# Ensure fractions are normalized to [0, 1]
# Example: 30% to 60% of the line
result = sd.sql("SELECT ST_LineSubstring(geom, 0.3, 0.6) FROM my_lines").to_arrow_table()
```

## API Test: `ST_M`

### Signature
```python
ST_M(arg1: geometry) -> double | ST_M(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_m.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_M
description: Returns the M (measure) coordinate of a Point geometry.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

Extracts the M (measure) coordinate from a Point geometry or geography. Returns
NULL if the geometry has no M dimension or for non-point geometries.

## Examples

```sql
SELECT ST_M(ST_PointM(1.0, 2.0, 42.0));
```

```sql
SELECT ST_M(ST_Point(1.0, 2.0));
```


### Goal
Extracts the M (measure) coordinate value from a Point geometry or geography object.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object from which to extract the measure coordinate.

### Input
The function accepts a `geometry` or `geography` type. The input must be a Point geometry that contains an M dimension. If the input is a non-point geometry or lacks an M dimension, the function will return `NULL`.

### Output
Returns `double` — The M (measure) coordinate value of the point, or `NULL` if the dimension is missing or the geometry type is not a Point.

### Valid Call Patterns
```python
# Inferred from SQL examples provided in documentation
result = sd.sql("SELECT ST_M(ST_PointM(1.0, 2.0, 42.0))").to_arrow_table()
print(result)
# __CHECK__ witness: 42.0
```

### LLM Instruction Prompt
- Use `ST_M` only when the input is a Point geometry or geography.
- If the input geometry is not guaranteed to have an M dimension, ensure the downstream logic handles potential `NULL` return values.

### Prompt Snippet
```text
ST_M(geometry_column)
```

### Common Failure Modes
- Passing a non-point geometry (e.g., LineString or Polygon) will result in a `NULL` return value.
- Passing a Point geometry that was created without an M dimension (e.g., using `ST_Point` instead of `ST_PointM`) will result in a `NULL` return value.

### Fix Code Hint
```python
# Ensure the geometry is constructed with an M dimension if you expect a non-null result
sd.sql("SELECT ST_M(ST_PointM(x, y, m)) FROM table")
```

## API Test: `ST_MLineFromText`

### Signature
```python
ST_MLineFromText(wkt: string) -> geometry | ST_MLineFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_mlinefromtext.qmd:19_

### Goal
Constructs a `MultiLineString` geometry object from a provided Well-Known Text (WKT) string representation.

### Parameters
- `wkt` (`string`): A string containing the Well-Known Text representation of a MultiLineString.
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to assign to the resulting geometry.

### Input
- **Data:** A valid WKT string starting with `MULTILINESTRING`.
- **Preconditions:** The input string must strictly represent a `MultiLineString`. If the WKT describes a different geometry type (e.g., `POINT` or `POLYGON`), the function will raise an error.

### Output
Returns `geometry` — A materialized geometry object representing the MultiLineString, optionally tagged with the provided SRID.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))')").to_arrow_table()
print("__CHECK__")

# Construction with SRID
result_srid = sd.sql("SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))', 4326)").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_MLineFromText` when converting raw WKT strings into SedonaDB geometry objects.
- Always ensure the input string is a valid `MULTILINESTRING` WKT.
- If an SRID is known, provide it as the second argument to ensure correct spatial reference handling.

### Prompt Snippet
```text
ST_MLineFromText(wkt_string, [srid_integer])
```

### Common Failure Modes
- **Invalid WKT:** Providing a WKT string that does not represent a `MultiLineString` (e.g., `ST_MLineFromText('POINT(0 0)')`) will cause the query to fail.
- **Malformed WKT:** Providing a syntactically incorrect WKT string will result in a parsing error.
- **SRID Mismatch:** If the resulting geometry is used in subsequent operations (like `RS_Clip` or spatial joins) with other geometries, ensure the SRIDs match, as SedonaDB requires consistent CRS for spatial operations.

### Fix Code Hint
```python
# Ensure the WKT string is correctly formatted for a MultiLineString
# Example: 'MULTILINESTRING ((x1 y1, x2 y2), (x3 y3, x4 y4))'
query = "SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))', 4326)"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_MMax`

### Signature
```python
ST_MMax(arg1: geometry) -> double | ST_MMax(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_mmax.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MMax
description: Returns the maximum M (measure) value from a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_MMax(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'));
```

```sql
SELECT ST_MMax(ST_GeogFromWKT('LINESTRING M (1 2 3, 5 6 7)'));
```


### Goal
`ST_MMax` extracts the maximum M (measure) coordinate value present within the bounding box of a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (geometry or geography) from which to extract the maximum M-coordinate.

### Input
The input must be a valid geometry or geography object containing M-coordinates (e.g., `LINESTRING M`, `POINT M`). The function operates on the bounding box of the provided spatial object.

### Output
Returns `double` — The maximum M-coordinate value found in the input object's bounding box.

### Valid Call Patterns
```python
# Example inferred from signature and SQL documentation
import sedona.db
sd = sedona.db.connect()

# Verify M-coordinate extraction
result = sd.sql("SELECT ST_MMax(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)')) as m_max").to_arrow_table()
assert result["m_max"][0] == 7.0
print(f"__CHECK__ witness: {result['m_max'][0]}")
```

### LLM Instruction Prompt
- Use `ST_MMax` to retrieve the upper bound of the M-dimension for geometries or geographies.
- Ensure the input object is constructed with M-coordinates (e.g., using `ST_GeomFromWKT` with M-enabled WKT).
- The function returns a `double` type; handle potential nulls if the input geometry lacks M-coordinates.

### Prompt Snippet
```text
ST_MMax(geometry_column)
```

### Common Failure Modes
- **Missing M-coordinates:** If the input geometry does not contain M-coordinates, the function may return `null` or an unexpected value depending on the underlying engine implementation.
- **Invalid Geometry:** Passing a non-spatial type or a malformed WKT string will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the geometry is M-enabled before calling ST_MMax
sd.sql("SELECT ST_MMax(ST_GeomFromWKT('LINESTRING M (0 0 0, 1 1 10)'))").to_arrow_table()
```

## API Test: `ST_MMin`

### Signature
```python
ST_MMin(arg1: geometry) -> double | ST_MMin(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_mmin.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MMin
description: Returns the minimum M-coordinate (measure) of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_MMin(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'));
```

```sql
SELECT ST_MMin(ST_GeogFromWKT('LINESTRING M (1 2 3, 5 6 7)'));
```


### Goal
Returns the minimum M-coordinate (measure) value found within the bounding box of a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (geometry or geography) containing M-coordinates (measures).

### Input
- **Data:** Geometry or geography objects that support M-coordinates (e.g., `LINESTRING M`, `POINT M`).
- **Preconditions:** The input object must contain M-coordinates; if the geometry lacks a measure dimension, the behavior is undefined or may return null.

### Output
Returns `double` — The minimum M-coordinate value present in the input object's bounding box.

### Valid Call Patterns
```python
# Example inferred from signature and SQL examples
result = sd.sql("SELECT ST_MMin(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_MMin` to extract the minimum measure value from geometries or geographies that include an M-dimension.
- Ensure the input is a valid geometry or geography object created via SedonaDB constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`.

### Prompt Snippet
```text
ST_MMin(geometry_column)
```

### Common Failure Modes
- **Missing M-dimension:** Passing a standard 2D geometry (X, Y) or 3D geometry (X, Y, Z) that lacks the M-coordinate will result in an error or null return.
- **Invalid Input Type:** Passing non-spatial data types or unsupported formats will cause a query execution failure.

### Fix Code Hint
```python
# Ensure the geometry has M-coordinates before calling ST_MMin
sd.sql("SELECT ST_MMin(ST_ForceM(geom)) FROM my_table")
```

## API Test: `ST_MPointFromText`

### Signature
```python
ST_MPointFromText(wkt: string) -> geometry | ST_MPointFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_mpointfromtext.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MPointFromText
description: Constructs a MultiPoint from Well-Known Text (WKT), erroring if the input is not a MultiPoint.
kernels:
  - returns: geometry
    args:
    - name: wkt
      type: string
  - returns: geometry
    args:
    - name: wkt
      type: string
    - name: srid
      type: crs
---

## Description

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a MultiPoint.

## Examples

```sql
SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))');
```

With an SRID or CRS:

```sql
SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))', 4326);
```


### Goal
Constructs a `MultiPoint` geometry object from a provided Well-Known Text (WKT) string for use in spatial analysis.

### Parameters
- `wkt` (`string`): A string representation of a MultiPoint geometry (e.g., `'MULTIPOINT ((0 0), (1 1))'`).
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to associate with the geometry.

### Input
The input must be a valid WKT string specifically representing a `MULTIPOINT` geometry. If the string represents any other geometry type (e.g., `POINT`, `POLYGON`), the function will raise an error.

### Output
Returns a `geometry` object representing the MultiPoint, which can be used in subsequent spatial operations like `ST_Intersects` or `ST_Contains`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))') as geom").to_arrow_table()
print("__CHECK__")

# Construction with SRID
result_srid = sd.sql("SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))', 4326) as geom").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure the input string is a valid WKT `MULTIPOINT` format.
- If an SRID is provided, ensure it is a valid integer CRS identifier.
- Use this function when you need to materialize a MultiPoint geometry from a string literal in a SQL query.

### Prompt Snippet
```text
ST_MPointFromText(wkt_string, [srid_integer])
```

### Common Failure Modes
- **Invalid WKT:** Providing a WKT string that is not a `MULTIPOINT` (e.g., `POINT(0 0)`) will cause the function to error.
- **Malformed WKT:** Providing a string that does not conform to the WKT specification for MultiPoints will result in a parsing error.
- **CRS Mismatch:** If the resulting geometry is used in operations with other geometries that have a different, incompatible CRS, the operation may fail or produce incorrect results.

### Fix Code Hint
```python
# Ensure the WKT string is explicitly a MULTIPOINT
# Incorrect: ST_MPointFromText('POINT(0 0)')
# Correct: ST_MPointFromText('MULTIPOINT((0 0))')
```

## API Test: `ST_MPolyFromText`

### Signature
```python
ST_MPolyFromText(wkt: string) -> geometry | ST_MPolyFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_mpolyfromtext.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MPolyFromText
description: Constructs a MultiPolygon from Well-Known Text (WKT), erroring if the input is not a MultiPolygon.
kernels:
  - returns: geometry
    args:
    - name: wkt
      type: string
  - returns: geometry
    args:
    - name: wkt
      type: string
    - name: srid
      type: crs
---

### Goal
Constructs a `MultiPolygon` geometry object from a provided Well-Known Text (WKT) string representation.

### Parameters
- `wkt` (`string`): A valid WKT string representing a MultiPolygon geometry.
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to associate with the geometry.

### Input
- **Data:** A string containing valid WKT for a `MULTIPOLYGON`.
- **Preconditions:** The input string must strictly conform to the WKT format for a MultiPolygon. If the string represents a different geometry type (e.g., `POINT` or `POLYGON`), the function will raise an error.

### Output
Returns a `geometry` object representing the MultiPolygon defined by the input WKT.

### Valid Call Patterns
```python
# Basic construction
result = sd.sql("SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))')").to_arrow_table()
print("__CHECK__")

# Construction with SRID
result = sd.sql("SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))', 4326)").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_MPolyFromText` only when the input WKT is guaranteed to be a `MULTIPOLYGON`.
- If an SRID is known, always provide it as the second argument to ensure correct spatial context.
- Ensure the WKT string is properly escaped if used within a Python f-string or SQL query string.

### Prompt Snippet
```text
ST_MPolyFromText(wkt_string, [srid_integer])
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `MULTIPOLYGON` (e.g., `POINT(0 0)`) will cause the function to error.
- **Malformed WKT:** Providing a string that does not follow the WKT specification for MultiPolygons will result in a parsing error.
- **CRS Mismatch:** If the resulting geometry is used in subsequent operations (like `RS_Clip` or spatial joins) with other geometries or rasters, ensure the SRID matches the project's coordinate system requirements.

### Fix Code Hint
```python
# Ensure the input is a MultiPolygon WKT
wkt_input = "MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))"
query = f"SELECT ST_MPolyFromText('{wkt_input}', 4326)"
result = sd.sql(query).to_arrow_table()
```

## API Test: `ST_MakeEnvelope`

### Signature
```python
ST_MakeEnvelope(min_x: double, min_y: double, max_x: double, max_y: double, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_makeenvelope.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MakeEnvelope
description: Constructs a rectangular Polygon from minimum and maximum coordinates.
kernels:
  - returns: geometry
    args:
    - name: min_x
      type: double
    - name: min_y
      type: double
    - name: max_x
      type: double
    - name: max_y
      type: double
    - name: srid
      type: crs
---

## Description

Constructs a rectangular Polygon from minimum X, minimum Y, maximum X, and maximum Y coordinates. An optional SRID or CRS can be provided as a fifth argument.

## Examples

```sql
SELECT ST_MakeEnvelope(1.234, 2.234, 3.345, 3.345);
```

With an SRID:

```sql
SELECT ST_MakeEnvelope(1.234, 2.234, 3.345, 3.345, 4326);
```


### Goal
Constructs a rectangular `Polygon` geometry defined by the provided minimum and maximum coordinate bounds, optionally assigning a spatial reference system identifier (SRID).

### Parameters
- `min_x` (`double`): The minimum X-coordinate (longitude or easting) of the envelope.
- `min_y` (`double`): The minimum Y-coordinate (latitude or northing) of the envelope.
- `max_x` (`double`): The maximum X-coordinate (longitude or easting) of the envelope.
- `max_y` (`double`): The maximum Y-coordinate (latitude or northing) of the envelope.
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (e.g., 4326).

### Input
The function accepts four `double` values representing the bounding box coordinates. An optional fifth argument defines the SRID. Precondition: `min_x` must be less than `max_x` and `min_y` must be less than `max_y` to produce a valid non-degenerate polygon.

### Output
Returns a `geometry` object of type `Polygon` representing the rectangular area defined by the input bounds.

### Valid Call Patterns
```python
# Basic usage
result = sd.sql("SELECT ST_MakeEnvelope(0.0, 0.0, 10.0, 10.0) as geom").to_arrow_table()
print("__CHECK__")

# Usage with SRID
result = sd.sql("SELECT ST_MakeEnvelope(0.0, 0.0, 400.0, 400.0, 4326) as geom").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure `min_x < max_x` and `min_y < max_y`.
- If an SRID is provided, ensure it matches the coordinate system of other geometries used in subsequent spatial operations (e.g., `RS_Clip` or `RS_AsRaster`) to avoid CRS mismatch errors.

### Prompt Snippet
```text
ST_MakeEnvelope(min_x, min_y, max_x, max_y, [srid])
```

### Common Failure Modes
- **CRS Mismatch:** Providing an SRID that does not match the CRS of other geometries in a spatial join or raster operation will cause the operation to fail.
- **Invalid Bounds:** Providing `min_x >= max_x` or `min_y >= max_y` may result in an invalid geometry or an empty polygon.
- **Type Errors:** Passing non-numeric types to the coordinate arguments will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure coordinates are floats and bounds are valid
min_x, min_y, max_x, max_y = 0.0, 0.0, 10.0, 10.0
query = f"SELECT ST_MakeEnvelope({min_x}, {min_y}, {max_x}, {max_y}, 4326)"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_MakeLine`

### Signature
```python
ST_MakeLine(arg1: geometry, arg2: geometry) -> geometry | ST_MakeLine(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_makeline.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MakeLine
description: Creates a LineString from two or more input geometries or geographies.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Description

Creates a `LineString` from two or more input `Point`, `MultiPoint`, or `LineString` geometries or geographies. The function connects the inputs in the order they are provided to form a single continuous line. Combining geographies returns a geography.

## Examples

```sql
SELECT ST_MakeLine(ST_Point(0, 1), ST_Point(2, 3)) as geom;
```


### Goal
`ST_MakeLine` constructs a `LineString` geometry or geography by connecting two or more input spatial objects in the sequence they are provided.

### Parameters
- `arg1` (`geometry`): The starting point or geometry segment of the line.
- `arg2` (`geometry`): The ending point or geometry segment of the line.

### Input
The function accepts `Point`, `MultiPoint`, or `LineString` types. Inputs must share the same spatial type (geometry or geography). If using geographies, ensure the session context supports the coordinate reference system (CRS) of the inputs.

### Output
Returns `geography | geometry` — A `LineString` object representing the path formed by connecting the input geometries or geographies in order.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from SQL example
query = "SELECT ST_MakeLine(ST_Point(0, 1), ST_Point(2, 3)) as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure that both `arg1` and `arg2` are of the same spatial type (both geometry or both geography).
- Use `ST_MakeLine` to connect points or existing line segments into a single continuous `LineString`.

### Prompt Snippet
```text
Use ST_MakeLine(arg1, arg2) to create a LineString from two spatial inputs.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in an execution error.
- **CRS Mismatch:** If inputs have conflicting Coordinate Reference Systems, the operation may fail or produce invalid spatial results.
- **Invalid Geometry:** Providing non-spatial types or unsupported geometry types (e.g., `Raster`) as arguments will cause a query failure.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_MakeLine(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(1 1)'))").to_arrow_table()
```

## API Test: `ST_MakeValid`

### Signature
```python
ST_MakeValid(arg1: geometry) -> geometry | ST_MakeValid(arg1: geometry, keepCollapsed: boolean) -> geometry
```
_Source: source/docs/reference/sql/st_makevalid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MakeValid
description: Creates a valid representation of an invalid geometry.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args:
    - geometry
    - name: keepCollapsed
      type: boolean
---

## Description

Collapsed geometries are either converted to empty (keepCollapsed=false) or a valid geometry of lower dimension (keepCollapsed=true). Default is keepCollapsed=false.

## Examples

```sql
SELECT ST_MakeValid(ST_GeomFromWKT('LINESTRING(1 1, 1 1)'));
```


### Goal
`ST_MakeValid` repairs invalid geometries by applying geometric algorithms to produce a valid representation of the input.

### Parameters
- `arg1` (`geometry`): The input geometry that requires validation.
- `keepCollapsed` (`boolean`): Optional flag. If `true`, collapsed geometries are preserved as lower-dimension valid geometries; if `false` (default), they are converted to empty geometries.

### Input
- **Data:** Accepts standard geometry types (Point, LineString, Polygon, etc.) provided via WKT, WKB, or GeoParquet.
- **Preconditions:** The input must be a valid geometry object recognized by the SedonaDB SQL engine.

### Output
Returns `geometry` — A new, valid geometry object representing the repaired input.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic usage (inferred from signature and example)
result = sd.sql("SELECT ST_MakeValid(ST_GeomFromWKT('LINESTRING(1 1, 1 1)'))").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_MakeValid` when processing geometries that may contain self-intersections or invalid topology.
- If the geometry is expected to collapse (e.g., a line with zero length), explicitly set `keepCollapsed` to `true` if you wish to retain the lower-dimension result, otherwise rely on the default `false`.

### Prompt Snippet
```text
ST_MakeValid(geometry_column)
ST_MakeValid(geometry_column, true)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-geometry type (e.g., a raster or string that is not a valid WKT) will result in a SQL execution error.
- **Memory Constraints:** Attempting to repair extremely complex geometries with thousands of self-intersections may lead to high memory consumption on the single-node engine.

### Fix Code Hint
```python
# Ensure the input is cast to geometry if coming from a raw string
sd.sql("SELECT ST_MakeValid(ST_GeomFromWKT(my_wkt_col)) FROM my_table")
```

## API Test: `ST_MaxDistance`

### Signature
```python
ST_MaxDistance(arg1: geometry, arg2: geometry) -> double | ST_MaxDistance(arg1: geography, arg2: geography) -> double
```
_Source: source/docs/reference/sql/st_maxdistance.qmd:19_

### Goal
Calculates the maximum distance between any pair of points within two provided geometries or geographies.

### Parameters
- `arg1` (`geometry` | `geography`): The first input geometry or geography object.
- `arg2` (`geometry` | `geography`): The second input geometry or geography object.

### Input
- **Data Formats:** Accepts standard SedonaDB geometry types (WKT, WKB) or geography types.
- **Preconditions:** For geometry inputs, both must share the same coordinate reference system (CRS). For geography inputs, the function uses a spherical approximation to return the distance in meters.

### Output
Returns `double` — The maximum distance between the two inputs. For geometry, the value is in the units of the CRS; for geography, the value is in meters.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Geometry example
result = sd.sql("""
    SELECT ST_MaxDistance(
        ST_GeomFromText('POINT (0 0)'),
        ST_GeomFromText('LINESTRING (3 0, 0 4)')
    ) as max_dist
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_MaxDistance(arg1, arg2)` to compute the maximum distance.
- Ensure both arguments are of the same type (both geometry or both geography).
- If using geometry, ensure both inputs are in the same CRS.

### Prompt Snippet
```text
ST_MaxDistance(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different coordinate reference systems will result in an error.
- **Type Mismatch:** Attempting to compare a `geometry` type with a `geography` type is not supported.
- **Null Inputs:** Passing `NULL` as an argument will result in a `NULL` return value.

### Fix Code Hint
```python
# Ensure CRS alignment before calling ST_MaxDistance
sd.sql("SELECT ST_MaxDistance(ST_Transform(geom1, 4326), geom2) FROM table")
```

## API Test: `ST_MinimumClearance`

### Signature
```python
ST_MinimumClearance(arg1: geometry) -> double
```
_Source: source/docs/reference/sql/st_minimumclearance.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MinimumClearance
description: Returns the minimum clearance of a geometry.
kernels:
  - returns: double
    args: [geometry]
---

## Description

The minimum clearance is a metric that quantifies a geometry's tolerance to changes in coordinate precision or vertex positions. It represents the maximum distance by which vertices can be adjusted without introducing invalidity to the geometry's structure. A larger minimum clearance value indicates greater robustness against such perturbations.

For a geometry with a minimum clearance of x, the following conditions hold:

- No two distinct vertices are separated by a distance less than x.
- No vertex lies within a distance x from any line segment it is not an endpoint of.

For geometries with no definable minimum clearance, such as single Point geometries or MultiPoint geometries where all points occupy the same location, the function returns Double.MAX_VALUE.

## Examples

```sql
SELECT ST_MinimumClearance(
  ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))')
);
```


### Goal
Calculates the minimum clearance of a geometry to measure its structural robustness against vertex displacement or coordinate precision changes.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Polygon, LineString) for which the minimum clearance distance is to be calculated.

### Input
- **Data:** A valid geometry object.
- **Formats:** Supports geometries created via WKT, WKB, or loaded from GeoParquet.
- **Preconditions:** The input must be a valid geometry type.

### Output
Returns `double` — The minimum clearance value representing the maximum distance vertices can move without invalidating the geometry. Returns `Double.MAX_VALUE` for geometries with no definable clearance (e.g., single points).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and SQL example
query = "SELECT ST_MinimumClearance(ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_MinimumClearance` to evaluate the structural integrity of vector geometries.
- Always pass a valid geometry object as the argument.
- Note that for single points, the function will return `Double.MAX_VALUE`.

### Prompt Snippet
```text
ST_MinimumClearance(geometry_column)
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or string) will result in a SQL execution error.
- Passing an invalid or malformed WKT string to the geometry constructor before calling the function.

### Fix Code Hint
```python
# Ensure the input is a valid geometry
sd.sql("SELECT ST_MinimumClearance(ST_GeomFromWKT('POINT(0 0)'))").to_arrow_table()
```

## API Test: `ST_MinimumClearanceLine`

### Signature
```python
ST_MinimumClearanceLine(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_minimumclearanceline.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_MinimumClearanceLine
description: Returns a LineString representing the minimum clearance distance of the input geometry.
kernels:
  - returns: geometry
    args: [geometry]
---

## Description

This function returns a two-point LineString geometry representing the minimum clearance distance of the input geometry. If the input geometry does not have a defined minimum clearance, such as for single Points or coincident MultiPoints, an empty LineString geometry is returned instead.

## Examples

```sql
SELECT ST_MinimumClearanceLine(
  ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))')
);
```


### Goal
Calculates the minimum clearance distance of a geometry and returns it as a two-point LineString, identifying the shortest distance between two non-adjacent vertices or segments.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Polygon, LineString) for which the minimum clearance line is to be calculated.

### Input
- **Data:** Standard SedonaDB geometry types (WKT, WKB, or GeoParquet-loaded geometries).
- **Preconditions:** The input must be a valid geometry. If the geometry is a single Point or a MultiPoint with coincident points, the function will return an empty LineString.

### Output
Returns `geometry` — A `LineString` representing the minimum clearance distance. If no clearance is defined, it returns an empty `LineString`.

### Valid Call Patterns
```python
# Inferred from signature and SQL example
import sedona.db
sd = sedona.db.connect()
query = "SELECT ST_MinimumClearanceLine(ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))'))"
result = sd.sql(query).to_arrow_table()
print("__CHECK__", result)
```

### LLM Instruction Prompt
- Always use `ST_MinimumClearanceLine(geometry_expression)` as a standalone SQL function call.
- Ensure the input is a valid geometry type; if using WKT strings, wrap them in `ST_GeomFromWKT`.
- Do not expect a non-empty result for simple point geometries.

### Prompt Snippet
```text
Use ST_MinimumClearanceLine(geometry_col) to find the minimum clearance line of a geometry column in SedonaDB.
```

### Common Failure Modes
- **Empty Results:** Providing a single Point or coincident MultiPoints will result in an empty LineString, which may cause downstream errors if the application expects a non-empty geometry.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT strings will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure input is valid geometry
query = "SELECT ST_MinimumClearanceLine(ST_MakeValid(geom)) FROM table"
```

## API Test: `ST_NPoints`

### Signature
```python
ST_NPoints(arg1: geometry) -> integer | ST_NPoints(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_npoints.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_NPoints
description: Returns the number of points of the geometry or geography.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

## Description

Returns the total number of coordinate points in the geometry, counting all vertices across all components.

Aliases: `ST_NumPoints`.

## Examples

```sql
SELECT ST_NPoints(
  ST_GeomFromWKT('LINESTRING (1 2, 3 4, 5 6)')
);
```

```sql
SELECT ST_NPoints(
  ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
);
```

```sql
SELECT ST_NPoints(
  ST_GeogFromWKT('LINESTRING (1 2, 3 4, 5 6)')
);
```

```sql
SELECT ST_NPoints(
  ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
);
```


### Goal
Calculates the total count of coordinate vertices within a geometry or geography object, including all vertices across multi-part geometries.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the vertex count is required.

### Input
The function accepts standard geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`. No specific CRS preconditions are required as the function operates on the vertex structure.

### Output
Returns `integer` representing the total number of coordinate points found in the input object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify vertex count for a LineString
result = sd.sql("SELECT ST_NPoints(ST_GeomFromWKT('LINESTRING (1 2, 3 4, 5 6)')) as npoints").to_arrow_table()
assert result["npoints"][0] == 3
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_NPoints` to count vertices.
- Ensure the input is a valid geometry or geography object.
- Use `sd.sql(...).to_arrow_table()` for execution.

### Prompt Snippet
```text
ST_NPoints(geometry_expression)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) directly without a constructor like `ST_GeomFromWKT`.
- Expecting the function to return the number of components (e.g., number of polygons in a MultiPolygon) rather than the total vertex count.

### Fix Code Hint
```python
# Ensure input is cast to geometry if it is a raw WKT string
sd.sql("SELECT ST_NPoints(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `ST_NRings`

### Signature
```python
ST_NRings(arg1: geometry) -> integer | ST_NRings(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_nrings.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_NRings
description: Returns the total number of rings in a geometry or geography (both exterior and interior rings).
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

### Goal
Calculates the total count of exterior and interior rings within a polygon or multipolygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (typically a Polygon or MultiPolygon) to be analyzed.

### Input
- **Data:** Accepts standard WKT, WKB, or GeoParquet-derived geometry/geography types.
- **Preconditions:** The input must be a valid polygon-based geometry. If the input is a non-polygon type (e.g., Point, LineString), the function will return 0.

### Output
Returns `integer` — The total number of rings. For a simple polygon, this is 1 (exterior) plus the number of interior holes. For a multipolygon, it is the sum of all rings across all constituent polygons.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify ring count for a simple polygon
result = sd.sql("SELECT ST_NRings(ST_GeomFromText('POLYGON ((0 0, 0 1, 1 1, 1 0, 0 0))')) as n_rings").to_arrow_table()
assert result["n_rings"][0] == 1
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_NRings` to count rings in polygon geometries.
- Always pass the geometry or geography object as the single argument.
- Remember that non-polygon types return 0.

### Prompt Snippet
```text
ST_NRings(geometry_column)
```

### Common Failure Modes
- **Non-Polygon Input:** Passing a `Point` or `LineString` will not raise an error but will return `0`, which may be logically incorrect for the user's workflow.
- **Invalid WKT:** Passing malformed WKT strings to `ST_GeomFromText` before passing to `ST_NRings` will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure the input is a valid polygon before calling ST_NRings
sd.sql("SELECT ST_NRings(geom) FROM table WHERE ST_GeometryType(geom) IN ('ST_Polygon', 'ST_MultiPolygon')")
```

## API Test: `ST_Normalize`

### Signature
```python
ST_Normalize(arg1: geometry) -> geometry | ST_Normalize(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_normalize.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Normalize
description: Returns the geometry or geography in its canonical form.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

This function is mostly useful for testing and may rearrange vertices within a ring
or interior rings within a polygon.

## Examples

```sql
SELECT ST_Normalize(
    ST_GeomFromWKT('POLYGON ((10 10, 10 0, 0 0, 0 10, 10 10))')
);
```

```sql
SELECT ST_Normalize(
    ST_GeogFromWKT('POLYGON ((10 10, 10 0, 0 0, 0 10, 10 10))')
);
```


### Goal
Returns the input geometry or geography object in its canonical form, primarily used for testing purposes to ensure consistent vertex ordering.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be normalized.

### Input
- **Data:** A valid `geometry` or `geography` object.
- **Preconditions:** The input must be a valid spatial type supported by SedonaDB. No specific CRS matching is required as this is a structural transformation.

### Output
Returns a `geometry` or `geography` object (matching the input type) in its canonical form, which may involve reordering vertices within rings or reordering interior rings within a polygon.

### Valid Call Patterns
```python
# Inferred from SQL examples
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT ST_Normalize(ST_GeomFromWKT('POLYGON ((10 10, 10 0, 0 0, 0 10, 10 10))'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Normalize` when you need to standardize the internal representation of a geometry or geography for testing or comparison.
- Always pass the geometry/geography object as the single argument to the function.
- Ensure the input is a valid spatial type (e.g., created via `ST_GeomFromWKT` or `ST_GeogFromWKT`).

### Prompt Snippet
```text
ST_Normalize(geometry_column)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) instead of a geometry/geography object.
- Expecting the function to change the spatial extent or shape; it only changes the internal vertex/ring ordering.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type first
sd.sql("SELECT ST_Normalize(ST_GeomFromText('...'))")
```

## API Test: `ST_NumGeometries`

### Signature
```python
ST_NumGeometries(arg1: geometry) -> integer | ST_NumGeometries(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_numgeometries.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_NumGeometries
description: Returns the number of geometries in a geometry or geography collection.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

## Description

Returns the number of Geometries. If geometry is a GEOMETRYCOLLECTION (or MULTI*) return the number of geometries, for single geometries will return 1.

## Examples

```sql
SELECT ST_NumGeometries(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'));
```

```sql
SELECT ST_NumGeometries(ST_GeogFromWKT('MULTIPOINT ((0 0), (1 1))'));
```


### Goal
Calculates the count of individual geometry components within a `GEOMETRYCOLLECTION` or `MULTI*` geometry/geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect. Can be a collection type (e.g., `MULTIPOINT`, `MULTILINESTRING`, `MULTIPOLYGON`, `GEOMETRYCOLLECTION`) or a single geometry.

### Input
- **Data:** Accepts standard vector formats supported by SedonaDB (WKT, WKB, GeoParquet).
- **Preconditions:** The input must be a valid geometry or geography object. If the input is a single geometry (e.g., `POINT`, `POLYGON`), the function returns 1.

### Output
Returns `integer` — The total count of constituent geometries within the input object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify count for a MultiPoint geometry
result = sd.sql("SELECT ST_NumGeometries(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))')) as count").to_arrow_table()
assert result["count"][0] == 2
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_NumGeometries` to determine the cardinality of collection-type geometries.
- Always pass the geometry or geography object as the sole argument.
- Do not use this function on raster objects; it is strictly for vector geometry/geography types.

### Prompt Snippet
```text
ST_NumGeometries(geometry_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a raster object instead of a geometry/geography object will result in a SQL execution error.
- **Null Inputs:** Passing a `NULL` geometry will return `NULL` rather than an integer.
- **Invalid WKT:** Providing malformed WKT strings to `ST_GeomFromWKT` before passing to `ST_NumGeometries` will cause the query to fail.

### Fix Code Hint
```python
# Ensure the input is a valid geometry type before calling
sd.sql("SELECT ST_NumGeometries(ST_GeomFromWKT(my_wkt_col)) FROM my_table")
```

## API Test: `ST_NumInteriorRings`

### Signature
```python
ST_NumInteriorRings(arg1: geometry) -> integer | ST_NumInteriorRings(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_numinteriorrings.qmd:19_

### Goal
Calculates the count of interior rings (holes) contained within a polygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The polygon object to analyze.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** The input must be a polygon. If the input is a `MULTIPOLYGON`, the function evaluates only the first polygon in the collection.
- **Formats:** Typically provided via WKT (e.g., `ST_GeomFromText`) or WKB.

### Output
Returns `integer` — The number of interior rings. Returns `0` if the polygon has no holes, and `NULL` if the input is not a polygon type.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify hole count in a geometry polygon
query = "SELECT ST_NumInteriorRings(ST_GeomFromText('POLYGON ((0 0, 0 10, 10 10, 10 0, 0 0), (1 1, 1 2, 2 2, 2 1, 1 1))')) as num_rings"
result = sd.sql(query).to_arrow_table()
assert result["num_rings"][0] == 1
print(f"__CHECK__ witness: {result['num_rings'][0]}")
```

### LLM Instruction Prompt
- Use `ST_NumInteriorRings` to count holes in polygon data.
- If the input is a `MULTIPOLYGON`, note that the result only reflects the first polygon.
- If the input is not a polygon, expect a `NULL` result.
- Use the alias `ST_NumInteriorRing` if required by legacy SQL compatibility.

### Prompt Snippet
```text
ST_NumInteriorRings(geometry_column)
```

### Common Failure Modes
- **Non-Polygon Input:** Passing a `POINT` or `LINESTRING` will return `NULL` rather than raising an error.
- **Multipolygon Misinterpretation:** Users expecting a total count for all polygons in a `MULTIPOLYGON` will receive only the count for the first polygon.
- **Type Mismatch:** Passing non-spatial types will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure input is a polygon before calling
query = "SELECT ST_NumInteriorRings(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Polygon'"
```

## API Test: `ST_NumPoints`

### Signature
```python
ST_NumPoints(arg1: geometry) -> integer | ST_NumPoints(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_numpoints.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_NumPoints
description: Returns the number of points of a linestring geometry or geography.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

### Goal
Calculates the total number of coordinate points (vertices) specifically for linestring geometries or geographies.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object, typically a `LINESTRING`.

### Input
- **Data Formats:** Accepts geometries or geographies created via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`.
- **Preconditions:** The function is designed for `LINESTRING` types. While it accepts other types, it will return `NULL` for non-linestring geometries.

### Output
Returns `integer` — The count of coordinate points defining the linestring.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = "SELECT ST_NumPoints(ST_GeomFromWKT('LINESTRING (1 2, 3 4, 5 6)')) as num_points"
result = sd.sql(query).to_arrow_table()
assert result["num_points"][0] == 3
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_NumPoints` only when the input is a `LINESTRING`.
- If the input geometry type is unknown or not a `LINESTRING`, use `ST_NPoints()` instead to avoid `NULL` results.
- Always wrap the input in a constructor like `ST_GeomFromWKT` if passing a WKT string literal.

### Prompt Snippet
```text
ST_NumPoints(ST_GeomFromWKT('LINESTRING (...)'))
```

### Common Failure Modes
- **Non-Linestring Input:** Passing a `POLYGON` or `POINT` will return `NULL` rather than the vertex count. Use `ST_NPoints()` for general vertex counting.
- **Type Mismatch:** Passing non-spatial types or invalid WKT strings will result in a SQL execution error.

### Fix Code Hint
```python
# If you need the vertex count for a Polygon, use ST_NPoints instead:
query = "SELECT ST_NPoints(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))"
```

## API Test: `ST_Overlaps`

### Signature
```python
ST_Overlaps(arg1: geometry, arg2: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_overlaps.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Overlaps
description: Returns true if A overlaps B.
kernels:
  - returns: boolean
    args: [geometry, geometry]
---

## Description

Returns `true` if the two geometries share space and have the same dimension, but are not completely contained by each other.

## Examples

```sql
SELECT ST_Overlaps(
    ST_GeomFromWKT('POLYGON((2.5 2.5, 2.5 4.5, 4.5 4.5, 4.5 2.5, 2.5 2.5))'),
    ST_GeomFromWKT('POLYGON((4 4, 4 6, 6 6, 6 4, 4 4))')
);
```


### Goal
Determines if two geometries spatially overlap by sharing interior space without one fully containing the other.

### Parameters
- `arg1` (`geometry`): The first geometry object to compare.
- `arg2` (`geometry`): The second geometry object to compare.

### Input
- **Data Formats:** Geometries must be provided as SedonaDB-compatible geometry types (e.g., created via `ST_GeomFromWKT`, `ST_Point`, etc.).
- **Preconditions:** Both geometries must have the same dimension (e.g., both polygons or both lines). If geometries have mismatched Coordinate Reference Systems (CRS), they must be reprojected to a common CRS before comparison.

### Output
Returns `boolean` — `true` if the geometries overlap, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT inputs
query = """
SELECT ST_Overlaps(
    ST_GeomFromWKT('POLYGON((0 0, 0 2, 2 2, 2 0, 0 0))'),
    ST_GeomFromWKT('POLYGON((1 1, 1 3, 3 3, 3 1, 1 1))')
) as is_overlapping
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Overlaps(geom1, geom2)` within a `SELECT` statement.
- Ensure both arguments are valid geometry types.
- Verify that both geometries share the same dimension before calling.

### Prompt Snippet
```text
ST_Overlaps(geometry_a, geometry_b)
```

### Common Failure Modes
- **Dimension Mismatch:** Calling the function with geometries of different dimensions (e.g., a Point and a Polygon) will not return the expected overlap result.
- **CRS Mismatch:** If geometries are in different coordinate systems, the operation may fail or return incorrect results; ensure both are in the same CRS.
- **Containment:** If one geometry is completely inside the other, `ST_Overlaps` returns `false` because the definition requires that they are not completely contained by each other.

### Fix Code Hint
```python
# Ensure geometries are in the same CRS if necessary
# Use ST_Transform(geom, target_crs) if CRS mismatch is suspected
sd.sql("SELECT ST_Overlaps(ST_Transform(g1, 4326), ST_Transform(g2, 4326)) FROM table")
```

## API Test: `ST_Perimeter`

### Signature
```python
ST_Perimeter(arg1: geometry) -> double | ST_Perimeter(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_perimeter.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Perimeter
description: Calculates the perimeter of a given geometry or geography.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

This function calculates the perimeter of a given geometry or geography. It supports Polygon, MultiPolygon, and GeometryCollection geometries (as long as the GeometryCollection contains polygonal geometries). For other types, it returns 0. To measure lines, use ST_Length. For geography, returns the spherical approximation of the geodesic perimeter in meters.

## Examples

```sql
SELECT ST_Perimeter(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'));
```

```sql
SELECT ST_Perimeter(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'));
```


### Goal
Calculates the total length of the boundary of a polygonal geometry or the spherical geodesic perimeter of a geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (Polygon, MultiPolygon, or GeometryCollection containing polygons) for which the perimeter is calculated.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Formats:** Supports WKT, WKB, or GeoParquet-loaded spatial columns.
- **Preconditions:** For `GeometryCollection` inputs, the collection must contain polygonal geometries. Non-polygonal types (e.g., Point, LineString) will return 0.

### Output
Returns `double` — The perimeter value. For `geography` inputs, the result is the spherical approximation of the geodesic perimeter in meters.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_Perimeter(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')) as perimeter").to_arrow_table()
print(result)
# __CHECK__ witness: 4.0
```

### LLM Instruction Prompt
- Use `ST_Perimeter` only for polygonal geometries or geography objects.
- If the input is a line, use `ST_Length` instead.
- Ensure the input is a valid spatial type registered in the SedonaDB session.

### Prompt Snippet
```text
ST_Perimeter(geometry_column)
```

### Common Failure Modes
- **Non-polygonal input:** Passing a Point or LineString will return 0.0 rather than an error.
- **GeometryCollection:** If the collection contains no polygons, the result will be 0.0.
- **CRS Mismatch:** While `ST_Perimeter` operates on the provided object, ensure the input geometry is in a projected coordinate system if planar perimeter is required, or use `geography` for geodesic calculations.

### Fix Code Hint
```python
# If input is a line, switch to ST_Length
sd.sql("SELECT ST_Length(ST_GeomFromWKT('LINESTRING(0 0, 1 1)'))").to_arrow_table()

# If input is a geometry, ensure it is a polygon
sd.sql("SELECT ST_Perimeter(ST_GeomFromWKT('POLYGON((...))'))").to_arrow_table()
```

## API Test: `ST_Point`

### Signature
```python
ST_Point(x: double, y: double, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_point.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Point
description: Constructs a Point Geometry from X and Y.
kernels:
  - returns: geometry
    args:
    - name: x
      type: double
    - name: "y"
      type: double
    - name: srid
      type: crs
---

## Description

Constructs a Point geometry from X and Y coordinates. An optional SRID or CRS can be provided as a third argument.

## Examples

```sql
SELECT ST_AsText(ST_Point(-74.0060, 40.7128));
```

With an SRID:

```sql
SELECT ST_Point(-74.0060, 40.7128, 4326);
```

With a CRS:

```sql
SELECT ST_Point(-74.0060, 40.7128, 'EPSG:3857');
```


### Goal
Constructs a new Point geometry object from specified X and Y coordinates, optionally assigning a Coordinate Reference System (CRS) or SRID.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `srid` (`crs`): An optional identifier for the Coordinate Reference System (e.g., integer 4326 or string 'EPSG:3857').

### Input
The function accepts two `double` values for coordinates. An optional third argument defines the spatial reference. If provided, the SRID/CRS must be a valid identifier recognized by the SedonaDB engine.

### Output
Returns a `geometry` object representing a single point in the specified coordinate space.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic point construction
result = sd.sql("SELECT ST_Point(0, 1) as geom").to_arrow_table()
print(result)
# __CHECK__

# Point construction with SRID
result_srid = sd.sql("SELECT ST_Point(-74.0060, 40.7128, 4326) as geom").to_arrow_table()
print(result_srid)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Point(x, y)` for points without a defined CRS.
- Use `ST_Point(x, y, srid)` when the point must be associated with a specific spatial reference system.
- Ensure `x` and `y` are provided as numeric types.

### Prompt Snippet
```text
Use ST_Point(x, y) or ST_Point(x, y, srid) to create a geometry point.
```

### Common Failure Modes
- Providing non-numeric types for `x` or `y` will result in a SQL execution error.
- Providing an invalid or unsupported SRID string/integer will cause the query to fail during geometry construction.
- Mismatched CRS: If the resulting point is used in spatial operations (e.g., `RS_Clip` or spatial joins) with other geometries or rasters, ensure the SRIDs match to avoid runtime errors.

### Fix Code Hint
```python
# Ensure coordinates are floats and SRID is a valid integer or string
sd.sql("SELECT ST_Point(CAST(0.0 AS DOUBLE), CAST(1.0 AS DOUBLE), 4326)")
```

## API Test: `ST_PointFromText`

### Signature
```python
ST_PointFromText(wkt: string) -> geometry | ST_PointFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_pointfromtext.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PointFromText
description: Constructs a Point from Well-Known Text (WKT), erroring if the input is not a Point.
kernels:
  - returns: geometry
    args:
    - name: wkt
      type: string
  - returns: geometry
    args:
    - name: wkt
      type: string
    - name: srid
      type: crs
---

## Description

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a Point.

## Examples

```sql
SELECT ST_PointFromText('POINT (30 10)');
```

With an SRID or CRS:

```sql
SELECT ST_PointFromText('POINT (30 10)', 4326);
```


### Goal
Constructs a spatial `geometry` object from a Well-Known Text (WKT) string representation of a point for use in SedonaDB geospatial analysis.

### Parameters
- `wkt` (`string`): A string containing the Well-Known Text representation of a point (e.g., `'POINT (30 10)'`).
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to associate with the geometry.

### Input
The function accepts a string formatted as WKT. The input must strictly represent a `POINT` geometry; providing other geometry types (e.g., `POLYGON`, `LINESTRING`) will result in an error.

### Output
Returns a `geometry` object representing the point defined by the WKT string, optionally tagged with the provided SRID.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_PointFromText('POINT (30 10)') as geom").to_arrow_table()
print("__CHECK__", result)

# Construction with SRID
result_srid = sd.sql("SELECT ST_PointFromText('POINT (30 10)', 4326) as geom").to_arrow_table()
print("__CHECK__", result_srid)
```

### LLM Instruction Prompt
- Always use `sd.sql("...").to_arrow_table()` to execute the function.
- Ensure the WKT string explicitly starts with `POINT`.
- If an SRID is provided, ensure it is a valid integer CRS code.

### Prompt Snippet
```text
ST_PointFromText('POINT (x y)')
ST_PointFromText('POINT (x y)', srid)
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `POINT` (e.g., `POLYGON ((...))`) will raise an error.
- **Malformed WKT:** Passing a string that does not conform to the WKT specification for points will cause a parsing error.
- **CRS Mismatch:** If performing subsequent operations (like `RS_Clip` or spatial joins) with this geometry, ensure the SRID matches the other datasets in the query to avoid runtime errors.

### Fix Code Hint
```python
# Ensure the input is a point and the WKT is correctly formatted
query = "SELECT ST_PointFromText('POINT (0 0)')"
# If you have a polygon, use ST_GeomFromText instead
```

## API Test: `ST_PointFromWKB`

### Signature
```python
ST_PointFromWKB(wkb: binary) -> geometry | ST_PointFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_pointfromwkb.qmd:19_

## Description

Parses a WKB binary value and returns a Point geometry. Returns `NULL` if the
WKB represents another geometry type. An optional SRID or CRS can be provided
as a second argument.

## Examples

```sql
SELECT ST_PointFromWKB(
  decode('0101000000000000000000F03F0000000000000040', 'hex')
);
```

With an SRID or CRS:

```sql
SELECT ST_PointFromWKB(
  decode('0101000000000000000000F03F0000000000000040', 'hex'),
  4326
);
```

### Goal
Constructs a `Point` geometry object from a Well-Known Binary (WKB) representation for use in spatial analysis.

### Parameters
- `wkb` (`binary`): The Well-Known Binary representation of the geometry.
- `srid` (`crs`): An optional Coordinate Reference System identifier to associate with the resulting geometry.

### Input
The function expects a `binary` input representing a WKB-encoded geometry. If the input binary does not represent a `Point` geometry, the function returns `NULL`.

### Output
Returns a `geometry` object representing the point.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic usage
query = "SELECT ST_PointFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex')) as geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")

# Usage with SRID
query_srid = "SELECT ST_PointFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326) as geom"
result_srid = sd.sql(query_srid).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
When using `ST_PointFromWKB`, ensure the input binary is correctly encoded as WKB. If the binary represents a non-point geometry (e.g., a Polygon or LineString), the function will return `NULL`. Always verify the geometry type if the input source is dynamic.

### Prompt Snippet
```text
ST_PointFromWKB(binary_wkb_expression, [srid_integer])
```

### Common Failure Modes
- **Non-Point Input:** Providing WKB for a LineString or Polygon will result in a `NULL` return value rather than an error.
- **Invalid Binary:** Providing malformed binary data that does not conform to the WKB specification will cause the query to fail.
- **CRS Mismatch:** If performing subsequent spatial operations (like `RS_Clip` or spatial joins) with the resulting geometry, ensure the `srid` provided matches the CRS of other datasets in the query to avoid execution errors.

### Fix Code Hint
```python
# If you receive NULL, verify the WKB input type:
# Ensure the binary string is decoded from hex correctly:
query = "SELECT ST_PointFromWKB(decode('...', 'hex'), 4326)"
```

## API Test: `ST_PointM`

### Signature
```python
ST_PointM(x: double, y: double, m: double) -> geometry
```
_Source: source/docs/reference/sql/st_pointm.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PointM
description: Constructs a Point with an M (measure) coordinate from X, Y, and M values.
kernels:
  - returns: geometry
    args:
    - name: x
      type: double
    - name: "y"
      type: double
    - name: m
      type: double
---

## Examples

```sql
SELECT ST_PointM(-64.36, 45.09, 50.0);
```


### Goal
Constructs a 3D-like geometry object (Point) that includes an X coordinate, a Y coordinate, and an additional M (measure) value for linear referencing or spatial analysis.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `m` (`double`): The measure value associated with the point, often used for linear referencing systems.

### Input
The function accepts three `double` precision floating-point numbers. No specific file format is required as this is a scalar constructor function used within SQL queries.

### Output
Returns `geometry` — A point geometry object containing the specified X, Y, and M coordinates.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example of calling ST_PointM via the Python SQL interface
result = sd.sql("SELECT ST_PointM(-64.36, 45.09, 50.0) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_PointM` when you need to create a point geometry that includes a measure (M) dimension.
- Ensure all three arguments are provided as `double` types.
- The function is a scalar function and should be called within a `SELECT` statement.

### Prompt Snippet
```text
SELECT ST_PointM(x_val, y_val, m_val)
```

### Common Failure Modes
- Providing non-numeric types (e.g., strings) for the coordinates or measure will result in a SQL execution error.
- Attempting to use the resulting geometry in functions that do not support M-coordinates may lead to unexpected behavior or truncation of the M-value depending on the downstream function's implementation.

### Fix Code Hint
```python
# Ensure inputs are cast to float if they are coming from dynamic sources
x, y, m = float(x), float(y), float(m)
query = f"SELECT ST_PointM({x}, {y}, {m})"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_PointN`

### Signature
```python
ST_PointN(arg1: geometry, n: integer) -> geometry | ST_PointN(arg1: geography, n: integer) -> geography
```
_Source: source/docs/reference/sql/st_pointn.qmd:19_

### Goal
Extracts a specific vertex from a `LineString` geometry or geography object based on its index.

### Parameters
- `arg1` (`geometry` | `geography`): The input `LineString` object from which to extract a point.
- `n` (`integer`): The 1-based index of the point to retrieve. Negative values are supported, where -1 represents the last point in the sequence.

### Input
- **Data:** Requires a valid `LineString` geometry or geography.
- **Preconditions:** The input must be a `LineString`. If the input is not a `LineString` (e.g., a `Point` or `Polygon`), or if the index `n` is out of bounds, the function returns `NULL`.

### Output
Returns `geography | geometry` — A `Point` object representing the Nth vertex of the input linestring.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Extract the 2nd point from a geometry LineString
result = sd.sql("SELECT ST_PointN(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'), 2) as point").to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a Point(1 2)
```

### LLM Instruction Prompt
- Use `ST_PointN` when you need to isolate a specific coordinate from a path or linestring.
- Always ensure the input is a `LineString` type.
- Remember that indexing is 1-based; use negative integers to access points relative to the end of the line.

### Prompt Snippet
```text
ST_PointN(geometry_column, index_integer)
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a `Polygon` or `Point` instead of a `LineString` will result in a `NULL` return value.
- **Index Out of Bounds:** Providing an index `n` that exceeds the number of points in the `LineString` will return `NULL`.
- **Zero Index:** Using `0` as an index is invalid as the function expects 1-based indexing.

### Fix Code Hint
```python
# Ensure the input is a LineString and the index is within the valid range [1, num_points] or [-num_points, -1]
sd.sql("SELECT ST_PointN(geom, 1) FROM my_table WHERE ST_GeometryType(geom) = 'ST_LineString'")
```

## API Test: `ST_PointOnSurface`

### Signature
```python
ST_PointOnSurface(arg1: geometry) -> geometry | ST_PointOnSurface(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_pointonsurface.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PointOnSurface
description: Returns a point guaranteed to lie on the surface of a geometry or geography.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

Unlike ST_Centroid which does not necessarily intersect a geometry or geography, ST_PointOnSurface
makes an attempt to find an interior point close to the middle (using deterministic heuristics).

## Examples

```sql
SELECT ST_PointOnSurface(
    ST_GeogFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))')
);
```


### Goal
Computes a representative point that is guaranteed to lie within the interior or on the boundary of the input geometry or geography.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object for which an interior point is to be calculated.

### Input
- **Data:** Accepts standard geometry or geography types (e.g., Polygons, MultiPolygons).
- **Formats:** Supports WKT, WKB, or GeoParquet-loaded geometries.
- **Preconditions:** The input must be a valid spatial object. Unlike `ST_Centroid`, this function is specifically designed to ensure the resulting point intersects the input object.

### Output
Returns `geography | geometry` — A `Point` object that is guaranteed to be contained within the input geometry or geography.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT geometry
query = "SELECT ST_PointOnSurface(ST_GeomFromText('POLYGON((0 0, 10 0, 10 10, 0 10, 0 0))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_PointOnSurface` when you need a point guaranteed to be inside a polygon, as opposed to `ST_Centroid` which may return a point outside the shape (e.g., for concave polygons).
- Ensure the input is a valid geometry or geography type.

### Prompt Snippet
```text
ST_PointOnSurface(geometry_column)
```

### Common Failure Modes
- **Invalid Geometry:** Passing a null or malformed geometry string will result in a query execution error.
- **Type Mismatch:** Attempting to pass a raster object directly without first converting it to a geometry (e.g., via `ST_ConvexHull` or similar) will fail.
- **Empty Input:** Passing an empty geometry may return an empty point or null depending on the specific engine state.

### Fix Code Hint
```python
# Ensure input is valid and non-empty
query = "SELECT ST_PointOnSurface(geom) FROM table WHERE ST_IsValid(geom) AND NOT ST_IsEmpty(geom)"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_PointZ`

### Signature
```python
ST_PointZ(x: double, y: double, z: double) -> geometry
```
_Source: source/docs/reference/sql/st_pointz.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PointZ
description: Constructs a Point with a Z coordinate from X, Y, and Z values.
kernels:
  - returns: geometry
    args:
    - name: x
      type: double
    - name: "y"
      type: double
    - name: z
      type: double
---

## Examples

```sql
SELECT ST_PointZ(-64.36, 45.09, 10.5);
```


### Goal
Constructs a 3D Point geometry object by combining specified X, Y, and Z coordinate values.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `z` (`double`): The Z-coordinate (elevation or altitude) of the point.

### Input
The function accepts three `double` precision floating-point numbers. No specific file format is required as this is a scalar constructor function.

### Output
Returns `geometry` — A 3D Point geometry object containing the provided X, Y, and Z coordinates.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and SQL example
result = sd.sql("SELECT ST_PointZ(-64.36, 45.09, 10.5) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_PointZ` when you need to create a 3D point geometry from explicit coordinate values.
- Ensure all three arguments are provided as `double` types.
- The function is a scalar SQL function and should be called within a `SELECT` statement via `sd.sql()`.

### Prompt Snippet
```text
ST_PointZ(x, y, z)
```

### Common Failure Modes
- Providing fewer than three arguments: The function signature strictly requires three `double` values.
- Passing non-numeric types: Passing strings or other types that cannot be cast to `double` will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure all inputs are cast to float if they are coming from variables
x, y, z = -64.36, 45.09, 10.5
query = f"SELECT ST_PointZ({float(x)}, {float(y)}, {float(z)})"
sd.sql(query).to_arrow_table()
```

## API Test: `ST_PointZM`

### Signature
```python
ST_PointZM(x: double, y: double, z: double, m: double) -> geometry
```
_Source: source/docs/reference/sql/st_pointzm.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PointZM
description: Constructs a Point with X, Y, Z and M coordinates.
kernels:
  - returns: geometry
    args:
    - name: x
      type: double
    - name: "y"
      type: double
    - name: z
      type: double
    - name: m
      type: double
---

## Examples

```sql
SELECT ST_PointZM(-64.36, 45.09, 10.5, 50.0);
```


### Goal
Constructs a 4-dimensional Point geometry containing X, Y, Z (elevation/height), and M (measure) coordinates.

### Parameters
- `x` (`double`): The X coordinate of the point.
- `y` (`double`): The Y coordinate of the point.
- `z` (`double`): The Z coordinate (elevation or height) of the point.
- `m` (`double`): The M coordinate (measure) of the point.

### Input
The function accepts four `double` precision floating-point values. No specific file format is required as this is a scalar constructor function.

### Output
Returns `geometry` — A point object represented in 4D (XYZM) space.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and SQL example
result = sd.sql("SELECT ST_PointZM(-64.36, 45.09, 10.5, 50.0) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_PointZM` when you need to create a point geometry that includes both elevation (Z) and a linear referencing measure (M).
- Ensure all four arguments are provided as numeric types (doubles).

### Prompt Snippet
```text
ST_PointZM(x, y, z, m)
```

### Common Failure Modes
- Providing fewer than four arguments will result in a SQL execution error as the function signature requires exactly four parameters.
- Passing non-numeric types (e.g., strings) for any coordinate will cause a type mismatch error.

### Fix Code Hint
```python
# Ensure all inputs are cast to float if they originate from dynamic sources
sd.sql("SELECT ST_PointZM(CAST(x AS DOUBLE), CAST(y AS DOUBLE), CAST(z AS DOUBLE), CAST(m AS DOUBLE)) FROM my_table")
```

## API Test: `ST_Points`

### Signature
```python
ST_Points(arg1: geometry) -> geometry | ST_Points(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_points.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Points
description: Returns a MultiPoint geometry consisting of all the coordinates of the input geometry or geography.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

Returns a MultiPoint geometry consisting of all the coordinates of the input geometry. It preserves duplicate points as well as M and Z coordinates.

## Examples

```sql
SELECT ST_Points(ST_GeomFromText('LINESTRING (2 4, 3 3, 4 2)'));
```

```sql
SELECT ST_Points(ST_GeogFromWKT('LINESTRING (2 4, 3 3, 4 2)'));
```


### Goal
Extracts all individual coordinate vertices from a complex geometry or geography object and returns them as a single `MultiPoint` collection.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object (e.g., LineString, Polygon) from which to extract vertices.

### Input
- **Data:** Accepts standard vector types including LineStrings, Polygons, and Multi-geometries.
- **Preconditions:** The input must be a valid geometry or geography object. If using geography, ensure the input is correctly defined via `ST_GeogFromWKT` or similar constructors.

### Output
Returns `geography | geometry` — A `MultiPoint` object containing all coordinate points found in the input, preserving original Z and M dimensions and duplicate vertices.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Extract points from a LineString geometry
result = sd.sql("SELECT ST_Points(ST_GeomFromText('LINESTRING (2 4, 3 3, 4 2)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Points` when you need to decompose complex geometries into their constituent vertices for point-based analysis or visualization.
- Ensure the input type matches the expected return type (geometry for geometry, geography for geography).

### Prompt Snippet
```text
SELECT ST_Points(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or an unsupported geometry type that does not contain vertices.
- **CRS Mismatch:** If performing operations involving multiple geometries, ensure they share the same CRS, as SedonaDB requires explicit reprojection for mismatched spatial references.

### Fix Code Hint
```python
# Ensure input is a valid geometry before calling ST_Points
sd.sql("SELECT ST_Points(ST_MakeValid(geom)) FROM my_table")
```

## API Test: `ST_PolygonFromText`

### Signature
```python
ST_PolygonFromText(wkt: string) -> geometry | ST_PolygonFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_polygonfromtext.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_PolygonFromText
description: Constructs a Polygon from Well-Known Text (WKT), erroring if the input is not a Polygon.
kernels:
  - returns: geometry
    args:
    - name: wkt
      type: string
  - returns: geometry
    args:
    - name: wkt
      type: string
    - name: srid
      type: crs
---

## Description

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a Polygon.

## Examples

```sql
SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))');
```

With an SRID or CRS:

```sql
SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))', 4326);
```


### Goal
Constructs a geometry object from a Well-Known Text (WKT) string specifically representing a polygon for use in SedonaDB spatial operations.

### Parameters
- `wkt` (`string`): A valid Well-Known Text string representing a polygon geometry.
- `srid` (`crs`): An optional integer representing the Coordinate Reference System (CRS) identifier to associate with the resulting geometry.

### Input
The function accepts a string formatted as WKT. The input must strictly represent a `POLYGON` type; providing other geometry types (e.g., `POINT`, `LINESTRING`) will result in an error.

### Output
Returns a `geometry` object that can be used in subsequent spatial analysis, joins, or raster-geometry interactions (e.g., `RS_Clip`).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic construction
result = sd.sql("SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))') as geom").to_arrow_table()
print("__CHECK__", result)

# Construction with SRID
result_srid = sd.sql("SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))', 4326) as geom").to_arrow_table()
print("__CHECK__", result_srid)
```

### LLM Instruction Prompt
- Always ensure the WKT string is a valid `POLYGON` definition.
- If the spatial operation involves other geometries or rasters with a defined CRS, provide the `srid` parameter to ensure coordinate system compatibility.

### Prompt Snippet
```text
Use ST_PolygonFromText(wkt) or ST_PolygonFromText(wkt, srid) to create a polygon geometry from a WKT string.
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `POLYGON` (e.g., `POINT(0 0)`) will cause the function to raise an error.
- **Malformed WKT:** Providing a string that does not conform to the WKT specification for polygons will result in a parsing error.
- **CRS Mismatch:** If the resulting geometry is used in operations with other spatial data (like `RS_Clip`), ensure the `srid` matches the target dataset to avoid runtime errors.

### Fix Code Hint
```python
# Ensure the WKT string is a valid polygon
wkt_input = 'POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'
# If using with other data, verify the SRID matches
query = f"SELECT ST_PolygonFromText('{wkt_input}', 4326)"
```

## API Test: `ST_Polygonize`

### Signature
```python
ST_Polygonize(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_polygonize.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Polygonize
description: Builds a polygonal geometry from linear components in the input geometry.
kernels:
  - returns: geometry
    args: [geometry]
---

## Description

Builds a polygonal geometry from linear components in the input geometry. Handles GeometryCollection and multi-geometries by extracting linear components. Typically used with collections of complete, simple linework that forms polygon boundaries.

Unlike many geometry constructors, this doesn't require explicit closure of rings—any closed rings in the input will become polygon boundaries.

## Examples

```sql
SELECT ST_Polygonize(ST_GeomFromText('MULTILINESTRING ((0 0, 0 10, 10 10), (10 10, 10 0, 0 0))'));
```


### Goal
Constructs a polygonal geometry by assembling closed rings from the linear components provided in the input geometry.

### Parameters
- `arg1` (`geometry`): A geometry containing linear components (e.g., `LineString`, `MultiLineString`, or `GeometryCollection`) that form the boundaries of the desired polygons.

### Input
The input must be a valid geometry object containing linear segments. It is typically used with collections of complete, simple linework. The function does not require explicit closure of rings; it automatically identifies and processes closed rings within the input.

### Output
Returns a `geometry` object representing the polygons formed by the input linear components.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = "SELECT ST_Polygonize(ST_GeomFromText('MULTILINESTRING ((0 0, 0 10, 10 10), (10 10, 10 0, 0 0))')) as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Polygonize` when you need to convert a set of lines into a polygon.
- Ensure the input is a geometry type containing linear components.
- Do not assume the input rings are already closed; the function handles the closure logic.

### Prompt Snippet
```text
ST_Polygonize(geometry_column)
```

### Common Failure Modes
- Passing a geometry that contains no linear components (e.g., a single `Point`), which will result in an empty geometry or an error.
- Providing disconnected or incomplete linework that does not form a closed ring, which may result in an empty geometry output.

### Fix Code Hint
```python
# Ensure input is a valid MultiLineString or GeometryCollection
# If input is raw coordinates, wrap in ST_GeomFromText or ST_GeomFromWKB
sd.sql("SELECT ST_Polygonize(ST_GeomFromText('...'))").to_arrow_table()
```

## API Test: `ST_Polygonize_Agg`

### Signature
```python
ST_Polygonize_Agg(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_polygonize_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Polygonize_Agg
description: Creates polygons from a set of geometries that contain linework representing the edges of a polygon.
kernels:
  - returns: geometry
    args: [geometry]
---

## Examples

```sql
SELECT ST_Polygonize_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('LINESTRING (0 0, 0 10)')),
    (ST_GeomFromText('LINESTRING (0 10, 10 10)')),
    (ST_GeomFromText('LINESTRING (10 10, 10 0)')),
    (ST_GeomFromText('LINESTRING (10 0, 0 0)'))
) AS t(geom);
```


### Goal
`ST_Polygonize_Agg` is an aggregate function that constructs polygons from a collection of input linestring geometries that represent the edges of a polygon.

### Parameters
- `arg1` (`geometry`): A set of linestring geometries that form the boundaries of the desired polygons.

### Input
The input must be a set of geometries (typically `LINESTRING`) that form closed loops. The function operates as an aggregate, meaning it processes multiple rows of geometry data to produce a single resulting geometry or collection of geometries.

### Output
Returns `geometry` — A polygon or a collection of polygons formed by the input linework.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using the aggregate function via SQL
query = """
SELECT ST_Polygonize_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('LINESTRING (0 0, 0 10)')),
    (ST_GeomFromText('LINESTRING (0 10, 10 10)')),
    (ST_GeomFromText('LINESTRING (10 10, 10 0)')),
    (ST_GeomFromText('LINESTRING (10 0, 0 0)'))
) AS t(geom)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness:", result)
```

### LLM Instruction Prompt
- Use `ST_Polygonize_Agg` only as an aggregate function in a `SELECT` statement.
- Ensure the input geometries are linestrings that share endpoints to form valid closed polygons.
- The function is not a scalar function; it must be used on a column of geometries.

### Prompt Snippet
```text
ST_Polygonize_Agg(geometry_column)
```

### Common Failure Modes
- Providing geometries that do not form closed loops, resulting in an empty or null geometry output.
- Attempting to use the function as a scalar function on a single geometry rather than an aggregate over a set of rows.
- Inputting non-linestring geometries (e.g., points) which cannot be polygonized.

### Fix Code Hint
```python
# Ensure input is a set of connected linestrings
# If the result is null, verify that the linestrings form a closed, valid polygon boundary.
```

## API Test: `ST_ReducePrecision`

### Signature
```python
ST_ReducePrecision(arg1: geometry, grid_size: double) -> geometry | ST_ReducePrecision(arg1: geography, grid_size: double) -> geography
```
_Source: source/docs/reference/sql/st_reduceprecision.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ReducePrecision
description: Reduces the coordinate precision of a geometry or geography to the specified grid size.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: grid_size
      type: double
  - returns: geography
    args:
    - geography
    - name: grid_size
      type: double
---

## Description

Rebuilds a new geometry after specifying all vertices to the specified size. Z and M values are preserved
but not affected by the grid snapping. The implementation is similar to a unary union and is designed to
output valid geometry (e.g., by avoiding creating new crossing edges when snapping vertices).

## Examples

```sql
SELECT ST_ReducePrecision(
    ST_GeogFromWKT('POINT (1.123456 2.654321)'),
    0.01
);
```


### Goal
Reduces the coordinate precision of a geometry or geography object by snapping its vertices to a grid of the specified size.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object whose vertex precision is to be reduced.
- `grid_size` (`double`): The size of the grid to which the vertices will be snapped.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** The input must be a valid geometry or geography object. Z and M values are preserved but remain unaffected by the snapping process.

### Output
Returns `geography | geometry` — A new geometry or geography object with vertices snapped to the specified grid size, ensuring the output remains valid.

### Valid Call Patterns
```python
# Example using the Python SQL interface
query = "SELECT ST_ReducePrecision(ST_GeogFromWKT('POINT (1.123456 2.654321)'), 0.01) as reduced_geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_ReducePrecision(arg1, grid_size)` as a standalone SQL function call within `sd.sql()`.
- Ensure the `grid_size` is provided as a positive double.
- The function preserves Z and M values; do not expect these to be modified by the snapping.

### Prompt Snippet
```text
ST_ReducePrecision(geometry_column, 0.001)
```

### Common Failure Modes
- Providing a non-numeric value for `grid_size`.
- Passing an invalid geometry or geography object that cannot be processed by the snapping algorithm.
- Expecting Z or M values to be snapped; the documentation explicitly states these are preserved but not affected.

### Fix Code Hint
```python
# Ensure grid_size is a float and input is a valid geometry
sd.sql("SELECT ST_ReducePrecision(geom, 0.1) FROM my_table").to_arrow_table()
```

## API Test: `ST_Relate`

### Signature
```python
ST_Relate(arg1: geometry, arg2: geometry) -> string | ST_Relate(arg1: geometry, arg2: geometry, intersectionMatrixPattern: string) -> boolean
```
_Source: source/docs/reference/sql/st_relate.qmd:18_

### Goal
`ST_Relate` computes the DE-9IM intersection matrix string for two geometries or validates if their spatial relationship matches a specific DE-9IM pattern.

### Parameters
- `arg1` (`geometry`): The primary geometry object to evaluate.
- `arg2` (`geometry`): The secondary geometry object to compare against the primary.
- `intersectionMatrixPattern` (`string`): A 9-character DE-9IM pattern string (using 0-2, T, F, or * wildcards) to test against the relationship of the two geometries.

### Input
- **Data:** Standard geometry types (Point, Polygon, etc.) created via constructors like `ST_GeomFromText`.
- **Preconditions:** Both geometries must be valid. If the geometries have defined Coordinate Reference Systems (CRS), they must match; otherwise, the operation may fail or produce undefined results.

### Output
Returns `boolean | string` — When two arguments are provided, it returns a 9-character string representing the DE-9IM matrix. When three arguments are provided, it returns `true` if the relationship matches the pattern, otherwise `false`.

### Valid Call Patterns
```python
# Inferred from SQL examples
import sedona.db
sd = sedona.db.connect()

# Get DE-9IM string
result = sd.sql("SELECT ST_Relate(ST_GeomFromText('POINT (0.5 0.5)'), ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print("__CHECK__", result)

# Test against pattern
result_bool = sd.sql("SELECT ST_Relate(ST_GeomFromText('POINT (0.5 0.5)'), ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), '0FFFFF212')").to_arrow_table()
print("__CHECK__", result_bool)
```

### LLM Instruction Prompt
- Use `ST_Relate(geom1, geom2)` to retrieve the full DE-9IM matrix string.
- Use `ST_Relate(geom1, geom2, pattern)` to perform a boolean check against a specific DE-9IM pattern.
- Ensure all geometry inputs are valid WKT or geometry objects.

### Prompt Snippet
```text
ST_Relate(geometry, geometry) -> string
ST_Relate(geometry, geometry, string) -> boolean
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different CRS without prior reprojection.
- **Invalid Pattern:** Providing a `intersectionMatrixPattern` that is not exactly 9 characters or contains invalid characters (only 0-2, T, F, * are allowed).
- **Invalid Geometry:** Passing null or malformed geometry objects will result in an execution error.

### Fix Code Hint
```python
# Ensure geometries are in the same CRS before calling ST_Relate
query = """
SELECT ST_Relate(
    ST_Transform(geom1, 'EPSG:4326'), 
    ST_Transform(geom2, 'EPSG:4326')
) FROM my_table
"""
```

## API Test: `ST_Reverse`

### Signature
```python
ST_Reverse(arg1: geometry) -> geometry | ST_Reverse(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_reverse.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Reverse
description: Returns the geometry or geography with vertex order reversed.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Examples

```sql
SELECT ST_Reverse(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'));
```

```sql
SELECT ST_Reverse(ST_GeogFromWKT('LINESTRING (0 0, 1 2, 2 4)'));
```


### Goal
Reverses the sequence of vertices in a given geometry or geography object, effectively flipping the direction of lines or the orientation of polygon rings.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object whose vertex order is to be reversed.

### Input
- **Data Formats:** Accepts standard WKT, WKB, or GeoParquet-loaded geometries and geographies.
- **Preconditions:** The input must be a valid geometry or geography type. No specific CRS matching is required for this operation as it is a structural transformation of the object's internal vertex list.

### Output
Returns a `geometry` or `geography` object (matching the input type) where the vertex order is reversed.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_Reverse(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Reverse` when the directionality of a `LINESTRING` or the winding order of a `POLYGON` needs to be inverted.
- Ensure the input is explicitly cast or constructed as a `geometry` or `geography` type using constructors like `ST_GeomFromText` or `ST_GeogFromWKT`.

### Prompt Snippet
```text
ST_Reverse(arg1)
```

### Common Failure Modes
- **Type Mismatch:** Passing a raster object instead of a geometry or geography will result in a SQL execution error.
- **Invalid Geometry:** If the input geometry is malformed or null, the function may return null or raise an error depending on the underlying DataFusion engine state.

### Fix Code Hint
```python
# Ensure input is a valid geometry before reversing
sd.sql("SELECT ST_Reverse(ST_GeomFromText('LINESTRING (0 0, 1 1)'))").to_arrow_table()
```

## API Test: `ST_Rotate`

### Signature
```python
ST_Rotate(arg1: geometry, rot: double) -> geometry
```
_Source: source/docs/reference/sql/st_rotate.qmd:19_

### Goal
Rotates a geometry counter-clockwise around the Z axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry to be rotated.
- `rot` (`double`): The rotation angle in radians.

### Input
- **Data:** A valid geometry object (e.g., Point, LineString, Polygon).
- **Preconditions:** The geometry must be defined in a coordinate system where rotation around the Z axis is meaningful.

### Output
Returns `geometry` — A new geometry object representing the input rotated counter-clockwise by the specified angle.

### Valid Call Patterns
```python
# Inferred from SQL example:
result = sd.sql("SELECT ST_Rotate(ST_GeomFromText('POINT (1 0)'), radians(90))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Rotate(geometry, double)` to perform counter-clockwise rotations.
- Ensure the rotation angle is provided in radians; use the `radians()` SQL function if the input is in degrees.
- The function operates on the Z axis; ensure the input geometry is compatible with this transformation.

### Prompt Snippet
```text
ST_Rotate(geometry_column, radians(angle_in_degrees))
```

### Common Failure Modes
- **Invalid Angle:** Providing an angle in degrees instead of radians without conversion.
- **Null Geometry:** Passing a null geometry input will result in a null output.
- **Unsupported Geometry Types:** Attempting to rotate complex geometries that may result in invalid topological states (e.g., self-intersecting polygons).

### Fix Code Hint
```python
# Ensure radians are used for the rotation parameter
sd.sql("SELECT ST_Rotate(geom, radians(45.0)) FROM my_table").to_arrow_table()
```

## API Test: `ST_RotateX`

### Signature
```python
ST_RotateX(arg1: geometry, rot: double) -> geometry
```
_Source: source/docs/reference/sql/st_rotatex.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_RotateX
description: Rotates a geometry around the X axis by an angle in radians.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: rot
      type: double
---

## Examples

```sql
SELECT ST_RotateX(ST_GeomFromText('POINT Z (1 0 0)'), radians(90));
```


### Goal
Rotates a 3D geometry around the X-axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry to be rotated. Must be a 3D geometry (e.g., POINT Z) for the rotation to have a meaningful effect.
- `rot` (`double`): The rotation angle in radians.

### Input
The input must be a valid SedonaDB geometry object. If using WKT, ensure the geometry includes Z-coordinates (e.g., `POINT Z (...)`) as the function operates on the X-axis rotation in 3D space.

### Output
Returns a new `geometry` object representing the input geometry rotated around the X-axis.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example: Rotating a 3D point by 90 degrees (pi/2 radians)
query = "SELECT ST_RotateX(ST_GeomFromText('POINT Z (1 0 0)'), radians(90)) as rotated_geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_RotateX(geometry, double)` as a standalone function call within the `sd.sql()` string.
- Ensure the input geometry is 3D; if the geometry is 2D, the rotation around the X-axis may not produce the expected transformation.
- Use the `radians()` SQL function to convert degrees to radians if the input angle is provided in degrees.

### Prompt Snippet
```text
ST_RotateX(geometry_column, radians(angle_in_degrees))
```

### Common Failure Modes
- **2D Geometry:** Passing a 2D geometry (e.g., `POINT (1 0)`) may result in no change or unexpected behavior, as there is no Z-component to rotate.
- **Invalid Geometry:** Passing a non-geometry type or a malformed WKT string will cause a SQL execution error.
- **Type Mismatch:** Providing a non-numeric value for the `rot` parameter will result in a type mismatch error.

### Fix Code Hint
```python
# Ensure the geometry is 3D before rotation
query = "SELECT ST_RotateX(ST_Force3D(my_geom_col), radians(45)) FROM my_table"
```

## API Test: `ST_RotateY`

### Signature
```python
ST_RotateY(arg1: geometry, rot: double) -> geometry
```
_Source: source/docs/reference/sql/st_rotatey.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_RotateY
description: Rotates a geometry around the Y axis by an angle in radians.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: rot
      type: double
---

## Examples

```sql
SELECT ST_RotateY(ST_GeomFromText('POINT Z (1 0 0)'), radians(90));
```


### Goal
Performs a 3D rotation of a geometry object around the Y-axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry to be rotated. Must be a valid geometry type (e.g., Point, LineString, Polygon).
- `rot` (`double`): The rotation angle in radians.

### Input
The function accepts standard SedonaDB geometry types. Input geometries should ideally contain Z-coordinates to meaningfully participate in a Y-axis rotation.

### Output
Returns a new `geometry` object representing the input geometry rotated around the Y-axis.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example: Rotating a 3D point by 90 degrees (pi/2 radians)
query = "SELECT ST_RotateY(ST_GeomFromText('POINT Z (1 0 0)'), radians(90)) as rotated_geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__
print(result)
```

### LLM Instruction Prompt
- Always use `ST_RotateY(geometry, double)` as a standalone SQL function call.
- Ensure the rotation angle is provided in radians; use the `radians()` SQL function if the input is in degrees.
- The input geometry must be a valid SedonaDB geometry type.

### Prompt Snippet
```text
ST_RotateY(geometry_column, radians(angle_in_degrees))
```

### Common Failure Modes
- Providing an angle in degrees instead of radians (use `radians()` to convert).
- Passing a non-geometry type as the first argument.
- Attempting to rotate 2D geometries where the Z-coordinate is missing or implicitly zero, which may result in unexpected coordinate transformations.

### Fix Code Hint
```python
# Ensure radians are used for the rotation parameter
sd.sql("SELECT ST_RotateY(geom, radians(45.0)) FROM my_table").to_arrow_table()
```

## API Test: `ST_SRID`

### Signature
```python
ST_SRID(arg1: geometry) -> integer | ST_SRID(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_srid.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_SRID
description: Returns the SRID (spatial reference identifier) of a geometry.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT ST_SRID(ST_SetSRID(ST_Point(-122.35, 47.65), 4326));
```


### Goal
Returns the integer Spatial Reference Identifier (SRID) associated with a given geometry or geography object.

### Parameters
- `arg1` (`geometry`): The geometry or geography object for which the SRID is to be retrieved.

### Input
The input must be a valid geometry or geography object. If the geometry has not been assigned an SRID, the function typically returns 0.

### Output
Returns `integer` — The SRID value representing the coordinate reference system of the input object.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using ST_SRID to verify the SRID of a point
query = "SELECT ST_SRID(ST_SetSRID(ST_Point(-122.35, 47.65), 4326)) as srid"
result = sd.sql(query).to_arrow_table()

assert result["srid"][0] == 4326
print(f"__CHECK__ SRID is {result['srid'][0]}")
```

### LLM Instruction Prompt
- Use `ST_SRID(geometry_expression)` to extract the CRS identifier.
- Ensure the input geometry has been initialized with a CRS (e.g., via `ST_SetSRID`) if a specific SRID is expected.

### Prompt Snippet
```text
ST_SRID(<geometry_column_or_expression>)
```

### Common Failure Modes
- Passing a null geometry: The function will return null.
- Uninitialized SRID: If a geometry is created without an explicit SRID, the function will return 0, which may cause logic errors in downstream spatial operations that require specific CRS matching.

### Fix Code Hint
```python
# If SRID is 0, set it explicitly before processing
sd.sql("SELECT ST_SRID(ST_SetSRID(my_geom, 4326)) FROM my_table")
```

## API Test: `ST_Scale`

### Signature
```python
ST_Scale(arg1: geometry, scaleX: double, scaleY: double) -> geometry | ST_Scale(arg1: geometry, scaleX: double, scaleY: double, scaleZ: double) -> geometry
```
_Source: source/docs/reference/sql/st_scale.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Scale
description: Scales a geometry by multiplying ordinates with scale factors.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: scaleX
      type: double
    - name: scaleY
      type: double
  - returns: geometry
    args:
    - geometry
    - name: scaleX
      type: double
    - name: scaleY
      type: double
    - name: scaleZ
      type: double
---

## Description

Scales a geometry by multiplying each coordinate by the corresponding scale factor. A 3D variant accepts `scaleZ`.

## Examples

```sql
SELECT ST_Scale(ST_GeomFromText('POINT (1 2)'), 2.0, 3.0);
```


### Goal
`ST_Scale` performs an affine transformation on a geometry by multiplying its coordinate ordinates by specified scale factors to resize the object.

### Parameters
- `arg1` (`geometry`): The input geometry to be scaled.
- `scaleX` (`double`): The factor by which to multiply the X-coordinate.
- `scaleY` (`double`): The factor by which to multiply the Y-coordinate.
- `scaleZ` (`double`): The factor by which to multiply the Z-coordinate (optional, for 3D geometries).

### Input
The input must be a valid geometry object (e.g., created via `ST_GeomFromText` or loaded from a GeoParquet file). No specific CRS preconditions are required, but the operation is applied relative to the coordinate origin (0,0,0).

### Output
Returns a new `geometry` object with coordinates transformed by the provided scale factors.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# 2D Scaling
result = sd.sql("SELECT ST_Scale(ST_GeomFromText('POINT (1 2)'), 2.0, 3.0) as geom").to_arrow_table()
print(result)
# __CHECK__ witness: The point (1, 2) scaled by (2.0, 3.0) results in (2.0, 6.0)
```

### LLM Instruction Prompt
- Use `ST_Scale` when you need to resize geometries.
- Always provide exactly two scale factors for 2D geometries or three for 3D geometries.
- Ensure the input geometry is valid before scaling.

### Prompt Snippet
```text
ST_Scale(geometry_column, scale_x, scale_y)
ST_Scale(geometry_column, scale_x, scale_y, scale_z)
```

### Common Failure Modes
- Providing only two arguments for a 3D geometry may result in unexpected behavior or errors depending on the engine's strictness regarding Z-ordinates.
- Passing non-numeric values for scale factors will cause a SQL execution error.
- Attempting to scale a null geometry will return null.

### Fix Code Hint
```python
# Ensure inputs are cast to double if they are integers
sd.sql("SELECT ST_Scale(geom, CAST(2 AS DOUBLE), CAST(3 AS DOUBLE)) FROM my_table")
```

## API Test: `ST_Segmentize`

### Signature
```python
ST_Segmentize(arg1: geometry, max_segment_length: double) -> geometry | ST_Segmentize(arg1: geography, max_segment_length: double) -> geography
```
_Source: source/docs/reference/sql/st_segmentize.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Segmentize
description: Densifies a geometry by adding intermediate points along segments that exceed a maximum length.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: max_segment_length
      type: double
  - returns: geography
    args:
    - geography
    - name: max_segment_length
      type: double
---

## Description

Densifies a geometry or geography by adding intermediate points along line segments
that exceed the specified maximum segment length. For geometry, uses planar distance; for
geography, uses the spherical approximation of the geodesic distance in meters and points
are added along the great circle arc. In both cases, points are added so that each
resulting segment is no longer than `max_segment_length`.

## Examples

```sql
SELECT ST_Segmentize(
    ST_GeomFromText('LINESTRING (0 0, 0 10)'),
    2
);
```

```sql
SELECT ST_Segmentize(
    ST_GeogFromWKT('LINESTRING (0 0, 0 10)'),
    100000
);
```


### Goal
Densifies vector geometries or geographies by inserting intermediate vertices into segments that exceed a specified maximum length threshold.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object (e.g., LineString, Polygon) to be densified.
- `max_segment_length` (`double`): The maximum allowed length for any single segment in the resulting geometry. For `geography`, this value is interpreted in meters.

### Input
- **Data:** Standard vector formats (WKT, WKB, GeoParquet).
- **Preconditions:** The input must be a valid `geometry` or `geography` type. For `geography` inputs, ensure the coordinate system supports geodesic calculations.

### Output
Returns `geography | geometry` — A new spatial object of the same type as the input, containing additional vertices such that no segment exceeds the `max_segment_length`.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_Segmentize(ST_GeomFromText('LINESTRING (0 0, 0 10)'), 2.0)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Segmentize(geometry, double)` for planar geometries or `ST_Segmentize(geography, double)` for spherical/geodesic calculations.
- Ensure the `max_segment_length` is a positive double.

### Prompt Snippet
```text
ST_Segmentize(geometry_column, max_length_double)
```

### Common Failure Modes
- **Type Mismatch:** Passing a raster object instead of a geometry/geography.
- **Invalid Length:** Providing a non-positive or non-numeric value for `max_segment_length`.
- **CRS Issues:** Attempting to perform `geography` operations on geometries without a valid CRS or in a coordinate system where geodesic distance is not defined.

### Fix Code Hint
```python
# Ensure input is cast to the correct type if necessary
sd.sql("SELECT ST_Segmentize(ST_GeomFromText('LINESTRING (0 0, 0 10)'), 2.0)").to_arrow_table()
```

## API Test: `ST_SetCRS`

### Signature
```python
ST_SetCRS(arg1: geometry, target_crs: string) -> geometry | ST_SetCRS(arg1: geography, target_crs: string) -> geography
```
_Source: source/docs/reference/sql/st_setcrs.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_SetCRS
description: Sets the Coordinate Reference System (CRS) for a geometry.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: target_crs
      type: string
  - returns: geography
    args:
    - geography
    - name: target_crs
      type: string
---

## Description

Sets a CRS to a geometry/geography. This is metadata-only - no coordinates are transformed.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

This is different from ST_Transform which actually transforms coordinates.

`target_crs` accepts an authority code (e.g. `'EPSG:4326'`), PROJJSON, or WKT.
It is stored as given and read back unchanged by `ST_CRS`.

## Examples

```sql
SELECT ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4269');
```


### Goal
Assigns a specific Coordinate Reference System (CRS) to a geometry or geography object as metadata without modifying the underlying coordinate values.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to which the CRS will be assigned.
- `target_crs` (`string`): The CRS definition to apply, provided as an authority code (e.g., `'EPSG:4326'`), PROJJSON string, or WKT string.

### Input
The input must be a valid geometry or geography object. This function is metadata-only; it does not perform coordinate reprojection. Ensure the input coordinates are already in the system defined by `target_crs`.

### Output
Returns a `geometry` or `geography` object with the updated CRS metadata.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example: Setting CRS for a point geometry
query = "SELECT ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4269') as geom"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The geometry now carries the EPSG:4269 metadata.
```

### LLM Instruction Prompt
- Use `ST_SetCRS` only when you need to define the CRS of a geometry that lacks one or to override existing metadata.
- Do not use this function to convert coordinates between systems; use `ST_Transform` for that purpose.
- Ensure the `target_crs` string is a valid EPSG code, WKT, or PROJJSON.

### Prompt Snippet
```text
ST_SetCRS(geometry_column, 'EPSG:4326')
```

### Common Failure Modes
- **Coordinate Mismatch:** Using `ST_SetCRS` to "convert" coordinates will result in incorrect spatial positioning because the function does not transform the data.
- **Invalid CRS String:** Providing a malformed CRS string (e.g., an unsupported authority code) will cause the query to fail.
- **CRS Mismatch in Operations:** If you use the resulting geometry in spatial operations (like `RS_Clip` or spatial joins) with other objects, ensure the CRSs match, or the operation may fail or produce unexpected results.

### Fix Code Hint
```python
# If you need to change the coordinate values to match a new CRS, use ST_Transform instead:
# query = "SELECT ST_Transform(ST_SetCRS(geom, 'EPSG:4326'), 'EPSG:3857') FROM table"
```

## API Test: `ST_SetSRID`

### Signature
```python
ST_SetSRID(arg1: geometry, srid: integer) -> geometry | ST_SetSRID(arg1: geography, srid: integer) -> geography
```
_Source: source/docs/reference/sql/st_setsrid.qmd:19_

### Goal
Sets the spatial reference system identifier (SRID) metadata for a geometry or geography object without performing coordinate transformation.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be tagged with an SRID.
- `srid` (`integer`): The integer identifier representing the target spatial reference system.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Preconditions:** This function modifies metadata only. It does not reproject coordinates. If coordinate transformation is required, use `ST_Transform` instead. Ensure the SRID provided corresponds to a valid coordinate reference system supported by the engine.

### Output
Returns `geography | geometry` — The input object with the updated SRID metadata.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Basic usage
result = sd.sql("SELECT ST_SetSRID(ST_Point(-122.35, 47.65), 4326)").to_arrow_table()
print(result)
# __CHECK__

# Usage within rasterization workflows
query = """
SELECT RS_AsRaster(
  ST_SetSRID(ST_GeomFromText('POLYGON ((10 10, 50 10, 50 50, 10 50, 10 10))'), 4326),
  RS_MakeEmptyRaster(1, 'uint8', 400, 400, ST_MakeEnvelope(0, 0, 400, 400, 4326)),
  'uint8', false, 1.0, 0.0, false
)
"""
result = sd.sql(query).to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_SetSRID` when you need to define the CRS of a geometry that lacks one or to override existing metadata.
- Do not use this function to reproject data; it is strictly for metadata assignment.
- Ensure the `srid` argument is an integer.

### Prompt Snippet
```text
ST_SetSRID(geometry_column, 4326)
```

### Common Failure Modes
- **Coordinate Mismatch:** Using `ST_SetSRID` to "reproject" data will result in incorrect spatial calculations because the underlying coordinates remain unchanged.
- **Invalid SRID:** Providing an SRID that is not recognized by the system may lead to errors in subsequent spatial operations that rely on CRS metadata.
- **Type Mismatch:** Passing a non-integer value to the `srid` parameter will cause a SQL execution error.

### Fix Code Hint
```python
# If you need to change the actual coordinates to a new CRS, use ST_Transform:
# sd.sql("SELECT ST_Transform(geom, 3857) FROM table")
```

## API Test: `ST_Simplify`

### Signature
```python
ST_Simplify(arg1: geometry, tolerance: double) -> geometry | ST_Simplify(arg1: geography, tolerance: double) -> geography
```
_Source: source/docs/reference/sql/st_simplify.qmd:19_

### Goal
Simplifies an input geometry or geography using the Douglas-Peucker algorithm to reduce the number of vertices while maintaining the general shape.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be simplified.
- `tolerance` (`double`): The distance tolerance for the Douglas-Peucker algorithm; higher values result in greater simplification. For `geography` types, this value is interpreted in meters.

### Input
- **Data:** Accepts `geometry` or `geography` types.
- **Preconditions:** The input must be a valid spatial object. Note that this function does not guarantee the topological validity of the output; if validity must be preserved, use `ST_SimplifyPreserveTopology` instead.

### Output
Returns `geography | geometry` representing the simplified version of the input object.

### Valid Call Patterns
```python
# Example inferred from SQL documentation
import sedona.db
sd = sedona.db.connect()

# Geometry simplification
result = sd.sql("SELECT ST_Simplify(ST_GeomFromWKT('LINESTRING (0 0, 0.5 0.1, 1 0, 1.5 0.1, 2 0)'), 0.2)").to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a simplified LINESTRING geometry.
```

### LLM Instruction Prompt
- Use `ST_Simplify` when you need to reduce the vertex count of a geometry or geography.
- Always specify the `tolerance` as a `double`.
- Remember that for `geography` types, the `tolerance` is in meters.
- If the output must remain topologically valid, do not use this function; use `ST_SimplifyPreserveTopology` instead.

### Prompt Snippet
```text
ST_Simplify(geometry_column, tolerance_value)
```

### Common Failure Modes
- **Invalid Geometry:** The output of `ST_Simplify` may result in self-intersecting or otherwise invalid geometries.
- **Type Mismatch:** Providing a non-numeric value for the `tolerance` parameter will cause a query execution error.
- **Geography vs Geometry:** Confusing the units of `tolerance` (meters for `geography` vs. coordinate units for `geometry`) can lead to unexpected simplification results.

### Fix Code Hint
```python
# If the output geometry is invalid, switch to the topology-preserving version:
sd.sql("SELECT ST_SimplifyPreserveTopology(geom, 0.2) FROM table").to_arrow_table()
```

## API Test: `ST_SimplifyPreserveTopology`

### Signature
```python
ST_SimplifyPreserveTopology(arg1: geometry, tolerance: double) -> geometry
```
_Source: source/docs/reference/sql/st_simplifypreservetopology.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_SimplifyPreserveTopology
description: Simplifies a geometry, ensuring the result is a valid geometry with the same topology.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: tolerance
      type: double
---

## Description

Simplifies a geometry, ensuring that the result is a valid geometry having the same dimension and number of components as the input, and with the components having the same topological relationship.

## Examples

```sql
SELECT ST_SimplifyPreserveTopology(
    ST_GeomFromText('POLYGON((8 25, 28 22, 28 20, 15 11, 33 3, 56 30, 46 33,46 34, 47 44, 35 36, 45 33, 43 19, 29 21, 29 22,35 26, 24 39, 8 25))'),
    10
);
```


### Goal
Reduces the number of vertices in a geometry while maintaining its original topological structure and validity.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Polygon, LineString) to be simplified.
- `tolerance` (`double`): The distance tolerance for simplification; vertices closer than this distance may be removed, provided the topology is preserved.

### Input
- **Data:** Geometry objects provided via WKT, WKB, or GeoParquet.
- **Preconditions:** The input must be a valid geometry type.

### Output
Returns `geometry` — A simplified version of the input geometry that retains the same dimension, number of components, and topological relationships.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = """
SELECT ST_SimplifyPreserveTopology(
    ST_GeomFromText('POLYGON((8 25, 28 22, 28 20, 15 11, 33 3, 56 30, 46 33,46 34, 47 44, 35 36, 45 33, 43 19, 29 21, 29 22,35 26, 24 39, 8 25))'),
    10.0
) as simplified_geom
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_SimplifyPreserveTopology(geometry, double)` as a standalone SQL function.
- Ensure the tolerance is provided as a numeric double.
- The function is intended for vector geometries; do not attempt to pass raster objects directly.

### Prompt Snippet
```text
ST_SimplifyPreserveTopology(geometry_column, 5.0)
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or string) as the first argument.
- Providing a negative value for the `tolerance` parameter.
- Attempting to simplify geometries that are already too simple to be reduced further under the given tolerance.

### Fix Code Hint
```python
# Ensure the input is a valid geometry and tolerance is positive
query = "SELECT ST_SimplifyPreserveTopology(geom, 1.0) FROM my_table WHERE geom IS NOT NULL"
```

## API Test: `ST_Snap`

### Signature
```python
ST_Snap(arg1: geometry, arg2: geometry, tolerance: double) -> geometry
```
_Source: source/docs/reference/sql/st_snap.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Snap
description: Snaps input geometry to reference geometry within tolerance.
kernels:
  - returns: geometry
    args:
    - geometry
    - geometry
    - name: tolerance
      type: double
---

## Description

Snaps input geometry to reference geometry within tolerance. Result geometry aligns to the reference geometry where within tolerance distance.

## Examples

```sql
SELECT ST_Snap(
    ST_GeomFromWKT('POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_GeomFromWKT('LINESTRING(0 0.1, 1 0.1)'),
    0.2
);
```


### Goal
`ST_Snap` modifies the vertices of an input geometry to align with the vertices or segments of a reference geometry if they fall within a specified distance threshold.

### Parameters
- `arg1` (`geometry`): The input geometry to be modified.
- `arg2` (`geometry`): The reference geometry to which the input geometry will be snapped.
- `tolerance` (`double`): The maximum distance between vertices of the input geometry and the reference geometry for snapping to occur.

### Input
The function accepts standard SedonaDB geometry types (e.g., Point, LineString, Polygon) created via constructors like `ST_GeomFromWKT` or loaded from GeoParquet. Both geometries should ideally share the same coordinate reference system (CRS) to ensure the `tolerance` distance is interpreted correctly.

### Output
Returns a `geometry` object where vertices of `arg1` have been moved to match the geometry of `arg2` if they are within the `tolerance` distance.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example inferred from signature and documentation
query = """
SELECT ST_Snap(
    ST_GeomFromWKT('POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_GeomFromWKT('LINESTRING(0 0.1, 1 0.1)'),
    0.2
) as snapped_geom
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__", result)
```

### LLM Instruction Prompt
- Always use `ST_Snap(arg1, arg2, tolerance)` within a `sd.sql()` call.
- Ensure `arg1` and `arg2` are valid geometry objects.
- The `tolerance` must be a numeric double value.

### Prompt Snippet
```text
ST_Snap(geometry_input, geometry_reference, tolerance_double)
```

### Common Failure Modes
- **CRS Mismatch:** If `arg1` and `arg2` are in different coordinate systems, the snapping distance may be calculated incorrectly or result in an error.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT strings will cause the query to fail.
- **Tolerance too small:** If the `tolerance` is set to 0 or a value smaller than the precision of the coordinates, no snapping will occur.

### Fix Code Hint
```python
# Ensure both geometries are in the same CRS before snapping
query = """
SELECT ST_Snap(
    ST_Transform(geom1, 'EPSG:3857'),
    ST_Transform(geom2, 'EPSG:3857'),
    0.5
) FROM my_table
"""
```

## API Test: `ST_StartPoint`

### Signature
```python
ST_StartPoint(arg1: geometry) -> geometry | ST_StartPoint(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_startpoint.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_StartPoint
description: Returns the start point of a linestring geometry.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Examples

```sql
SELECT ST_StartPoint(
  ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)')
);
```


### Goal
Extracts the first coordinate (Point) from a provided Linestring geometry or geography object.

### Parameters
- `arg1` (`geometry`): A Linestring geometry or geography object from which the initial vertex is to be extracted.

### Input
- **Data:** Linestring geometries or geographies.
- **Formats:** WKT, WKB, or GeoParquet-loaded geometries.
- **Preconditions:** The input must be a valid Linestring type.

### Output
Returns `geography | geometry` — A Point object representing the first coordinate of the input Linestring.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = "SELECT ST_StartPoint(ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)')) as start_point"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a Point geometry (100, 150).
```

### LLM Instruction Prompt
- Use `ST_StartPoint(arg1)` to retrieve the first vertex of a Linestring.
- Ensure the input is a valid Linestring geometry; passing non-linestring types may result in errors.
- The function preserves the coordinate system (CRS) of the input.

### Prompt Snippet
```text
ST_StartPoint(geometry_column)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a Point, Polygon, or MultiPolygon instead of a Linestring will cause the function to fail or return null.
- **Null Input:** Passing a null geometry will return null.

### Fix Code Hint
```python
# Ensure the input is a Linestring before calling ST_StartPoint
query = "SELECT ST_StartPoint(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_LineString'"
```

## API Test: `ST_SymDifference`

### Signature
```python
ST_SymDifference(arg1: geometry, arg2: geometry) -> geometry | ST_SymDifference(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_symdifference.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_SymDifference
description: Returns the parts of geometries or geographies A and B that do not overlap.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Examples

```sql
SELECT ST_SymDifference(
    ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
);
```

```sql
SELECT ST_SymDifference(
    ST_GeogFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
);
```


### Goal
Computes the symmetric difference of two spatial objects, returning a new geometry or geography containing only the areas that do not overlap between the two inputs.

### Parameters
- `arg1` (`geometry`): The first input geometry or geography object.
- `arg2` (`geometry`): The second input geometry or geography object to compare against the first.

### Input
- **Data Types:** Accepts `geometry` or `geography` types.
- **Preconditions:** Both inputs must be of the same type (both `geometry` or both `geography`). If using `geometry`, ensure both inputs share the same Coordinate Reference System (CRS) to avoid projection errors.

### Output
Returns `geography | geometry` — A new spatial object representing the symmetric difference (the union of the two inputs minus their intersection).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = """
SELECT ST_SymDifference(
    ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
) as sym_diff
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (geometry or geography).
- When using geometry, verify that both inputs are in the same CRS.
- Use `ST_GeomFromWKT` or `ST_GeogFromWKT` to construct inputs if they are provided as WKT strings.

### Prompt Snippet
```text
ST_SymDifference(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other will result in an execution error.
- **CRS Mismatch:** Providing geometries with different coordinate systems will cause the operation to fail.
- **Invalid WKT:** Passing malformed WKT strings to the constructor functions will raise a parsing error.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type and CRS
sd.sql("SELECT ST_SymDifference(ST_Transform(geom1, 4326), ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `ST_TessellateGeog`

### Signature
```python
ST_TessellateGeog(arg1: geometry, tolerance: double) -> geography
```
_Source: source/docs/reference/sql/st_tessellategeog.qmd:19_

### Goal
Converts a planar `geometry` to a `geography` type by densifying edges into geodesic segments to accurately represent paths on a sphere.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., LINESTRING) to be converted.
- `tolerance` (`double`): The maximum allowed deviation in meters between the original planar edge and the resulting geodesic approximation.

### Input
- **Data:** Accepts standard geometry types (e.g., WKT, WKB).
- **Preconditions:** The input must be a valid `geometry`. The `tolerance` must be a positive `double` representing meters.

### Output
Returns `geography` — A representation of the input geometry where edges are interpolated as geodesic segments on a sphere.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example inferred from SQL documentation
query = """
SELECT ST_TessellateGeog(
    ST_GeomFromText('LINESTRING (-10 50, 10 50)'),
    1000
)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_TessellateGeog` when you need to convert planar geometries to spherical coordinates while maintaining geodesic accuracy.
- Always provide the `tolerance` as a `double` representing the maximum deviation in meters.
- Ensure the input is a valid `geometry` object.

### Prompt Snippet
```text
ST_TessellateGeog(geometry_column, tolerance_value)
```

### Common Failure Modes
- **Invalid Tolerance:** Providing a non-numeric or negative value for `tolerance` will result in a runtime error.
- **Invalid Geometry:** Passing a non-geometry type or malformed WKT will cause the function to fail.
- **Type Mismatch:** Attempting to use this function on an object that is already a `geography` type rather than a `geometry` type.

### Fix Code Hint
```python
# Ensure tolerance is a float and geometry is valid
sd.sql("SELECT ST_TessellateGeog(ST_GeomFromText('LINESTRING(0 0, 1 1)'), 500.0)").to_arrow_table()
```

## API Test: `ST_TessellateGeom`

### Signature
```python
ST_TessellateGeom(arg1: geography, tolerance: double) -> geometry
```
_Source: source/docs/reference/sql/st_tessellategeom.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_TessellateGeom
description: Converts a geography to geometry, densifying edges to approximate geodesic segments as planar lines.

kernels:
  - returns: geometry
    args:
    - geography
    - name: tolerance
      type: double
---

## Description

Converts a geography to a geometry while tessellating edges using spherical (geodesic)
interpolation. The tolerance parameter specifies the maximum deviation in meters
between the original spherical edge and the tessellated planar approximation. This is
useful when converting geographies to geometries while preserving the shape of
geodesic edges as approximated line segments.

When an edge crosses the antimeridian, the returned coordinates are "wrapped" such that
the returned longitude is either greater than 180 or less than -180. This helps produce
valid geometry in more cases (e.g., linestring or polygon crossing the antimeridian);
however, it does not produce valid geometry in the case of a polygon ring that spans
the full longitude range (e.g., Antarctica).

## Examples

```sql
SELECT ST_TessellateGeom(
    ST_GeogFromText('LINESTRING (-10 50, 10 50)'),
    1000
);
```

When a linestring crosses the antimeridian, coordinates are wrapped to keep the
geometry continuous:

```sql
SELECT ST_TessellateGeom(
    ST_GeogFromText('LINESTRING (170 70, -175 70, -175 60, 170 60)'),
    1000
);
```


### Goal
Converts a `geography` object into a `geometry` object by densifying edges to approximate geodesic curves as planar line segments.

### Parameters
- `arg1` (`geography`): The input geography object to be converted.
- `tolerance` (`double`): The maximum allowed deviation in meters between the original spherical edge and the resulting planar approximation.

### Input
- **Data:** Requires a valid `geography` type input.
- **Preconditions:** The input must be a valid geography. The function is typically used to prepare data for planar operations while maintaining the spatial accuracy of geodesic paths.

### Output
Returns `geometry` — A planar geometry representation where edges have been densified to approximate the original geodesic path within the specified tolerance.

### Valid Call Patterns
```python
# Example of calling ST_TessellateGeom via the Python SQL interface
query = """
SELECT ST_TessellateGeom(
    ST_GeogFromText('LINESTRING (-10 50, 10 50)'),
    1000.0
) as geom
"""
result = sd.sql(query).to_arrow_table()
# __CHECK__ witness: result contains the tessellated geometry
```

### LLM Instruction Prompt
- Always use `ST_TessellateGeom(geography_input, tolerance_double)` in SQL queries.
- Ensure the tolerance is provided as a double (e.g., `1000.0`).
- Remember that this function is specifically for converting `geography` to `geometry`.

### Prompt Snippet
```text
ST_TessellateGeom(geography_col, 1000.0)
```

### Common Failure Modes
- **Invalid Geometry:** Providing a polygon ring that spans the full longitude range (e.g., Antarctica) will not produce valid geometry.
- **Type Mismatch:** Passing a `geometry` type instead of a `geography` type as the first argument.
- **Tolerance Issues:** Setting an extremely small tolerance may result in an excessive number of vertices, potentially impacting performance.

### Fix Code Hint
```python
# Ensure input is cast to geography if it is currently a geometry
query = "SELECT ST_TessellateGeom(ST_GeographyFromText(ST_AsText(geom_col)), 1000.0) FROM table"
```

## API Test: `ST_ToGeography`

### Signature
```python
ST_ToGeography(arg1: geometry) -> geography | ST_ToGeography(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_togeography.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ToGeography
description: Converts a geometry to a geography by changing the edge interpretation to spherical.
kernels:
  - returns: geography
    args: [geometry]
  - returns: geography
    args: [geography]
---

## Description

Converts a geometry to a geography by changing the edge interpretation from planar
to spherical. This is a metadata-only operation that does not modify the underlying
coordinate data. If the input is already a geography, it is returned unchanged.

## Examples

```sql
SELECT ST_ToGeography(ST_GeomFromText('POINT (-122.35 47.62)'));
```


### Goal
Converts a planar `geometry` object into a `geography` object by updating its edge interpretation to spherical for accurate geodesic calculations.

### Parameters
- `arg1` (`geometry`): The input geometry (or existing geography) to be converted.

### Input
- **Data:** Accepts standard geometry types (e.g., Point, Polygon) created via constructors like `ST_GeomFromText`.
- **Preconditions:** The input must be a valid geometry object. This is a metadata-only operation; coordinate data remains unchanged.

### Output
Returns `geography` — A geography object representing the same spatial data as the input, but interpreted with spherical edges.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and SQL example
query = "SELECT ST_ToGeography(ST_GeomFromText('POINT (-122.35 47.62)'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result is a geography object.
```

### LLM Instruction Prompt
- Use `ST_ToGeography` when you need to perform spherical calculations on a geometry that is currently defined in a planar coordinate system.
- Do not attempt to modify the coordinate values; this function only changes the metadata interpretation.

### Prompt Snippet
```text
ST_ToGeography(geometry_column)
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or string) will result in a SQL execution error.
- Expecting the function to reproject coordinates; it does not perform coordinate transformation, only changes the edge interpretation.

### Fix Code Hint
```python
# Ensure the input is a valid geometry before conversion
sd.sql("SELECT ST_ToGeography(ST_GeomFromText('POINT(0 0)'))").to_arrow_table()
```

## API Test: `ST_ToGeometry`

### Signature
```python
ST_ToGeometry(arg1: geometry) -> geometry | ST_ToGeometry(arg1: geography) -> geometry
```
_Source: source/docs/reference/sql/st_togeometry.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ToGeometry
description: Converts a geography to a geometry by changing the edge interpretation to planar.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args: [geography]
---

## Description

Converts a geography to a geometry by changing the edge interpretation from spherical
to planar. This is a metadata-only operation that does not modify the underlying
coordinate data. If the input is already a geometry, it is returned unchanged.

## Examples

```sql
SELECT ST_ToGeometry(ST_GeogFromText('POINT (-122.35 47.62)'));
```


### Goal
`ST_ToGeometry` performs a metadata-only conversion of a geography object into a geometry object by reinterpreting spherical edges as planar edges.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to be converted. If the input is already a geometry, it is returned as-is.

### Input
The function accepts either a `geography` type (spherical interpretation) or a `geometry` type (planar interpretation). No specific file format is required as input, provided the object is already loaded into the SedonaDB session as a valid spatial type.

### Output
Returns a `geometry` object representing the same coordinate data as the input, but with planar edge interpretation.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from SQL example:
query = "SELECT ST_ToGeometry(ST_GeogFromText('POINT (-122.35 47.62)'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_ToGeometry` when you need to perform planar spatial operations (like `ST_Distance` or `ST_Intersects`) on data that is currently stored or defined as `geography`.
- Do not attempt to modify coordinate values; this is a metadata-only operation.

### Prompt Snippet
```text
Use ST_ToGeometry(arg1) to cast a geography object to a geometry object for planar analysis.
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) that cannot be implicitly cast to a geography or geometry.
- Expecting a coordinate transformation; remember that this function only changes the edge interpretation metadata, not the underlying coordinate values.

### Fix Code Hint
```python
# Ensure the input is a valid spatial type before calling
sd.sql("SELECT ST_ToGeometry(ST_GeogFromText('POINT(0 0)'))").to_arrow_table()
```

## API Test: `ST_Touches`

### Signature
```python
ST_Touches(arg1: geometry, arg2: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_touches.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Touches
description: Returns true if A touches B.
kernels:
  - returns: boolean
    args: [geometry, geometry]
---

## Description

Returns `true` if the two geometries have at least one boundary point in common but no interior points in common.

## Examples

```sql
SELECT ST_Touches(
    ST_GeomFromText('POINT(1 2)'),
    ST_GeomFromText('POLYGON((1 1, 1 4, 4 4, 4 1, 1 1))')
);
```


### Goal
Determines if two geometries share boundary points without overlapping their interior regions, a fundamental check for spatial adjacency in vector analysis.

### Parameters
- `arg1` (`geometry`): The first geometry object to evaluate.
- `arg2` (`geometry`): The second geometry object to evaluate.

### Input
Requires two valid geometry objects (e.g., Point, LineString, Polygon). Inputs are typically provided as WKT, WKB, or GeoParquet-loaded geometries. Ensure both geometries are in the same Coordinate Reference System (CRS) before comparison.

### Output
Returns `boolean` — `true` if the geometries touch, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred from signature and documentation examples
query = """
SELECT ST_Touches(
    ST_GeomFromText('POINT(1 2)'),
    ST_GeomFromText('POLYGON((1 1, 1 4, 4 4, 4 1, 1 1))')
) as touches
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Touches(geom1, geom2)` to verify if two geometries share a boundary.
- Ensure both inputs are valid geometry types.
- If inputs are from different sources, verify they share the same CRS to avoid unexpected results.

### Prompt Snippet
```text
ST_Touches(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** Comparing geometries in different coordinate systems may lead to incorrect results or errors.
- **Invalid Geometry:** Passing malformed WKT or non-geometry types will cause the query to fail.
- **Interior Overlap:** If the geometries share any interior points (e.g., one polygon is inside another), the function will return `false` even if they share boundary points.

### Fix Code Hint
```python
# Ensure geometries are in the same CRS before calling ST_Touches
query = """
SELECT ST_Touches(geom1, ST_Transform(geom2, 'EPSG:4326')) 
FROM my_table
"""
```

## API Test: `ST_Transform`

### Signature
```python
ST_Transform(arg1: geometry, target_crs: string) -> geometry | ST_Transform(arg1: geometry, source_crs: string, target_crs: string) -> geometry | ST_Transform(arg1: geography, target_crs: string) -> geography | ST_Transform(arg1: geography, source_crs: string, target_crs: string) -> geography
```
_Source: source/docs/reference/sql/st_transform.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Transform
description: Transforms a geometry from one coordinate reference system to another.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: target_crs
      type: string
  - returns: geometry
    args:
    - geometry
    - name: source_crs
      type: string
    - name: target_crs
      type: string
  - returns: geography
    args:
    - geography
    - name: target_crs
      type: string
  - returns: geography
    args:
    - geography
    - name: source_crs
      type: string
    - name: target_crs
      type: string
---

## Description

Transforms the coordinate reference system of a geometry between EPSG reference systems. If provided with 2 arguments, target_crs can be specified as an EPSG code (e.g., 'EPSG:4326') or as a PROJ string.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

When 3 arguments are provided, a source_crs can be used to specify the source CRS in case the input geometry doesn't have an SRID defined. The source_crs takes precedence over the geometry's SRID.

Geography input is also accepted; because geography is always defined on a geographic (longitude/latitude) CRS, the target_crs must be geographic as well.

## Examples

```sql
SELECT ST_Transform(ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4326'), 'EPSG:26910');
```


### Goal
`ST_Transform` converts the coordinate reference system (CRS) of a geometry or geography object to a specified target CRS.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be transformed.
- `source_crs` (`string`): The EPSG code or PROJ string defining the current CRS of the input. This overrides any existing SRID on the geometry.
- `target_crs` (`string`): The EPSG code or PROJ string defining the desired output CRS.

### Input
The input must be a valid geometry or geography object. If the input lacks an SRID, a `source_crs` must be provided. For geography inputs, the `target_crs` must be a geographic (longitude/latitude) CRS.

### Output
Returns a `geometry` or `geography` object transformed into the coordinate system specified by `target_crs`.

### Valid Call Patterns
```python
# Example using 2 arguments (geometry with existing SRID)
sd.sql("SELECT ST_Transform(ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4326'), 'EPSG:26910')").to_arrow_table()

# Example using 3 arguments (explicit source CRS)
sd.sql("SELECT ST_Transform(ST_Point(-122.35, 47.65), 'EPSG:4326', 'EPSG:26910')").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql("...").to_arrow_table()` for execution.
- If the input geometry does not have an SRID, you must use the 3-argument version of `ST_Transform` to specify the `source_crs`.
- Ensure `target_crs` is a valid EPSG code or PROJ string.

### Prompt Snippet
```text
ST_Transform(geometry_expression, 'EPSG:XXXX')
ST_Transform(geometry_expression, 'EPSG:YYYY', 'EPSG:XXXX')
```

### Common Failure Modes
- Providing a non-geographic CRS as the `target_crs` when the input is a `geography` type.
- Omitting the `source_crs` when the input geometry does not have an SRID defined, leading to transformation errors.
- Using invalid or unsupported EPSG codes/PROJ strings.

### Fix Code Hint
```python
# If transformation fails due to missing SRID, explicitly define it:
sd.sql("SELECT ST_Transform(ST_GeomFromText('POINT(-122.35 47.65)'), 'EPSG:4326', 'EPSG:26910')")
```

## API Test: `ST_Translate`

### Signature
```python
ST_Translate(arg1: geometry, deltaX: double, deltaY: double) -> geometry | ST_Translate(arg1: geometry, deltaX: double, deltaY: double, deltaZ: double) -> geometry
```
_Source: source/docs/reference/sql/st_translate.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Translate
description: Returns a geometry with coordinates translated by deltaX and deltaY.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: deltaX
      type: double
    - name: deltaY
      type: double
  - returns: geometry
    args:
    - geometry
    - name: deltaX
      type: double
    - name: deltaY
      type: double
    - name: deltaZ
      type: double
---

## Description

Translates (shifts) a geometry by the given deltaX and deltaY offsets. A 3D variant accepts `deltaZ`.

## Examples

```sql
SELECT ST_Translate(ST_GeomFromText('POINT(1 3)'), 1, 2);
```


### Goal
Shifts the coordinates of a geometry object by specified offsets along the X, Y, and optionally Z axes.

### Parameters
- `arg1` (`geometry`): The input geometry to be translated.
- `deltaX` (`double`): The distance to shift the geometry along the X-axis.
- `deltaY` (`double`): The distance to shift the geometry along the Y-axis.
- `deltaZ` (`double`): The distance to shift the geometry along the Z-axis (optional, for 3D translation).

### Input
The function accepts standard geometry types (e.g., Point, Polygon) as input. The geometry must be valid within the SedonaDB engine.

### Output
Returns a new `geometry` object with coordinates shifted by the provided delta values.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# 2D Translation
result = sd.sql("SELECT ST_Translate(ST_GeomFromText('POINT(1 3)'), 1, 2) as geom").to_arrow_table()
print(result)
# __CHECK__ witness: POINT(2 5)

# 3D Translation
result_3d = sd.sql("SELECT ST_Translate(ST_GeomFromText('POINT Z (1 3 5)'), 1, 2, 3) as geom").to_arrow_table()
print(result_3d)
# __CHECK__ witness: POINT Z (2 5 8)
```

### LLM Instruction Prompt
- Use `ST_Translate` to shift geometries.
- Ensure the number of arguments matches the desired dimensionality (2 or 3).
- The function is a scalar SQL function; use it within a `SELECT` statement.

### Prompt Snippet
```text
ST_Translate(geometry_column, deltaX, deltaY)
ST_Translate(geometry_column, deltaX, deltaY, deltaZ)
```

### Common Failure Modes
- Providing only two arguments when attempting to translate a 3D geometry may result in the Z-coordinate remaining unchanged or unexpected behavior depending on the geometry type.
- Passing non-numeric values for delta parameters will cause a type mismatch error.
- Attempting to translate a null geometry will return null.

### Fix Code Hint
```python
# Ensure all delta parameters are cast to double if coming from integer variables
delta_x = 1.0
delta_y = 2.0
query = f"SELECT ST_Translate(geom, {delta_x}, {delta_y}) FROM my_table"
```

## API Test: `ST_UnaryUnion`

### Signature
```python
ST_UnaryUnion(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_unaryunion.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_UnaryUnion
description: Returns a single geometry which is the union of all components.
kernels:
  - returns: geometry
    args: [geometry]
---

## Description

Returns a single geometry which is the union of all components of the input geometry. Useful for computing the union of a set of geometries stored in a single geometry (e.g., a GEOMETRYCOLLECTION or MULTI* geometry).

## Examples

```sql
SELECT ST_UnaryUnion(ST_GeomFromWKT('MULTIPOLYGON(((0 0, 10 0, 10 10, 0 10, 0 0)), ((10 0, 20 0, 20 10, 10 10, 10 0)))'));
```


### Goal
`ST_UnaryUnion` computes the geometric union of all components within a single input geometry, effectively dissolving boundaries between overlapping or adjacent parts.

### Parameters
- `arg1` (`geometry`): A geometry object containing multiple components, such as a `MULTIPOLYGON`, `MULTILINESTRING`, or `GEOMETRYCOLLECTION`.

### Input
- **Data:** Accepts standard geometry types (WKT, WKB, or GeoParquet-loaded geometries).
- **Preconditions:** The input must be a valid geometry type. If the input is a simple geometry (e.g., a single `POLYGON`), the function returns the input geometry itself.

### Output
Returns `geometry` — A single, merged geometry representing the union of all input components.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using WKT input
query = "SELECT ST_UnaryUnion(ST_GeomFromWKT('MULTIPOLYGON(((0 0, 10 0, 10 10, 0 10, 0 0)), ((10 0, 20 0, 20 10, 10 10, 10 0)))')) as unioned_geom"
result = sd.sql(query).to_arrow_table()

# __CHECK__ witness
print(result)
```

### LLM Instruction Prompt
- Always use `ST_UnaryUnion` when you need to dissolve internal boundaries of a `MULTI*` geometry or `GEOMETRYCOLLECTION`.
- Ensure the input is a valid geometry type; if the input is a string, wrap it in a constructor like `ST_GeomFromWKT`.

### Prompt Snippet
```text
ST_UnaryUnion(geometry_column)
```

### Common Failure Modes
- **Invalid Geometry:** Passing a non-geometry type or a malformed WKT string will result in a SQL execution error.
- **Empty Input:** Passing a null or empty geometry may return null or an empty geometry depending on the underlying engine state.

### Fix Code Hint
```python
# Ensure input is cast to geometry if coming from a raw string
sd.sql("SELECT ST_UnaryUnion(ST_GeomFromWKT(my_wkt_col)) FROM my_table")
```

## API Test: `ST_Union`

### Signature
```python
ST_Union(arg1: geometry, arg2: geometry) -> geometry | ST_Union(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_union.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Union
description: Returns a geometry or geography that represents the point set union of two geometries or geographies.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Examples

```sql
SELECT ST_Union(
    ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
);
```

```sql
SELECT ST_Union(
    ST_GeogFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeogFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
);
```


### Goal
`ST_Union` computes the spatial union of two input geometries or geographies, resulting in a single object representing the combined point set.

### Parameters
- `arg1` (`geometry`): The first geometry or geography object to be combined.
- `arg2` (`geometry`): The second geometry or geography object to be combined.

### Input
The function accepts two geometries (WKT, WKB, or GeoParquet-derived) or two geographies. Inputs must be of the same type (both geometry or both geography). If using geometries, they should share the same Coordinate Reference System (CRS).

### Output
Returns a `geometry` or `geography` object representing the union of the two inputs.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
query = """
SELECT ST_Union(
    ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
) as union_geom
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (geometry or geography).
- Do not mix geometry and geography types in a single `ST_Union` call.
- Use `ST_GeomFromWKT` or similar constructors to prepare inputs if they are provided as strings.

### Prompt Snippet
```text
ST_Union(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in an execution error.
- **CRS Mismatch:** Providing geometries with different Coordinate Reference Systems may lead to errors or undefined behavior; ensure inputs are reprojected to a common CRS if necessary.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type/CRS
sd.sql("SELECT ST_Union(ST_Transform(geom1, 4326), ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `ST_Union_Agg`

### Signature
```python
ST_Union_Agg(arg1: geometry) -> geometry
```
_Source: source/docs/reference/sql/st_union_agg.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Union_Agg
description: Returns a geometry that represents the point set union of all geometries.
kernels:
  - returns: geometry
    args: [geometry]
---

## Examples

```sql
SELECT ST_Union_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('POLYGON ((0 0, 0 2, 2 2, 2 0, 0 0))')),
    (ST_GeomFromText('POLYGON ((1 1, 1 3, 3 3, 3 1, 1 1))'))
) AS t(geom);
```


### Goal
Computes the spatial union of a set of geometries, resulting in a single geometry representing the combined point set of all input geometries.

### Parameters
- `arg1` (`geometry`): A column or expression containing the geometries to be aggregated.

### Input
- **Data:** Accepts standard geometry types (Point, Polygon, etc.) provided as columns in a table or subquery.
- **Preconditions:** All geometries in the input set should share the same Coordinate Reference System (CRS) to ensure valid spatial union operations.

### Output
Returns `geometry` — A single geometry object representing the union of all input geometries.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using the SQL interface to aggregate geometries
query = """
SELECT ST_Union_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('POLYGON ((0 0, 0 2, 2 2, 2 0, 0 0))')),
    (ST_GeomFromText('POLYGON ((1 1, 1 3, 3 3, 3 1, 1 1))'))
) AS t(geom)
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result contains a single geometry representing the union.
```

### LLM Instruction Prompt
- Use `ST_Union_Agg` as an aggregate function within a `SELECT` statement.
- Ensure the input column contains valid geometry types.
- Do not attempt to use this function on raster types; it is strictly for vector geometry aggregation.

### Prompt Snippet
```text
ST_Union_Agg(geometry_column)
```

### Common Failure Modes
- **CRS Mismatch:** Attempting to union geometries with different CRSs may lead to unexpected results or errors.
- **Empty Input:** If the input set is empty, the function may return `NULL`.
- **Invalid Geometries:** Input geometries that are malformed or self-intersecting in ways that violate OGC standards may cause the aggregation to fail.

### Fix Code Hint
```python
# Ensure all geometries are in the same CRS before aggregation
sd.sql("SELECT ST_Union_Agg(ST_Transform(geom, 'EPSG:4326')) FROM my_table")
```

## API Test: `ST_Within`

### Signature
```python
ST_Within(arg1: geometry, arg2: geometry) -> boolean | ST_Within(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_within.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Within
description: Returns true if A is completely inside B.
kernels:
  - returns: boolean
    args: [geometry, geometry]
  - returns: boolean
    args: [geography, geography]
---

## Description

Returns `true` if geometry A is completely inside geometry B. This is the inverse
of `ST_Contains`: `ST_Within(A, B)` is equivalent to `ST_Contains(B, A)`.

## Examples

```sql
SELECT ST_Within(
    ST_GeomFromWKT('POINT (1 1)'),
    ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))')
);
```

```sql
SELECT ST_Within(
    ST_GeogFromWKT('POINT (1 1)'),
    ST_GeogFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))')
);
```


### Goal
Determines if a spatial object (geometry or geography) is entirely contained within the boundary of another spatial object.

### Parameters
- `arg1` (`geometry`): The subject geometry or geography to test for containment.
- `arg2` (`geometry`): The target geometry or geography that must fully enclose `arg1`.

### Input
- **Data Formats:** Accepts WKT, WKB, or registered views containing geometry/geography columns.
- **Preconditions:** Both `arg1` and `arg2` must be of the same type (both `geometry` or both `geography`). If using `geometry`, both inputs must share the same Coordinate Reference System (CRS).

### Output
Returns `boolean` — `true` if `arg1` is completely inside `arg2`, otherwise `false`.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("""
    SELECT ST_Within(
        ST_GeomFromWKT('POINT (1 1)'),
        ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))')
    ) as is_within
""").to_arrow_table()

print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (geometry or geography).
- If using geometry, verify that both inputs are in the same CRS to avoid runtime errors.
- Use `ST_Within(A, B)` as the inverse of `ST_Contains(B, A)`.

### Prompt Snippet
```text
ST_Within(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** Providing geometries with different CRSs will cause the operation to fail.
- **Type Mismatch:** Attempting to compare a `geometry` type with a `geography` type.
- **Invalid WKT:** Passing malformed WKT strings to the constructor functions.

### Fix Code Hint
```python
# Ensure CRS alignment before comparison
sd.sql("SELECT ST_Within(geom_a, ST_Transform(geom_b, 4326)) FROM my_table")
```

## API Test: `ST_X`

### Signature
```python
ST_X(arg1: geometry) -> double | ST_X(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_x.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_X
description: Returns the X coordinate (longitude for geography) of the point, or NULL if not available.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

Returns NULL for non-point geometries or NULL input.

## Examples

```sql
SELECT ST_X(ST_Point(-74.0060, 40.7128));
```

```sql
SELECT ST_X(ST_GeogPoint(-74.0060, 40.7128));
```


### Goal
Extracts the X-coordinate (or longitude for geography types) from a point geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): A point geometry or geography object from which the X-coordinate is to be extracted.

### Input
The function accepts a valid `geometry` or `geography` type. If the input is not a point (e.g., a polygon or line string) or is NULL, the function returns NULL.

### Output
Returns `double` — The X-coordinate or longitude value of the input point.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Extract X coordinate from a geometry point
result = sd.sql("SELECT ST_X(ST_Point(-74.0060, 40.7128)) as x_coord").to_arrow_table()
print(result)
# __CHECK__ witness: x_coord: -74.006
```

### LLM Instruction Prompt
- Use `ST_X` only when the input is guaranteed to be a point geometry or geography.
- If the input geometry type is unknown, consider wrapping the call in a conditional or ensuring the input is filtered to point types first.

### Prompt Snippet
```text
ST_X(point_column)
```

### Common Failure Modes
- Passing a non-point geometry (e.g., `ST_Polygon` or `ST_LineString`) will result in a NULL return value rather than an error.
- Passing a NULL input will result in a NULL return value.

### Fix Code Hint
```python
# Ensure the input is a point before calling ST_X
sd.sql("SELECT ST_X(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Point'")
```

## API Test: `ST_XMax`

### Signature
```python
ST_XMax(arg1: geometry) -> double | ST_XMax(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_xmax.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_XMax
description: Returns the maximum X coordinate (longitude for geography) of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_XMax(ST_GeomFromText('LINESTRING (0 0, 5 4)'));
```

```sql
SELECT ST_XMax(ST_GeogFromWKT('LINESTRING (0 0, 5 4)'));
```


### Goal
Calculates the maximum X-coordinate (or longitude for geography types) of the bounding box enclosing the provided spatial object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., Point, LineString, Polygon) for which the maximum X-coordinate is to be determined.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Formats:** Can be derived from WKT, WKB, or GeoParquet inputs registered as views in SedonaDB.
- **Preconditions:** The input must be a valid spatial object. If using geography, the coordinate system must be defined to ensure the longitude interpretation is correct.

### Output
Returns `double` — The maximum X-coordinate value of the object's bounding box.

### Valid Call Patterns
```python
# Example inferred from signature and SQL examples
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT ST_XMax(ST_GeomFromText('LINESTRING (0 0, 5 4)')) as xmax").to_arrow_table()
assert result["xmax"][0] == 5.0
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_XMax` when you need to extract the easternmost boundary of a spatial feature.
- Ensure the input is a valid geometry or geography type; if the input is a string, wrap it in a constructor like `ST_GeomFromText`.

### Prompt Snippet
```text
ST_XMax(geometry_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., integer or string without conversion) will result in a SQL execution error.
- **Null Inputs:** Passing a `NULL` geometry will return `NULL` rather than a numeric value.
- **Empty Geometries:** Calling this on an empty geometry may return `NaN` or `NULL` depending on the underlying engine implementation.

### Fix Code Hint
```python
# Ensure input is cast to geometry if it is currently a string
sd.sql("SELECT ST_XMax(ST_GeomFromText(my_wkt_col)) FROM my_table")
```

## API Test: `ST_XMin`

### Signature
```python
ST_XMin(arg1: geometry) -> double | ST_XMin(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_xmin.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_XMin
description: Returns the minimum X coordinate (longitude for geography) of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_XMin(ST_GeomFromText('LINESTRING (0 0, 5 4)'));
```

```sql
SELECT ST_XMin(ST_GeogFromWKT('LINESTRING (0 0, 5 4)'));
```


### Goal
Calculates the minimum X-coordinate (or longitude for geography types) of the bounding box enclosing the provided geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the minimum X-coordinate is to be determined.

### Input
- **Data Formats:** Accepts standard geometry or geography types created via constructors like `ST_GeomFromText` or `ST_GeogFromWKT`.
- **Preconditions:** The input must be a valid spatial object. If the input is null, the function returns null.

### Output
Returns `double` — The minimum X-coordinate value of the object's bounding box.

### Valid Call Patterns
```python
# Inferred from SQL examples provided in documentation
import sedona.db
sd = sedona.db.connect()

# Geometry example
result = sd.sql("SELECT ST_XMin(ST_GeomFromText('LINESTRING (0 0, 5 4)'))").to_arrow_table()
assert result.column(0)[0].as_py() == 0.0
print("__CHECK__")

# Geography example
result = sd.sql("SELECT ST_XMin(ST_GeogFromWKT('LINESTRING (0 0, 5 4)'))").to_arrow_table()
assert result.column(0)[0].as_py() == 0.0
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_XMin` as a standalone function call in the SQL string passed to `sd.sql()`.
- Ensure the input is a valid geometry or geography object; if passing raw strings, wrap them in the appropriate constructor (e.g., `ST_GeomFromText`).

### Prompt Snippet
```text
Use ST_XMin(geometry_expression) to extract the minimum X coordinate from a spatial object.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., integer or string) directly without a constructor will result in a SQL execution error.
- **Null Inputs:** Passing a null geometry will return null rather than a numeric value; ensure downstream code handles potential nulls in the `ArrowTable`.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_XMin(ST_GeomFromText(my_wkt_column)) FROM my_table")
```

## API Test: `ST_Y`

### Signature
```python
ST_Y(arg1: geometry) -> double | ST_Y(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_y.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Y
description: Returns the Y coordinate (latitude for geography) of the point, or NULL if not available.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

Returns NULL for non-point geometries or NULL input.

## Examples

```sql
SELECT ST_Y(ST_Point(-74.0060, 40.7128));
```

```sql
SELECT ST_Y(ST_GeogPoint(-74.0060, 40.7128));
```


### Goal
Extracts the Y-coordinate (or latitude for geography types) from a point geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): A point object from which the Y-coordinate is to be extracted.

### Input
The function accepts a `geometry` or `geography` type. The input must be a point; if the input is a non-point geometry (e.g., Polygon, LineString) or NULL, the function returns NULL.

### Output
Returns `double` — The Y-coordinate value of the point, or NULL if the input is not a point or is missing.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Extract Y coordinate from a geometry point
result = sd.sql("SELECT ST_Y(ST_Point(-74.0060, 40.7128)) as y_coord").to_arrow_table()
print(result)
# __CHECK__ witness: y_coord: 40.7128
```

### LLM Instruction Prompt
- Use `ST_Y` only when the input is guaranteed to be a point geometry or geography.
- If the input geometry type is unknown, consider wrapping the call in a CASE statement or filtering for point types to avoid unexpected NULL results.

### Prompt Snippet
```text
ST_Y(point_column)
```

### Common Failure Modes
- Passing a non-point geometry (e.g., `ST_Polygon(...)`) will result in a NULL return value rather than an error.
- Passing a NULL input will result in a NULL output.

### Fix Code Hint
```python
# Ensure the geometry is a point before extraction
sd.sql("SELECT ST_Y(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Point'")
```

## API Test: `ST_YMax`

### Signature
```python
ST_YMax(arg1: geometry) -> double | ST_YMax(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_ymax.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_YMax
description: Returns the maximum Y coordinate (latitude for geography) of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

For geography, the maximum latitude is computed considering spherical geometry (e.g., the maximum latitude of a linestring that crosses the North Pole is 90).

## Examples

```sql
SELECT ST_YMax(ST_GeomFromText('LINESTRING(0 0, 5 4)'));
```


```sql
SELECT ST_YMax(ST_GeogFromText('LINESTRING(0 0, 5 4)'));
```


### Goal
Calculates the maximum Y-coordinate (or latitude for geography types) of the bounding box enclosing the provided geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the maximum Y-coordinate is to be determined.

### Input
- **Data:** Accepts standard geometry or geography types created via constructors like `ST_GeomFromText` or `ST_GeogFromText`.
- **Preconditions:** The input must be a valid spatial object. For geography types, the function accounts for spherical geometry, such as polar crossings.

### Output
Returns `double` — The maximum Y-coordinate value of the object's bounding box.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_YMax(ST_GeomFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()
assert result[0][0] == 4.0
print("__CHECK__")

# Example using geography
result = sd.sql("SELECT ST_YMax(ST_GeogFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()
assert result[0][0] == 4.0
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_YMax` when you need to extract the northernmost boundary of a spatial feature.
- Ensure the input is explicitly cast to `geometry` or `geography` if it is provided as a string literal.
- Do not use this function on raster types; use raster-specific metadata functions instead.

### Prompt Snippet
```text
ST_YMax(ST_GeomFromText('...'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or a raster object will result in a SQL execution error.
- **Invalid WKT:** Passing malformed WKT strings to the geometry/geography constructor will cause the query to fail.
- **Null Inputs:** Passing a `NULL` geometry will return `NULL`.

### Fix Code Hint
```python
# Ensure the input is a valid geometry object
sd.sql("SELECT ST_YMax(ST_GeomFromText('POINT(1 2)'))")
```

## API Test: `ST_YMin`

### Signature
```python
ST_YMin(arg1: geometry) -> double | ST_YMin(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_ymin.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_YMin
description: Returns the minimum Y coordinate (latitude for geography) of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

For geography, the minimum latitude is computed considering spherical geometry (e.g., the minimum latitude of a linestring that crosses the South Pole is -90).

## Examples

```sql
SELECT ST_YMin(ST_GeomFromText('LINESTRING(0 0, 5 4)'));
```


```sql
SELECT ST_YMin(ST_GeogFromText('LINESTRING(0 0, 5 4)'));
```


### Goal
Calculates the minimum Y-coordinate (or minimum latitude for geography types) of the bounding box enclosing the provided spatial object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., Point, LineString, Polygon) for which the minimum Y-coordinate is to be determined.

### Input
- **Data:** Accepts standard geometry or geography types.
- **Formats:** Can be constructed via WKT (e.g., `ST_GeomFromText`) or other SedonaDB-supported geometry/geography constructors.
- **Preconditions:** The input must be a valid geometry or geography object.

### Output
Returns `double` — The minimum Y-coordinate value of the object's bounding box.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_YMin(ST_GeomFromText('LINESTRING(0 0, 5 4)')) as ymin").to_arrow_table()
assert result["ymin"][0] == 0.0
print(f"__CHECK__ {result['ymin'][0]}")
```

### LLM Instruction Prompt
- Use `ST_YMin` to extract the lower bound of a spatial object's Y-axis.
- Ensure the input is explicitly cast to a geometry or geography type if provided as a string (e.g., using `ST_GeomFromText`).
- Note that for geography, the result is interpreted as latitude.

### Prompt Snippet
```text
ST_YMin(geometry_expression)
```

### Common Failure Modes
- Passing a non-spatial type or a null value will result in an error or null output.
- Mismatched CRS: While `ST_YMin` operates on the bounding box, ensure the input geometry is in the expected coordinate system if performing subsequent spatial operations.

### Fix Code Hint
```python
# Ensure input is a valid geometry object
sd.sql("SELECT ST_YMin(ST_GeomFromText('POINT(10 20)'))")
```

## API Test: `ST_Z`

### Signature
```python
ST_Z(arg1: geometry) -> double | ST_Z(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_z.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Z
description: Returns the Z coordinate of the point, or NULL if not available.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Description

Returns NULL if the geometry has no Z dimension or for non-point geometries.

## Examples

```sql
SELECT ST_Z(ST_PointZ(-74.0060, 40.7128, 100.0));
```


### Goal
Extracts the Z-coordinate (elevation) value from a 3D geometry or geography object.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object from which to extract the Z-coordinate.

### Input
The input must be a valid geometry or geography object containing Z-dimension data. If the input is a 2D geometry or a non-point geometry type, the function will return NULL.

### Output
Returns `double` — The Z-coordinate value of the input point, or NULL if the Z-dimension is missing or the geometry type is not a point.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example inferred from signature and documentation
result = sd.sql("SELECT ST_Z(ST_PointZ(-74.0060, 40.7128, 100.0)) as z_val").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Z` only when the input is expected to be a 3D point geometry or geography.
- Always account for the possibility of a NULL return value if the input geometry lacks Z-coordinates.

### Prompt Snippet
```text
ST_Z(geometry_column)
```

### Common Failure Modes
- Passing a 2D geometry (e.g., `ST_Point(x, y)`), which results in a NULL return value.
- Passing non-point geometries (e.g., `ST_Polygon`), which results in a NULL return value.
- Attempting to call `ST_Z` on a null input column, which will propagate as NULL.

### Fix Code Hint
```python
# Ensure the geometry is 3D before extraction
sd.sql("SELECT ST_Z(geom) FROM table WHERE ST_Is3D(geom) = true")
```

## API Test: `ST_ZMax`

### Signature
```python
ST_ZMax(arg1: geometry) -> double | ST_ZMax(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_zmax.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ZMax
description: Returns the maximum Z coordinate of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_ZMax(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)'));
```

```sql
SELECT ST_GeogFromWKT('LINESTRING Z (1 3 4, 5 6 7)');
```


### Goal
Returns the maximum Z-coordinate value found within the 3D bounding box of a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., Point Z, LineString Z, Polygon Z) from which to extract the maximum Z-coordinate.

### Input
- **Data Formats:** Accepts standard WKT or WKB representations of 3D geometries or geographies.
- **Preconditions:** The input object must contain Z-coordinates (e.g., `LINESTRING Z`). If the geometry is 2D, the behavior is undefined or may return null/zero depending on the underlying engine implementation.

### Output
Returns `double` — The maximum Z-coordinate value present in the input geometry's bounding box.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify ST_ZMax with geometry
result = sd.sql("SELECT ST_ZMax(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)')) as zmax").to_arrow_table()
assert result["zmax"][0] == 7.0
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_ZMax` as a standalone SQL function within an `sd.sql()` query.
- Ensure the input geometry or geography is explicitly 3D (e.g., using `ST_GeomFromWKT` with Z-coordinates).

### Prompt Snippet
```text
SELECT ST_ZMax(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Missing Z-dimension:** Passing a 2D geometry (e.g., `POINT(1 2)`) will not provide a Z-coordinate, potentially resulting in null or unexpected values.
- **Invalid WKT:** Providing a malformed WKT string that does not conform to the `Z` coordinate format will cause a query execution error.

### Fix Code Hint
```python
# Ensure the geometry is constructed with Z-coordinates
sd.sql("SELECT ST_ZMax(ST_GeomFromWKT('POINT Z (1 2 10)'))").to_arrow_table()
```

## API Test: `ST_ZMin`

### Signature
```python
ST_ZMin(arg1: geometry) -> double | ST_ZMin(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_zmin.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_ZMin
description: Returns the minimum Z coordinate of a geometry or geography's bounding box.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

## Examples

```sql
SELECT ST_ZMin(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)'));
```

```sql
SELECT ST_ZMin(ST_GeogFromWKT('LINESTRING Z (1 3 4, 5 6 7)'));
```


### Goal
Returns the minimum Z-coordinate value found within the 3D bounding box of a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., Point, LineString, Polygon) containing Z-coordinates for which the minimum Z value is to be calculated.

### Input
- **Data Formats:** Accepts standard SedonaDB geometry or geography types, typically constructed via WKT (e.g., `ST_GeomFromWKT`) or other spatial constructors.
- **Preconditions:** The input object must contain Z-coordinates (e.g., 3D geometries). If the geometry is 2D, the behavior is undefined or may return null/zero depending on the underlying engine implementation.

### Output
Returns `double` — The minimum Z-coordinate value present in the geometry or geography.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Example using geometry
result = sd.sql("SELECT ST_ZMin(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)'))").to_arrow_table()
print(result)
# __CHECK__ witness: 4.0
```

### LLM Instruction Prompt
- Use `ST_ZMin` when you need to extract the lowest elevation or Z-dimension value from a 3D spatial object.
- Ensure the input is a 3D geometry or geography; otherwise, the result may not be meaningful.
- Always use the SQL interface via `sd.sql(...)` as shown in the examples.

### Prompt Snippet
```text
ST_ZMin(ST_GeomFromWKT('LINESTRING Z (x1 y1 z1, x2 y2 z2)'))
```

### Common Failure Modes
- **2D Geometries:** Passing a 2D geometry (lacking Z-coordinates) may result in unexpected values or errors.
- **Null Inputs:** Passing a null geometry will return a null result.
- **Type Mismatch:** Passing a non-spatial type will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure the geometry is 3D before calling ST_ZMin
sd.sql("SELECT ST_ZMin(ST_Force3D(my_geom_col)) FROM my_table")
```

## API Test: `ST_Zmflag`

### Signature
```python
ST_Zmflag(arg1: geometry) -> integer | ST_Zmflag(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_zmflag.qmd:19_

_Source doc:_ ---
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

title: ST_Zmflag
description: Returns a code indicating the dimension of the coordinates in a geometry.
kernels:
  - returns: integer
    args: [geometry]
  - returns: integer
    args: [geography]
---

## Description

Returns a code indicating the coordinate dimension:

- 0 = 2D (XY)
- 1 = 3D with M coordinate (XYM)
- 2 = 3D with Z coordinate (XYZ)
- 3 = 4D (XYZM)

## Examples

2D (XY) returns 0:

```sql
SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING(1 2, 3 4)'));
```

3D with M coordinate (XYM) returns 1:

```sql
SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING M(1 2 3, 3 4 3)'));
```

3D with Z coordinate (XYZ) returns 2:

```sql
SELECT ST_GeomFromWKT('LINESTRING Z(1 2 3, 3 4 5)'));
```

4D (XYZM) returns 3:

```sql
SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING ZM(1 2 3 4, 3 4 5 6)'));
```


### Goal
`ST_Zmflag` identifies the coordinate dimensionality and presence of Z or M measures in a geometry or geography object to determine its coordinate structure.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect for coordinate dimension flags.

### Input
The function accepts standard geometry or geography types. Inputs are typically generated via constructors like `ST_GeomFromWKT` or loaded from spatial formats like GeoParquet. No specific CRS preconditions are required, as the function operates on the coordinate structure itself.

### Output
Returns an `integer` representing the coordinate dimension: 0 (XY), 1 (XYM), 2 (XYZ), or 3 (XYZM).

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Verify 3D XYZ geometry
result = sd.sql("SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING Z(1 2 3, 3 4 5)')) as flag").to_arrow_table()
assert result["flag"][0] == 2
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` to execute the function.
- Ensure the input is a valid geometry or geography object.
- Map the returned integer to the documented dimension codes (0-3) when interpreting results.

### Prompt Snippet
```text
ST_Zmflag(geometry_column)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) directly without a constructor like `ST_GeomFromWKT` will result in a SQL execution error.
- Assuming the function returns a boolean; it returns an integer code, so comparisons should be against 0, 1, 2, or 3.

### Fix Code Hint
```python
# Ensure input is cast to geometry if coming from a raw format
sd.sql("SELECT ST_Zmflag(ST_GeomFromWKT(my_wkt_col)) FROM my_table")
```

## API Test: `rs_ensureloaded`

### Signature
```python
rs_ensureloaded(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona/src/context.rs:1315_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Ensures that necessary raster-related resources or internal state are initialized within the SedonaDB execution context.

### Parameters
_None._

### Input
No specific input parameters are documented. The function is intended to be called within a `sd.sql()` execution context where a connection has been established via `sedona.db.connect()`.

### Output
Returns `unspecified` — The return value is not documented; the function is typically invoked for its side effects on the database session state.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Inferred usage based on SQL registration
result = sd.sql("SELECT rs_ensureloaded()").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `rs_ensureloaded` only when explicitly required to initialize raster context state.
- Do not pass arguments to this function as none are documented.
- Always execute via `sd.sql("SELECT rs_ensureloaded()")` to ensure it is processed by the SedonaDB engine.

### Prompt Snippet
```text
SELECT rs_ensureloaded()
```

### Common Failure Modes
- **Context Absence:** Calling the function before establishing a connection with `sedona.db.connect()`.
- **Assumption of Return Value:** Attempting to use the result of this function in further calculations, which may lead to errors as the return type is undefined.
- **Misuse:** Invoking this function as a substitute for explicit raster loading functions like `RS_FromPath` or `RS_FromGDALRaster`.

### Fix Code Hint
```python
# Ensure the connection is active before calling
sd = sedona.db.connect()
sd.sql("SELECT rs_ensureloaded()").to_arrow_table()
```

## API Test: `sd_simplifystorage`

### Signature
```python
sd_simplifystorage(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/sd_simplifystorage.rs:36_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Performs internal storage simplification operations within the SedonaDB engine.

### Parameters
_None._

### Input
The function is a registered SQL function; however, as it lacks documented parameters or a defined contract in the official reference, no specific input types or formats can be guaranteed.

### Output
Returns `unspecified` — The return type is not documented in the configured SQL reference.

### Valid Call Patterns
```python
# Example inferred from signature; usage is not verified by the project test suite.
sd.sql("SELECT sd_simplifystorage()").to_arrow_table()
```

### LLM Instruction Prompt
- Do not attempt to pass arguments to `sd_simplifystorage` as the parameter list is undocumented.
- Treat this function as an internal or experimental utility; verify results materialized via `to_arrow_table()` before relying on them in production workflows.

### Prompt Snippet
```text
sd.sql("SELECT sd_simplifystorage()").to_arrow_table()
```

### Common Failure Modes
- **Undocumented Contract:** Calling this function may result in runtime errors or unexpected behavior because the function signature and expected input/output types are not defined in the official SedonaDB SQL reference.
- **Experimental Status:** As an undocumented function, it may be subject to removal or breaking changes in future versions of SedonaDB.

### Fix Code Hint
```python
# If sd_simplifystorage fails, verify if the operation can be achieved 
# using documented raster or geometry functions like RS_Clip or ST_Simplify.
result = sd.sql("SELECT ST_Simplify(geom, 0.1) FROM my_table").to_arrow_table()
```

## API Test: `st_aswkb`

### Signature
```python
st_aswkb(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_asbinary.rs:37_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Converts a geometry object into its Well-Known Binary (WKB) representation.

### Parameters
_None._

### Input
The function expects a geometry object (e.g., Point, Polygon) as input. The input must be a valid geometry type recognized by SedonaDB.

### Output
Returns `unspecified` — The function returns the binary representation of the geometry in WKB format.

### Valid Call Patterns
```python
# Example inferred from signature (not verified against test suite)
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT st_aswkb(ST_Point(0, 1)) as wkb_geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_aswkb` only when a binary representation of a geometry is required for serialization or external storage.
- Ensure the input is a valid geometry type; if the input is a string, cast it using `ST_GeomFromText` first.
- Do not assume specific parameter signatures as this function is not fully documented in the original SQL reference.

### Prompt Snippet
```text
Use st_aswkb(geometry_column) to convert geometries to WKB format.
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or a numeric value) to the function, which will result in a runtime execution error.
- Assuming the function accepts additional parameters (like endianness or SRID inclusion) that are not documented in the provided API facts.

### Fix Code Hint
```python
# Ensure input is a geometry type
sd.sql("SELECT st_aswkb(ST_GeomFromText('POINT(0 0)'))").to_arrow_table()
```

## API Test: `st_aswkt`

### Signature
```python
st_aswkt(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_astext.rs:39_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Converts a geometry object into its Well-Known Text (WKT) string representation.

### Parameters
_None._

### Input
The function expects a geometry object (e.g., Point, Polygon) as input, typically generated by spatial constructors like `ST_Point` or `ST_GeomFromText`.

### Output
Returns `unspecified` — The function returns a string representing the geometry in WKT format.

### Valid Call Patterns
```python
# Inferred from signature:
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT st_aswkt(ST_Point(0, 1))").to_arrow_table()
# __CHECK__ witness: result
```

### LLM Instruction Prompt
- Use `st_aswkt` to serialize geometry objects into human-readable WKT strings for verification or export.
- Ensure the input is a valid geometry type; the function is called within a SQL query string passed to `sd.sql()`.

### Prompt Snippet
```text
SELECT st_aswkt(geometry_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-geometry type (e.g., a raster or a numeric value) to the function, which will result in a SQL execution error.
- Attempting to use the function without a valid SedonaDB connection established.
- Assuming the function supports specific WKT variants (e.g., EWKT) without explicit documentation; stick to standard WKT.

### Fix Code Hint
```python
# Ensure the input is a valid geometry
sd.sql("SELECT st_aswkt(ST_GeomFromText('POINT(0 0)'))").to_arrow_table()
```

## API Test: `st_force3dz`

### Signature
```python
st_force3dz(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_force_dim.rs:159_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Forces a geometry to 3D dimensionality by adding a Z-coordinate (typically initialized to 0) to the input geometry.

### Parameters
_None._

### Input
The function expects a valid geometry object (e.g., Point, LineString, Polygon) as input. The input must be a valid SedonaDB geometry type.

### Output
Returns `unspecified` — The resulting geometry with 3D coordinates (XYZ).

### Valid Call Patterns
```python
# Inferred call pattern based on SedonaDB SQL surface conventions
result = sd.sql("SELECT st_force3dz(ST_Point(0, 1))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `st_force3dz` only when a geometry must be converted to 3D format.
- Since the function signature is not fully documented in the reference, assume it operates on standard geometry types and returns a geometry.
- Always verify the output using `to_arrow_table()` to ensure the geometry transformation was applied as expected.

### Prompt Snippet
```text
Use st_force3dz(geometry_column) to ensure the geometry has a Z-dimension.
```

### Common Failure Modes
- **Invalid Geometry:** Passing a non-geometry type or a null value may result in an execution error.
- **Undocumented Contract:** Because this function lacks a formal page in the SQL reference, behavior regarding specific geometry types (e.g., GeometryCollections) or Z-value initialization is not guaranteed and may vary.
- **Type Mismatch:** Attempting to use this on raster types will fail, as it is a vector geometry function.

### Fix Code Hint
```python
# Ensure the input is a valid geometry before calling
sd.sql("SELECT st_force3dz(geom) FROM my_table WHERE geom IS NOT NULL").to_arrow_table()
```

## API Test: `st_geogfromtext`

### Signature
```python
st_geogfromtext(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkt.rs:88_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a geography object from a Well-Known Text (WKT) representation.

### Parameters
_None._

### Input
The function expects a string input representing a geometry in WKT format. As this function is not explicitly documented in the provided SQL reference, users should ensure the input string conforms to standard OGC WKT specifications.

### Output
Returns `unspecified` — The result is a geography-typed object representing the spatial feature defined by the input WKT string.

### Valid Call Patterns
```python
# Inferred from signature:
result = sd.sql("SELECT st_geogfromtext('POINT(0 0)') as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_geogfromtext` only when a geography-typed object is explicitly required by a downstream function.
- Ensure the input is a valid WKT string literal.
- Note that this function is not documented in the official SQL reference; verify results using `to_arrow_table()` to ensure the output matches expected spatial characteristics.

### Prompt Snippet
```text
sd.sql("SELECT st_geogfromtext('POINT(longitude latitude)')")
```

### Common Failure Modes
- **Undocumented Contract:** Because this function lacks an individual page in the SQL reference, behavior regarding specific WKT variants or coordinate order (e.g., Longitude/Latitude vs Latitude/Longitude) may vary.
- **Type Mismatch:** Passing a non-string type or an invalid WKT string will result in a runtime execution error.
- **CRS Assumptions:** If the underlying engine expects a specific CRS for geography types, ensure the WKT input is compatible with the session's coordinate system settings.

### Fix Code Hint
```python
# If st_geogfromtext fails, verify the WKT string format or 
# consider using ST_GeomFromText if a geometry type is sufficient.
result = sd.sql("SELECT ST_GeomFromText('POINT(0 0)')").to_arrow_table()
```

## API Test: `st_geometryfromtext`

### Signature
```python
st_geometryfromtext(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkt.rs:61_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Converts a Well-Known Text (WKT) string representation into a SedonaDB geometry object for use in spatial operations.

### Parameters
_None._

### Input
The function expects a string input representing a geometry in WKT format (e.g., `'POINT(0 0)'` or `'POLYGON((...))'`). Preconditions include ensuring the WKT string is syntactically valid according to OGC standards.

### Output
Returns `unspecified` — The result is a materialized geometry object compatible with SedonaDB spatial functions like `RS_Clip` or `RS_Intersects`.

### Valid Call Patterns
```python
# Inferred from project usage patterns for geometry construction
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT st_geometryfromtext('POINT(0 1)') as geom").to_arrow_table()
print(result)
# __CHECK__ witness: materialized geometry object
```

### LLM Instruction Prompt
- Use `st_geometryfromtext` to instantiate geometry objects from WKT strings when performing spatial queries or raster-geometry interactions.
- Ensure the WKT string is passed as a single string literal within the SQL query.
- Note that this function is distinct from `ST_GeomFromText` (which is the standard documented alias); verify if the environment requires the specific `st_geometryfromtext` identifier.

### Prompt Snippet
```text
sd.sql("SELECT st_geometryfromtext('POINT(x y)')")
```

### Common Failure Modes
- **Invalid WKT:** Providing a malformed WKT string will result in a runtime execution error.
- **CRS Mismatch:** If the resulting geometry is used in operations like `RS_Clip` against a raster with a defined CRS, ensure the geometry is either in the same CRS or that the operation is valid under SedonaDB's CRS propagation rules.
- **Case Sensitivity:** Depending on the specific SQL surface configuration, ensure the function name matches the registered identifier exactly.

### Fix Code Hint
```python
# Ensure the WKT string is properly escaped and formatted
query = "SELECT st_geometryfromtext('POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))')"
result = sd.sql(query).to_arrow_table()
```

## API Test: `st_geomfromtext`

### Signature
```python
st_geomfromtext(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkt.rs:60_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a geometry object from a Well-Known Text (WKT) representation for use in spatial operations.

### Parameters
_None._

### Input
The function accepts a WKT string (e.g., `'POLYGON((...))'`) and an optional CRS identifier (e.g., `'OGC:CRS84'`). Preconditions include ensuring the WKT string is valid and, if a CRS is provided, that it matches the CRS of other geometries or rasters involved in the same spatial operation.

### Output
Returns `unspecified` — A geometry object materialized from the provided WKT string, compatible with SedonaDB spatial functions like `RS_Clip` or `RS_Intersects`.

### Valid Call Patterns
```python
# Inferred from project README usage:
result = sd.sql("SELECT ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_GeomFromText` to convert WKT strings into geometry objects for spatial filtering or raster clipping.
- Always ensure the WKT string is properly formatted.
- If performing operations with rasters (e.g., `RS_Clip`), ensure the CRS provided to `ST_GeomFromText` matches the raster's CRS.

### Prompt Snippet
```text
ST_GeomFromText('WKT_STRING', 'CRS_IDENTIFIER')
```

### Common Failure Modes
- **CRS Mismatch:** Providing a CRS that does not match the target raster or geometry will cause the operation to error.
- **Invalid WKT:** Passing a malformed WKT string will result in a runtime execution error.
- **Memory Constraints:** While `ST_GeomFromText` is lightweight, using it to generate extremely complex geometries in a large `SELECT` statement may impact memory usage during materialization.

### Fix Code Hint
```python
# Ensure CRS consistency
sd.sql("SELECT RS_Clip(raster_col, 1, ST_GeomFromText('POINT(0 0)', 'EPSG:4326')) FROM my_table")
```

## API Test: `st_geomfromwkbunchecked`

### Signature
```python
st_geomfromwkbunchecked(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkb.rs:57_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a geometry object from a Well-Known Binary (WKB) representation without performing validation checks on the input geometry.

### Parameters
_None._

### Input
The function expects a binary input representing a WKB-encoded geometry. As this is an "unchecked" function, the caller is responsible for ensuring the input binary is a valid WKB representation, as the engine will not perform standard geometry validity verification.

### Output
Returns `unspecified` — The resulting geometry object materialized from the provided WKB binary.

### Valid Call Patterns
```python
# Inferred call pattern based on SQL surface registration
result = sd.sql("SELECT st_geomfromwkbunchecked(wkb_column) FROM my_table").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_geomfromwkbunchecked` only when the input WKB is guaranteed to be valid and performance is prioritized over validation.
- Always wrap the call in `sd.sql(...).to_arrow_table()` when executing via the Python interface.
- Do not assume specific parameter signatures as they are not documented in the official SQL reference.

### Prompt Snippet
```text
sd.sql("SELECT st_geomfromwkbunchecked(binary_col) FROM table_name").to_arrow_table()
```

### Common Failure Modes
- **Invalid WKB:** Providing malformed binary data will result in runtime errors during query execution because the function skips validation.
- **Type Mismatch:** Passing non-binary types (e.g., strings or integers) to the function will cause a type mismatch error in the DataFusion execution engine.
- **Null Inputs:** Passing a `NULL` value may result in a `NULL` geometry output depending on the specific SQL dialect implementation.

### Fix Code Hint
```python
# If the WKB source is untrusted, use a standard validation-enabled function instead
# or sanitize the input before passing it to st_geomfromwkbunchecked.
sd.sql("SELECT ST_GeomFromWKB(wkb_column) FROM my_table").to_arrow_table()
```

## API Test: `st_linefromwkb`

### Signature
```python
st_linefromwkb(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkb.rs:106_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a LineString geometry object from a Well-Known Binary (WKB) representation.

### Parameters
_None._

### Input
The function expects a WKB-encoded binary string or byte array representing a LineString geometry. As this function is not explicitly documented in the provided SQL reference, users should ensure the input conforms to standard OGC WKB specifications for LineString types.

### Output
Returns `unspecified` — The resulting geometry object materialized as a LineString in the SedonaDB engine.

### Valid Call Patterns
```python
# Inferred from signature:
result = sd.sql("SELECT st_linefromwkb(wkb_column) as geom FROM my_table").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_linefromwkb` only when the input is confirmed to be a WKB-encoded LineString.
- Since this function lacks explicit documentation in the provided reference, verify the output geometry type using `ST_GeometryType` if the result is unexpected.
- Always wrap the call in a `sd.sql()` execution block.

### Prompt Snippet
```text
SELECT st_linefromwkb(wkb_data) FROM table_name
```

### Common Failure Modes
- **Invalid WKB:** Providing binary data that does not conform to the WKB format for a LineString will result in a runtime execution error.
- **Type Mismatch:** Passing non-binary types (e.g., plain strings or integers) will cause the function to fail during query execution.
- **Undocumented Behavior:** Because this function is not fully documented in the provided SQL reference, it may be subject to change or removal in future versions of the SedonaDB engine.

### Fix Code Hint
```python
# Ensure the input is valid WKB before passing to the function
# If the input is a hex string, use ST_GeomFromWKB(ST_AsBinary(hex_string)) 
# or ensure the column is cast to binary type first.
sd.sql("SELECT st_linefromwkb(CAST(wkb_col AS BINARY)) FROM my_table").to_arrow_table()
```

## API Test: `st_linestringfromtext`

### Signature
```python
st_linestringfromtext(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkt.rs:106_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a geometry object of type `LineString` from a Well-Known Text (WKT) string representation.

### Parameters
_None._

### Input
The function expects a string input representing a WKT `LINESTRING` (e.g., `'LINESTRING(0 0, 1 1, 2 2)'`). As this function is not explicitly documented in the provided reference, users should ensure the input string strictly adheres to standard OGC WKT formatting.

### Output
Returns `unspecified` — The result is a materialized geometry object representing the line defined by the input coordinates.

### Valid Call Patterns
```python
# Inferred from signature:
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT st_linestringfromtext('LINESTRING(0 0, 1 1)') as geom").to_arrow_table()
print(result)
# __CHECK__ witness: The result should contain a geometry column with the materialized LineString.
```

### LLM Instruction Prompt
- Use `st_linestringfromtext` only when the input is a valid WKT string representing a line.
- Do not assume specific coordinate system support unless explicitly provided in the query context.
- Always wrap the function call within an `sd.sql()` execution block.

### Prompt Snippet
```text
sd.sql("SELECT st_linestringfromtext('LINESTRING(x1 y1, x2 y2)')")
```

### Common Failure Modes
- **Invalid WKT:** Providing a string that does not conform to the `LINESTRING` WKT format will result in a runtime SQL error.
- **Type Mismatch:** Passing non-string types or empty inputs may cause the underlying DataFusion engine to reject the query.
- **Undocumented Contract:** Because this function lacks an individual page in the official reference, behavior regarding SRID/CRS injection or 3D coordinate support is unknown and should be verified via `to_arrow_table()` before production use.

### Fix Code Hint
```python
# Ensure the input is a valid WKT string
wkt_input = 'LINESTRING(0 0, 10 10)'
query = f"SELECT st_linestringfromtext('{wkt_input}')"
result = sd.sql(query).to_arrow_table()
```

## API Test: `st_longestline`

### Signature
```python
st_longestline(parameters not documented in the configured SQL reference)
```
_Source: source/c/sedona-s2geography/src/kernels.rs:115_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Calculates the longest line segment between two geometries, typically used to determine the maximum distance between points within a geometry or between two distinct geometries.

### Parameters
_None._

### Input
The function expects geometry inputs (e.g., `Polygon`, `MultiPoint`, or `LineString`). As a registered SQL function in SedonaDB, inputs must be valid geometry types. If inputs have defined Coordinate Reference Systems (CRS), they must match or be reprojectable to a common CRS before processing.

### Output
Returns `unspecified` — The result is a `LineString` geometry representing the longest distance between the input geometries.

### Valid Call Patterns
```python
# Example inferred from signature; not verified against test suite
result = sd.sql("SELECT ST_LongestLine(geom1, geom2) FROM my_table").to_arrow_table()
```

### LLM Instruction Prompt
- Use `st_longestline` only when performing spatial analysis on vector geometries.
- Do not assume specific parameter signatures as they are not documented in the official reference.
- Ensure all input geometries are valid and share the same CRS before execution.

### Prompt Snippet
```text
Use st_longestline(geom_a, geom_b) to find the longest line segment between two geometries in SedonaDB.
```

### Common Failure Modes
- **CRS Mismatch:** If input geometries have mismatched CRS definitions, the query may fail or produce undefined results.
- **Unsupported Types:** Passing raster objects or non-geometry types as arguments will result in a SQL execution error.
- **Undocumented Signature:** Because the function lacks an individual page in the official reference, relying on specific argument order or optional parameters not explicitly defined in the project context may lead to runtime errors.

### Fix Code Hint
```python
# Ensure geometries are in the same CRS before calling
sd.sql("""
    SELECT ST_LongestLine(
        ST_Transform(geom1, 'EPSG:4326'), 
        ST_Transform(geom2, 'EPSG:4326')
    ) FROM my_table
""").to_arrow_table()
```

## API Test: `st_numinteriorring`

### Signature
```python
st_numinteriorring(parameters not documented in the configured SQL reference)
```
_Source: source/c/sedona-geos/src/register.rs:81_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Returns the number of interior rings (holes) within a given geometry object.

### Parameters
_None._

### Input
The function expects a geometry object (typically a Polygon or MultiPolygon) as input. The input must be a valid geometry type supported by SedonaDB.

### Output
Returns `unspecified` — The result is an integer representing the count of interior rings found within the geometry.

### Valid Call Patterns
```python
# Inferred from signature:
result = sd.sql("SELECT st_numinteriorring(geom) FROM my_table").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_numinteriorring` only when operating on geometry columns containing polygons.
- Do not assume the return type is a specific integer width (e.g., Int32 vs Int64) as the contract is not explicitly documented.
- Ensure the input geometry is materialized or registered as a view before calling.

### Prompt Snippet
```text
SELECT st_numinteriorring(geometry_column) FROM table_name
```

### Common Failure Modes
- Passing a non-polygon geometry type (e.g., Point or LineString) may result in an execution error or undefined behavior.
- Calling the function on a null geometry column may return null or trigger a runtime exception depending on the underlying engine state.
- Attempting to use this function on raster data without first converting the raster to a geometry type.

### Fix Code Hint
```python
# Ensure the input is a valid polygon before calling
sd.sql("SELECT st_numinteriorring(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Polygon'")
```

## API Test: `st_shortestline`

### Signature
```python
st_shortestline(parameters not documented in the configured SQL reference)
```
_Source: Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract._

### Goal
Calculates the shortest line segment between two geometries.

### Parameters
_None._

### Input
The function expects two geometry objects as input. As this function is not explicitly documented in the provided SQL reference, users must ensure that input geometries are valid and share the same Coordinate Reference System (CRS) to avoid potential runtime errors, consistent with SedonaDB's general spatial operation requirements.

### Output
Returns `unspecified` — The result is expected to be a geometry object representing the shortest line segment connecting the two input geometries.

### Valid Call Patterns
```python
# Example inferred from signature; not verified against test suite
result = sd.sql("SELECT ST_ShortestLine(ST_Point(0, 0), ST_Point(1, 1))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `st_shortestline` only when performing spatial analysis requiring the shortest distance connection between two geometries.
- Ensure both input geometries are valid and defined within the same CRS.
- Since this function is not fully documented in the provided reference, treat its return type and parameter requirements as experimental.

### Prompt Snippet
```text
SELECT ST_ShortestLine(geom1, geom2) FROM table_name
```

### Common Failure Modes
- **CRS Mismatch:** If the input geometries have mismatched CRSs, the operation may fail or produce undefined results, as SedonaDB requires explicit reprojection for mismatched spatial data.
- **Invalid Geometry:** Providing non-geometry types or malformed WKT/WKB will result in a SQL execution error.
- **Undocumented Contract:** Because this function lacks an official page in the SQL reference, behavior regarding specific geometry types (e.g., GeometryCollections) or edge cases is unknown and may change in future versions.

### Fix Code Hint
```python
# Ensure geometries are in the same CRS before calling
sd.sql("""
    SELECT ST_ShortestLine(
        ST_Transform(geom1, 'EPSG:4326'), 
        ST_Transform(geom2, 'EPSG:4326')
    ) FROM my_table
""").to_arrow_table()
```

