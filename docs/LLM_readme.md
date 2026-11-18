# SedonaDB 0.5.0 canonical SQL Original R0 — LLM_readme

LLM-facing API documentation. Generated from the API surface, project profile, and distilled docs.

## API Test: `sql:RS_AsGeoTiff`

### Signature
```python
RS_AsGeoTiff(arg1: raster) -> binary | RS_AsGeoTiff(arg1: raster, tile_size: integer) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double, tile_size: integer) -> binary | RS_AsGeoTiff(arg1: raster, compression: string, quality: double, tile_width: integer, tile_height: integer) -> binary
```
_Source: source/docs/reference/sql/rs_asgeotiff.qmd:19_

### Goal
`RS_AsGeoTiff` encodes an in-database raster object into a GeoTIFF binary byte stream, enabling the export of raster data for external use.

### Parameters
- `arg1` (`raster`): The input raster object to be encoded.
- `compression` (`string`): The codec to use (case-insensitive): 'None', 'PackBits', 'Deflate', 'Huffman', 'LZW', or 'JPEG'.
- `quality` (`double`): A value between 0.0 and 1.0 for lossy JPEG compression; ignored by other codecs.
- `tile_width` (`integer`): The width of the GeoTIFF tiles in pixels; must be a multiple of 16.
- `tile_height` (`integer`): The height of the GeoTIFF tiles in pixels; must be a multiple of 16.

### Input
The function requires a valid `raster` object. If the input is `NULL`, the result is `NULL`. When specifying tiling, both `tile_width` and `tile_height` must be multiples of 16.

### Output
Returns `binary` containing the raw bytes of a GeoTIFF file.

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
- Always use `RS_AsGeoTiff` to convert raster objects to binary GeoTIFF format.
- Ensure that if `tile_width` or `tile_height` are provided, they are multiples of 16.
- Use the `compression` string case-insensitively, but prefer standard naming (e.g., 'DEFLATE', 'LZW').
- Remember that `quality` is only applicable to 'JPEG' compression.

### Prompt Snippet
```text
Use RS_AsGeoTiff(raster, ...) to export raster data. Ensure tile dimensions are multiples of 16.
```

### Common Failure Modes
- **Invalid Tile Dimensions:** Providing `tile_width` or `tile_height` that are not multiples of 16 will cause an error.
- **Unsupported Compression:** Using 'Huffman' on non-1-bit single-band data will result in an error.
- **Quality Range:** Providing a `quality` value outside the 0.0–1.0 range for JPEG compression will raise an error.
- **Experimental Status:** As an experimental function, behavior may change; ensure the environment supports the current SedonaDB 0.5.0 raster API.

### Fix Code Hint
```python
# Ensure tile dimensions are multiples of 16
sd.sql("SELECT RS_AsGeoTiff(my_raster, 'LZW', 0.9, 32, 32)").to_arrow_table()
```

## API Test: `sql:RS_AsRaster`

### Signature
```python
RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double, use_geometry_extent: boolean) -> raster | RS_AsRaster(geometry: geometry, reference_raster: raster, pixel_type: string, all_touched: boolean, burn_value: double, nodata_value: double, use_geometry_extent: boolean) -> raster
```
_Source: source/docs/reference/sql/rs_asraster.qmd:19_

### Goal
Creates a single-band in-database raster by rasterizing a geometry onto the grid defined by a reference raster.

### Parameters
- `geometry` (`geometry`): The OGC geometry to be rasterized.
- `reference_raster` (`raster`): The raster object providing the grid, CRS, and geotransform for the output.
- `pixel_type` (`string`): The data type for the output pixels (e.g., 'int8', 'uint8', 'int16', 'uint16', 'int32', 'uint32', 'int64', 'uint64', 'float32', 'float64').
- `all_touched` (`boolean`): If true, all pixels touched by the geometry are burned; if false, only pixels whose centers are covered by the geometry are burned.
- `burn_value` (`double`): The value to assign to pixels covered by the geometry.
- `nodata_value` (`double`): The value used to represent "no data" in the output raster.
- `use_geometry_extent` (`boolean`): If true, the output raster extent is cropped to the geometry; if false, it matches the reference raster extent.

### Input
- **Geometry:** Standard OGC WKT/WKB geometries.
- **Raster:** GDAL-readable raster objects.
- **Preconditions:** If both inputs have a CRS, they must match; if they differ, the geometry is automatically reprojected. If exactly one input has a CRS, the operation will fail.

### Output
Returns `raster` — A new single-band in-database raster object.

### Valid Call Patterns
```python
# Example using the documented SQL pattern via Python
query = """
SELECT RS_AsRaster(
    RS_ConvexHull(raster),
    raster,
    'float64',
    false,
    1.0,
    0.0,
    false
)
FROM (SELECT RS_FromPath('https://download.osgeo.org/geotiff/samples/gdal_eg/cea.tif') AS raster) r
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure the `pixel_type` is a valid string from the supported list (e.g., 'float64').
- If the geometry and raster have different CRS, ensure they are compatible for reprojection; otherwise, the operation will fail.
- Use the `all_touched` and `use_geometry_extent` flags to control the spatial precision and output dimensions of the resulting raster.

### Prompt Snippet
```text
RS_AsRaster(geometry, reference_raster, pixel_type, all_touched, burn_value, nodata_value, use_geometry_extent)
```

### Common Failure Modes
- **CRS Mismatch:** Providing inputs where only one has a defined CRS will trigger an error.
- **Experimental Status:** As an experimental function, behavior may change; ensure the SedonaDB version is 0.5.0.
- **Memory Constraints:** Large rasters or complex geometries may hit memory limits during the rasterization process.

### Fix Code Hint
```python
# Ensure both inputs have the same CRS or neither has one to avoid errors
sd.sql("SELECT RS_AsRaster(geom, rast, 'float64', false, 1.0, 0.0, false) FROM table").to_arrow_table()
```

## API Test: `sql:RS_BandIsNoData`

### Signature
```python
RS_BandIsNoData(arg1: raster) -> boolean | RS_BandIsNoData(arg1: raster, band: integer) -> boolean
```
_Source: source/docs/reference/sql/rs_bandisnodata.qmd:19_

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

title: RS_BandIsNoData
description: >
  Returns true if every pixel of the band holds the band's nodata value. Returns
  false if any pixel holds data, or if the band has no nodata value.
kernels:
  - returns: boolean
    args: [raster]
  - returns: boolean
    args:
    - raster
    - name: band
      type: integer
      description: Band index (1-based). Defaults to 1 if not specified.
---

## Description

A pixel counts as nodata when it equals the band's nodata value; a NaN pixel
matches a NaN nodata value. A band without a nodata value is never all nodata,
even if every pixel is 0. Only 2-D rasters are supported.

## Examples

```sql
SELECT RS_BandIsNoData(RS_Example());
```

```sql
SELECT RS_BandIsNoData(RS_Example(), 1);
```


### Goal
Determines if a specific raster band consists entirely of pixels matching the band's defined nodata value.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`integer`): The 1-based index of the band to check. Defaults to 1 if not provided.

### Input
- **Raster:** Must be a 2-D raster object.
- **Preconditions:** The band must have a defined nodata value to return `true`. If no nodata value is set for the band, the function returns `false` regardless of pixel values.

### Output
Returns `boolean` — `true` if every pixel in the specified band equals the band's nodata value; `false` otherwise.

### Valid Call Patterns
```python
# Check the first band (default)
sd.sql("SELECT RS_BandIsNoData(raster_col) FROM my_table").to_arrow_table()

# Check a specific band (e.g., band 2)
sd.sql("SELECT RS_BandIsNoData(raster_col, 2) FROM my_table").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_BandIsNoData(raster_object)` or `RS_BandIsNoData(raster_object, band_index)`.
- Ensure the raster is 2-D.
- Remember that band indices are 1-based.

### Prompt Snippet
```text
RS_BandIsNoData(raster, band)
```

### Common Failure Modes
- **Missing Nodata Value:** If the band has no nodata value defined, the function returns `false` even if all pixels are 0.
- **Unsupported Dimensions:** Attempting to use this on non-2-D rasters will result in an error.
- **Invalid Band Index:** Providing a band index outside the range of available bands (e.g., 0 or > `RS_NumBands`) will cause an execution error.

### Fix Code Hint
```python
# Ensure the band has a nodata value set before checking
sd.sql("SELECT RS_BandIsNoData(RS_SetBandNoDataValue(raster, 1, 0.0), 1) FROM my_table")
```

## API Test: `sql:RS_BandNoDataValue`

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
Retrieves the defined "nodata" value for a specific band within a raster object to facilitate accurate pixel-based analysis and masking.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract metadata.
- `band` (`int`): The 1-based index of the band to query; defaults to 1 if omitted.

### Input
Requires a valid SedonaDB `raster` object. The raster must be loaded or created within the session (e.g., via `RS_FromPath` or `RS_MakeEmptyRaster`). Band indices are strictly 1-based.

### Output
Returns `float64` — The nodata value associated with the specified band. Returns `null` if no nodata value is explicitly defined for that band.

### Valid Call Patterns
```python
# Retrieve nodata value for the first band (default)
sd.sql("SELECT RS_BandNoDataValue(raster_col) FROM my_raster_table").to_arrow_table()

# Retrieve nodata value for a specific band (e.g., band 2)
sd.sql("SELECT RS_BandNoDataValue(raster_col, 2) FROM my_raster_table").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for the `band` parameter.
- If the raster band does not have a nodata value, the function returns `null`; ensure downstream logic handles `null` values appropriately.
- Use the function within a `SELECT` statement as a scalar operation on a raster column.

### Prompt Snippet
```text
Use RS_BandNoDataValue(raster_column, band_index) to inspect the nodata metadata of a raster band.
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index less than 1 or greater than the total number of bands in the raster will result in an error.
- **Null Handling:** Attempting to perform arithmetic on the result without checking for `null` (e.g., if the raster has no nodata value defined) may lead to unexpected query results.
- **Type Mismatch:** Passing a non-raster object as the first argument will cause a runtime execution error.

### Fix Code Hint
```python
# Ensure the band index is valid by checking the total number of bands first
sd.sql("SELECT RS_BandNoDataValue(raster, 1) FROM table WHERE 1 <= RS_NumBands(raster)").to_arrow_table()
```

## API Test: `sql:RS_BandPath`

### Signature
```python
RS_BandPath(arg1: raster) -> utf8
```
_Source: source/docs/reference/sql/rs_bandpath.qmd:19_

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

title: RS_BandPath
description: >
  Retrieves the file path of an out-of-database (out-db) raster band, returning
  the external raster file location referenced by the raster.
kernels:
  - returns: utf8
    args: [raster]
---

## Description

Primarily used with out-db rasters, where only raster path and geo-referencing
metadata are stored in the database.

## Examples

```sql
SELECT RS_BandPath(RS_Example());
```


### Goal
Retrieves the external file system path associated with an out-of-database (out-db) raster band.

### Parameters
- `arg1` (`raster`): The raster object from which to extract the source file path.

### Input
The input must be a valid `raster` object. This function is specifically designed for out-of-database rasters where the database stores only the file path and geo-referencing metadata rather than the full pixel data.

### Output
Returns `utf8` — A string representing the absolute or relative file path to the external raster file referenced by the input raster.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
result = sd.sql("SELECT RS_BandPath(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_BandPath(raster_column)` within a `SELECT` statement.
- Ensure the input is a valid `raster` type.
- Note that this function is intended for out-db rasters; behavior on in-memory or fully materialized rasters may be undefined or return null/empty strings.

### Prompt Snippet
```text
SELECT RS_BandPath(raster_col) FROM table_name;
```

### Common Failure Modes
- Passing a raster that is not configured as an out-db raster (may return an empty string or null).
- Providing an invalid raster object or a non-raster type, which will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the raster was loaded as an out-db reference if the path is required
sd.sql("SELECT RS_BandPath(my_raster_col) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_BandPixelType`

### Signature
```python
RS_BandPixelType(arg1: raster) -> utf8 | RS_BandPixelType(arg1: raster, band: int) -> utf8
```
_Source: source/docs/reference/sql/rs_bandpixeltype.qmd:19_

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

title: RS_BandPixelType
description: Returns the pixel data type of the specified band as a string.
kernels:
  - returns: utf8
    args: [raster]
  - returns: utf8
    args:
    - raster
    - name: band
      type: int
      description: Band index (1-based). Defaults to 1 if not specified.
---

## Examples

```sql
SELECT RS_BandPixelType(RS_Example());
```

```sql
SELECT RS_BandPixelType(RS_Example(), 1);
```


### Goal
Returns the pixel data type of a specified raster band as a string (e.g., 'uint8', 'float32').

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): The 1-based index of the band to query. Defaults to 1 if not specified.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be loaded or created within the SedonaDB session (e.g., via `RS_FromPath` or `RS_MakeEmptyRaster`).

### Output
Returns `utf8` — A string representing the pixel data type (e.g., `int8`, `uint8`, `int16`, `uint16`, `int32`, `uint32`, `int64`, `uint64`, `float32`, `float64`).

### Valid Call Patterns
```python
# Verify pixel type of the first band
result = sd.sql("SELECT RS_BandPixelType(RS_Example())").to_arrow_table()
print(result)
# __CHECK__

# Verify pixel type of a specific band (e.g., band 1)
result = sd.sql("SELECT RS_BandPixelType(RS_Example(), 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_BandPixelType` to inspect raster metadata.
- Remember that band indices are 1-based; providing 0 or an index exceeding the number of bands will result in an error.
- If the band is not specified, the function defaults to band 1.

### Prompt Snippet
```text
Use RS_BandPixelType(raster_column, band_index) to retrieve the pixel type string.
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index less than 1 or greater than the total number of bands in the raster.
- **Null Input:** Passing a null or non-raster object as `arg1`.
- **Missing Raster:** Attempting to call the function on a column that does not contain valid raster objects.

### Fix Code Hint
```python
# Ensure the band index is within the valid range [1, RS_NumBands(raster)]
sd.sql("SELECT RS_BandPixelType(raster, 1) FROM my_rasters WHERE RS_NumBands(raster) >= 1")
```

## API Test: `sql:RS_BandToDim`

### Signature
```python
RS_BandToDim(arg1: raster, dim_name: utf8) -> raster
```
_Source: source/docs/reference/sql/rs_bandtodim.qmd:19_

### Goal
Collapses all bands of a multi-band raster into a single band by prepending a new dimension, facilitating N-dimensional data structures for formats like Zarr.

### Parameters
- `arg1` (`raster`): The input multi-band raster to be collapsed.
- `dim_name` (`utf8`): The name for the newly created dimension (e.g., 'time', 'band').

### Input
- **Data:** A multi-band raster object.
- **Preconditions:** 
    - All bands must have identical dimension names, shapes, data types, and nodata values.
    - The `dim_name` must not already exist as a dimension name on the input raster.
- **Formats:** Supports GDAL-readable raster formats.

### Output
Returns `raster` — A single-band raster where the original bands are concatenated along the newly introduced dimension.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
sd.sql("SELECT RS_BandToDim(RS_Example(), 'band')").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_BandToDim(raster, dim_name)` to collapse bands.
- Ensure the input raster has uniform band properties (shape, type, nodata) before calling.
- Verify that `dim_name` is a unique string not present in the raster's existing dimensions.
- Use this function when preparing data for N-dimensional export formats like Zarr.

### Prompt Snippet
```text
RS_BandToDim(raster_object, 'new_dimension_name')
```

### Common Failure Modes
- **Dimension Conflict:** Providing a `dim_name` that already exists in the raster metadata will cause an error.
- **Heterogeneous Bands:** If bands differ in shape, data type, or nodata values, the operation will fail to prevent loss of sentinel information.
- **Experimental Status:** As an experimental function, behavior may change; ensure the environment is updated to the latest SedonaDB 0.5.0 release.

### Fix Code Hint
```python
# Ensure all bands are uniform before collapsing
# If bands differ, use RS_Resample or other transformation tools first
sd.sql("SELECT RS_BandToDim(my_raster, 'time')").to_arrow_table()
```

## API Test: `sql:RS_CRS`

### Signature
```python
RS_CRS(arg1: raster) -> string
```
_Source: source/docs/reference/sql/rs_crs.qmd:19_

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

title: RS_CRS
description: Returns the CRS string for a raster.
kernels:
  - returns: string
    args: [raster]
---

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT RS_CRS(RS_Example());
```


### Goal
Retrieves the Coordinate Reference System (CRS) definition string associated with a given raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the CRS metadata.

### Input
- **Data:** A valid `raster` object (e.g., loaded via `RS_FromPath` or generated via `RS_MakeEmptyRaster`).
- **Preconditions:** The raster must have a defined CRS; if the raster was created without a CRS, the behavior depends on the underlying storage metadata.

### Output
Returns `string` — The CRS definition string (e.g., 'EPSG:4326') associated with the raster.

### Valid Call Patterns
```python
# Using the Python SQL connection
result = sd.sql("SELECT RS_CRS(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_CRS(raster_expression)` within a `sd.sql()` query.
- Ensure the input is a valid `raster` type.
- Do not attempt to cast the output string to a geometry; it is strictly metadata.

### Prompt Snippet
```text
SELECT RS_CRS(raster_column) FROM table_name;
```

### Common Failure Modes
- **Null Input:** Passing a null or invalid raster object will result in a query execution error.
- **Missing CRS:** If the raster does not have an assigned CRS, the function may return an empty string or null depending on the raster's internal metadata state.
- **Type Mismatch:** Passing a `geometry` type instead of a `raster` type will cause a signature mismatch error.

### Fix Code Hint
```python
# Ensure the raster is valid before querying metadata
sd.sql("SELECT RS_CRS(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL").to_arrow_table()
```

## API Test: `sql:RS_Clip`

### Signature
```python
RS_Clip(arg1: raster, band: integer, geom: geometry) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double, crop: boolean) -> raster | RS_Clip(arg1: raster, band: integer, geom: geometry, all_touched: boolean, no_data_value: double, crop: boolean, lenient: boolean) -> raster
```
_Source: source/docs/reference/sql/rs_clip.qmd:19_

### Goal
`RS_Clip` performs a spatial clip of a raster to a provided geometry, masking pixels outside the geometry with a nodata value and optionally cropping the raster extent to the geometry's bounding box.

### Parameters
- `arg1` (`raster`): The input raster object to be clipped.
- `band` (`integer`): The 1-based index of the band to clip, or 0 to clip every band. A NULL band results in a NULL output.
- `geom` (`geometry`): The clipping geometry. If both the raster and geometry have a CRS, the geometry is reprojected to the raster's CRS.
- `all_touched` (`boolean`): If true, includes every pixel the geometry touches; if false (default), only includes pixels whose center falls inside the geometry.
- `no_data_value` (`double`): The value assigned to pixels outside the geometry. Defaults to the band's existing nodata value or the data type's minimum.
- `crop` (`boolean`): If true (default), crops the output raster to the geometry's bounding box intersected with the raster extent.
- `lenient` (`boolean`): If true (default), returns NULL if the geometry does not intersect the raster; if false, raises an error.

### Input
- **Raster:** An in-database raster object.
- **Geometry:** Standard OGC WKT/WKB geometry.
- **Preconditions:** Both the raster and geometry must either both have a CRS or both lack one; it is an error if only one side has a CRS.

### Output
Returns `raster` — A new raster object containing the clipped data, with pixels outside the geometry set to the specified nodata value.

### Valid Call Patterns
```python
# Basic clip
sd.sql("SELECT RS_Clip(RS_Example(), 1, ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'))").to_arrow_table()

# Full parameter clip
sd.sql("SELECT RS_Clip(RS_Example(), 1, ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'), false, 0.0, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_Clip` within a `sd.sql()` query.
- Ensure the `band` index is 1-based.
- If providing a geometry, ensure it is compatible with the raster's CRS (or both lack a CRS).
- Remember that `RS_Clip` is experimental and behavior may change.

### Prompt Snippet
```text
Use RS_Clip(raster, band, geometry, ...) to perform spatial clipping. Ensure CRS consistency between the raster and geometry.
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and raster where only one has a defined CRS will cause an error.
- **Invalid Band Index:** Using a 0-based index instead of 1-based for the `band` parameter.
- **No Intersection:** If `lenient` is false and the geometry does not intersect the raster, the query will fail.
- **Data Type Mismatch:** Providing a `no_data_value` that does not fit the band's underlying data type.

### Fix Code Hint
```python
# Ensure CRS consistency by transforming the geometry if necessary
sd.sql("""
    SELECT RS_Clip(
        raster, 
        1, 
        ST_Transform(geom, RS_SRID(raster))
    ) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `sql:RS_Contains`

### Signature
```python
RS_Contains(arg1: raster, arg2: geometry) -> boolean | RS_Contains(arg1: geometry, arg2: raster) -> boolean | RS_Contains(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_contains.qmd:19_

### Goal
Determines if the convex hull of the first spatial argument (raster or geometry) completely contains the second spatial argument (raster or geometry).

### Parameters
- `arg1` (`raster` or `geometry`): The primary spatial object whose convex hull is evaluated for containment.
- `arg2` (`geometry` or `raster`): The secondary spatial object to be checked for containment within the first argument's extent.

### Input
- **Data Types:** Accepts `raster` objects (e.g., from `RS_FromPath` or `RS_Example()`) and `geometry` objects (e.g., from `ST_Point`, `ST_GeomFromText`).
- **Preconditions:** If arguments have different Coordinate Reference Systems (CRS), the system attempts to transform the second argument into the CRS of the first. If transformation fails, both are transformed to WGS 84 as a fallback. If either argument lacks a CRS, the comparison proceeds without transformation.

### Output
Returns `boolean` — `true` if the convex hull of the first argument contains the second argument, otherwise `false`.

### Valid Call Patterns
```python
# Raster-Geometry containment check
sd.sql("SELECT RS_Contains(RS_Example(), ST_Point(0.5, 0.5, 4326))").to_arrow_table()

# Raster-Raster containment check
sd.sql("SELECT RS_Contains(RS_Example(), RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Contains` to evaluate spatial containment between rasters and geometries.
- Always ensure the first argument is the container and the second is the contained object.
- When comparing two rasters, the function implicitly compares their convex hulls.

### Prompt Snippet
```text
RS_Contains(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** If only one argument has a CRS defined, the operation may fail or produce unexpected results. Ensure both inputs have defined CRSes or both are undefined.
- **Argument Order:** Swapping the container and the contained object will return incorrect results (e.g., `RS_Contains(point, raster)` checks if the point contains the raster).
- **Raster Validity:** Ensure the raster is properly initialized (e.g., via `RS_FromPath` or `RS_Example`) before passing it to the function.

### Fix Code Hint
```python
# Ensure both inputs are in the same CRS if possible
sd.sql("""
    SELECT RS_Contains(
        raster_col, 
        ST_Transform(geom_col, 4326)
    ) FROM my_table
""").to_arrow_table()
```

## API Test: `sql:RS_ConvexHull`

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
- `arg1` (`raster`): The input raster object for which the convex hull geometry is to be calculated.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be properly initialized (e.g., via `RS_FromPath` or `RS_MakeEmptyRaster`). The operation assumes the raster has a defined spatial extent.

### Output
Returns `geometry` — A polygon representing the convex hull of the input raster's spatial footprint.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT RS_ConvexHull(raster_column) FROM my_raster_table").to_arrow_table()

# Example using the documented example pattern
sd.sql("SELECT RS_ConvexHull(RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_ConvexHull(raster_expression)` within a `sd.sql()` query.
- Ensure the input is a valid raster type; if the raster is loaded from a file, ensure the path is accessible to the SedonaDB engine.

### Prompt Snippet
```text
SELECT RS_ConvexHull(raster_input)
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type or a null value will result in a query execution error.
- **Uninitialized Raster:** Attempting to compute the hull of an empty or improperly loaded raster may return an empty geometry or trigger an error.
- **Memory Constraints:** While `RS_ConvexHull` is a metadata-based operation on the extent, ensure the input raster is correctly registered in the session.

### Fix Code Hint
```python
# Ensure the raster is loaded correctly before processing
sd.sql("SELECT RS_ConvexHull(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
# __CHECK__
```

## API Test: `sql:RS_DimNames`

### Signature
```python
RS_DimNames(arg1: raster) -> list<utf8> | RS_DimNames(arg1: raster, band: int) -> list<utf8>
```
_Source: source/docs/reference/sql/rs_dimnames.qmd:19_

### Goal
Retrieves the ordered list of dimension names (e.g., `["y", "x"]`, `["time", "y", "x"]`) associated with a specific raster band or the entire raster.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): Optional 1-based index of the band. If omitted, all bands must share identical dimension names.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be loaded or created (e.g., via `RS_FromPath` or `RS_Example`).
- **Constraints:** If the `band` index is provided, it must be a valid 1-based index; out-of-range indices or `null` return `null`.

### Output
Returns `list<utf8>` — An ordered list of strings representing the dimension names corresponding to the indices returned by `RS_Shape`.

### Valid Call Patterns
```python
# Example using the default band check
sd.sql("SELECT RS_DimNames(RS_Example())").to_arrow_table()

# Example specifying a band index
sd.sql("SELECT RS_DimNames(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_DimNames(raster)` to get dimensions for all bands if they are uniform.
- Use `RS_DimNames(raster, band)` when bands have heterogeneous dimensions or to explicitly target a specific band.
- Note that this function is **experimental** and subject to change.

### Prompt Snippet
```text
# To get dimension names for a raster:
sd.sql("SELECT RS_DimNames(my_raster_col) FROM my_table")
```

### Common Failure Modes
- **Dimension Mismatch:** If the `band` index is omitted and bands have different dimension names, the query will return an error.
- **Invalid Index:** Providing a `band` index that is less than 1 or greater than the number of bands will result in a `null` return value.
- **Experimental Status:** As an experimental feature, behavior may change in future SedonaDB versions.

### Fix Code Hint
```python
# If RS_DimNames fails due to band mismatch, specify the band explicitly:
sd.sql("SELECT RS_DimNames(raster_col, 1) FROM my_table")
```

__CHECK__

## API Test: `sql:RS_DimSize`

### Signature
```python
RS_DimSize(arg1: raster, dim_name: utf8) -> int64 | RS_DimSize(arg1: raster, dim_name: utf8, band: int) -> int64
```
_Source: source/docs/reference/sql/rs_dimsize.qmd:19_

### Goal
Retrieves the size of a specific named dimension (such as 'x', 'y', or 'time') from a raster or a specific band within that raster.

### Parameters
- `arg1` (`raster`): The input raster object to query.
- `dim_name` (`utf8`): The name of the dimension to query (e.g., 'x', 'y', 'time').
- `band` (`int`): Optional 1-based index of the band to query. If omitted, the function attempts to resolve the dimension across all bands.

### Input
- **Data:** A valid `raster` object (e.g., created via `RS_FromPath` or `RS_MakeEmptyRaster`).
- **Preconditions:** The dimension name must exist within the raster's metadata. If the band index is provided, it must be a valid 1-based integer.

### Output
Returns `int64` representing the size of the requested dimension. Returns `NULL` if the band index is provided but the band does not carry the specified dimension.

### Valid Call Patterns
```python
# Querying dimension size across all bands
sd.sql("SELECT RS_DimSize(RS_Example(), 'x')").to_arrow_table()

# Querying dimension size for a specific band
sd.sql("SELECT RS_DimSize(RS_Example(), 'y', 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_DimSize(raster, dim_name)` when querying the global dimension size.
- Use `RS_DimSize(raster, dim_name, band)` when targeting a specific band index.
- Always ensure the raster input is a valid SedonaDB raster object.
- Note that this function is marked as **experimental**.

### Prompt Snippet
```text
RS_DimSize(raster_obj, 'dimension_name')
RS_DimSize(raster_obj, 'dimension_name', band_index)
```

### Common Failure Modes
- **Dimension Mismatch:** If bands carry the dimension but disagree on its size, an error is returned; specify a `band` index to resolve.
- **Missing Dimension:** If no band carries the dimension (e.g., a typo in `dim_name`), an error is returned.
- **Invalid Band Index:** A null or out-of-range band index returns `NULL`.
- **Experimental Status:** Behavior may change; ensure the environment supports the current 0.5.0 experimental feature set.

### Fix Code Hint
```python
# If RS_DimSize fails due to heterogeneous band sizes, specify the band:
sd.sql("SELECT RS_DimSize(my_raster, 'x', 1) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_DimToBand`

### Signature
```python
RS_DimToBand(arg1: raster, dim_name: utf8) -> raster
```
_Source: source/docs/reference/sql/rs_dimtoband.qmd:19_

### Goal
Promotes a specified non-spatial dimension within a raster into individual bands, enabling standard band-based operations on previously dimension-indexed data.

### Parameters
- `arg1` (`raster`): The input raster object containing the dimension to be promoted.
- `dim_name` (`utf8`): The name of the non-spatial dimension (e.g., 'wavelength', 'time', 'band') to be converted into separate bands.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must contain the specified `dim_name` within at least one of its bands. Spatial dimensions (`x_dim`, `y_dim`) cannot be promoted.
- **Experimental:** This function is marked as experimental and may change without notice.

### Output
Returns a `raster` where each index along the specified dimension has been transformed into a distinct band, with the original dimension removed.

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT RS_DimToBand(RS_BandToDim(RS_Example(), 'band'), 'band')").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_DimToBand(raster, 'dimension_name')` syntax.
- Ensure the input raster is a valid SedonaDB raster object.
- Note that this function is experimental; verify the dimension name exists in the raster metadata before calling.

### Prompt Snippet
```text
RS_DimToBand(raster_object, 'dimension_name')
```

### Common Failure Modes
- **Typo in Dimension Name:** If no band contains the specified `dim_name`, the function will return an error.
- **Spatial Dimension Promotion:** Attempting to promote `x_dim` or `y_dim` is not supported and will fail.
- **Missing Dimension:** If the input raster does not contain the dimension, the operation fails.

### Fix Code Hint
```python
# Verify the dimension exists before promoting
# Ensure the raster is properly loaded via RS_FromPath or RS_Example()
result = sd.sql("SELECT RS_DimToBand(my_raster, 'target_dim')").to_arrow_table()
# __CHECK__ witness: verify the number of bands has increased relative to the dimension size
```

## API Test: `sql:RS_Envelope`

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
Computes the spatial extent of a raster object and returns it as a `geometry` (bounding box).

### Parameters
- `arg1` (`raster`): The input raster object for which the bounding box is to be calculated.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. Ensure the raster is properly initialized within the SedonaDB session before calling this function.

### Output
Returns a `geometry` object representing the rectangular envelope (bounding box) of the input raster.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_Envelope(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_Envelope(raster_expression)` in SQL queries.
- Ensure the input is a valid raster type; if the input is a file path, wrap it in a raster-loading function first.

### Prompt Snippet
```text
SELECT RS_Envelope(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a `geometry` or `string` type instead of a `raster` object.
- Attempting to call the function on a null or uninitialized raster object.
- Using the function in a context where the raster's georeference metadata is missing or invalid.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
sd.sql("SELECT RS_Envelope(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `sql:RS_Example`

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
Generates a predefined, simple in-database `raster` object to facilitate testing, demonstration, and verification of raster-based SQL operations.

### Parameters
_None._

### Input
No input parameters are required. This function is intended for use in standalone SQL queries or as a source for testing other raster-processing functions like `RS_Clip`, `RS_Resample`, or `RS_ZonalStats`.

### Output
Returns a `raster` object — a structured in-database grid representation containing sample data suitable for spatial analysis.

### Valid Call Patterns
```python
# Example: Using RS_Example as an input for a clipping operation
query = "SELECT RS_Width(RS_Clip(RS_Example(), 1, ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'))) AS width"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always call `RS_Example()` without arguments.
- Use this function when the user requests a sample raster for testing or when demonstrating raster-to-geometry interaction workflows.
- Do not attempt to pass parameters to this function, as it is a parameterless generator.

### Prompt Snippet
```text
Use RS_Example() to generate a test raster for your SedonaDB SQL query.
```

### Common Failure Modes
- **Contextual Misuse:** Attempting to pass arguments to `RS_Example()` will result in a SQL syntax error.
- **Experimental Limitations:** While `RS_Example()` is stable, operations performed on the resulting raster (such as `RS_Clip` or `RS_Resample`) are marked as experimental in SedonaDB 0.5.0 and may be subject to change or memory constraints.
- **CRS Mismatch:** If using the output of `RS_Example()` with other geometries, ensure the geometries are defined in a compatible CRS, as SedonaDB requires consistent CRS handling for spatial operations.

### Fix Code Hint
```python
# Ensure the function is called as a standalone SQL expression
sd.sql("SELECT RS_Example()").to_arrow_table()
```

## API Test: `sql:RS_FromGDALRaster`

### Signature
```python
RS_FromGDALRaster(content: binary) -> raster
```
_Source: source/docs/reference/sql/rs_fromgdalraster.qmd:19_

### Goal
Decodes binary raster data (such as GeoTIFF or PNG byte streams) into an in-database raster object with all band data materialized inline.

### Parameters
- `content` (`binary`): The raw byte stream of a raster file in any format supported by GDAL. A `NULL` input results in a `NULL` raster.

### Input
- **Data:** Binary byte stream representing a raster file.
- **Formats:** Any format readable by GDAL (e.g., GeoTIFF, PNG).
- **Preconditions:** The input must be a valid binary representation of a raster. This function is the inverse of `RS_AsGeoTiff` and the binary-content counterpart to `RS_FromPath`.

### Output
Returns `raster` — An in-database raster object with all band data materialized in memory.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT RS_Width(RS_FromGDALRaster(RS_AsGeoTiff(RS_Example())))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_FromGDALRaster(content)` to convert binary data into a raster.
- Ensure the input is a `binary` type column or literal.
- Use this function when you need to perform in-database raster analysis on binary data previously exported or stored as blobs.

### Prompt Snippet
```text
RS_FromGDALRaster(binary_column)
```

### Common Failure Modes
- **Invalid Binary:** Passing non-raster binary data or corrupted byte streams will cause the GDAL parser to fail.
- **Memory Limits:** Since this function materializes all band data inline, very large rasters may exceed memory limits (note: per-band limit is 4 GiB).
- **NULL Handling:** Passing a `NULL` input will return `NULL` rather than an empty raster.

### Fix Code Hint
```python
# Ensure the input is valid binary before calling
sd.sql("SELECT RS_FromGDALRaster(my_binary_col) FROM my_table WHERE my_binary_col IS NOT NULL")
```

## API Test: `sql:RS_FromPath`

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
Creates an out-of-database raster object by loading metadata from a specified file path, allowing for efficient processing without loading full pixel data into memory.

### Parameters
- `path` (`string`): The file system path to the raster file (e.g., GeoTIFF).

### Input
- **Data:** A valid path to a GDAL-readable raster file.
- **Preconditions:** The file must be accessible by the SedonaDB engine. The raster is treated as an "out-of-database" (out-db) object, meaning the engine references the file on disk rather than embedding the binary pixel data.

### Output
Returns a `raster` object containing the metadata and references to the source file bands.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT RS_FromPath('/path/to/raster.tiff')").to_arrow_table()
print("Raster loaded successfully")
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_FromPath` when the goal is to reference large raster files on disk to avoid memory overhead.
- Ensure the path provided is a valid string literal or column reference.
- Use this function as the primary input for subsequent raster operations like `RS_MetaData` or `RS_BandPath`.

### Prompt Snippet
```text
Use RS_FromPath(path) to initialize an out-of-database raster from a file path.
```

### Common Failure Modes
- **File Not Found:** Providing a path that does not exist or is inaccessible to the SedonaDB process.
- **Unsupported Format:** Providing a file path that is not a GDAL-readable raster format.
- **Permission Denied:** The SedonaDB process lacks read permissions for the specified file path.

### Fix Code Hint
```python
# Ensure the path is absolute or relative to the SedonaDB working directory
path = "/absolute/path/to/data.tiff"
query = f"SELECT RS_FromPath('{path}')"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:RS_GeoReference`

### Signature
```python
RS_GeoReference(arg1: raster) -> utf8 | RS_GeoReference(arg1: raster, format: utf8) -> utf8
```
_Source: source/docs/reference/sql/rs_georeference.qmd:19_

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

title: RS_GeoReference
description: >
  Returns the georeference metadata of raster as a string in GDAL or ESRI format
  as commonly seen in a world file. Default is GDAL if not specified. Both formats
  output six lines: scalex, skewy, skewx, scaley, upperleftx, upperlefty. In GDAL
  format the upper-left coordinates refer to the corner of the upper-left pixel,
  while in ESRI format they are shifted to the center of the upper-left pixel.
kernels:
  - returns: utf8
    args: [raster]
  - returns: utf8
    args:
    - raster
    - name: format
      type: utf8
      description: >
        Output format: 'GDAL' (default; alias 'PIXEL') reports the upper-left
        corner of the upper-left pixel; 'ESRI' (alias 'NODE') shifts the
        coordinates to the center of the upper-left pixel. The center shift maps
        through the full affine transform, so it is exact for skewed (rotated)
        rasters too.
---

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT RS_GeoReference(RS_Example());
```


### Goal
Returns the georeference metadata of a raster as a six-line string in GDAL or ESRI world file format.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `format` (`utf8`): The output format string. Use 'GDAL' (or 'PIXEL') for upper-left corner coordinates, or 'ESRI' (or 'NODE') for coordinates shifted to the center of the upper-left pixel. Defaults to 'GDAL'.

### Input
Requires a valid `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`.

### Output
Returns `utf8` — A string containing six lines representing the affine transformation: `scalex`, `skewy`, `skewx`, `scaley`, `upperleftx`, and `upperlefty`.

### Valid Call Patterns
```python
# Default GDAL format
sd.sql("SELECT RS_GeoReference(RS_Example())").to_arrow_table()

# Explicit ESRI format
sd.sql("SELECT RS_GeoReference(RS_Example(), 'ESRI')").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_GeoReference(raster_expression)` or `RS_GeoReference(raster_expression, format_string)`.
- Ensure the input is a valid raster object.
- When using the `format` parameter, only 'GDAL', 'PIXEL', 'ESRI', or 'NODE' are valid.

### Prompt Snippet
```text
Use RS_GeoReference(raster) to extract world file metadata.
```

### Common Failure Modes
- Passing a non-raster type (e.g., geometry) as the first argument.
- Providing an unsupported format string, which may result in runtime errors or unexpected default behavior.
- Attempting to call on a null or uninitialized raster object.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling
sd.sql("SELECT RS_GeoReference(raster_col) FROM my_raster_table WHERE raster_col IS NOT NULL").to_arrow_table()
```

## API Test: `sql:RS_GeoTransform`

### Signature
```python
RS_GeoTransform(arg1: raster) -> struct
```
_Source: source/docs/reference/sql/rs_geotransform.qmd:19_

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

title: RS_GeoTransform
description: >
  Returns the raster's geotransform decomposed into a struct of six doubles,
  matching Sedona Spark's RS_GeoTransform: magnitudeI and magnitudeJ (pixel
  sizes along the transformed i and j axes), thetaI (rotation of the raster in
  radians, positive clockwise), thetaIJ (angle from the transformed i axis to
  the transformed j axis in radians, positive counter-clockwise), and offsetX
  and offsetY (the upper-left corner of the upper-left pixel). For the raw
  affine coefficients instead of this decomposition, use the individual
  accessors (RS_ScaleX, RS_SkewX, RS_UpperLeftX, ...) or RS_GeoReference.
kernels:
  - returns: struct
    args: [raster]
---

## Examples

```sql
SELECT RS_GeoTransform(RS_Example());
```


### Goal
Decomposes a raster's geotransform affine coefficients into a structured set of six human-readable spatial parameters (pixel sizes, rotation angles, and origin coordinates).

### Parameters
- `arg1` (`raster`): The input raster object for which the geotransform metadata is to be extracted.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. No specific CRS preconditions are required, but the raster must be properly initialized with georeferencing metadata.

### Output
Returns a `struct` containing six `double` values:
1. `magnitudeI`: Pixel size along the transformed i-axis.
2. `magnitudeJ`: Pixel size along the transformed j-axis.
3. `thetaI`: Rotation of the raster in radians (positive clockwise).
4. `thetaIJ`: Angle from the transformed i-axis to the transformed j-axis in radians (positive counter-clockwise).
5. `offsetX`: X-coordinate of the upper-left corner of the upper-left pixel.
6. `offsetY`: Y-coordinate of the upper-left corner of the upper-left pixel.

### Valid Call Patterns
```python
# Using the Python SQL connection
sd.sql("SELECT RS_GeoTransform(raster_col) FROM my_raster_table").to_arrow_table()

# Using the documented example
sd.sql("SELECT RS_GeoTransform(RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_GeoTransform(raster_expression)` in SQL queries.
- If the user requires raw affine coefficients instead of the decomposed struct, suggest using individual accessors like `RS_ScaleX`, `RS_SkewX`, or `RS_UpperLeftX` instead.

### Prompt Snippet
```text
SELECT RS_GeoTransform(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input Type:** Passing a `geometry` or non-raster type will result in a function signature mismatch error.
- **Uninitialized Raster:** Calling this on a raster that lacks georeferencing metadata may return default or null values depending on the raster's internal state.
- **Decomposition vs. Raw:** Users expecting raw affine coefficients (e.g., for GDAL compatibility) may be confused by the returned struct; remind them that this function performs a decomposition.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
# If the raster is missing metadata, verify it was loaded correctly via RS_FromPath
result = sd.sql("SELECT RS_GeoTransform(r) FROM (SELECT RS_FromPath('path/to/raster.tif') as r) t").to_arrow_table()
print(result)
__CHECK__
```

## API Test: `sql:RS_Height`

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
- `arg1` (`raster`): The input raster object for which the height dimension is to be queried.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. No specific preconditions regarding band count or pixel type are required for this metadata accessor.

### Output
Returns `bigint` — The total number of pixels along the vertical axis (height) of the raster.

### Valid Call Patterns
```python
# Using the Python SQL connection
result = sd.sql("SELECT RS_Height(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_Height(raster_expression)` in SQL queries.
- Ensure the input is a valid raster object; if the input is a file path, wrap it in a raster-loading function like `RS_FromPath`.

### Prompt Snippet
```text
SELECT RS_Height(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a `geometry` type instead of a `raster` type will result in a type mismatch error.
- Calling the function on a null raster object will return `NULL`.

### Fix Code Hint
```python
# Ensure the input is a valid raster
sd.sql("SELECT RS_Height(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `sql:RS_Intersects`

### Signature
```python
RS_Intersects(arg1: raster, arg2: geometry) -> boolean | RS_Intersects(arg1: geometry, arg2: raster) -> boolean | RS_Intersects(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_intersects.qmd:19_

### Goal
Determines if the convex hull of a raster intersects with a geometry or another raster's convex hull to facilitate spatial filtering and analysis.

### Parameters
- `arg1` (`raster`): The primary raster object to evaluate; its convex hull is used for the intersection test.
- `arg2` (`geometry`): The secondary geometry or raster object to test against the primary argument.

### Input
- **Data:** Raster objects (e.g., from `RS_FromPath`) and OGC-compliant geometries (e.g., `ST_Point`, `ST_GeomFromText`).
- **Preconditions:** If arguments have different CRSes, the system attempts to transform the geometry or the second raster's extent into the first raster's CRS. If either argument lacks a CRS, the comparison is performed directly without transformation.
- **Densification:** When CRSes differ, the raster footprint is densified with approximately 10 points per edge to ensure accurate reprojection.

### Output
Returns `boolean` — `true` if the convex hulls of the two provided inputs intersect, otherwise `false`.

### Valid Call Patterns
```python
# Example 1: Raster and Geometry
sd.sql("SELECT RS_Intersects(RS_Example(), ST_Point(0.5, 0.5, 4326))").to_arrow_table()

# Example 2: Raster and Raster
sd.sql("SELECT RS_Intersects(RS_Example(), RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_Intersects(arg1, arg2)` as a standalone SQL function call within `sd.sql()`.
- Ensure that if one argument is a raster and the other is a geometry, the geometry is valid OGC WKT/WKB.
- If comparing two rasters, remember that the operation compares their convex hulls.

### Prompt Snippet
```text
RS_Intersects(raster_obj, geometry_obj)
```

### Common Failure Modes
- **CRS Mismatch:** If exactly one argument has a CRS defined, the operation may fail or produce unexpected results; ensure both have defined CRSes or both are undefined.
- **Memory Constraints:** While `RS_Intersects` is a predicate, ensure the input rasters are valid objects; attempting to process extremely large, un-indexed raster datasets may impact performance.
- **Invalid Geometry:** Passing a malformed WKT string to the geometry argument will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure CRS consistency if the operation fails
sd.sql("SELECT RS_Intersects(raster_col, ST_SetSRID(geom_col, 4326)) FROM my_table")
```

## API Test: `sql:RS_IsEmpty`

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
Determines if a raster object contains zero visible volume by checking if any of its dimensions have a length of zero.

### Parameters
- `arg1` (`raster`): The raster object to inspect for visible-volume emptiness.

### Input
The input must be a valid SedonaDB `raster` object. Note that this function checks for visible-volume emptiness (cell count), not byte-emptiness; a lazy-loaded raster that has not yet resolved its pixels is not considered empty.

### Output
Returns a `boolean` value: `true` if the raster has zero visible volume (at least one dimension has length 0), and `false` otherwise.

### Valid Call Patterns
```python
# Verify emptiness of a sample raster
result = sd.sql("SELECT RS_IsEmpty(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_IsEmpty(raster_expression)` to check if a raster has zero visible volume.
- Do not confuse this with byte-level emptiness; lazy rasters are not empty.
- Ensure the input is a valid `raster` type.

### Prompt Snippet
```text
SELECT RS_IsEmpty(your_raster_column) FROM your_table;
```

### Common Failure Modes
- Passing a non-raster type (e.g., `geometry` or `string`) will result in a type mismatch error.
- Misinterpreting a lazy-loaded raster as empty because it has not yet materialized its pixel data.

### Fix Code Hint
```python
# Ensure the input is a materialized or valid raster object
sd.sql("SELECT RS_IsEmpty(my_raster) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_MakeEmptyRaster`

### Signature
```python
RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, extent: geometry) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, extent: geometry) -> raster | RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, upper_left_x: double, upper_left_y: double, cell_size: double) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, upper_left_x: double, upper_left_y: double, cell_size: double) -> raster | RS_MakeEmptyRaster(num_bands: integer, width: integer, height: integer, upper_left_x: double, upper_left_y: double, scale_x: double, scale_y: double, skew_x: double, skew_y: double, srid: integer) -> raster | RS_MakeEmptyRaster(num_bands: integer, band_data_type: string, width: integer, height: integer, upper_left_x: double, upper_left_y: double, scale_x: double, scale_y: double, skew_x: double, skew_y: double, srid: integer) -> raster
```
_Source: source/docs/reference/sql/rs_makeemptyraster.qmd:19_

### Goal
Creates an in-database, zero-filled raster grid from specified dimensions and spatial placement parameters, serving as a template for rasterization or spatial analysis.

### Parameters
- `num_bands` (`integer`): Number of zero-filled bands to create; use `0` for a bandless grid template.
- `band_data_type` (`string`): Pixel type (e.g., 'uint8', 'float64', 'I', 'D'); defaults to 'float64'.
- `width` (`integer`): Grid width in pixels.
- `height` (`integer`): Grid height in pixels.
- `upper_left_x` (`double`): X coordinate of the upper-left corner of the upper-left pixel.
- `upper_left_y` (`double`): Y coordinate of the upper-left corner of the upper-left pixel.
- `scale_x` (`double`): Pixel width (or cell size).
- `scale_y` (`double`): Pixel height (negative for north-up rasters).
- `skew_x` (`double`): X-axis skew.
- `skew_y` (`double`): Y-axis skew.
- `srid` (`integer`): EPSG code of the raster's CRS; use `0` for no CRS.

### Input
- **Extent-based:** Requires a `geometry` (or `geography`) object defining the bounding box.
- **Coordinate-based:** Requires explicit upper-left coordinates and cell size or affine transform parameters.
- **Preconditions:** The envelope must have a positive width and height. If using `extent`, the raster inherits the geometry's CRS.

### Output
Returns a `raster` object containing zero-filled bands (or no bands if `num_bands` is 0).

### Valid Call Patterns
```python
# Create a 400x400 uint8 grid from an extent
sd.sql("""
  SELECT RS_MakeEmptyRaster(1, 'uint8', 400, 400, ST_MakeEnvelope(0, 0, 400, 400, 4326))
""").to_arrow_table()

# Create a grid from upper-left corner and cell size
sd.sql("SELECT RS_MakeEmptyRaster(2, 10, 10, 0.0, 0.0, 1.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Always specify `num_bands` as the first argument.
- If `band_data_type` is omitted, the system defaults to `float64`.
- Ensure the `extent` geometry has a positive area.
- Use `0` for `srid` if no coordinate system is required.

### Prompt Snippet
```text
Use RS_MakeEmptyRaster(num_bands, width, height, extent) for geometry-based grids, or RS_MakeEmptyRaster(num_bands, width, height, upper_left_x, upper_left_y, cell_size) for coordinate-based grids.
```

### Common Failure Modes
- **Invalid Data Type:** SedonaDB raises an error for unrecognized `band_data_type` strings (unlike SedonaSpark's silent fallback).
- **Negative Dimensions:** Providing non-positive width or height will result in an error.
- **CRS Mismatch:** Operations involving multiple rasters or geometries require consistent CRS; ensure `extent` geometries are correctly projected.
- **Memory Limits:** A single band is limited to 4 GiB; ensure `width * height * bytes-per-pixel` does not exceed system memory.

### Fix Code Hint
```python
# Ensure band_data_type is a valid string and dimensions are positive
query = "SELECT RS_MakeEmptyRaster(1, 'uint8', 100, 100, ST_MakeEnvelope(0, 0, 10, 10, 4326))"
result = sd.sql(query).to_arrow_table()
print("Raster created successfully.")
# __CHECK__
```

## API Test: `sql:RS_MetaData`

### Signature
```python
RS_MetaData(arg1: raster) -> struct
```
_Source: source/docs/reference/sql/rs_metadata.qmd:19_

### Goal
`RS_MetaData` extracts comprehensive georeferencing, dimensional, and structural metadata from a raster object into a structured format for inspection.

### Parameters
- `arg1` (`raster`): The input raster object to be inspected.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_Example`).
- **Preconditions:** The raster must be successfully loaded into the SedonaDB session.

### Output
Returns `struct` — A structured object containing:
- `upperLeftX`, `upperLeftY`: Origin coordinates of the raster geotransform.
- `gridWidth`, `gridHeight`: Raster dimensions in pixels.
- `scaleX`, `scaleY`: Pixel scale factors.
- `skewX`, `skewY`: Geotransform skew terms.
- `srid`: The SRID of the raster (defaults to `0` if not available).
- `numSampleDimensions`: The total number of bands.
- `tileWidth`, `tileHeight`: GDAL block size derived tile dimensions (returns `0` for rasters with no bands).

### Valid Call Patterns
```python
# Example using Python API
result = sd.sql("SELECT RS_MetaData(RS_Example())").to_arrow_table()

# Example using SQL syntax
query = """
SELECT meta."gridWidth", meta."gridHeight"
FROM (
  SELECT RS_MetaData(RS_Example()) AS meta
) q
"""
result = sd.sql(query).to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_MetaData(raster_expression)` as a standalone function call within a `SELECT` statement.
- Access the returned struct fields using standard SQL dot notation or alias referencing (e.g., `meta."gridWidth"`).
- Ensure the input is a valid raster object; if loading from a file, use `RS_FromPath`.

### Prompt Snippet
```text
SELECT RS_MetaData(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input:** Passing a non-raster type (e.g., `geometry` or `string`) to the function will result in a type mismatch error.
- **Null Input:** Passing a `NULL` raster will return `NULL` for the entire struct.
- **Missing Bands:** If the raster has no bands, `tileWidth` and `tileHeight` will be `0`, which may cause logic errors in downstream processing if not handled.

### Fix Code Hint
```python
# Ensure the input is a valid raster before calling metadata
sd.sql("SELECT RS_MetaData(raster_col) FROM my_raster_table WHERE raster_col IS NOT NULL")
```

## API Test: `sql:RS_MinConvexHull`

### Signature
```python
RS_MinConvexHull(arg1: raster) -> geometry | RS_MinConvexHull(arg1: raster, band: integer) -> geometry
```
_Source: source/docs/reference/sql/rs_minconvexhull.qmd:19_

### Goal
Computes the footprint of a raster by returning a polygon representing the smallest pixel-aligned rectangle that encompasses all pixels not equal to the specified band's nodata value.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `band` (`integer`): The 1-based index of the band to evaluate for nodata values. If omitted, a pixel is considered to hold data if any band contains data at that location.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must have defined nodata values if trimming is desired; otherwise, the function spans the entire grid. Only 2-D rasters are supported.

### Output
Returns `geometry` — A polygon representing the footprint of the raster's data pixels, aligned to the raster's grid axes. Returns `null` if no pixel holds data.

### Valid Call Patterns
```python
# Basic usage
sd.sql("SELECT RS_MinConvexHull(RS_Example())").to_arrow_table()

# Usage with specific band index
sd.sql("SELECT RS_MinConvexHull(RS_SetBandNoDataValue(RS_Example(), 1, 1), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_MinConvexHull` to extract the bounding footprint of valid raster data.
- Always specify the band index if the raster has multiple bands and you only wish to evaluate the footprint of a specific band.
- Remember that the result is a grid-aligned rectangle, not a tight convex hull of the pixel values themselves.

### Prompt Snippet
```text
RS_MinConvexHull(raster_obj)
RS_MinConvexHull(raster_obj, band_index)
```

### Common Failure Modes
- **Unsupported Dimensions:** Passing a raster that is not 2-D will result in an error.
- **No Data:** If all pixels in the specified band (or all bands, if no index is provided) are equal to the nodata value, the function returns `null`.
- **Missing Nodata:** If a band does not have a nodata value defined, the function will treat the entire grid as valid data.

### Fix Code Hint
```python
# Ensure nodata is set before calling if you need to trim specific values
sd.sql("SELECT RS_MinConvexHull(RS_SetBandNoDataValue(my_raster, 1, 0), 1)").to_arrow_table()
```

## API Test: `sql:RS_NumBands`

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
Returns the total count of bands contained within a specified SedonaDB raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to retrieve the band count.

### Input
- **Data:** A valid raster object (e.g., created via `RS_FromPath`, `RS_FromGDALRaster`, or `RS_MakeEmptyRaster`).
- **Preconditions:** The input must be a valid, non-null raster object.

### Output
Returns `uint32` — The number of bands present in the raster as an unsigned 32-bit integer.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_NumBands(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_NumBands(raster_expression)` in SQL queries.
- Ensure the input is a valid raster type; if the raster is derived from a file, ensure it has been successfully loaded into the session.

### Prompt Snippet
```text
SELECT RS_NumBands(<raster_column_or_function>)
```

### Common Failure Modes
- **Null Input:** Passing a `NULL` or non-raster type will result in a query execution error.
- **Invalid Raster:** If the raster object was not correctly initialized or loaded (e.g., a failed `RS_FromPath`), the function will fail.

### Fix Code Hint
```python
# Ensure the raster is valid before calling RS_NumBands
sd.sql("SELECT RS_NumBands(raster_col) FROM my_raster_table WHERE raster_col IS NOT NULL")
```

## API Test: `sql:RS_NumDimensions`

### Signature
```python
RS_NumDimensions(arg1: raster) -> int32 | RS_NumDimensions(arg1: raster, band: int) -> int32
```
_Source: source/docs/reference/sql/rs_numdimensions.qmd:19_

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

title: RS_NumDimensions
description: Returns the number of dimensions in a raster band.
kernels:
  - returns: int32
    args: [raster]
  - returns: int32
    args:
    - raster
    - name: band
      type: int
      description: Band index (1-based). A null or out-of-range index returns null.
---

::: callout-warning
**Experimental.** This function is experimental; its behavior may change without notice.
:::

## Description

Returns the number of dimensions in a raster band. A standard 2D raster has 2
dimensions (`y` and `x`). N-dimensional rasters (e.g., from Zarr or NetCDF
sources) may have additional dimensions such as `time`, `pressure`, or
`wavelength`.

When the band index is omitted, all bands must agree on the number of
dimensions; if they disagree, an error is returned prompting the user to
specify a band index.

## Examples

```sql
SELECT RS_NumDimensions(RS_Example());
```


### Goal
Determines the count of spatial and non-spatial dimensions (e.g., x, y, time, pressure) associated with a specific raster band or the entire raster object.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): Optional 1-based index of the specific band to query.

### Input
- **Data:** A valid SedonaDB `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Preconditions:** If the `band` index is omitted, all bands within the raster must share the same number of dimensions; otherwise, the operation will fail.

### Output
Returns `int32` — The total number of dimensions (e.g., 2 for standard 2D rasters, >2 for multi-dimensional data like NetCDF).

### Valid Call Patterns
```python
# Example 1: Inspecting dimensions of a default raster
sd.sql("SELECT RS_NumDimensions(RS_Example())").to_arrow_table()

# Example 2: Inspecting dimensions of a specific band (1-based)
sd.sql("SELECT RS_NumDimensions(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_NumDimensions(raster)` or `RS_NumDimensions(raster, band)` as the SQL function call.
- Remember that `band` is 1-based.
- If the user does not specify a band, ensure the raster is expected to have uniform dimensions across all bands.

### Prompt Snippet
```text
Use RS_NumDimensions(raster_column) to get the dimension count. If the raster has multiple bands with different dimensions, use RS_NumDimensions(raster_column, band_index).
```

### Common Failure Modes
- **Dimension Mismatch:** Calling without a `band` index on a multi-band raster where bands have differing dimension counts.
- **Out-of-Range Index:** Providing a `band` index that does not exist in the raster returns `null`.
- **Experimental Status:** As an experimental feature, behavior may change; ensure the SedonaDB environment is updated to 0.5.0.

### Fix Code Hint
```python
# If RS_NumDimensions(raster) fails due to inconsistent band dimensions:
# Explicitly query the dimension of a specific band:
sd.sql("SELECT RS_NumDimensions(raster_col, 1) FROM my_raster_table").to_arrow_table()
```

## API Test: `sql:RS_PixelAsCentroid`

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
Computes the geographic coordinate of the center of a specific pixel within a raster, returning it as a `Point` geometry.

### Parameters
- `arg1` (`raster`): The source raster object from which to extract the pixel location.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
Requires a valid `raster` object (e.g., created via `RS_FromPath` or `RS_MakeEmptyRaster`). The `colX` and `rowY` indices must be within the bounds of the raster dimensions (1 to width/height).

### Output
Returns a `geometry` object representing the `Point` at the center of the specified pixel.

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT RS_PixelAsCentroid(RS_Example(), 1, 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_PixelAsCentroid` when you need to convert a specific raster pixel coordinate into a spatial `Point` geometry.
- Ensure `colX` and `rowY` are provided as integers.
- Note that indices are 1-based.

### Prompt Snippet
```text
RS_PixelAsCentroid(raster_object, col_index, row_index)
```

### Common Failure Modes
- **Out of Bounds:** Providing `colX` or `rowY` values that exceed the raster's width or height will result in an error.
- **Invalid Raster:** Passing a null or improperly initialized raster object.
- **Indexing:** Using 0-based indexing instead of the required 1-based indexing will return the incorrect pixel or fail.

### Fix Code Hint
```python
# Ensure indices are within bounds by checking dimensions first
# Example: SELECT RS_PixelAsCentroid(raster, 1, 1) FROM table WHERE 1 <= 1 AND 1 <= RS_Width(raster)
```

## API Test: `sql:RS_PixelAsPoint`

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
`RS_PixelAsPoint` computes the spatial coordinate of the upper-left corner of a specific pixel within a raster, returning it as a `geometry` Point.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the coordinate.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
The input must be a valid SedonaDB `raster` object. The `colX` and `rowY` parameters must be integers representing valid pixel indices within the raster's dimensions.

### Output
Returns a `geometry` object representing the Point at the upper-left corner of the specified pixel in the raster's coordinate reference system.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_PixelAsPoint(RS_Example(), 1, 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
When calling `RS_PixelAsPoint`, ensure the input is a valid `raster` type. Use 1-based indexing for `colX` and `rowY`. The function returns a `geometry` type, which can be further processed by other `ST_*` spatial operations.

### Prompt Snippet
```text
RS_PixelAsPoint(raster_object, column_index, row_index)
```

### Common Failure Modes
- **Out of Bounds:** Providing `colX` or `rowY` values that exceed the raster's width or height will result in an error or invalid geometry.
- **Invalid Raster:** Passing a non-raster object or a null raster will cause the query to fail.
- **Indexing:** Using 0-based indexing instead of the required 1-based indexing will return the incorrect pixel location.

### Fix Code Hint
```python
# Ensure indices are within bounds using RS_Width and RS_Height
# Example:
# SELECT RS_PixelAsPoint(raster, 1, 1) 
# FROM (SELECT RS_FromPath('path/to/raster.tif') AS raster) r
# WHERE 1 <= 1 AND 1 <= RS_Width(raster) AND 1 <= RS_Height(raster);
```

## API Test: `sql:RS_PixelAsPolygon`

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
Computes the spatial extent of a specific pixel within a raster, returning it as a `geometry` polygon.

### Parameters
- `arg1` (`raster`): The source raster object from which to extract the pixel geometry.
- `colX` (`integer`): The 1-based column index of the pixel.
- `rowY` (`integer`): The 1-based row index of the pixel.

### Input
The caller must provide a valid SedonaDB `raster` object. The `colX` and `rowY` indices must be within the bounds of the raster dimensions (as determined by `RS_Width` and `RS_Height`).

### Output
Returns a `geometry` object representing the bounding polygon of the pixel at the specified grid coordinates.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
result = sd.sql("SELECT RS_PixelAsPolygon(RS_Example(), 1, 1)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use the 1-based indexing convention for `colX` and `rowY`.
- Ensure the input raster is a valid SedonaDB raster object.
- The function returns a `geometry` type; ensure downstream operations are compatible with OGC-compliant polygons.

### Prompt Snippet
```text
Use RS_PixelAsPolygon(raster_obj, col, row) to convert a specific pixel coordinate into a geometry polygon.
```

### Common Failure Modes
- **Out of Bounds:** Providing `colX` or `rowY` values that exceed the raster's width or height will result in an error.
- **Invalid Raster:** Passing a non-raster object or a null raster will cause the operation to fail.
- **Indexing:** Using 0-based indexing instead of the required 1-based indexing will return the incorrect pixel or trigger an out-of-bounds error.

### Fix Code Hint
```python
# Ensure indices are within bounds before calling
# Example: Check dimensions first
df = sd.sql("SELECT RS_Width(r), RS_Height(r) FROM (SELECT RS_Example() as r) t").to_arrow_table()
# Then call RS_PixelAsPolygon with valid 1-based indices
```

## API Test: `sql:RS_Polygonize`

### Signature
```python
RS_Polygonize(arg1: raster, band: integer) -> list
```
_Source: source/docs/reference/sql/rs_polygonize.qmd:19_

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

title: RS_Polygonize
description: Converts a raster band into polygons grouped by connected pixel value regions.
kernels:
  - returns: list
    args:
    - raster
    - name: band
      type: integer
---

## Description

Converts the specified raster `band` into vector polygons, returning one item per
connected region of pixels that shares the same value.

Each returned list item contains:

- `geom`: the polygon geometry encoded as WKB
- `value`: the pixel value represented by that polygon

## Examples

```sql
SELECT RS_Polygonize(RS_Example(), 1);
```


### Goal
`RS_Polygonize` performs vectorization by converting a specific raster band into a set of polygons, where each polygon represents a contiguous region of pixels sharing an identical value.

### Parameters
- `arg1` (`raster`): The input raster object to be processed.
- `band` (`integer`): The 1-based index of the raster band to vectorize.

### Input
- **Data:** A valid SedonaDB `raster` object (e.g., created via `RS_FromPath` or `RS_Example`).
- **Preconditions:** The raster must contain the specified band index. Band indices are 1-based.

### Output
Returns `list` — A list of structures, where each item contains a `geom` (polygon geometry in WKB format) and a `value` (the pixel value associated with that geometry).

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT RS_Polygonize(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use the 1-based index for the `band` parameter.
- Ensure the input `arg1` is a valid raster object.
- Note that the output is a list of complex structures; if further processing is required, use SQL to expand or filter the results.

### Prompt Snippet
```text
SELECT RS_Polygonize(raster_column, 1) FROM table_name;
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index that does not exist in the raster (e.g., 0 or a number greater than `RS_NumBands`).
- **Null Raster:** Passing a null or invalid raster object will result in an execution error.
- **Memory Constraints:** Polygonizing very large, high-resolution rasters with many unique pixel regions may exceed memory limits.

### Fix Code Hint
```python
# Ensure the band exists before calling
# Verify band index is within [1, RS_NumBands(raster)]
sd.sql("SELECT RS_Polygonize(raster, 1) FROM my_raster_table WHERE RS_NumBands(raster) >= 1").to_arrow_table()
```

## API Test: `sql:RS_RasterToWorldCoord`

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
Converts specific raster pixel grid coordinates into their corresponding spatial world coordinates represented as a `geometry` point.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform metadata required for coordinate mapping.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
The caller must provide a valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example()`). Pixel coordinates are 1-based; values outside the grid dimensions are permitted and will be extrapolated based on the raster's geotransform.

### Output
Returns a `geometry` point representing the world coordinate of the upper-left corner of the specified pixel.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
result = sd.sql("SELECT RS_RasterToWorldCoord(RS_Example(), 10, 20)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use 1-based indexing for `x` and `y` parameters.
- Ensure the input raster is a valid SedonaDB `raster` type.
- Use the function within a `sd.sql()` query context.

### Prompt Snippet
```text
SELECT RS_RasterToWorldCoord(raster_column, x_val, y_val) FROM table_name;
```

### Common Failure Modes
- Providing 0-based indices (e.g., `0, 0`) will result in coordinates extrapolated from the geotransform rather than the first pixel.
- Passing a non-raster object as the first argument will cause a type mismatch error.
- Rasters missing geotransform metadata will fail to produce valid world coordinates.

### Fix Code Hint
```python
# Ensure indices are 1-based
# If you have 0-based indices, add 1 before passing to the function
sd.sql("SELECT RS_RasterToWorldCoord(raster, x + 1, y + 1) FROM my_table")
```

## API Test: `sql:RS_RasterToWorldCoordX`

### Signature
```python
RS_RasterToWorldCoordX(arg1: raster, x: integer, y: integer) -> double
```
_Source: source/docs/reference/sql/rs_rastertoworldcoordx.qmd:19_

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

title: RS_RasterToWorldCoordX
description: Converts raster pixel coordinates to world X coordinate.
kernels:
  - returns: double
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
SELECT RS_RasterToWorldCoordX(RS_Example(), 10, 20);
```


### Goal
Converts a specific raster pixel coordinate (x, y) into its corresponding world X coordinate based on the raster's geotransform.

### Parameters
- `arg1` (`raster`): The input raster object containing georeferencing information.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
The caller must provide a valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`). Pixel coordinates are 1-based; `(1, 1)` represents the upper-left corner of the upper-left pixel.

### Output
Returns `double` — The world X coordinate corresponding to the upper-left corner of the specified pixel.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_RasterToWorldCoordX(RS_Example(), 10, 20) as world_x").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use 1-based indexing for `x` and `y` parameters.
- Ensure the input `arg1` is a valid raster object.
- Note that coordinates outside the grid (including 0) are valid and will be extrapolated using the raster's geotransform.

### Prompt Snippet
```text
RS_RasterToWorldCoordX(raster_object, x_index, y_index)
```

### Common Failure Modes
- Providing 0-based indices instead of 1-based indices, leading to unexpected coordinate results.
- Passing a non-raster object as the first argument.
- Attempting to use this function on a raster that lacks a valid geotransform.

### Fix Code Hint
```python
# Ensure indices are 1-based
# If you have 0-based indices, add 1 before passing to the function
x_1based = x_0based + 1
y_1based = y_0based + 1
sd.sql(f"SELECT RS_RasterToWorldCoordX(raster, {x_1based}, {y_1based})").to_arrow_table()
```

## API Test: `sql:RS_RasterToWorldCoordY`

### Signature
```python
RS_RasterToWorldCoordY(arg1: raster, x: integer, y: integer) -> double
```
_Source: source/docs/reference/sql/rs_rastertoworldcoordy.qmd:19_

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

title: RS_RasterToWorldCoordY
description: Converts raster pixel coordinates to world Y coordinate.
kernels:
  - returns: double
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
SELECT RS_RasterToWorldCoordY(RS_Example(), 10, 20);
```


### Goal
Converts a specific raster pixel coordinate (x, y) into its corresponding world Y coordinate based on the raster's geotransform.

### Parameters
- `arg1` (`raster`): The input raster object containing georeferencing information.
- `x` (`integer`): The 1-based column index of the pixel.
- `y` (`integer`): The 1-based row index of the pixel.

### Input
The caller must provide a valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`). Pixel coordinates are 1-based; `(1, 1)` represents the upper-left pixel. Coordinates outside the grid boundaries are supported and will be extrapolated using the raster's geotransform.

### Output
Returns `double` — The world Y coordinate corresponding to the upper-left corner of the specified pixel.

### Valid Call Patterns
```python
# Example using the Python SQL interface
result = sd.sql("SELECT RS_RasterToWorldCoordY(RS_Example(), 10, 20)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use 1-based indexing for `x` and `y` parameters.
- Ensure the input `arg1` is a valid raster object.
- Use this function when you need to map pixel-space locations to geographic or projected coordinate systems.

### Prompt Snippet
```text
RS_RasterToWorldCoordY(raster_object, x_index, y_index)
```

### Common Failure Modes
- Providing 0-based indices instead of 1-based indices, which will result in coordinates shifted by one pixel.
- Passing a non-raster object as the first argument.
- Attempting to use this function on a raster that lacks valid georeferencing metadata.

### Fix Code Hint
```python
# Ensure indices are 1-based
# If you have 0-based indices (col, row), use (col + 1, row + 1)
query = "SELECT RS_RasterToWorldCoordY(raster_col, col + 1, row + 1) FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:RS_ReplaceBandNoDataValue`

### Signature
```python
RS_ReplaceBandNoDataValue(arg1: raster, band: int, nodata: float64) -> raster
```
_Source: source/docs/reference/sql/rs_replacebandnodatavalue.qmd:19_

### Goal
Updates a raster band's nodata value while simultaneously remapping all existing nodata pixels to the new value, ensuring that the set of pixels identified as "nodata" remains consistent.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`int`): The 1-based index of the band to update.
- `nodata` (`float64`): The new nodata value to assign; it must be compatible with the band's underlying data type.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The specified band must already have a defined nodata value; otherwise, the operation will result in an error.
- **Formats:** Supports GDAL-readable raster formats loaded into the SedonaDB environment.

### Output
Returns `raster` — A new raster object where the specified band's nodata value is updated and all pixels previously marked as nodata are rewritten to the new value.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_ReplaceBandNoDataValue(RS_Example(), 1, 200)").to_arrow_table()
print(__CHECK__ := result)
```

### LLM Instruction Prompt
- Always use `RS_ReplaceBandNoDataValue(raster, band, nodata)` as a standalone SQL function call.
- Ensure the `band` index is 1-based.
- Verify that the `nodata` value is provided as a `float64` and is compatible with the target band's data type.
- Do not use this function on rasters that lack an existing nodata value.

### Prompt Snippet
```text
RS_ReplaceBandNoDataValue(raster_obj, 1, 255.0)
```

### Common Failure Modes
- **Missing Nodata:** Calling the function on a band that does not have an existing nodata value will trigger an error.
- **Type Mismatch:** Providing a `nodata` value that cannot be represented by the band's pixel type (e.g., assigning a large float to an `int8` band).
- **Invalid Dimensions:** Attempting to use this function on non-2-D rasters.
- **Null Inputs:** Passing a null raster, band index, or nodata value will result in a null raster output.

### Fix Code Hint
```python
# Ensure the band has a nodata value before replacement
# If unknown, check with RS_BandNoDataValue(raster, band) first
sd.sql("SELECT RS_ReplaceBandNoDataValue(my_raster, 1, 200.0)").to_arrow_table()
```

## API Test: `sql:RS_ReprojectMatch`

### Signature
```python
RS_ReprojectMatch(raster: raster, reference: raster) -> raster | RS_ReprojectMatch(raster: raster, reference: raster, algorithm: string) -> raster
```
_Source: source/docs/reference/sql/rs_reprojectmatch.qmd:19_

### Goal
`RS_ReprojectMatch` reprojects an input raster to match the CRS, grid, resolution, and extent of a reference raster.

### Parameters
- `raster` (`raster`): The source raster to be reprojected.
- `reference` (`raster`): The target raster whose grid, CRS, and extent define the output geometry.
- `algorithm` (`string`): Optional resampling method (case-insensitive): `NearestNeighbor` (default), `Bilinear`, `Cubic` (alias `Bicubic`), `CubicSpline`, `Lanczos`, `Average`, or `Mode`.

### Input
- **Data:** GDAL-readable raster objects.
- **Preconditions:** 
    - Both rasters must either both have a CRS or both lack one; mixed CRS states will cause an error.
    - `Int64` and `UInt64` pixel types are not supported; cast to `Int32` or `Float64` before calling.
    - The entire input raster is loaded into memory.

### Output
Returns a new `raster` object with the spatial properties of the `reference` raster and the recomputed pixel values of the input `raster`.

### Valid Call Patterns
```python
# Identity reproject (example)
sd.sql("SELECT RS_Width(RS_ReprojectMatch(RS_Example(), RS_Example()))").to_arrow_table()

# Reproject with explicit algorithm
sd.sql("SELECT RS_Height(RS_ReprojectMatch(RS_Example(), RS_Example(), 'Bilinear'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_ReprojectMatch(raster, reference)` or `RS_ReprojectMatch(raster, reference, algorithm)`.
- Ensure both input rasters have consistent CRS status (both defined or both undefined).
- If the input raster is `Int64` or `UInt64`, instruct the user to cast it to `Int32` or `Float64` first.
- Remember that the reference raster's pixel values are ignored; only its grid metadata is used.

### Prompt Snippet
```text
# Example of reprojecting a raster to match a reference grid
sd.sql("SELECT RS_ReprojectMatch(my_raster, ref_raster, 'Cubic')").to_arrow_table()
```

### Common Failure Modes
- **CRS Mismatch:** Providing one raster with a CRS and one without results in an error.
- **Unsupported Type:** Passing `Int64` or `UInt64` rasters will fail because they cannot be accurately resampled via GDAL's warp.
- **Memory Exhaustion:** Large rasters may exceed memory limits as the entire input is read into memory.

### Fix Code Hint
```python
# Cast unsupported types before reprojecting
sd.sql("""
    SELECT RS_ReprojectMatch(
        RS_Cast(my_raster, 'Float64'), 
        ref_raster, 
        'Bilinear'
    )
""").to_arrow_table()
```

## API Test: `sql:RS_Resample`

### Signature
```python
RS_Resample(raster: raster, reference_raster: raster, use_scale: boolean, algorithm: string) -> raster | RS_Resample(raster: raster, width_or_scale: double, height_or_scale: double, use_scale: boolean, algorithm: string) -> raster | RS_Resample(raster: raster, width_or_scale: double, height_or_scale: double, grid_x: double, grid_y: double, use_scale: boolean, algorithm: string) -> raster
```
_Source: source/docs/reference/sql/rs_resample.qmd:19_

### Goal
`RS_Resample` recomputes pixel values of a raster onto a new grid (defined by dimensions, pixel scale, or a reference raster) while preserving the original CRS, band count, and nodata values.

### Parameters
- `raster` (`raster`): The input raster to be resampled.
- `width_or_scale` (`double`): Output width in pixels (if `use_scale` is false) or target x pixel size (if `use_scale` is true).
- `height_or_scale` (`double`): Output height in pixels (if `use_scale` is false) or target y pixel size (if `use_scale` is true; negative for north-up).
- `grid_x` (`double`): X coordinate of the grid to which the output origin is snapped.
- `grid_y` (`double`): Y coordinate of the grid to which the output origin is snapped.
- `use_scale` (`boolean`): If false, treats dimensions as pixel counts (preserving extent); if true, treats dimensions as target pixel sizes.
- `algorithm` (`string`): Resampling method: `NearestNeighbor` (default), `Bilinear`, `Cubic`, `CubicSpline`, `Lanczos`, `Average`, or `Mode`.

### Input
- **Data:** A valid in-database `raster` object.
- **Preconditions:** The input raster must not be skewed or rotated if performing a scale change or origin snap. If using `reference_raster`, it must share the same CRS as the input raster.
- **Memory:** Extent-growing operations (scale changes and origin snaps) load the entire raster into memory.

### Output
Returns a new `raster` object with the modified pixel grid, maintaining the original band count, order, and per-band nodata values.

### Valid Call Patterns
```python
# Downsample example raster to 32x16
sd.sql("SELECT RS_Resample(RS_Example(), 32, 16, false, 'NearestNeighbor')").to_arrow_table()

# Upsample example raster to 128x64 using Bilinear
sd.sql("SELECT RS_Resample(RS_Example(), 128, 64, false, 'Bilinear')").to_arrow_table()
```

### LLM Instruction Prompt
- Always specify the `algorithm` string (e.g., 'NearestNeighbor').
- Ensure `use_scale` is set to `false` when providing explicit pixel dimensions.
- Remember that `RS_Resample` does not perform reprojection; use `RS_ReprojectMatch` if CRS transformation is required.

### Prompt Snippet
```text
RS_Resample(raster_obj, width, height, use_scale_bool, 'AlgorithmName')
```

### Common Failure Modes
- **Skewed/Rotated Rasters:** Attempting a scale change or origin snap on a skewed or rotated raster will result in an error.
- **Fractional Dimensions:** Providing fractional values for `width_or_scale` or `height_or_scale` when `use_scale` is `false` is an error.
- **CRS Mismatch:** Using a `reference_raster` with a different CRS than the input raster will cause an error.
- **Memory Limits:** Large rasters may exceed memory limits during extent-growing operations.

### Fix Code Hint
```python
# Ensure dimensions are integers when use_scale is False
sd.sql("SELECT RS_Resample(my_raster, 100.0, 100.0, false, 'Bilinear')") 
# If error occurs, check if raster is skewed:
# sd.sql("SELECT RS_GeoReference(my_raster)")
```

## API Test: `sql:RS_Rotation`

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
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. No specific pre-processing is required, but the raster must contain valid georeferencing metadata (skew parameters) to return a non-zero rotation.

### Output
Returns a `double` representing the rotation angle of the raster in radians.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
result = sd.sql("SELECT RS_Rotation(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_Rotation(raster_expression)` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster object; if using a file, load it first using `RS_FromPath`.

### Prompt Snippet
```text
SELECT RS_Rotation(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input Type:** Passing a `geometry` or `string` instead of a `raster` object will result in a SQL execution error.
- **Missing Metadata:** If the raster lacks georeferencing information or skew parameters, the function may return `0.0` or an unexpected value depending on the underlying raster implementation.
- **Null Input:** Passing a `NULL` raster will result in a `NULL` output.

### Fix Code Hint
```python
# Ensure the raster is properly loaded before calling RS_Rotation
sd.sql("SELECT RS_Rotation(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `sql:RS_SRID`

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
Retrieves the Spatial Reference System Identifier (SRID) integer associated with a given raster object to facilitate coordinate system identification and comparison.

### Parameters
- `arg1` (`raster`): The input raster object for which the SRID metadata is to be queried.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must have a defined Coordinate Reference System (CRS). If the raster was created without a CRS, the behavior of the SRID return value is dependent on the underlying raster metadata.

### Output
Returns `integer` — The SRID representing the coordinate reference system of the raster.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_SRID(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_SRID(raster_expression)` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster object (e.g., from `RS_FromPath` or `RS_Example`).

### Prompt Snippet
```text
SELECT RS_SRID(raster_column) FROM table_name;
```

### Common Failure Modes
- **Null/Missing CRS:** If the raster does not have an assigned CRS, the function may return a default value or null depending on the internal implementation.
- **Invalid Input:** Passing a non-raster type (e.g., `geometry` or `string`) will result in a SQL execution error.
- **Raster Availability:** Ensure the raster is materialized or loaded into the session before calling the function.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
sd.sql("SELECT RS_SRID(my_raster_col) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_ScaleX`

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
Returns the horizontal pixel size (scale X) of a raster object, representing the width of a single pixel in the raster's coordinate system.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the horizontal pixel dimension.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be properly initialized (e.g., via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`).

### Output
Returns `double` — The width of a pixel in the units of the raster's coordinate reference system.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_ScaleX(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_ScaleX(raster_expression)` in SQL queries.
- Ensure the input is a valid `raster` type; if the input is a file path, wrap it in a raster-loading function first.

### Prompt Snippet
```text
SELECT RS_ScaleX(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input Type:** Passing a `geometry` or `string` instead of a `raster` object will result in a SQL execution error.
- **Null Raster:** If the input raster is `NULL`, the function will return `NULL`.
- **Uninitialized Raster:** Attempting to call this on a raster that failed to load (e.g., invalid file path in `RS_FromPath`) will cause an error.

### Fix Code Hint
```python
# Ensure the raster is valid before calling RS_ScaleX
sd.sql("SELECT RS_ScaleX(raster) FROM (SELECT RS_FromPath('path/to/raster.tif') as raster) WHERE raster IS NOT NULL")
```

## API Test: `sql:RS_ScaleY`

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
Retrieves the vertical dimension (pixel height) of a raster object, representing the scale factor along the Y-axis.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the Y-axis scale metadata.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. No specific preconditions are required other than the input being a valid, non-null raster.

### Output
Returns a `double` representing the pixel height (scale Y) of the raster.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_ScaleY(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_ScaleY(raster_expression)` to extract the Y-axis scale.
- Ensure the input is a valid raster object; if the input is a path or binary, ensure it has been loaded into a raster type first.

### Prompt Snippet
```text
SELECT RS_ScaleY(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-raster type (e.g., `geometry` or `string`) to the function will result in a SQL execution error.
- Attempting to call the function on a null raster object will return `NULL`.

### Fix Code Hint
```python
# Ensure the input is a valid raster object before calling RS_ScaleY
sd.sql("SELECT RS_ScaleY(my_raster_col) FROM my_table WHERE my_raster_col IS NOT NULL").to_arrow_table()
```

## API Test: `sql:RS_SetBandNoDataValue`

### Signature
```python
RS_SetBandNoDataValue(arg1: raster, nodata: float64) -> raster | RS_SetBandNoDataValue(arg1: raster, band: int, nodata: float64) -> raster
```
_Source: source/docs/reference/sql/rs_setbandnodatavalue.qmd:19_

### Goal
Sets the nodata value metadata for a specific band of a raster, packing the provided value into the band's native data type without modifying actual pixel values.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`int`): The 1-based index of the band to update.
- `nodata` (`float64`): The nodata value to assign to the band, or `NULL` to clear the existing nodata value.

### Input
- **Data:** A valid `raster` object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Preconditions:** For single-band rasters, the two-argument form is sufficient. For multi-band rasters, the three-argument form must be used to specify the target band index.
- **Metadata-only:** This operation is metadata-only; raster cell values are not modified.

### Output
Returns a `raster` object with the updated nodata metadata. A null input raster or invalid band index yields a null raster.

### Valid Call Patterns
```python
# Set nodata for a specific band (1-based index)
sd.sql("SELECT RS_SetBandNoDataValue(RS_Example(), 1, 0)").to_arrow_table()

# Clear nodata for a specific band
sd.sql("SELECT RS_SetBandNoDataValue(RS_Example(), 1, CAST(NULL AS DOUBLE))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use the 1-based indexing for the `band` parameter.
- If the raster is multi-band, you must use the three-argument signature.
- To clear a nodata value, pass `CAST(NULL AS DOUBLE)` as the `nodata` argument.
- Remember that this is a metadata-only operation; do not expect changes to the underlying pixel values.

### Prompt Snippet
```text
RS_SetBandNoDataValue(raster_obj, band_index, nodata_value)
```

### Common Failure Modes
- **Invalid Band Index:** Providing a band index outside the range of available bands (1 to `RS_NumBands(raster)`) will result in a null raster or error.
- **Multi-band Ambiguity:** Attempting to use the two-argument form on a multi-band raster may lead to unexpected behavior or errors.
- **Type Mismatch:** Passing a non-numeric value (other than `NULL`) for the `nodata` parameter will cause a type error.

### Fix Code Hint
```python
# Ensure the band index is valid before calling
# Use CAST(NULL AS DOUBLE) to clear existing nodata
query = "SELECT RS_SetBandNoDataValue(my_raster, 1, CAST(NULL AS DOUBLE))"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:RS_SetCRS`

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
Updates the Coordinate Reference System (CRS) metadata of a raster object without modifying its underlying pixel values or geotransform.

### Parameters
- `arg1` (`raster`): The input raster object to be updated.
- `target_crs` (`string`): The CRS identifier to assign to the raster (e.g., 'EPSG:4326').

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be loaded into the SedonaDB session. This operation is metadata-only; it does not perform coordinate reprojection.

### Output
Returns `raster` — A new raster object with the updated CRS metadata.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT RS_SetCRS(RS_Example(), 'EPSG:4326')").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_SetCRS` only when you need to define or override the CRS metadata of a raster.
- Do not use this function for coordinate transformation; it does not reproject data.
- Ensure the `target_crs` string follows standard authority code formats (e.g., 'EPSG:4326').

### Prompt Snippet
```text
RS_SetCRS(raster_object, 'EPSG:4326')
```

### Common Failure Modes
- **Invalid CRS String:** Providing a malformed or unsupported CRS string will result in an execution error.
- **Missing Raster:** Attempting to call the function on a null or non-raster input.
- **Misunderstanding Scope:** Expecting the function to reproject coordinates; it only updates the metadata header.

### Fix Code Hint
```python
# Ensure the CRS string is valid and the input is a valid raster
result = sd.sql("SELECT RS_SetCRS(my_raster, 'EPSG:3857')").to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:RS_SetGeoReference`

### Signature
```python
RS_SetGeoReference(arg1: raster, georeference: string) -> raster | RS_SetGeoReference(arg1: raster, georeference: string, format: string) -> raster | RS_SetGeoReference(arg1: raster, upperleftx: numeric, upperlefty: numeric, scalex: numeric, scaley: numeric, skewx: numeric, skewy: numeric) -> raster
```
_Source: source/docs/reference/sql/rs_setgeoreference.qmd:19_

### Goal
`sql:RS_SetGeoReference` updates the affine geotransform metadata of a raster object without performing any pixel resampling.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `georeference` (`string`): A string containing six whitespace-separated numbers in the order: `scalex`, `skewy`, `skewx`, `scaley`, `upperleftx`, `upperlefty`.
- `format` (`string`): Optional coordinate convention; 'GDAL' (default/alias 'PIXEL') for pixel corner, 'ESRI' (alias 'NODE') for pixel center.
- `upperleftx` (`numeric`): X-coordinate of the upper-left pixel's corner.
- `upperlefty` (`numeric`): Y-coordinate of the upper-left pixel's corner.
- `scalex` (`numeric`): The width of a pixel.
- `scaley` (`numeric`): The height of a pixel (use a negative value for a north-up raster).
- `skewx` (`numeric`): The row rotation term.
- `skewy` (`numeric`): The column rotation term.

### Input
The function accepts a `raster` object. When using the string-based georeference, the input must be a string of six whitespace-separated numbers. When using numeric arguments, the input must be six numeric values representing the affine transform components. This is a metadata-only operation; no resampling of cell values occurs.

### Output
Returns a `raster` object with the updated affine geotransform metadata.

### Valid Call Patterns
```python
# Using string georeference (GDAL default)
sd.sql("SELECT RS_SetGeoReference(RS_Example(), '2 0 0 -3 100 200')").to_arrow_table()

# Using string georeference with ESRI format
sd.sql("SELECT RS_SetGeoReference(RS_Example(), '2 0 0 -3 101 198.5', 'ESRI')").to_arrow_table()

# Using numeric arguments (GDAL/corner coordinates)
sd.sql("SELECT RS_SetGeoReference(RS_Example(), 100, 200, 2, -3, 0, 0)").to_arrow_table()
```

### LLM Instruction Prompt
When calling `sql:RS_SetGeoReference`, ensure the georeference string or numeric arguments follow the specific order defined in the signature. If using the string format, verify the order is `scalex, skewy, skewx, scaley, upperleftx, upperlefty`. If using numeric arguments, ensure the order is `upperleftx, upperlefty, scalex, scaley, skewx, skewy`.

### Prompt Snippet
```text
Use RS_SetGeoReference(raster, georeference_string) or RS_SetGeoReference(raster, ulx, uly, scalex, scaley, skewx, skewy) to update raster metadata.
```

### Common Failure Modes
- Providing an incorrect number of values in the georeference string.
- Confusing the order of parameters between the string-based and numeric-based overloads.
- Expecting pixel values to change (this function is metadata-only and does not perform resampling).
- Providing a positive `scaley` value when a north-up raster is intended (typically requires a negative value).

### Fix Code Hint
```python
# Ensure the georeference string matches the required order: scalex, skewy, skewx, scaley, upperleftx, upperlefty
# Example: '2 0 0 -3 100 200'
result = sd.sql("SELECT RS_SetGeoReference(my_raster, '2 0 0 -3 100 200')").to_arrow_table()
```

## API Test: `sql:RS_SetSRID`

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
Updates the spatial reference identifier (SRID) metadata of a raster object without modifying its underlying pixel values or geotransform parameters.

### Parameters
- `arg1` (`raster`): The input raster object to be updated.
- `srid` (`integer`): The integer identifier representing the target coordinate reference system (e.g., 4326 for WGS 84).

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be loaded into the session (e.g., via `RS_FromPath` or `RS_MakeEmptyRaster`). This operation is metadata-only and does not perform coordinate reprojection.

### Output
Returns `raster` — A new raster object with the updated SRID metadata.

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT RS_SetSRID(RS_Example(), 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_SetSRID(raster, srid)` to update the CRS metadata of a raster.
- Ensure the `srid` is provided as an integer.
- Remember that this function does not reproject data; it only updates the SRID tag.

### Prompt Snippet
```text
RS_SetSRID(raster_column, 4326)
```

### Common Failure Modes
- **Invalid SRID:** Providing an unsupported or non-existent integer SRID may result in runtime errors during subsequent spatial operations.
- **Metadata-Only Assumption:** Users may incorrectly assume this function performs coordinate transformation; it does not. If actual coordinate transformation is required, use appropriate reprojection functions.

### Fix Code Hint
```python
# Ensure the raster is valid before setting SRID
sd.sql("SELECT RS_SetSRID(my_raster_col, 3857) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_SetValue`

### Signature
```python
RS_SetValue(arg1: raster, col_x: integer, row_y: integer, new_value: double) -> raster | RS_SetValue(arg1: raster, band: integer, col_x: integer, row_y: integer, new_value: double) -> raster
```
_Source: source/docs/reference/sql/rs_setvalue.qmd:19_

### Goal
Updates the value of a specific pixel within a designated band of a 2-D raster.

### Parameters
- `arg1` (`raster`): The input raster object to be modified.
- `band` (`integer`): The 1-based index of the band to modify. If omitted, the raster must contain exactly one band.
- `col_x` (`integer`): The 1-based column index of the pixel to update.
- `row_y` (`integer`): The 1-based row index of the pixel to update.
- `new_value` (`double`): The new value to store in the pixel.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must be 2-D. Pixel coordinates `(col_x, row_y)` must be within the grid dimensions. If a `band` is specified, it must exist within the raster.

### Output
Returns a new `raster` object with the specified pixel updated to `new_value`.

### Valid Call Patterns
```python
# Example using band index
sd.sql("SELECT RS_SummaryStats(RS_SetValue(RS_Example(), 1, 2, 2, 42), 'max', 1)").to_arrow_table()

# Example without band index (single-band raster)
sd.sql("SELECT RS_SetValue(raster_col, 1, 1, 10.5) FROM my_rasters").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for `col_x`, `row_y`, and `band`.
- Ensure the raster is 2-D before calling.
- Remember that `new_value` is a `double`; integer bands will truncate this value toward zero, and float32 bands will round to the nearest float32.

### Prompt Snippet
```text
RS_SetValue(raster, [band, ]col_x, row_y, new_value)
```

### Common Failure Modes
- **Out of Bounds:** Providing `col_x` or `row_y` values that fall outside the raster grid dimensions.
- **Invalid Band:** Specifying a `band` index that does not exist in the raster.
- **Unsupported Raster:** Attempting to use this function on non-2-D rasters.
- **Value Overflow:** Providing a `new_value` that exceeds the range of the target band's pixel type (e.g., values beyond ±2^53 for 64-bit integer bands).
- **Missing Band Index:** Omitting the `band` argument on a multi-band raster.

### Fix Code Hint
```python
# Ensure coordinates are within bounds and band exists
# Check raster dimensions using RS_Width and RS_Height if necessary
sd.sql("SELECT RS_SetValue(r, 1, 1, 100.0) FROM (SELECT RS_Example() as r) t").to_arrow_table()
```

## API Test: `sql:RS_Shape`

### Signature
```python
RS_Shape(arg1: raster) -> list<int64> | RS_Shape(arg1: raster, band: int) -> list<int64>
```
_Source: source/docs/reference/sql/rs_shape.qmd:19_

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

title: RS_Shape
description: Returns the shape (size of each dimension) of a raster band.
kernels:
  - returns: list<int64>
    args: [raster]
  - returns: list<int64>
    args:
    - raster
    - name: band
      type: int
      description: Band index (1-based). A null or out-of-range index returns null.
---

::: callout-warning
**Experimental.** This function is experimental; its behavior may change without notice.
:::

## Description

Returns the shape of a raster band as a list of dimension sizes. The entries
correspond to the dimension names returned by
[RS_DimNames](rs_dimnames.qmd).

For a standard 2D raster this returns `[height, width]`. For an N-dimensional
raster with a time dimension it might return `[12, 256, 256]` meaning 12 time
steps at 256x256 spatial resolution.

When the band index is omitted, all bands must agree on their shape; if
they disagree, an error is returned prompting the user to specify a band
index.

## Examples

```sql
SELECT RS_Shape(RS_Example());
```


### Goal
Returns the dimension sizes of a raster band as a list of integers, providing metadata on the spatial or temporal extent of the raster.

### Parameters
- `arg1` (`raster`): The input raster object to inspect.
- `band` (`int`): The 1-based index of the specific band to query. If omitted, the function assumes all bands share the same shape.

### Input
- **Data:** A valid `raster` object (e.g., created via `RS_FromPath` or `RS_Example`).
- **Preconditions:** If the raster contains multiple bands with differing shapes, the `band` parameter must be provided to avoid an error.

### Output
Returns `list<int64>` — A list of integers representing the size of each dimension (e.g., `[height, width]` for 2D rasters).

### Valid Call Patterns
```python
# Example 1: Implicit band selection
sd.sql("SELECT RS_Shape(RS_Example())").to_arrow_table()

# Example 2: Explicit band selection
sd.sql("SELECT RS_Shape(RS_Example(), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Shape(raster)` for single-band or uniform-shape rasters.
- Use `RS_Shape(raster, band_index)` when querying specific bands or when bands have inconsistent dimensions.
- Note that this function is experimental and may change.

### Prompt Snippet
```text
RS_Shape(raster_object)
RS_Shape(raster_object, band_index)
```

### Common Failure Modes
- **Band Mismatch:** If the band index is omitted and the raster bands have inconsistent shapes, the query will fail.
- **Invalid Index:** Providing a null or out-of-range band index returns `null`.
- **Experimental Status:** As an experimental feature, behavior may be unstable or subject to change in future versions.

### Fix Code Hint
```python
# If RS_Shape fails due to inconsistent band shapes, specify the band index:
sd.sql("SELECT RS_Shape(my_raster, 1) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_SkewX`

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
Retrieves the X-axis skew (rotation) component from the georeferencing transformation matrix of a raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the X skew metadata.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`. No specific preprocessing is required, but the raster must contain valid georeferencing metadata.

### Output
Returns a `double` representing the X skew parameter of the raster's affine transformation.

### Valid Call Patterns
```python
# Using the Python SQL connection
sd.sql("SELECT RS_SkewX(raster_col) FROM my_raster_table").to_arrow_table()

# Using the documented example pattern
sd.sql("SELECT RS_SkewX(RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_SkewX(raster_expression)` within a SQL query string passed to `sd.sql()`.
- Ensure the input is a valid raster object; if the input is a path or binary, wrap it in the appropriate constructor (e.g., `RS_FromPath`) before passing it to `RS_SkewX`.

### Prompt Snippet
```text
SELECT RS_SkewX(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-raster type (e.g., `geometry` or `string`) will result in a SQL execution error.
- Attempting to call the function on a null raster object will return `NULL`.
- Ensure the raster has been correctly initialized; uninitialized or malformed raster objects may cause the function to fail during metadata extraction.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
result = sd.sql("SELECT RS_SkewX(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
print(result)
# __CHECK__
```

## API Test: `sql:RS_SkewY`

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
Retrieves the Y-axis skew (rotation) component from the georeferencing transformation matrix of a given raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the Y skew metadata.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be properly initialized (e.g., via `RS_FromPath` or `RS_MakeEmptyRaster`). The function assumes the raster contains valid georeferencing metadata.

### Output
Returns `double` — The Y skew value representing the rotation component of the raster's affine transformation.

### Valid Call Patterns
```python
# Using the Python SQL connection to verify the Y skew of a raster
result = sd.sql("SELECT RS_SkewY(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_SkewY(raster_expression)` within a `SELECT` statement.
- Ensure the input is a valid `raster` type; if the input is a file path, wrap it in a raster-loading function like `RS_FromPath` first.

### Prompt Snippet
```text
SELECT RS_SkewY(<raster_column_or_function>)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a `geometry` or `string` instead of a `raster` object will result in a SQL execution error.
- **Null Raster:** If the input raster is `NULL`, the function will return `NULL`.
- **Missing Metadata:** If the raster object was created without georeferencing information, the skew value may default to 0.0 or be undefined depending on the raster source.

### Fix Code Hint
```python
# Ensure the raster is valid before querying metadata
sd.sql("SELECT RS_SkewY(raster_col) FROM (SELECT RS_FromPath('/path/to/raster.tif') AS raster_col) t").to_arrow_table()
```

## API Test: `sql:RS_Slice`

### Signature
```python
RS_Slice(arg1: raster, dim_name: utf8, index: int) -> raster
```
_Source: source/docs/reference/sql/rs_slice.qmd:19_

### Goal
Extracts a single slice along a specified non-spatial dimension from a raster, removing that dimension from the output while preserving spatial dimensions.

### Parameters
- `arg1` (`raster`): The input raster object containing one or more bands with non-spatial dimensions.
- `dim_name` (`utf8`): The name of the non-spatial dimension to slice (e.g., 'time').
- `index` (`int`): The zero-based index along the specified dimension to extract.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must contain at least one band with the specified `dim_name`. Spatial dimensions (`x_dim`, `y_dim`) cannot be sliced. If the raster is heterogeneous, bands lacking the named dimension are passed through unchanged.

### Output
Returns a `raster` object with the specified dimension removed from all bands that previously contained it.

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT RS_Slice(RS_BandToDim(RS_Example(), 'band'), 'band', 0)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Slice(raster, dim_name, index)` to reduce the dimensionality of a raster.
- Ensure the `dim_name` matches a dimension present in at least one band of the input raster.
- Remember that `index` is zero-based.
- Note that this function is experimental and may change.

### Prompt Snippet
```text
RS_Slice(raster_object, 'dimension_name', index_integer)
```

### Common Failure Modes
- **Dimension Not Found:** Errors if no band in the raster carries the specified `dim_name` (often due to a typo).
- **Index Out of Range:** Errors if the provided `index` is invalid for any band that contains the named dimension.
- **Spatial Slicing:** Attempting to slice `x_dim` or `y_dim` is not supported.
- **Experimental Status:** Behavior may change in future versions; verify compatibility with the current SedonaDB 0.5.0 environment.

### Fix Code Hint
```python
# Ensure the dimension exists before slicing
# If slicing fails, verify the dimension name using metadata inspection
# Example:
sd.sql("SELECT RS_Slice(my_raster, 'time', 0)").to_arrow_table()
```

## API Test: `sql:RS_SliceRange`

### Signature
```python
RS_SliceRange(arg1: raster, dim_name: utf8, start: int, end: int) -> raster
```
_Source: source/docs/reference/sql/rs_slicerange.qmd:19_

### Goal
Narrows a specified non-spatial dimension of a raster to a half-open index range `[start, end)`, preserving the dimension in the output with a reduced size.

### Parameters
- `arg1` (`raster`): The input raster object containing the dimension to be sliced.
- `dim_name` (`utf8`): The name of the non-spatial dimension to narrow (e.g., 'time').
- `start` (`int`): The inclusive, zero-based start index of the range.
- `end` (`int`): The exclusive end index of the range.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must contain the dimension specified by `dim_name`. Spatial dimensions (`x_dim`, `y_dim`) cannot be narrowed. If the raster is heterogeneous, bands lacking the specified dimension are passed through unchanged.

### Output
Returns a `raster` with the specified dimension reduced to the size `end - start`.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT RS_SliceRange(RS_BandToDim(RS_Example(), 'band'), 'band', 0, 2)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_SliceRange` to subset non-spatial dimensions.
- Ensure `start < end` and that the indices are within the bounds of the dimension for all bands that carry it.
- If the dimension name is unknown, verify it exists in the raster metadata before calling.

### Prompt Snippet
```text
RS_SliceRange(raster_obj, 'dimension_name', start_idx, end_idx)
```

### Common Failure Modes
- **Dimension Not Found:** Errors if no band in the raster carries the specified `dim_name` (usually indicates a typo).
- **Invalid Range:** Errors if `start >= end` or if the requested range is out of bounds for any band containing the dimension.
- **Spatial Dimension Attempt:** Errors if attempting to slice `x_dim` or `y_dim`.

### Fix Code Hint
```python
# Ensure indices are valid and dimension exists
# Example: Slice 'time' dimension from index 0 to 5
query = "SELECT RS_SliceRange(my_raster, 'time', 0, 5)"
result = sd.sql(query).to_arrow_table()
```

## API Test: `sql:RS_Stack`

### Signature
```python
RS_Stack(arg1: raster, rasters: raster) -> raster
```
_Source: source/docs/reference/sql/rs_stack.qmd:19_

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

title: RS_Stack
description: >
  Returns one raster holding the bands of any number of rasters, in argument
  order.
kernels:
  - returns: raster
    args:
    - raster
    - name: rasters
      type: raster
      description: >
        One or more further rasters (any number), on the same grid as the
        first: the same width, height, geotransform and CRS.
---

::: callout-warning
**Experimental.** This function is experimental; its behavior may change without notice.
:::

## Description

Sedona Spark 1.9 calls this function `RS_Union`.

The result holds every band of the first raster, then every band of the second,
and so on. Each band keeps its own pixel type, nodata value and name, so rasters
of different pixel types can be stacked. Every raster must be on the first
raster's grid (the same width, height, geotransform and CRS); a raster on any
other grid is an error. A band's spatial dimensions take the first raster's
names, matched by role, so a `lat`/`lon` raster stacked onto a `y`/`x` raster
gives `y`/`x` bands; a band with another dimension already named `y` or `x`
is an error. A NULL raster gives a NULL result.

No pixel is read: bands are carried over as they are, so stacking rasters that
are still out-of-database does not load them.

## Examples

```sql
SELECT RS_NumBands(RS_Stack(RS_Example(), RS_Example()));
```


### Goal
Combines multiple rasters into a single multi-band raster by stacking their bands in the order provided.

### Parameters
- `arg1` (`raster`): The primary raster that defines the target grid (width, height, geotransform, and CRS) and spatial dimension names.
- `rasters` (`raster`): One or more additional rasters to be stacked onto the primary raster.

### Input
- **Data:** Raster objects (in-database or out-of-database).
- **Preconditions:** All input rasters must share the exact same grid definition (width, height, geotransform, and CRS) as `arg1`.
- **Constraints:** A NULL input results in a NULL output.

### Output
Returns `raster` — A new raster object containing all bands from the input rasters, preserving individual band metadata (pixel type, nodata value, and name).

### Valid Call Patterns
```python
# Example using the documented SQL pattern
query = "SELECT RS_Stack(RS_Example(), RS_Example())"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure all input rasters are aligned to the same grid as the first argument.
- Note that this function is experimental and may change.
- Remember that `RS_Stack` does not trigger pixel loading for out-of-database rasters.

### Prompt Snippet
```text
RS_Stack(raster_a, raster_b)
```

### Common Failure Modes
- **Grid Mismatch:** Providing rasters with different dimensions, geotransforms, or CRS than the first raster will cause an error.
- **Dimension Conflict:** Attempting to stack a raster that introduces a dimension name already present in the primary raster (e.g., conflicting 'x' or 'y' roles) will cause an error.

### Fix Code Hint
```python
# Ensure rasters are aligned using RS_ReprojectMatch before stacking
aligned_raster = sd.sql("SELECT RS_ReprojectMatch(raster_to_align, reference_raster)").to_arrow_table()
```

## API Test: `sql:RS_SummaryStats`

### Signature
```python
RS_SummaryStats(arg1: raster, stat_type: string) -> double | RS_SummaryStats(arg1: raster, stat_type: string, band: integer) -> double | RS_SummaryStats(arg1: raster, stat_type: string, band: integer, exclude_nodata: boolean) -> double
```
_Source: source/docs/reference/sql/rs_summarystats.qmd:19_

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

title: RS_SummaryStats
description: >
  Returns one summary statistic of a band's pixel values as a double.
kernels:
  - returns: double
    args:
    - raster
    - name: stat_type
      type: string
      description: >
        One of `count`, `sum`, `mean`, `stddev`, `min` or `max`
        (case-insensitive). `stddev` is the population standard deviation.
  - returns: double
    args:
    - raster
    - name: stat_type
      type: string
    - name: band
      type: integer
      description: Band index (1-based). Defaults to 1 if not specified.
  - returns: double
    args:
    - raster
    - name: stat_type
      type: string
    - name: band
      type: integer
    - name: exclude_nodata
      type: boolean
      description: >
        Whether to leave pixels holding the band's nodata value out of the
        statistic. Defaults to true.
---

## Description

Over no pixels (for example, a band that is entirely nodata), `count` and
`sum` are 0 and the other statistics are NaN; over one pixel, `stddev` is 0.
A NaN pixel is left out as nodata when the band's nodata value is NaN.
Otherwise it is data: it makes `sum` and `mean` NaN, and `stddev` too unless
it is the only pixel, while `min` and `max` skip it unless every pixel is NaN.

An unknown `stat_type`, a band the raster does not have, or a 64-bit integer
pixel beyond 2^53 in magnitude (which a double cannot hold exactly) is an
error. Only 2-D rasters are supported.

## Examples

```sql
SELECT RS_SummaryStats(RS_Example(), 'mean');
```

```sql
SELECT RS_SummaryStats(RS_Example(), 'count', 1, false);
```


### Goal
Computes a single summary statistic (count, sum, mean, stddev, min, or max) for a specified band of a 2-D raster.

### Parameters
- `arg1` (`raster`): The input 2-D raster object.
- `stat_type` (`string`): The statistic to compute: `count`, `sum`, `mean`, `stddev` (population), `min`, or `max` (case-insensitive).
- `band` (`integer`): The 1-based index of the band to analyze. Defaults to 1.
- `exclude_nodata` (`boolean`): Whether to exclude pixels matching the band's nodata value. Defaults to `true`.

### Input
Requires a valid 2-D raster object. The raster must contain the specified band index.

### Output
Returns a `double` representing the calculated statistic. If the band is entirely nodata, `count` and `sum` return 0, while others return `NaN`.

### Valid Call Patterns
```python
# Example 1: Basic mean calculation
sd.sql("SELECT RS_SummaryStats(RS_Example(), 'mean')").to_arrow_table()

# Example 2: Count including nodata values
sd.sql("SELECT RS_SummaryStats(RS_Example(), 'count', 1, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use the `RS_SummaryStats(raster, stat_type, ...)` signature.
- Ensure `stat_type` is one of the six supported strings.
- Remember that `band` is 1-based.
- If `exclude_nodata` is omitted, it defaults to `true`.

### Prompt Snippet
```text
RS_SummaryStats(raster_obj, 'mean', 1, true)
```

### Common Failure Modes
- Providing an invalid `stat_type` string.
- Requesting a `band` index that does not exist in the raster.
- Attempting to process a raster with 64-bit integer pixels exceeding 2^53 in magnitude.
- Attempting to process non-2-D rasters.

### Fix Code Hint
```python
# Ensure band index is valid and stat_type is correctly spelled
result = sd.sql("SELECT RS_SummaryStats(my_raster, 'stddev', 1, true)").to_arrow_table()
```

## API Test: `sql:RS_SummaryStatsAll`

### Signature
```python
RS_SummaryStatsAll(arg1: raster) -> struct | RS_SummaryStatsAll(arg1: raster, band: integer) -> struct | RS_SummaryStatsAll(arg1: raster, band: integer, exclude_nodata: boolean) -> struct
```
_Source: source/docs/reference/sql/rs_summarystatsall.qmd:19_

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

title: RS_SummaryStatsAll
description: >
  Returns every summary statistic of a band's pixel values as a struct of
  six doubles: count, sum, mean, stddev, min and max.
kernels:
  - returns: struct
    args: [raster]
  - returns: struct
    args:
    - raster
    - name: band
      type: integer
      description: Band index (1-based). Defaults to 1 if not specified.
  - returns: struct
    args:
    - raster
    - name: band
      type: integer
    - name: exclude_nodata
      type: boolean
      description: >
        Whether to leave pixels holding the band's nodata value out of the
        statistics. Defaults to true.
---

## Description

The fields are the statistics `RS_SummaryStats` returns one at a time, with
the same values: `stddev` is the population standard deviation, and over no
pixels (for example, a band that is entirely nodata) `count` and `sum` are 0
and the other fields are NaN. Computing them together scans the band three
times, where six `RS_SummaryStats` calls would scan it more often.

A band the raster does not have, or a 64-bit integer pixel beyond 2^53 in
magnitude (which a double cannot hold exactly), is an error. Only 2-D rasters
are supported.

## Examples

```sql
SELECT RS_SummaryStatsAll(RS_Example());
```

```sql
SELECT RS_SummaryStatsAll(RS_Example(), 1, false);
```


### Goal
Computes all summary statistics (count, sum, mean, stddev, min, max) for a specified raster band in a single operation.

### Parameters
- `arg1` (`raster`): The input 2-D raster object.
- `band` (`integer`): The 1-based index of the band to analyze. Defaults to 1 if not specified.
- `exclude_nodata` (`boolean`): Whether to exclude pixels matching the band's nodata value from the calculation. Defaults to true.

### Input
- **Data:** A 2-D raster object.
- **Preconditions:** The raster must contain the specified band index. 64-bit integer pixels must not exceed 2^53 in magnitude.

### Output
Returns a `struct` containing six `double` values: `count`, `sum`, `mean`, `stddev` (population standard deviation), `min`, and `max`.

### Valid Call Patterns
```python
# Example using default band and nodata exclusion
sd.sql("SELECT RS_SummaryStatsAll(RS_Example())").to_arrow_table()

# Example specifying band and including nodata
sd.sql("SELECT RS_SummaryStatsAll(RS_Example(), 1, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `RS_SummaryStatsAll` for efficient multi-statistic calculation to minimize band scans.
- Ensure the band index is 1-based.
- Remember that `stddev` is the population standard deviation.
- If a band is entirely nodata, `count` and `sum` will be 0, and other fields will be `NaN`.

### Prompt Snippet
```text
RS_SummaryStatsAll(raster, band, exclude_nodata)
```

### Common Failure Modes
- Providing a band index that does not exist in the raster.
- Attempting to process a raster with 64-bit integer pixels exceeding 2^53 in magnitude.
- Attempting to process non-2-D rasters.

### Fix Code Hint
```python
# Ensure band index is valid before calling
sd.sql("SELECT RS_SummaryStatsAll(raster_col, 1, true) FROM my_table").to_arrow_table()
```

## API Test: `sql:RS_Tile`

### Signature
```python
RS_Tile(arg1: raster, width: integer, height: integer, pad_with_no_data: boolean, no_data_value: double) -> list | RS_Tile(arg1: raster, band_indices: array, width: integer, height: integer, pad_with_no_data: boolean, no_data_value: double) -> list
```
_Source: source/docs/reference/sql/rs_tile.qmd:19_

### Goal
`RS_Tile` splits an input raster into a grid of fixed-size tiles, returning a list of structs containing the grid position and the tile raster.

### Parameters
- `arg1` (`raster`): The input raster object to be tiled.
- `band_indices` (`array`): Optional `Array[Int]` of 1-based band indices to include in each tile, in the specified order.
- `width` (`integer`): The desired width of each tile in pixels (must be >= 1).
- `height` (`integer`): The desired height of each tile in pixels (must be >= 1).
- `pad_with_no_data` (`boolean`): Optional (default false). If true, pads partial edge tiles to the full tile size using a nodata value.
- `no_data_value` (`double`): Optional. The value used for padding pixels if `pad_with_no_data` is true.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be loaded into the session. If using `band_indices`, ensure indices are 1-based and valid for the input raster.
- **Memory:** Note that `RS_Tile` materializes a full copy of the raster in memory; very small tile sizes (e.g., 1x1) will generate a large number of tiles and may impact memory usage.

### Output
Returns `list` — A list of structs, where each struct contains:
- `x`: The 0-based grid column index.
- `y`: The 0-based grid row index.
- `tile`: The resulting tile raster with a geotransform shifted to the tile's origin.

### Valid Call Patterns
```python
# Tile every band into 32x16 tiles
sd.sql("SELECT UNNEST(RS_Tile(RS_Example(), 32, 16)) AS tile").to_arrow_table()

# Tile with padding
sd.sql("SELECT UNNEST(RS_Tile(RS_Example(), 40, 20, true, 0.0)) AS tile").to_arrow_table()

# Tile specific bands
sd.sql("SELECT UNNEST(RS_Tile(RS_Example(), make_array(1, 3), 32, 16)) AS tile").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `UNNEST()` on the result of `RS_Tile` if you need to access individual tile rows in a SQL query.
- Ensure `width` and `height` are positive integers.
- When using `pad_with_no_data = true`, provide a `no_data_value` if the raster band does not have a defined nodata value.

### Prompt Snippet
```text
Use RS_Tile(raster, width, height) or RS_Tile(raster, band_indices, width, height) to split rasters. Wrap in UNNEST() to expand the list of tiles into rows.
```

### Common Failure Modes
- **Invalid Dimensions:** Providing `width` or `height` < 1 will cause an error.
- **Memory Exhaustion:** Using extremely small tile sizes on large rasters can exceed memory limits due to the full materialization of the raster copy.
- **Type Mismatch:** Providing a `no_data_value` that does not fit the band's data type will result in an error.

### Fix Code Hint
```python
# Ensure valid dimensions and handle potential memory issues by choosing appropriate tile sizes
query = "SELECT UNNEST(RS_Tile(my_raster, 64, 64)) AS tile FROM my_table"
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness: RS_Tile executed successfully")
```

## API Test: `sql:RS_UpperLeftX`

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
Returns the X-coordinate of the upper-left corner of the provided raster object, facilitating spatial metadata inspection.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the georeferenced upper-left X coordinate.

### Input
- **Data:** A valid SedonaDB `raster` object.
- **Preconditions:** The raster must be properly georeferenced. If the raster was created via `RS_MakeEmptyRaster`, ensure it has been initialized with valid geotransform parameters.

### Output
Returns `double` — The X-coordinate value corresponding to the upper-left corner of the raster in the coordinate system defined by the raster's metadata.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_UpperLeftX(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_UpperLeftX(raster_expression)` as a standalone SQL function call.
- Ensure the input is a valid `raster` type; if the input is a file path, wrap it in `RS_FromPath(...)` first.

### Prompt Snippet
```text
SELECT RS_UpperLeftX(raster_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input Type:** Passing a `geometry` or `string` instead of a `raster` object will result in a type mismatch error.
- **Uninitialized Raster:** Calling this on a raster that lacks georeferencing metadata may return unexpected results or null values.
- **Null Input:** Passing a `NULL` raster will return `NULL`.

### Fix Code Hint
```python
# Ensure the raster is loaded correctly before inspection
sd.sql("SELECT RS_UpperLeftX(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `sql:RS_UpperLeftY`

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
Returns the Y-coordinate of the upper-left corner of the provided raster object.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the upper-left Y coordinate.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_FromGDALRaster`.

### Output
Returns a `double` representing the Y-coordinate value in the raster's coordinate reference system.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_UpperLeftY(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `RS_UpperLeftY(raster_expression)` to retrieve the Y-coordinate of the upper-left corner.
- Ensure the input is a valid raster object; if the raster is derived from a file, ensure the file path is accessible to the SedonaDB engine.

### Prompt Snippet
```text
SELECT RS_UpperLeftY(raster_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-raster type (e.g., `geometry` or `string`) to the function will result in a type mismatch error.
- If the raster object is null or improperly initialized (e.g., from a failed `RS_FromPath` call), the function may return null or throw an execution error.

### Fix Code Hint
```python
# Ensure the raster is valid before calling the accessor
sd.sql("SELECT RS_UpperLeftY(raster) FROM rasters WHERE raster IS NOT NULL").to_arrow_table()
```

## API Test: `sql:RS_Value`

### Signature
```python
RS_Value(arg1: raster, point: geometry) -> float64 | RS_Value(arg1: raster, point: geometry, band: int) -> float64 | RS_Value(arg1: raster, colX: int, rowY: int, band: int) -> float64
```
_Source: source/docs/reference/sql/rs_value.qmd:19_

### Goal
Extracts the pixel value from a 2-D raster at a specific location or grid coordinate without performing interpolation.

### Parameters
- `arg1` (`raster`): The source raster object to sample.
- `colX` (`int`): The 1-based column index of the pixel.
- `rowY` (`int`): The 1-based row index of the pixel.
- `band` (`int`): The 1-based band index to sample.

### Input
- **Raster:** A 2-D raster object (e.g., from `RS_FromPath` or `RS_Example`).
- **Geometry:** A point geometry (e.g., `ST_Point`). If both the raster and point have a CRS, the point is reprojected to the raster's CRS.
- **Preconditions:** Band indices are 1-based. For multiband rasters, the `band` parameter is mandatory. Only 2-D rasters are supported.

### Output
Returns `float64` representing the pixel value. Returns `NULL` if the location is outside the raster bounds or if the pixel value matches the band's nodata value.

### Valid Call Patterns
```python
# Sampling by point geometry
sd.sql("SELECT RS_Value(RS_Example(), ST_Point(74.58, 110.57, 'OGC:CRS84'), 1)").to_arrow_table()

# Sampling by grid coordinate
sd.sql("SELECT RS_Value(RS_Example(), 1, 1, 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for `band`, `colX`, and `rowY`.
- If the raster is multiband, you must explicitly provide the `band` argument.
- Do not assume interpolation; this function performs nearest-pixel sampling.
- Ensure the point geometry is provided in a compatible CRS if the raster has one.

### Prompt Snippet
```text
Use RS_Value(raster, colX, rowY, band) for grid-based access or RS_Value(raster, point, band) for spatial access.
```

### Common Failure Modes
- **Ambiguous Band:** Calling `RS_Value` on a multiband raster without specifying the `band` argument will raise an error.
- **Out of Bounds:** Providing grid coordinates of `0` or coordinates outside the raster extent returns `NULL`.
- **CRS Mismatch:** If the point and raster have conflicting or missing CRS information, the operation may fail or produce unexpected results.
- **Experimental Status:** As an experimental feature, behavior may change; ensure the environment is updated to the latest SedonaDB 0.5.0.

### Fix Code Hint
```python
# Ensure band is specified for multiband rasters
sd.sql("SELECT RS_Value(my_raster, 1, 1, 1) FROM my_table")

# Verify point CRS matches raster CRS if necessary
sd.sql("SELECT RS_Value(my_raster, ST_Transform(my_point, 'EPSG:4326'), 1) FROM my_table")
```

## API Test: `sql:RS_Values`

### Signature
```python
RS_Values(arg1: raster, points: geometry) -> list | RS_Values(arg1: raster, points: geometry, band: int) -> list
```
_Source: source/docs/reference/sql/rs_values.qmd:19_

### Goal
Extracts pixel values from a 2-D raster at the locations of specified points, returning them as a list of doubles without performing any resampling.

### Parameters
- `arg1` (`raster`): The source 2-D raster object to be sampled.
- `points` (`geometry`): A `MultiPoint` or `Point` geometry defining the locations to sample. If both the raster and geometry have a CRS, the points are reprojected to the raster's CRS.
- `band` (`int`): The 1-based index of the raster band to sample. Defaults to 1 for single-band rasters; required for multiband rasters.

### Input
- **Data:** A valid 2-D raster and a `geometry` (Point or MultiPoint).
- **Preconditions:** The raster must be 2-D. If the raster is multiband, the `band` parameter must be explicitly provided. If the geometry and raster have different CRS, they must be compatible for reprojection.

### Output
Returns `list` — A list of double-precision values corresponding to the pixel values at each point in the input geometry, maintaining the input order. Elements are `NULL` if the point is outside the raster bounds or if the pixel value matches the band's nodata value.

### Valid Call Patterns
```python
# Sampling a single-band raster at multiple points
sd.sql("SELECT RS_Values(RS_Example(), ST_GeomFromText('MULTIPOINT (74.58 110.57, 500 500)', 'OGC:CRS84'), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Values` to extract pixel data at specific coordinates.
- Always specify the `band` index if the raster is known to be multiband to avoid ambiguity errors.
- Ensure the input geometry is a `Point` or `MultiPoint`.
- Remember that this function performs nearest-pixel sampling (no resampling).

### Prompt Snippet
```text
RS_Values(raster_obj, geometry_obj, band_index)
```

### Common Failure Modes
- **Ambiguity Error:** Calling `RS_Values` without a `band` index on a multiband raster.
- **Dimension Error:** Attempting to use `RS_Values` on a non-2-D raster.
- **Null Results:** Receiving `NULL` values when points fall outside the raster extent or hit nodata pixels.
- **CRS Mismatch:** Failure if the raster and geometry have incompatible coordinate systems.

### Fix Code Hint
```python
# Ensure band is specified for multiband rasters
sd.sql("SELECT RS_Values(my_raster, my_points, 1)").to_arrow_table()

# Handle NULLs if points might be outside the raster
sd.sql("SELECT COALESCE(RS_Values(my_raster, my_points, 1), [])").to_arrow_table()
```

## API Test: `sql:RS_Width`

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
Returns the horizontal dimension (width) of a raster object measured in pixels.

### Parameters
- `arg1` (`raster`): The input raster object from which to extract the width metadata.

### Input
The input must be a valid SedonaDB `raster` object. Rasters can be created via `RS_FromPath`, `RS_MakeEmptyRaster`, or `RS_AsRaster`. No specific file format constraints exist beyond the raster being successfully loaded into the SedonaDB environment.

### Output
Returns `bigint` — The total number of pixels along the x-axis (width) of the raster.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT RS_Width(RS_Example())").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `RS_Width(raster_expression)` in SQL queries.
- Ensure the input is a valid raster object; if the input is a file path, wrap it in a loading function like `RS_FromPath`.
- The return type is a `bigint`, suitable for arithmetic or filtering in subsequent SQL clauses.

### Prompt Snippet
```text
Use RS_Width(raster) to retrieve the pixel width of a raster object.
```

### Common Failure Modes
- Passing a `geometry` or `geography` type instead of a `raster` will result in a type mismatch error.
- Calling the function on a null or uninitialized raster object may return null or trigger an execution error.
- Ensure the raster is properly loaded into the session; if using `RS_FromPath`, verify the file path is accessible to the SedonaDB engine.

### Fix Code Hint
```python
# Ensure the input is a valid raster object
sd.sql("SELECT RS_Width(RS_FromPath('/path/to/raster.tif'))").to_arrow_table()
```

## API Test: `sql:RS_Within`

### Signature
```python
RS_Within(arg1: raster, arg2: geometry) -> boolean | RS_Within(arg1: geometry, arg2: raster) -> boolean | RS_Within(arg1: raster, arg2: raster) -> boolean
```
_Source: source/docs/reference/sql/rs_within.qmd:19_

### Goal
Determines if the convex hull of the first spatial object (raster or geometry) is completely contained within the spatial extent of the second object (raster or geometry).

### Parameters
- `arg1` (`raster` or `geometry`): The primary spatial object whose convex hull is evaluated for containment.
- `arg2` (`raster` or `geometry`): The secondary spatial object defining the boundary for the containment check.

### Input
- **Data Types:** Accepts `raster` objects (e.g., from `RS_FromPath` or `RS_Example`) and `geometry` objects (e.g., from `ST_Point`, `ST_GeomFromText`).
- **Preconditions:** If both inputs have defined CRSes, the system attempts to transform the geometry or the second raster's extent into the CRS of the first argument. If transformation fails, it falls back to WGS 84. If either input lacks a CRS, the comparison is performed directly without transformation.

### Output
Returns `boolean` — `true` if the convex hull of `arg1` is entirely within the extent of `arg2`, otherwise `false`.

### Valid Call Patterns
```python
# Example 1: Geometry within Raster
sd.sql("SELECT RS_Within(ST_Point(0.5, 0.5, 4326), RS_Example())").to_arrow_table()

# Example 2: Raster within Raster
sd.sql("SELECT RS_Within(RS_Example(), RS_Example())").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_Within(A, B)` to check if the convex hull of `A` is inside `B`.
- Remember that `RS_Within(A, B)` is equivalent to `RS_Contains(B, A)`.
- Always ensure inputs are valid SedonaDB spatial objects.
- If CRSes differ, the system handles transformation automatically, but ensure at least one input has a valid CRS if coordinate alignment is required.

### Prompt Snippet
```text
RS_Within(arg1, arg2)
```

### Common Failure Modes
- **CRS Mismatch:** If only one argument has a CRS defined, the operation may fail or produce unexpected results.
- **Invalid Geometry:** Passing a malformed WKT string to `ST_GeomFromText` before passing it to `RS_Within` will cause an execution error.
- **Memory Limits:** Comparing very large rasters may hit memory constraints as the operation involves extent calculation and potential transformation.

### Fix Code Hint
```python
# Ensure both inputs are valid and handle CRS if necessary
sd.sql("""
    SELECT RS_Within(
        ST_GeomFromText('POINT(0 0)', 4326), 
        RS_FromPath('/path/to/raster.tif')
    )
""").to_arrow_table()
```

## API Test: `sql:RS_WorldToRasterCoord`

### Signature
```python
RS_WorldToRasterCoord(arg1: raster, x: double, y: double) -> geometry | RS_WorldToRasterCoord(arg1: raster, point: geometry) -> geometry
```
_Source: source/docs/reference/sql/rs_worldtorastercoord.qmd:19_

### Goal
Converts world coordinates (geospatial reference system) into 1-based pixel coordinates (raster grid space) represented as a point geometry.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform metadata required for coordinate conversion.
- `x` (`double`): The world coordinate X value (e.g., longitude or easting).
- `y` (`double`): The world coordinate Y value (e.g., latitude or northing).

### Input
- **Raster:** Must be a valid SedonaDB raster object.
- **Coordinates:** World coordinates are expected to be in the same spatial reference system as the raster.
- **Point:** If using the geometry overload, the point coordinates are used as-is in the raster's space; no reprojection is performed.

### Output
Returns `geometry` — A `POINT` geometry where the X and Y values correspond to the 1-based pixel column and row indices. Coordinates inside the upper-left pixel map to `(1, 1)`.

### Valid Call Patterns
```python
# Using coordinate primitives
sd.sql("SELECT RS_WorldToRasterCoord(RS_Example(), 10.0, 20.0)").to_arrow_table()

# Using a point geometry
sd.sql("SELECT RS_WorldToRasterCoord(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoord` to map world-space locations to pixel-space indices.
- If the input is a point geometry, ensure it is in the same coordinate space as the raster, as this function does not perform reprojection.
- Remember that the output is a 1-based coordinate system; values outside the raster extent will be extrapolated based on the geotransform.

### Prompt Snippet
```text
RS_WorldToRasterCoord(raster_obj, x_val, y_val)
RS_WorldToRasterCoord(raster_obj, point_geom)
```

### Common Failure Modes
- **Coordinate Mismatch:** Providing world coordinates in a different CRS than the raster will result in incorrect pixel mapping.
- **Extrapolation:** Coordinates outside the raster bounds will return 0 or negative values; ensure logic handles these as "out-of-bounds" rather than invalid errors.
- **Missing Raster:** Passing a null or invalid raster object will cause the operation to fail.

### Fix Code Hint
```python
# Ensure the point is in the correct CRS before conversion
sd.sql("""
    SELECT RS_WorldToRasterCoord(raster, ST_Transform(geom, RS_SRID(raster))) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `sql:RS_WorldToRasterCoordX`

### Signature
```python
RS_WorldToRasterCoordX(arg1: raster, x: double, y: double) -> integer | RS_WorldToRasterCoordX(arg1: raster, point: geometry) -> integer
```
_Source: source/docs/reference/sql/rs_worldtorastercoordx.qmd:19_

### Goal
Converts a given world coordinate into its corresponding 1-based column index (X coordinate) within the specified raster grid.

### Parameters
- `arg1` (`raster`): The source raster object to query for coordinate mapping.
- `x` (`double`): The world X coordinate to convert.
- `y` (`double`): The world Y coordinate to convert.
- `point` (`geometry`): An alternative input representing the world coordinate as a point geometry; coordinates are used as-is in the raster's space without reprojection.

### Input
- **Raster:** Must be a valid SedonaDB raster object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Coordinates:** World coordinates provided as `double` values or a `geometry` point.
- **Preconditions:** The operation relies on the raster's geotransform metadata. If using the `geometry` overload, ensure the point coordinates are already in the raster's coordinate space, as this function does not perform reprojection.

### Output
Returns `integer` — A 1-based pixel column index. Coordinates falling within the upper-left pixel map to `1`. Coordinates outside the raster extent are extrapolated based on the geotransform and may return `0` or negative values.

### Valid Call Patterns
```python
# Using coordinate primitives
sd.sql("SELECT RS_WorldToRasterCoordX(RS_Example(), 10.0, 20.0)").to_arrow_table()

# Using a geometry point
sd.sql("SELECT RS_WorldToRasterCoordX(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoordX` to map spatial locations to grid indices.
- If the input is a `geometry`, use the `(raster, geometry)` overload.
- If the input is raw coordinates, use the `(raster, double, double)` overload.
- Remember that the output is 1-based; a result of 1 indicates the first column.

### Prompt Snippet
```text
RS_WorldToRasterCoordX(raster_obj, x_val, y_val)
RS_WorldToRasterCoordX(raster_obj, point_geom)
```

### Common Failure Modes
- **Coordinate Mismatch:** Providing coordinates in a different CRS than the raster will result in incorrect indices, as this function does not reproject.
- **Invalid Raster:** Passing a null or improperly initialized raster object will cause the query to fail.
- **Extrapolation:** Users may be surprised by `0` or negative results; clarify that these are expected for points outside the raster bounds.

### Fix Code Hint
```python
# Ensure coordinates are in the raster's CRS before calling
sd.sql("SELECT RS_WorldToRasterCoordX(raster, ST_X(geom), ST_Y(geom)) FROM table")
```

## API Test: `sql:RS_WorldToRasterCoordY`

### Signature
```python
RS_WorldToRasterCoordY(arg1: raster, x: double, y: double) -> integer | RS_WorldToRasterCoordY(arg1: raster, point: geometry) -> integer
```
_Source: source/docs/reference/sql/rs_worldtorastercoordy.qmd:19_

### Goal
Converts a given world coordinate (either as X/Y doubles or a geometry point) into the corresponding 1-based raster Y-axis pixel coordinate.

### Parameters
- `arg1` (`raster`): The source raster object containing the geotransform information required for coordinate mapping.
- `x` (`double`): The world X coordinate to convert.
- `y` (`double`): The world Y coordinate to convert.

### Input
- **Raster:** Must be a valid SedonaDB raster object (e.g., loaded via `RS_FromPath` or `RS_Example`).
- **Coordinates:** World coordinates are interpreted in the raster's own coordinate space.
- **Geometry:** If using the `point` overload, the geometry must be a valid `geometry` type. Note that this form does not perform reprojection; coordinates are used as given.

### Output
Returns `integer` — A 1-based pixel coordinate along the Y-axis. Coordinates mapping to the upper-left pixel return `1`. Values outside the raster extent are extrapolated based on the geotransform and may return `0` or negative values.

### Valid Call Patterns
```python
# Using X and Y coordinates
sd.sql("SELECT RS_WorldToRasterCoordY(RS_Example(), 10.0, 20.0)").to_arrow_table()

# Using a geometry point
sd.sql("SELECT RS_WorldToRasterCoordY(RS_Example(), ST_Point(10.0, 20.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `RS_WorldToRasterCoordY` when you need to map spatial world coordinates to specific pixel indices for raster analysis.
- Always ensure the input raster is valid and that the world coordinates are provided in the same coordinate system as the raster, as no automatic reprojection occurs.
- Remember that the output is 1-based; a result of 1 corresponds to the first row of the raster.

### Prompt Snippet
```text
RS_WorldToRasterCoordY(raster_obj, x_double, y_double)
RS_WorldToRasterCoordY(raster_obj, geometry_point)
```

### Common Failure Modes
- **Coordinate Mismatch:** Providing coordinates in a different CRS than the raster will result in incorrect pixel mapping, as the function does not reproject.
- **Invalid Raster:** Passing a null or non-raster object as `arg1` will cause the query to fail.
- **Extrapolation:** Users should be aware that coordinates outside the raster bounds will return valid integer results (0 or negative) based on the geotransform, which may not be intended if the user expects only valid pixel indices.

### Fix Code Hint
```python
# Ensure the point is in the same CRS as the raster before calling
sd.sql("SELECT RS_WorldToRasterCoordY(raster, ST_Transform(point, RS_SRID(raster))) FROM table")
```

## API Test: `sql:RS_ZonalStats`

### Signature
```python
RS_ZonalStats(arg1: raster, roi: geometry, stat_type: string) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean, exclude_no_data: boolean) -> double | RS_ZonalStats(arg1: raster, roi: geometry, band: integer, stat_type: string, all_touched: boolean, exclude_no_data: boolean, lenient: boolean) -> double
```
_Source: source/docs/reference/sql/rs_zonalstats.qmd:19_

### Goal
Computes a single summary statistic (e.g., mean, count, min, max) for raster pixels within a specified region of interest (ROI) geometry.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `roi` (`geometry`): The region-of-interest geometry used to mask the raster.
- `band` (`integer`): The 1-based band index to compute statistics over.
- `stat_type` (`string`): The statistic to calculate: `count`, `sum`, `mean`, `median`, `mode`, `stddev`, `variance`, `min`, or `max`.
- `all_touched` (`boolean`): If true, includes every pixel the ROI touches; if false, only includes pixels whose center falls inside the ROI. Defaults to `false`.
- `exclude_no_data` (`boolean`): If true, skips pixels equal to the band's nodata value. Defaults to `true`.
- `lenient` (`boolean`): If true, returns `NULL` when the ROI does not intersect the raster; if false, raises an error. Defaults to `true`.

### Input
- **Raster:** Must be a valid SedonaDB raster object.
- **Geometry:** Standard OGC WKT/WKB geometry.
- **CRS Consistency:** If both the raster and geometry have a CRS, they must match or the geometry will be reprojected. It is an error if exactly one side has a CRS.
- **Band Indexing:** 1-based indexing is required for multiband rasters.

### Output
Returns `double` representing the calculated statistic. `count` is returned as a whole number; all other statistics are floating point. If the ROI overlaps the raster but selects no pixels, `count` is 0 and other statistics are `NULL`.

### Valid Call Patterns
```python
# Example 1: Basic mean calculation
sd.sql("SELECT RS_ZonalStats(RS_Example(), ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'), 1, 'mean')").to_arrow_table()

# Example 2: Count with explicit flags
sd.sql("SELECT RS_ZonalStats(RS_Example(), ST_GeomFromText('POLYGON ((60 90, 160 90, 160 190, 60 190, 60 90))', 'OGC:CRS84'), 1, 'count', false, false)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use the `RS_ZonalStats` function within a `sd.sql()` call.
- Ensure the `stat_type` string is one of the supported values: `count`, `sum`, `mean`, `median`, `mode`, `stddev`, `variance`, `min`, or `max`.
- Remember that `band` is required for multiband rasters.
- If the ROI might not intersect the raster, rely on the default `lenient=true` behavior or explicitly set it.

### Prompt Snippet
```text
SELECT RS_ZonalStats(raster_col, roi_geom, band_index, 'stat_name', all_touched, exclude_no_data, lenient)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry and raster where only one has a defined CRS will cause an error.
- **Missing Band Index:** Omitting the `band` argument on a multiband raster.
- **Invalid Statistic:** Using a string not in the supported list (e.g., 'average' is not supported, use 'mean').
- **Non-intersecting ROI:** If `lenient` is set to `false` and the ROI does not intersect the raster, the query will fail.

### Fix Code Hint
```python
# Ensure CRS consistency before calling
sd.sql("SELECT RS_ZonalStats(raster, ST_Transform(roi, RS_SRID(raster)), 1, 'mean') FROM table")
```

## API Test: `sql:RS_ZonalStatsAll`

### Signature
```python
RS_ZonalStatsAll(arg1: raster, roi: geometry) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean, exclude_no_data: boolean) -> struct | RS_ZonalStatsAll(arg1: raster, roi: geometry, band: integer, all_touched: boolean, exclude_no_data: boolean, lenient: boolean) -> struct
```
_Source: source/docs/reference/sql/rs_zonalstatsall.qmd:19_

### Goal
Computes a comprehensive set of summary statistics (count, sum, mean, median, mode, stddev, variance, min, max) for raster pixels within a specified geometry region of interest.

### Parameters
- `arg1` (`raster`): The input raster object to analyze.
- `roi` (`geometry`): The region-of-interest geometry used to mask the raster.
- `band` (`integer`): The 1-based band index to compute statistics over; required for multiband rasters.
- `all_touched` (`boolean`): If true, includes every pixel the ROI touches; if false (default), only includes pixels whose center falls within the ROI.
- `exclude_no_data` (`boolean`): If true (default), excludes pixels matching the band's nodata value.
- `lenient` (`boolean`): If true (default), returns NULL if the ROI does not intersect the raster; if false, raises an error.

### Input
- **Raster:** Must be a valid SedonaDB raster object.
- **Geometry:** Must be a valid OGC WKT/WKB geometry.
- **CRS Consistency:** Both the raster and the ROI must either share a CRS or both lack one; it is an error if exactly one side has a CRS.
- **Band Indexing:** Band indices are 1-based.

### Output
Returns a `struct` containing the fields: `count` (64-bit integer), `sum`, `mean`, `median`, `mode`, `stddev`, `variance`, `min`, and `max` (all floating point). If no pixels are selected, `count` is 0 and other fields are NULL.

### Valid Call Patterns
```python
# Example using the documented SQL pattern
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
- Always specify the `band` index for multiband rasters, as the band-less overload does not default to band 1.
- Ensure the `roi` geometry and the raster share the same CRS or both lack one before calling.
- Use `all_touched=True` if you need to include pixels partially covered by the ROI boundary.

### Prompt Snippet
```text
RS_ZonalStatsAll(raster_obj, roi_geom, band_index, all_touched=False, exclude_no_data=True, lenient=True)
```

### Common Failure Modes
- **CRS Mismatch:** Providing a geometry with a CRS when the raster has none (or vice versa) will cause an error.
- **Missing Band Index:** Omitting the `band` argument on a multiband raster will result in an error.
- **Non-intersecting ROI:** If `lenient` is set to `False` and the ROI does not intersect the raster, the query will fail instead of returning NULL.

### Fix Code Hint
```python
# Ensure CRS alignment before calling
sd.sql("SELECT RS_ZonalStatsAll(raster, ST_Transform(roi, RS_SRID(raster)), 1) FROM data").to_arrow_table()
```

## API Test: `sql:S2_CellIdFromPoint`

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
Computes the 64-bit S2 cell identifier at the finest resolution (level 30) for a given point geography to facilitate spatial indexing and optimized query performance.

### Parameters
- `arg1` (`geography`): A point geography object representing the location for which the S2 cell ID is to be calculated.

### Input
- **Data Type:** Must be a `geography` type.
- **Preconditions:** The input must be a valid point geography. If the input is an empty point geometry, the function returns `NULL`.
- **Environment:** Requires the `s2geography` bounder to be registered on the session.

### Output
Returns `int64` — A 64-bit integer representing the unique S2 cell ID at level 30.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT S2_CellIdFromPoint(ST_GeogPoint(-64, 45))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always ensure the input is a `geography` type. If the input is a `geometry` type, it must be converted to `geography` first (e.g., using `ST_ToGeography`).
- Use this function specifically for spatial indexing tasks where high-resolution (level 30) cell IDs are required.

### Prompt Snippet
```text
Use S2_CellIdFromPoint(geography_column) to generate a 64-bit S2 cell ID for spatial indexing.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` type instead of `geography` will cause an execution error.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered on the current session.
- **Empty Input:** Providing an empty point geometry results in a `NULL` return value rather than an integer.

### Fix Code Hint
```python
# Ensure input is geography and handle potential NULLs
query = """
SELECT S2_CellIdFromPoint(ST_ToGeography(geom)) as s2_id 
FROM my_table 
WHERE geom IS NOT NULL
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:S2_CoveringCellIds`

### Signature
```python
S2_CoveringCellIds(arg1: geography) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64, max_level: int64) -> array | S2_CoveringCellIds(arg1: geography, min_level: int64, max_level: int64, max_cells: int64) -> array
```
_Source: source/docs/reference/sql/s2_coveringcellids.qmd:19_

### Goal
Generates an array of 64-bit S2 cell IDs that provide a minimal-overlap covering for a given `geography` object to facilitate spatial indexing and efficient query performance.

### Parameters
- `arg1` (`geography`): The input geography to be covered by S2 cells.
- `min_level` (`int64`): The minimum S2 cell level (0–30). Defaults to 0.
- `max_level` (`int64`): The maximum S2 cell level (0–30). Defaults to 30.
- `max_cells` (`int64`): The maximum number of cells to return. Must be > 0. Defaults to 8.

### Input
- **Data Type:** Requires a `geography` type. If starting with `geometry`, use `ST_ToGeography` or `ST_GeogFromWKT` to prepare the input.
- **Preconditions:** The session must have the `s2geography` bounder registered.
- **Formats:** Supports standard WKT/WKB inputs via conversion functions.

### Output
Returns `array` — An array of 64-bit integers representing the S2 cell IDs that cover the input geography. Returns an empty array for empty geometries.

### Valid Call Patterns
```python
# Basic usage
sd.sql("SELECT S2_CoveringCellIds(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'))").to_arrow_table()

# Full parameter usage
sd.sql("SELECT S2_CoveringCellIds(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'), 10, 12, 16)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the input is a `geography` type; if the input is `geometry`, wrap it in `ST_ToGeography()`.
- When specifying levels, ensure `min_level` and `max_level` are within the valid range of 0 to 30.
- Use the function to optimize spatial queries by filtering based on the returned cell ID array.

### Prompt Snippet
```text
S2_CoveringCellIds(geography_input, min_level, max_level, max_cells)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` type directly without converting to `geography` will cause an execution error.
- **Invalid Levels:** Providing `min_level` or `max_level` outside the 0–30 range will result in an error.
- **Missing Bounder:** Failure to register the `s2geography` bounder in the session will prevent the function from executing.

### Fix Code Hint
```python
# Ensure geography type and valid levels
query = """
SELECT S2_CoveringCellIds(ST_ToGeography(geom), 0, 30, 8) 
FROM my_table
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:SD_Format`

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
`sql:SD_Format` generates a human-readable or display-optimized string representation of a geospatial object or scalar value based on provided formatting constraints.

### Parameters
- `value` (`any`): The input object (e.g., `geometry`, `geography`, or scalar) to be formatted.
- `options` (`string`): A JSON-formatted string containing configuration keys, such as `width_hint`, to control the output display.

### Input
- **Data:** Accepts standard SedonaDB types including `geometry` (WKT/WKB-derived) and scalar types.
- **Preconditions:** The `options` parameter must be a valid JSON string.

### Output
Returns `any` — A formatted representation of the input, typically as a string, adjusted according to the provided JSON hints.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT SD_Format(ST_Point(1.0, 2.0), '{\"width_hint\": 16}')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `SD_Format(value, options)` as a standalone SQL function call.
- Ensure the `options` argument is a valid JSON string (e.g., `'{"width_hint": 16}'`).
- Use this function when preparing spatial data for logging, UI display, or debugging.

### Prompt Snippet
```text
Use SD_Format(value, '{"width_hint": <integer>}') to format spatial objects for display.
```

### Common Failure Modes
- **Invalid JSON:** Passing a malformed string as the `options` argument will cause a runtime error.
- **Type Mismatch:** Providing an unsupported type as the `value` argument may result in an error if the formatter cannot handle the specific object type.

### Fix Code Hint
```python
# Ensure options is a valid JSON string
options_json = '{"width_hint": 20}'
query = f"SELECT SD_Format(geom, '{options_json}') FROM my_table"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:SD_Order`

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
`sql:SD_Order` computes a sortable representation of a geospatial object or data value to facilitate deterministic ordering in SQL queries.

### Parameters
- `value` (`any`): The input geometry or data value to be transformed into a sortable format.

### Input
The function accepts standard OGC WKT/WKB geometries or other data types supported by SedonaDB. No specific file format is required as input, as the function operates on materialized in-memory objects within the SQL session.

### Output
Returns `any` — A value that can be used in `ORDER BY` clauses to sort the input records.

### Valid Call Patterns
```python
# Example using Python SQL connection
query = """
SELECT city, geom, SD_Order(geom) as sort_key
FROM my_table
ORDER BY sort_key
"""
table = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `SD_Order(value)` as a scalar function within a `SELECT` statement.
- Use the output of `SD_Order` in an `ORDER BY` clause to perform spatial or data-based sorting.
- Ensure the input `value` is a valid geometry or data type recognized by SedonaDB.

### Prompt Snippet
```text
Use SD_Order(geom) to generate a sortable key for geometry columns in SedonaDB.
```

### Common Failure Modes
- Passing `NULL` values may result in undefined sorting behavior depending on the underlying engine's handling of nulls in `ORDER BY`.
- Attempting to use `SD_Order` on complex types that do not have a defined sortable representation may result in a runtime error.

### Fix Code Hint
```python
# Ensure the input is not null before ordering
sd.sql("SELECT geom FROM my_table WHERE geom IS NOT NULL ORDER BY SD_Order(geom)").to_arrow_table()
```

## API Test: `sql:ST_Affine`

### Signature
```python
ST_Affine(geom: geometry, a: double, b: double, d: double, e: double, xOff: double, yOff: double) -> geometry | ST_Affine(geom: geometry, a: double, b: double, c: double, d: double, e: double, f: double, g: double, h: double, i: double, xOff: double, yOff: double, zOff: double) -> geometry
```
_Source: source/docs/reference/sql/st_affine.qmd:19_

### Goal
Applies a 2D or 3D affine transformation (scaling, rotation, shearing, and translation) to a geometry object.

### Parameters
- `geom` (`geometry`): The input geometry to be transformed.
- `a` (`double`): The coefficient for the X-axis scaling/rotation (x' = a*x + b*y + c*z + xOff).
- `b` (`double`): The coefficient for the Y-axis scaling/rotation.
- `c` (`double`): The coefficient for the Z-axis scaling/rotation (used in 3D overload).
- `d` (`double`): The coefficient for the X-axis scaling/rotation (y' = d*x + e*y + f*z + yOff).
- `e` (`double`): The coefficient for the Y-axis scaling/rotation.
- `f` (`double`): The coefficient for the Z-axis scaling/rotation (used in 3D overload).
- `g` (`double`): The coefficient for the X-axis scaling/rotation (z' = g*x + h*y + i*z + zOff).
- `h` (`double`): The coefficient for the Y-axis scaling/rotation.
- `i` (`double`): The coefficient for the Z-axis scaling/rotation.
- `xOff` (`double`): The X-axis translation offset.
- `yOff` (`double`): The Y-axis translation offset.
- `zOff` (`double`): The Z-axis translation offset (used in 3D overload).

### Input
The function accepts standard OGC WKT/WKB `geometry` types. The caller must ensure the geometry is valid; use `ST_MakeValid` if necessary. The transformation coefficients are standard affine matrix values.

### Output
Returns a new `geometry` object with the affine transformation applied to its coordinates.

### Valid Call Patterns
```python
# 2D Affine Transformation
sd.sql("SELECT ST_AsText(ST_Affine(ST_GeomFromText('POINT (1 2)'), 1, 0, 0, 1, 10, 20))").to_arrow_table()
```

### LLM Instruction Prompt
- Use the 6-parameter signature for 2D transformations and the 13-parameter signature for 3D transformations.
- Ensure all numeric parameters are provided as `double` types.
- The function does not modify the input geometry in place; it returns a new geometry.

### Prompt Snippet
```text
ST_Affine(geom, a, b, d, e, xOff, yOff)
ST_Affine(geom, a, b, c, d, e, f, g, h, i, xOff, yOff, zOff)
```

### Common Failure Modes
- Providing an incorrect number of arguments (e.g., 7 or 8 parameters) will result in a signature mismatch error.
- Applying a 3D transformation to a 2D geometry may result in unexpected coordinate behavior if the Z-dimension is not present.
- Invalid geometry inputs (e.g., self-intersecting polygons) may cause the transformation to fail or produce invalid output geometries.

### Fix Code Hint
```python
# Ensure the geometry is valid before transformation
sd.sql("SELECT ST_Affine(ST_MakeValid(geom), 1, 0, 0, 1, 10, 20) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Analyze_Agg`

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
Computes a comprehensive statistical summary of a collection of `geometry` or `geography` objects, including bounding box, geometry type counts, and average size/complexity metrics.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial column or expression to be aggregated.

### Input
The function accepts `geometry` or `geography` types. Inputs should be valid OGC-compliant geometries (WKT/WKB). For `geography` types, ensure the session's geography bounder is registered.

### Output
Returns a `struct` containing: `count`, `minx`, `miny`, `maxx`, `maxy`, `mean_size_in_bytes`, `mean_points_per_geometry`, `puntal_count`, `lineal_count`, `polygonal_count`, `geometrycollection_count`, `mean_envelope_width`, `mean_envelope_height`, and `mean_envelope_area`.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_Analyze_Agg(geom) as stats FROM my_table").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
When calling `ST_Analyze_Agg`, ensure the input is a valid spatial column. If the input is `geography`, verify that the session environment supports geography operations. Do not attempt to pass multiple arguments; this is a single-argument aggregate function.

### Prompt Snippet
```text
SELECT ST_Analyze_Agg(geometry_column) FROM table_name;
```

### Common Failure Modes
- Passing non-spatial types (e.g., strings or integers) instead of `geometry` or `geography`.
- Using `geography` types without the required `s2geography` bounder registered in the session.
- Attempting to use the function as a scalar operation on a single row without an aggregation context (though it functions as an aggregate).

### Fix Code Hint
```python
# Ensure input is cast to geometry if necessary
sd.sql("SELECT ST_Analyze_Agg(ST_GeomFromText(wkt_col)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Area`

### Signature
```python
ST_Area(arg1: geometry) -> double | ST_Area(arg1: geography) -> double
```
_Source: source/docs/reference/sql/st_area.qmd:19_

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

title: ST_Area
description: Returns the area of a geometry or geography.
kernels:
  - returns: double
    args: [geometry]
  - returns: double
    args: [geography]
---

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
Computes the surface area of a polygon or multi-polygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (polygon or multi-polygon) for which the area is calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geometry` types, the input must be in a projected coordinate system to obtain meaningful area units. For `geography` types, the session must have the `s2geography` bounder registered.

### Output
Returns `double` — The area of the input object. For `geometry`, the value is in the squared units of the coordinate reference system; for `geography`, the value is in square meters (spherical Earth model).

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Geometry area calculation
result_geom = sd.sql("SELECT ST_Area(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print(result_geom)
# __CHECK__

# Geography area calculation
result_geog = sd.sql("SELECT ST_Area(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print(result_geog)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Area` to calculate the area of polygons.
- Distinguish between `geometry` (planar, units of CRS) and `geography` (spherical, square meters).
- Ensure input geometries are valid polygons or multi-polygons.

### Prompt Snippet
```text
ST_Area(geometry_column)
```

### Common Failure Modes
- **Invalid Geometry:** Passing non-polygon types (e.g., `Point` or `LineString`) may result in an area of 0 or an error.
- **Geography Bounder:** Geography operations will fail if the `s2geography` bounder is not registered in the session.
- **Unit Confusion:** Assuming `geometry` area is in square meters when the CRS is in degrees or feet.

### Fix Code Hint
```python
# If geometry is in degrees, transform to a projected CRS before ST_Area
sd.sql("SELECT ST_Area(ST_Transform(geom, 3857)) FROM table")
```

## API Test: `sql:ST_AsBinary`

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
Converts a `geometry` or `geography` object into its Well-Known Binary (WKB) representation for efficient storage or serialization.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be converted into WKB format.

### Input
The input must be a valid `geometry` or `geography` object. These are typically generated via constructors like `ST_Point`, `ST_GeomFromText`, or `ST_GeogFromWKT`.

### Output
Returns `binary` — A byte stream representing the Well-Known Binary (WKB) format of the input spatial object.

### Valid Call Patterns
```python
# Example 1: Converting a geometry point
sd.sql("SELECT ST_AsBinary(ST_Point(1.0, 2.0))").to_arrow_table()

# Example 2: Converting a geography point
sd.sql("SELECT ST_AsBinary(ST_GeogPoint(1.0, 2.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_AsBinary(arg1)` to serialize spatial objects.
- If the downstream consumer requires standard `Binary` storage instead of `BinaryView`, wrap the call as `sd_simplifystorage(ST_AsBinary(arg1))`.
- Use the alias `ST_AsWKB` if preferred for compatibility.

### Prompt Snippet
```text
ST_AsBinary(geometry_column)
```

### Common Failure Modes
- **Storage Type Mismatch:** Downstream consumers expecting standard `Binary` may fail if the output is returned as `BinaryView` (the default behavior). Use `sd_simplifystorage()` to resolve this.
- **Invalid Input:** Passing a non-spatial type (e.g., `string` or `integer`) that cannot be implicitly cast to `geometry` or `geography`.

### Fix Code Hint
```python
# Wrap with sd_simplifystorage if standard Binary is required
query = "SELECT sd_simplifystorage(ST_AsBinary(geom)) FROM my_table"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_AsEWKB`

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
Converts a `geometry` or `geography` object into its Extended Well-Known Binary (EWKB) representation, which includes SRID metadata for PostGIS compatibility.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be serialized into the EWKB binary format.

### Input
- **Data:** A valid `geometry` or `geography` object created via constructors like `ST_Point` or `ST_GeomFromText`.
- **Preconditions:** The input must be a valid spatial object. If the object has an associated CRS, the SRID will be encoded into the binary header.

### Output
Returns `binary` — A byte stream representing the EWKB format of the input, containing the geometry type, coordinates, and SRID information.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example using a geometry with SRID
query = "SELECT ST_AsEWKB(ST_Point(1.0, 2.0, 4326)) as ewkb_data"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_AsEWKB(geometry_expression)` in SQL queries.
- Ensure the input geometry is properly constructed (e.g., using `ST_Point` or `ST_GeomFromText`).
- Use this function when PostGIS-compatible binary output is required for downstream applications.

### Prompt Snippet
```text
Use ST_AsEWKB(geometry) to serialize spatial objects into binary format with SRID headers.
```

### Common Failure Modes
- **Invalid Input:** Passing a non-spatial type (e.g., string or integer) instead of a `geometry` or `geography` object will result in a query execution error.
- **Missing SRID:** If the input geometry lacks an SRID, the resulting EWKB will not contain valid SRID metadata, potentially causing compatibility issues with strict PostGIS consumers.

### Fix Code Hint
```python
# Ensure the geometry has an SRID if needed for PostGIS compatibility
query = "SELECT ST_AsEWKB(ST_SetSRID(ST_Point(1.0, 2.0), 4326))"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_AsGeoJSON`

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
`sql:ST_AsGeoJSON` converts a SedonaDB `geometry` object into its corresponding GeoJSON string representation.

### Parameters
- `arg1` (`geometry`): The input geometry object (e.g., Point, Polygon, LineString) to be serialized into GeoJSON format.

### Input
- **Data:** A valid `geometry` object created via constructors like `ST_Point`, `ST_GeomFromText`, or derived from spatial operations.
- **Preconditions:** The input must be a valid `geometry` type. If the geometry is invalid, ensure it is repaired using `ST_MakeValid` before serialization.

### Output
Returns `string` — A GeoJSON-formatted string representing the geometry object.

### Valid Call Patterns
```python
# Using the Python SQL connection to execute the operation
result = sd.sql("SELECT ST_AsGeoJSON(ST_Point(1.0, 2.0))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_AsGeoJSON(geometry_expression)` within a `sd.sql()` query.
- Ensure the input is a valid `geometry` type; if working with WKT strings, wrap them in `ST_GeomFromText(...)` first.

### Prompt Snippet
```text
Use ST_AsGeoJSON(geometry_column) to serialize geometries to GeoJSON strings.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `string` (WKT) directly without using a constructor like `ST_GeomFromText` will result in a SQL execution error.
- **Invalid Geometry:** Attempting to serialize a geometry that is not OGC-compliant may result in unexpected output or errors; use `ST_IsValid` to verify before conversion.

### Fix Code Hint
```python
# Ensure input is a geometry object
query = "SELECT ST_AsGeoJSON(ST_GeomFromText('POINT(1 2)'))"
df = sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_AsText`

### Signature
```python
ST_AsText(arg1: geometry) -> string | ST_AsText(arg1: geography) -> string
```
_Source: source/docs/reference/sql/st_astext.qmd:19_

### Goal
Converts a `geometry` or `geography` object into its Well-Known Text (WKT) string representation.

### Parameters
- `arg1` (`geometry` | `geography`): The spatial object to be serialized into a WKT string.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) objects.
- **Preconditions:** The input must be a valid spatial object created via constructors like `ST_Point`, `ST_GeomFromText`, or `ST_GeogPoint`.

### Output
Returns `string` — The Well-Known Text (WKT) representation of the input spatial object.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example 1: Geometry
result_geom = sd.sql("SELECT ST_AsText(ST_Point(1.0, 2.0))").to_arrow_table()
print(result_geom)
# __CHECK__ witness: POINT (1.0 2.0)

# Example 2: Geography
result_geog = sd.sql("SELECT ST_AsText(ST_GeogPoint(1.0, 2.0))").to_arrow_table()
print(result_geog)
# __CHECK__ witness: POINT (1.0 2.0)
```

### LLM Instruction Prompt
- Use `ST_AsText` to serialize spatial objects for human-readable output or text-based storage.
- Note that `ST_AsText` is an alias for `ST_AsWKT`.
- Ensure the input is a valid spatial type; passing non-spatial types will result in a query execution error.

### Prompt Snippet
```text
Use ST_AsText(geometry_column) to convert spatial data to WKT format.
```

### Common Failure Modes
- **Invalid Input:** Passing a non-spatial type (e.g., `integer` or `string`) to the function will cause a type mismatch error.
- **Null Input:** If the input geometry is `NULL`, the function returns `NULL`.
- **Geography Bounder:** Ensure the session's geography bounder is registered if performing operations on `geography` types.

### Fix Code Hint
```python
# Ensure the input is cast or constructed as a geometry/geography
sd.sql("SELECT ST_AsText(ST_GeomFromText('POINT(1 2)'))").to_arrow_table()
```

## API Test: `sql:ST_Azimuth`

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
Calculates the clockwise angle in radians from the first point to the second point relative to true north.

### Parameters
- `arg1` (`geometry`): The starting point geometry.
- `arg2` (`geometry`): The target point geometry.

### Input
- **Data Types:** Requires two `geometry` objects.
- **Preconditions:** Both inputs must be valid `geometry` points. If either input is not a point, the function returns `NULL`.

### Output
Returns `double` — The azimuth angle in radians, measured clockwise from north.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example: Calculate azimuth from (0, 25) to (0, 0)
result = sd.sql("SELECT ST_Azimuth(ST_Point(0.0, 25.0), ST_Point(0.0, 0.0)) as azimuth").to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a double representing the angle in radians.
```

### LLM Instruction Prompt
- Always use `ST_Point` or similar geometry constructors to provide the `arg1` and `arg2` arguments.
- Ensure both arguments are valid point geometries; otherwise, the function will return `NULL`.
- The output is in radians; do not assume degrees.

### Prompt Snippet
```text
Use ST_Azimuth(point1, point2) to compute the clockwise angle in radians from point1 to point2.
```

### Common Failure Modes
- **Non-point inputs:** Passing a `POLYGON` or `LINESTRING` instead of a `POINT` results in a `NULL` return value.
- **Missing arguments:** Providing fewer than two arguments will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure inputs are points
query = "SELECT ST_Azimuth(ST_Point(x1, y1), ST_Point(x2, y2))"
# If inputs are stored in a table, ensure they are cast or verified as points
```

## API Test: `sql:ST_Boundary`

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
Computes the closure of the combinatorial boundary of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the boundary is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` types, ensure the session's `s2geography` bounder is registered.

### Output
Returns a `geometry` or `geography` object representing the boundary:
- For **Polygons**: Returns the exterior and interior rings.
- For **Linestrings**: Returns the endpoints.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_Boundary(ST_GeomFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_Boundary(ST_GeogFromWKT('POLYGON((1 1, 0 0, -1 1, 1 1))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Boundary(arg1)` as a standalone SQL function call.
- Ensure the input is explicitly cast to `geometry` or `geography` using constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT` if the input is a string.
- Match the input type (`geometry` vs `geography`) to the desired output type.

### Prompt Snippet
```text
Use ST_Boundary(geometry_expression) to extract the boundary of a polygon or linestring.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a function expecting `geography` or vice versa without proper conversion.
- **Geography Bounder:** Failure to register the `s2geography` bounder when processing `geography` types.
- **Empty Geometries:** Passing null or empty geometries may result in empty boundary outputs.

### Fix Code Hint
```python
# Ensure correct type conversion before calling ST_Boundary
sd.sql("SELECT ST_Boundary(ST_GeomFromWKT(wkt_column)) FROM my_table")
```

## API Test: `sql:ST_Buffer`

### Signature
```python
ST_Buffer(arg1: geometry, distance: float64) -> geometry | ST_Buffer(arg1: geometry, distance: float64, params: string) -> geometry | ST_Buffer(arg1: geography, distance: float64) -> geography | ST_Buffer(arg1: geography, distance: float64, num_quad_segs: integer) -> geography | ST_Buffer(arg1: geography, distance: float64, params: string) -> geography
```
_Source: source/docs/reference/sql/st_buffer.qmd:19_

### Goal
Computes a geometry or geography representing all points within a specified distance from the input geometry or geography.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to buffer.
- `distance` (`float64`): The radius of the buffer. For `geography`, this value is specified in meters. Negative values are supported for polygon inputs to shrink the polygon.
- `params` (`string`): Optional space-separated `key=value` parameters (e.g., `quad_segs`, `endcap`, `join`, `mitre_limit`, `side`) identical to PostGIS buffer parameter strings.

### Input
- **Data:** Standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` operations, the `s2geography` bounder must be registered on the session.

### Output
Returns `geography | geometry` representing the buffered spatial object.

### Valid Call Patterns
```python
# Geometry buffer with distance
sd.sql("SELECT ST_Buffer(geom, 1.0) AS result FROM input").to_arrow_table()

# Geometry buffer with parameters
sd.sql("SELECT ST_Buffer(geom, 1.0, 'quad_segs=2') AS result FROM input").to_arrow_table()

# Geography buffer
sd.sql("SELECT ST_Buffer(ST_GeogFromText('...'), 1000.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` for execution.
- Ensure the input `arg1` matches the expected type (`geometry` vs `geography`) for the chosen kernel.
- When using `params`, ensure the string follows the `key=value` format.

### Prompt Snippet
```text
ST_Buffer(geom, distance)
ST_Buffer(geom, distance, 'quad_segs=2')
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a `geography` kernel or vice versa.
- **Geography Precondition:** Failing to ensure the `s2geography` bounder is registered when performing geography-based buffering.
- **Invalid Parameters:** Providing unsupported keys in the `params` string.

### Fix Code Hint
```python
# Ensure correct type conversion if needed
sd.sql("SELECT ST_Buffer(ST_ToGeography(geom), 1000.0) FROM input").to_arrow_table()
```

## API Test: `sql:ST_BuildArea`

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
`ST_BuildArea` reconstructs a polygon or multipolygon geometry by interpreting the input geometry's edges as linework and closing any open rings.

### Parameters
- `arg1` (`geometry`): The input geometry (typically a `LINESTRING` or `MULTILINESTRING`) whose edges are used to form the resulting area.

### Input
- **Data Format:** Standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the input is already a `POLYGON` or `MULTIPOLYGON`, it is returned unchanged.

### Output
Returns `geometry` — A `POLYGON` or `MULTIPOLYGON` representing the enclosed area, or an empty polygon for empty input. Returns `NULL` if no closed ring can be formed.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example using WKT input
query = "SELECT ST_BuildArea(ST_GeomFromWKT('LINESTRING (0 0, 1 0, 1 1, 0 1, 0 0)'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a polygon geometry representing the square.
```

### LLM Instruction Prompt
- Always use `ST_BuildArea(geometry)` as a standalone SQL function within `sd.sql()`.
- Ensure the input is a valid geometry; if providing WKT, wrap it in `ST_GeomFromWKT()`.
- Remember that this function is intended for linework; if the input cannot form a closed ring, the function will return `NULL`.

### Prompt Snippet
```text
Use ST_BuildArea(geometry_expression) to convert linework into a polygon.
```

### Common Failure Modes
- **NULL Returns:** If the provided linework is insufficient to form a closed ring, the function returns `NULL`.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT strings will result in execution errors.
- **Type Mismatch:** Ensure the input is a `geometry` type; `geography` types may require conversion via `ST_ToGeometry` (if supported) or explicit casting.

### Fix Code Hint
```python
# Ensure input is a valid linework geometry
sd.sql("SELECT ST_BuildArea(ST_MakeValid(my_geom)) FROM my_table")
```

## API Test: `sql:ST_CRS`

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
Retrieves the Coordinate Reference System (CRS) metadata string associated with a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object whose CRS metadata is to be inspected.

### Input
- **Data:** A valid `geometry` or `geography` object created via constructors like `ST_Point`, `ST_GeomFromText`, or loaded from supported formats (GeoParquet, WKT, WKB).
- **Preconditions:** The object must have a CRS defined; if no CRS is associated, the behavior depends on the underlying implementation (typically returns an empty string or null).

### Output
Returns `string` — The CRS metadata in the exact format it was originally set (e.g., `'EPSG:4326'`, PROJJSON, or WKT).

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_CRS(ST_Point(0.25, 0.25, 4326)) AS crs_info").to_arrow_table()
print(result)
# __CHECK__ witness: crs_info column contains '4326' or 'EPSG:4326'
```

### LLM Instruction Prompt
- Always use `ST_CRS(arg1)` as a standalone SQL function within an `sd.sql()` query.
- Ensure the input `arg1` is a valid spatial object (geometry or geography).
- Do not attempt to modify the CRS using this function; it is for metadata inspection only.

### Prompt Snippet
```text
Use ST_CRS(geometry_column) to retrieve the CRS metadata string for a spatial object.
```

### Common Failure Modes
- **Missing CRS:** If the geometry was created without an SRID/CRS, the function may return an empty string or null.
- **Type Mismatch:** Passing a non-spatial type (e.g., `string` or `integer`) directly to `ST_CRS` will result in a SQL execution error.
- **Invalid Geometry:** If the input geometry is malformed or invalid, the function may fail to resolve the metadata.

### Fix Code Hint
```python
# Ensure the geometry has a CRS set before calling ST_CRS
sd.sql("SELECT ST_CRS(ST_SetSRID(ST_Point(0, 0), 4326))").to_arrow_table()
```

## API Test: `sql:ST_Centroid`

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
Computes the geometric center (centroid) of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the centroid is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` types, the session must have the `s2geography` bounder registered.

### Output
Returns a `geometry` or `geography` object representing the centroid point of the input.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Geometry example
result = sd.sql("SELECT ST_Centroid(ST_GeomFromWKT('LINESTRING (-64 45, 52 8)'))").to_arrow_table()
print(result)
# __CHECK__

# Geography example
result = sd.sql("SELECT ST_Centroid(ST_GeogFromWKT('LINESTRING (-64 45, 52 8)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Centroid` to find the center point of a spatial object.
- Ensure the input type matches the desired calculation method (planar `geometry` vs. spherical `geography`).
- Always wrap the input in the appropriate constructor (e.g., `ST_GeomFromWKT` or `ST_GeogFromWKT`) if the input is a string.

### Prompt Snippet
```text
SELECT ST_Centroid(ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` when a `geography` calculation is required (or vice versa) may lead to unexpected results or errors.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not initialized in the session.
- **Invalid Input:** Providing malformed WKT strings will cause the query to fail.

### Fix Code Hint
```python
# Ensure correct type usage
sd.sql("SELECT ST_Centroid(ST_GeogFromWKT(wkt_string)) FROM table")
```

## API Test: `sql:ST_ClosestPoint`

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
Computes the 2-dimensional point on the first geometry or geography object that is spatially nearest to the second object.

### Parameters
- `arg1` (`geometry` or `geography`): The reference object on which the closest point is to be found.
- `arg2` (`geometry` or `geography`): The target object used to determine proximity.

### Input
- **Data Types:** Accepts standard OGC WKT/WKB geometries or spherical geography types.
- **Preconditions:** Both inputs must share the same coordinate system or type (geometry vs. geography). If using `geography`, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A point object representing the location on `arg1` that minimizes the distance to `arg2`.

### Valid Call Patterns
```python
# Example using geography types
sd.sql("""
SELECT ST_ClosestPoint(
    ST_GeogFromText('LINESTRING(0 0, 10 10)'),
    ST_GeogPoint(5, 0)
)
""").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_ClosestPoint(arg1, arg2)` within a `sd.sql()` query.
- Ensure both arguments are of the same type (both `geometry` or both `geography`).
- If inputs are in different coordinate systems, perform a transformation using `ST_Transform` before calling this function.

### Prompt Snippet
```text
ST_ClosestPoint(geom1, geom2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as one argument and a `geography` as the other will cause an execution error.
- **CRS Inconsistency:** If inputs have conflicting or missing CRS metadata, the operation may fail or produce incorrect results.
- **Geography Bounder:** Failure to register the `s2geography` bounder in the session will prevent `geography` type operations.

### Fix Code Hint
```python
# Ensure consistent types and CRS
sd.sql("SELECT ST_ClosestPoint(ST_Transform(g1, 4326), ST_Transform(g2, 4326)) FROM table")
```

## API Test: `sql:ST_Collect_Agg`

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
Aggregates multiple individual `geometry` or `geography` objects from a set of rows into a single collection object.

### Parameters
- `arg1` (`geometry` or `geography`): The column containing the spatial objects to be collected.

### Input
- **Data:** A column of `geometry` or `geography` types.
- **Preconditions:** All inputs in the set must share the same type (either all `geometry` or all `geography`).

### Output
Returns a single `geometry` (as a `GeometryCollection` or `Multi`-type) or `geography` object representing the union of all input spatial features.

### Valid Call Patterns
```python
# Example using sd.sql() to aggregate points into a single collection
query = """
SELECT ST_Collect_Agg(geom) AS collected_points
FROM (
  VALUES
    (ST_Point(-122.4194, 37.7749)),
    (ST_Point(-118.2437, 34.0522)),
    (ST_Point(-122.6765, 45.5231))
) AS cities(geom);
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Collect_Agg` only as an aggregate function in a `SELECT` statement.
- Ensure all input geometries are of the same type to allow for `Multi`-type optimization.
- Do not mix `geometry` and `geography` types in the same aggregation.

### Prompt Snippet
```text
SELECT ST_Collect_Agg(column_name) FROM table_name;
```

### Common Failure Modes
- **Type Mismatch:** Attempting to aggregate a mix of `geometry` and `geography` types will result in an execution error.
- **Empty Input:** If the input set is empty, the result may be null or an empty collection depending on the underlying engine state.

### Fix Code Hint
```python
# Ensure all inputs are cast to the same type if necessary
sd.sql("SELECT ST_Collect_Agg(ST_GeomFromText(wkt)) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_ConcaveHull`

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
Computes a concave hull polygon that encloses the input geometry, with the tightness of the hull determined by the `pct_convex` parameter.

### Parameters
- `geom` (`geometry`): The input geometry (e.g., `LINESTRING`, `MULTIPOINT`) to be enclosed by the concave hull.
- `pct_convex` (`double`): A value between 0 and 1 that controls the concavity; 1.0 results in a convex hull, while smaller values result in a tighter, more concave shape.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the geometry is in a different CRS than the session, ensure it is transformed or that the CRS is consistent.

### Output
Returns `geometry` — A polygon representing the concave hull of the input geometry.

### Valid Call Patterns
```python
# Example using the documented SQL pattern via Python
query = """
SELECT ST_ConcaveHull(
    ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'),
    0.3
)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` to execute the query.
- Ensure the `pct_convex` parameter is a `double` (e.g., `0.3` or `1.0`).
- Use `ST_GeomFromText` or similar constructors to provide the `geometry` input if not using a registered view.

### Prompt Snippet
```text
ST_ConcaveHull(geom, pct_convex)
```

### Common Failure Modes
- **Invalid Geometry:** Passing a geometry that is not supported or is malformed.
- **Parameter Range:** Providing a `pct_convex` value outside the expected range (though 1.0 is the standard upper bound for convex results).
- **CRS Mismatch:** If the input geometry has a CRS that conflicts with the session or other operations, the function may fail or produce unexpected results.

### Fix Code Hint
```python
# Ensure geometry is valid and pct_convex is a float
sd.sql("SELECT ST_ConcaveHull(ST_MakeValid(geom), 0.5) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_Contains`

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
- **Data Formats:** Standard OGC WKT/WKB geometries or `geography` types.
- **Preconditions:** Both arguments must share the same spatial type (`geometry` or `geography`). If using `geography`, the session must have the `s2geography` bounder registered. If CRS differs, geometries are reprojected to the container's CRS; if only one side has a CRS, the operation fails.

### Output
Returns `boolean` — `true` if no points of `arg2` lie outside `arg1` and at least one point of `arg2` lies inside `arg1`; otherwise `false`.

### Valid Call Patterns
```python
# Geometry containment
result = sd.sql("SELECT ST_Contains(ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), ST_Point(0.25, 0.25)) as val").to_arrow_table()
assert result["val"][0] == True
print("__CHECK__")

# Geography containment
result = sd.sql("SELECT ST_Contains(ST_GeogFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), ST_GeogPoint(0.25, 0.25)) as val").to_arrow_table()
assert result["val"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (both `geometry` or both `geography`).
- Use `ST_GeomFromText` for planar `geometry` and `ST_GeogFromText` for spherical `geography`.
- Ensure CRS consistency; if inputs have different CRS, use `ST_Transform` to align them before calling `ST_Contains`.

### Prompt Snippet
```text
SELECT ST_Contains(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will cause an execution error.
- **CRS Inconsistency:** If one input has a CRS and the other does not, the operation will fail.
- **Geography Bounder:** Using `geography` types without the `s2geography` session bounder registered.

### Fix Code Hint
```python
# Ensure both are geometry
sd.sql("SELECT ST_Contains(ST_Transform(geom1, 4326), ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `sql:ST_ConvexHull`

### Signature
```python
ST_ConvexHull(arg1: geometry) -> geometry | ST_ConvexHull(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_convexhull.qmd:19_

### Goal
Computes the smallest convex polygon that encloses all points within a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object (e.g., `MULTIPOINT`, `LINESTRING`, or `POLYGON`) for which the convex hull is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` inputs, ensure the session's `s2geography` bounder is registered. If using `geometry` types, ensure the input is in the desired planar coordinate system.

### Output
Returns a `geometry` or `geography` object representing the convex hull polygon.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_ConvexHull(ST_GeomFromText('MULTIPOINT ((0 0), (0 1), (1 0))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_ConvexHull(ST_GeogFromWKT('MULTIPOINT ((0 0), (0 1), (1 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_ConvexHull` to derive the bounding convex polygon for a set of points or complex geometries.
- Always match the input type (`geometry` vs `geography`) to the appropriate constructor (`ST_GeomFromText` vs `ST_GeogFromWKT`).
- If performing group-wise operations, prefer the aggregate variant `ST_ConvexHull_Agg` instead of manual grouping.

### Prompt Snippet
```text
ST_ConvexHull(arg1)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geography` type to a function expecting a `geometry` kernel (or vice versa) may result in execution errors.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not initialized in the current session.
- **Empty Input:** Providing an empty geometry or null input may return an empty result or null, depending on the underlying engine state.

### Fix Code Hint
```python
# Ensure input is cast to the correct type if necessary
sd.sql("SELECT ST_ConvexHull(ST_GeomFromText('...'))")
```

__CHECK__

## API Test: `sql:ST_ConvexHull_Agg`

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
- `arg1` (`geometry`): A column or expression containing the geometries to be aggregated.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a set of geometries grouped by a SQL `GROUP BY` clause or processed as a single aggregate over the entire result set.

### Output
Returns `geometry` — A single polygon representing the convex hull of all input geometries.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using the aggregate function in a SQL query
result = sd.sql("""
    SELECT ST_ConvexHull_Agg(geom) 
    FROM (VALUES (ST_Point(0, 0)), (ST_Point(0, 1)), (ST_Point(1, 0)), (ST_Point(1, 1))) AS t(geom)
""").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_ConvexHull_Agg` as an aggregate function within a `SELECT` statement.
- Ensure the input column contains valid `geometry` types.
- If grouping is required, include a `GROUP BY` clause in the SQL query.

### Prompt Snippet
```text
SELECT ST_ConvexHull_Agg(geom) FROM table_name GROUP BY category_column;
```

### Common Failure Modes
- **Empty Input:** If the input group is empty, the result may be null or undefined.
- **Invalid Geometries:** Passing non-geometry types or malformed WKT/WKB will cause execution errors.
- **CRS Mismatch:** While the function operates on geometries, ensure all inputs share the same coordinate reference system to avoid unexpected spatial results.

### Fix Code Hint
```python
# Ensure geometries are valid before aggregation
sd.sql("SELECT ST_ConvexHull_Agg(ST_MakeValid(geom)) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_CoveredBy`

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
Determines if the first geometry is entirely contained within the second geometry, returning true if no point of the first geometry lies outside the second.

### Parameters
- `arg1` (`geometry`): The geometry to be tested for coverage.
- `arg2` (`geometry`): The geometry acting as the covering boundary.

### Input
- **Data Types:** Standard OGC `geometry` types (Points, Polygons, etc.).
- **Formats:** WKT, WKB, or EWKT/EWKB.
- **Preconditions:** Both geometries should ideally share the same CRS; if CRS differs, SedonaDB will attempt to reproject to the target CRS. If only one side has a CRS, the operation may fail.

### Output
Returns `boolean` — `true` if `arg1` is covered by `arg2`, otherwise `false`.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_CoveredBy(
        ST_Point(0.25, 0.25),
        ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
    ) as is_covered
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_CoveredBy(geomA, geomB)` where `geomA` is the inner geometry and `geomB` is the outer boundary.
- Ensure both inputs are valid `geometry` objects; use `ST_MakeValid` if input geometries are potentially malformed.

### Prompt Snippet
```text
ST_CoveredBy(geometry_inner, geometry_outer)
```

### Common Failure Modes
- **CRS Mismatch:** If one geometry has a defined CRS and the other does not, the operation may fail. Ensure both are in the same coordinate system.
- **Invalid Geometries:** Passing self-intersecting or otherwise invalid geometries can lead to unexpected results; use `ST_MakeValid` beforehand.
- **Type Confusion:** Attempting to pass `geography` types to this `geometry` kernel may result in errors; ensure types are consistent.

### Fix Code Hint
```python
# Ensure geometries are valid and in the same CRS before checking coverage
query = """
SELECT ST_CoveredBy(
    ST_MakeValid(geom1),
    ST_MakeValid(geom2)
) 
FROM my_table
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_Covers`

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
Determines if the first geometry spatially covers the second geometry, returning true if no point in the second geometry lies outside the first.

### Parameters
- `arg1` (`geometry`): The covering geometry (e.g., Polygon).
- `arg2` (`geometry`): The geometry to be covered (e.g., Point, LineString, or Polygon).

### Input
- **Data Formats:** Standard OGC WKT or WKB geometries.
- **Preconditions:** Both geometries must be in the same coordinate reference system (CRS). If CRS differs, geometries must be reprojected to a common CRS before calling this function.

### Output
Returns `boolean` — `true` if `arg1` covers `arg2`, otherwise `false`.

### Valid Call Patterns
```python
# Example using Python SQL connection
query = """
SELECT ST_Covers(
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_Point(0.25, 0.25)
) AS val
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: val=True
```

### LLM Instruction Prompt
- Always use `ST_GeomFromText` or similar constructors to define input geometries if they are not already registered as views.
- Ensure both geometries share the same CRS; if not, use `ST_Transform` to align them.

### Prompt Snippet
```text
ST_Covers(geometry_a, geometry_b)
```

### Common Failure Modes
- **CRS Mismatch:** If the input geometries have different CRS definitions, the operation will fail.
- **Invalid Geometries:** Passing malformed WKT strings or invalid geometries (e.g., self-intersecting polygons) may result in runtime errors; use `ST_MakeValid` if necessary.
- **Null Inputs:** Passing `NULL` as either argument will result in a `NULL` output.

### Fix Code Hint
```python
# Ensure CRS alignment before checking coverage
sd.sql("""
SELECT ST_Covers(
    geom_a, 
    ST_Transform(geom_b, ST_SRID(geom_a))
) 
FROM my_table
""").to_arrow_table()
```

## API Test: `sql:ST_Crosses`

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
Determines if two geometries spatially cross each other by sharing some, but not all, interior points, with an intersection dimension lower than the input geometries.

### Parameters
- `arg1` (`geometry`): The first geometry object (e.g., created via `ST_GeomFromWKT`).
- `arg2` (`geometry`): The second geometry object to test against the first.

### Input
Requires two valid `geometry` objects. Inputs must be in the same Coordinate Reference System (CRS); if CRS differs, geometries must be reprojected to a common CRS before comparison.

### Output
Returns `boolean` — `true` if the geometries cross, `false` otherwise.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_Crosses(
        ST_GeomFromWKT('LINESTRING(0 0, 2 2)'),
        ST_GeomFromWKT('LINESTRING(0 2, 2 0)')
    )
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_GeomFromWKT` or similar constructors to define input geometries within the SQL string.
- Ensure both geometries are in the same CRS; if not, use `ST_Transform` to align them before calling `ST_Crosses`.

### Prompt Snippet
```text
ST_Crosses(ST_GeomFromWKT('...'), ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **CRS Mismatch:** If inputs have different CRS, the operation may fail or return incorrect results.
- **Invalid Geometries:** Passing malformed WKT strings or invalid geometries (e.g., self-intersecting polygons where not allowed) will cause execution errors.
- **Unsupported Types:** Attempting to use `geography` types where `geometry` is expected without explicit conversion.

### Fix Code Hint
```python
# Ensure CRS alignment if inputs originate from different sources
query = """
SELECT ST_Crosses(
    ST_Transform(geom1, 'EPSG:4326'),
    ST_Transform(geom2, 'EPSG:4326')
) FROM my_table
"""
```

## API Test: `sql:ST_DWithin`

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
Determines if two spatial objects (geometry or geography) are within a specified distance of each other.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to evaluate.
- `arg2` (`geometry` or `geography`): The second spatial object to evaluate.
- `distance` (`double`): The threshold distance. For `geography` types, this value must be in meters.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) objects.
- **Preconditions:** For `geography` operations, the session must have the `s2geography` bounder registered.
- **Consistency:** Both inputs must be of the same type (both `geometry` or both `geography`).

### Output
Returns `boolean` — `true` if the distance between the two objects is less than or equal to the specified `distance`, otherwise `false`.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_DWithin(ST_Point(0, 0), ST_Point(1, 1), 2.0)").to_arrow_table()

# Geography example
sd.sql("SELECT ST_DWithin(ST_GeogPoint(0, 0), ST_GeogPoint(1, 1), 200000.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both `arg1` and `arg2` are of the same spatial type (geometry or geography).
- If using `geography`, ensure the `distance` parameter is provided in meters.
- Use `ST_Point` for geometry inputs and `ST_GeogPoint` for geography inputs.

### Prompt Snippet
```text
ST_DWithin(arg1, arg2, distance)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` and a `geography` object in the same call will result in an error.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered in the current session.
- **Unit Confusion:** Providing distance in degrees for `geography` types instead of meters will lead to incorrect results.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_DWithin(ST_Transform(geom1, 4326)::geography, ST_Transform(geom2, 4326)::geography, 100.0)")
```

## API Test: `sql:ST_DelaunayTriangles`

### Signature
```python
ST_DelaunayTriangles(arg1: geometry) -> geometry | ST_DelaunayTriangles(arg1: geometry, tolerance: float64) -> geometry | ST_DelaunayTriangles(arg1: geometry, tolerance: float64, flags: integer) -> geometry
```
_Source: source/docs/reference/sql/st_delaunaytriangles.qmd:19_

### Goal
Computes a Delaunay triangulation of the vertices of an input geometry, returning a collection of triangles that cover the input's vertex set.

### Parameters
- `arg1` (`geometry`): The input geometry whose vertices will be used to construct the triangulation.
- `tolerance` (`float64`): Snap-rounding tolerance for vertices; use 0.0 for exact computation.
- `flags` (`integer`): Output format control: 0 for `GEOMETRYCOLLECTION` of `POLYGON` triangles, or 1 for `MULTILINESTRING` edge-only output.

### Input
The function accepts standard OGC WKT/WKB `geometry` types. Input geometries should be prepared as valid spatial objects (e.g., via `ST_GeomFromWKT`).

### Output
Returns a `geometry` object. By default, this is a `GEOMETRYCOLLECTION` of `POLYGON` triangles; if `flags` is set to 1, it returns a `MULTILINESTRING`.

### Valid Call Patterns
```python
# Standard triangulation
sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'))").to_arrow_table()

# Triangulation with exact tolerance
sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'), 0.0)").to_arrow_table()

# Triangulation with edge-only output (flags=1)
sd.sql("SELECT ST_DelaunayTriangles(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 0), (0 1))'), 0.0, 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` for execution.
- Ensure the input geometry is valid; use `ST_MakeValid` if necessary before passing to `ST_DelaunayTriangles`.
- When using the `flags` parameter, ensure the `tolerance` parameter is also provided as a `float64`.

### Prompt Snippet
```text
Use ST_DelaunayTriangles(geometry, tolerance, flags) to generate triangulated meshes from point sets.
```

### Common Failure Modes
- Providing an invalid geometry that cannot be triangulated.
- Omitting the `tolerance` argument when attempting to specify the `flags` parameter.
- Mismatched types: Ensure `tolerance` is passed as a float (e.g., `0.0`) and `flags` as an integer.

### Fix Code Hint
```python
# Ensure correct argument types and order
# Incorrect: ST_DelaunayTriangles(geom, 1) -> Might fail if 1 is interpreted as int
# Correct: ST_DelaunayTriangles(geom, 0.0, 1)
result = sd.sql("SELECT ST_DelaunayTriangles(geom, 0.0, 1) FROM my_table").to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_Difference`

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
    ST_GeogFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
) AS val;
```


### Goal
Computes the spatial set difference, returning the portion of the first geometry that does not overlap with the second geometry.

### Parameters
- `arg1` (`geometry`): The primary geometry or geography object from which the intersection will be subtracted.
- `arg2` (`geometry`): The geometry or geography object representing the area to be removed from the first argument.

### Input
Requires two valid `geometry` or `geography` objects. Inputs must be of the same type (both `geometry` or both `geography`). For `geography` types, the session must have the `s2geography` bounder registered.

### Output
Returns `geography | geometry` — A new geometry or geography object representing the remainder of `arg1` after removing the intersection with `arg2`.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Difference(ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'), ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Difference(ST_GeogFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'), ST_GeogFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (`geometry` or `geography`).
- Use `ST_GeomFromText` for planar `geometry` inputs and `ST_GeogFromText` for spherical `geography` inputs.
- The function is called as a standalone SQL function within `sd.sql()`.

### Prompt Snippet
```text
ST_Difference(arg1, arg2)
```

### Common Failure Modes
- Type Mismatch: Attempting to pass a `geometry` as `arg1` and a `geography` as `arg2` will result in an error.
- Geography Bounder: Using `geography` types without the `s2geography` bounder registered on the session.
- Invalid Geometries: Passing malformed WKT strings or invalid OGC geometries.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_Difference(ST_GeomFromText(wkt1), ST_GeomFromText(wkt2)) FROM my_table")
```

## API Test: `sql:ST_Dimension`

### Signature
```python
ST_Dimension(arg1: geometry) -> integer | ST_Dimension(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_dimension.qmd:19_

### Goal
Returns the inherent spatial dimension of a `geometry` or `geography` object, identifying whether it is a point (0), line (1), or polygon (2).

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. For `geography` types, ensure the session's geography bounder is registered.

### Output
Returns `integer`:
- `0`: Points or multipoints.
- `1`: Linestrings or multilinestrings.
- `2`: Polygons or multipolygons.
- For geometry collections, it returns the largest dimension among the contained components.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_Dimension(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_Dimension(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Dimension(arg1)` to determine the dimensionality of a spatial object.
- Ensure the input is explicitly cast to `geometry` or `geography` if it is provided as a raw WKT string (e.g., using `ST_GeomFromWKT` or `ST_GeogFromWKT`).

### Prompt Snippet
```text
SELECT ST_Dimension(ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or a raw string without a constructor (e.g., `ST_GeomFromWKT`) will result in a SQL execution error.
- **Geography Bounder:** If using `geography` types, failure to have the `s2geography` bounder registered on the session will cause the operation to fail.

### Fix Code Hint
```python
# Ensure input is wrapped in a constructor if passing a WKT string
result = sd.sql("SELECT ST_Dimension(ST_GeomFromWKT('POINT(0 0)'))").to_arrow_table()
assert result[0].as_py() == 0
print("__CHECK__")
```

## API Test: `sql:ST_Disjoint`

### Signature
```python
ST_Disjoint(arg1: geometry, arg2: geometry) -> boolean | ST_Disjoint(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_disjoint.qmd:19_

### Goal
Determines if two spatial objects (geometries or geographies) are disjoint, meaning they do not share any common points.

### Parameters
- `arg1` (`geometry` | `geography`): The first spatial object to evaluate.
- `arg2` (`geometry` | `geography`): The second spatial object to evaluate.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** Both arguments must be of the same type (both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered. If coordinate systems differ, geometries are reprojected to the target CRS; if only one side has a CRS, the operation will fail.

### Output
Returns `boolean` — `true` if the two objects share no points, `false` otherwise. This is the logical inverse of `ST_Intersects`.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_Disjoint(ST_GeomFromText('POLYGON ((0 0, 1 0, 0 1, 0 0))'), ST_GeomFromText('POLYGON ((2 2, 3 2, 2 3, 2 2))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_Disjoint(ST_GeogFromText('POLYGON ((0 0, 1 0, 0 1, 0 0))'), ST_GeogFromText('POLYGON ((2 2, 3 2, 2 3, 2 2))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Disjoint(arg1, arg2)` as a standalone SQL function call.
- Ensure both arguments are explicitly cast to the same type (`geometry` or `geography`) using constructors like `ST_GeomFromText` or `ST_GeogFromText` if they are provided as strings.
- Verify that both inputs share a compatible CRS or have no CRS defined to avoid runtime errors.

### Prompt Snippet
```text
ST_Disjoint(geom1, geom2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` and a `geography` in the same call.
- **CRS Inconsistency:** Providing inputs with different coordinate systems where one is missing a CRS definition.
- **Invalid Geometries:** Passing malformed WKT strings that fail to parse into valid spatial objects.

### Fix Code Hint
```python
# Ensure consistent types and valid WKT
sd.sql("""
    SELECT ST_Disjoint(
        ST_GeomFromText('POINT(0 0)', 4326), 
        ST_GeomFromText('POINT(1 1)', 4326)
    )
""").to_arrow_table()
```

## API Test: `sql:ST_Distance`

### Signature
```python
ST_Distance(arg1: geometry, arg2: geometry) -> double | ST_Distance(arg1: geography, arg2: geography) -> double
```
_Source: source/docs/reference/sql/st_distance.qmd:19_

### Goal
Computes the distance between two spatial objects, returning the 2D Cartesian distance for `geometry` types or the spherical geodesic distance for `geography` types.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object.
- `arg2` (`geometry` or `geography`): The second spatial object.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) objects.
- **Preconditions:** For `geometry` inputs, both objects should ideally share the same coordinate reference system (CRS). For `geography` inputs, the session must have the `s2geography` bounder registered.
- **Formats:** Geometries can be constructed via `ST_Point`, `ST_GeomFromText`, or `ST_GeomFromWKT`.

### Output
Returns `double` — The distance value. For `geometry`, the value is in the units of the coordinate reference system; for `geography`, the value is in meters.

### Valid Call Patterns
```python
# Geometry distance (Cartesian)
sd.sql("SELECT ST_Distance(ST_Point(0, 0), ST_Point(1, 1))").to_arrow_table()

# Geography distance (Spherical)
sd.sql("SELECT ST_Distance(ST_GeogPoint(0, 0), ST_GeogPoint(1, 1))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Distance` for calculating spatial separation.
- Ensure both arguments are of the same type (`geometry` or `geography`).
- If using `geography`, ensure the session is configured for spherical calculations.

### Prompt Snippet
```text
SELECT ST_Distance(geom1, geom2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` and a `geography` object to the same function call will result in an error.
- **CRS Mismatch:** For `geometry` types, if the inputs have different coordinate systems, the result may be mathematically invalid or the operation may fail.
- **Missing Bounder:** Geography operations require the `s2geography` bounder to be registered on the session; failure to do so will cause runtime errors.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_Distance(ST_Transform(geom1, 4326), ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `sql:ST_Dump`

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
- `arg1` (`geometry` or `geography`): The input geometry or geography object to be expanded.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If the input is a simple geometry, it is returned as-is.

### Output
Returns `struct` — A record containing the expanded components of the input geometry or collection.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example using SQL query via Python
query = "SELECT ST_Dump(ST_GeomFromText('MULTIPOINT ((10 40), (40 30), (20 20), (30 10))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Dump` to flatten multi-part geometries (e.g., `MULTIPOINT`, `MULTIPOLYGON`, `GEOMETRYCOLLECTION`) into individual rows.
- Ensure the input is a valid geometry or geography object; use `ST_GeomFromText` or similar constructors if passing raw WKT strings.

### Prompt Snippet
```text
Use ST_Dump(geometry_column) to expand multi-part geometries into individual components.
```

### Common Failure Modes
- **Invalid Input:** Passing a non-geometry type or a malformed WKT string will result in a runtime error.
- **Simple Geometry:** If the input is already a simple geometry (e.g., `POINT`), the function returns the geometry itself without expansion.
- **Geography/Geometry Mismatch:** Ensure the input type matches the expected kernel; geography operations may require the `s2geography` bounder to be registered on the session.

### Fix Code Hint
```python
# Ensure input is valid geometry before dumping
sd.sql("SELECT ST_Dump(ST_MakeValid(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_EndPoint`

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
Returns the final coordinate point of a given `LINESTRING` geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input linestring geometry or geography from which the last point is extracted.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a linestring type. If the input is a `geography` type, ensure the session's geography bounder is registered.

### Output
Returns a `geometry` or `geography` object representing the last point of the input linestring.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_EndPoint(ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_EndPoint` to extract the terminal vertex of a linestring.
- Ensure the input is a valid linestring; non-linestring geometries may result in undefined behavior or errors.
- Maintain consistency between `geometry` and `geography` types; do not mix them in operations unless explicitly cast.

### Prompt Snippet
```text
Use ST_EndPoint(geometry_column) to retrieve the final point of a linestring.
```

### Common Failure Modes
- **Invalid Input Type:** Passing a point, polygon, or non-linestring geometry will cause the operation to fail.
- **Geography Bounder:** Geography operations require the `s2geography` bounder to be registered on the session; failure to do so will result in execution errors.
- **Null Inputs:** Passing a `NULL` geometry will return `NULL`.

### Fix Code Hint
```python
# Ensure input is a valid linestring before calling ST_EndPoint
sd.sql("SELECT ST_EndPoint(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_LineString'")
```

## API Test: `sql:ST_Envelope`

### Signature
```python
ST_Envelope(arg1: geometry) -> geometry | ST_Envelope(arg1: geography) -> geometry
```
_Source: source/docs/reference/sql/st_envelope.qmd:19_

### Goal
Computes the minimum axis-aligned bounding box (envelope) for a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the bounding box is calculated.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Supports standard OGC WKT/WKB inputs, typically constructed via `ST_GeomFromWKT` or `ST_GeogFromWKT`.
- **Preconditions:** For `geography` inputs, ensure the session's geography bounder is registered.

### Output
Returns `geometry` — A rectangular polygon representing the minimum axis-aligned bounding box. If the input is a point, the output is a point; for perfectly vertical or horizontal linestrings, the output may be a linestring. For `geography` objects spanning the antimeridian, a `MULTIPOLYGON` is returned.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Geometry example
result = sd.sql("SELECT ST_Envelope(ST_GeomFromWKT('LINESTRING(0 0, 3 4)'))").to_arrow_table()
print("CHECK: Envelope computed successfully")

# Geography example
result = sd.sql("SELECT ST_Envelope(ST_GeogFromWKT('LINESTRING(0 0, 3 4)'))").to_arrow_table()
print("CHECK: Geography envelope computed successfully")
```

### LLM Instruction Prompt
- Always use `ST_Envelope(arg1)` as a standalone SQL function call.
- If the input is a `geography` type, be aware that the resulting bounds are slightly expanded compared to `geometry` due to numerical inaccuracies in spherical calculations.
- Use `ST_GeomFromWKT` or `ST_GeogFromWKT` to prepare inputs if they are provided as strings.

### Prompt Snippet
```text
SELECT ST_Envelope(ST_GeomFromWKT('...'));
```

### Common Failure Modes
- **Type Mismatch:** Passing non-spatial types or unsupported formats will result in a SQL execution error.
- **Geography Antimeridian:** Expect a `MULTIPOLYGON` return type when processing `geography` objects that cross the antimeridian; downstream code expecting a simple `POLYGON` may fail.
- **Numerical Inaccuracy:** Do not assume the envelope of a `geography` object is identical to its `geometry` counterpart; geographical bounds are intentionally expanded.

### Fix Code Hint
```python
# Ensure input is cast to the correct spatial type if necessary
sd.sql("SELECT ST_Envelope(ST_GeomFromWKT(wkt_column)) FROM my_table")
```

## API Test: `sql:ST_Envelope_Agg`

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
Computes the collective bounding box (envelope) of a set of geometries or geographies grouped within a SQL query.

### Parameters
- `arg1` (`geometry` or `geography`): The input column containing the spatial objects to be aggregated into a single bounding box.

### Input
- Data: A column of `geometry` or `geography` types.
- Preconditions: For `geography` inputs, ensure the `s2geography` bounder is registered on the session.

### Output
Returns `geometry` — A single polygon representing the minimum bounding box that encompasses all input geometries. If the input is a geography crossing the antimeridian, it returns a multi-geometry.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using SQL aggregate syntax
result = sd.sql("""
    SELECT ST_Envelope_Agg(geom) 
    FROM (VALUES (ST_Point(0, 0)), (ST_Point(5, 5)), (ST_Point(10, 2))) AS t(geom)
""").to_arrow_table()
print("__CHECK__", result)
```

### LLM Instruction Prompt
- Use `ST_Envelope_Agg` as an aggregate function in a `SELECT` statement.
- Ensure the input column contains valid spatial types (`geometry` or `geography`).
- Do not expect a `geography` return type; the result is always a `geometry`.

### Prompt Snippet
```text
SELECT ST_Envelope_Agg(column_name) FROM table_name;
```

### Common Failure Modes
- Attempting to use `ST_Envelope_Agg` on non-spatial data types.
- Geography inputs failing if the `s2geography` bounder is not initialized in the session.
- Empty input sets resulting in null or undefined behavior depending on the SQL engine state.

### Fix Code Hint
```python
# Ensure inputs are cast to geometry/geography if they are WKT strings
sd.sql("SELECT ST_Envelope_Agg(ST_GeomFromText(wkt_col)) FROM table_name")
```

## API Test: `sql:ST_Equals`

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
Determines if two spatial objects are topologically equivalent by verifying they represent the same point set, ignoring differences in vertex ordering.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to compare.
- `arg2` (`geometry` or `geography`): The second spatial object to compare.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Inputs are typically constructed via `ST_GeomFromText` (for geometry) or `ST_GeogFromText` (for geography).
- **Preconditions:** Both arguments must be of the same type (both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered.

### Output
Returns `boolean` — `true` if the geometries are topologically equal, `false` otherwise.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Equals(ST_GeomFromText('LINESTRING(0 0, 1 1)'), ST_GeomFromText('LINESTRING(1 1, 0 0)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Equals(ST_GeogFromText('LINESTRING(0 0, 1 1)'), ST_GeogFromText('LINESTRING(1 1, 0 0)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (`geometry` vs `geography`).
- Use `ST_GeomFromText` for planar geometry comparisons and `ST_GeogFromText` for spherical geography comparisons.
- Do not attempt to compare a `geometry` object directly with a `geography` object.

### Prompt Snippet
```text
Use ST_Equals(arg1, arg2) to check for topological equality. Ensure both arguments are cast to the same type (geometry or geography) using appropriate constructors like ST_GeomFromText or ST_GeogFromText.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` object and a `geography` object in the same call will result in an execution error.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered in the current session.
- **Invalid WKT:** Providing malformed WKT strings to the constructors will cause the query to fail.

### Fix Code Hint
```python
# Ensure consistent types
sd.sql("SELECT ST_Equals(ST_GeomFromText('POINT(1 1)'), ST_GeomFromText('POINT(1 1)'))")
```
__CHECK__

## API Test: `sql:ST_ExteriorRing`

### Signature
```python
ST_ExteriorRing(arg1: geometry) -> geometry | ST_ExteriorRing(arg1: geography) -> geography
```
_Source: source/docs/reference/sql/st_exteriorring.qmd:19_

### Goal
Extracts the outer boundary of a polygon as a `LINESTRING` geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The input polygon from which the exterior ring is to be extracted.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a polygon type. If the input is not a polygon, the function returns `NULL`.

### Output
Returns `geometry` or `geography` — A `LINESTRING` representing the exterior boundary of the input polygon.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_ExteriorRing(ST_GeomFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_ExteriorRing(ST_GeogFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_ExteriorRing` to isolate the outer boundary of a polygon.
- Ensure the input is a polygon; non-polygon inputs will result in `NULL`.
- Match the input type (`geometry` vs `geography`) to the appropriate constructor (`ST_GeomFromWKT` vs `ST_GeogFromWKT`).

### Prompt Snippet
```text
SELECT ST_ExteriorRing(ST_GeomFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))'));
```

### Common Failure Modes
- **Non-Polygon Input:** Passing a `POINT` or `LINESTRING` will return `NULL` rather than raising an error.
- **Type Mismatch:** Attempting to use a `geometry` function on a `geography` object without proper conversion or vice versa may lead to execution errors depending on the session's bounder configuration.

### Fix Code Hint
```python
# Ensure input is a polygon before calling
sd.sql("SELECT ST_ExteriorRing(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Polygon'")
```

## API Test: `sql:ST_FlipCoordinates`

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
Returns a new geometry or geography object where the X and Y coordinate values of every vertex in the input are swapped.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object (e.g., Point, LineString, Polygon) whose vertex coordinates are to be flipped.

### Input
- **Data Types:** Accepts standard OGC `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The input must be a valid spatial object constructed via functions like `ST_Point`, `ST_GeomFromText`, or `ST_GeogFromWKT`.

### Output
Returns a `geometry` or `geography` object (matching the input type) where the X and Y coordinates are transposed.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_FlipCoordinates(ST_Point(1, 2))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_FlipCoordinates(arg1)` to swap X and Y coordinates.
- Ensure the input `arg1` is a valid `geometry` or `geography` type.
- The function preserves the original spatial type (geometry remains geometry, geography remains geography).

### Prompt Snippet
```text
ST_FlipCoordinates(geometry_or_geography_input)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., string or integer) instead of a `geometry` or `geography` object will result in a SQL execution error.
- **Null Input:** Passing `NULL` as an argument will return `NULL`.
- **Coordinate System Assumptions:** While the function swaps coordinates, it does not perform a coordinate system transformation; ensure the resulting coordinate order is appropriate for the intended CRS.

### Fix Code Hint
```python
# Ensure input is cast to a geometry if provided as WKT
sd.sql("SELECT ST_FlipCoordinates(ST_GeomFromText('POINT(1 2)'))")
```

## API Test: `sql:ST_Force2D`

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

## Description

Discards any Z and M values present (if any).

## Examples

```sql
SELECT ST_Force2D(ST_GeomFromWKT('POINT Z (1 2 3)'));
```

```sql
SELECT ST_Force2D(ST_GeogFromWKT('POINT Z (1 2 3)'));
```


### Goal
`sql:ST_Force2D` converts a 3D or 4D geometry or geography object into a 2D (XY) coordinate model by stripping away any existing Z (elevation) or M (measure) dimensions.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object containing Z or M coordinates that need to be removed.

### Input
- **Data:** Standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If the input is already 2D, the function returns the geometry unchanged.

### Output
Returns `geography | geometry` — A new spatial object containing only the X and Y coordinates of the original input.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Force2D(ST_GeomFromWKT('POINT Z (1 2 3)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Force2D(ST_GeogFromWKT('POINT Z (1 2 3)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Force2D` when you need to normalize spatial data to 2D for operations that do not support Z/M dimensions.
- Ensure the input is a valid geometry or geography object created via `ST_GeomFromWKT` or similar constructors.

### Prompt Snippet
```text
SELECT ST_Force2D(arg1)
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) directly without using a constructor like `ST_GeomFromWKT`.
- Attempting to use the function on unsupported data types outside of `geometry` or `geography`.

### Fix Code Hint
```python
# Ensure input is cast to a geometry type first
sd.sql("SELECT ST_Force2D(ST_GeomFromWKT(wkt_string))")
```

__CHECK__

## API Test: `sql:ST_Force3D`

### Signature
```python
ST_Force3D(arg1: geometry) -> geometry | ST_Force3D(arg1: geometry, z: double) -> geometry | ST_Force3D(arg1: geography) -> geography | ST_Force3D(arg1: geography, z: double) -> geography
```
_Source: source/docs/reference/sql/st_force3d.qmd:19_

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

title: ST_Force3D
description: Forces a geometry or geography into a XYZ coordinate model with an optional Z value.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args:
    - geometry
    - name: z
      type: double
  - returns: geography
    args: [geography]
  - returns: geography
    args:
    - geography
    - name: z
      type: double
---

## Description

If the geometry already has Z values they are preserved; otherwise the optional `z` argument (default 0) is used.

Aliases: `ST_Force3DZ`.

## Examples

```sql
SELECT ST_Force3D(ST_GeomFromWKT('POINT (1 2)'), 10.0);
```

```sql
SELECT ST_Force3D(ST_GeomFromWKT('POINT (1 2)'));
```

```sql
SELECT ST_Force3D(ST_GeogFromWKT('POINT (1 2)'), 10.0);
```


### Goal
Forces a `geometry` or `geography` object into a 3D (XYZ) coordinate model by adding a Z-coordinate if one is not already present.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be converted to 3D.
- `z` (`double`): The optional Z-coordinate value to assign if the input lacks Z-dimensions; defaults to 0.0.

### Input
- Accepts `geometry` (planar) or `geography` (spherical) types.
- Input objects can be constructed via standard OGC WKT/WKB methods (e.g., `ST_GeomFromWKT`, `ST_GeogFromWKT`).
- If the input already contains Z-values, they are preserved.

### Output
Returns a `geometry` or `geography` object (matching the input type) that is guaranteed to have a Z-coordinate component.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Force3D(ST_GeomFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Force3D(ST_GeogFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Force3D` when a 3D coordinate model is required for downstream spatial analysis.
- If the input is `geography`, ensure the session has the `s2geography` bounder registered.
- Use the alias `ST_Force3DZ` if preferred for compatibility with other spatial database standards.

### Prompt Snippet
```text
ST_Force3D(geometry_or_geography_column, [z_value])
```

### Common Failure Modes
- **Type Mismatch:** Passing an unsupported type that is not a valid `geometry` or `geography` object.
- **Geography Bounder:** Failure to register the `s2geography` bounder on the session when performing operations on `geography` types.
- **Null Inputs:** Passing a `NULL` geometry/geography will result in a `NULL` output.

### Fix Code Hint
```python
# Ensure geometry is valid before forcing 3D
sd.sql("SELECT ST_Force3D(ST_MakeValid(geom), 5.0) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_Force3DM`

### Signature
```python
ST_Force3DM(arg1: geometry) -> geometry | ST_Force3DM(arg1: geometry, m: double) -> geometry | ST_Force3DM(arg1: geography) -> geography | ST_Force3DM(arg1: geography, m: double) -> geography
```
_Source: source/docs/reference/sql/st_force3dm.qmd:19_

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

title: ST_Force3DM
description: Forces a geometry or geography into a XYM coordinate model with an optional M value.
kernels:
  - returns: geometry
    args: [geometry]
  - returns: geometry
    args:
    - geometry
    - name: m
      type: double
  - returns: geography
    args: [geography]
  - returns: geography
    args:
    - geography
    - name: m
      type: double
---

## Description

If the geometry already has M values they are preserved; otherwise the optional `m` argument (default 0) is used.

## Examples

```sql
SELECT ST_Force3DM(ST_GeomFromWKT('POINT (1 2)'), 10.0);
```

```sql
SELECT ST_Force3DM(ST_GeomFromWKT('POINT (1 2)'));
```

```sql
SELECT ST_Force3DM(ST_GeogFromWKT('POINT (1 2)'), 10.0);
```


### Goal
Forces a `geometry` or `geography` object into a XYM coordinate model, assigning a specified M value or defaulting to 0 if none is provided.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be converted to the XYM coordinate model.
- `m` (`double`): The optional M value to assign to the coordinates; defaults to 0.0 if not specified.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Standard OGC WKT/WKB inputs are supported via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`.
- **Preconditions:** If the input already contains M values, they are preserved; otherwise, the function adds the M dimension.

### Output
Returns `geography | geometry` — A new geometry or geography object containing the original XY coordinates and the specified (or default) M value.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Force3DM(ST_GeomFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Force3DM(ST_GeogFromWKT('POINT (1 2)'), 10.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Force3DM` to ensure a geometry or geography has an M dimension.
- If the M value is critical, explicitly provide the `m` double argument.
- Ensure the input is a valid `geometry` or `geography` type before calling.

### Prompt Snippet
```text
Use ST_Force3DM(geometry, m) to force a geometry into XYM format.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-geometry/geography type as `arg1`.
- **Missing M-value:** If the user expects a specific M value but omits the argument, the function defaults to 0.0.
- **Geography Constraints:** Geography operations require the `s2geography` bounder to be registered on the session.

### Fix Code Hint
```python
# Ensure input is cast to geometry/geography if coming from raw WKT
sd.sql("SELECT ST_Force3DM(ST_GeomFromWKT(wkt_col), 5.0) FROM my_table")
```__CHECK__

## API Test: `sql:ST_Force4D`

### Signature
```python
ST_Force4D(arg1: geometry) -> geometry | ST_Force4D(arg1: geometry, z: double) -> geometry | ST_Force4D(arg1: geometry, z: double, m: double) -> geometry | ST_Force4D(arg1: geography) -> geography | ST_Force4D(arg1: geography, z: double) -> geography | ST_Force4D(arg1: geography, z: double, m: double) -> geography
```
_Source: source/docs/reference/sql/st_force4d.qmd:19_

### Goal
Forces a `geometry` or `geography` object into a 4D XYZM coordinate model, applying specified Z and M values if they are not already present.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be converted to 4D.
- `z` (`double`): The Z-coordinate value to assign if the input lacks a Z dimension; defaults to 0.0.
- `m` (`double`): The M-coordinate value to assign if the input lacks an M dimension; defaults to 0.0.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Supports standard OGC WKT/WKB inputs.
- **Preconditions:** If the input already contains Z or M values, those values are preserved; the provided `z` and `m` arguments are only applied to missing dimensions.

### Output
Returns a `geometry` or `geography` object (matching the input type) that is guaranteed to have XYZM coordinates.

### Valid Call Patterns
```python
# Example using geometry with explicit Z and M
sd.sql("SELECT ST_Force4D(ST_GeomFromWKT('POINT (1 2)'), 10.0, 20.0)").to_arrow_table()

# Example using geography with explicit Z and M
sd.sql("SELECT ST_Force4D(ST_GeogFromWKT('POINT ZM (1 2 3 4)'), 10.0, 20.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Force4D` to ensure spatial objects conform to a 4D coordinate model.
- If the input is a `geography` type, ensure the session has the `s2geography` bounder registered.
- Do not provide Z or M arguments if you intend to use the default value of 0.0.

### Prompt Snippet
```text
Use ST_Force4D(arg1, z, m) to force a geometry or geography into XYZM. If the input already has Z/M, they are preserved.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a function expecting `geography` or vice versa may cause runtime errors depending on the specific SQL context.
- **Missing Bounder:** Geography operations require the `s2geography` bounder to be registered on the session; failure to do so will result in an error.
- **Invalid WKT:** Providing malformed WKT strings to `ST_GeomFromWKT` or `ST_GeogFromWKT` before passing them to `ST_Force4D` will cause the query to fail.

### Fix Code Hint
```python
# Ensure the input is valid before forcing dimensions
sd.sql("SELECT ST_Force4D(ST_MakeValid(geom), 0.0, 0.0) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_GeoHash`

### Signature
```python
ST_GeoHash(arg1: geometry) -> string | ST_GeoHash(arg1: geometry, precision: integer) -> string | ST_GeoHash(arg1: geography) -> string | ST_GeoHash(arg1: geography, precision: integer) -> string
```
_Source: source/docs/reference/sql/st_geohash.qmd:19_

### Goal
Encodes a `geometry` or `geography` object into a base-32 GeoHash string representation, using the center of the bounding box for non-point geometries.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to encode. Must be in WGS84 (`EPSG:4326` / `OGC:CRS84`) or have no CRS defined.
- `precision` (`integer`): The number of base-32 characters in the resulting GeoHash. Capped at 20; non-positive values return an empty string.

### Input
- **Data:** `geometry` or `geography` objects (WKT/WKB).
- **Preconditions:** 
    - The input must use WGS84 CRS or no CRS. Use `ST_Transform()` to reproject non-WGS84 data before calling.
    - If `precision` is omitted, `arg1` must be a `POINT`.
    - Geography inputs require the `s2geography` bounder to be registered on the session.
    - Latitudes must be within [-90, 90]; otherwise, the function returns `NULL`.

### Output
Returns `string` — A base-32 GeoHash string representing the spatial location.

### Valid Call Patterns
```python
# Encode a point with specific precision
sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POINT (21.4234 52.0423)'), 10)").to_arrow_table()

# Encode a polygon (uses center of bounding box)
sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'), 8)").to_arrow_table()

# Encode a point with default maximum precision (20)
sd.sql("SELECT ST_GeoHash(ST_GeomFromWKT('POINT (21.4234 52.0423)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always verify the input CRS is WGS84 or undefined before calling `ST_GeoHash`.
- If the input is not a `POINT` and `precision` is omitted, the query will fail; ensure `precision` is provided for non-point geometries.
- For mixed-CRS columns, filter rows using `WHERE ST_SRID(geom) = 4326` before applying the function.

### Prompt Snippet
```text
ST_GeoHash(geometry_column, precision_integer)
```

### Common Failure Modes
- **CRS Mismatch:** Passing a geometry with a non-WGS84 CRS (e.g., NAD83) raises an error.
- **Invalid Precision:** Omitting `precision` for non-point geometries (e.g., `POLYGON`) raises an error.
- **Out of Bounds:** Latitudes outside [-90, 90] return `NULL`.
- **Missing Bounder:** Using `geography` types without the `s2geography` bounder registered.

### Fix Code Hint
```python
# Reproject to WGS84 before encoding
query = """
SELECT ST_GeoHash(ST_Transform(geom, 4326), 10) 
FROM my_table 
WHERE ST_SRID(geom) != 4326
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_GeogFromWKB`

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
Constructs a spherical `geography` object from a Well-Known Binary (WKB) representation.

### Parameters
- `wkb` (`binary`): A binary byte string representing the geometry in OGC Well-Known Binary (WKB) format.

### Input
The input must be a valid `binary` type containing WKB data. Preconditions include ensuring the binary data is correctly formatted according to OGC standards. Geography operations require the `s2geography` bounder to be registered on the session.

### Output
Returns a `geography` object, which represents the spatial data in a spherical coordinate system, suitable for geodesic calculations.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using a hex-encoded WKB literal
query = "SELECT ST_GeogFromWKB(x'010200000002000000000000000084d600c0000000000080b5d6bf00000060e1eff7bf00000080075de5bf') as geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_GeogFromWKB` when converting binary WKB data into spherical `geography` types.
- Ensure the input is provided as a `binary` literal (e.g., `x'...'` in SQL).
- Verify that the session has the `s2geography` bounder registered before executing geography-based queries.

### Prompt Snippet
```text
Use ST_GeogFromWKB(wkb_binary) to create a geography object from WKB.
```

### Common Failure Modes
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered on the session.
- **Invalid WKB:** Providing malformed or truncated binary data will result in a parsing error.
- **Type Mismatch:** Attempting to use `geography` functions on `geometry` types (or vice versa) without explicit conversion.

### Fix Code Hint
```python
# Ensure the binary string is correctly formatted as a hex literal
wkb_hex = "010200000002000000000000000084d600c0000000000080b5d6bf00000060e1eff7bf00000080075de5bf"
query = f"SELECT ST_GeogFromWKB(x'{wkb_hex}')"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_GeogFromWKT`

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
Constructs a spherical `geography` object from a Well-Known Text (WKT) string representation.

### Parameters
- `wkt` (`string`): The Well-Known Text representation of the spatial object (e.g., 'POINT(0 0)').
- `srid` (`integer`): The Spatial Reference System Identifier to associate with the geography.

### Input
The input must be a valid OGC WKT string. Precondition: The session must have the `s2geography` bounder registered to perform geography-based operations.

### Output
Returns a `geography` object representing the spherical spatial data.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_GeogFromWKT('LINESTRING (1 2, 3 4, 5 6)')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_GeogFromWKT('POINT (-74.006 40.712)', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_GeogFromWKT` when the analysis requires spherical calculations (e.g., distance in meters, area on a sphere) rather than planar `geometry` calculations.
- Always ensure the WKT string is properly formatted according to OGC standards.
- If an SRID is known, provide it as the second argument to ensure correct coordinate reference system handling.

### Prompt Snippet
```text
Use ST_GeogFromWKT(wkt_string) or ST_GeogFromWKT(wkt_string, srid) to create a geography object.
```

### Common Failure Modes
- **Missing Bounder:** Failure to register the `s2geography` bounder on the session will cause geography operations to fail.
- **Invalid WKT:** Providing a malformed WKT string will result in a parsing error.
- **Type Mismatch:** Attempting to use `geography` functions on `geometry` types (or vice versa) without explicit conversion.

### Fix Code Hint
```python
# Ensure the geography bounder is active and WKT is valid
result = sd.sql("SELECT ST_GeogFromWKT('POINT(1 2)')").to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_GeogPoint`

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
Creates a spherical `geography` POINT object from specified longitude and latitude coordinates for use in spherical spatial analysis.

### Parameters
- `longitude` (`double`): The longitudinal coordinate in decimal degrees.
- `latitude` (`double`): The latitudinal coordinate in decimal degrees.

### Input
Requires two `double` precision floating-point values representing coordinates. Precondition: The session must have the `s2geography` bounder registered to support `geography` type operations.

### Output
Returns a `geography` object representing a point on the sphere.

### Valid Call Patterns
```python
# Using the Python SQL connection
sd.sql("SELECT ST_GeogPoint(-64.0, 45.0) AS geom").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_GeogPoint(longitude, latitude)` as a standalone SQL function call.
- Ensure inputs are provided as `double` types.
- Remember that this function returns a `geography` type, which is distinct from the planar `geometry` type.

### Prompt Snippet
```text
Use ST_GeogPoint(longitude, latitude) to construct a geography point.
```

### Common Failure Modes
- Passing non-numeric types (e.g., strings) instead of `double` will cause a type mismatch error.
- Attempting to use the resulting `geography` object in functions that strictly require planar `geometry` (e.g., certain `ST_Buffer` variants) may result in execution errors.
- Failure to register the `s2geography` bounder in the session will prevent the creation or processing of `geography` types.

### Fix Code Hint
```python
# Ensure coordinates are cast to double if coming from other sources
sd.sql("SELECT ST_GeogPoint(CAST(lon AS DOUBLE), CAST(lat AS DOUBLE)) FROM points_table").to_arrow_table()
```

## API Test: `sql:ST_GeomCollFromText`

### Signature
```python
ST_GeomCollFromText(wkt: string) -> geometry | ST_GeomCollFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomcollfromtext.qmd:19_

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

title: ST_GeomCollFromText
description: Constructs a GeometryCollection from Well-Known Text (WKT), erroring if the input is not a GeometryCollection.
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

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a GeometryCollection.

## Examples

```sql
SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))');
```

With an SRID or CRS:

```sql
SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))', 4326);
```


### Goal
Constructs a `geometry` object from a Well-Known Text (WKT) string specifically representing a `GEOMETRYCOLLECTION`.

### Parameters
- `wkt` (`string`): A Well-Known Text string that must define a `GEOMETRYCOLLECTION`.
- `srid` (`crs`): An optional integer or string representing the Coordinate Reference System (e.g., `4326` or `'EPSG:4326'`).

### Input
The input must be a valid WKT string. Precondition: The WKT must explicitly represent a `GEOMETRYCOLLECTION`; otherwise, the operation will raise an error.

### Output
Returns a `geometry` object representing the collection of geometries defined in the WKT.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_GeomCollFromText('GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_GeomCollFromText` only when the input WKT is guaranteed to be a `GEOMETRYCOLLECTION`.
- If an SRID is required for spatial operations, provide it as the second argument.
- Always wrap the SQL call in `sd.sql(...).to_arrow_table()` for Python execution.

### Prompt Snippet
```text
Use ST_GeomCollFromText(wkt) or ST_GeomCollFromText(wkt, srid) to create a GeometryCollection.
```

### Common Failure Modes
- **Invalid WKT Type:** Passing a WKT string that is not a `GEOMETRYCOLLECTION` (e.g., `POINT`, `POLYGON`) will cause the function to error.
- **Missing SRID:** If the downstream spatial analysis requires a specific CRS, failing to provide the `srid` parameter may lead to CRS mismatch errors in subsequent operations.

### Fix Code Hint
```python
# Ensure the WKT string starts with 'GEOMETRYCOLLECTION'
wkt_input = 'GEOMETRYCOLLECTION (POINT (0 0), LINESTRING (0 0, 1 1))'
query = f"SELECT ST_GeomCollFromText('{wkt_input}', 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_GeomFromEWKB`

### Signature
```python
ST_GeomFromEWKB(ewkb: binary) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromewkb.qmd:19_

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

title: ST_GeomFromEWKB
description: Constructs a geometry from Extended Well-Known Binary (EWKB).
kernels:
  - returns: geometry
    args:
    - name: ewkb
      type: binary
---

## Description

Parses an EWKB binary value and returns a geometry. EWKB extends the WKB format to include SRID information in the binary header and is useful for PostGIS compatibility.

## Examples

```sql
SELECT ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040');
```


### Goal
Constructs a `geometry` object from an Extended Well-Known Binary (EWKB) representation, enabling the integration of PostGIS-compatible binary data into SedonaDB workflows.

### Parameters
- `ewkb` (`binary`): A binary string representing the geometry in EWKB format, which includes spatial reference system (SRID) information in the header.

### Input
The input must be a valid `binary` type containing an EWKB-encoded geometry. In SQL, this is typically provided using the hex literal syntax (e.g., `x'...'`).

### Output
Returns a `geometry` object that can be used in subsequent spatial operations such as `ST_Intersects`, `ST_Buffer`, or `ST_Area`.

### Valid Call Patterns
```python
# Python usage via SedonaDB SQL connection
result = sd.sql("SELECT ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
When calling `ST_GeomFromEWKB`, ensure the input is a valid binary representation of a geometry. If the input is a hex string, prefix it with `x` (e.g., `x'01...'`) to ensure it is interpreted as `binary` by the SQL engine.

### Prompt Snippet
```text
SELECT ST_GeomFromEWKB(x'0101000000000000000000f03f0000000000000040');
```

### Common Failure Modes
- **Invalid Binary Format:** Providing a string that is not a valid EWKB byte sequence will result in a parsing error.
- **Missing Hex Prefix:** Passing a raw string without the `x` prefix in SQL may cause type mismatch errors if the engine expects a binary literal.
- **Unsupported EWKB Version:** While SedonaDB supports standard EWKB, highly non-standard or corrupted binary headers may fail to materialize.

### Fix Code Hint
```python
# Ensure binary literals are correctly formatted for the SQL engine
ewkb_hex = "0101000000000000000000f03f0000000000000040"
query = f"SELECT ST_GeomFromEWKB(x'{ewkb_hex}')"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_GeomFromEWKT`

### Signature
```python
ST_GeomFromEWKT(ewkt: string) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromewkt.qmd:19_

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

title: ST_GeomFromEWKT
description: Constructs a geometry from Extended Well-Known Text (EWKT).
kernels:
  - returns: geometry
    args:
    - name: ewkt
      type: string
---

## Description

Parses an EWKT string in the format `SRID=<srid>;<wkt>` and returns a geometry with the embedded SRID set. EWKT is used by PostGIS and other systems for human-readable output when an item-level SRID required.

## Examples

```sql
SELECT ST_GeomFromEWKT('SRID=4326;POINT(40.7128 -74.0060)');
```


### Goal
Constructs a `geometry` object from an Extended Well-Known Text (EWKT) string, allowing for the explicit definition of an SRID within the input string.

### Parameters
- `ewkt` (`string`): A string formatted as `SRID=<srid>;<wkt>`, where `<srid>` is the integer spatial reference identifier and `<wkt>` is the standard Well-Known Text representation of the geometry.

### Input
The input must be a valid EWKT string. The function expects the `SRID=<srid>;` prefix to correctly parse and assign the coordinate reference system metadata to the resulting geometry object.

### Output
Returns a `geometry` object with the specified SRID embedded as metadata.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using the documented SQL pattern
result = sd.sql("SELECT ST_GeomFromEWKT('SRID=4326;POINT(40.7128 -74.0060)')").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use the `ST_GeomFromEWKT(ewkt)` function signature when parsing geometries that require an explicit SRID.
- Ensure the input string strictly follows the `SRID=<srid>;<wkt>` format.

### Prompt Snippet
```text
Use ST_GeomFromEWKT('SRID=<srid>;<wkt>') to create a geometry with a specific SRID.
```

### Common Failure Modes
- **Malformed String:** Providing a WKT string without the `SRID=<srid>;` prefix or with incorrect syntax will cause a parsing error.
- **Invalid SRID:** Providing an unsupported or non-integer SRID value may result in runtime errors during geometry construction.

### Fix Code Hint
```python
# Ensure the string is properly formatted
ewkt_input = "SRID=4326;POINT(40.7128 -74.0060)"
query = f"SELECT ST_GeomFromEWKT('{ewkt_input}')"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_GeomFromWKB`

### Signature
```python
ST_GeomFromWKB(wkb: binary) -> geometry | ST_GeomFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromwkb.qmd:19_

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

title: ST_GeomFromWKB
description: Constructs a Geometry from Well-Known Binary (WKB).
kernels:
  - returns: geometry
    args:
    - name: wkb
      type: binary
  - returns: geometry
    args:
    - name: wkb
      type: binary
    - name: srid
      type: crs
---

## Description

Parses a WKB binary value and returns a geometry. The input is validated by
default. An optional SRID or CRS can be provided as a second argument.

## Examples

```sql
SELECT ST_GeomFromWKB(
  decode('0101000000000000000000F03F0000000000000040', 'hex')
);
```

```sql
SELECT ST_GeomFromWKB(
  decode('0101000000000000000000F03F0000000000000040', 'hex'),
  4326
);
```

```sql
SELECT ST_GeomFromWKB(
  decode('0101000000000000000000F03F0000000000000040', 'hex'),
  'OGC:CRS27'
);
```


### Goal
Constructs a SedonaDB `geometry` object from a Well-Known Binary (WKB) representation.

### Parameters
- `wkb` (`binary`): The binary representation of the geometry (WKB).
- `srid` (`crs`): An optional Coordinate Reference System identifier, provided as an integer (e.g., `4326`) or a string (e.g., `'OGC:CRS27'`).

### Input
The caller must provide a valid WKB binary sequence. If the input is a hex-encoded string, it must be converted to binary using the `decode(string, 'hex')` function before passing it to `ST_GeomFromWKB`.

### Output
Returns a `geometry` object representing the spatial feature defined by the input WKB.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT ST_GeomFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `decode(..., 'hex')` if the input WKB is provided as a hex string.
- If an SRID is known, provide it as the second argument to ensure correct CRS tracking.
- The function returns a `geometry` type; ensure subsequent operations are compatible with planar geometry.

### Prompt Snippet
```text
ST_GeomFromWKB(decode('...', 'hex'), [srid])
```

### Common Failure Modes
- Providing a malformed WKB string that cannot be parsed.
- Forgetting to decode a hex-encoded string into binary, which will cause a type mismatch error.
- Providing an invalid or unsupported CRS string.

### Fix Code Hint
```python
# Ensure hex strings are decoded to binary
query = "SELECT ST_GeomFromWKB(decode('0101...', 'hex'), 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_GeomFromWKT`

### Signature
```python
ST_GeomFromWKT(wkt: string) -> geometry | ST_GeomFromWKT(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_geomfromwkt.qmd:19_

### Goal
Constructs a `geometry` object from a Well-Known Text (WKT) string representation, optionally assigning a spatial reference system.

### Parameters
- `wkt` (`string`): A standard OGC Well-Known Text string representing a geometry (e.g., 'POINT (30 10)').
- `srid` (`crs`): An optional spatial reference identifier (integer) or CRS authority string (e.g., 4326 or 'OGC:CRS27') to associate with the geometry.

### Input
The function accepts a valid WKT string. If an SRID is provided, it must be a valid integer or a recognized CRS authority string. No prior conversion is required, but the WKT must be syntactically correct according to OGC standards.

### Output
Returns a `geometry` object representing the spatial feature defined by the input WKT, with the specified CRS metadata if provided.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_GeomFromWKT('POINT (30 10)')").to_arrow_table()

# Construction with integer SRID
sd.sql("SELECT ST_GeomFromWKT('POINT (30 10)', 4326)").to_arrow_table()

# Construction with CRS authority string
sd.sql("SELECT ST_GeomFromWKT('POINT (30 10)', 'OGC:CRS27')").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_GeomFromWKT` to instantiate geometries from WKT strings.
- Use the aliases `ST_GeomFromText` or `ST_GeometryFromText` if required by existing SQL patterns.
- Ensure the WKT string is properly quoted within the SQL query string.
- If a spatial reference is known, always provide the `srid` argument to ensure correct CRS propagation in subsequent spatial operations.

### Prompt Snippet
```text
Use ST_GeomFromWKT(wkt_string) or ST_GeomFromWKT(wkt_string, srid) to create geometry objects.
```

### Common Failure Modes
- **Invalid WKT:** Providing a malformed WKT string will result in a runtime execution error.
- **CRS Mismatch:** If performing spatial operations (e.g., `ST_Intersects`) between geometries with different SRIDs, the operation may fail or produce unexpected results if the system cannot reconcile the coordinate systems.
- **Missing Quotes:** Forgetting to wrap the WKT string in single quotes within the SQL query will cause a syntax error.

### Fix Code Hint
```python
# Ensure WKT is a valid string and SRID is a supported format
wkt_input = "POINT (30 10)"
srid_input = 4326
query = f"SELECT ST_GeomFromWKT('{wkt_input}', {srid_input})"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_GeometryN`

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
Extracts the Nth component geometry from a collection-type geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The input collection object (e.g., `GEOMETRYCOLLECTION`, `MULTIPOINT`, `MULTILINESTRING`, `MULTICURVE`, or `MULTIPOLYGON`).
- `n` (`integer`): The 1-based index of the geometry to retrieve from the collection.

### Input
The input must be a valid OGC-compliant collection geometry or geography. The caller must ensure the input is not a simple geometry (e.g., `POINT`, `LINESTRING`) if the intent is to extract a sub-element, as these do not contain multiple geometries.

### Output
Returns a `geometry` or `geography` object corresponding to the Nth element of the input collection, or `NULL` if the index `n` is out of range.

### Valid Call Patterns
```python
# Extract the first point from a MULTIPOINT collection
sd.sql("SELECT ST_GeometryN(ST_GeomFromText('MULTIPOINT ((1 2), (3 4), (5 6))'), 1)").to_arrow_table()

# Extract the first point from a geography MULTIPOINT collection
sd.sql("SELECT ST_GeometryN(ST_GeogFromWKT('MULTIPOINT ((1 2), (3 4), (5 6))'), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for the `n` parameter.
- Ensure the input `arg1` is a collection type; if the input is a simple geometry, the function may return `NULL` or unexpected results.
- Use `ST_GeomFromText` for `geometry` types and `ST_GeogFromWKT` for `geography` types to ensure correct type dispatching.

### Prompt Snippet
```text
ST_GeometryN(arg1, n)
```

### Common Failure Modes
- **Index Out of Range:** Providing an `n` value less than 1 or greater than the number of geometries in the collection returns `NULL`.
- **Type Mismatch:** Passing a simple geometry (e.g., `POINT`) instead of a collection type.
- **Geography Bounder:** Geography operations require the `s2geography` bounder to be registered on the session; failure to do so may cause runtime errors.

### Fix Code Hint
```python
# Verify the number of geometries before extraction if index is dynamic
# Ensure the input is a collection type
result = sd.sql("SELECT ST_GeometryN(geom, 1) FROM my_table WHERE ST_NumGeometries(geom) >= 1").to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_GeometryType`

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
Returns the OGC-compliant geometry type name for a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object whose type is to be inspected.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) objects.
- **Formats:** Input objects are typically constructed via `ST_GeomFromWKT`, `ST_GeogFromWKT`, or other geometry-returning functions.

### Output
Returns `string` — The OGC geometry type name prefixed with `ST_` (e.g., `'ST_Point'`, `'ST_Polygon'`).

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_GeometryType(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_GeometryType(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 0 1, 0 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_GeometryType(arg1)` as a standalone SQL function call within `sd.sql()`.
- Ensure the input `arg1` is a valid spatial object (geometry or geography).

### Prompt Snippet
```text
Use ST_GeometryType(geometry_column) to retrieve the OGC type string for spatial data.
```

### Common Failure Modes
- Passing a non-spatial type (e.g., `string` or `integer`) directly without using a constructor like `ST_GeomFromWKT`.
- Expecting a specific OGC type name that does not include the `ST_` prefix (the function explicitly returns the prefixed version).

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a WKT string
sd.sql("SELECT ST_GeometryType(ST_GeomFromWKT(wkt_column)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_HasM`

### Signature
```python
ST_HasM(arg1: geometry) -> boolean | ST_HasM(arg1: geography) -> boolean
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
Determines whether a given `geometry` or `geography` object contains a Measure (M) dimension.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect for the presence of an M coordinate.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If the input is a collection, the function evaluates the presence of the M dimension within the object structure.

### Output
Returns `boolean` — `true` if the geometry or geography has an M dimension, `false` otherwise.

### Valid Call Patterns
```python
# Verify M dimension presence in a WKT geometry
result = sd.sql("SELECT ST_HasM(ST_GeomFromWKT('POINT M (1 2 3)'))").to_arrow_table()
print(result)
# __CHECK__ witness: result should contain True

# Verify absence of M dimension
result = sd.sql("SELECT ST_HasM(ST_GeomFromWKT('POINT (1 2)'))").to_arrow_table()
print(result)
# __CHECK__ witness: result should contain False
```

### LLM Instruction Prompt
- Always use `ST_HasM(arg1)` as a standalone SQL function call within `sd.sql()`.
- Ensure the input is a valid spatial object (e.g., created via `ST_GeomFromWKT` or `ST_Point`).
- Do not attempt to pass non-spatial types as the argument.

### Prompt Snippet
```text
SELECT ST_HasM(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., `string` or `integer`) directly to `ST_HasM` will result in a query execution error.
- **Null Input:** If the input geometry is `NULL`, the function will return `NULL` rather than a boolean.
- **Dimension Confusion:** Confusing the M dimension with the Z dimension; `ST_HasM` specifically checks for the Measure coordinate, not the elevation (Z) coordinate.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_HasM(ST_GeomFromWKT(wkt_column)) FROM my_table")
```

## API Test: `sql:ST_HasZ`

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
Determines whether a given `geometry` or `geography` object contains a Z-coordinate (3D) dimension.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect for the presence of a Z dimension.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If the input is a collection, the function evaluates the presence of the Z dimension based on the object's internal structure.

### Output
Returns `boolean` — `true` if the object has a Z dimension, `false` otherwise.

### Valid Call Patterns
```python
# Verify Z dimension for a 3D point
result = sd.sql("SELECT ST_HasZ(ST_GeomFromWKT('POINT Z (1 2 3)'))").to_arrow_table()
assert result[0][0] == True
print("__CHECK__")

# Verify Z dimension for a 2D point
result = sd.sql("SELECT ST_HasZ(ST_GeomFromWKT('POINT (1 2)'))").to_arrow_table()
assert result[0][0] == False
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_HasZ(arg1)` as a standalone SQL function call.
- Ensure the input is wrapped in a geometry constructor like `ST_GeomFromWKT` if passing raw WKT strings.
- The function is polymorphic and accepts both `geometry` and `geography` types.

### Prompt Snippet
```text
Use ST_HasZ(geometry_expression) to check for 3D coordinates.
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-spatial type (e.g., `string` or `integer`) directly without a constructor will result in a SQL execution error.
- **Null Input:** If the input geometry is `NULL`, the function will return `NULL` rather than a boolean.
- **Dimension Confusion:** Confusing Z (elevation) with M (measure) dimensions; `ST_HasZ` specifically checks for the Z dimension.

### Fix Code Hint
```python
# Ensure input is cast to geometry if it is a raw string
sd.sql("SELECT ST_HasZ(ST_GeomFromWKT(my_wkt_column)) FROM my_table")
```

## API Test: `sql:ST_HausdorffDistance`

### Signature
```python
ST_HausdorffDistance(arg1: geometry, arg2: geometry) -> double | ST_HausdorffDistance(arg1: geometry, arg2: geometry, densifyFrac: double) -> double
```
_Source: source/docs/reference/sql/st_hausdorffdistance.qmd:19_

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

title: ST_HausdorffDistance
description: Returns the Hausdorff distance between two geometries.
kernels:
  - returns: double
    args: [geometry, geometry]
  - returns: double
    args:
    - geometry
    - geometry
    - name: densifyFrac
      type: double
      description: >
        A fraction by which to densify each segment of the geometries before computing
        the distance. A value of 0.25 will densify the segment by adding vertices every
        1/4 of the segment length. Densification can give a more accurate result for
        geometries with long segments or complex shapes.
---

## Description

The Hausdorff distance is a measure of the similarity between two geometric shapes. This implementation returns the greatest of all distances from a point in one geometry to the closest point in the other geometry.

Returns `NULL` if either geometry is `NULL`. Returns 0 if either geometry is empty.

## Examples

Basic Hausdorff distance between two points:

```sql
SELECT ST_HausdorffDistance(ST_Point(0, 0), ST_Point(3, 4));
```

Hausdorff distance between two linestrings:

```sql
SELECT ST_HausdorffDistance(
    ST_GeomFromText('LINESTRING (0 0, 2 0)'),
    ST_GeomFromText('LINESTRING (0 1, 2 1)')
);
```

Using densification for more accurate results as needed:

```sql
SELECT ST_HausdorffDistance(
    ST_GeomFromText('LINESTRING (130 0, 0 0, 0 150)'),
    ST_GeomFromText('LINESTRING (10 10, 10 150, 130 10)'),
    0.5
);
```


### Goal
Computes the Hausdorff distance between two geometries to quantify their geometric similarity by finding the maximum distance from any point in one geometry to the nearest point in the other.

### Parameters
- `arg1` (`geometry`): The first input geometry (e.g., Point, LineString, Polygon).
- `arg2` (`geometry`): The second input geometry to compare against the first.
- `densifyFrac` (`double`): Optional fraction used to densify segments before calculation; a value of 0.25 adds vertices every 1/4 of the segment length to improve accuracy for complex shapes.

### Input
Requires two valid `geometry` objects. Geometries can be constructed via `ST_GeomFromText` or other geometry-returning functions. Ensure geometries are in the same coordinate system if spatial accuracy is required.

### Output
Returns `double` — The calculated Hausdorff distance value. Returns `NULL` if either input is `NULL`, and `0` if either input is empty.

### Valid Call Patterns
```python
# Basic usage
sd.sql("SELECT ST_HausdorffDistance(ST_Point(0, 0), ST_Point(3, 4))").to_arrow_table()

# Usage with densification
sd.sql("SELECT ST_HausdorffDistance(ST_GeomFromText('LINESTRING (0 0, 2 0)'), ST_GeomFromText('LINESTRING (0 1, 2 1)'), 0.5)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql("...").to_arrow_table()` for execution.
- Use `ST_GeomFromText` to wrap WKT strings if passing raw geometry data.
- If the geometry is complex or has long segments, include the `densifyFrac` parameter to ensure accuracy.

### Prompt Snippet
```text
SELECT ST_HausdorffDistance(geom1, geom2)
-- or
SELECT ST_HausdorffDistance(geom1, geom2, densifyFrac)
```

### Common Failure Modes
- Passing `NULL` values results in a `NULL` output.
- Mismatched coordinate systems may lead to unexpected distance results if the geometries are not in the same CRS.
- Providing a non-numeric value for `densifyFrac` will cause a type error.

### Fix Code Hint
```python
# Ensure geometries are valid and non-null before calling
sd.sql("""
    SELECT ST_HausdorffDistance(g1, g2, 0.25) 
    FROM (SELECT ST_GeomFromText('...') as g1, ST_GeomFromText('...') as g2)
""").to_arrow_table()
```

## API Test: `sql:ST_InteriorRingN`

### Signature
```python
ST_InteriorRingN(arg1: geometry, n: integer) -> geometry | ST_InteriorRingN(arg1: geography, n: integer) -> geography
```
_Source: source/docs/reference/sql/st_interiorringn.qmd:19_

### Goal
Extracts the Nth interior `LINESTRING` ring from a `POLYGON` geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The input polygon from which to extract an interior ring.
- `n` (`integer`): The 1-based index of the interior ring to retrieve.

### Input
- **Data:** A `POLYGON` geometry or geography object.
- **Preconditions:** The input must be a valid `POLYGON`. If the input is not a polygon, or if `n` is out of range (e.g., `n` is greater than the number of interior rings), the function returns `NULL`.
- **Indexing:** `n` uses 1-based indexing, where `1` corresponds to the first interior ring.

### Output
Returns `geography | geometry` — A `LINESTRING` representing the Nth interior ring of the input polygon.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_InteriorRingN(ST_GeomFromText('POLYGON((0 0, 0 5, 5 5, 5 0, 0 0), (1 1, 2 1, 2 2, 1 2, 1 1))'), 1)").to_arrow_table()

# Geography example
sd.sql("SELECT ST_InteriorRingN(ST_GeogFromText('POLYGON((0 0, 0 5, 5 5, 5 0, 0 0), (1 1, 2 1, 2 2, 1 2, 1 1))'), 1)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use 1-based indexing for the `n` parameter.
- Ensure the input `arg1` is a `POLYGON` type; otherwise, the operation will return `NULL`.
- Match the input type (`geometry` or `geography`) with the appropriate constructor (`ST_GeomFromText` or `ST_GeogFromText`).

### Prompt Snippet
```text
ST_InteriorRingN(polygon_object, index_integer)
```

### Common Failure Modes
- **Index Out of Range:** Providing an `n` value that exceeds the number of interior rings in the polygon results in a `NULL` return.
- **Invalid Geometry Type:** Passing a `POINT`, `LINESTRING`, or `MULTIPOLYGON` instead of a `POLYGON` results in a `NULL` return.
- **0-based Indexing:** Attempting to use `0` as the index will fail to return the first ring, as the API strictly follows 1-based indexing.

### Fix Code Hint
```python
# Verify the number of interior rings first if unsure of the index
sd.sql("SELECT ST_NumInteriorRings(geom) FROM my_table").to_arrow_table()
# Then call ST_InteriorRingN with a valid index
sd.sql("SELECT ST_InteriorRingN(geom, 1) FROM my_table").to_arrow_table()
```

__CHECK__

## API Test: `sql:ST_Intersection`

### Signature
```python
ST_Intersection(arg1: geometry, arg2: geometry) -> geometry | ST_Intersection(arg1: geography, arg2: geography) -> geography
```
_Source: source/docs/reference/sql/st_intersection.qmd:19_

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

title: ST_Intersection
description: Computes the intersection of two geometries or geographies.
kernels:
  - returns: geometry
    args: [geometry, geometry]
  - returns: geography
    args: [geography, geography]
---

## Examples

```sql
SELECT ST_Intersection(
    ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))'),
    ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))')
) AS val;
```

```{python}
import _matplotlib_defaults
import sedonadb

sd = sedonadb.connect()

sd.sql("""
SELECT
    ST_GeomFromText('POLYGON ((1 1, 11 1, 11 11, 1 11, 1 1))') as geom_a,
    ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0))') as geom_b
""").to_view("input")

df = sd.sql("SELECT *, ST_Intersection(geom_a, geom_b) as result FROM input").to_pandas()

ax = df.geom_a.plot(alpha=0.5)
df.geom_b.plot(ax=ax, alpha=0.5)
df.result.plot(ax=ax, facecolor='none', edgecolor='red')
```


### Goal
Computes the spatial intersection of two input geometries or geographies, returning the shared region as a new geometry or geography object.

### Parameters
- `arg1` (`geometry`): The first geometry or geography object to intersect.
- `arg2` (`geometry`): The second geometry or geography object to intersect.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography types.
- **Preconditions:** Both inputs must share the same coordinate reference system (CRS). If CRS differs, geometries must be reprojected to a common CRS before intersection. If only one input has a defined CRS, the operation will fail.

### Output
Returns `geography | geometry` — A new geometry or geography object representing the intersection of the two inputs.

### Valid Call Patterns
```python
# Using Python SQL connection
result = sd.sql("SELECT ST_Intersection(geom_a, geom_b) as result FROM input").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both inputs are of the same type (both `geometry` or both `geography`).
- Verify that both inputs share the same CRS before calling the function.
- Use `ST_GeomFromText` or similar constructors to prepare inputs if they are provided as WKT strings.

### Prompt Snippet
```text
SELECT ST_Intersection(geom_a, geom_b)
```

### Common Failure Modes
- **CRS Mismatch:** Operation fails if inputs have different CRS or if only one input has a CRS defined.
- **Type Mismatch:** Attempting to intersect a `geometry` with a `geography` type.
- **Invalid Geometries:** Input geometries that are not OGC-compliant may cause errors; use `ST_MakeValid` prior to intersection if necessary.

### Fix Code Hint
```python
# Ensure CRS consistency before intersection
sd.sql("""
SELECT ST_Intersection(
    ST_Transform(geom_a, 'EPSG:4326'), 
    ST_Transform(geom_b, 'EPSG:4326')
) FROM input
""").to_arrow_table()
```

## API Test: `sql:ST_Intersection_Agg`

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
Computes the cumulative spatial intersection of all `geometry` objects provided in an aggregate input set.

### Parameters
- `arg1` (`geometry`): A column or expression containing the geometries to be intersected.

### Input
- **Data:** A collection of `geometry` objects (e.g., Polygons, Lines).
- **Preconditions:** All input geometries must be valid OGC geometries. If geometries have different Coordinate Reference Systems (CRS), they must be transformed to a common CRS before aggregation to ensure meaningful results.

### Output
Returns `geometry` — The resulting geometry representing the common area or intersection shared by all input geometries. If no intersection exists, the result may be an empty geometry.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example using the documented SQL pattern
query = """
SELECT ST_Intersection_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('POLYGON ((0 0, 0 4, 4 4, 4 0, 0 0))')),
    (ST_GeomFromText('POLYGON ((2 2, 2 6, 6 6, 6 2, 2 2))'))
) AS t(geom);
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Intersection_Agg` only within a `SELECT` statement as an aggregate function.
- Ensure the input column contains valid `geometry` types.
- If the input geometries are expected to have different CRS, explicitly include an `ST_Transform` call in the subquery before passing the geometry to the aggregate.

### Prompt Snippet
```text
Use ST_Intersection_Agg(geom_column) to compute the intersection of all geometries in a group.
```

### Common Failure Modes
- **CRS Mismatch:** Attempting to intersect geometries with different SRIDs without prior transformation will result in errors or undefined behavior.
- **Invalid Geometries:** Inputting self-intersecting or otherwise invalid geometries may cause the operation to fail; use `ST_MakeValid` on inputs if necessary.
- **Empty Intersection:** If the input geometries do not overlap, the function returns an empty geometry, which might cause downstream errors if not handled.

### Fix Code Hint
```python
# Ensure geometries are valid and in the same CRS before aggregation
query = """
SELECT ST_Intersection_Agg(ST_MakeValid(geom)) 
FROM my_table 
WHERE ST_SRID(geom) = 4326
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_Intersects`

### Signature
```python
ST_Intersects(arg1: geometry, arg2: geometry) -> boolean | ST_Intersects(arg1: geography, arg2: geography) -> boolean
```
_Source: source/docs/reference/sql/st_intersects.qmd:19_

### Goal
Determines whether two spatial objects (geometries or geographies) share any common space, returning true if they intersect.

### Parameters
- `arg1` (`geometry` or `geography`): The first spatial object to evaluate.
- `arg2` (`geometry` or `geography`): The second spatial object to evaluate.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Standard OGC WKT/WKB geometries or geography objects created via `ST_GeogFromText` or `ST_GeogPoint`.
- **Preconditions:** Both arguments must share the same spatial type (both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered. If CRS differs, geometries are reprojected to the raster's CRS; if only one side has a CRS, the operation will fail.

### Output
Returns `boolean` — `true` if the two objects intersect, `false` otherwise.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Geometry intersection
result = sd.sql("""
SELECT ST_Intersects(
    ST_Point(0.5, 0.5),
    ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))')
)
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Intersects(arg1, arg2)` as a standalone SQL function call.
- Ensure both arguments are of the same spatial type (`geometry` or `geography`).
- If using `geography`, ensure the inputs are constructed using geography-specific constructors (e.g., `ST_GeogPoint`).

### Prompt Snippet
```text
Use ST_Intersects(geomA, geomB) to check for spatial intersection. Ensure both inputs are either geometry or geography types.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` object to one argument and a `geography` object to the other.
- **CRS Inconsistency:** Attempting to intersect objects with different coordinate systems where one lacks a defined CRS.
- **Missing Bounder:** Calling `ST_Intersects` on `geography` types without the `s2geography` bounder registered in the session.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type if necessary
sd.sql("SELECT ST_Intersects(ST_SetSRID(geom1, 4326), ST_SetSRID(geom2, 4326)) FROM table")
```

## API Test: `sql:ST_IsClosed`

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
Determines if a `LineString` or `MultiLineString` geometry or geography is closed by verifying that the start and end points are identical.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate. For `MultiLineString` inputs, all constituent elements must be closed for the function to return `true`.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** Ensure the input is a `LineString` or `MultiLineString`. Non-linear geometries (e.g., `Point`, `Polygon`) may not yield meaningful results for this predicate.

### Output
Returns `boolean` — `true` if the start and end points of the input are the same, otherwise `false`.

### Valid Call Patterns
```python
# Geometry example
result = sd.sql("SELECT ST_IsClosed(ST_GeomFromText('LINESTRING(0 0, 1 1, 1 0, 0 0)'))").to_arrow_table()
print(result) # __CHECK__

# Geography example
result = sd.sql("SELECT ST_IsClosed(ST_GeogFromWKT('LINESTRING(0 0, 1 1, 1 0, 0 0)'))").to_arrow_table()
print(result) # __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_IsClosed(arg1)` as a standalone SQL function call within `sd.sql()`.
- Ensure the input is explicitly cast or constructed using `ST_GeomFromText` or `ST_GeogFromWKT` if the input is a raw WKT string.

### Prompt Snippet
```text
SELECT ST_IsClosed(ST_GeomFromText('LINESTRING(...)'));
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-linear geometry type (like a `Point`) may result in unexpected behavior or errors.
- **MultiLineString Requirement:** For `MultiLineString` inputs, the function returns `false` if even one of the constituent `LineString` elements is not closed.
- **Geography Bounder:** Ensure the session's geography bounder is registered if performing operations on `geography` types.

### Fix Code Hint
```python
# Ensure input is a valid LineString or MultiLineString
sd.sql("SELECT ST_IsClosed(ST_MakeValid(geom)) FROM table_name")
```

## API Test: `sql:ST_IsCollection`

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
Determines whether a given `geometry` or `geography` object is a collection type (GEOMETRYCOLLECTION, MULTIPOINT, MULTIPOLYGON, or MULTILINESTRING).

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect for collection status.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` object created via constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT`.

### Output
Returns `boolean` — `TRUE` if the input is a collection type, `FALSE` otherwise.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_IsCollection(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_IsCollection(ST_GeogFromWKT('MULTIPOINT ((0 0), (1 1))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_IsCollection` to verify if a spatial object is a collection type.
- Ensure the input is wrapped in a valid constructor (e.g., `ST_GeomFromWKT`) if passing a WKT string.
- The function returns a boolean; use it in `WHERE` clauses or `SELECT` projections to filter or flag collection geometries.

### Prompt Snippet
```text
ST_IsCollection(geometry_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., string or integer) directly without a constructor will result in a SQL execution error.
- **Null Inputs:** Passing `NULL` will return `NULL` rather than `FALSE`.
- **Invalid WKT:** Providing malformed WKT strings to the constructor will cause the query to fail before `ST_IsCollection` is evaluated.

### Fix Code Hint
```python
# Ensure input is a valid geometry object
query = "SELECT ST_IsCollection(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

## API Test: `sql:ST_IsEmpty`

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
Determines whether a given `geometry` or `geography` object contains no points or is explicitly defined as an empty set.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to evaluate for emptiness.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If using `geography`, ensure the session's geography bounder is registered.

### Output
Returns `boolean` — `true` if the geometry or geography is empty (e.g., `POINT EMPTY`), and `false` otherwise.

### Valid Call Patterns
```python
# Geometry check
sd.sql("SELECT ST_IsEmpty(ST_GeomFromWKT('POINT EMPTY'))").to_arrow_table()

# Geography check
sd.sql("SELECT ST_IsEmpty(ST_GeogFromWKT('POINT EMPTY'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_IsEmpty(arg1)` to verify if a spatial object is empty.
- Ensure the input is correctly cast to `geometry` or `geography` using constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT` if the input is a string.
- Do not assume non-empty geometries are empty; this function is specifically for checking the "EMPTY" state of a spatial object.

### Prompt Snippet
```text
ST_IsEmpty(geometry_column)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., string or integer) directly without a constructor will result in a SQL execution error.
- **Geography Bounder:** If performing checks on `geography` types, failure to have the `s2geography` bounder registered on the session may cause errors.
- **Null Inputs:** Passing `NULL` as `arg1` will return `NULL` rather than `true` or `false`.

### Fix Code Hint
```python
# Ensure input is a valid geometry object
sd.sql("SELECT ST_IsEmpty(ST_GeomFromWKT(wkt_column)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_IsRing`

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
- `arg1` (`geometry`): The input geometry to evaluate. Must be a `LineString` type.

### Input
- **Data Format:** Standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a `LineString`. If the geometry is not a `LineString`, the behavior is undefined or may return `false` depending on the underlying implementation.

### Output
Returns `boolean` — `true` if the geometry is both closed (start and end points are identical) and simple (no self-intersections), otherwise `false`.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_IsRing(ST_GeomFromText('LINESTRING(0 0, 0 1, 1 1, 1 0, 0 0)'))").to_arrow_table()
print(result)
# __CHECK__ witness: true
```

### LLM Instruction Prompt
- Always use `ST_IsRing(geometry_expression)` in SQL queries.
- Ensure the input is a `LineString` geometry; if the input is a different type, consider using a conversion or validation function first.

### Prompt Snippet
```text
SELECT ST_IsRing(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Non-LineString Input:** Passing a `Point`, `Polygon`, or `MultiLineString` may result in unexpected behavior or errors.
- **Self-Intersecting Lines:** A closed line that crosses itself will return `false` because it fails the `ST_IsSimple` requirement.
- **Open Lines:** A line that does not return to its start point will return `false` because it fails the `ST_IsClosed` requirement.

### Fix Code Hint
```python
# Ensure the geometry is a valid LineString before checking
sd.sql("SELECT ST_IsRing(ST_GeomFromText('LINESTRING(0 0, 1 1, 0 1, 0 0)'))").to_arrow_table()
```

## API Test: `sql:ST_IsSimple`

### Signature
```python
ST_IsSimple(arg1: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_issimple.qmd:19_

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

title: ST_IsSimple
description: Tests if geometry's only self-intersections are at boundary points.
kernels:
  - returns: boolean
    args: [geometry]
---

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
Determines if a given geometry is "simple" by verifying that it contains no anomalous geometric points, such as self-intersections or self-tangency, except at boundary points.

### Parameters
- `arg1` (`geometry`): The input geometry object to be tested for simplicity.

### Input
- **Data Format:** Accepts standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the input is provided as a string (e.g., WKT), it must be wrapped in a constructor like `ST_GeomFromWKT` or `ST_GeomFromText`.

### Output
Returns `boolean` — `true` if the geometry is simple, `false` otherwise.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_IsSimple(ST_GeomFromWKT('POLYGON((1 1, 3 1, 3 3, 1 3, 1 1))'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_IsSimple(geometry_expression)` in SQL queries.
- Ensure the input is a valid `geometry` type; use `ST_GeomFromWKT` or `ST_GeomFromText` if the input is a string literal.

### Prompt Snippet
```text
SELECT ST_IsSimple(ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **Invalid Geometry Input:** Passing a non-geometry type or a malformed WKT string will result in a query execution error.
- **Complex Geometries:** Geometries with self-intersections (e.g., a "figure-eight" linestring) will correctly return `false`, which may be unexpected if the user assumes all valid OGC geometries are simple.

### Fix Code Hint
```python
# Ensure the geometry is properly constructed before testing
sd.sql("SELECT ST_IsSimple(ST_MakeValid(geom)) FROM table_name")
```

## API Test: `sql:ST_IsValid`

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
Determines if a given geometry object conforms to OGC standards for spatial validity.

### Parameters
- `arg1` (`geometry`): The geometry object to be validated.

### Input
- Data: A `geometry` type object (e.g., created via `ST_GeomFromText`, `ST_Point`, etc.).
- Formats: Supports standard OGC WKT/WKB geometries.
- Preconditions: The input must be a valid `geometry` object; if the input is invalid or null, the behavior follows standard SQL null-handling rules.

### Output
Returns `boolean` — `true` if the geometry is well-formed according to OGC rules, `false` otherwise.

### Valid Call Patterns
```python
# Using the Python SQL connection
result = sd.sql("SELECT ST_IsValid(ST_GeomFromText('POLYGON ((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))'))").to_arrow_table()
print(result)
# __CHECK__ witness: result contains a boolean column
```

### LLM Instruction Prompt
- Always use `ST_IsValid(geometry_expression)` within a `SELECT` statement via `sd.sql()`.
- If the geometry is suspected to be invalid, suggest using `ST_IsValidReason` for diagnostics or `ST_MakeValid` for repair.

### Prompt Snippet
```text
SELECT ST_IsValid(geometry_column) FROM table_name;
```

### Common Failure Modes
- Passing non-geometry types (e.g., strings or integers) directly without conversion functions like `ST_GeomFromText`.
- Assuming that `ST_IsValid` repairs the geometry; it is a read-only check. Use `ST_MakeValid` for repair.

### Fix Code Hint
```python
# If ST_IsValid returns false, repair the geometry:
repaired_df = sd.sql("SELECT ST_MakeValid(geom) FROM table_name WHERE NOT ST_IsValid(geom)").to_arrow_table()
```

## API Test: `sql:ST_IsValidReason`

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
`sql:ST_IsValidReason` provides a diagnostic text string explaining the specific OGC compliance violation for an invalid geometry or confirms the geometry is valid.

### Parameters
- `arg1` (`geometry`): The geometry object to be inspected for validity.

### Input
- **Data:** A `geometry` type object (e.g., created via `ST_GeomFromText`, `ST_Point`, etc.).
- **Preconditions:** The input must be a valid SedonaDB `geometry` type. If the geometry is constructed from WKT or WKB, ensure the format is correct before passing it to the function.

### Output
Returns `string` — A descriptive message detailing the reason for invalidity (e.g., "Self-intersection") or the literal string `"Valid Geometry"`.

### Valid Call Patterns
```python
# Example using Python SQL connection
import sedonadb
sd = sedonadb.connect()
result = sd.sql("""
    SELECT ST_IsValidReason(ST_GeomFromText('POLYGON((0 0, 2 0, 2 2, 1 1, 0 2, 0 0))')) AS reason
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_IsValidReason` to diagnose geometry issues when `ST_IsValid` returns `false`.
- The function accepts a single `geometry` argument.
- Do not attempt to pass `geography` types unless explicitly cast to `geometry`.

### Prompt Snippet
```text
Use ST_IsValidReason(geometry_column) to retrieve the error message for invalid geometries.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geography` type or a non-spatial data type will result in a query execution error.
- **Null Input:** Passing a `NULL` geometry will return `NULL` rather than a string.
- **Invalid WKT:** If the input geometry cannot be parsed (e.g., malformed WKT), the function will fail during the geometry construction phase before `ST_IsValidReason` is evaluated.

### Fix Code Hint
```python
# Ensure geometry is valid before processing or diagnose with ST_IsValidReason
sd.sql("SELECT ST_IsValidReason(geom) FROM my_table WHERE ST_IsValid(geom) = FALSE")
```

## API Test: `sql:ST_KNN`

### Signature
```python
ST_KNN(geomA: geometry, geomB: geometry, k: integer, use_spheroid: boolean) -> boolean | ST_KNN(geomA: geometry, geomB: geometry, k: integer) -> boolean | ST_KNN(geomA: geometry, geomB: geometry) -> boolean
```
_Source: source/docs/reference/sql/st_knn.qmd:19_

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

title: ST_KNN
description: Returns true if geomA finds k nearest neighbors from geomB.
kernels:
  - returns: boolean
    args:
    - name: geomA
      type: geometry
      description: The geometry around which to search.
    - name: geomB
      type: geometry
      description: Column containing candidate geometries.
    - name: k
      type: integer
      description: The number of nearest neighbours to return. Defaults to 1.
    - name: use_spheroid
      type: boolean
      description: true to use spherical distance, false for Euclidean. Defaults to false.
  - returns: boolean
    args:
    - name: geomA
      type: geometry
    - name: geomB
      type: geometry
    - name: k
      type: integer
  - returns: boolean
    args:
    - name: geomA
      type: geometry
    - name: geomB
      type: geometry
---

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
`ST_KNN` acts as a join predicate to identify the k-nearest neighbor geometries from a candidate set relative to a target geometry.

### Parameters
- `geomA` (`geometry`): The geometry around which to search.
- `geomB` (`geometry`): Column containing candidate geometries to evaluate.
- `k` (`integer`): The number of nearest neighbors to return. Defaults to 1.
- `use_spheroid` (`boolean`): If true, uses spherical distance; if false, uses Euclidean distance. Defaults to false.

### Input
Requires two sets of geometries (typically provided as columns in a join operation). Geometries must be valid OGC types. Precondition: The function is intended for use within a SQL `JOIN` clause.

### Output
Returns `boolean` — A boolean value indicating if the geometry satisfies the k-nearest neighbor condition within the spatial join execution engine.

### Valid Call Patterns
```python
# Example of using ST_KNN in a join query via Python
query = """
SELECT * FROM table1 
JOIN table2 ON ST_KNN(table2.geom1, table1.geom2, 3)
"""
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_KNN` within a `JOIN ... ON` clause.
- Do not attempt to use `ST_KNN` as a standalone scalar function in a `SELECT` list, as it is a stub for the spatial join engine.
- Ensure `k` is a positive integer.

### Prompt Snippet
```text
ST_KNN(geomA, geomB, k)
```

### Common Failure Modes
- Using `ST_KNN` outside of a join condition.
- Providing non-geometry types for `geomA` or `geomB`.
- Assuming the function returns a list of geometries (it is a predicate that returns a boolean).

### Fix Code Hint
```python
# Ensure the join condition is correctly structured
sd.sql("SELECT * FROM t1 JOIN t2 ON ST_KNN(t2.geom, t1.geom, 5)")
```

## API Test: `sql:ST_Length`

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
Computes the linear length of a `geometry` or `geography` object, specifically supporting `LineString`, `MultiLineString`, and `GeometryCollections` containing linear segments.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to measure.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Formats:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a linear geometry type (`LineString`, `MultiLineString`, or `GeometryCollection` of lines). Polygons are not supported; use `ST_Perimeter` for polygons instead.

### Output
Returns `double` — The length of the input. For `geometry`, the value is in the units of the coordinate reference system; for `geography`, the value is the spherical approximation of the geodesic length in meters.

### Valid Call Patterns
```python
# Geometry length calculation
sd.sql("SELECT ST_Length(ST_GeomFromWKT('LINESTRING (0 0, 0 1)'))").to_arrow_table()

# Geography length calculation
sd.sql("SELECT ST_Length(ST_GeogFromWKT('LINESTRING (0 0, 0 1)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always verify that the input geometry is a linear type (`LineString`, `MultiLineString`) before calling `ST_Length`.
- If the input is a `Polygon`, suggest `ST_Perimeter` instead.
- Ensure the input is correctly cast to `geometry` or `geography` using `ST_GeomFromWKT` or `ST_GeogFromWKT` if provided as a string.

### Prompt Snippet
```text
Use ST_Length(geom) for LineString/MultiLineString length. For polygons, use ST_Perimeter.
```

### Common Failure Modes
- **Unsupported Geometry Type:** Passing a `Polygon` or `Point` will result in an error or incorrect measurement; use `ST_Perimeter` for polygons.
- **Geography Bounder:** Geography operations require the `s2geography` bounder to be registered on the session.
- **CRS Mismatch:** For `geometry` types, the result is dependent on the units of the coordinate reference system; ensure the CRS is appropriate for the desired unit of measure.

### Fix Code Hint
```python
# Ensure input is a linear geometry
query = "SELECT ST_Length(ST_GeomFromWKT('LINESTRING (0 0, 0 1)'))"
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness: length computed")
```

## API Test: `sql:ST_LineFromText`

### Signature
```python
ST_LineFromText(wkt: string) -> geometry | ST_LineFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_linefromtext.qmd:19_

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

title: ST_LineFromText
description: Constructs a LineString from Well-Known Text (WKT), erroring if the input is not a LineString.
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

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a LineString.

Aliases: `ST_LineStringFromText`.

## Examples

```sql
SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)');
```

With an SRID or CRS:

```sql
SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)', 4326);
```


### Goal
Constructs a `geometry` object of type `LineString` from a provided Well-Known Text (WKT) string.

### Parameters
- `wkt` (`string`): A Well-Known Text representation of a LineString (e.g., `'LINESTRING (0 0, 1 1)'`).
- `srid` (`crs`): An optional integer or string representing the Coordinate Reference System (e.g., `4326` or `'EPSG:4326'`).

### Input
- **Format:** Must be a valid OGC WKT string representing a `LineString`.
- **Preconditions:** The input string must strictly define a `LineString` geometry; providing other geometry types (e.g., `POINT`, `POLYGON`) will result in an error.

### Output
Returns a `geometry` object representing the parsed LineString.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_LineFromText('LINESTRING (0 0, 1 1, 2 2)', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_LineFromText` when converting raw WKT strings into SedonaDB `geometry` objects.
- Always verify that the WKT input is a `LINESTRING` to avoid runtime errors.
- If a spatial reference system is required for subsequent operations (like `ST_Transform`), include the `srid` parameter.

### Prompt Snippet
```text
ST_LineFromText(wkt_string, [srid])
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `LINESTRING` (e.g., `POINT(0 0)`) will raise an error.
- **Malformed WKT:** Providing an improperly formatted WKT string (e.g., missing parentheses or invalid coordinate pairs) will cause the operation to fail.

### Fix Code Hint
```python
# Ensure the WKT is a valid LineString
wkt_input = "LINESTRING (0 0, 1 1)"
query = f"SELECT ST_LineFromText('{wkt_input}', 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_LineInterpolatePoint`

### Signature
```python
ST_LineInterpolatePoint(arg1: geometry, fraction: double) -> geometry | ST_LineInterpolatePoint(arg1: geography, fraction: double) -> geography
```
_Source: source/docs/reference/sql/st_lineinterpolatepoint.qmd:19_

### Goal
Calculates a point located at a specific fractional distance along a provided `LINESTRING` geometry or geography.

### Parameters
- `arg1` (`geometry` | `geography`): A `LINESTRING` object representing the path along which the point is interpolated.
- `fraction` (`double`): A value between 0.0 and 1.0 representing the percentage of the total linestring length where the point should be placed.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The input `arg1` must be a `LINESTRING`. For `geography` inputs, the interpolation is performed using spherical interpolation between vertices.
- **Formats:** Standard OGC WKT/WKB geometries or geographies created via `ST_GeomFromWKT` or `ST_GeogFromWKT`.

### Output
Returns a `geometry` or `geography` point object representing the interpolated location.

### Valid Call Patterns
```python
# Geometry interpolation
sd.sql("SELECT ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (0 0, 1 1)'), 0.5)").to_arrow_table()

# Geography interpolation
sd.sql("SELECT ST_LineInterpolatePoint(ST_GeogFromWKT('LINESTRING (0 0, 1 1)'), 0.5)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the first argument is a `LINESTRING`.
- Ensure the `fraction` argument is a `double` between 0 and 1.
- Use `ST_GeomFromWKT` for planar geometry or `ST_GeogFromWKT` for spherical geography inputs.
- The function returns a single point geometry/geography; ensure the downstream operations expect this type.

### Prompt Snippet
```text
ST_LineInterpolatePoint(arg1, fraction)
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a geometry type other than `LINESTRING` (e.g., `POINT` or `POLYGON`) will result in an error.
- **Fraction Out of Bounds:** Providing a `fraction` value outside the [0, 1] range may lead to undefined behavior or errors.
- **Geography/Geometry Mismatch:** Ensure the input type matches the expected kernel; geography operations require the `s2geography` bounder to be registered on the session.

### Fix Code Hint
```python
# Ensure input is a LINESTRING and fraction is within [0, 1]
query = "SELECT ST_LineInterpolatePoint(ST_GeomFromWKT('LINESTRING (0 0, 10 10)'), 0.5)"
result = sd.sql(query).to_arrow_table()
print(result) # __CHECK__
```

## API Test: `sql:ST_LineLocatePoint`

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
Calculates the relative position of the point on a LineString as a normalized fraction (0.0 to 1.0) of the line's total length.

### Parameters
- `arg1` (`geometry` or `geography`): The reference LineString geometry or geography.
- `arg2` (`geometry` or `geography`): The point geometry or geography to locate on the line.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The first argument (`arg1`) must be a `LineString`. Both arguments must share the same type (`geometry` or `geography`).

### Output
Returns `double` — A value between 0.0 and 1.0 representing the fraction of the total 2D length (for geometry) or spherical distance (for geography) where the closest point resides.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_LineLocatePoint(ST_GeomFromText('LINESTRING (0 0, 0 1)'), ST_GeomFromText('POINT (0 0.5)'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_LineLocatePoint(ST_GeogFromWKT('LINESTRING (0 0, 1 1, 2 2)'), ST_GeogFromWKT('POINT (1 1)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the first argument is a `LineString`.
- Ensure both arguments are of the same type (both `geometry` or both `geography`).
- Use `ST_GeomFromText` for planar geometry and `ST_GeogFromWKT` for spherical geography.

### Prompt Snippet
```text
ST_LineLocatePoint(line_geom, point_geom)
```

### Common Failure Modes
- Passing a non-LineString geometry as the first argument.
- Mixing `geometry` and `geography` types in the same function call.
- Providing inputs with mismatched coordinate reference systems (CRS) if applicable.

### Fix Code Hint
```python
# Ensure inputs are cast to the same type
sd.sql("""
    SELECT ST_LineLocatePoint(
        ST_GeomFromText('LINESTRING (0 0, 0 1)'), 
        ST_GeomFromText('POINT (0 0.5)')
    )
""").to_arrow_table()
# __CHECK__ witness: The result should be a double value (e.g., 0.5)
```

## API Test: `sql:ST_LineMerge`

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
# "AS IS\" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
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
SELECT ST_LineMerge(
    ST_GeogFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))')
);
```


### Goal
`sql:ST_LineMerge` merges a collection of potentially connected line segments into the fewest possible `LineString` objects.

### Parameters
- `arg1` (`geometry` or `geography`): A collection of line segments, typically provided as a `MULTILINESTRING` or a collection of `LineString` geometries.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a collection of line segments. If using `geography` types, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A `LineString` or `MultiLineString` representing the merged segments.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_LineMerge(ST_GeomFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_LineMerge(ST_GeogFromWKT('MULTILINESTRING ((0 0, 1 0), (1 0, 1 1))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_LineMerge` when you need to simplify a collection of connected line segments into continuous paths.
- Ensure the input type (`geometry` vs `geography`) matches the desired output type.
- Always wrap the input in a constructor like `ST_GeomFromWKT` or `ST_GeogFromWKT` if passing raw WKT strings.

### Prompt Snippet
```text
SELECT ST_LineMerge(ST_GeomFromWKT('MULTILINESTRING ((...))'));
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a function expecting `geography` or vice versa without explicit conversion.
- **Disconnected Segments:** If segments do not share endpoints, they cannot be merged and will remain as separate components in the resulting `MultiLineString`.
- **Geography Bounder:** Failure to register the `s2geography` bounder when performing operations on `geography` types.

### Fix Code Hint
```python
# Ensure input is correctly typed and segments are connected
sd.sql("SELECT ST_LineMerge(ST_MakeValid(geom)) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_LineStringFromWKB`

### Signature
```python
ST_LineStringFromWKB(wkb: binary) -> geometry | ST_LineStringFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_linestringfromwkb.qmd:19_

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

title: ST_LineStringFromWKB
description: Constructs a LineString from Well-Known Binary (WKB), returning NULL if the input is another geometry type.
kernels:
  - returns: geometry
    args:
    - name: wkb
      type: binary
  - returns: geometry
    args:
    - name: wkb
      type: binary
    - name: srid
      type: crs
---

## Description

Parses a WKB binary value and returns a LineString geometry. Returns `NULL` if
the WKB represents another geometry type. An optional SRID or CRS can be
provided as a second argument.

Alias: `ST_LineFromWKB`.

## Examples

```sql
SELECT ST_LineStringFromWKB(
  decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex')
);
```

With an SRID or CRS:

```sql
SELECT ST_LineStringFromWKB(
  decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex'),
  4326
);
```


### Goal
Constructs a `LineString` geometry object from a Well-Known Binary (WKB) representation.

### Parameters
- `wkb` (`binary`): The Well-Known Binary (WKB) byte sequence representing a LineString.
- `srid` (`crs`): An optional integer SRID or CRS string (e.g., 'EPSG:4326') to assign to the resulting geometry.

### Input
The caller must provide a valid WKB binary sequence. If the binary data represents a geometry type other than a `LineString`, the function will return `NULL`.

### Output
Returns a `geometry` object representing the parsed LineString.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_LineStringFromWKB(decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex'))").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_LineStringFromWKB(decode('01020000000200000000000000000000000000000000000000000000000000F03F000000000000F03F', 'hex'), 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_LineStringFromWKB` when converting raw binary WKB data into usable geometry objects.
- Always ensure the input binary is a valid WKB representation of a LineString.
- If an SRID is known, provide it as the second argument to ensure correct spatial reference metadata.

### Prompt Snippet
```text
ST_LineStringFromWKB(wkb_binary_column, 4326)
```

### Common Failure Modes
- Providing WKB data that represents a different geometry type (e.g., `Point` or `Polygon`) results in a `NULL` return value.
- Providing malformed or truncated binary data will result in an error during parsing.

### Fix Code Hint
```python
# Ensure the input is a valid LineString WKB and handle potential NULLs
sd.sql("SELECT ST_LineStringFromWKB(wkb_col, 4326) as geom FROM table WHERE geom IS NOT NULL")
```

## API Test: `sql:ST_LineSubstring`

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
Extracts a segment of a `LINESTRING` geometry based on specified start and end fractions of its total 2D length.

### Parameters
- `arg1` (`geometry`): The input `LINESTRING` geometry to be sliced.
- `start_fraction` (`double`): The starting point of the substring, expressed as a fraction of the total 2D length (must be between 0 and 1).
- `end_fraction` (`double`): The ending point of the substring, expressed as a fraction of the total 2D length (must be between 0 and 1).

### Input
The input must be a valid `geometry` object of type `LINESTRING`. Geometries can be constructed using standard OGC WKT/WKB functions like `ST_GeomFromWKT`.

### Output
Returns a `geometry` object representing the extracted `LINESTRING` substring.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_AsText(
        ST_LineSubstring(ST_GeomFromWKT('LINESTRING(0 0, 10 10)'), 0.3, 0.6)
    )
""").to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a LINESTRING representing the segment between 30% and 60% of the input length.
```

### LLM Instruction Prompt
- Always use `ST_LineSubstring(geometry, start_fraction, end_fraction)` within a SQL query string passed to `sd.sql()`.
- Ensure `start_fraction` and `end_fraction` are provided as `double` values between 0.0 and 1.0.
- Wrap the result in `ST_AsText()` if you need to verify the geometry coordinates in the output.

### Prompt Snippet
```text
ST_LineSubstring(geometry_column, 0.0, 1.0)
```

### Common Failure Modes
- Providing `start_fraction` or `end_fraction` values outside the [0, 1] range.
- Passing a geometry type other than `LINESTRING` (e.g., `POINT` or `POLYGON`), which may result in an execution error.
- Using invalid WKT strings for the input geometry.

### Fix Code Hint
```python
# Ensure fractions are clamped to [0, 1]
start = max(0.0, min(1.0, start_val))
end = max(0.0, min(1.0, end_val))
query = f"SELECT ST_LineSubstring(geom, {start}, {end}) FROM table"
```

## API Test: `sql:ST_M`

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
Extracts the M (measure) coordinate value from a `geometry` or `geography` Point object.

### Parameters
- `arg1` (`geometry` or `geography`): The input Point object from which to extract the measure coordinate.

### Input
- **Data:** A `geometry` or `geography` type object.
- **Preconditions:** The input must be a Point geometry. If the geometry lacks an M dimension or is not a Point, the function returns `NULL`.

### Output
Returns `double` — The M (measure) coordinate value of the point, or `NULL` if the dimension is missing or the geometry type is invalid.

### Valid Call Patterns
```python
# Extract M coordinate from a PointM geometry
sd.sql("SELECT ST_M(ST_PointM(1.0, 2.0, 42.0))").to_arrow_table()

# Extract M coordinate from a standard Point (returns NULL)
sd.sql("SELECT ST_M(ST_Point(1.0, 2.0))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_M` to retrieve the measure coordinate.
- Ensure the input is a Point geometry; non-point geometries will result in `NULL`.
- If the geometry does not contain an M dimension, the result will be `NULL`.

### Prompt Snippet
```text
ST_M(geometry_column)
```

### Common Failure Modes
- **Non-Point Geometries:** Passing a `LineString`, `Polygon`, or other non-point geometry will return `NULL`.
- **Missing M Dimension:** Passing a standard 2D or 3D point (without M) will return `NULL`.
- **Type Mismatch:** Passing non-geospatial types will result in a SQL execution error.

### Fix Code Hint
```python
# Ensure the geometry has an M dimension before calling ST_M
# If the geometry is 2D, use ST_AddMeasure or similar to construct a valid M-enabled point
sd.sql("SELECT ST_M(ST_PointM(x, y, m)) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_MLineFromText`

### Signature
```python
ST_MLineFromText(wkt: string) -> geometry | ST_MLineFromText(wkt: string, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_mlinefromtext.qmd:19_

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

title: ST_MLineFromText
description: Constructs a MultiLineString from Well-Known Text (WKT), erroring if the input is not a MultiLineString.
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

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a MultiLineString.

## Examples

```sql
SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))');
```

With an SRID or CRS:

```sql
SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))', 4326);
```


### Goal
Constructs a `geometry` object of type `MultiLineString` from a provided Well-Known Text (WKT) string.

### Parameters
- `wkt` (`string`): A Well-Known Text representation of a `MULTILINESTRING`.
- `srid` (`crs`): An optional spatial reference system identifier (e.g., `4326`) or CRS string to associate with the geometry.

### Input
The input must be a valid WKT string representing a `MULTILINESTRING`. If an SRID is provided, it must be a valid integer or CRS string supported by the SedonaDB environment.

### Output
Returns a `geometry` object representing the parsed `MultiLineString`.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_MLineFromText('MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_MLineFromText` only when the input WKT is strictly a `MULTILINESTRING`.
- If an SRID is required for spatial operations, pass it as the second argument.
- Ensure the WKT string is properly escaped if used within a Python f-string.

### Prompt Snippet
```text
ST_MLineFromText(wkt_string)
ST_MLineFromText(wkt_string, srid_integer)
```

### Common Failure Modes
- **Invalid Geometry Type:** The function raises an error if the provided WKT string does not represent a `MULTILINESTRING` (e.g., passing a `POINT` or `POLYGON` WKT).
- **Malformed WKT:** Passing an improperly formatted WKT string will result in a parsing error.

### Fix Code Hint
```python
# Ensure the WKT is a MultiLineString
wkt_input = 'MULTILINESTRING ((0 0, 1 1), (2 2, 3 3))'
result = sd.sql(f"SELECT ST_MLineFromText('{wkt_input}')").to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_MMax`

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
Returns the maximum M (measure) coordinate value present in the bounding box of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object containing M-dimension data.

### Input
The input must be a valid `geometry` or `geography` object that includes an M (measure) dimension. Inputs are typically constructed via `ST_GeomFromWKT` or `ST_GeogFromWKT` using formats that support M-values (e.g., `LINESTRING M`).

### Output
Returns a `double` representing the maximum M-value found within the spatial object's bounding box.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_MMax(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_MMax(ST_GeogFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_MMax` as a standalone SQL function call within `sd.sql()`.
- Ensure the input geometry or geography explicitly contains M-dimension data; otherwise, the result may be undefined or null.

### Prompt Snippet
```text
Use ST_MMax(geometry_expression) to extract the maximum M-value from a geometry or geography object.
```

### Common Failure Modes
- Passing a geometry that lacks an M-dimension (e.g., standard 2D `POINT` or `LINESTRING`), which may result in an error or null return.
- Providing an invalid WKT string that does not conform to the expected M-dimension format.

### Fix Code Hint
```python
# Ensure the geometry is constructed with M-dimension support
sd.sql("SELECT ST_MMax(ST_GeomFromWKT('LINESTRING M (0 0 0, 10 10 20)'))").to_arrow_table()
```

## API Test: `sql:ST_MMin`

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
Returns the minimum M-coordinate (measure) value present within the bounding box of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object containing M-dimension (measure) data.

### Input
The input must be a valid `geometry` or `geography` object that includes an M-coordinate dimension. Data can be provided via WKT, WKB, or other supported SedonaDB construction functions (e.g., `ST_GeomFromWKT`, `ST_GeogFromWKT`).

### Output
Returns `double` — The minimum M-coordinate value found in the geometry or geography's bounding box.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_MMin(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_MMin(ST_GeogFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_MMin` with a geometry or geography object that explicitly contains an M-dimension.
- Ensure the input is constructed using valid WKT/WKB formats that include the 'M' suffix or dimension.

### Prompt Snippet
```text
Use ST_MMin(arg1) to extract the minimum measure value from a geometry or geography.
```

### Common Failure Modes
- Passing a geometry or geography that lacks an M-dimension (e.g., standard 2D or 3D geometries), which may result in null or undefined behavior.
- Providing an invalid WKT string that does not conform to the expected M-coordinate format.

### Fix Code Hint
```python
# Ensure the geometry is constructed with M-dimension support
sd.sql("SELECT ST_MMin(ST_GeomFromWKT('LINESTRING M (1 2 3, 5 6 7)'))").to_arrow_table()
# __CHECK__ witness: The result should be 3.0
```

## API Test: `sql:ST_MPointFromText`

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
Constructs a `geometry` object of type `MultiPoint` from a provided Well-Known Text (WKT) string.

### Parameters
- `wkt` (`string`): A Well-Known Text representation of a MultiPoint geometry (e.g., `'MULTIPOINT ((0 0), (1 1))'`).
- `srid` (`crs`): An optional integer or string representing the Coordinate Reference System (e.g., `4326` for EPSG:4326).

### Input
The input must be a valid WKT string specifically representing a `MULTIPOINT` geometry. If the provided WKT string does not conform to the `MULTIPOINT` specification, the operation will raise an error.

### Output
Returns a `geometry` object representing the parsed MultiPoint.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_MPointFromText('MULTIPOINT ((0 0), (1 1), (2 2))', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the input string is a valid WKT `MULTIPOINT` format.
- If an SRID is required for spatial operations, provide it as the second argument.
- Use `sd.sql(...).to_arrow_table()` to execute the query and materialize the resulting geometry.

### Prompt Snippet
```text
ST_MPointFromText('MULTIPOINT ((x1 y1), (x2 y2))', [srid])
```

### Common Failure Modes
- **Invalid WKT:** Providing a WKT string that is not a `MULTIPOINT` (e.g., `POINT` or `POLYGON`) will cause the function to error.
- **Malformed WKT:** Syntax errors in the WKT string (e.g., missing parentheses or incorrect coordinate formatting) will result in a parsing failure.

### Fix Code Hint
```python
# Ensure the WKT string is correctly formatted as a MultiPoint
wkt_input = 'MULTIPOINT ((0 0), (1 1))'
query = f"SELECT ST_MPointFromText('{wkt_input}', 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_MPolyFromText`

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

## Description

Parses a WKT string and returns a geometry. Raises an error if the WKT does not represent a MultiPolygon.

## Examples

```sql
SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))');
```

With an SRID or CRS:

```sql
SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))', 4326);
```


### Goal
Constructs a `geometry` object from a Well-Known Text (WKT) string specifically representing a MultiPolygon.

### Parameters
- `wkt` (`string`): A Well-Known Text (WKT) string representing a MultiPolygon geometry.
- `srid` (`crs`): An optional integer or string representing the Coordinate Reference System (CRS) or SRID to assign to the resulting geometry.

### Input
The caller must provide a valid WKT string that strictly conforms to the `MULTIPOLYGON` format. If the input string does not represent a MultiPolygon, the operation will raise an error.

### Output
Returns a `geometry` object representing the parsed MultiPolygon.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_MPolyFromText('MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_MPolyFromText` when the input is explicitly a MultiPolygon WKT.
- If an SRID is known, provide it as the second argument to ensure correct CRS metadata.
- Ensure the WKT string is properly escaped if used within a Python f-string.

### Prompt Snippet
```text
Use ST_MPolyFromText(wkt) or ST_MPolyFromText(wkt, srid) to create a MultiPolygon geometry.
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `MULTIPOLYGON` (e.g., `POINT` or `POLYGON`) will cause the function to error.
- **Malformed WKT:** Providing a syntactically incorrect WKT string will result in a parsing error.

### Fix Code Hint
```python
# Ensure the WKT is a valid MultiPolygon
wkt_input = "MULTIPOLYGON (((0 0, 1 0, 1 1, 0 0)), ((2 2, 3 2, 3 3, 2 2)))"
query = f"SELECT ST_MPolyFromText('{wkt_input}', 4326) AS geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness: geometry materialized")
```

## API Test: `sql:ST_MakeEnvelope`

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
Constructs a rectangular `geometry` (Polygon) defined by the provided minimum and maximum coordinate bounds.

### Parameters
- `min_x` (`double`): The minimum X-coordinate (longitude or easting) of the envelope.
- `min_y` (`double`): The minimum Y-coordinate (latitude or northing) of the envelope.
- `max_x` (`double`): The maximum X-coordinate (longitude or easting) of the envelope.
- `max_y` (`double`): The maximum Y-coordinate (latitude or northing) of the envelope.
- `srid` (`crs`): An optional Coordinate Reference System identifier (e.g., `4326` or `'EPSG:4326'`) to assign to the resulting geometry.

### Input
The function accepts four `double` values representing the bounding box corners. An optional fifth argument defines the CRS. No specific file format is required as inputs are provided as scalar literals.

### Output
Returns a `geometry` object representing a rectangular Polygon covering the specified coordinate range.

### Valid Call Patterns
```python
# Example using the Python SQL connection
sd.sql("SELECT ST_MakeEnvelope(1.234, 2.234, 3.345, 3.345, 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql("...").to_arrow_table()` for execution.
- Ensure coordinate order is `min_x, min_y, max_x, max_y`.
- If an SRID is provided, ensure it is a valid integer or string CRS identifier.

### Prompt Snippet
```text
Use ST_MakeEnvelope(min_x, min_y, max_x, max_y, [srid]) to create a bounding box geometry.
```

### Common Failure Modes
- Providing coordinates where `min_x > max_x` or `min_y > max_y`, which may result in an invalid or empty geometry.
- Passing an unsupported or malformed CRS string as the fifth argument.
- Omitting the SRID when the downstream operation requires a specific coordinate system for spatial joins or transformations.

### Fix Code Hint
```python
# Ensure bounds are correct and SRID is provided if needed
query = "SELECT ST_MakeEnvelope(0.0, 0.0, 10.0, 10.0, 4326) as geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__ witness: geometry created")
```

## API Test: `sql:ST_MakeLine`

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
Creates a `LineString` by connecting two or more input `Point`, `MultiPoint`, or `LineString` geometries or geographies in the order provided.

### Parameters
- `arg1` (`geometry`): The starting geometry or geography object (e.g., `Point`, `MultiPoint`, or `LineString`).
- `arg2` (`geometry`): The subsequent geometry or geography object to connect to the previous input.

### Input
The function accepts `geometry` (planar) or `geography` (spherical) types. Inputs must be consistent in type (all `geometry` or all `geography`). If using `geography`, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A `LineString` object representing the continuous path formed by connecting the input geometries or geographies.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT ST_MakeLine(ST_Point(0, 1), ST_Point(2, 3)) as geom").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_MakeLine` to construct a `LineString` from two or more spatial inputs.
- Ensure all inputs are of the same type (either all `geometry` or all `geography`).
- The function connects inputs in the exact order they are passed as arguments.

### Prompt Snippet
```text
ST_MakeLine(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Attempting to mix `geometry` and `geography` types in the same function call will result in an error.
- **Geography Bounder:** Using `geography` types without the `s2geography` bounder registered on the session.
- **Invalid Input:** Providing non-spatial types or unsupported geometry types (e.g., `Polygon`) as inputs.

### Fix Code Hint
```python
# Ensure consistent types and valid spatial objects
sd.sql("SELECT ST_MakeLine(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(1 1)'))").to_arrow_table()
```
__CHECK__

## API Test: `sql:ST_MakeValid`

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
Creates a valid OGC-compliant representation of an input geometry that is currently considered invalid.

### Parameters
- `arg1` (`geometry`): The input geometry object to be repaired.
- `keepCollapsed` (`boolean`): Optional flag; if true, collapsed geometries are converted to a valid geometry of lower dimension; if false (default), they are converted to empty.

### Input
- Data: Standard OGC WKT or WKB geometries.
- Preconditions: The input must be a valid `geometry` type. If the geometry is already valid, it is returned as-is.

### Output
Returns `geometry` — A new, valid geometry object representing the repaired input.

### Valid Call Patterns
```python
# Basic usage
sd.sql("SELECT ST_MakeValid(ST_GeomFromWKT('LINESTRING(1 1, 1 1)'))").to_arrow_table()

# With keepCollapsed parameter
sd.sql("SELECT ST_MakeValid(ST_GeomFromWKT('POLYGON((0 0, 0 1, 1 1, 0 0))'), true)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_MakeValid` to repair geometries that fail `ST_IsValid` checks.
- Always specify `keepCollapsed` as a boolean if the desired behavior for collapsed geometries (e.g., degenerate lines or points) differs from the default `false`.

### Prompt Snippet
```text
ST_MakeValid(geometry_column, keepCollapsed=true)
```

### Common Failure Modes
- Passing non-geometry types (e.g., strings or integers) directly without using a constructor like `ST_GeomFromWKT`.
- Assuming `ST_MakeValid` will fix all topological errors; while it repairs invalid geometries, complex self-intersections may result in different geometry types than the input.

### Fix Code Hint
```python
# Ensure input is a valid geometry type before calling
sd.sql("SELECT ST_MakeValid(ST_GeomFromWKT(wkt_col)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_MaxDistance`

### Signature
```python
ST_MaxDistance(arg1: geometry, arg2: geometry) -> double | ST_MaxDistance(arg1: geography, arg2: geography) -> double
```
_Source: source/docs/reference/sql/st_maxdistance.qmd:19_

### Goal
Calculates the maximum 2D Cartesian distance (for `geometry`) or spherical distance in meters (for `geography`) between any pair of vertices across two input spatial objects.

### Parameters
- `arg1` (`geometry` | `geography`): The first spatial object (Point, LineString, Polygon, etc.).
- `arg2` (`geometry` | `geography`): The second spatial object to compare against the first.

### Input
- **Formats:** Standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** Both inputs must be of the same type (both `geometry` or both `geography`). For `geography` operations, the session must have the `s2geography` bounder registered.

### Output
Returns `double` — The maximum distance between any two vertices in the provided inputs, expressed in the units of the coordinate reference system (for `geometry`) or in meters (for `geography`).

### Valid Call Patterns
```python
# Geometry example
result = sd.sql("""
SELECT ST_MaxDistance(
    ST_GeomFromText('POINT (0 0)'),
    ST_GeomFromText('LINESTRING (3 0, 0 4)')
)
""").to_arrow_table()
print(result)
# __CHECK__ witness: 5.0

# Geography example
result = sd.sql("""
SELECT ST_MaxDistance(
    ST_GeogFromText('POLYGON ((0 0, 1 0, 0 1, 0 0))'),
    ST_GeogFromText('POLYGON ((10 10, 11 10, 10 11, 10 10))')
)
""").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (`geometry` or `geography`).
- Use `ST_GeomFromText` for planar `geometry` and `ST_GeogFromText` for spherical `geography`.
- Do not mix `geometry` and `geography` types in a single call.

### Prompt Snippet
```text
ST_MaxDistance(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in an execution error.
- **Missing Bounder:** Using `geography` types without the `s2geography` bounder registered on the session.
- **Coordinate System Mismatch:** If CRS differs, operations may fail or produce unexpected results; ensure inputs are in the same CRS or reprojected using `ST_Transform` prior to calling.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type
sd.sql("SELECT ST_MaxDistance(ST_Transform(geom1, 4326), ST_Transform(geom2, 4326)) FROM table")
```

## API Test: `sql:ST_MinimumClearance`

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
Calculates the minimum clearance of a geometry, representing the maximum distance vertices can be moved without altering the topological validity of the geometry.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Polygon, LineString) for which the minimum clearance metric is to be computed.

### Input
- **Data Format:** Standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type.

### Output
Returns `double` — The minimum clearance value. For geometries where no clearance is definable (e.g., single points), it returns `Double.MAX_VALUE`.

### Valid Call Patterns
```python
# Example using Python SQL connection
query = "SELECT ST_MinimumClearance(ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a double representing the clearance value.
```

### LLM Instruction Prompt
- Always use `ST_MinimumClearance(geometry_expression)` as the SQL function call.
- Ensure the input is a valid geometry object created via functions like `ST_GeomFromWKT` or `ST_Point`.

### Prompt Snippet
```text
SELECT ST_MinimumClearance(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Geometry:** Passing a non-geometry type or a malformed WKT string will result in a query execution error.
- **Unsupported Geometry:** While the function handles most geometries, single points or degenerate MultiPoints will return `Double.MAX_VALUE` rather than a geometric distance.

### Fix Code Hint
```python
# Ensure the input is a valid geometry before calling
sd.sql("SELECT ST_MinimumClearance(ST_MakeValid(geom)) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_MinimumClearanceLine`

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
Computes the minimum clearance distance of a geometry and returns it as a two-point `LineString` representing the segment of that distance.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., `POLYGON`, `LINESTRING`) for which the minimum clearance distance is to be calculated.

### Input
- **Data Format:** Standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the geometry is a single `Point` or contains coincident `MultiPoints`, the function will return an empty `LineString`.

### Output
Returns `geometry` — A `LineString` geometry consisting of two points that define the minimum clearance distance.

### Valid Call Patterns
```python
# Example using Python SQL connection
import sedonadb
sd = sedonadb.connect()
query = "SELECT ST_MinimumClearanceLine(ST_GeomFromWKT('POLYGON ((65 18, 62 16, 64.5 16, 62 14, 65 14, 65 18))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__ witness: The result is a LineString geometry.
```

### LLM Instruction Prompt
- Always use `ST_MinimumClearanceLine(geometry_expression)` as the SQL call pattern.
- Ensure the input is a valid `geometry` object; use `ST_GeomFromWKT` or `ST_GeomFromText` if the input is a string literal.

### Prompt Snippet
```text
SELECT ST_MinimumClearanceLine(ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **Empty Result:** Providing a single `Point` or coincident `MultiPoints` will result in an empty `LineString` rather than an error.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT strings will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure input is a valid geometry before calling
sd.sql("SELECT ST_MinimumClearanceLine(ST_MakeValid(geom)) FROM table")
```

## API Test: `sql:ST_NPoints`

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
Calculates the total count of coordinate vertices within a given `geometry` or `geography` object, including all vertices across all components of the shape.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the vertex count is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If using `geography`, ensure the session's geography bounder is registered.

### Output
Returns `integer` — The total number of coordinate points (vertices) present in the input geometry or geography.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_NPoints(ST_GeomFromWKT('LINESTRING (1 2, 3 4, 5 6)'))").to_arrow_table()
print(result)
# __CHECK__ witness: 3

# Example using geography
result = sd.sql("SELECT ST_NPoints(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
print(result)
# __CHECK__ witness: 5
```

### LLM Instruction Prompt
- Always use `ST_NPoints` to count vertices.
- Use the alias `ST_NumPoints` if required by legacy compatibility, but prefer `ST_NPoints`.
- Ensure the input is explicitly cast to `geometry` or `geography` using constructors like `ST_GeomFromWKT` or `ST_GeogFromWKT` if the input is a string.

### Prompt Snippet
```text
Use ST_NPoints(geometry_expression) to retrieve the vertex count.
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or integer) directly without a constructor.
- Geography operations failing if the `s2geography` bounder is not registered in the session.
- Providing an invalid WKT string, which will cause the constructor to fail before `ST_NPoints` is evaluated.

### Fix Code Hint
```python
# Ensure input is a valid geometry object
sd.sql("SELECT ST_NPoints(ST_GeomFromWKT(wkt_column)) FROM table_name")
```

## API Test: `sql:ST_NRings`

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

## Description

Returns the total number of rings (exterior and interior) in a polygon or multipolygon. For a simple polygon, this is 1 plus the number of holes. For a multipolygon, it is the sum of all rings in all polygons. Returns 0 for non-polygon geometry types.

## Examples

```sql
SELECT ST_NRings(ST_GeomFromText('POLYGON ((0 0, 0 1, 1 1, 1 0, 0 0))'));
```

```sql
SELECT ST_NRings(ST_GeogFromWKT('POLYGON ((0 0, 0 1, 1 1, 1 0, 0 0))'));
```


### Goal
Calculates the total count of exterior and interior rings within a polygon or multipolygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to analyze; must be a polygon or multipolygon type to return a non-zero value.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a valid polygon or multipolygon. If the input is not a polygon-based type, the function returns 0.

### Output
Returns `integer` — The total number of rings (exterior plus interior) contained within the input geometry or geography.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_NRings(ST_GeomFromText('POLYGON ((0 0, 0 1, 1 1, 1 0, 0 0))'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_NRings(ST_GeogFromWKT('POLYGON ((0 0, 0 1, 1 1, 1 0, 0 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_NRings(arg1)` as a standalone SQL function call within `sd.sql()`.
- Ensure the input is a valid polygon or multipolygon; otherwise, the result will be 0.

### Prompt Snippet
```text
SELECT ST_NRings(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Non-Polygon Types:** Passing a `Point`, `LineString`, or other non-polygon geometry will result in a return value of 0.
- **Invalid Geometries:** If the input geometry is malformed, the function may return an error or unexpected results; use `ST_MakeValid` prior to calling if validity is uncertain.

### Fix Code Hint
```python
# Ensure input is a polygon before calling
sd.sql("SELECT ST_NRings(ST_MakeValid(geom)) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_Normalize`

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
Returns the input `geometry` or `geography` object in its canonical form, primarily used for testing purposes to ensure consistent vertex ordering.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to be normalized.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` object. For `geography` types, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A canonical representation of the input object, where vertices within rings or interior rings within polygons may be rearranged.

### Valid Call Patterns
```python
# Geometry normalization
sd.sql("SELECT ST_Normalize(ST_GeomFromWKT('POLYGON ((10 10, 10 0, 0 0, 0 10, 10 10))'))").to_arrow_table()

# Geography normalization
sd.sql("SELECT ST_Normalize(ST_GeogFromWKT('POLYGON ((10 10, 10 0, 0 0, 0 10, 10 10))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Normalize` when you need to standardize the internal vertex representation of a geometry or geography for comparison or testing.
- Ensure the input is explicitly cast or created using `ST_GeomFromWKT` or `ST_GeogFromWKT` if passing raw strings.

### Prompt Snippet
```text
ST_Normalize(arg1)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a function expecting a `geography` context or vice versa without proper conversion.
- **Invalid Input:** Providing malformed WKT strings that cannot be parsed into a valid spatial object.
- **Geography Bounder:** Failure to register the `s2geography` bounder on the session when processing `geography` types.

### Fix Code Hint
```python
# Ensure valid input type and session configuration
sd.sql("SELECT ST_Normalize(ST_GeomFromWKT('...'))")
# If using geography, verify s2geography is active
```

## API Test: `sql:ST_NumGeometries`

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
Returns the count of individual geometry components within a `geometry` or `geography` collection object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object, typically a collection type such as `GEOMETRYCOLLECTION`, `MULTIPOINT`, `MULTILINESTRING`, or `MULTIPOLYGON`.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid spatial object. For single geometries (e.g., `POINT`, `POLYGON`), the function returns 1.

### Output
Returns `integer` — The total number of constituent geometries found within the input collection.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_NumGeometries(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'))").to_arrow_table()
print(result)
# __CHECK__

# Example using geography
result = sd.sql("SELECT ST_NumGeometries(ST_GeogFromWKT('MULTIPOINT ((0 0), (1 1))'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_NumGeometries` to inspect the cardinality of spatial collections.
- Ensure the input is a valid `geometry` or `geography` type; use `ST_GeomFromWKT` or `ST_GeogFromWKT` to cast strings if necessary.

### Prompt Snippet
```text
SELECT ST_NumGeometries(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Input:** Passing a non-spatial type or a null value will result in an error.
- **Type Mismatch:** Ensure the input type matches the expected `geometry` or `geography` kernel; while both are supported, ensure the input is correctly constructed for the intended spatial domain.

### Fix Code Hint
```python
# Ensure input is a valid geometry collection
sd.sql("SELECT ST_NumGeometries(ST_GeomFromWKT('MULTIPOINT ((0 0), (1 1))'))").to_arrow_table()
```

## API Test: `sql:ST_NumInteriorRings`

### Signature
```python
ST_NumInteriorRings(arg1: geometry) -> integer | ST_NumInteriorRings(arg1: geography) -> integer
```
_Source: source/docs/reference/sql/st_numinteriorrings.qmd:19_

### Goal
Returns the count of interior rings (holes) contained within a polygon geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The polygon object to inspect for interior rings.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a polygon type. For multipolygons, the function evaluates only the first polygon in the collection.

### Output
Returns `integer` — The number of interior rings (holes) found. Returns 0 for polygons without holes and `NULL` for non-polygon geometry types.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_NumInteriorRings(ST_GeomFromText('POLYGON ((0 0, 0 10, 10 10, 10 0, 0 0), (1 1, 1 2, 2 2, 2 1, 1 1))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_NumInteriorRings(ST_GeogFromWKT('POLYGON ((0 0, 0 10, 10 10, 10 0, 0 0), (1 1, 1 2, 2 2, 2 1, 1 1))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_NumInteriorRings` to count holes in polygons.
- Use the alias `ST_NumInteriorRing` if preferred.
- Ensure the input is a polygon; non-polygon types will return `NULL`.
- For multipolygons, note that the result is specific to the first polygon in the set.

### Prompt Snippet
```text
SELECT ST_NumInteriorRings(geometry_column) FROM table;
```

### Common Failure Modes
- **Non-Polygon Input:** Passing a `POINT`, `LINESTRING`, or other non-polygon type results in a `NULL` return value.
- **Multipolygon Ambiguity:** Users expecting a total count for all polygons in a `MULTIPOLYGON` will only receive the count for the first polygon.
- **Geography Bounder:** Ensure the session's geography bounder is registered if working with `geography` types.

### Fix Code Hint
```python
# Ensure input is a polygon before calling
sd.sql("SELECT ST_NumInteriorRings(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_Polygon'")
```

## API Test: `sql:ST_Overlaps`

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
Determines if two geometries share space and possess the same dimension without one being entirely contained within the other.

### Parameters
- `arg1` (`geometry`): The first geometry object to compare.
- `arg2` (`geometry`): The second geometry object to compare.

### Input
- **Data Formats:** Accepts standard OGC WKT or WKB geometries.
- **Preconditions:** Both geometries must have the same dimension (e.g., both polygons or both linestrings). If coordinate systems differ, geometries are reprojected to the target CRS; if only one side has a CRS, the operation will fail.

### Output
Returns `boolean` — `true` if the geometries overlap according to the spatial predicate definition, otherwise `false`.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
SELECT ST_Overlaps(
    ST_GeomFromWKT('POLYGON((2.5 2.5, 2.5 4.5, 4.5 4.5, 4.5 2.5, 2.5 2.5))'),
    ST_GeomFromWKT('POLYGON((4 4, 4 6, 6 6, 6 4, 4 4))')
)
""").to_arrow_table()
print(result)
# __CHECK__ witness: True
```

### LLM Instruction Prompt
- Always use `ST_GeomFromWKT` or `ST_GeomFromText` to construct geometry inputs if they are provided as strings.
- Ensure both geometries are of the same dimension before calling `ST_Overlaps`.
- Verify that both geometries share a common CRS if they are not in the same coordinate system.

### Prompt Snippet
```text
ST_Overlaps(geometry_a, geometry_b)
```

### Common Failure Modes
- **Dimension Mismatch:** Calling the function with geometries of different dimensions (e.g., a Point and a Polygon) will not return `true` as they do not share the same dimension.
- **Containment:** If one geometry is completely inside the other, `ST_Overlaps` returns `false` (as per the definition that they must not be completely contained by each other).
- **CRS Mismatch:** If only one geometry has a defined CRS, the operation will fail.

### Fix Code Hint
```python
# Ensure geometries are valid and share the same CRS before comparison
sd.sql("SELECT ST_Overlaps(ST_MakeValid(geom1), ST_MakeValid(geom2)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Perimeter`

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
Calculates the perimeter of a `geometry` (planar) or `geography` (spherical) object, specifically supporting polygonal types.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to measure. Supported types include `Polygon`, `MultiPolygon`, and `GeometryCollection` (containing polygonal geometries).

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` inputs, ensure the session's geography bounder is registered. For `geometry` inputs, the result is in the units of the coordinate reference system (CRS).

### Output
Returns `double` — The perimeter of the input object. For `geography` types, the result is the spherical approximation of the geodesic perimeter in meters.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_Perimeter(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_Perimeter(ST_GeogFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Perimeter` only for `Polygon`, `MultiPolygon`, or `GeometryCollection` types.
- If the input is a `LineString`, use `ST_Length` instead.
- Ensure the input is explicitly cast to `geometry` or `geography` using `ST_GeomFromWKT` or `ST_GeogFromWKT` if provided as a string.

### Prompt Snippet
```text
ST_Perimeter(ST_GeomFromWKT('...'))
```

### Common Failure Modes
- **Non-polygonal input:** Passing a `Point` or `LineString` will return `0`.
- **Geography missing bounder:** Geography operations require the `s2geography` bounder to be registered on the session.
- **CRS mismatch:** For `geometry` types, the result is dependent on the CRS units; ensure the CRS is appropriate for the desired measurement.

### Fix Code Hint
```python
# Ensure input is a polygon if expecting a non-zero result
sd.sql("SELECT ST_Perimeter(ST_GeomFromWKT('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()
# __CHECK__ witness: The result should be a double representing the perimeter.
```

## API Test: `sql:ST_Point`

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
Constructs a new `geometry` object of type Point from provided X and Y coordinate values, with optional spatial reference system metadata.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `srid` (`crs`): Optional coordinate reference system identifier, provided as an integer (SRID) or a string (e.g., 'EPSG:4326').

### Input
Requires two `double` values for coordinates. An optional third argument can be an integer SRID or a string representing the CRS. No specific file format is required as this is a constructor for in-memory geometry objects.

### Output
Returns a `geometry` object representing a single point in the specified or default coordinate system.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_Point(-74.0060, 40.7128) as geom").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_Point(-74.0060, 40.7128, 4326) as geom").to_arrow_table()

# Construction with CRS string
sd.sql("SELECT ST_Point(-74.0060, 40.7128, 'EPSG:3857') as geom").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` for executing queries.
- When constructing points for spatial predicates, ensure the `srid` matches the reference geometry's CRS to avoid implicit conversion errors.
- Use `ST_AsText()` if you need to verify the geometry output in a human-readable format.

### Prompt Snippet
```text
sd.sql("SELECT ST_Point(x_val, y_val, srid_val)").to_arrow_table()
```

### Common Failure Modes
- Providing non-numeric values for `x` or `y` coordinates.
- Mismatched CRS: If performing spatial operations (e.g., `ST_Intersects`) between a point and another geometry, ensure both share the same CRS or are explicitly transformed.
- Invalid CRS strings: Providing a string that does not conform to standard EPSG or OGC formats.

### Fix Code Hint
```python
# Ensure coordinates are floats and CRS is a valid string or integer
x, y = -74.0060, 40.7128
srid = 4326
query = f"SELECT ST_Point({x}, {y}, {srid}) as geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_PointFromText`

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
Constructs a `geometry` object from a Well-Known Text (WKT) string specifically representing a Point.

### Parameters
- `wkt` (`string`): A Well-Known Text string representing a point (e.g., `'POINT (30 10)'`).
- `srid` (`crs`): An optional Coordinate Reference System identifier (e.g., `4326`) to assign to the resulting geometry.

### Input
The caller must provide a valid WKT string that strictly represents a Point geometry. If the input string represents any other geometry type (e.g., `LINESTRING`, `POLYGON`), the operation will raise an error.

### Output
Returns a `geometry` object representing the point defined by the input WKT, optionally associated with the provided SRID.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_PointFromText('POINT (30 10)')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_PointFromText('POINT (30 10)', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure the `wkt` string starts with `POINT`.
- If an SRID is provided, ensure it is a valid integer or CRS identifier supported by the session.
- Use `sd.sql(...).to_arrow_table()` for executing the query in Python.

### Prompt Snippet
```text
ST_PointFromText(wkt_string, [srid_integer])
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that is not a `POINT` (e.g., `POLYGON`) will cause the function to error.
- **Malformed WKT:** Passing a string that does not conform to the OGC WKT standard for points.
- **Missing CRS:** If the downstream analysis requires a specific coordinate system, failing to provide the `srid` parameter may result in geometry operations defaulting to an undefined or planar coordinate system.

### Fix Code Hint
```python
# Ensure the WKT is a point and verify the SRID if necessary
wkt_input = "POINT (30 10)"
srid = 4326
query = f"SELECT ST_PointFromText('{wkt_input}', {srid})"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_PointFromWKB`

### Signature
```python
ST_PointFromWKB(wkb: binary) -> geometry | ST_PointFromWKB(wkb: binary, srid: crs) -> geometry
```
_Source: source/docs/reference/sql/st_pointfromwkb.qmd:19_

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

title: ST_PointFromWKB
description: Constructs a Point from Well-Known Binary (WKB), returning NULL if the input is another geometry type.
kernels:
  - returns: geometry
    args:
    - name: wkb
      type: binary
  - returns: geometry
    args:
    - name: wkb
      type: binary
    - name: srid
      type: crs
---

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
Constructs a `geometry` object of type `Point` by parsing a Well-Known Binary (WKB) representation.

### Parameters
- `wkb` (`binary`): The Well-Known Binary (WKB) byte sequence representing a point geometry.
- `srid` (`crs`): An optional Coordinate Reference System identifier (e.g., `4326`) to assign to the resulting geometry.

### Input
The input must be a valid `binary` type containing WKB data. If the binary data represents a geometry type other than a `Point`, the function returns `NULL`.

### Output
Returns a `geometry` object of type `Point`.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_PointFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'))").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_PointFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `decode(..., 'hex')` if the input WKB is provided as a hex string literal.
- Ensure the input binary represents a `Point` geometry; otherwise, the function will return `NULL`.
- Use the two-argument signature when a specific CRS is required for the geometry.

### Prompt Snippet
```text
ST_PointFromWKB(decode('<hex_string>', 'hex'), <srid_integer>)
```

### Common Failure Modes
- **Non-Point Input:** Providing WKB for a `LineString` or `Polygon` will result in a `NULL` return value.
- **Invalid Binary:** Providing malformed or non-WKB binary data will cause a runtime error.
- **Missing Decode:** Passing a raw hex string directly without the `decode(..., 'hex')` function will result in a type mismatch error.

### Fix Code Hint
```python
# Ensure the input is decoded from hex if provided as a string
query = "SELECT ST_PointFromWKB(decode('0101000000000000000000F03F0000000000000040', 'hex'), 4326)"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_PointM`

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
Constructs a 2D point geometry that includes an additional M (measure) coordinate value.

### Parameters
- `x` (`double`): The X-coordinate of the point.
- `y` (`double`): The Y-coordinate of the point.
- `m` (`double`): The M (measure) value to be associated with the point.

### Input
The function accepts three `double` precision floating-point numbers. No specific file format is required as this is a constructor function for in-memory geometry objects.

### Output
Returns a `geometry` object representing a point with X, Y, and M coordinates.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using the documented SQL pattern
result = sd.sql("SELECT ST_PointM(-64.36, 45.09, 50.0) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_PointM(x, y, m)` to create a point with a measure coordinate.
- Ensure all three arguments are provided as `double` types.
- The function returns a `geometry` type; use `ST_AsText()` if you need to verify the output as a WKT string.

### Prompt Snippet
```text
ST_PointM(x, y, m)
```

### Common Failure Modes
- Providing non-numeric types for `x`, `y`, or `m` will result in a SQL execution error.
- Attempting to use this function in a context expecting a 3D point (Z-coordinate) will fail, as this function specifically targets the M-dimension.

### Fix Code Hint
```python
# Ensure inputs are cast to float if they are coming from integer columns
sd.sql("SELECT ST_PointM(CAST(x AS DOUBLE), CAST(y AS DOUBLE), CAST(m AS DOUBLE)) FROM table")
```

## API Test: `sql:ST_PointN`

### Signature
```python
ST_PointN(arg1: geometry, n: integer) -> geometry | ST_PointN(arg1: geography, n: integer) -> geography
```
_Source: source/docs/reference/sql/st_pointn.qmd:19_

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

title: ST_PointN
description: Returns the Nth point in a linestring.
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

Return the Nth point in a single linestring. Negative values are counted backwards from the end of the LineString such that -1 is the last point. Returns NULL if there is no linestring in the geometry.

## Examples

```sql
SELECT ST_PointN(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'), 2);
```

```sql
SELECT ST_PointN(ST_GeogFromWKT('LINESTRING (0 0, 1 2, 2 4)'), 2);
```


### Goal
Extracts a specific vertex (point) from a `LineString` geometry or geography object based on its 1-based index.

### Parameters
- `arg1` (`geometry` | `geography`): The input `LineString` object from which to extract the point.
- `n` (`integer`): The 1-based index of the point to retrieve. Negative values are supported, where -1 represents the last point in the sequence.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The input must be a `LineString`. If the input is not a `LineString` (e.g., a `Point` or `Polygon`), the function returns `NULL`.

### Output
Returns `geography | geometry` — A single `Point` object corresponding to the Nth vertex of the input linestring.

### Valid Call Patterns
```python
# Extract the 2nd point from a geometry LineString
sd.sql("SELECT ST_PointN(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'), 2)").to_arrow_table()

# Extract the last point from a geography LineString
sd.sql("SELECT ST_PointN(ST_GeogFromWKT('LINESTRING (0 0, 1 2, 2 4)'), -1)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_PointN` to retrieve specific vertices from linestrings.
- Ensure the index `n` is a non-zero integer.
- Remember that indexing is 1-based; `1` is the start, `-1` is the end.

### Prompt Snippet
```text
ST_PointN(geometry_column, index_integer)
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a `Point` or `Polygon` instead of a `LineString` results in a `NULL` return value.
- **Out of Bounds:** Providing an index `n` that exceeds the number of points in the linestring will return `NULL`.
- **Zero Index:** Using `0` as the index is invalid as the function uses 1-based indexing.

### Fix Code Hint
```python
# Ensure the input is a LineString before calling ST_PointN
sd.sql("SELECT ST_PointN(geom, 1) FROM table WHERE ST_GeometryType(geom) = 'ST_LineString'")
```

## API Test: `sql:ST_PointOnSurface`

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
- `arg1` (`geometry` or `geography`): The input spatial object (e.g., Polygon, MultiPolygon) for which an interior point is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or `geography` types.
- **Preconditions:** For `geography` types, ensure the session's `s2geography` bounder is registered.

### Output
Returns `geography | geometry` — A `Point` object that is guaranteed to intersect the input geometry or geography.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_PointOnSurface(
        ST_GeogFromWKT('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0), (1 1, 1 3, 3 3, 3 1, 1 1))')
    )
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_PointOnSurface` when you need a point that is guaranteed to be contained within the input geometry, as `ST_Centroid` may return a point outside the geometry (e.g., for concave shapes).
- Ensure the input type matches the expected kernel (geometry vs. geography).

### Prompt Snippet
```text
Use ST_PointOnSurface(geometry_column) to ensure the resulting point lies on the surface of the input.
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` to a function expecting `geography` or vice versa without proper conversion.
- **Geography Bounder:** Failure to register the `s2geography` bounder when working with `geography` types.
- **Empty Input:** Passing an empty geometry or null value may result in a null output.

### Fix Code Hint
```python
# Ensure input is valid and cast if necessary
sd.sql("SELECT ST_PointOnSurface(ST_MakeValid(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_PointZ`

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
Constructs a 3D `geometry` point object by combining specified X, Y, and Z coordinate values.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `z` (`double`): The Z-coordinate (elevation or height) of the point.

### Input
Requires three `double` precision floating-point values. No specific file format is required as this is a constructor function for in-memory geometry creation.

### Output
Returns a `geometry` object representing a 3D point.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()
# Example verified from documentation
result = sd.sql("SELECT ST_PointZ(-64.36, 45.09, 10.5) as geom").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_PointZ` when you need to create a 3D point geometry from raw coordinate primitives.
- Ensure all three arguments are provided as `double` types.
- The function returns a `geometry` type; use `ST_AsText()` if you need to inspect the resulting WKT representation.

### Prompt Snippet
```text
ST_PointZ(x, y, z)
```

### Common Failure Modes
- Providing fewer than three arguments will result in a signature mismatch error.
- Passing non-numeric types (e.g., strings) for coordinates will cause a type-check failure.
- The function does not accept an SRID; if a specific coordinate reference system is required, wrap the result in `ST_SetSRID(ST_PointZ(...), srid)`.

### Fix Code Hint
```python
# Ensure all inputs are cast to float if coming from a dynamic source
sd.sql("SELECT ST_PointZ(CAST(x AS DOUBLE), CAST(y AS DOUBLE), CAST(z AS DOUBLE)) FROM table")
```

## API Test: `sql:ST_PointZM`

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
Constructs a 4-dimensional `geometry` point object containing X, Y, Z (elevation/altitude), and M (measure) coordinate values.

### Parameters
- `x` (`double`): The X-coordinate (longitude or easting) of the point.
- `y` (`double`): The Y-coordinate (latitude or northing) of the point.
- `z` (`double`): The Z-coordinate representing elevation or altitude.
- `m` (`double`): The M-coordinate representing a linear referencing measure.

### Input
Requires four `double` precision floating-point values. No specific file format is required as this is a constructor function for in-memory geometry creation.

### Output
Returns a `geometry` object representing a point with Z and M dimensions.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT ST_PointZM(-64.36, 45.09, 10.5, 50.0) as geom").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always provide exactly four `double` arguments to `ST_PointZM`.
- Use this function when creating points that require both elevation (Z) and linear referencing (M) metadata.

### Prompt Snippet
```text
ST_PointZM(x, y, z, m)
```

### Common Failure Modes
- Providing fewer than four arguments will result in a function signature mismatch error.
- Passing non-numeric types (e.g., strings) for coordinates will cause a type validation error.

### Fix Code Hint
```python
# Ensure all inputs are cast to float if they originate from string or integer sources
sd.sql("SELECT ST_PointZM(CAST(x AS DOUBLE), CAST(y AS DOUBLE), CAST(z AS DOUBLE), CAST(m AS DOUBLE)) FROM points_table")
```

## API Test: `sql:ST_Points`

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
Extracts all coordinate vertices from an input `geometry` or `geography` object and returns them as a single `MultiPoint` collection.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object (e.g., `LINESTRING`, `POLYGON`, or `GEOMETRYCOLLECTION`) from which to extract vertices.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. The function preserves existing M and Z coordinates and duplicate points present in the input.

### Output
Returns `geography | geometry` — A `MultiPoint` object containing all coordinate vertices found in the input.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Points(ST_GeomFromText('LINESTRING (2 4, 3 3, 4 2)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Points(ST_GeogFromWKT('LINESTRING (2 4, 3 3, 4 2)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Points` when you need to decompose complex geometries (like lines or polygons) into their constituent vertex points.
- Ensure the input is a valid spatial type; if the input is a string, wrap it in a constructor like `ST_GeomFromText` or `ST_GeogFromWKT`.
- The function preserves Z and M dimensions; ensure the output consumer is prepared to handle these dimensions if present.

### Prompt Snippet
```text
ST_Points(arg1)
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type (e.g., `string` or `integer`) directly without a constructor will result in a SQL execution error.
- **Invalid Geometry:** If the input geometry is malformed or invalid, the operation may fail; use `ST_MakeValid` prior to `ST_Points` if validity is uncertain.

### Fix Code Hint
```python
# Ensure input is valid geometry before extraction
sd.sql("SELECT ST_Points(ST_MakeValid(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_PolygonFromText`

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
Constructs a `geometry` object from a Well-Known Text (WKT) string specifically representing a polygon.

### Parameters
- `wkt` (`string`): A Well-Known Text string defining the polygon coordinates (e.g., `'POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))'`).
- `srid` (`crs`): An optional integer or string representing the Coordinate Reference System (e.g., `4326` for EPSG:4326).

### Input
The input must be a valid WKT string representing a polygon. If the provided WKT string does not represent a polygon, the operation will raise an error.

### Output
Returns a `geometry` object representing the polygon defined by the input WKT.

### Valid Call Patterns
```python
# Basic construction
sd.sql("SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))')").to_arrow_table()

# Construction with SRID
sd.sql("SELECT ST_PolygonFromText('POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))', 4326)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `sd.sql(...).to_arrow_table()` for execution.
- Ensure the WKT string is strictly formatted as a polygon; otherwise, the function will fail.
- If an SRID is provided, ensure it is a valid CRS identifier.

### Prompt Snippet
```text
Use ST_PolygonFromText(wkt) or ST_PolygonFromText(wkt, srid) to create a geometry object from a WKT string.
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a WKT string that represents a Point, LineString, or other non-polygon geometry will cause an error.
- **Malformed WKT:** Providing an improperly formatted WKT string (e.g., unclosed rings) will result in a parsing error.
- **Missing CRS:** If spatial operations require a specific CRS, failing to provide the `srid` parameter may lead to incorrect spatial calculations later in the pipeline.

### Fix Code Hint
```python
# Ensure the WKT is a valid polygon and verify the SRID if spatial joins are planned
wkt_input = 'POLYGON ((0 0, 4 0, 4 4, 0 4, 0 0))'
query = f"SELECT ST_PolygonFromText('{wkt_input}', 4326) as geom"
result = sd.sql(query).to_arrow_table()
print("__CHECK__")
```

## API Test: `sql:ST_Polygonize`

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
- `arg1` (`geometry`): A geometry object (typically a `MultiLineString` or `GeometryCollection`) containing the linear segments to be polygonized.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input should consist of linear components that form closed rings. The function does not require explicit closure of rings in the input data; it automatically identifies and processes closed loops into polygon boundaries.

### Output
Returns `geometry` — A polygonal geometry (or collection of polygons) derived from the input linework.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using the documented SQL pattern
query = "SELECT ST_Polygonize(ST_GeomFromText('MULTILINESTRING ((0 0, 0 10, 10 10), (10 10, 10 0, 0 0))'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Polygonize(geometry_expression)` within a `sd.sql()` call.
- Ensure the input geometry contains linear components (e.g., `LineString`, `MultiLineString`).
- Do not assume the input rings are already closed; the function handles the closure logic.

### Prompt Snippet
```text
ST_Polygonize(ST_GeomFromText('...'))
```

### Common Failure Modes
- **Non-linear input:** Passing a geometry that contains no linear components (e.g., a single `Point`) will result in an empty or invalid geometry output.
- **Disconnected linework:** If the input lines do not form complete closed loops, the function may fail to produce a valid polygon.
- **CRS Mismatch:** If the input geometry is part of a larger query involving multiple spatial objects, ensure all geometries share the same CRS to avoid unexpected behavior.

### Fix Code Hint
```python
# Ensure input is a valid collection of lines
sd.sql("SELECT ST_Polygonize(ST_GeomFromText('MULTILINESTRING ((0 0, 0 10, 10 10), (10 10, 10 0, 0 0))'))").to_arrow_table()
```

## API Test: `sql:ST_Polygonize_Agg`

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
`ST_Polygonize_Agg` is an aggregate function that constructs one or more polygons from a collection of input geometries representing the edges of those polygons.

### Parameters
- `arg1` (`geometry`): A set of geometries (typically `LINESTRING` or `MULTILINESTRING`) that form the closed boundaries of the desired polygons.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input geometries must contain linework that forms closed loops to successfully generate polygons.

### Output
Returns `geometry` — A single `POLYGON` or `MULTIPOLYGON` object representing the area enclosed by the input linework.

### Valid Call Patterns
```python
# Example using Python SQL connection
query = """
SELECT ST_Polygonize_Agg(geom) FROM (
  VALUES
    (ST_GeomFromText('LINESTRING (0 0, 0 10)')),
    (ST_GeomFromText('LINESTRING (0 10, 10 10)')),
    (ST_GeomFromText('LINESTRING (10 10, 10 0)')),
    (ST_GeomFromText('LINESTRING (10 0, 0 0)'))
) AS t(geom);
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Polygonize_Agg` only as an aggregate function within a `SELECT` statement.
- Ensure input geometries are valid linework that can form closed boundaries.
- The function is part of the SedonaDB 0.5.0 SQL operation suite.

### Prompt Snippet
```text
ST_Polygonize_Agg(geom)
```

### Common Failure Modes
- **Non-closed linework:** If the input linework does not form a closed loop, the function may return an empty geometry or null.
- **Disconnected edges:** If the input geometries do not share endpoints to form a complete polygon, the aggregation will fail to produce a polygon.
- **Type Mismatch:** Passing non-geometry types or geometries that cannot be polygonized (e.g., isolated points).

### Fix Code Hint
```python
# Ensure linework is closed before aggregation
sd.sql("SELECT ST_Polygonize_Agg(ST_MakeValid(geom)) FROM input_table").to_arrow_table()
```

## API Test: `sql:ST_ReducePrecision`

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
Reduces the coordinate precision of a `geometry` or `geography` object by snapping all vertices to a specified grid size, ensuring the resulting geometry remains valid.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be processed.
- `grid_size` (`double`): The size of the grid to which the vertices will be snapped.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` object. Z and M values are preserved but remain unaffected by the snapping process.

### Output
Returns `geography | geometry` — A new spatial object with vertices snapped to the specified grid, maintaining topological validity.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_ReducePrecision(ST_GeogFromWKT('POINT (1.123456 2.654321)'), 0.01)").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_ReducePrecision(arg1, grid_size)` to snap vertices.
- Ensure `grid_size` is a positive `double`.
- The function preserves Z and M dimensions while snapping X and Y coordinates.

### Prompt Snippet
```text
ST_ReducePrecision(geometry_column, 0.001)
```

### Common Failure Modes
- **Invalid Grid Size:** Providing a non-positive or non-numeric `grid_size` will result in an execution error.
- **Type Mismatch:** Passing an unsupported data type instead of a `geometry` or `geography` object.
- **Memory Constraints:** While designed for efficiency, extremely complex geometries with millions of vertices may hit memory limits during the unary-union-like processing.

### Fix Code Hint
```python
# Ensure the input is a valid geometry and the grid size is a positive float
sd.sql("SELECT ST_ReducePrecision(geom, 0.001) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_Relate`

### Signature
```python
ST_Relate(arg1: geometry, arg2: geometry) -> string | ST_Relate(arg1: geometry, arg2: geometry, intersectionMatrixPattern: string) -> boolean
```
_Source: source/docs/reference/sql/st_relate.qmd:18_

### Goal
Computes the DE-9IM (Dimensionally Extended 9-Intersection Model) intersection matrix string for two geometries or validates if their spatial relationship matches a specific intersection pattern.

### Parameters
- `arg1` (`geometry`): The primary geometry object to evaluate.
- `arg2` (`geometry`): The secondary geometry object to evaluate against the primary.
- `intersectionMatrixPattern` (`string`): A 9-character DE-9IM pattern string. Each character can be 0, 1, 2, T, F, or * (wildcard).

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** Geometries must be valid. If CRS differs, geometries are reprojected to the target CRS; if only one side has a CRS, the operation fails.

### Output
Returns `boolean | string` — When two arguments are provided, returns a 9-character string representing the DE-9IM matrix. When three arguments are provided, returns `true` if the relationship matches the pattern, otherwise `false`.

### Valid Call Patterns
```python
# Two-argument form returning the DE-9IM string
sd.sql("SELECT ST_Relate(ST_GeomFromText('POINT (0.5 0.5)'), ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'))").to_arrow_table()

# Three-argument form returning a boolean
sd.sql("SELECT ST_Relate(ST_GeomFromText('POINT (0.5 0.5)'), ST_GeomFromText('POLYGON ((0 0, 1 0, 1 1, 0 1, 0 0))'), '0FFFFF212')").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Relate(geom1, geom2)` to retrieve the full DE-9IM matrix string.
- Use `ST_Relate(geom1, geom2, pattern)` to perform a boolean check against a specific 9-character DE-9IM pattern.
- Ensure inputs are valid `geometry` types; use `ST_GeomFromText` or similar constructors if inputs are WKT strings.

### Prompt Snippet
```text
ST_Relate(geometry, geometry)
ST_Relate(geometry, geometry, string)
```

### Common Failure Modes
- **CRS Mismatch:** If one geometry has a CRS and the other does not, the operation will fail. Ensure both geometries share the same CRS or are in the same coordinate space.
- **Invalid Pattern:** Providing a pattern string that is not exactly 9 characters or contains invalid characters (other than 0-2, T, F, *) will result in an error.
- **Invalid Geometry:** Passing malformed WKT or invalid geometries (e.g., self-intersecting polygons where not allowed) may cause runtime exceptions.

### Fix Code Hint
```python
# Ensure geometries are valid and share the same CRS
sd.sql("""
    SELECT ST_Relate(
        ST_MakeValid(geom1), 
        ST_Transform(geom2, ST_SRID(geom1)), 
        '0FFFFF212'
    ) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `sql:ST_Reverse`

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
`sql:ST_Reverse` returns a new geometry or geography object with the order of its constituent vertices reversed.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object whose vertex sequence is to be reversed.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. For `geography` types, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A spatial object of the same type as the input, containing the same vertices but in reverse order.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_Reverse(ST_GeomFromText('LINESTRING (0 0, 1 2, 2 4)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_Reverse(ST_GeogFromWKT('LINESTRING (0 0, 1 2, 2 4)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Reverse` to flip the vertex order of a `geometry` or `geography` object.
- Ensure the input is correctly constructed using `ST_GeomFromText` or `ST_GeogFromWKT` before passing it to the function.
- The function preserves the original type (geometry remains geometry, geography remains geography).

### Prompt Snippet
```text
Use ST_Reverse(arg1) to reverse the vertex order of a geometry or geography object.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or an unsupported format that cannot be parsed into a geometry/geography.
- **Geography Bounder:** Failure to register the `s2geography` bounder on the session when performing operations on `geography` types.
- **Invalid Geometry:** Attempting to reverse a geometry that is not well-formed or is null.

### Fix Code Hint
```python
# Ensure input is valid and cast if necessary
sd.sql("SELECT ST_Reverse(ST_MakeValid(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Rotate`

### Signature
```python
ST_Rotate(arg1: geometry, rot: double) -> geometry
```
_Source: source/docs/reference/sql/st_rotate.qmd:19_

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

title: ST_Rotate
description: Rotates a geometry counter-clockwise around the Z axis by an angle in radians.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: rot
      type: double
---

## Examples

```sql
SELECT ST_Rotate(ST_GeomFromText('POINT (1 0)'), radians(90));
```


### Goal
Rotates a given geometry counter-clockwise around the Z axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Point, LineString, Polygon) to be rotated.
- `rot` (`double`): The rotation angle in radians.

### Input
- The input `geometry` must be a valid OGC WKT/WKB geometry.
- The `rot` parameter is a `double` representing the angle in radians. Use the `radians()` SQL function to convert degrees to radians if necessary.

### Output
Returns a new `geometry` object representing the input geometry rotated counter-clockwise around the Z axis.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_Rotate(ST_GeomFromText('POINT (1 0)'), radians(90))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Rotate(geometry, double)` syntax.
- Ensure the rotation angle is provided in radians; if the user provides degrees, wrap the value in the `radians()` SQL function.
- The function operates on the Z axis; ensure the input geometry is compatible with the expected spatial context.

### Prompt Snippet
```text
Use ST_Rotate(geometry, double) to rotate a geometry. Example: ST_Rotate(geom, radians(45)).
```

### Common Failure Modes
- Providing the rotation angle in degrees instead of radians (results in incorrect rotation).
- Passing a non-geometry type as the first argument.
- Attempting to rotate geometries that are not supported by the underlying engine's geometry model.

### Fix Code Hint
```python
# Ensure radians are used
sd.sql("SELECT ST_Rotate(geom, radians(angle_in_degrees)) FROM table")
```

## API Test: `sql:ST_RotateX`

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
Performs a 3D rotation of a geometry object around the X axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry to be rotated; typically a 3D geometry (e.g., `POINT Z`).
- `rot` (`double`): The rotation angle in radians.

### Input
The input must be a valid `geometry` object. If the geometry is 2D, the rotation may not produce the expected 3D transformation results. The `rot` parameter must be a `double` representing the angle in radians.

### Output
Returns a new `geometry` object representing the input geometry rotated around the X axis.

### Valid Call Patterns
```python
# Example using Python SQL connection
import sedona.db
sd = sedona.db.connect()
result = sd.sql("SELECT ST_RotateX(ST_GeomFromText('POINT Z (1 0 0)'), radians(90))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_RotateX(geometry, double)` as a standalone SQL function within `sd.sql()`.
- Ensure the input geometry is compatible with 3D operations (e.g., `POINT Z`).
- Use the `radians()` SQL function if the input angle is provided in degrees.

### Prompt Snippet
```text
SELECT ST_RotateX(geometry_column, radians(angle_in_degrees)) FROM table;
```

### Common Failure Modes
- Providing a 2D geometry that lacks Z-dimension data, which may result in unexpected behavior or identity transformations.
- Passing an angle in degrees instead of radians (use `radians()` to convert).
- Passing non-geometry types as the first argument.

### Fix Code Hint
```python
# Ensure geometry is 3D and angle is in radians
query = "SELECT ST_RotateX(ST_GeomFromText('POINT Z (1 0 0)'), radians(90))"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_RotateY`

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
Rotates a 3D geometry around the Y axis by a specified angle in radians.

### Parameters
- `arg1` (`geometry`): The input geometry to be rotated. Must be a 3D geometry (e.g., `POINT Z`) to observe rotation around the Y axis.
- `rot` (`double`): The rotation angle in radians.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input geometry should contain Z-coordinates to effectively perform a rotation around the Y axis.

### Output
Returns a `geometry` object representing the rotated input.

### Valid Call Patterns
```python
# Example using SedonaDB Python SQL connection
result = sd.sql("SELECT ST_RotateY(ST_GeomFromText('POINT Z (1 0 0)'), radians(90))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `sd.sql("...").to_arrow_table()` for executing this function.
- Ensure the input geometry is a 3D geometry (e.g., `POINT Z`) when using `ST_RotateY`.
- Use the `radians()` SQL function to convert degrees to radians if the input angle is in degrees.

### Prompt Snippet
```text
Use ST_RotateY(geometry, double) to rotate 3D geometries around the Y axis.
```

### Common Failure Modes
- Passing a 2D geometry (e.g., `POINT (1 0)`) which may not produce the expected 3D rotation effect.
- Providing the rotation angle in degrees instead of radians (use `radians()` to convert).
- Input geometry is null or invalid.

### Fix Code Hint
```python
# Ensure geometry is 3D and angle is in radians
query = "SELECT ST_RotateY(ST_GeomFromText('POINT Z (1 0 0)'), radians(90))"
sd.sql(query).to_arrow_table()
```

## API Test: `sql:ST_SRID`

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
Returns the Spatial Reference Identifier (SRID) integer associated with a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object whose coordinate reference system metadata is to be inspected.

### Input
The input must be a valid `geometry` or `geography` object. These are typically created via constructors like `ST_Point` or `ST_GeomFromText` and may have an SRID assigned via `ST_SetSRID`.

### Output
Returns `integer` — The SRID value representing the coordinate reference system of the input object.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_SRID(ST_SetSRID(ST_Point(-122.35, 47.65), 4326))").to_arrow_table()
print(result)
# __CHECK__ witness: 4326
```

### LLM Instruction Prompt
- Use `ST_SRID(arg1)` to retrieve the integer identifier for a geometry's CRS.
- Ensure the input is a valid spatial type (`geometry` or `geography`).
- Note that this function is metadata-only and does not perform coordinate transformations.

### Prompt Snippet
```text
Use ST_SRID(geometry_column) to extract the SRID integer from a spatial column in your SedonaDB query.
```

### Common Failure Modes
- Passing a non-spatial type (e.g., `string` or `float`) will result in a query execution error.
- If the geometry has no SRID explicitly set, the behavior depends on the default CRS handling of the session.
- This function does not reproject data; use `ST_Transform` if coordinate conversion is required.

### Fix Code Hint
```python
# Ensure the geometry has an SRID set before querying if it is unknown
sd.sql("SELECT ST_SRID(ST_SetSRID(geom, 4326)) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_Scale`

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
Scales a geometry object by multiplying its coordinate ordinates by specified scale factors along the X, Y, and optionally Z axes.

### Parameters
- `arg1` (`geometry`): The input geometry (e.g., Point, Polygon) to be scaled.
- `scaleX` (`double`): The factor by which to multiply the X-coordinate.
- `scaleY` (`double`): The factor by which to multiply the Y-coordinate.
- `scaleZ` (`double`): The factor by which to multiply the Z-coordinate (optional, for 3D scaling).

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the geometry is 2D, only `scaleX` and `scaleY` are applicable.

### Output
Returns `geometry` — A new geometry object with coordinates transformed by the provided scale factors.

### Valid Call Patterns
```python
# 2D Scaling
sd.sql("SELECT ST_Scale(ST_GeomFromText('POINT (1 2)'), 2.0, 3.0)").to_arrow_table()

# 3D Scaling (Inferred from signature)
sd.sql("SELECT ST_Scale(ST_GeomFromText('POINT Z (1 2 3)'), 2.0, 3.0, 4.0)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Scale` to perform linear coordinate scaling.
- Ensure the number of scale factors matches the dimensionality of the input geometry (2 factors for 2D, 3 for 3D).
- Always wrap the geometry input in a constructor like `ST_GeomFromText` if providing a literal.

### Prompt Snippet
```text
ST_Scale(geometry_expression, scaleX, scaleY)
ST_Scale(geometry_expression, scaleX, scaleY, scaleZ)
```

### Common Failure Modes
- Providing only two scale factors for a 3D geometry may result in unexpected behavior or errors depending on the implementation's strictness.
- Passing non-numeric values for scale factors will cause a SQL execution error.
- Attempting to scale a `geography` type directly; ensure the input is a `geometry` type.

### Fix Code Hint
```python
# Ensure input is geometry and scale factors are doubles
query = "SELECT ST_Scale(ST_GeomFromText('POINT (1 2)'), 2.0, 3.0) as scaled_geom"
result = sd.sql(query).to_arrow_table()
print("Result materialized: __CHECK__")
```

## API Test: `sql:ST_Segmentize`

### Signature
```python
ST_Segmentize(arg1: geometry, max_segment_length: double) -> geometry | ST_Segmentize(arg1: geography, max_segment_length: double) -> geography
```
_Source: source/docs/reference/sql/st_segmentize.qmd:19_

### Goal
Densifies a `geometry` or `geography` object by inserting intermediate points along segments that exceed a specified maximum length.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be densified.
- `max_segment_length` (`double`): The maximum allowed length for any segment in the resulting geometry; for `geometry` types, this uses planar distance, while for `geography` types, it uses the spherical approximation of geodesic distance in meters.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` inputs, ensure the session's geography bounder is registered. The input must be a valid `geometry` or `geography` object.

### Output
Returns a `geometry` or `geography` object (matching the input type) with additional vertices added such that no segment exceeds the `max_segment_length`.

### Valid Call Patterns
```python
# Geometry densification
sd.sql("SELECT ST_Segmentize(ST_GeomFromText('LINESTRING (0 0, 0 10)'), 2)").to_arrow_table()

# Geography densification
sd.sql("SELECT ST_Segmentize(ST_GeogFromWKT('LINESTRING (0 0, 0 10)'), 100000)").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_Segmentize(arg1, max_segment_length)` to densify geometries.
- Ensure the input type (`geometry` vs `geography`) matches the intended distance calculation (planar vs spherical).
- The `max_segment_length` must be a positive `double`.

### Prompt Snippet
```text
ST_Segmentize(geometry_or_geography_column, max_length_double)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` when a `geography` calculation is required (or vice versa) will result in incorrect distance logic.
- **Invalid Input:** Providing a non-spatial type or an invalid WKT string will cause the query to fail.
- **Geography Bounder:** Failure to register the `s2geography` bounder on the session will cause `geography` operations to fail.

### Fix Code Hint
```python
# Ensure input is cast to the correct type if necessary
sd.sql("SELECT ST_Segmentize(ST_GeomFromText('LINESTRING (0 0, 0 10)'), 2.0)").to_arrow_table()
# __CHECK__ witness: ST_Segmentize successfully densified the linestring.
```

## API Test: `sql:ST_SetCRS`

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
- `target_crs` (`string`): The CRS definition to apply, provided as an authority code (e.g., `'EPSG:4326'`), PROJJSON, or WKT string.

### Input
- **Data:** A valid `geometry` or `geography` object.
- **Preconditions:** The input must be a valid spatial object. This operation is metadata-only; it does not perform coordinate reprojection.

### Output
Returns `geography | geometry` — The input object with the specified CRS metadata attached.

### Valid Call Patterns
```python
# Example using Python SQL connection
sd.sql("SELECT ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4269')").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_SetCRS` only when you need to define the CRS of a geometry that lacks one or to override existing metadata.
- Do not use `ST_SetCRS` for coordinate transformation; use `ST_Transform` if the goal is to change the coordinate values to a new system.
- Ensure the `target_crs` string is a valid authority code, PROJJSON, or WKT.

### Prompt Snippet
```text
ST_SetCRS(geometry_column, 'EPSG:4326')
```

### Common Failure Modes
- **Coordinate Transformation Expectation:** Users often mistakenly expect `ST_SetCRS` to reproject data; it does not. If coordinates are in the wrong system, the data will be spatially incorrect after this call.
- **Invalid CRS String:** Providing a malformed CRS string (e.g., an unsupported authority code) will result in an execution error.
- **Geography/Geometry Mismatch:** Ensure the input type matches the intended kernel; while both are supported, mixing them in complex pipelines without explicit casting can lead to unexpected behavior.

### Fix Code Hint
```python
# If you need to change the actual coordinates, use ST_Transform instead:
sd.sql("SELECT ST_Transform(ST_SetCRS(geom, 'EPSG:4326'), 'EPSG:26910') FROM table").to_arrow_table()
```

## API Test: `sql:ST_SetSRID`

### Signature
```python
ST_SetSRID(arg1: geometry, srid: integer) -> geometry | ST_SetSRID(arg1: geography, srid: integer) -> geography
```
_Source: source/docs/reference/sql/st_setsrid.qmd:19_

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

title: ST_SetSRID
description: Sets the SRID (spatial reference identifier) for a geometry.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: srid
      type: integer
  - returns: geography
    args:
    - geography
    - name: srid
      type: integer
---

## Description

Sets the SRID (spatial reference system identifier) of the geometry. This is metadata only - no coordinate transformation occurs. Use ST_Transform to actually reproject coordinates.

See [Coordinate Reference Systems](../../concepts/crs.md) for how SedonaDB represents and compares CRS.

## Examples

```sql
SELECT ST_SetSRID(ST_Point(-122.35, 47.65), 4326);
```


### Goal
Assigns a specific Spatial Reference System Identifier (SRID) to a geometry or geography object as metadata without modifying the underlying coordinate values.

### Parameters
- `arg1` (`geometry`): The input geometry or geography object to be tagged with an SRID.
- `srid` (`integer`): The integer identifier representing the target spatial reference system (e.g., 4326 for WGS 84).

### Input
The input must be a valid `geometry` or `geography` object. This operation is metadata-only; it does not perform coordinate reprojection. If actual coordinate transformation is required, use `ST_Transform` instead.

### Output
Returns a `geometry` or `geography` object with the updated SRID metadata.

### Valid Call Patterns
```python
# Example using SedonaDB SQL connection
result = sd.sql("SELECT ST_SetSRID(ST_Point(-122.35, 47.65), 4326) as geom").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Use `ST_SetSRID` only when you need to define the CRS metadata for a geometry that lacks it or needs to be explicitly set.
- Do not use this function to reproject data; if the coordinates are in a different system than the target SRID, use `ST_Transform` after setting the correct initial SRID.

### Prompt Snippet
```text
SELECT ST_SetSRID(geometry_column, 4326) FROM table_name;
```

### Common Failure Modes
- **Coordinate Mismatch:** Using `ST_SetSRID` to "convert" coordinates between systems; the function only updates the metadata tag, which will lead to incorrect spatial analysis if the coordinates do not match the assigned SRID.
- **Invalid SRID:** Providing an SRID that is not supported or recognized by the system.

### Fix Code Hint
```python
# If you need to change the coordinate system, use ST_Transform:
# sd.sql("SELECT ST_Transform(ST_SetSRID(geom, 4326), 3857) FROM table")
```

## API Test: `sql:ST_Simplify`

### Signature
```python
ST_Simplify(arg1: geometry, tolerance: double) -> geometry | ST_Simplify(arg1: geography, tolerance: double) -> geography
```
_Source: source/docs/reference/sql/st_simplify.qmd:19_

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

title: ST_Simplify
description: Simplifies an input geometry or geography using the Douglas-Peucker algorithm.
kernels:
  - returns: geometry
    args:
    - geometry
    - name: tolerance
      type: double
  - returns: geography
    args:
    - geography
    - name: tolerance
      type: double
---

## Description

Simplifies a geometry or geography using the Douglas-Peucker algorithm. The `tolerance` parameter controls the degree of simplification: higher values produce simpler geometries. For geography, the tolerance is specified in meters and a different algorithm is used.

This function may produce invalid geometries; use `ST_SimplifyPreserveTopology` when validity must be maintained.

## Examples

```sql
SELECT ST_Simplify(
    ST_GeomFromWKT('LINESTRING (0 0, 0.5 0.1, 1 0, 1.5 0.1, 2 0)'),
    0.2
);
```

```sql
SELECT ST_Simplify(
    ST_GeogFromWKT('LINESTRING (0 0, 0.5 0.1, 1 0, 1.5 0.1, 2 0)'),
    50000
);
```


### Goal
Reduces the number of vertices in a `geometry` or `geography` object using the Douglas-Peucker algorithm to simplify the shape while maintaining its general structure.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to be simplified.
- `tolerance` (`double`): The distance threshold for simplification. For `geometry`, this is in the units of the coordinate system; for `geography`, this is in meters.

### Input
- **Data:** Standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. If using `geography`, ensure the session's geography bounder is registered.

### Output
Returns `geography | geometry` — A simplified version of the input object with fewer vertices.

### Valid Call Patterns
```python
# Geometry simplification
sd.sql("SELECT ST_Simplify(ST_GeomFromWKT('LINESTRING (0 0, 0.5 0.1, 1 0, 1.5 0.1, 2 0)'), 0.2)").to_arrow_table()

# Geography simplification
sd.sql("SELECT ST_Simplify(ST_GeogFromWKT('LINESTRING (0 0, 0.5 0.1, 1 0, 1.5 0.1, 2 0)'), 50000)").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Simplify` for Douglas-Peucker simplification.
- If the input is `geography`, ensure the `tolerance` is provided in meters.
- Note that this function does not guarantee topological validity; suggest `ST_SimplifyPreserveTopology` if validity is a requirement.

### Prompt Snippet
```text
ST_Simplify(geometry_or_geography_column, tolerance_double)
```

### Common Failure Modes
- **Invalid Geometry:** The output may result in self-intersecting or otherwise invalid geometries.
- **Geography Bounder:** Failure to register the `s2geography` bounder on the session will cause errors for `geography` inputs.
- **Tolerance Mismatch:** Using a `geometry` tolerance (e.g., 0.2) for a `geography` object (which expects meters) will result in unexpected simplification levels.

### Fix Code Hint
```python
# If ST_Simplify produces invalid results, switch to ST_SimplifyPreserveTopology:
sd.sql("SELECT ST_SimplifyPreserveTopology(geom, 0.2) FROM table").to_arrow_table()
```

## API Test: `sql:ST_SimplifyPreserveTopology`

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
- `tolerance` (`double`): The distance tolerance for simplification; vertices closer than this distance may be removed or collapsed.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. If the input is invalid, consider using `ST_MakeValid` prior to simplification.

### Output
Returns `geometry` — A simplified version of the input geometry that preserves the original dimension, number of components, and topological relationships.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_SimplifyPreserveTopology(
        ST_GeomFromText('POLYGON((8 25, 28 22, 28 20, 15 11, 33 3, 56 30, 46 33,46 34, 47 44, 35 36, 45 33, 43 19, 29 21, 29 22,35 26, 24 39, 8 25))'),
        10.0
    )
""").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_SimplifyPreserveTopology` when simplification is required but the resulting geometry must remain valid and topologically consistent.
- Ensure the `tolerance` parameter is provided as a `double`.

### Prompt Snippet
```text
ST_SimplifyPreserveTopology(geometry_column, tolerance_value)
```

### Common Failure Modes
- **Invalid Input:** Passing a geometry that is already topologically invalid may result in errors or unexpected output; use `ST_MakeValid` first.
- **Tolerance Mismatch:** Using a `tolerance` value that is too large may collapse the geometry into a lower-dimensional object or an empty geometry.
- **Type Mismatch:** Passing non-geometry types or incorrect parameter types (e.g., string instead of double for tolerance) will cause execution failure.

### Fix Code Hint
```python
# Ensure geometry is valid before simplifying
sd.sql("SELECT ST_SimplifyPreserveTopology(ST_MakeValid(geom), 0.5) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Snap`

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
`sql:ST_Snap` modifies the vertices of an input geometry by snapping them to the nearest vertices or segments of a reference geometry if they fall within a specified distance threshold.

### Parameters
- `arg1` (`geometry`): The input geometry whose vertices are to be snapped.
- `arg2` (`geometry`): The reference geometry to which the input geometry vertices will be snapped.
- `tolerance` (`double`): The maximum distance within which a vertex of `arg1` will be snapped to the nearest point on `arg2`.

### Input
- **Data Formats:** Accepts standard OGC WKT or WKB geometries.
- **Preconditions:** Both geometries should ideally share the same Coordinate Reference System (CRS). If CRS differs, SedonaDB may attempt to reproject, but operations fail if only one input has a defined CRS.

### Output
Returns `geometry` — A new geometry object with vertices adjusted to align with the reference geometry where the distance is within the specified tolerance.

### Valid Call Patterns
```python
# Example using the SedonaDB Python SQL connection
query = """
SELECT ST_Snap(
    ST_GeomFromWKT('POLYGON((0 0, 1 0, 1 1, 0 1, 0 0))'),
    ST_GeomFromWKT('LINESTRING(0 0.1, 1 0.1)'),
    0.2
) as snapped_geom
"""
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `sd.sql(query).to_arrow_table()` to execute the operation.
- Ensure `arg1` and `arg2` are valid geometry objects (e.g., created via `ST_GeomFromWKT` or `ST_Point`).
- The `tolerance` must be a `double` representing the snapping threshold.

### Prompt Snippet
```text
Use ST_Snap(arg1, arg2, tolerance) to align geometry vertices. Ensure both geometries are in the same CRS.
```

### Common Failure Modes
- **CRS Mismatch:** Operation fails if one geometry has a CRS and the other does not, or if they are incompatible.
- **Invalid Geometry:** Providing non-geometry types or malformed WKT strings will result in a SQL execution error.
- **Tolerance Value:** A negative or zero tolerance may result in no snapping or unexpected behavior depending on the underlying implementation.

### Fix Code Hint
```python
# Ensure both geometries are in the same CRS before snapping
sd.sql("""
SELECT ST_Snap(
    ST_Transform(geom1, 'EPSG:4326'),
    ST_Transform(geom2, 'EPSG:4326'),
    0.05
) FROM my_table
""").to_arrow_table()
```

## API Test: `sql:ST_StartPoint`

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
Extracts the first coordinate point from a `LINESTRING` geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input linestring geometry or geography from which to extract the starting point.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a linestring type. If the input is a collection or a different geometry type, the operation may not return the expected start point.

### Output
Returns a `geometry` or `geography` object representing the first point of the input linestring.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Example using SQL query via Python
query = "SELECT ST_StartPoint(ST_GeomFromText('LINESTRING(100 150, 50 60, 70 80, 160 170)'))"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_StartPoint(arg1)` to retrieve the first vertex of a linestring.
- Ensure the input is a valid linestring geometry or geography.
- The function preserves the type (geometry or geography) of the input.

### Prompt Snippet
```text
ST_StartPoint(geometry_column)
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-linestring geometry (e.g., a `POINT` or `POLYGON`) may result in an error or unexpected behavior.
- **Null Input:** Passing a `NULL` geometry will return `NULL`.
- **Geography/Geometry Mismatch:** Ensure the input type matches the expected kernel; if using geography, ensure the session's geography bounder is registered.

### Fix Code Hint
```python
# Ensure input is a linestring before calling
sd.sql("SELECT ST_StartPoint(geom) FROM table WHERE ST_GeometryType(geom) = 'ST_LineString'")
```

## API Test: `sql:ST_SymDifference`

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
Computes the symmetric difference of two spatial objects, returning a new geometry or geography containing only the portions of the inputs that do not overlap.

### Parameters
- `arg1` (`geometry`): The first input geometry or geography object.
- `arg2` (`geometry`): The second input geometry or geography object.

### Input
- **Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** Both inputs must be of the same type (both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered.

### Output
Returns a `geometry` or `geography` object representing the spatial symmetric difference (the union of the inputs minus their intersection).

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_SymDifference(ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'), ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_SymDifference(ST_GeogFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'), ST_GeogFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (geometry or geography).
- Use `ST_GeomFromWKT` or `ST_GeogFromWKT` to construct inputs if they are provided as strings.
- Do not mix geometry and geography types in a single call.

### Prompt Snippet
```text
ST_SymDifference(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in an execution error.
- **Missing Bounder:** Using `geography` types without the `s2geography` bounder registered on the session.
- **Invalid Geometries:** Input geometries that are not OGC-compliant may cause errors; use `ST_MakeValid` prior to processing if necessary.

### Fix Code Hint
```python
# Ensure both inputs are cast to the same type
sd.sql("SELECT ST_SymDifference(ST_GeomFromWKT(wkt1), ST_GeomFromWKT(wkt2))")
```

## API Test: `sql:ST_TessellateGeog`

### Signature
```python
ST_TessellateGeog(arg1: geometry, tolerance: double) -> geography
```
_Source: source/docs/reference/sql/st_tessellategeog.qmd:19_

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

title: ST_TessellateGeog
description: Converts a geometry to geography, densifying edges to approximate planar lines as geodesic segments.
kernels:
  - returns: geography
    args:
    - geometry
    - name: tolerance
      type: double
---

## Description

Converts a geometry to a geography while tessellating edges using spherical (geodesic)
interpolation. The tolerance parameter specifies the maximum deviation in meters
between the original edge and the tessellated approximation. This is useful when converting
planar geometries to geographies where edges need to follow geodesic paths on the sphere.

## Examples

```sql
SELECT ST_TessellateGeog(
    ST_GeomFromText('LINESTRING (-10 50, 10 50)'),
    1000
);
```


### Goal
Converts a planar `geometry` object into a spherical `geography` object by densifying edges to approximate geodesic segments within a specified distance tolerance.

### Parameters
- `arg1` (`geometry`): The input planar geometry (e.g., `LINESTRING`, `POLYGON`) to be converted.
- `tolerance` (`double`): The maximum allowed deviation in meters between the original planar edge and the resulting geodesic approximation.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The input must be a valid `geometry` type. The operation assumes the input is planar; the resulting `geography` is suitable for spherical calculations.

### Output
Returns `geography` — A spherical representation of the input geometry where edges are interpolated as geodesic paths.

### Valid Call Patterns
```python
# Example using Python SQL connection
query = "SELECT ST_TessellateGeog(ST_GeomFromText('LINESTRING (-10 50, 10 50)'), 1000.0)"
result = sd.sql(query).to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_TessellateGeog(geometry, double)` as the function signature.
- Ensure the `tolerance` is provided as a `double` representing meters.
- Use this function when transitioning from planar spatial analysis to spherical/geodesic analysis to ensure edge accuracy.

### Prompt Snippet
```text
ST_TessellateGeog(geometry_column, 1000.0)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geography` type as `arg1` instead of a `geometry` type.
- **Invalid Tolerance:** Providing a non-numeric or negative value for `tolerance`.
- **Geometry Validity:** Input geometries that are invalid (e.g., self-intersecting) may cause errors during the tessellation process; use `ST_MakeValid` first if necessary.

### Fix Code Hint
```python
# Ensure input is valid geometry and tolerance is a positive double
sd.sql("SELECT ST_TessellateGeog(ST_MakeValid(geom), 500.0) FROM table").to_arrow_table()
```

## API Test: `sql:ST_TessellateGeom`

### Signature
```python
ST_TessellateGeom(arg1: geography, tolerance: double) -> geometry
```
_Source: source/docs/reference/sql/st_tessellategeom.qmd:19_

### Goal
Converts a `geography` object to a `geometry` object by densifying edges using geodesic interpolation to approximate spherical segments as planar lines.

### Parameters
- `arg1` (`geography`): The input spherical geography object to be converted.
- `tolerance` (`double`): The maximum allowed deviation in meters between the original spherical edge and the resulting planar approximation.

### Input
- **Data:** Requires a valid `geography` object (e.g., created via `ST_GeogFromText`).
- **Preconditions:** The session must have the `s2geography` bounder registered.
- **Formats:** Standard OGC WKT/WKB representations for geography.

### Output
Returns `geometry` — A planar geometry object where edges have been densified to maintain the shape of the original geodesic segments within the specified tolerance.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_TessellateGeom(
        ST_GeogFromText('LINESTRING (-10 50, 10 50)'),
        1000.0
    )
""").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always use `ST_TessellateGeom(geography, double)` as a standalone SQL function call.
- Ensure the first argument is a valid `geography` type.
- Use a `double` for the tolerance parameter representing meters.

### Prompt Snippet
```text
SELECT ST_TessellateGeom(ST_GeogFromText('...'), 1000.0);
```

### Common Failure Modes
- **Antarctica/Full-Range Polygons:** The function does not produce valid geometry for polygon rings that span the full longitude range (e.g., Antarctica).
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered on the session.
- **Type Mismatch:** Passing a `geometry` type instead of a `geography` type will result in a signature mismatch error.

### Fix Code Hint
```python
# Ensure input is cast to geography if it is currently a geometry
sd.sql("SELECT ST_TessellateGeom(ST_ToGeography(geom), 1000.0) FROM table")
```

## API Test: `sql:ST_ToGeography`

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
Converts a planar `geometry` object into a spherical `geography` object by updating its edge interpretation metadata.

### Parameters
- `arg1` (`geometry`): The input geometry (or geography) to be converted.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** The session must have the `s2geography` bounder registered to perform geography-based operations.

### Output
Returns `geography` — A spherical representation of the input geometry, maintaining the original coordinate data.

### Valid Call Patterns
```python
# Example using the Python SQL connection
result = sd.sql("SELECT ST_ToGeography(ST_GeomFromText('POINT (-122.35 47.62)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_ToGeography` when you need to perform spherical calculations (e.g., accurate distance or area on a globe) on data currently stored as planar `geometry`.
- Do not attempt to modify coordinate values; this is a metadata-only operation.
- Ensure the input is a valid geometry object.

### Prompt Snippet
```text
ST_ToGeography(geometry_column)
```

### Common Failure Modes
- **Missing Bounder:** Failure to register the `s2geography` bounder on the session will cause geography operations to fail.
- **Invalid Geometry:** Passing non-geometry types or malformed WKT/WKB will result in a runtime error.
- **Coordinate System:** While this changes edge interpretation, ensure the underlying coordinates are appropriate for spherical interpretation (e.g., longitude/latitude).

### Fix Code Hint
```python
# Ensure the session is configured for geography operations
sd.sql("SELECT ST_ToGeography(geom) FROM my_table")
```

## API Test: `sql:ST_ToGeometry`

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
Converts a `geography` object to a `geometry` object by reinterpreting its spherical edges as planar, facilitating planar spatial analysis on data originally defined in a spherical coordinate system.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object to be converted. If the input is already a `geometry`, it is returned unchanged.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The operation is metadata-only and does not modify coordinate data. No specific file format is required, but inputs must be valid SedonaDB spatial objects.

### Output
Returns `geometry` — A planar representation of the input spatial object.

### Valid Call Patterns
```python
# Example using Python SQL connection
import sedonadb
sd = sedonadb.connect()
result = sd.sql("SELECT ST_ToGeometry(ST_GeogFromText('POINT (-122.35 47.62)'))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_ToGeometry(arg1)` to cast a `geography` type to `geometry` when planar spatial operations are required.
- Do not attempt to use this function to reproject coordinates; it is a metadata-only operation.
- If the input is already a `geometry`, the function will return it as-is.

### Prompt Snippet
```text
Use ST_ToGeometry(geography_column) to convert spherical geography data to planar geometry for planar-based spatial analysis.
```

### Common Failure Modes
- **Type Mismatch:** Passing an object that is not a valid `geometry` or `geography` type.
- **Assumption of Reprojection:** Expecting the function to change coordinate values; it only changes the edge interpretation metadata.

### Fix Code Hint
```python
# Ensure the input is a valid geography object before conversion
sd.sql("SELECT ST_ToGeometry(ST_GeogFromText('POINT (-122.35 47.62)'))").to_arrow_table()
```

## API Test: `sql:ST_Touches`

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
Determines if two geometries share at least one boundary point while maintaining disjoint interiors, a fundamental spatial predicate for topological analysis in SedonaDB.

### Parameters
- `arg1` (`geometry`): The first geometry object (e.g., Point, LineString, or Polygon) to evaluate.
- `arg2` (`geometry`): The second geometry object to compare against the first.

### Input
- **Data Formats:** Accepts standard OGC WKT, WKB, or EWKT/EWKB geometry representations.
- **Preconditions:** Geometries must be valid; use `ST_MakeValid` if input data is potentially malformed. If geometries have different Coordinate Reference Systems (CRS), they must be transformed to a common CRS using `ST_Transform` before comparison.

### Output
Returns `boolean` — `true` if the geometries touch (share boundary points but not interior points), otherwise `false`.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("""
    SELECT ST_Touches(
        ST_GeomFromText('POINT(1 2)'),
        ST_GeomFromText('POLYGON((1 1, 1 4, 4 4, 4 1, 1 1))')
    ) as touches
""").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_GeomFromText` or similar constructors to wrap WKT strings when passing geometry literals to `ST_Touches`.
- Ensure both arguments are valid `geometry` types; do not pass `geography` types directly without conversion if the session expects planar geometry.

### Prompt Snippet
```text
ST_Touches(ST_GeomFromText('...'), ST_GeomFromText('...'))
```

### Common Failure Modes
- **CRS Mismatch:** If inputs are in different coordinate systems, the operation may fail or return incorrect results; ensure consistent CRS.
- **Invalid Geometries:** Passing self-intersecting or otherwise invalid polygons can lead to unexpected predicate results; verify with `ST_IsValid`.
- **Interior Overlap:** If the geometries share any interior points (e.g., overlapping polygons), the function returns `false` even if they share boundary points.

### Fix Code Hint
```python
# Ensure geometries are valid and in the same CRS before checking
sd.sql("""
    SELECT ST_Touches(
        ST_MakeValid(geom1), 
        ST_Transform(geom2, 4326)
    ) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `sql:ST_Transform`

### Signature
```python
ST_Transform(arg1: geometry, target_crs: string) -> geometry | ST_Transform(arg1: geometry, source_crs: string, target_crs: string) -> geometry | ST_Transform(arg1: geography, target_crs: string) -> geography | ST_Transform(arg1: geography, source_crs: string, target_crs: string) -> geography
```
_Source: source/docs/reference/sql/st_transform.qmd:19_

### Goal
`sql:ST_Transform` reprojects a geometry or geography object from its current coordinate reference system (CRS) to a specified target CRS.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object to be transformed.
- `source_crs` (`string`): Optional; the EPSG code or PROJ string defining the source CRS. This overrides the geometry's existing SRID.
- `target_crs` (`string`): The destination EPSG code (e.g., 'EPSG:4326') or PROJ string for the transformation.

### Input
- **Data:** Standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must have a defined CRS (or a `source_crs` must be provided). For `geography` inputs, the `target_crs` must also be a geographic (longitude/latitude) CRS.

### Output
Returns `geography | geometry` — A new spatial object with coordinates transformed to the target CRS.

### Valid Call Patterns
```python
# Standard transformation with 2 arguments
sd.sql("SELECT ST_Transform(ST_SetCRS(ST_Point(-122.35, 47.65), 'EPSG:4326'), 'EPSG:26910')").to_arrow_table()

# Transformation with explicit source CRS
sd.sql("SELECT ST_Transform(ST_Point(-122.35, 47.65), 'EPSG:4326', 'EPSG:26910')").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Transform(geometry, target_crs)` or `ST_Transform(geometry, source_crs, target_crs)`.
- Ensure `target_crs` is a valid EPSG string or PROJ string.
- If the input is `geography`, ensure the `target_crs` is also geographic.
- Use `sd.sql(query).to_arrow_table()` for execution.

### Prompt Snippet
```text
Use ST_Transform(arg1, target_crs) to reproject geometries. If the input lacks an SRID, provide the source_crs as the second argument.
```

### Common Failure Modes
- **Missing CRS:** Attempting to transform a geometry that lacks an SRID without providing a `source_crs`.
- **Geography Constraint:** Providing a non-geographic `target_crs` when the input is of type `geography`.
- **Invalid CRS String:** Providing an unsupported or malformed EPSG/PROJ string.

### Fix Code Hint
```python
# Ensure the geometry has a CRS before transforming
sd.sql("SELECT ST_Transform(ST_SetCRS(geom, 'EPSG:4326'), 'EPSG:3857') FROM table").to_arrow_table()
```

## API Test: `sql:ST_Translate`

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
`sql:ST_Translate` shifts the coordinates of a geometry object by specified offsets along the X, Y, and optionally Z axes.

### Parameters
- `arg1` (`geometry`): The input geometry to be translated.
- `deltaX` (`double`): The distance to shift the geometry along the X-axis.
- `deltaY` (`double`): The distance to shift the geometry along the Y-axis.
- `deltaZ` (`double`): The distance to shift the geometry along the Z-axis (optional).

### Input
The input must be a valid `geometry` type (e.g., created via `ST_GeomFromText` or `ST_Point`). No specific file format is required as long as the object is materialized as a `geometry` in the SedonaDB session.

### Output
Returns a new `geometry` object with coordinates shifted by the provided delta values.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# 2D Translation
result = sd.sql("SELECT ST_Translate(ST_GeomFromText('POINT(1 3)'), 1, 2)").to_arrow_table()
print(result)
# __CHECK__

# 3D Translation
result_3d = sd.sql("SELECT ST_Translate(ST_GeomFromText('POINT Z (1 3 5)'), 1, 2, 10)").to_arrow_table()
print(result_3d)
# __CHECK__
```

### LLM Instruction Prompt
- Use `ST_Translate` to shift geometries.
- Ensure the input geometry is valid.
- Use the 3-argument signature for 2D geometries and the 4-argument signature for 3D geometries.

### Prompt Snippet
```text
Use ST_Translate(geometry, deltaX, deltaY) for 2D or ST_Translate(geometry, deltaX, deltaY, deltaZ) for 3D.
```

### Common Failure Modes
- Providing an incorrect number of arguments for the geometry's dimensionality.
- Passing non-numeric values for delta parameters.
- Attempting to translate a `geography` type directly if the function expects a `geometry` (ensure conversion if necessary).

### Fix Code Hint
```python
# Ensure geometry is valid before translation
sd.sql("SELECT ST_Translate(ST_MakeValid(geom), 1.0, 2.0) FROM table")
```

## API Test: `sql:ST_UnaryUnion`

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
Computes the geometric union of all components within a single input geometry, effectively dissolving boundaries between overlapping or adjacent parts.

### Parameters
- `arg1` (`geometry`): A single geometry object containing multiple components, such as a `GEOMETRYCOLLECTION` or a `MULTI*` geometry (e.g., `MULTIPOLYGON`, `MULTILINESTRING`).

### Input
- **Data Format:** Standard OGC WKT or WKB geometries.
- **Preconditions:** The input must be a valid geometry object. If the input is a collection of separate rows, use an aggregate function like `ST_Union_Agg` instead of `ST_UnaryUnion`.

### Output
Returns `geometry` — A single, merged geometry representing the union of all input components.

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_UnaryUnion(ST_GeomFromWKT('MULTIPOLYGON(((0 0, 10 0, 10 10, 0 10, 0 0)), ((10 0, 20 0, 20 10, 10 10, 10 0)))'))").to_arrow_table()
print(result)
# __CHECK__ witness: The result should be a single polygon representing the merged area.
```

### LLM Instruction Prompt
- Use `ST_UnaryUnion` only when the input is a single geometry object containing multiple parts.
- Do not use this function for aggregating multiple rows; use `ST_Union_Agg` for row-wise aggregation.
- Ensure the input is a valid `geometry` type.

### Prompt Snippet
```text
ST_UnaryUnion(geometry_column)
```

### Common Failure Modes
- **Input Type Mismatch:** Passing a non-geometry type or a null value will result in an execution error.
- **Incorrect Usage:** Attempting to use `ST_UnaryUnion` on a set of rows instead of a single geometry object (use `ST_Union_Agg` for row-based operations).
- **Invalid Geometry:** If the input geometry is topologically invalid, the operation may fail or produce unexpected results; use `ST_MakeValid` prior to unioning if necessary.

### Fix Code Hint
```python
# If unioning multiple rows, use the aggregate version:
sd.sql("SELECT ST_Union_Agg(geom) FROM table_name").to_arrow_table()

# If unioning a single complex geometry:
sd.sql("SELECT ST_UnaryUnion(geom) FROM table_name").to_arrow_table()
```

## API Test: `sql:ST_Union`

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
    ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
);
```


### Goal
`sql:ST_Union` computes the geometric point set union of two input spatial objects, resulting in a single `geometry` or `geography` representing their combined area.

### Parameters
- `arg1` (`geometry`): The first input geometry or geography object.
- `arg2` (`geometry`): The second input geometry or geography object to be merged with the first.

### Input
- **Data Formats:** Accepts standard OGC WKT, WKB, EWKT, or EWKB formats.
- **Preconditions:** Both inputs must be of the same type (either both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered.

### Output
Returns `geography | geometry` — A new spatial object representing the union of the two input sets.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()
# Example using geometry
result = sd.sql("""
    SELECT ST_Union(
        ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))'),
        ST_GeomFromWKT('POLYGON ((1 0, 3 0, 3 2, 1 2, 1 0))')
    )
""").to_arrow_table()
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same spatial type (geometry or geography).
- Use `ST_GeomFromWKT` or `ST_GeogFromWKT` to prepare inputs if they are provided as strings.
- Do not mix geometry and geography types in a single `ST_Union` call.

### Prompt Snippet
```text
ST_Union(arg1, arg2)
```

### Common Failure Modes
- **Type Mismatch:** Passing a `geometry` as `arg1` and a `geography` as `arg2` will result in an execution error.
- **Missing Bounder:** Geography operations will fail if the `s2geography` bounder is not registered in the current session.
- **Invalid Geometries:** If inputs are not valid OGC geometries, the operation may fail; use `ST_MakeValid` prior to unioning if necessary.

### Fix Code Hint
```python
# Ensure consistent types and valid geometries
sd.sql("SELECT ST_Union(ST_MakeValid(geom1), ST_MakeValid(geom2)) FROM table")
```

## API Test: `sql:ST_Union_Agg`

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
Computes the geometric union of a set of input geometries, returning a single geometry that represents the point set union of all input features.

### Parameters
- `arg1` (`geometry`): A column or expression containing the geometries to be aggregated.

### Input
- **Data:** Standard OGC WKT/WKB geometries.
- **Preconditions:** Input geometries should ideally share the same Coordinate Reference System (CRS). If CRS differs, SedonaDB may attempt to reproject, but consistent CRS is recommended for predictable results.

### Output
Returns `geometry` — A single geometry object representing the merged spatial extent of all input geometries.

### Valid Call Patterns
```python
# Using the Python SQL connection to perform the aggregation
sd.sql("""
    SELECT ST_Union_Agg(geom) 
    FROM (VALUES (ST_GeomFromText('POLYGON ((0 0, 0 2, 2 2, 2 0, 0 0))')), 
                 (ST_GeomFromText('POLYGON ((1 1, 1 3, 3 3, 3 1, 1 1))'))) AS t(geom)
""").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_Union_Agg` as an aggregate function within a `SELECT` statement.
- Ensure the input column contains valid `geometry` types.
- When using multiple geometries, ensure they are provided as rows in a table or subquery.

### Prompt Snippet
```text
SELECT ST_Union_Agg(column_name) FROM table_name;
```

### Common Failure Modes
- **Empty Input:** If the input set is empty, the result may be null or undefined depending on the SQL engine state.
- **CRS Mismatch:** If input geometries have conflicting CRS definitions, the operation may fail or produce unexpected results.
- **Invalid Geometries:** Input geometries that are not OGC-compliant may cause the aggregation to fail; use `ST_MakeValid` prior to aggregation if necessary.

### Fix Code Hint
```python
# Ensure geometries are valid before aggregation
sd.sql("""
    SELECT ST_Union_Agg(ST_MakeValid(geom)) 
    FROM my_table
""").to_arrow_table()
```

## API Test: `sql:ST_Within`

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
- **Data Formats:** Accepts standard OGC WKT, WKB, EWKT, or EWKB.
- **Preconditions:** Both arguments must share the same spatial type (both `geometry` or both `geography`). If using `geography`, the session must have the `s2geography` bounder registered. If CRS differs, geometries are reprojected to the target's CRS; if only one side has a CRS, the operation will fail.

### Output
Returns `boolean` — `true` if `arg1` is completely inside `arg2`, otherwise `false`.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("""
    SELECT ST_Within(
        ST_GeomFromWKT('POINT (1 1)'),
        ST_GeomFromWKT('POLYGON ((0 0, 2 0, 2 2, 0 2, 0 0))')
    ) as is_within
""").to_arrow_table()
assert result["is_within"][0] == True
print("__CHECK__")
```

### LLM Instruction Prompt
- Always ensure both arguments are of the same type (`geometry` or `geography`).
- Use `ST_GeomFromWKT` or `ST_GeogFromWKT` to construct inputs from strings.
- Remember that `ST_Within(A, B)` is the inverse of `ST_Contains(B, A)`.

### Prompt Snippet
```text
Use ST_Within(arg1, arg2) to check if arg1 is inside arg2. Ensure both inputs are cast to the same spatial type (geometry or geography).
```

### Common Failure Modes
- **Type Mismatch:** Attempting to compare a `geometry` type with a `geography` type.
- **CRS Inconsistency:** Providing inputs with different CRS where one is missing a CRS definition, causing the operation to fail.
- **Geography Bounder:** Failing to register the `s2geography` bounder when performing operations on `geography` types.

### Fix Code Hint
```python
# Ensure both inputs are explicitly cast to the same type
sd.sql("SELECT ST_Within(ST_SetSRID(geom1, 4326), ST_SetSRID(geom2, 4326)) FROM table")
```

## API Test: `sql:ST_X`

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
Extracts the X-coordinate (or longitude for `geography` types) from a point geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input point object from which to extract the coordinate.

### Input
- **Data:** A valid `geometry` or `geography` object.
- **Preconditions:** The input must be a point-type geometry or geography. If the input is a non-point geometry (e.g., `POLYGON`, `LINESTRING`) or `NULL`, the function returns `NULL`.

### Output
Returns `double` — The X-coordinate or longitude value of the point.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_X(ST_Point(-74.0060, 40.7128))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_X(ST_GeogPoint(-74.0060, 40.7128))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_X` with a point-type input.
- If the input geometry type is unknown, ensure it is a point before calling `ST_X` to avoid unexpected `NULL` results.
- Use `ST_Point` for planar `geometry` and `ST_GeogPoint` for spherical `geography` inputs.

### Prompt Snippet
```text
Use ST_X(point_geom) to extract the X coordinate. Returns NULL for non-point geometries.
```

### Common Failure Modes
- **Non-Point Input:** Passing a `POLYGON` or `LINESTRING` will result in a `NULL` return value rather than an error.
- **NULL Input:** Passing a `NULL` geometry results in a `NULL` output.
- **Type Mismatch:** Ensure the input is explicitly a `geometry` or `geography` type; raw strings or numbers are not accepted without conversion (e.g., via `ST_GeomFromText`).

### Fix Code Hint
```python
# Ensure input is a point if you receive unexpected NULLs
sd.sql("SELECT ST_X(ST_Point(1.0, 2.0))").to_arrow_table()
```

## API Test: `sql:ST_XMax`

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
Returns the maximum X coordinate (or longitude for geography types) of the bounding box for a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object for which the maximum X-axis coordinate of its bounding box is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geography objects.
- **Preconditions:** The input must be a valid `geometry` or `geography` type. For `geography` types, ensure the session's geography bounder is registered.

### Output
Returns `double` — The maximum X coordinate (longitude) value of the object's bounding box.

### Valid Call Patterns
```python
import sedonadb
sd = sedonadb.connect()

# Geometry example
res_geom = sd.sql("SELECT ST_XMax(ST_GeomFromText('LINESTRING (0 0, 5 4)'))").to_arrow_table()
print(res_geom) # __CHECK__

# Geography example
res_geog = sd.sql("SELECT ST_XMax(ST_GeogFromWKT('LINESTRING (0 0, 5 4)'))").to_arrow_table()
print(res_geog) # __CHECK__
```

### LLM Instruction Prompt
- Use `ST_XMax` to extract the right-most boundary of a spatial object.
- Ensure the input is explicitly cast to `geometry` or `geography` if it is provided as a string (e.g., using `ST_GeomFromText` or `ST_GeogFromWKT`).
- Do not use this function on non-spatial types.

### Prompt Snippet
```text
Use ST_XMax(arg1) to retrieve the maximum X coordinate of the bounding box for the provided geometry or geography.
```

### Common Failure Modes
- **Type Mismatch:** Passing a non-spatial type or a string that has not been converted to a geometry/geography object.
- **Geography Bounder:** Failure to register the `s2geography` bounder when performing operations on `geography` types.
- **Null Inputs:** Passing a `NULL` geometry will result in a `NULL` output.

### Fix Code Hint
```python
# Ensure input is converted to a spatial type before calling ST_XMax
sd.sql("SELECT ST_XMax(ST_GeomFromText(wkt_column)) FROM my_table")
```

## API Test: `sql:ST_XMin`

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
Returns the minimum X coordinate (or longitude for geography types) of the bounding box for a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the minimum X-axis extent is calculated.

### Input
The function accepts `geometry` (planar) or `geography` (spherical) types. Inputs are typically constructed via `ST_GeomFromText` or `ST_GeogFromWKT`. No specific CRS transformation is required, but the input must be a valid spatial object.

### Output
Returns a `double` representing the minimum X coordinate value of the object's bounding box.

### Valid Call Patterns
```python
import sedona.db
sd = sedona.db.connect()

# Geometry example
result_geom = sd.sql("SELECT ST_XMin(ST_GeomFromText('LINESTRING (0 0, 5 4)'))").to_arrow_table()
print(result_geom)
# __CHECK__ witness: 0.0

# Geography example
result_geog = sd.sql("SELECT ST_XMin(ST_GeogFromWKT('LINESTRING (0 0, 5 4)'))").to_arrow_table()
print(result_geog)
# __CHECK__ witness: 0.0
```

### LLM Instruction Prompt
- Always use `ST_XMin` as a standalone SQL function within `sd.sql()`.
- Ensure the input is a valid `geometry` or `geography` type; if the input is a string, wrap it in the appropriate constructor (e.g., `ST_GeomFromText`).

### Prompt Snippet
```text
Use ST_XMin(geometry_expression) to extract the minimum X coordinate from a spatial object.
```

### Common Failure Modes
- Passing a non-spatial type (e.g., a raw string or numeric type) will result in a SQL execution error.
- Attempting to use `ST_XMin` on an empty geometry may return `NULL` or an undefined value depending on the underlying engine state.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_XMin(ST_GeomFromText(my_wkt_column)) FROM my_table")
```

## API Test: `sql:ST_Y`

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
Extracts the Y-coordinate (or latitude for `geography` types) from a point geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input point object from which to extract the Y-coordinate.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The input must be a point geometry. If the input is a non-point geometry (e.g., `LineString`, `Polygon`) or `NULL`, the function returns `NULL`.

### Output
Returns `double` — The Y-coordinate value of the point, or `NULL` if the input is not a point or is missing.

### Valid Call Patterns
```python
# Example using geometry
result = sd.sql("SELECT ST_Y(ST_Point(-74.0060, 40.7128))").to_arrow_table()
print(result)
# __CHECK__

# Example using geography
result = sd.sql("SELECT ST_Y(ST_GeogPoint(-74.0060, 40.7128))").to_arrow_table()
print(result)
# __CHECK__
```

### LLM Instruction Prompt
- Always use `ST_Y(arg1)` as a standalone SQL function call within `sd.sql()`.
- Ensure the input `arg1` is a point-type geometry or geography; if the input is a complex geometry, the function will return `NULL`.

### Prompt Snippet
```text
ST_Y(point_geometry_column)
```

### Common Failure Modes
- **Non-Point Input:** Passing a `Polygon` or `LineString` will result in a `NULL` return value rather than an error.
- **Null Input:** Passing a `NULL` value results in a `NULL` output.
- **Type Mismatch:** Ensure the input is a valid SedonaDB geometry or geography object; passing raw strings or non-spatial types will cause a query execution error.

### Fix Code Hint
```python
# If you have a non-point geometry, use ST_Centroid() to get a point first
sd.sql("SELECT ST_Y(ST_Centroid(geom)) FROM my_table").to_arrow_table()
```

## API Test: `sql:ST_YMax`

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
Returns the maximum Y coordinate (or latitude for geography) of the bounding box of a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the maximum Y-coordinate or latitude is to be calculated.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** For `geography` types, the session must have the `s2geography` bounder registered.

### Output
Returns `double` — The maximum Y coordinate value (planar) or maximum latitude value (spherical) of the input object's bounding box.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_YMax(ST_GeomFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_YMax(ST_GeogFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_YMax` to extract the upper boundary of a spatial object.
- Ensure the input is explicitly cast to `geometry` or `geography` using `ST_GeomFromText` or `ST_GeogFromText` if provided as a WKT string.
- Do not use this function for raster objects; use `RS_MetaData` or similar raster-specific accessors instead.

### Prompt Snippet
```text
SELECT ST_YMax(ST_GeomFromText('...'))
```

### Common Failure Modes
- **Type Mismatch:** Passing a raster object instead of a geometry/geography.
- **Geography Bounder:** Calling `ST_YMax` on a `geography` type without the `s2geography` bounder registered in the session.
- **Null Input:** Passing a `NULL` geometry/geography will return `NULL`.

### Fix Code Hint
```python
# Ensure input is valid geometry/geography
sd.sql("SELECT ST_YMax(ST_MakeValid(geom)) FROM table_name")
```

## API Test: `sql:ST_YMin`

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
Computes the minimum Y-coordinate (or minimum latitude for geography types) of the bounding box for a given geometry or geography object.

### Parameters
- `arg1` (`geometry` or `geography`): The input spatial object for which the minimum Y-coordinate is to be calculated.

### Input
- **Data Formats:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** Input objects should be valid OGC geometries or geographies. For `geography` types, the session must have the `s2geography` bounder registered.

### Output
Returns `double` — The minimum Y-coordinate value (or minimum latitude) of the object's bounding box.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_YMin(ST_GeomFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_YMin(ST_GeogFromText('LINESTRING(0 0, 5 4)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_YMin` as a standalone SQL function within `sd.sql()`.
- Ensure the input is explicitly cast or created using `ST_GeomFromText` or `ST_GeogFromText` if the input is a WKT string.
- Distinguish between `geometry` and `geography` inputs, as the calculation logic differs for spherical coordinates.

### Prompt Snippet
```text
SELECT ST_YMin(ST_GeomFromText('...'));
```

### Common Failure Modes
- Passing a non-spatial type or an invalid WKT string will result in a query execution error.
- Using `geography` without the required `s2geography` bounder registered in the session.
- Assuming planar behavior for `geography` types; remember that `geography` calculations account for spherical geometry (e.g., South Pole crossing).

### Fix Code Hint
```python
# Ensure valid WKT and correct type constructor
result = sd.sql("SELECT ST_YMin(ST_GeomFromText('POINT(1 2)'))").to_arrow_table()
print(result) # __CHECK__
```

## API Test: `sql:ST_Z`

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
Extracts the Z-coordinate (elevation) value from a 3D point geometry or geography object.

### Parameters
- `arg1` (`geometry` | `geography`): The input spatial object from which to extract the Z-coordinate.

### Input
- **Data Types:** Accepts `geometry` (planar) or `geography` (spherical) types.
- **Preconditions:** The input must contain a Z-dimension. If the geometry is 2D or a non-point geometry type, the function returns `NULL`.

### Output
Returns `double` — The Z-coordinate value of the point, or `NULL` if the Z-dimension is missing or the input is not a point.

### Valid Call Patterns
```python
# Example using SedonaDB SQL connection
result = sd.sql("SELECT ST_Z(ST_PointZ(-74.0060, 40.7128, 100.0))").to_arrow_table()
print(result)
# __CHECK__ witness: 100.0
```

### LLM Instruction Prompt
- Always use `ST_Z` as a standalone SQL function within an `sd.sql()` query.
- Ensure the input geometry is constructed with a Z-dimension (e.g., using `ST_PointZ`) if a numeric result is expected.

### Prompt Snippet
```text
Use ST_Z(geometry_column) to retrieve the elevation value from a 3D point.
```

### Common Failure Modes
- **Non-Point Geometries:** Passing a `LINESTRING` or `POLYGON` will result in `NULL`, even if they contain Z-coordinates.
- **Missing Z-Dimension:** Passing a standard 2D `ST_Point` will result in `NULL`.
- **Type Mismatch:** Passing non-spatial types will cause a SQL execution error.

### Fix Code Hint
```python
# Ensure the geometry is 3D before calling ST_Z
sd.sql("SELECT ST_Z(ST_Force3D(ST_Point(-74.0, 40.7)))").to_arrow_table()
```

## API Test: `sql:ST_ZMax`

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
SELECT ST_ZMax(ST_GeogFromWKT('LINESTRING Z (1 3 4, 5 6 7)'));
```


### Goal
Returns the maximum Z-coordinate value present within the bounding box of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., `LINESTRING Z`, `POINT Z`) from which to extract the maximum Z-coordinate.

### Input
- **Data Formats:** Accepts standard OGC WKT/WKB geometries or geographies.
- **Preconditions:** The input object must contain Z-dimension information (e.g., 3D geometries). If the geometry is 2D, the behavior is undefined or may return a null/default value.

### Output
Returns `double` — The maximum Z-coordinate value found in the geometry or geography's bounding box.

### Valid Call Patterns
```python
# Example using geometry
sd.sql("SELECT ST_ZMax(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)'))").to_arrow_table()

# Example using geography
sd.sql("SELECT ST_ZMax(ST_GeogFromWKT('LINESTRING Z (1 3 4, 5 6 7)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Always use `ST_ZMax` to extract the vertical extent of 3D spatial objects.
- Ensure the input is explicitly cast or created as a 3D type (e.g., using `ST_GeomFromWKT` with Z-coordinates) to ensure the function has valid Z-data to process.

### Prompt Snippet
```text
Use ST_ZMax(geometry_column) to retrieve the maximum Z-coordinate from a 3D geometry or geography column.
```

### Common Failure Modes
- **Missing Z-dimension:** Passing a 2D geometry (e.g., `POINT (1 1)`) will fail to return a meaningful Z-maximum.
- **Type Mismatch:** Passing a non-spatial type or a null geometry will result in an execution error.
- **Geography Bounder:** Ensure the session's geography bounder is correctly initialized if working with `geography` types.

### Fix Code Hint
```python
# Ensure geometry is 3D before calling ST_ZMax
sd.sql("SELECT ST_ZMax(ST_Force3D(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_ZMin`

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
Returns the minimum Z-coordinate value found within the 3D bounding box of a given `geometry` or `geography` object.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object (e.g., `LINESTRING Z`, `POINT Z`) from which to extract the minimum Z-coordinate.

### Input
The input must be a valid `geometry` or `geography` object containing Z-dimension data. Standard OGC WKT/WKB formats are supported.

### Output
Returns `double` — The minimum Z-coordinate value of the input object's bounding box.

### Valid Call Patterns
```python
# Geometry example
sd.sql("SELECT ST_ZMin(ST_GeomFromWKT('LINESTRING Z (1 3 4, 5 6 7)'))").to_arrow_table()

# Geography example
sd.sql("SELECT ST_ZMin(ST_GeogFromWKT('LINESTRING Z (1 3 4, 5 6 7)'))").to_arrow_table()
```

### LLM Instruction Prompt
- Use `ST_ZMin` to extract the lowest Z-coordinate from 3D geometries or geographies.
- Ensure the input object explicitly contains Z-dimension data (e.g., using `ST_GeomFromWKT` with 'Z' coordinates).
- The function returns a `double` scalar.

### Prompt Snippet
```text
Use ST_ZMin(geometry_column) to retrieve the minimum Z-coordinate from a 3D spatial object.
```

### Common Failure Modes
- Passing a 2D geometry (missing Z-dimension) may result in unexpected behavior or null values.
- Providing an invalid WKT string that does not conform to the expected Z-dimension format.

### Fix Code Hint
```python
# Ensure the geometry is 3D before calling ST_ZMin
sd.sql("SELECT ST_ZMin(ST_Force3D(geom)) FROM table").to_arrow_table()
```

## API Test: `sql:ST_Zmflag`

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
SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING Z(1 2 3, 3 4 5)'));
```

4D (XYZM) returns 3:

```sql
SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING ZM(1 2 3 4, 3 4 5 6)'));
```


### Goal
Determines the coordinate dimension flag of a given `geometry` or `geography` object to identify the presence of Z (elevation) or M (measure) coordinates.

### Parameters
- `arg1` (`geometry` or `geography`): The spatial object to inspect for coordinate dimension metadata.

### Input
- **Data:** A valid `geometry` or `geography` object.
- **Formats:** Standard OGC WKT/WKB or objects constructed via `ST_GeomFromWKT` or `ST_GeogFromWKT`.
- **Preconditions:** The input must be a valid spatial object; no specific CRS is required for this metadata inspection.

### Output
Returns `integer` — A flag code: 0 (XY), 1 (XYM), 2 (XYZ), or 3 (XYZM).

### Valid Call Patterns
```python
# Example using Python SQL connection
result = sd.sql("SELECT ST_Zmflag(ST_GeomFromWKT('LINESTRING Z(1 2 3, 3 4 5)'))").to_arrow_table()
print(result)
# __CHECK__ witness: 2
```

### LLM Instruction Prompt
- Always use `ST_Zmflag(arg1)` as a standalone SQL function call.
- Ensure the input is a valid spatial object (geometry or geography).
- Interpret the returned integer according to the mapping: 0=2D, 1=XYM, 2=XYZ, 3=XYZM.

### Prompt Snippet
```text
SELECT ST_Zmflag(geometry_column) FROM table_name;
```

### Common Failure Modes
- Passing a non-spatial type (e.g., string or numeric) will result in a SQL execution error.
- The behavior for mixed-dimension collections is not explicitly defined in the documentation and may lead to unexpected results.

### Fix Code Hint
```python
# Ensure the input is cast to a geometry type if it is currently a string
sd.sql("SELECT ST_Zmflag(ST_GeomFromWKT(wkt_column)) FROM my_table")
```

## API Test: `sql:rs_ensureloaded`

### Signature
```python
rs_ensureloaded(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona/src/context.rs:1315_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Ensures that internal raster data structures are fully loaded into memory for subsequent spatial operations.

### Parameters
_None._

### Input
The function accepts parameters as defined by the user-defined runtime signature; however, no specific parameter contract is documented in the official SedonaDB 0.5.0 SQL reference.

### Output
Returns `unspecified` — The function is an internal utility; its return value is not documented for public API usage.

### Valid Call Patterns
```python
# Inferred from signature: no documented usage examples exist in the project README or test suite.
sd.sql("SELECT rs_ensureloaded()").to_arrow_table()
```

### LLM Instruction Prompt
- Do not assume `rs_ensureloaded` accepts specific arguments.
- Use this function only if explicitly required by internal system state management, as it is not part of the standard public-facing geospatial analysis workflow.
- Always verify the existence of the raster context before calling.

### Prompt Snippet
```text
# Example of calling the internal function
sd.sql("SELECT rs_ensureloaded()").to_arrow_table()
```

### Common Failure Modes
- **Undocumented Contract:** Calling this function with arguments not supported by the underlying Rust implementation will result in a runtime SQL error.
- **Context Dependency:** May fail if called outside of a valid raster-processing session or if the internal state is not initialized.
- **Experimental Status:** As an undocumented internal function, it is subject to removal or signature changes in future SedonaDB versions.

### Fix Code Hint
```python
# If rs_ensureloaded fails, verify that the raster has been initialized 
# via RS_FromPath or RS_MakeEmptyRaster before attempting to ensure it is loaded.
sd.sql("SELECT RS_MetaData(raster) FROM (SELECT RS_FromPath('path/to/raster.tif') AS raster) t").to_arrow_table()
```

## API Test: `sql:sd_simplifystorage`

### Signature
```python
sd_simplifystorage(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/sd_simplifystorage.rs:36_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Ensures that binary data is stored using standard `Binary` format rather than Arrow `BinaryView` storage.

### Parameters
_None._

### Input
The function is intended to be used in conjunction with `ST_AsBinary`. It requires a geometry or geography object that has been serialized to binary format.

### Output
Returns `binary` — The input data converted to a standard `Binary` storage format.

### Valid Call Patterns
```python
# Inferred from documentation: used to cast binary output from ST_AsBinary
query = "SELECT sd_simplifystorage(ST_AsBinary(ST_Point(0, 0)))"
result = sd.sql(query).to_arrow_table()
print(result)
```

### LLM Instruction Prompt
- Use `sd_simplifystorage` only when the output of a spatial serialization function (like `ST_AsBinary`) needs to be forced into a standard `Binary` storage format for compatibility with downstream Arrow consumers.
- Do not pass arguments to this function as the signature is user-defined and undocumented.

### Prompt Snippet
```text
sd_simplifystorage(ST_AsBinary(geometry_column))
```

### Common Failure Modes
- **Invalid Input Type:** Passing a non-binary type (e.g., a raw geometry object instead of the result of `ST_AsBinary`) will result in a runtime SQL execution error.
- **Missing Context:** Using this function without a preceding binary serialization operation is unsupported and may lead to unexpected behavior or type mismatches.

### Fix Code Hint
```python
# Ensure the input is serialized to binary first
sd.sql("SELECT sd_simplifystorage(ST_AsBinary(geom)) FROM my_table").to_arrow_table()
```

## API Test: `sql:st_geomfromwkbunchecked`

### Signature
```python
st_geomfromwkbunchecked(parameters not documented in the configured SQL reference)
```
_Source: source/rust/sedona-functions/src/st_geomfromwkb.rs:57_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Constructs a geometry object from a Well-Known Binary (WKB) representation without performing standard OGC validity checks.

### Parameters
_None._

### Input
The function expects a binary input representing a Well-Known Binary (WKB) geometry. As this is an "unchecked" variant, the caller is responsible for ensuring the input binary is a valid, well-formed WKB sequence, as the function skips standard validation routines.

### Output
Returns `geometry` — The materialized geometry object derived from the provided WKB binary.

### Valid Call Patterns
```python
# Inferred from signature:
sd.sql("SELECT st_geomfromwkbunchecked(wkb_column) FROM table_name").to_arrow_table()
```

### LLM Instruction Prompt
- Use `st_geomfromwkbunchecked` only when performance is critical and the input WKB is guaranteed to be valid by upstream processes.
- Do not use this function for untrusted user input, as it bypasses standard geometry validation.
- Always ensure the input is a valid binary type compatible with SedonaDB's geometry representation.

### Prompt Snippet
```text
SELECT st_geomfromwkbunchecked(binary_wkb_data) FROM my_table;
```

### Common Failure Modes
- **Invalid Binary:** Providing malformed or truncated WKB data will result in runtime errors or undefined behavior, as the function does not perform validation.
- **Type Mismatch:** Passing non-binary types (e.g., strings or integers) instead of a binary WKB stream will cause query execution failure.
- **Memory Constraints:** Large, complex geometries may exceed memory limits if the underlying WKB is excessively large.

### Fix Code Hint
```python
# If validation is required, use the standard ST_GeomFromWKB instead:
sd.sql("SELECT ST_GeomFromWKB(wkb_column) FROM table_name").to_arrow_table()
```

## API Test: `sql:st_longestline`

### Signature
```python
st_longestline(parameters not documented in the configured SQL reference)
```
_Source: source/c/sedona-s2geography/src/kernels.rs:115_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Computes the longest line segment between two geometries, representing the maximum distance between any two points within the input geometries.

### Parameters
_None._

### Input
- **Data:** Accepts standard OGC WKT/WKB geometry objects.
- **Preconditions:** Inputs must be valid geometries. If using `geography` types, the session must have the `s2geography` bounder registered.
- **CRS:** If inputs have differing CRS, they must be transformed to a common CRS before processing.

### Output
Returns `geometry` — A `LINESTRING` representing the longest distance between the two input geometries.

### Valid Call Patterns
```python
# Inferred from signature:
sd.sql("SELECT ST_AsText(st_longestline(geom1, geom2)) FROM table").to_arrow_table()
```

### LLM Instruction Prompt
- Use `st_longestline` only when calculating the maximum distance segment between two geometries.
- Always ensure inputs are valid geometries using `ST_MakeValid` if necessary.
- Do not assume specific parameter arity; verify the function signature against the specific SedonaDB 0.5.0 environment if runtime errors occur.

### Prompt Snippet
```text
SELECT st_longestline(ST_GeomFromText('...'), ST_GeomFromText('...'))
```

### Common Failure Modes
- **Invalid Geometry:** Passing malformed WKT or invalid OGC geometries will cause execution failure.
- **CRS Mismatch:** Attempting to compute the longest line between geometries with incompatible or missing CRS definitions will result in an error.
- **Unsupported Types:** Passing raster objects directly to this function is not supported; use vectorization operations first.

### Fix Code Hint
```python
# Ensure geometries are valid and in the same CRS before calling
query = """
SELECT st_longestline(
    ST_MakeValid(geom1), 
    ST_Transform(geom2, 4326)
) FROM my_table
"""
sd.sql(query).to_arrow_table()
```

## API Test: `sql:st_numinteriorring`

### Signature
```python
st_numinteriorring(parameters not documented in the configured SQL reference)
```
_Source: source/c/sedona-geos/src/register.rs:81_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Returns the number of interior rings (holes) within a given polygon geometry.

### Parameters
_None._

### Input
The function expects a `geometry` object, specifically a `POLYGON` or `MULTIPOLYGON`. The input must be a valid OGC geometry type. If the input is not a polygon, the behavior is undefined.

### Output
Returns an `integer` representing the count of interior rings (holes) contained within the geometry.

### Valid Call Patterns
```python
# Inferred from signature:
sd.sql("SELECT st_numinteriorring(ST_GeomFromText('POLYGON ((0 0, 10 0, 10 10, 0 10, 0 0), (2 2, 2 4, 4 4, 4 2, 2 2))'))").to_arrow_table()
# __CHECK__ witness: 1
```

### LLM Instruction Prompt
- Use `st_numinteriorring` to count the number of interior rings in a polygon.
- Ensure the input is a valid `geometry` type (e.g., created via `ST_GeomFromText`).
- Note that this function is registered in the SedonaDB SQL environment and must be called via `sd.sql()`.

### Prompt Snippet
```text
SELECT st_numinteriorring(geometry_column) FROM table_name;
```

### Common Failure Modes
- **Invalid Geometry Type:** Passing a `POINT` or `LINESTRING` instead of a `POLYGON` may result in an error or unexpected return value.
- **Null Input:** Passing a `NULL` geometry will return `NULL`.
- **Unregistered Function:** If the SedonaDB session is not properly initialized, the function name may not be recognized by the SQL engine.

### Fix Code Hint
```python
# Ensure the input is a valid polygon before calling
sd.sql("""
    SELECT st_numinteriorring(geom) 
    FROM (SELECT ST_GeomFromText('POLYGON((0 0, 10 0, 10 10, 0 10, 0 0))') as geom)
""").to_arrow_table()
```

## API Test: `sql:st_shortestline`

### Signature
```python
st_shortestline(parameters not documented in the configured SQL reference)
```
_Source: source/c/sedona-s2geography/src/kernels.rs:116_

_Source doc:_ Registered SQL function with no individual page in the configured original SQL reference. Do not invent an undocumented contract.

### Goal
Computes the shortest line segment between two input geometries.

### Parameters
_None._

### Input
The function accepts two geometry or geography objects as input. Inputs must be valid OGC WKT/WKB geometries or geography types. If the inputs have differing Coordinate Reference Systems (CRS), they must be transformed to a common CRS prior to invocation, as SedonaDB requires consistent CRS handling for spatial operations.

### Output
Returns `unspecified` — The result is a `geometry` or `geography` object representing the shortest line segment connecting the two input geometries.

### Valid Call Patterns
```python
# Inferred from signature:
sd.sql("SELECT ST_ShortestLine(geom1, geom2) FROM table").to_arrow_table()
```

### LLM Instruction Prompt
- When calling `st_shortestline`, ensure both input geometries are provided as valid `geometry` or `geography` types.
- Always verify that both inputs share the same CRS or have been explicitly transformed using `ST_Transform` before calling this function to avoid runtime errors.
- Do not assume specific parameter arity; the function signature is user-defined at runtime.

### Prompt Snippet
```text
SELECT ST_ShortestLine(ST_GeomFromText('POINT(0 0)'), ST_GeomFromText('POINT(1 1)'))
```

### Common Failure Modes
- **CRS Mismatch:** Calling the function with geometries in different coordinate systems will cause the operation to fail.
- **Invalid Geometry:** Providing malformed WKT or WKB strings will result in an execution error; use `ST_MakeValid` to repair geometries if necessary.
- **Unsupported Types:** Attempting to pass raster objects directly to this vector-based function will result in a type error.

### Fix Code Hint
```python
# Ensure CRS consistency before calling
query = """
SELECT ST_ShortestLine(
    ST_Transform(geom1, 'EPSG:4326'), 
    ST_Transform(geom2, 'EPSG:4326')
) FROM my_table
"""
sd.sql(query).to_arrow_table()
```

