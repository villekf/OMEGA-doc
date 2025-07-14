Using your own source/detector coordinates or list-mode data
============================================================

This page gives some tips on how to reconstruct data that cannot be reconstructed through built-in methods. This can be a different modality, non-cylindrical PET scanner, list-mode PET data or something else. 
There are no restrictions on the geometry, but the reconstruction process is not optimized. This means that speed will most likely be impacted and the quality may not be the best possible. However, it should be possible
to reconstruct data from any geometry and even any modality as long as it can be reconstructed through ray-tracing means using voxel-basis. In this case ray-tracing refers to the computation of the intersection length of a ray in a single 
voxel/pixel. One thing to note is that by default OMEGA computes everything in emission tomography way. What this means, is that the intersection length is converted into probability. To avoid this, use ``options.CT = true``
(or ``options.CT = True`` in Python). This should be put before any reconstructions or before constructing the class object to compute your own algorithms.

The origin is always assumed to be in the center. You can move the center of the field-of-view (image) with the variables ``options.oOffsetX``, ``options.oOffsetY`` and ``options.oOffsetZ``.

Normally OMEGA loads all measurement and coordinate variables to the device (e.g. GPU). However, with list-mode data or custom coordinates, this can take a lot of device memory and lead to out-of-memory errors.
To circumvent this, set ``options.loadTOF = false`` and use subsets. This sends only the data needed for the current subset to the device. This will lead to slower reconstruction though, but can prevent
out-of-memory errors. See `Using high-dimensional data <https://omega-doc.readthedocs.io/en/latest/highdim.html>`_ for details.

There are two ways to perform reconstructions with custom scanner or list-mode data: index-based reconstruction and coordinate-based reconstruction.

For the choice of algorithm, PDHG is recommended, but for PET, and emission tomography in general, PKMA or OSEM can be good choices.

Index-based reconstruction
--------------------------

Index-based reconstruction is suitable for symmetric systems and is enabled by setting ``options.useIndexBasedReconstruction`` to true. It requires two index-vectors ``options.trIndex`` and ``options.axIndex``, as well as coordinate vectors ``options.x`` and ``options.z``. trIndex contains the transaxial, 
in this case x and y, indices that correspond to the coordinates in ``options.x``. The indices have to be in pairs, i.e. two indices are required for source-detector or detector-detector pairs. axIndex is the same, but for axial, 
i.e. z, coordinates. The indices also have to be zero-based! Each index pair must correspond to a measurement in ``options.SinM``. An example: You have 5 measurements, and two unique coordinates. 
In such case, for example, ``options.trIndex = [0 2 0 3 1 2 1 3 1 3]`` would point to the coordinates in the corresponding index. If ``options.x = [1.5 3 1 2]`` and the first two elements are x-coordinates and the next y-coordinates,
Thus 0 in trIndex would point to 1.5 in x, 2 to 1, and so on. Note that the order in ``options.x`` doesn't matter, but ``options.trIndex`` has to have both the x and y indices next to each other in memory, i.e. first x then y, then x, then y, etc. OMEGA also uses column-major
ordering and thus in Python you should use Fortran-ordered arrays or simply vectors. ``Inveon_PET_main_listmode_example`` contains a list-mode example of index-based reconstruction, but it is suitable for other types of data
as well. Since index-based reconstruction has both the index vectors and coordinate vectors, it is suitable only when the coordinate vectors are small. The index-vectors are 16-bit unsigned integers and thus can have a value of
65535 at most, but use less memory than the pure coordinate-based reconstruction showcased below. Note that ``options.trIndex`` and ``options.axIndex`` should have twice the number of elements than in ``options.SinM``, due to the 
pair nature of indices. In general, the indices are used to point to specific coordinates at each measurement. The order of the measurements and indices has to be the same! For example the above indices 0 and 2, should refer to the 
coordinates of the first measurement, 0 and 3 to the second measurement and so on. 

Here is also the same text as in the list-mode example: If true, requires ``options.trIndex`` and ``options.axIndex`` variables. These should include the transaxial and axial, respectively, detector
coordinate indices. Two indices are required per measurement, i.e. source-detector or detector-detector pairs. The indexing has to be zero-based! The transaxial coordinates should be stored in ``options.x`` and
axial coordinates in ``options.z``. The indices should correspond to the coordinates in each. Note that ``options.x`` should have both the x- and y-coordinates while ``options.z`` should have only the z-coordinates. You can
also include randoms by inputting them as negative measurements. The indices are used in the same order as measurements.

Coordinate-based reconstruction
-------------------------------

This method is simpler than the index-based one, but can consume more memory. Coordinate-based method also works with any type of geometry. You need to put the x-, y- and z-coordinates into ``options.x``, first for 
source or detector 1 and then for detector or detector 2, i.e. a total of six coordinates for one measurement. The order should be x, y and z or y, x, and z. The number of elements in ``options.x`` has to be six times that of
``options.SinM``, which has to contain the measurements. The order in ``options.x`` should thus be: [source1X, source1Y, source1Z, detector1X, detector1Y, detector1Z, source2X, source2Y, source2Z, detector2X, detector2Y, detector2Z,
source3X, ...], where the number refers to the index in ``options.SinM``, i.e. the measurement number. Source can also be another detector in PET cases. Since OMEGA is column-major, in Python you should use Fortran-ordering, or
for simplicity simply use vectors.

Same example as above, ``Inveon_PET_main_listmode_example``, showcases the coordinate-based reconstruction. Alternatively there is also ``custom_detector_exampleSimple`` and ``custom_coordinates_custom_algorithm_example``. Latter
shows how to compute your own custom algorithms using the built-in forward and/or backward projection operators.

Using list-mode data
--------------------

While the above two can be used for the coordinates of list-mode data, there are few special things to take into account when using list-mode data. First ``options.SinM`` should preferably contain only ones or minus ones. Negative
values should be used for randoms. If you add randoms manually, you don't actually need to set ``options.randoms_correction`` to true (in fact, it should stay false in such a case!). While the measurements are not really needed for list-mode reconstruction, OMEGA requires 
their inclusion at the moment.

Another thing to note is the computation of the sensitivity image, such as the one required by MLEM/OSEM. If the sensitivity image is computed with the same coordinates as the list-mode data, the reconstructions will fail. 
There are two alternatives, one is to compute the sensitivity image using all applicable LORs by using the built-in feature. This is, however, only applicable to cylindrical scanners and scanners where the reconstruction 
can be performed in normal, sinogram, mode. This feature can be enabled with ``options.compute_sensitivity_image``. Note that the scanner properties have to be correctly set to use this feature! Alternatively, simply use an
algorithm that doesn't require a sensitivity image, such as PDHG. 

TOF-data
--------

List-mode, or custom coordinate, data now also supports TOF reconstruction. This can be enabled with either the coordinate- or index-based reconstruction. The TOF data is included as indices that refer to TOF windows. This means that it is not recommended 
to have separate TOF bin/window for each measurement but rather to divide the data into TOF bins as with sinogram data. A maximum of 256 bins can be included by default. The TOF indices should be included into ``options.TOFIndicess`` 
variable that should be unsigned char (``uint8``) with zero-based indexing. The TOF time windows should be stored in ``options.TOFCenter``. The windows should start from the zero bin and then include the negative and positive bins,
for example ``options.TOFCenter = [0, 1, -1, 2, -2]``. In most settings, it should be enough to simply give the values specified in ``TOF PROPERTIES`` in the PET examples to automatically create the ``TOFCenter`` variable. In fact,
by default ``options.TOFCenter`` is formed internally and overwrites any user input variable. Thus, it is recommended to use the built-in variables and only provide ``options.TOFIndicess``. The nubmer of TOF indices has to equal the number of measurements
and the order needs to be the same! This means that the first TOF index should correspond to the first measurement, second to the second measurement, and so on.