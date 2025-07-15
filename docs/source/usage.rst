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

Here is a relatively complete list of all adjustable parameters and their default values (in MATLAB format):

| ``CT = false;``, if true computes the exact intersection length instead of probability. Also, when using built-in algorithms, uses the transmission tomography equivalents.
| ``options.use_CPU = false;``, if true, uses CPU to compute the reconstructions. Implementation 2 only. Not recommended and the available features is limited, for example projector types 4 and 5 are not supported.
| ``options.projector_type = 11;``, the default is 4 for CT and 11 for others. The first value refers to the forward projection and the second to the backprojection. With one value, the same method is used for both.
| ``options.PET = false;``, used internally only. Should not be adjusted by the user!
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
