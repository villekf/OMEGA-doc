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
SPECT data is input as CT/PET data regarding the variables ``options.SinM``, ``options.nRowsD``, ``options.nColsD``, ``options.nProjections``, ``options.FOVa_x``, ``options.FOVa_y``, ``options.axial_fov``, ``options.angles``, ``options.x``, ``options.z``, ``options.dPitchX`` and ``options.dPitchY``. However, if reconstructing the image from sinograms, the distance between detector surface and FOV center is required in ``options.radiusPerProj``. The crystal thickness is read from ``options.cr_p`` and the intrinsic resolution from ``options.iR``.

Modern full-ring 360° SPECT imaging devices, such as the Veriton-CT or StarGuide, have one additional detector swivel angle. That swivel angle can be input to ``options.swivelAngles`` and the distance from detector surface to swivel centre of rotation to  ``options.CORtoDetectorSurface``. 

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

Here is a relatively complete list of all adjustable parameters and their default values (in MATLAB format). In most cases you don't need to adjust the values. Note that any values you input will overwrite the default values.
All the below variables need to be input to the struct you give as input in MATLAB/Octave or to the class object in Python. The default name is ``options``, but it can be anything.

PET scanner variables
^^^^^^^^^^^^^^^^^^^^^
| ``options.dPitchX = 0;``, detector pitch (size, mm) in row dimension. For PET, you can also use ``options.cr_p``.
| ``options.dPitchY = 0;``, detector pitch (size, mm) in column dimension. For PET, you can also use ``options.cr_pz``. If you omit this, ``dPitchX`` will be used for column direction as well.
| ``options.cryst_per_block = 0;``, number of PET crystals per block. Required if you wish to use built-in geometry. Can be a vector of two variables if dual-layer PET. The second element is for outer layer.
| ``options.linear_multip = 1;``, number of PET axial blocks.
| ``options.blocks_per_ring = 1;``, the number of PET blocks per ring. Required if you wish to use built-in geometry.
| ``options.cryst_per_block_axial = options.cryst_per_block;``, the number of crystals per PET block in the axial direction. Can be a vector of two variables if dual-layer PET. The second element is for outer layer.
| ``options.transaxial_multip = 1;``, the number of crystal groups in one PET block.
| ``options.rings = options.linear_multip * options.cryst_per_block;``, the number of PET crystal rings
| ``options.det_per_ring = options.blocks_per_ring*options.cryst_per_block;``, number of detectors per ring.
| ``options.det_w_pseudo = options.det_per_ring;``, number of detectors per ring including pseudo detectors.
| ``options.detectors = options.det_w_pseudo*options.rings;``, total number of detectors.
| ``options.diameter = 1;`` diameter of the PET scanner bore (mm). Required if you wish to use built-in geometry.
| ``options.pseudot = [];``, the number of pseudo rings.
| ``options.PET = false;``, used internally only. Should not be adjusted by the user! Set to true if subset type is 8-11 with PET data.
| ``options.DOI = 0;``, the depth of interaction (mm). Basically the ray/tube starts DOI mm deeper from the crystal if this is non-zero.
| ``options.nLayers = 1;``, number of crystal layers. Relevant only for sinogram reconstruction and can be at most 2. Note that the experimental sinogram dual-layer PET only works in MATLAB/Octave,
| ``options.crystH = [];``, the crystal height (mm) of the innermost crystal for dual-layer PET. Note that diameter should not include this!

CT scanner variables
^^^^^^^^^^^^^^^^^^^^
| ``options.CT = false;``, if true computes the exact intersection length instead of probability. Also, when using built-in algorithms, uses the transmission tomography equivalents.
| ``options.dPitchX = 0;``, detector pitch (size, mm) in row dimension.
| ``options.dPitchY = 0;``, detector pitch (size, mm) in column dimension. If you omit this, ``dPitchX`` will be used for column direction as well.
| ``options.sourceOffsetRow = 0;``, the row offset (in mm) of the source location. Either a vector for all projections or a scalar.
| ``options.sourceOffsetCol = 0;``, same as above, but for column direction.
| ``options.detOffsetRow = [];``, same as above, but for detector.
| ``options.detOffsetCol = [];``, you know the drill.
| ``options.pitchRoll = [];``, the pitch/yaw/roll values of the detector panel. See :doc:`geometry`.
| ``options.sourceToCRot = 0;``, source to center-of-rotation-distance (mm).
| ``options.sourceToDetector = 1;``, source to detector distance (mm).
| ``options.nProjections = 1;``, total number of projections.
| ``options.binning = 1;``, binning value for CT projections when loading data. Only used when loading data with the built-in functions! If > 1, then the data is binned.
| ``options.nRowsD = 400``, the number of detector pixels in the row direction.
| ``options.nColsD = 1``, the number of detector pixels in the column direction.

SPECT scanner variables
^^^^^^^^^^^^^^^^^^^^^^^
| ``options.SPECT = false;``, signifies that the input is SPECT data and uses some SPECT specific settings.
| ``options.dPitchX = 0;``, detector pitch (size, mm) in row direction.
| ``options.dPitchY = 0;``, detector pitch (size, mm) in column direction.
| ``options.nRowsD = 0``, the number of detector pixels in the row direction.
| ``options.nColsD = 0``, the number of detector pixels in the column direction.
| ``options.nProjections = 0;``, total number of projections.
| ``options.iR = 0;``, detector intrinsic resolution.
| ``options.cr_p = 0;``, detector crystal thickness.
| ``options.CORtoDetectorSurface = 0;``, distance from collimator-detector interface to swivel center-of-rotation
| ``options.colL = 0;``, collimator hole length.
| ``options.colR = 0;``, collimator hole radius.
| ``options.colD = 0;``, distance between collimator and detector.
| ``options.colFxy = 0;``, collimator focal distance (transaxial).
| ``options.colFz = 0;``, collimator focal distance (axial).

Projector settings
^^^^^^^^^^^^^^^^^^
| ``options.projector_type = 11;``, the default is 4 for CT and 11 for others. The first value refers to the forward projection and the second to the backprojection. With one value, the same method is used for both. 1 = Improved Siddon, 2 = Orthogonal distance based, 3 = Volume of intersection, 4 = Interpolation, 5 = Branchless distance-driven (CT), 6 = Rotation (SPECT).
| ``options.tube_width_z = 0;``, the radius (mm) of the orthogonal "tube" when using projector type 2 (ODRT). If this is zero, the below value is used and ODRT is computed as 2D slices. If this is non-zero, then the below values is not used at all. This is the maximum orthogonal distance allowed!
| ``options.tube_width_xy = 0;``, the half width (mm) of the orthogonal "slice" when using projector type 2 (ODRT). This value is only used if the above is zero. This uses a 2D transaxial slice instead of 3D tube and can be thus faster to compute. This is the maximum orthogonal distance allowed in 2D case!
| ``options.tube_radius = sqrt(2) * (options.cr_pz / 2);``, the tube radius (mm) of the volume-of-intersection tube. Only used by projector type 3!
| ``options.voxel_radius = 1;``, the relative size of the radius of the spheres modeling the voxels with projector type 3. 1 means that the radius is the same as the radius of the cubic voxel, i.e. the voxel just fits inside the sphere.
| ``options.use_psf = false;``, if true, applies PSF blurring, use the below values to adjust the FWHM. For more details, see :doc:`selectingprojector`.
| ``options.FWHM = [options.dPitchX options.dPitchX options.dPitchY];``, the FWHM (in mm) of the PSF blurring.
| ``options.deblurring = false;``, if true, applies deblurring after the reconstruction.
| ``options.use_64bit_atomics = true;``, if true, uses 64-bit integer atomics in backprojection. Applies only to projector types 1-3. If false, uses 32-bit float atomics, which are slower, but slightly more accurate. Only applies to OpenCL.
| ``options.use_32bit_atomics = false;``, if true, uses 32-bit integer atomics in backprojection. Applies only to projector types 1-3. Takes precedence over the 64-bit atomic version above, and is both faster but also less accurate. Only applies to OpenCL.
| ``options.nRays = 1;``, the number of rays with projector type 1 when using SPECT data. Only applies to SPECT data!
| ``options.n_rays_transaxial = 1;``, the number of transaxial rays with projector type 1, when not using SPECT data. The total number of rays is this multiplied with the below value.
| ``options.n_rays_axial = 1;`` the number of axial rays with projector type 1, when not using SPECT data. The total number of rays is this multiplied with the above.
| ``options.meanFP = false;`` applies only to projector type 5 forward projection. This reduces the dynamic range of the integral images by subtracting the mean from it and then adding it back later. Should only be used if there are numerical issues.
| ``options.meanBP = false;`` same as above, but for backprojection.
| ``options.useFDKWeights = true;``, applies to FDK only. Uses specific FDK weights in backprojection if true.
| ``options.dL = 0;``, the interpolation length for projector type 4. The relative size of the voxel, i.e. 1 would be the voxel size, 0.5 half the voxel size, etc. The default value is actually equal to the size of one voxel in the x-direction, but this is computed later.
| ``options.useTotLength = true;``, PET only. If true, uses the entire ray/tube length/volume to compute the probability, i.e. from detector to detector. If false, only the ray/tube inside the FOV is used.

Image volume settings
^^^^^^^^^^^^^^^^^^^^^
| ``options.Nx/Ny/Nz``, the number of voxels per each dimension. Nx and Ny HAVE to be input, but Nz is assumed to be 1 by default.
| ``options.flip_image = false;`` if true, flips the image in the horizontal direction. This is done in the detector space and thus has no effect on the quality or speed of the reconstruction.
| ``options.offangle = 0;``, rotates the image in the detector space with the specified amount. This is number of crystals in PET and degrees or radians in CT and SPECT. It is counter-clockwise for PET and CT, and clockwise for SPECT.
| ``options.oOffsetX = 0;``, offset value (mm) of the center of the image FOV in x-direction. Move the FOV with this and the below values if it's not centered on origin.
| ``options.oOffsetY = 0;``, offset value (mm) of the center of the image FOV in y-direction. 
| ``options.oOffsetZ = 0;``, offset value (mm) of the center of the image FOV in z-direction. 
| ``options.FOVa_y = options.FOVa_x;``, FOVa_x (mm) should always be input, but FOVa_y is not necessary. If not input, FOVa_y uses the same values as FOVa_x. These are the transaxial FOV sizes.
| ``options.axial_fov = 0;``, axial FOV (mm). This should always be greater than zero! If omitted, will use the size of the detector in the axial-direction, i.e. ``dPitchY``.
| ``options.x0 = ones(options.Nx, options.Ny, options.Nz) * 1e-5;``, initial value of the reconstruction.

Sinogram settings
^^^^^^^^^^^^^^^^^
| ``options.span = 3;``, the span/axial compression value of the sinogram.
| ``options.Ndist = 400;``, number of radial positions (views) in sinogram.
| ``options.Nang = options.det_w_pseudo / 2;``, number of angles (tangential positions) in sinogram.
| ``options.ring_difference = options.rings - 1;``, the maximum ring difference in the sinogram.
| ``options.NSinos = 1;``, total number of sinograms used in the reconstruction.
| ``options.TotSinos = options.NSinos;``, the total number of sinograms.
| ``options.segment_table``, there are two different default values for this, depending on the ``span`` value. The amount of sinograms contained in each segment. If omitted, will be computed.
| ``options.ndist_side = 1;``, if ``Ndist`` is even and you are loading sinogram data with OMEGA, this specifies where the "extra" row is taken. Can be either 1 or -1.
| ``options.sampling = 1;`` increase the sampling of the sinogram. I.e. interpolates more rows/columns to the sinogram.
| ``options.sampling_interpolation_method = 'linear';``, the interpolation type. All the methods are available that are supported by interp2 (see ``help interp2``) or ``interpn`` in Python.
| ``options.fill_sinogram_gaps = false;``, if using pseudo detectors, you can interpolate those gaps in the sinogram when this is true. MATLAB/Octave only!
| ``options.gap_filling_method = 'fillmissing';``, the type of filling method for sinogram gaps. Either MATLAB's built-ins ``fillmissing`` or ``fillmissing2``, or ``inpaint_nans`` from file exchange. 
| ``options.interpolation_method_fillmissing = 'linear';``, interpolation method for ``fillmissing`` and ``fillmissing2``.
| ``options.interpolation_method_inpaint = 0;``, interpolation method ``inpaint_nans``.

Computing settings
^^^^^^^^^^^^^^^^^^
| ``options.implementation = 2;``, the used implementation. MATLAB/Octave only! See :doc:`implementation` for details. In Python, the implementation is always 2.
| ``options.use_CPU = false;``, if true, uses CPU to compute the reconstructions. Implementation 2 only. Not recommended and the available features is limited, for example projector types 4 and 5 are not supported.
| ``options.use_CUDA = checkCUDA(options.use_device);``, if true, uses CUDA instead of OpenCL. On MATLAB/Octave the default value checks whether the selected device is CUDA-capable. On Python, the default is false.
| ``options.useSingles = true;``, MATLAB/Octave only and implementation 4 only! If false, uses double precision instead when computing implementation 4 reconstructions.
| ``options.largeDim = false;``, if true, uses :doc:`highdim`. Built-in algorithms only!
| ``options.loadTOF = true;``, if false, only the current subset is transfered to the GPU. See :doc:`highdim`. Built-in algorithms only!
| ``options.storeResidual = false;`` if true, outputs the residual, or primal-dual gap with PDHG and its variants for each (sub-)iteration. Works only for LS-based algorithms! Built-in algorithms only!
| ``options.usingLinearizedData = false;``, if true, doesn't linearize the data if the algorithm requires linearized data.
| ``options.use_device = 0;``, the GPU/OpenCL device used. In Python, this is ``deviceNum``.
| ``options.platform = 0;``, the used OpenCL platform number. MATLAB/Octave only! Implementation 3 and 5 only! Does affect implementation 2 when using the projector operators, but not when using built-in algorithms.
| ``options.useMAD = true;``, if true uses MAD with OpenCL and CUDA. Can increase computational speed but decrease accuracy very slightly.
| ``options.useImages = true;``, if true, uses OpenCL images/CUDA textures, instead of buffers. If you're using CPU as an OpenCL device, setting this to ``false`` might speed up reconstructions.

TOF settings
^^^^^^^^^^^^
| ``options.TOF_bins = 1;``, the number of TOF bins.
| ``options.TOF_bins_used = options.TOF_bins;``, the number of TOF bins used. Needs to be either the same as above or 1, in which case the TOF bins are summed together.
| ``options.TOF_FWHM = 0;``, TOF FWHM value in s.
| ``options.TOF_width = 0;``, TOF width of each bin in s.

Correction settings
^^^^^^^^^^^^^^^^^^^
| ``options.randoms_correction = false;``, if true, applies randoms correction. Randoms data must be supplied either in ``SinDelayed`` or in precomputed mat-file!
| ``options.SinDelayed = [];``, the randoms data.
| ``options.variance_reduction = false;``, if true, applies variance reduction to randoms data. 
| ``options.randoms_smoothing = false;``, if true, smooths the randoms with 7x7 moving mean.
| ``options.scatter_correction = false;``, if true, applies scatter correction. Scatter data must be input by the user into ``ScatterC``.
| ``options.ScatterC = [];``, the scatter data.
| ``options.scatter_variance_reduction = false;``, same as above, but for scatter.
| ``options.normalize_scatter = false;``, apply normalization correction to scatter data.
| ``options.scatter_smoothing = false;``, apply the above smoothing to scatter data.
| ``options.subtract_scatter = true;``, if false, the scatter data is multiplied with the forward and/or backprojection. Thus the correction becomes multiplicative. If false, the scatter is subtracted either as a precorrection or added to the forward projection in ordinary Poisson way.
| ``options.additionalCorrection = false;``, if true, the user needs to input correction data into ``corrVector``. This can be any kind of correction data in addition to any of the others and is multiplied with the forward/backward projection just like normalization, for example. Works for any data.
| ``options.attenuation_correction = false;``, if true, applies attenuation correction. Attenuation data must be present in ``vaimennus`` or input as a file into ``options.attenuation_datafile``. Note that the attenuation image, or sinogram, has to be equal to the ones used in the reconstruction! You also need to scale it for the energy used.
| ``options.attenuation_datafile = '';``, the path to the optional attenuation file, such as a mat-file or MetaImage file. Use the header for MetaImage or Interfile.
| ``options.rotateAttImage = 0;``, rotates the attenuation image with N * options.rotateAttImage degrees. 
| ``options.vaimennus = [];``, you can also manually input the attenuation data to here.
| ``options.CT_attenuation = true;``, by default, it is assumed that the attenuation data is an attenuation image, such as one derived from CT. However, if this is false, the attenuation data is assumed to be attenuation sinogram instead.
| ``options.SinM = [];``, if the measurement data is not automatically loaded (such as when using GATE data created by OMEGA), you can input the measurement data into this.
| ``options.compute_normalization = false;```, MATLAB/Octave only! Computes the normalization coefficients using the current measurement data if true and the function ``normalization_coefficients()`` is present. See ``help normalization_coefficients`` for more details.
| ``options.normalization_options = [1 1 1 1];`` Four different components can be computed in ``normalization_coefficients()``. This selects which of those are computed (1 included, 0 excluded).
| ``options.normalization_phantom_radius = inf;``, if the normalization measurement is a physical cylinder, input its radius here in cm. Otherwise, use inf.
| ``options.normalization_attenuation = [];``, if two values are present, computes attenuation for the normalization measurement. Needs to have the cylinder radius as cm and attenuation coefficient cm^2/m.
| ``options.normalization_scatter_correction = false;``, if true, attempts to perform scatter correction to the normalization measurement. Should be used only with a cylinder source.
| ``options.normalization_correction = false;``, if true, applies normalization correction to the reconstruction, either as a precorrection or during the reconstruction.
| ``options.use_user_normalization = false;``, if true, then assumes that the user has the normalization correction data. This will be prompted and should be either a mat-file or nrm-file.
| ``options.normalization = []``, alternatively, input the normalization data into here.
| ``options.arc_correction = false;``, if true, applies arc correction to the sinogram.
| ``options.arc_interpolation = 'linear';``, interpolation type of the arc correction. Applies only if above is true. See ``scatteredInterpolant`` or ``griddata`` (fallback, in Octave the only option) in MATLAB. See ``interpn`` in Python.
| ``options.global_correction_factor = [];`` a global constant that is applied to all forward/backward projection computations.
| ``options.corrections_during_reconstruction = true;``, if true, all corrections are applied during reconstruction. If false, randoms, scatter, and/or normalization correction is/are done as a precorrection. Attenuation correction is always applied during the reconstruction.
| ``options.useEFOV = false;``, use extended FOV.
| ``options.eFOVLength = 0.4;``, the extended FOV size, per side. I.e. the total size is increased by 80% with this value.
| ``options.useExtrapolation = false;``, use projection extrapolation if true.
| ``options.extrapLength = 0.2;``, the extrapolated projection amount per side. I.e. the total size is increased by 40% with this value.
| ``options.useExtrapolationWeighting = false;``, if true, uses a log-based weighting on the extrapolated projection parts. This means that the values will "fade" into air values using logarithmic weighting. Can be useful if the regions in the projections should not continue for the whole extrapolated length.
| ``options.useInpaint = false;``, if true, uses the MATLAB file exchange function ``inpaint_nans`` to perform the extrapolation of the projections. Not recommended. Requires ``inpaint_nans``. MATLAB only!
| ``options.useMultiResolutionVolumes = false;``, use multi-resolution reconstruction. Replaces the extended FOV region with a region with different voxel size. See below for the parameter to adjust.
| ``options.multiResolutionScale = .25;``, the reduced resolution of multi-resolution volumes. In this case .25 means that the resolution is 25% of the original, i.e. voxel size is 4 times larger.
| ``options.offsetCorrection = false;``, if true, uses offset correction.
| ``options.ordinaryPoisson = options.corrections_during_reconstruction``, even if ``options.corrections_during_reconstruction`` is set to true, setting this to false will cause randoms and/or scatter to be precorrected.

GATE specific settings
^^^^^^^^^^^^^^^^^^^^^^
| ``options.obtain_trues = false;``, applies to GATE data load only. If true, stores trues separately in ``SinTrues``.
| ``options.reconstruct_trues = false;``, if the trues have been separately stored, reconstruct those if true.
| ``options.store_scatter = false;``, applies to GATE data load only. If true, stores scatter separately. Set the type of scatter to store using the below variable. Stored in ``SinScatter``.
| ``options.scatter_components = [1 1 0 0];``, the scatter to store. First: Compton in phantom. Second: Compton in detector. Third: Rayleigh in phantom. Fourth: Rayleigh in detector.
| ``options.reconstruct_scatter = false;``, if true, reconstructs the previously stored scatter components.
| ``options.store_randoms = false;``, applies to GATE data load only. If true, stores randoms separately. Stored in ``SinRandoms``.
| ``options.source = false;``, applies to GATE data load only. If true, stores the "source image" or "ground truth", i.e. the number of photons emitted and detected per voxel. Uses the source coordinates the compile the ground truth image.

Reconstruction specific settings
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
| ``options.subsets = 1;``, the number of subsets. Note that this has to be minimum of 1! Note also that if you use forward/backward projection operators and want to use automatic subsets, you need to specify this before creating the class object (MATLAB/Octave) or initializing it (Python).
| ``options.subset_type = 8;``, the subset selection type. See :doc:`algorithms`. For PET data, it is recommended to use 1 instead of 8. In Python, this is ``subsetType``, which can be used in MATLAB/Octave as well.
| ``options.stochasticSubsetSelection = false;``, if true, the subsets are selected stochastically.
| ``options.bedOffset = 0;``, offset value of the bed for multi-bed case, i.e. how much is the bed moved. Should contain the offsets for each bed position. Built-in algorithms only!
| ``options.nBed = 1;``, number of bed positions. Built-in algorithms only!
| ``options.save_iter = false;``, if true, saves ALL intermediate iterations. Only full iterations, not sub-iterations. Built-in algorithms only!
| ``options.saveNIter = uint32([]);``, save specific iteration numbers. This uses zero-based indexing so 0 is the result of the first iteration, 1 the second, etc.
| ``options.Niter = 1;``, the total number of iterations when using built-in algorithms.
| ``options.enforcePositivity = true;``, if true, enforces positivity with most algorithms, but not CGLS or LSQR. Note that most Poisson-based algorithms already are inherently positive. Built-in algorithms only!
| ``options.FISTA_acceleration = false;``, if true, applies FISTA-type acceleration (momentum-based). Built-in algorithms only!
| ``options.storeFP = false;``, if true, stores ALL forward projections. Built-in algorithms only! This is the ``fp`` variable in some examples. Note that if you use subsets, the forward projection will be in the subset ordering, not in the original ordering. When not using subsets, these are automatically resized to the measurement data size.

Preconditioner settings
^^^^^^^^^^^^^^^^^^^^^^^
| ``options.precondTypeImage = [false;false;false;false;false;false;false];``, the selected image-based preconditioners. See `Preconditioners <https://omega-doc.readthedocs.io/en/latest/algorithms.html#preconditioners>`_.
| ``options.precondTypeMeas = [false;false];``, the selected measurement-based preconditioners.
| ``options.filteringIterations = 0;``, the number of filtering iterations when using either of the filtering-based preconditioners. This includes sub-iterations! For example with 2 iterations and 10 subsets, using 15 here would use the filtering with one full iteration and half of the subsets in the next iteration.
| ``options.gradV1 = 0.5;``, only used by precondTypeImage(5). See the article for details in :doc:`algorithms`.
| ``options.gradV2 = 2.5;``, only used by precondTypeImage(5). See the article for details in :doc:`algorithms`.
| ``options.gradInitIter = options.subsets;``, the iteration when precondTypeImage(5) is first used and computed.
| ``options.gradLastIter = options.gradInitIter;``, the last iteration when the gradient of precondTypeImage(5) is computed. After this, the same gradient is used.
| ``options.filterWindow = 'hamming';``, windowing type when using filtering-based preconditioners or FDK/FBP. Available windows are: hamming, hann, blackman, nuttal, parzen, cosine, gaussian, shepp-logan and none. 
| ``options.cutoffFrequency = 1;``, the cut-off frequency of the filtering window.
| ``options.normalFilterSigma = 0.25;``, the sigma value of the Gaussian windowing.

Dynamic imaging settings
^^^^^^^^^^^^^^^^^^^^^^^^
| ``options.tot_time = inf;``, the total time. Applies only when loading dynamic data with built-in functions!
| ``options.partitions = 1;``, either the number of time steps, or the size of each time step in seconds. The latter is only used when loading data with the built-in functions! Relevant only for dynamic data.
| ``options.start = 0;``, start time, as with above applies only with built-in functions loading the data. The data are thus saved starting from this time.
| ``options.end = options.tot_time;``, end time, as with above applies only with built-in functions loading the data. The data are thus saved up to this point.

Misc. settings
^^^^^^^^^^^^^^
| ``options.machine_name = '';``, the name of the scanner. Used only when loading and/or saving data.
| ``options.compute_sensitivity_image = false;``, only applies to listmode or similar type of data. If true, computes the sensitivity image using the scanner geometry instead of computing it from the measurements. Since the measurements can skip certain combinations or have more than one for some, setting this to true should give better quality. However, you'll need to fill the scanner properties to use this.
| ``options.useIndexBasedReconstruction = false;```, if true, uses index-based reconstruction. See :doc:`customcoordinates.
| ``options.epps = 1e-5;``, a small constant to prevent division by zero.
| ``options.verbose = 1;``, the verbosity level. See above.

Algorithm settings
^^^^^^^^^^^^^^^^^^
| ``options.tauCP = 0;``, primal value for PDHG, and its variants, and FISTA. Computed automatically if zero or empty.
| ``options.thetaCP = 1;``, the update parameter for PDHG estimates.
| ``options.sigmaCP = 1;``, dual value for PDHG. If zero, sigma will be identical to tau and both are scaled accordingly.
| ``options.sigma2CP = options.sigmaCP;`` dual value for TV or TGV. To increase convergence speed, it can be useful to use larger dual values for the regularization.
| ``options.tauCPFilt = 0;``, primal value for the filtered steps when using filtering-based preconditioner. Computed automatically if zero or empty.
| ``options.powerIterations = 20;``, the number of power iterations performed to determine the primal value. Only used if primal value is zero or empty.
| ``options.PDAdaptiveType = 0;``, adaptively updates the primal and dual values if 1 or 2. Different methods are used with 1 or 2, see :doc:`algorithms`.
| ``options.computeRelaxationParameters = false;``, experimental feature that computes relaxation parameters based on the data, if true. Not really recommended.
| ``options.relaxationScaling = false;``, experimental feature that tries to scale the relaxation parameters, if true. Not really recommended.
| ``options.lambda = [];``, the relaxation parameters for all algorithms using relaxation. In Python this is ``lambdaN``. If omitted, will be computed internally. See RAMLA in :doc:`algorithms`.
| ``options.h = 2;`` acceleration factor for ACOSEM.
| ``options.U = 10000;``, upper bound for MBSREM/MRAMLA.

Regularization settings
^^^^^^^^^^^^^^^^^^^^^^^^
| This section is still incomplete. See :doc:`algorithms` for more details.
| ``options.beta = 0;``, the regularization parameter.
| ``options.med_no_norm = false;``, if true, disables the normalization step when using MRP. Only affects MRP!
| ``options.Ndx = 1;``, neighborhood size in x-direction on one side. The total amount is always (Ndx * 2 + 1). See :doc:`algorithms`.
| ``options.Ndy = 1;``, neighborhood size in y-direction on one side. The total amount is always (Ndy * 2 + 1). See :doc:`algorithms`.
| ``options.Ndz = 1;``, neighborhood size in z-direction on one side. The total amount is always (Ndz * 2 + 1). See :doc:`algorithms`.
| ``options.weights = [];``, weights vector for quadratic prior.
| ``options.weights_huber = [];`´, weights vector for Huber prior.
| ``options.a_L = [];``, weights vector for L-filter.
| ``options.oneD_weights = false;``, if true, 1D filter weights are used, otherwise 2D.
| ``options.fmh_weights = [];``, weights vector for FMH.
| ``options.fmh_center_weight = 4;``, the weight value for the center voxel of FMH.
| ``options.mean_type = 4;``, mean type for weighted mean. 1/4 = Arithmetic mean, 2/5 = Harmonic mean, 3/6 = Geometric mean. 1-3 are similar to MRP, while 4-6 compute the gradient.
| ``options.weighted_weights = [];``, weights vector for weighted mean.
| ``options.weighted_center_weight = 4;``, the weight value for the center voxel of weighted mean.
| ``options.TVsmoothing = 1e-2;``, the "smoothing" value of TV, which prevents division and square root of zero.
| ``options.TV_use_anatomical = false;``, if true, uses anatomical weighting with TV, when supported.
| ``options.TVtype = 1;``, the "type" of TV. See :doc:`algorithms`. 
| ``options.APLSsmoothing = 1e-5;``, the "smoothing" value of APLS, which prevents division and square root of zero.
| ``options.sigma = 1;``, the filtering parameter of non-local methods. Higher values smooth the image, smaller values make it sharper.
| ``options.Nlx = 1;``, the non-local patch window size in x-direction. The total amount is always (Nlx * 2 + 1). See :doc:`algorithms`.
| ``options.Nly = 1;``, the non-local patch window size in y-direction. The total amount is always (Nly * 2 + 1). See :doc:`algorithms`.
| ``options.Nlz = 1;``, the non-local patch window size in z-direction. The total amount is always (Nlz * 2 + 1). See :doc:`algorithms`.
| ``options.RDP_gamma = 1;``, RDP "gamma" value that controls the edge preservation. Higher values sharpen the image, lower values smooth it.
| ``options.use2DTGV = false;``, if true, computes the TGV only on 2D (slice) region, without taking into account the 3rd dimension. Reduces memory cost, but decreases reconstruction quality.
| ``options.useL2Ball = true;``, if true, proximal TV and TGV are computed using L2 (ball). If false, L1 (ball) is used instead. Should have only a marginal effect on the reconstruction.