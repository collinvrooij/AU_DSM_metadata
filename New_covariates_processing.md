# Procedure for ingesting new data to the AU DSM covariate stack

Anders Bjørn Møller, March 2026.

# Introduction

This document outlines the main processing steps for adding new geographic data layers to the covariate stack for digital soil mapping at the Department of Agroecology at Aarhus University. The covariate stack contains a set of geographic raster layers, used as explanatory variables for mapping soil properties in Denmark. The covariate stack at 10 m resolution was originally developed in the DIGIJORD project (2021 – 2024) (Møller et al., 2025) to replace the earlier covariate stack, which had a resolution of 30.4 m.

The 10 m covariate stack has been updated several times since the original version. The changes have included steps to improve standardization and processing, as well as additions of new layers or removals of redundant layers. The covariate stack is mainly the outcome of an effort to standardize, share, document and facilitate the use of the main data layers used for soil mapping in Denmark. It serves a practical purpose, and it is not intended as a definitive or complete dataset of all potentially useful variables. Users should therefore not be limited to using the layers in the stack, if they find that other potential covariates would be useful for their work. At the same time, there is no requirement for using all available data layers in the stack for any specific purpose. Instead, users should only include the layers that they deem necessary for their own purposes.

As the covariate stack in its present form is still undergoing development, there is a need to document the main steps for ingesting new data layers to the stack. In this way, it should be possible to update it while maintaining its integrity as a standardized and accessible dataset.

# Specifications

The processing steps to ingest new data to the stack mainly serve to align the new layers with the general specifications, applied to all layers in the covariate stack:

1.        Format:              GeoTIFF (“.tif” extension)

2.       Projection:         ETRS89 UTM 32N

3.       Resolution:        10 m resolution.

4.       Extent:

5.       Xmin:   441000

6.       Xmax:   894000

7.       Ymin:    6049000

8.       Ymax:   6403000

9.       All layers in the stack are masked based on the national DEM (layer “dhm2015_terraen_10m.tif”). All cells outside the DEM are set to NA, and at the same time there should be no missing values within the coverage of the DEM.

10.    Each file contains only one layer.

11.     The layer name is the same as the file name (minus the “.tif” extension).

12.    All files are written with LZW compression. This is the default in R, when writing rasters to GeoTIFF via the _terra_ package.

13.    The rasters are written to tiled GeoTIFFs, wherein the pixels are organized into 256x256 blocks. This generally makes it faster to read the files and usually decreases the file size when compression is used. In R, this can be specified with the argument ‘gdal = “TILED=YES’, when writing rasters in the _terra_ package.

# Processing

The general processing steps are outlined in the following section. Generally, the processing of new data includes the following steps. The steps are only applied as needed, so the specific steps may not be relevant to all new layers. Furthermore, the specific order can vary, and some steps can often be combined:

1.        Gap fill.

2.       Fuzzification.

3.       Reprojection or resampling.

4.       Masking.

5.       Rounding.

6.       Rewriting (datatype, compression, tiling, names).

7.       Documentation.

8.       Creation of tiles (not the same as above).

## 1 Gap fill

If a layer is missing values within the coverage of the DEM, the affected pixels must be filled with appropriate replacement values. For example, for indicator rasters, the missing values can be replaced by zero, as the indicated class is not present within the missing cells.

For continuous variables, it may be feasible to use a smoothing filter to fill in missing values based on the neighboring cells. For large gaps, smoothing can be combined with aggregation in a stepwise procedure, which uses increasingly coarse spatial resolutions to fill in the largest gaps with smooth values. The custom function “fill_gaps_gauss” shows an example of this approach. The function has a weighting option to reduce the influence of isolated pixels:

[https://github.com/anbm-dk/AU_DSM_Covariates/blob/main/Fill_raster_gaps.R](https://github.com/anbm-dk/AU_DSM_Covariates/blob/main/Fill_raster_gaps.R)

The gap filling step, if applied, should ideally take place before any other step, such as fuzzification, reprojection, resampling, or masking. The is especially the case if the new layer has a resolution coarser than 10 m, as it is less computationally demanding to fill the gaps at a coarser resolution. However, if the new layer covers a much larger area than needed, it may be feasible to crop it to a smaller extent before filling the gaps.

In some cases, it can also be necessary to apply a gap filling step before reprojection/resampling to avoid unintentional NA values close to the coastline. Some methods for reprojection and resampling can introduce NA values along edges, as they only interpolate the values and thus lack a capacity to extrapolate. This can be avoided by filling the missing values along the coast before reprojection/resampling of the layer.

## 2 Fuzzification

By default, categorical variables are converted to indicator layers, with one indicator layer for each class (with values of 0 and 1, to represent absence and presence, respectively). However, an exception to this rule can potentially be made if the classes are ordered in a meaningful way (such as soil drainage classes). If the input layer only contains two classes (e.g. wetlands and non-wetlands), a single indicator layer can also be sufficient.

Furthermore, to avoid edge effects, a fuzzification step is generally applied to all categorical variables. Fuzzification uses a gaussian filter, and in most cases the sigma of the filter should reflect the spatial uncertainty related to the class boundaries. For example, if the classes were originally mapped as polygons at a scale of 1:10,000, the sigma of the gaussian filter should be 10 m to reflect the uncertainty of the polygon boundaries.

If the original map was created at a coarse resolution (e.g. 1:100,000), fuzzification can be combined with aggregation to speed up computation. For example, if the spatial uncertainty is 100 m, the input layer could be aggregated to 100 m resolution before applying the gaussian filter. This would decrease the necessary size of the filter, as well as the number of cells in the layer. After fuzzification, the layer can be resampled back to 10 m resolution.

Fuzzification can also be applied to categorical variables originally mapped at 10 m resolution, as the use of these layers as covariates can still produce unintended artifacts along the class boundaries. In these cases, the gaussian filter should have a sigma of ~5 m to reflect the low spatial uncertainty of the input layers. Likewise, fuzzification should also be applied to numeric variables that originate from polygon layers. In the present version of the stack, this is mainly the case for variables based on the farmers’ field registrations.

If the input layer is split into more than one indicator layer, the output fuzzified indicator layers should be standardized to a sum of 1 for each pixel (unless the sum of the layers is zero for that pixel).

The gap filling step should always be applied before fuzzification, if these two steps are used together. However, in all other cases, fuzzification should generally be applied before the other steps. In exceptional cases, fuzzification can be used after reprojection/resample to eliminate unforeseen artifacts in the output, but this is not the general rule.

Examples of the fuzzification procedure can be seen in this R script:

[https://github.com/anbm-dk/AU_DSM_Covariates/blob/main/04_Process_indicators.R](https://github.com/anbm-dk/AU_DSM_Covariates/blob/main/04_Process_indicators.R)

## 3 Reprojection or resampling

If the projection of the new layer differs from the specifications, it will be necessary to reproject it to match the projection, resolution and extent of the layers in the stack.

If the layer has the correct projection, but the resolution differs from 10 m, it will only be necessary to resample it to match the specifications. Resampling can also be applied if the cells of the layer to not match the stack, due to a difference in the origin. If the cell size of the new layer is smaller than 10 m, it may be necessary to aggregate it to a coarser resolution closer to 10 m before resampling. If feasible, reprojection and resampling should use bicubic spline interpolation.

Most methods for reprojection and resampling also include an option to specify an extent for the output layer. If the extent of the new layer differs, but the cells in the layer already align with the stack, it will only be necessary to crop the layer and/or pad it with new cells to match the specified extent.

In most cases, reprojection/resampling is carried out after the gap filling and/or fuzzification steps, if any of these two steps are used. As noted above, fuzzification can be used after reprojection/resampling in exceptional cases, but only when necessary. Other steps, such as masking and rounding, should not be carried out until the layer has been reprojected or resampled.

## 4 Masking

If the layer contains values outside the coverage of the DEM, these values should be set to NA. Masking should not take place until the layer has been subjected to the gap filling, fuzzification and/or reprojection/resampling steps (if they are used). However, masking can take place before or after rounding the values in the layer (if relevant), as the order of these two steps does not affect the output.

## 5 Rounding

If the values in the layer have an inappropriately large number of significant digits, the redundant digits should be rounded off to reduce the file size. For example, for a DEM, digits beyond 1 cm will usually be redundant, due to the uncertainty of the measured elevation. However, excessive rounding can also transform a smooth spatial gradient into a “stepped” pattern, which can cause unwanted artifacts in the output soil maps. The number of significant digits maintained in the layer is therefore a compromise between the file size and the representation of spatial trends.

Rounding should generally be one of the last processing steps to be carried out, and it is not always necessary.

## 6 Rewriting

The processing nearly always includes a step to rewrite the raster layer to a new file. In some cases, this will be the only necessary step, but it can also be combined with the other steps. The rewriting step ensures that the layer has the correct file type, layer name, datatype, compression and tiling, as listed in the specifications above.

The applied datatype (also known as “pixel depth”) specifies the bit depth of the pixels, which determines number of bytes used to save the values on the disk as well as the range of values that can be stored. For example, if a layer contains only integer values in the range 0 – 10,000, an unsigned 16-bit depth (or “INT2U” in R) will be a good choice.

These two links provide more information on the available bit-depth choices:

[https://pro.arcgis.com/en/pro-app/latest/help/data/imagery/bit-depth-capacity-for-raster-dataset-cells.htm](https://pro.arcgis.com/en/pro-app/latest/help/data/imagery/bit-depth-capacity-for-raster-dataset-cells.htm)

[https://rdrr.io/cran/terra/man/datatype.html](https://rdrr.io/cran/terra/man/datatype.html)

## 7 Documentation

The name and source of the new layers is listed in the overview table in the covariates folder, along with the name(s) of the person(s) who produced and/or processed the layer. The entry should also include a short full-text description of what the layer represents, the unit of the variable (if relevant), a link to the script used for processing the layer (if possible), a short description of the processing steps, links to any relevant literature, and the SCORPAN factor(s) related to the layer (McBratney et al., 2003).

## 8 Creation of tiles

When all layers in the stack have been processed to fit the specifications, as described above, a set of 591 geographic tiles is created, to facilitate parallel computation. These tiles are not the same as the tiling used internally for each layer (see the Rewriting step described above). Instead, each tile covers a specific area of Denmark, with a cropped file for that area for each layer in the stack, collected into one folder for. The name of the folder indicates the tile (e.g. “tile_001”) and the cropped layers have the same file names and layer names and the original layers.

The division of the layers into tiles uses a set of predefined polygons, stored together with the tiles. In the present version of the stack, the extents of the separate tiles have been expanded slightly to create a small overlap between the tiles. The main purpose of the overlap is to allow the implementation of convolutional neural networks.

The original files in the stack are maintained for illustration and documentation, and the output tiles are stored in another folder to avoid confusion.

* Added by Collin van Rooij 13-04-2026 *
When using the tiles to tile new layers, the tiles should be appended on all sides by 16 pixels (so 160 meters) before being used to clip the new data to. This ensure overlap in training and prediction so to not get arbitrary boundaries during training. 

# References

McBratney, A.B., Mendonça Santos, M.L., Minasny, B., 2003. On digital soil mapping. Geoderma 117(1-2), 3-52. [https://dx.doi.org/10.1016/s0016-7061(03)00223-4](https://dx.doi.org/10.1016/s0016-7061\(03\)00223-4)

Møller, A.B., Nyborg, L., Grogan, K., Druce, D., Svane, S.F., Greve, M.B., Gutierrez, S., Styczen, M., Greve, M.H., Knudsen, L., Beucher, A., 2025. High-resolution 3D soil texture mapping in Denmark using satellite time series and bare soil composites. Advances in Agronomy 194, 133-186. [https://dx.doi.org/10.1016/bs.agron.2025.07.001](https://dx.doi.org/10.1016/bs.agron.2025.07.001)