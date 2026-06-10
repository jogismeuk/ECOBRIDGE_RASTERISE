## Overview

This is an ArcGIS Python script that **rasterizes parcel polygons and determines the dominant land class for each parcel** across multiple raster datasets. It's part of a downscaling workflow.

## Key Workflow

1. **One-time Setup**: Converts parcel polygons to a raster grid using parcel IDs as zone identifiers
2. **Batch Processing**: For each raw raster dataset matching a pattern:
   - Uses `TabulateArea` to calculate area of each class within each parcel zone
   - Identifies the class with the largest area as the "majority class"
   - Creates an output raster where each parcel pixel shows its majority class
3. **Output**: Generates majority-class rasters for each input RAW raster

## Main Components

| Component | Purpose |
|-----------|---------|
| **Settings** | Hard-coded paths and field names (raster GDB, parcels FC, output GDB) |
| **Setup** | Creates output GDB, checks extensions, sets spatial environment |
| **Zone Raster Creation** | Converts parcels to raster once, stored as `parcel_zone_ids` |
| **TabulateArea** | Tallies area by class for each parcel zone |
| **Lookup** | Creates final output raster mapping zone → majority class |
| **Error Handling** | Tracks successes/failures with detailed logging |

## Key Design Decisions

- **Rasterize once**: Parcels are converted to raster only once (reused for all input rasters)
- **CLASSES_AS_FIELDS mode**: TabulateArea outputs one column per class (VALUE_1, VALUE_2, etc.) rather than rows
- **Majority by area**: The class occupying the most space within a parcel wins
- **Lookup approach**: Uses raster lookup table to avoid re-rasterizing

## Performance Considerations

- `parallelProcessingFactor = "0"` disables parallel processing (memory conservation)
- Intermediate tabulate tables can be deleted after processing to save space (commented out)
- Uses `tqdm` for progress bars if available

## Potential Issues to Watch

1. **Hard-coded paths** — must be updated for different machines/workflows
2. **Field name extraction** — `class_value_from_field()` assumes predictable TabulateArea field naming
3. **Memory** — processing many large rasters could be intensive
4. **NULL handling** — zones with no classes get `None` in majority field
