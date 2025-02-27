Technical info
===============

This page outlines some technical information on the software. Mainly how everything works. The text is largely based on version 1.2 and thus mainly features PET-related information in MATLAB. This page is still work in progress!

The general structure of OMEGA can be divided into three different layers. The top layer is the MATLAB/Octave/Python user-interface that contains the scripts and functions necessary to call the lower layers. 
The middle layer is the MATLAB (C) MEX-interface or C++ dynamic library that calls and computes the C++ code and then sends it back to the top layer. The bottom layer, which is not always used, contains the OpenCL/CUDA kernels that 
compute the OpenCL/CUDA code and then send the output data (reconstructed images) to the middle layer. The bottom layer is only used if OpenCL/CUDA code is used. The middle layer can also be ignored, but this is recommended only when
doing custom reconstructions in Python.

`GNU Octave <https://octave.org/>`_
For the user, only the top layer is exposed. This is achieved in the form of scripts that are referred to as *main-files*. These include, for example, `gate_main.m <https://github.com/villekf/OMEGA/blob/master/main-files/PET_main_gateExample.m>`_,
`CT_main_full.m <https://github.com/villekf/OMEGA/blob/master/main-files/CT_main_full.m>`_, or `gate_PET.py <`load_data.m <https://github.com/villekf/OMEGA/blob/master/source/Python/gate_PET.py>`_.
It is from these main-files that the actual functions are called. This
is achieved by storing all the user selected parameters to a
MATLAB/Octave/Python struct called ``options`` in the example files (the name can be anything though), which is then input to the
functions (e.g. when loading data or performing image reconstruction).

In the main-files most parameters are set either with numerical values
(e.g. ``blocks_per_ring``, ``span``, ``tube_radius``) or as a true/false
selection, where true means that the feature is included and false that
it is omitted (e.g. ``randoms_correction``, ``scatter_smoothing``,
``osem``). The main-files are divided into “sections” with specific
labels (SCANNER PROPERTIES, IMAGE PROPERTIES, SINOGRAM PROPERTIES, etc.)
that control different aspects. Many of these sections are completely
optional, e.g. CORRECTIONS section can be omitted if the user does not
wish to use any corrections. The only compulsory items are the FOV size, the number of voxels per dimension, and the source/detector coordinates for each measurement (and the measurement data itself).

There are also several functions that work very independently without
the need for the main-files. These include file import and export
functions, and visualization functions. For help on many of these functions, you should use
``help function_name`` or alternatively ``doc function_name``. E.g.
``help saveImage`` in MATLAB/Octave.

Data load
---------

The corresponding m-files are
`load_data.m <https://github.com/villekf/OMEGA/blob/master/source/load_data.m>`_,
`load_data_mCT.m <https://github.com/villekf/OMEGA/blob/master/source/load_data_mCT.m>`_
and
`load_data_Vision.m <https://github.com/villekf/OMEGA/blob/master/source/load_data_Vision.m>`_.

.. _execution-1:

Execution
~~~~~~~~~

Data load is divided into three different sections: GATE data, list-mode
data and sinogram data.

GATE data
^^^^^^^^^

For `GATE <http://www.opengatecollaboration.org/>`_ data, the data import is
separate for LMF (not supported anymore), ASCII and ROOT. When using Python, only ROOT is supported at the moment.

In LMF, the data import is done in a C++ file
(`gate_lmf_matlab.cpp <https://github.com/villekf/OMEGA/blob/master/source/gate_lmf_matlab.cpp>`_)
and each binary packet is read sequentially. In LMF, unlike other
methods, the coincidences are formed manually from the singles. This is
done by first checking if the time difference between the consecutive
singles is within the coincidence window length. If yes, then the
singles are assumed to be coincidences and all the specified values
(e.g. detector number, whether the coincidence is true, random or
scattered event, source coordinates) are saved. If not, then the first
single is discarded and the latter single becomes the first single and
compared with the next single, etc. For time steps, there is first a
check on whether the current time is larger than the time of the next
time step. If yes, then the current event index is stored. This index
indicates where the time was large enough for the next time step to
begin and is then used to separate the measurement data into the time
bins. Of all the three data types, LMF is the most limited one as it
does not support delayed coincidences or scattered events except for
Compton scatter in phantom. Sinograms are created during the data load
when using LMF data.

ASCII data import is different from both LMF and ROOT in sense that it
is done purely in MATLAB/Octave. It uses either ``readmatrix`` (if
available) or ``importdata`` to import the ASCII text into a matrix
containing all the rows and columns. If any of the columns on any of the
rows is corrupted/missing (replaced by NaN value), the code will detect
these, discard them and inform the line number where this occurred. All
the chosen variables are then stored in individual vectors/matrices.
Time steps are handled similarly to LMF. Sinograms are created during
the data load when using ASCII data.

ROOT data import is handled in a C++ MEX-file or through C++ library file in Python. Unlike LMF, ROOT has
three different MEX-versions. One uses the “traditional” C MEX-interface
(`GATE_root_matlab_C.cpp <https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab_C.cpp>`_)
and is intended for MATLAB version 2018b and earlier, the second uses
the new C++ MEX-interface
(`GATE_root_matlab.cpp <https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab.cpp>`_)
and is for MATLAB 2019a and newer and lastly there is a dedicated
version for Octave as well
(`GATE_root_matlab_oct.cpp <https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab_oct.cpp>`_).
All the ROOT import functions first open the trees (coincidences and
delays if selected) and then the desired branches. There are also checks
in place that first guarantee that a specific branch is available,
e.g. the scatter data. If not a message is displayed, but the data load
will still continue, unless it is the detector information that is
missing as it is vital for the data load. Sinograms are created during
the data load in all three cases. Currently, all the mex-files and the Python library file have the exact same functionality. The only difference is the "input" file, that handles all the input variables and data, as well as the 
outputs. If the index-based data load is used, the sinogram is not saved at the same time. TOF data load is handled automatically, if enabled.

List-mode data
^^^^^^^^^^^^^^

List-mode data, more specifically Siemens Inveon, Biograph mCT and
Biograph Vision list-mode data, is loaded in a separate MEX-file or library file in Python. For
Inveon, the source code is available
(`inveon_list2matlab.cpp <https://github.com/villekf/OMEGA/blob/master/source/inveon_list2matlab.cpp>`_),
but for mCT and Vision only the MEX-files themselves are distributed
(i.e. a closed source release) and at the moment there are no files for Python. For Inveon, the code loops through all
the bit-packets, determines whether they are prompt, delay or time tags
and then extracts the corresponding information. Sinograms are created during the data load, but not when any listmode reconstruction mode is used.

List-mode data is saved with ``_listmode`` in the end of the filename.
For 32-bit list-mode data (mCT only) ``_listmode_sinogram`` is added to
the end of the filename.

TOF data can be loaded from mCT and Vision, but currently ONLY with the
default number of bins (and bin width).

Sinogram data
^^^^^^^^^^^^^

This applies only to Biograph mCT and Vision. For Inveon, the sinogram
data load occurs in ``form_sinograms.m``. Uncompressed mCT and Vision
sinograms (.s or .ptd files) can be loaded. Corrections can be applied
normally. The data itself is saved with ``_machine_sinogram`` in the end
of the filename.

Saving data
^^^^^^^^^^^

GATE data and list-mode data go through the same procedures when saving
data. All steps are repeated for the selected number of time steps,
where first the sinogram is created. For GATE data, trues,
randoms and scatter are stored as well if selected. TOF data will have
different filenames from non-TOF data.

Forming sinograms
-----------------

The corresponding m-file is
`form_sinograms.m <https://github.com/villekf/OMEGA/blob/master/source/form_sinograms.m>`_.
Note that currently the sinograms are created already during data load, and the ``form_sinograms`` is simply used to apply potential precorrections and to save the data. Sinograms can be created separately as well with
the following files: 
`createSinogramASCII.cpp <https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCII.cpp>`_
is for the old C-API,
`createSinogramASCIICPP.cpp <https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCIICPP.cpp>`_
is for the C++-API and
`createSinogramASCIIOct.cpp <https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCIIOct.cpp>`_
is for Octave. Python also has a library version available. These files are, however, only used when loading ASCII GATE data in MATLAB/Octave.

.. _execution-2:

Execution
~~~~~~~~~

Sinograms can be formed during data load. Alternatively precorrections may be later applied to existing sinogram (e.g. no actual new sinogram is
created). When sinograms are formed, a raw uncorrected sinogram is
always created and saved regardless of the corrections applied. This is
saved as ``raw_SinM``.

*form_sinograms.m:*

When creating sinogram from raw ASCII data the first step is the formation of
an “initial Michelogram”. This is an intermediate step between the raw
data format and the Michelogram/sinogram format. The raw data is divided
into vectors that contain the future Michelogram bins. This is performed
in
`initial_michelogram.m <https://github.com/villekf/OMEGA/blob/master/source/initial_michelogram.m>`_.

Next step is the formation of the Michelograms by selecting the data
points that are within the predetermined orthogonal distance from the
center of the field-of-view. These are saved as unsigned 16-bit integers
and performed for all the selected data types (trues, prompts, delays,
etc.).

After this, the next step performs the axial compression, though using
span of 1 (no axial compression) is also possible.

*MEX/OCT:*

When the sinograms are created with the MEX/OCT-file, a separate
function computes the sinogram indices based on each ring number (axial
position) and ring position (transaxial position).

*Corrections:*

The last step, corrections, is applied if precorrections are selected. However, most corrections are not
applied if ``options.corrections_during_reconstruction = false``, with
the exception of sinogram gap filling. Corrections are handled in the
following order: Randoms (variance reduction, then smoothing) -> Scatter
without normalization (variance reduction, then smoothing) ->
normalization correction -> Scatter when using normalized scatter
(variance reduction, then smoothing) -> global correction factor ->
Sinogram gap filling. If any of the corrections are set as ``false``,
then that step is omitted. Only prompts go through corrections. Scatter
can be applied with normalization separately applied to it or
without separate normalization.

All the separate sinograms are saved in a same mat-file with the
sinogram dimensions in the name. Included are also structs that contain
whether certain corrections were applied (``appliedCorrections``) and
what corrections were applied to scatter or randoms (``ScatterProp``,
``RandomsProp``). In ``appliedCorrections`` normalization is stored as a
boolean variable (``false`` means no normalization), randoms and scatter
as char (empty array means no corrections, otherwise they can be
e.g. “randoms correction with smoothing”), gap filling as boolean,
mashing factor as an integer and lastly the user specified global
correction factor. The prop-structs contain booleans indicating whether
variance reduction and/or smoothing was applied.

Randoms correction is applied as randoms subtraction from the delayed
coincidences data. Scatter correction can be applied either as a
subtraction, or
alternatively by multiplication by setting ``options.subtract_scatter = false``. In the latter case the scatter data is
multiplied with the sinogram. Same steps are repeated for all time
steps.

When the function is used to modify the applied corrections
(e.g. ``form_sinograms(options, true)``), the sinogram creation step is
skipped and the uncorrected sinogram is loaded. By default,
``form_sinograms`` assumes that the sinogram needs to be created,
i.e. the boolean value after ``options`` needs to be true in order to
perform only corrections. Any sinogram, no matter where created, can be
corrected like this. However, the data needs to saved as ``raw_SinM`` in
a mat-file with the same name as the current scanner properties
(e.g. for non-TOF case
``[options.machine_name '_' options.name '_sinograms_combined_static_' num2str(options.Ndist) 'x' num2str(options.Nang) 'x' num2str(options.TotSinos) '_span' num2str(options.span) '.mat']``
for static data and
``[options.machine_name '_' options.name '_sinograms_combined_' num2str(options.partitions) 'timepoints_for_total_of_ ' num2str(options.tot_time) 's_' num2str(options.Ndist) 'x' num2str(options.Nang) 'x' num2str(options.TotSinos) '_span' num2str(options.span) '.mat']``
for dynamic).

*Saving:*

In the bottom of the m-file, there is a separate section for loading
Inveon Acquisition Workplace created sinograms. These sinograms
automatically have randoms corrections applied. All other corrections
can be applied just as with raw data. Dynamic data is also supported,
but the number of time steps have to be equal to the original data.

The output of ``form_sinograms`` can consist of the uncorrected
sinogram, corrected sinogram, corrected delayed sinogram, uncorrected
delayed sinogram as well as sinograms of trues, scatter and randoms. The
first input is either the corrected sinogram (if corrections were
applied) or the uncorrected sinogram (no corrections).

Attenuation correction
----------------------

*Inveon*

For Inveon two different attenuation correction types are available. The
first is based on the blank and transmission scans while the other is
CT-based. Both are controlled by
`attenuation_correction_factors.m <https://github.com/villekf/OMEGA/blob/master/source/attenuation_correction_factors.m>`_.
For the blank and transmission case the .atn-file provided by the Inveon
Acquisition workplace is needed. This is reconstructed into an
attenuation image by the aforementioned function. All the reconstruction
parameters have been pre-set. Implementation 4 with PSF is always used
for the reconstruction. In the CT-case the umap-file contains ready-made
attenuation images that are simply loaded and rotated. It is assumed
that the bed is always at the lower part of the image. For the .atn-case
the attenuation values are also scaled with
`attenuation122_to_511.m <https://github.com/villekf/OMEGA/blob/master/source/attenuation122_to_511.m>`_.

The scaling scales the 122 keV attenuation coefficients (blank and
transmission scan) to 511 keV. First the tabulated values for various
tissues and elements for both 122 and 511 keV cases are computed. The
input values are then scaled such that the peak is at the soft tissue
level (ignore air). Air is given small values. The values are
interpolated to densities and then interpolated again by using these
densities to 511 keV attenuation coefficients.

*mCT and Vision*

mCT and Vision attenuation correction uses CT-based attenuation
correction. The attenuation images for PET are computed with
`create_atten_matrix_CT.m <https://github.com/villekf/OMEGA/blob/master/source/create_atten_matrix_CT.m>`_
and
`attenuationCT_to_511.m <https://github.com/villekf/OMEGA/blob/master/source/attenuationCT_to_511.m>`_.
The CT images are first scaled to 511 keV by using trilinear
interpolation.

*Other data*

In general the attenuation correction requires an attenuation image that should be scaled to the proper energy and use units 1/mm. SPECT reconstructions need to be scaled for the SPECT energy, while PET ones
for PET energy. The files specified in the above mCT and Vision section can be used for other CT-data, but only work for PET cases.

The attenuation correction itself is performed slightly differently in PET and SPECT. In PET, the attenuation coefficients are multiplied with the length of intersection of the ray in the voxel. These are then summed for all
voxels and then the exponent is taken of the negative value. This is then multiplied with the probability. For backprojection, the attenuation is precompute. The use of attenuation correction can slow down the computations 
as the attenuation coefficients are required at every (sub-)iteration.

SPECT attenuation correction is similar, but a single summed variable is not used. Instead, only values summed up to that voxel are used. Thus, voxels further from the detector have greater attenuation correction, while in PET 
they are the same.

Normalization correction
------------------------

Normalization coefficients are computed by
`normalization_coefficients.m <https://github.com/villekf/OMEGA/blob/master/source/normalization_coefficients.m>`_.

Image reconstruction
--------------------

The image reconstruction phase has been divided into five separate types
that are referred as implementations. Note that Python only uses implementation 2. By default
implementation 1 (CPU) is double precision (64-bit) and 2, 3, 5 (OpenCL), and 4 (CPU) are in single
precision (32-bit). Implementation 4 can also use double precision with ``options.useSingles = false``. While MATLAB R2025a supports single precision sparse matrices, the support for those have not been validated 
for implementation 1.

All four implementations are explained here separately in the following
sections. The matrix-free formulation is explained in more detail after
the implementations have been presented.

Implementation 1
~~~~~~~~~~~~~~~~

Implementation 1 solves the image reconstruction problem in matrix form
and as such the system matrix is created as whole for each subset or, in
case of MLEM, the entire matrix in one go. Due to this the memory
requirements are high despite the system matrix being stored in sparse
format; size of the full system matrix can exceed even hundreds of
gigabytes. This is partially caused by MATLAB/Octave always storing
sparse matrices in double precision format with 64-bit integer indices
in 64-bit systems although single precision and 32-bit integers would be
enough. Implemenation 1 is only available for PET and CT, it is not currently available for SPECT.

Previously implementation 1 supported a non-parallel version, but this was removed in v2.0.

The current version performs a "precomputation" step before the actual system matrix is computed. The precomputation phase is needed in order to allocate correct
amount of memory for the sparse matrix. In this case, the sparse matrix
is directly created and ﬁlled in the C++ MEX-ﬁle. MATLAB sparse matrices
are in compressed sparse column (CSC) format, but PET data is handled
row by row (i.e. each measurement) basis, making it more suitable for
compressed sparse row (CSR) format. However, this can be solved by
simply considering the sparse system matrix to be transposed, as a
transposed CSC matrix is a CSR matrix. As such, the output is actually
the transposed system matrix. The
precomputed phase was developed after the case without precomputation,
initially without OpenMP support. The reconstruction
itself is handled completely in MATLAB/Octave. Due to this, the
reconstruction process can be relatively slow as sparse matrix
multiplications are not parallel in MATLAB (on CPU) in R2020b or earlier
(2021a and later should have parallel CPU sparse support). However, the
reconstructions in MATLAB/Octave also allow for all reconstruction
algorithms and priors to be supported. It is also possible to compute
simply the system matrix (or a subset of it) instead of the
reconstructions, allowing the user to use the system matrix in their own
algorithms. All computations done with implementation 1 are performed in
double precision. TOF data is not supported by implementation 1.

Implementation 2
~~~~~~~~~~~~~~~~

Implementation 2 is the recommended method for image reconstruction. It
utilizes OpenCL or CUDA and the open-source
`ArrayFire <https://arrayfire.com/download/>`_ library. Unlike
implementation 1, in this case the system matrix is never explicitly
computed, but rather the computations of the forward and backward
projections are done entirely matrix free. In implementation 2, both the forward and backward
projections are computed in an OpenCL kernel that also computes the
system matrix elements using the selected projector. In v1.2 and below, the algorithms themselves were largely computed in the kernel as well, but currently, only the forward and/or backward projection operations
are computed in the kernels. The forward projection thus outputs the forward projection vector, while backprojection the backprojection vector and optionally also the sensitivity image. 

All operations occur on the selected device and only the ﬁnal result from
each iteration is transferred to the host (if 
``options.save_iter = true``, otherwise only the last iteration).
Implementation 2 supports all algorithms and priors. Implementation 2
was developed after implementation 1 had been completed. Furthermore, a
CUDA formulation of implementation 2 exists since v1.1 and has the same
features as the OpenCL variant, but is considered only as an extra
feature at the moment. All operations are computed in single precision.

Implementation 3
~~~~~~~~~~~~~~~~

Implementation 3 is similar to 2 in that it utilizes OpenCL and has the
same matrix-free formalism. However, outside of the OpenCL kernel code
the two are very different. In implementation 3, the computations are
performed in “pure” OpenCL, i.e. there are no third-party (ArrayFire)
functions at work and everything is computed in custom-made OpenCL
kernels.

Since implementation 3 uses "pure" OpenCL, only OSEM and MLEM are currently supported. Furthermore, implementation 3 is no longer extensively tested and will be deprecated in the future. Currently implementation 3
also supports largely only PET data though CT should work too. SPECT is not supported.

Implementation 3 was developed after implementation 2 as a separate
project to enable multi-device support and additionally to provide
OpenCL reconstruction without the need for third-party libraries. Since v2.0, the support for multiple devices (GPUs) has been dropped.

Implementation 4
~~~~~~~~~~~~~~~~

Implementation 4 is a combination of implementations 1 and 3, meaning
that it is a pure CPU method that uses OpenMP for the parallellization,
as in implementation 1, but is implemented in matrix-free way as the
OpenCL methods. The matrix-free formulation itself does not essentially
differ from the OpenCL, except using C++ OpenMP code.

The functionality is the same as in the OpenCL methods, i.e. the forward and/or backward projection operations are computed in C++ using OpenMP. These are then used in MATLAB-based reconstructions. Due
to this, implementation 4 supports the same algorithms as implementation 1. 

Implementation 4 was developed after the other implementations
(excluding CUDA in implementation 2) as a fallback method for
matrix-free computation without the need for OpenCL. It was also
developed for CPUs that lack OpenCL support and to provide numerically
more accurate matrix-free formulation. However, as of v2.0, the default precision of implementation 4 is now single. Double precision can still be enabled with ``options.useSingles = false``.

Matrix-free formulation
~~~~~~~~~~~~~~~~~~~~~~~

The matrix-free forward and backprojection are implemented the same
regardless of the used projector or reconstruction algorithm. For PET and SPECT, the probability that a photon emitted from voxel *j* is detected along line of response (LOR) *i* is computed. For CT, and CT-like data,
the actual intersection length is computed. This means that in PET/SPECT, the intersection length is divided by the length of the ray. The length can be either the length of the ray in total (from detector to detector) or 
the length of the ray in the FOV only (set ``options.totLength = false`` for the latter). 

In all cases, the forward projection always computes each measurement in parallel. This is the case for ALL projectors. 
That is, all forward projectors are essentially ray-based.
For backprojection, the process is different depending on whether PET/SPECT or CT data used. For PET/SPECT, the backprojection uses the same ray-based approach as forward projection. CT-data, on the other hand, use
voxel-based approaches. In both cases, the sensitivity image (diagonal image-based preconditioner) can be computed at the same time. This applies to specific algorithms, such as OSEM, and is handled automatically.
The preconditioner itself is always computed separately. Sensitivity image, however, is only computed during the very
ﬁrst iterations unless not enough memory is available for storage in
which case it will be computed on-the-ﬂy. With PET/SPECT data, both the sensitivity image and
the backprojection are saved in a thread-safe way by using atomic
operations, more speciﬁcally the atomic addition. Atomic operations
guarantee that the read-write operation to the memory location is only
available to the current thread until the operation is completed,
essentially making the operation sequential. Atomic addition in this
case thus sums the input to the currently residing value in the current
voxel index. With sensitivity image, the LOR probability is thus
atomically added to the current sensitivity image vector at each voxel
the LOR goes through. For backprojection, the process is otherwise
identical, but instead of probability only, the LOR probability is
multiplied with *Θ\ i* before atomically added to the current *Δ*.

If the sensitivity image is saved, the subsequent iterations will be
much faster as any LORs with zero counts will be completely ignored (the
additions would be zero). Implementation 4 uses OpenMP atomic operations
for 32-bit ﬂoats to compute the additions. For implementations 2 and 3
there are two diﬀerent atomic version available. As there is no inherent
support for atomic addition for 32-bit ﬂoats in OpenCL, a similar method
as in
https://streamhpc.com/blog/2016-02-09/atomic-operations-for-floats-in-opencl-improved/[GROMACS]
has been implemented. However, since this is relatively slow another
approximate version is also provided that uses 64-bit signed integers
instead of ﬂoating point numbers. In this case, the ﬂoating point values
are converted to 64-bit signed integers, which causes some loss of
precision due to rounding, before atomically added. This provides some
speed-up compared to the 32-bit ﬂoat version, but cannot be used on some
hardware. If the user has selected this option, the support is
determined during compile time and the ﬂoat version is used if the
hardware does not support 64-bit atomics. The output sensitivity image
and *Δ* are then converted back to 32-bit ﬂoats before they are used in
the reconstruction algorithms.

TOF coefficients
~~~~~~~~~~~~~~~~

TOF coefficients are computed exactly the same for all implementations.
Though for implementation 4 the intermediate results are saved
regardless of user selection. TOF coefficients are computed only if TOF
data is selected. For implementations 2 and 3 they are included in the
kernel compilation only if TOF data has been selected. With
implementation 4 they are simply behind regular conditional expressions.

For TOF data the variance of the data and the bin center locations are
precomputed. The variance is determined from the
https://en.wikipedia.org/wiki/Full_width_at_half_maximum[FWHM]. Bin
centers are determined from the input bin width, bin number and bin
offset.

In the kernel itself, the first step is to compute the distance from the
FOV (voxel space) to the “source” (first detector/crystal). This is
achieved by using the parametrization of a line since the required
parameter (often *t*) is given by the Siddon’s algorithm. The half of
the total length of the ray is then subtracted from this value. The
intersection length is added to this value after each voxel. This length
is the distance from the current voxel boundary to the center of the
ray.

TOF coefficients are computed at each voxel for all TOF time bins.
Meaning that every time a voxel is intersected, the TOF coefficients are
looped through all the TOF bins. The only difference in the computations
of the TOF coefficients are the different values for the TOF bin center
locations. At the same time each of these TOF coefficients for the
corresponding voxel and summed together. Each TOF coefficient is then
later divided by this total sum. TOF coefficients themselves are
computed as a 1D integral from the current ray location to the next
(e.g. the intersection length is either added or subtracted from the
current distance from the center of the ray). The integral itself is
computed by using the
https://en.wikipedia.org/wiki/Trapezoidal_rule[trapezoidal rule]. By
default, four (4) trapezoidal integration points are used. Each original
probability is then multiplied with the TOF coefficients. *Δ* is
computed for each TOF bin and then summed together before the atomic
addition. Same goes for sensitivity image, although that could be
computed without any TOF information as well.

Due to the use of the trapezoidal rule, TOF bins with very high accuracy
may not be reliable unless the number of integration points is
increased. However, the default value should be fine even in 20-30 ps
range. For implementations 2 and 3, the number of integration points can
be adjusted by modifying ``general_opencl_functions.h`` and specifically
the value ``TRAPZ_BINS``. No recompilation is required. For
implementation 4, modify ``projector_functions.h`` and the same
``TRAPZ_BINS`` value. Recompilation IS required for implementation 4.
