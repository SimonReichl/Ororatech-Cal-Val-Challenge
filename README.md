# WK-challenge

This repository is my submission for the working student challenge. It contains 3 pipelines for the 3 different parts of the challenge.
Due to the missing georeference info I did some workarounds, e.g for the Tiffs I just used transformation = none to be able to view it in QGIS.
Also for the fire analysis I was not able to use Pixel geocoordinates. With those it would be easier to track the fire and mutliband indices could be integrated in the fire detection.

Input for every pipeline is the folder containing the npy files of the frames.
These files are loaded into an xarray.Dataset and then processed in several stages.



The first challenge was to identify dead pixel, here i defined them as all values that are NaNs. The 3 bands can be analyzed seperately via mode input and the mask can be saved as (geo)tiff.



After that I identified additional drifting pixels in order to create a mask with malfunctioning pixel. For drifting pixel 2 cases can be seen. First are pixels that are almost constant over time and the second one are pixels with a median over time differing from the background median over time (5x5 mean). Those get combined with an or logic. Here you can choose the mode aswell.



The last challenge was to detect fire pixel. Here I applied the combined malfunctioning mask from the 2 steps before to only use valid pixels.
First step was to identify potential firepixels. Detection for L2 is via threshold and for L1 pixels are classified as fire if they are significantly brighter than their local background and exceed a specified contrast level compared to a smoothed reference image.

Those pixels then get categorized in clusters, where then surrounding pixels get analyzed if their difference to the surrounding non-fire pixels exceed a certain threshold.

After that cluster centroids in L2 and L1 get compared if the movement between those is according to the groundtrack speed. Through looking at the frames in QGIS I was able to identify a speed of around 12 pixel/frame, so after 28 frames and with a tolerance of 20x20 pixel the fire in L1 should be seen in L2. Here I paired L2 and L1 centroids whenever there was a match. After that I had all verified fire clusters.
(with georeference it would be more accurate and easier by just checking if the fire remains in the same geolocation over the different frames)
On the fly (geo)tiffs for firemasks per frame can be produced.

Output is then a fire track mask tiff where all verified fire clusters can be seen in L2 coordinates.
Also the cluster data gets stored in fire data. With fireid, frames where they are visible and coordinates in L2 system.



I decided to first build the mask for malfunctioning pixels to prevent the fire detection algorithm to work with a lot of false data.
As an extension of the pipeline, the fire clusters can be compared to the total detected clusters and all remaining pixels can be reevaluated for drifting pixels and fed back into the malfunctioning pixel mask.
Also the dead pixel detection was prepared to be extended, to be able to work with incoming inputs, so it checks frame after frame and adjusts the mask accordingly.
To identify the firepixel more precise M0 can be integrated aswell, for this a clearer pixelvalue analysis approach is needed.
