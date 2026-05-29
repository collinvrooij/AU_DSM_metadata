\### Changelog for the AU DSM covariate stack ###

Authors:	Anders Bjørn Møller, Collin Xander van Rooij

Date:	2026-03-19



\## Version 20260526 ##

Collin, 29-5-2026:
-   Included the correct sources for the covariates insterted by Amelie and Anders
-   Included the NDVI and NDMI covariates in the overview
-   Included additional sources in the bibliography

Lucas, Sebastian and Collin:

\-	Changed the covariates file to include Lucas changes



\## Version 20260318 ##



Collin van Rooij, 1-5-2026

\-	Added the GBIF layers in the covariate metadatastack as an example

\-	Added a bibliography text file



\## Version 20260318 ##



Collin van Rooij



\-	Added a .md file to explain how to work with the AU DSM Metadata GitHub repo





\## Version 20260318 ##



Additions:

\-	Distance to churches built before 1800 AD (proxy for historic land use and

&#x09;management).

\-	Microtopographical layers summarizing variation at 1.25 m resolution.

\-	Historic land use probabilities in the 18th century, based on the map created

&#x09;by the The Royal Danish Academy of Sciences and Letters.

\-	Peat probability and relative sampling density based on the Ochre and Jupiter

&#x09;databases (The are mainly intended for mapping historic peat, but I have

&#x09;included them to facilitate their implementation.)



Changes:

\-	Updated and renamed ALOS/PALSAR layers, which now cover the years 2015 – 2023

&#x09;(Thanks to Sebastian).

\-	Renamed CHELSA layers for better sorting and interpretation.

\-	Renamed the SoilSuite layers to increase readability.

\-	Renamed indicator layers for better sorting and interpretation.

\-	Further removal of NA values across all layers.

\-	Included a proper changelog.

\-	Updated overview table with more references and links to the (most) relevant

&#x09;R scripts for processing the data.

\-	Fixed a typo in the mid slope position layer.



Removals:

\-	Cost\_dist: Missing values in islands.

\-	Sentinel-1 layers from 2020: Missing values.

\-	Redundant layers for detrended dem and vdtchn (as pointed out by Yang).

\-	GW layers from GEUS (missing values)







\## Version 20251219 ##



Additions:

\-	L-band radar satellite images from ALOS/PALSAR.

\-	A new bare soil composite from SoilSuite (20 m resolution), as well as the

&#x09;overall surface reflectance from Sentinel-2. The gaps in the bare soil

&#x09;composite have been filled in to avoid artifacts from NAs, but there are also

&#x09;layers which explicitly indicate the bare soil extent.

\-	Fuzzy indicator layers for nature types from basemap.



Removals:

\-	Satellite products from DIGIJORD, due to ownership of the data.

\-	Central wetlands: We are currently working on an updated map of historical

&#x09;peatlands, so this layer should no longer be used.

\-	Indicator layers with crisp boundaries and crisp layers of cropping history

&#x09;(IMK). The new version only uses fuzzified indicator rasters, as they produce

&#x09;a more realistic prediction surface.

\-	If you still need the central wetlands and/or the crisp indicator layers, you

&#x09;can find them here:

&#x09;O:\\Tech\_AGRO\\Jord\\DSM\\Extra\_rasters\\national\_10m\\unused\_covariates



Other changes:

\-	I have expanded all the tiles with 16 pixels in each direction to create an

&#x09;overlap between them. This means we can now use the tiles for predictions with

&#x09;convolutional neural networks with kernel sizes up to 32x32.

\-	The layer “wetlands\_10m\_fuzzy” has been renamed to “fuzzy\_adk\_wetlands” to

&#x09;indicate the data source (ADK: Arealdatakontoret – The Areal Data Office).

\-	Repaired missing values in some of the fuzzy indicator rasters.

\-	Old 30.4 m stack: Some of the layers in the old stack had mismatching

&#x09;extents. This has now been fixed.





\## Version 20240304 ##



Additions:

\-	A larger set (64 layers) with oblique geographic coordinates, in case you

&#x09;want to try a higher number.

\-	Fuzzy boundaries for the categorical indicator rasters (“fuzzy\_...”). In many

&#x09;cases these layers can help to create softer, more realistic changes from one

&#x09;class to another. I have based the width of the fuzzy boundaries on the

&#x09;approximate spatial uncertainty of the original maps. The original layers with

&#x09;crisp boundaries are still in the stack if you prefer those.

\-	Bare soil composite with filled gaps (“filled\_...”). In these layers I have

&#x09;first removed edge pixels and pixels with less than 10 bare soil images. I

&#x09;have then filled the gaps by interpolation, using an iterative procedure

&#x09;based on aggregation and smoothing. The values in the gaps do not necessarily

&#x09;represent the expected soil reflectance, but they often alleviate the problem

&#x09;of sharp boundaries at the edge of the bare soil extent.

\-	The layer “s2\_count\_max10.tif”, indicating the number of bare soil images for

&#x09;each pixel, with a maximum value of 10. In some cases, the extent of the bare

&#x09;soil area can be a useful covariate in itself. The layer

&#x09;“s2\_count\_max10\_fuzzy.tif” is a smoothed version, which I use myself.



Changes:

\- 	Produced a set of 591 tiles to facilitate parallel processing.





\## Version 20230323 ##



Additions:

\-	CHELSA bioclimatic variables from Sebastian.

\-	Cost distances and detrended DEM from Gasper.

\-	Hillyness layer from Mette.



Changes:

\-	Changed the layer names stored in the files, so they all match the file name.

&#x09;This way, covariates will have the correct names when loaded with terra in R.

\-	Included two columns in the overview table, containing a text description of

&#x09;each covariate as well as the units. This work is not complete, but you can

&#x09;help to fill out some of the missing values if you like.





\## Version 20230124 ##



Changes:

\- 	Replaced all dots (.) in the names with underscores. The names now

&#x09;exclusively separate words using underscores.

\- 	For standardization, all names are now in lowercase only.

\-	Masked all covariates to the coastline of the DEM, so no covariates have

&#x09;values outside of this mask.

\- 	Updated the overview csv with more information. The csv now also lists 	inclusion/exclusion from the stack for each covariate.





\## Version 20220920 ##



Features:

\- 	First version of the new 10 m covariate stack.

\- 	Overview table included as csv, listing names, coverage, SCORPAN factors and

&#x09;a few additional notes. We still don’t have full documentation on all the

&#x09;layers, but it’s a functioning stack, which you can load into R.

\-	I have cropped all the layers to the same extent as the bare soil composite,

&#x09;so they might not match some of the other 10 layers that we have.

\-	I have rounded most of the layers to four significant digits to reduce the

&#x09;file size. This is not a perfect solution, but it works.





\### END OF DOCUMENT ###

