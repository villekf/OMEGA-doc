Usage
=====

.. contents:: Table of Contents

Geometry information
--------------------

For more details on the geometry, see :doc:`geometry`.

For all data and modalities, there are two different ways to set up the geometry. One is a built-in method that is different for each modality and requires specific type of scanner(s). Another is user-input coordinates for each measurement. 
This page details the built-in features, but any type of data can be used if the geometry is defined completely manually through source-detector (or detector-detector) coordinate pairs. For the manual case, see :doc:`customcoordinates`.

Whenever inputing your own values, make sure the data is Fortran-ordered in Python!

PET data
^^^^^^^^

Built-in support is available for cylindrical PET scanners with block/bucket type setting (as shown in :doc:`geometry`). If the below parameters are specified correctly, the coordinates are computed automatically.

A necessary parameter to input is ``options.blocks_per_ring``, which is the number of blocks/buckets in the transaxial direction. This is the total number of blocks/buckets as shown in the geometry page in the transaxial direction, 
i.e. when looking to the bore. 

This is not necessary parameter and is only necessary if there are gaps between adjacent rings. ``options.linear_multip`` is the number of blocks/buckets in the axial direction, i.e. when moving along the bore direction.

Instead of using the above, it is also possible to use ``options.cryst_per_block_axial`` instead, IF there are no gaps between the rings. Otherwise, this should be number of crystals per block in the axial direction. Multiplying this
with the above ``linear_multip`` has to be the total number of rings!

A necessary parameter is also ``options.cryst_per_block`` which is the number of crystals per block in the transaxial direction. This multiplied with ``options.blocks_per_ring`` has to equal the number of crystals per ring!

If there are gaps between the rings, these can be input into ``options.ringGaps`` and the number of elements should equal to ``options.linear_multip`` - 1. The gaps are the size of the gap in mm. This is required ONLY if there are gaps!

``options.diameter`` is a compulsory parameter that is the distance diameter of the bore.

The crystal pitch in mm (size) is compulsory and is defined for transaxial direction with ``options.cr_p`` and for axial with ``options.cr_pz``.

The transaxial FOV is defined by ``options.FOVa_x`` for the horizontal size and ``options.FOVa_y`` for the vertical. ``options.FOVa_y`` can be omitted, but ``options.FOVa_x`` has to be input!

Axial FOV is defined with ``options.axial_fov`` and has to be input, even if you have only one ring!

If the scanner has pseudo-rings, the number of those can be input with ``options.pseudot``. You can omit this if there are no pseudo-rings.

``options.det_per_ring``, ``options.rings`` and ``options.detectors`` can be omitted, but ``options.det_w_pseudo`` has to be included if, and only if, pseudo-rings are included.

``options.machine_name`` is important if loading previously saved data or saving measurement, such as GATE, data. If you preload the measurement data into ``options.SinM``, you can omit this, otherwise include some name.

PET data also needs the following sinogram properties, if using sinogram data:

``options.span`` is the span factor/axial compression (default value is 3). ``options.ring_difference`` maximum ring difference (default is number of rings - 1). ``options.Ndist`` number of radial positions/views in the sinogram (default value is 400). 
``options.Nang`` number of angles (tangential positions) in sinogram (can be omitted, in which case it is assumed to be half the number of detectors per ring).
``options.segment_table`` the amount of sinograms contained on each segment. Note that this is only required if ``options.span`` > 1 and is computed automatically if omitted. ``options.TotSinos`` the total number of sinograms.
``options.NSinos`` the total number of sinograms used in the reconstructions, if smaller than ``options.TotSinos`` only the first ``options.NSinos`` sinograms are taken. It is possible to omit ``options.TotSinos``. 
``options.ndist_side`` is an optional value that is used only if the sinogram is created by OMEGA. If Ndist value is even, take one extra out of the negative side (+1) or from the positive side (-1). If you have sinogram data with pseudo 
detectors, you can interpolate the gaps by setting ``options.fill_sinogram_gaps = true``, note that this is MATLAB/Octave only! See the ``Biograph_mCT_main.m`` for more details on the gap filling.

For TOF data, see :doc:`tof`.

Note that the origin is assumed to be in the center of the scanner and by default that is also the origin of the FOV/image. If you want to move the FOV, use ``options.oOffsetX``, ``options.oOffsetY`` and ``options.oOffsetZ`` values.

Depth of interaction (DOI) effect can be somewhat included with ``options.DOI`` parameter. This simply assumes that the absorption point is not at the edge of the crystal but the specified depth (in mm) from the surface. This is a constant
value.

For dual, or multi, layer data, you should use index-based reconstruction, see :doc:`customcoordinates`. It is possible to use dual-layer data by setting ``options.nLayers = 2``, but this is not a recommended method. The sinogram parameters need to be modified
accordingly. If the size of the layers is different, you need to use pseudo detectors and rings. This is, however, not required with index-based reconstruction.

CT data
^^^^^^^

For CT data, the built-in geometry allows the use of cone beam CT data with flat panel. However, there are many ways to define the geometry of the source and/or detector.

In all cases, regardless of the source-detector geometry, the following variables are needed:

``options.nRowsD`` is the number of rows in the projection image. ``options.nColsD`` the number of columns. ``options.nProjections`` is the total number of projections. ``options.dPitchX`` is the size of single detector pixel in the row direction and 
``options.dPitchY`´ in the column direction. ``options.sourceToDetector`` is the source-to-detector distance. ``options.sourceToCRot`` is the source-to-center-of-rotation distance.

The transaxial FOV is defined by ``options.FOVa_x`` for the horizontal size and ``options.FOVa_y`` for the vertical. ``options.FOVa_y`` can be omitted, but ``options.FOVa_x`` has to be input! Axial FOV is defined with ``options.axial_fov`` 
and has to be input, even if you have only one column/row!

To input the source-detector geometry, there are multiple ways to achieve that. One is to let OMEGA handle as much as possible. If the source and detector are not shifted at all, then only the projection angles are needed: ``options.angles`` in either
degrees or radians. You can move the source in the row direction with ``options.sourceOffsetRow`` and in the column direction with ``options.sourceOffsetCol``. For detector, the same is possible with ``options.detOffsetRow`` and
``options.detOffsetCol``. In both shift cases, the variable can be either a scalar or vector. If vector, the number of elements has to equal the number of projections.

You can also input the coordinates of the source and center of the detector for each projection. These are input as pairs into ``options.x``, ``options.y`` and ``options.z``, i.e. first source then detector center. In case the panel rotates in other 
directions at each projection, you can input these into ``options.pitchRoll``, which is again in pairs. Alternatively, you can input the direction vectors of the panel at each projection to ``options.uV``. With either 2 or 6 elements per projection.
See :doc:`geometry` for more details on the ``pitchRoll``.

SPECT data
^^^^^^^^^^

Any data
^^^^^^^^

The number of voxels per dimension is defined with ``options.Nx``, ``options.Ny``, and ``options.Nz``. The image volume can be rotated in the measurement space by using ``options.offangle``. The behavior is slightly different depending on modality.
With PET, this is the number of detector elements, for CT it is the angle in radians and for SPECT it is the angle in radians. The direction is counter-clockwise in PET and CT and clockwise in SPECT.

Verbosity
---------

This largely applies only to built-in reconstruction, but the verbosity can be adjusted with ``options.verbose``, where 1 is the default value. This gives timing information for the whole reconstruction and shows when a sub-iteration and/or iteration
has been computed. Verbosity of 0 gives no messages and will be largely silent reconstruction. Verbosity of 2 gives more accurate timing information, such as time taken per sub-iteration and iteration, the estimated time left and the total time 
taken for the reconstruction process itself. Verbosity 3 increases the timing messages to be even more specific, but will also output some debugging messages. Verbosity of 1 or 2 is recommended.

Adjustable parameters
---------------------

Here is a relatively complete list of all adjustable parameters and their default values (in MATLAB format). In most cases you don't need to adjust the values.

| ``CT = false;``, if true computes the exact intersection length instead of probability. Also, when using built-in algorithms, uses the transmission tomography equivalents.
| ``options.use_CPU = false;``, if true, uses CPU to compute the reconstructions. Implementation 2 only. Not recommended and the available features is limited, for example projector types 4 and 5 are not supported.
| ``options.projector_type = 11;``, the default is 4 for CT and 11 for others. The first value refers to the forward projection and the second to the backprojection. With one value, the same method is used for both.
| ``options.PET = false;``, used internally only. Should not be adjusted by the user! Set to true if subset type is 8-11 with PET data.
| ``options.SPECT = false;``, signifies that the input is SPECT data and uses some SPECT specific settings.
| ``options.useSingles = true;``, MATLAB/Octave only and implementation 4 only! If false, uses double precision instead when computing implementation 4 reconstructions.
| ``options.largeDim = false;``, if true, uses :doc:`highdim`.
| ``options.loadTOF = true;``, if false, only the current subset is transfered to the GPU. See :doc:`highdim`.
| ``options.storeResidual = false;`` if true, outputs the residual, or primal-dual gap with PDHG and its variants for each (sub-)iteration. Works only for LS-based algorithms!
| ``options.sourceToCRot = 0;``, source to center-of-rotation-distance.
| ``options.sourceToDetector = 1;``, source to detector distance.
| ``options.rotateAttImage = 0;``, rotates the attenuation image with N * options.rotateAttImage degrees. 
| ``options.cryst_per_block = 0;``, number of PET crystals per block.
| ``options.linear_multip = 1;``, number of PET axial blocks.
| ``options.dPitchX = 0;``, detector pitch (size) in row dimension.
| ``options.dPitchY = 0;``, detector pitch (size) in column dimension.
| ``options.usingLinearizedData = false;``, if true, doesn't linearize the data if the algorithm requires linearization.
| ``options.TOF_bins = 1;``, the number of TOF bins.
| ``options.TOF_bins_used = 1;``, the number of TOF bins used. Needs to be either the same as above or 1, in which case the TOF bins are summed together.
| ``options.TOF_FWHM = 0;``, TOF FWHM value in s.
| ``options.TOF_width = 0;``, TOF width of each bin in s.
| ``options.cryst_per_block_axial = options.cryst_per_block;``, the number of crystals per PET block in the axial direction.
| ``options.transaxial_multip = 1;``, the number of crystal groups in one PET block.
| ``options.machine_name = '';``, the name of the scanner. Used only when loading and/or saving data.
| ``options.sourceOffsetCol = 0;``, the column offset (in mm) of the source location. Either a vector for all projections or a scalar.
| ``options.sourceOffsetRow = 0;``, same as above, but for row direction.
| ``options.detOffsetRow = [];``, same as above, but for detector.
| ``options.detOffsetCol = [];``, you know the drill.
| ``options.subsets = 1;``, the number of subsets. Note that this has to be minimum of 1!
| ``options.subset_type = 8;``, the subset selection type. See :doc:`algorithms`. For PET data, it is recommended to use 1 instead of 8.
| ``options.offsetCorrection = false;``, if true, uses offset correction.
| ``options.bedOffset = 0;``, offset value of the bed for multi-bed case. Should contain the offsets for each bed position.
| ``options.pitchRoll = [];``, the pitch/yaw/roll values of the detector panel. See :doc:`geometry`.
| ``options.nBed = 1;``, number of bed positions.
| ``options.flip_image = false;`` if true, flips the image in the horizontal direction. This is done in the detector space and thus has no effect on the quality or speed of the reconstruction.
| ``options.offangle = 0;``, rotates the image in the detector space with the specified amount. This is number of crystals in PET and degrees or radians in CT and SPECT. It is counter-clockwise for PET and CT, and clockwise for SPECT.
| ``options.tube_width_z = 0;``, the width of the orthogonal "tube" when using projector type 2 (ODRT). If this is zero, the below value is used and ODRT is computed as 2D slices. If this is non-zero, then the below values is not used at all.
| ``options.tube_width_xy = 0;``, the width of the orthogonal "slice" when using projector type 2 (ODRT). This value is only used if the above is zero. This uses a 2D transaxial slice instead of 3D tube and can be thus faster to compute.