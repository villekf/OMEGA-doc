Technical info
===============

This page outlines some technical information on the software. Mainly how everything works.

The general structure of OMEGA can be divided into three different layers. The top layer is the MATLAB/Octave/Python user-interface that contains the scripts and functions necessary to call the lower layers. 
The middle layer is the MATLAB (C) MEX-interface or C++ dynamic library that calls and computes the C++ code and then sends it back to the top layer. The bottom layer, which is not always used, contains the OpenCL/CUDA kernels that 
compute the OpenCL/CUDA code and then send the output data (reconstructed images) to the middle layer. The bottom layer is only used if OpenCL/CUDA code is used. The middle layer can also be ignored, but this is recommended only when
doing custom reconstructions in Python.

For the user, only the top layer is exposed. This is achieved in the form of scripts that are referred to as *main-files*. These include, for example, https://github.com/villekf/OMEGA/blob/master/main-files/PET_main_gateExample.m[``gate_main.m``],
https://github.com/villekf/OMEGA/blob/master/main-files/CT_main_full.m[``CT_main_full.m``], or https://github.com/villekf/OMEGA/blob/master/source/Python/gate_PET.py[``gate_PET.py``].
It is from these main-files that the actual functions are called. This
is achieved by storing all the user selected parameters to a
MATLAB/Octave struct called ``options``, which is then input to the
functions (e.g. when loading data or performing image reconstruction).

In the main-files most parameters are set either with numerical values
(e.g. ``blocks_per_ring``, ``span``, ``tube_radius``) or as a true/false
selection, where true means that the feature is included and false that
it is omitted (e.g. ``randoms_correction``, ``scatter_smoothing``,
``osem``). The main-files are divided into “sections” with specific
labels (SCANNER PROPERTIES, IMAGE PROPERTIES, SINOGRAM PROPERTIES, etc.)
that control different aspects. Many of these sections are completely
optional, e.g. CORRECTIONS section can be omitted if the user does not
wish to use any corrections. The only compulsory ones are SCANNER
PROPERTIES and either SINOGRAM or RAW DATA PROPERTIES. For
reconstruction it is also advisable to inspect the RECONSTRUCTION
PROPERTIES section, but the default values should always output a
working OSEM estimate. By default all non-compulsory options are set as
false (with the exception of the Inveon main-file which has several
corrections enabled by default).

There are also several functions that work very independently without
the need for the main-files. These include file import and export
functions, visualization functions and many reconstruction algorithm
functions. For help on many of these functions, you should use
``help function_name`` or alternatively ``doc function_name``. E.g.
``help saveImage``.

Technical aspects
=================

This section explains how, exactly, does each component of OMEGA
software work.

Precomputation phase
--------------------

The corresponding m-file is
https://github.com/villekf/OMEGA/blob/master/source/lor_pixel_count_prepass.m[``lor_pixel_count_prepass.m``].

Purpose
~~~~~~~

As mentioned in other help pages, the goal of the precomputation phase
is to determine the number of voxels that each line of response
traverses (interacts with). Line of responses that do not intersect the
FOV at all will have 0 interacted voxels and as such can be safely
removed. This step is also necessary for the parallel version of
implementation 1 as it requires the preallocation of the system matrix
in memory. This also means that, unlike with matrix free
implementations, the orthogonal and volume of intersection ray tracers
require their own precomputation passes since the number of voxels that
are interacted with is much higher.

Execution
~~~~~~~~~

Dedicated codes are available for the precomputation phase. Furthermore,
since the OpenCL implementations use single (32-bit float) precision
numbers and the C++ versions double (64-bit double) precision numbers,
different versions are available for both. This is because of
floating-point rounding effects that can cause LORs that are on the very
edge of a voxel to be in different voxels depending on whether the
single or double precision implementations are used. While this usually
affects only a very small portion of all the LORs, even a single wrong
one can cause a crash due to out-of-bounds variables. Implementation 2
also has its own precomputation code, although the OpenCL kernel itself
is identical to implementation 3. This is simply because of the
different device/platform executions.

The codes themselves are stripped versions of the actual projectors. In
all cases, the ray tracing itself is always performed, no parameters
outside of the ones necessary for the ray tracing are computed. For C++
version, there is a different file/function for this
(https://github.com/villekf/OMEGA/blob/master/source/improved_Siddon_algorithm_discard.cpp[``improved_Siddon_algorithm_discard.cpp``]),
but for OpenCL versions all the code are in “master” kernel file
(https://github.com/villekf/OMEGA/blob/master/source/multidevice_kernel.cl[``multidevice_kernel.cl``])
where the correct lines are compiled during runtime by taking advantage
of preprocessor directives (``#ifdef``, ``#else``, etc.). For
precomputation phase, the preprocessor directive that is defined is
``FIND_LORS``. No precomputation code is available for CUDA.

Data load
---------

The corresponding m-files are
https://github.com/villekf/OMEGA/blob/master/source/load_data.m[``load_data.m``],
https://github.com/villekf/OMEGA/blob/master/source/load_data_mCT.m[``load_data_mCT.m``]
and
https://github.com/villekf/OMEGA/blob/master/source/load_data_Vision.m[``load_data_Vision.m``].

.. _execution-1:

Execution
~~~~~~~~~

Data load is divided into three different sections: GATE data, list-mode
data and sinogram data.

GATE data
^^^^^^^^^

For http://www.opengatecollaboration.org/[GATE] data, the data import is
separate for LMF, ASCII and ROOT.

In LMF, the data import is done in a C++ file
(https://github.com/villekf/OMEGA/blob/master/source/gate_lmf_matlab.cpp[``gate_lmf_matlab.cpp``])
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

ROOT data import is handled in a C++ MEX-file. Unlike LMF, ROOT has
three different MEX-versions. One uses the “traditional” C MEX-interface
(https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab_C.cpp[``GATE_root_matlab_C.cpp``])
and is intended for MATLAB version 2018b and earlier, the second uses
the new C++ MEX-interface
(https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab.cpp[``GATE_root_matlab.cpp``])
and is for MATLAB 2019a and newer and lastly there is a dedicated
version for Octave as well
(https://github.com/villekf/OMEGA/blob/master/source/GATE_root_matlab_oct.cpp[``GATE_root_matlab_oct.cpp``]).
All the ROOT import functions first open the trees (coincidences and
delays if selected) and then the desired branches. There are also checks
in place that first guarantee that a specific branch is available,
e.g. the scatter data. If not a message is displayed, but the data load
will still continue, unless it is the detector information that is
missing as it is vital for the data load. Sinograms are created during
the data load in all three cases. However, for the C++ version the
sinogram creation does not occur when the ROOT data itself is loaded,
but only after the file has been loaded.

List-mode data
^^^^^^^^^^^^^^

List-mode data, more specifically Siemens Inveon, Biograph mCT and
Biograph Vision list-mode data, is loaded in a separate MEX-file. For
Inveon, the source code is available
(https://github.com/villekf/OMEGA/blob/master/source/inveon_list2matlab.cpp[``inveon_list2matlab.cpp``]),
but for mCT and Vision only the MEX-files themselves are distributed
(i.e. a closed source release). For Inveon, the code loops through all
the bit-packets, determines whether they are prompt, delay or time tags
and then extracts the corresponding information. Static and dynamic
cases are handled a bit differently; in the former case the counts are
stored in one detectors x detectors sized matrix, while in the latter
they are stored event-by-event basis. In dynamic case, however, the
events are stored in a same type of (sparse) matrix as in static case
for each time step with the use of ``accumarray`` function. Time steps
are handled as with LMF data, where the index is stored where the time
exceeded the previous time step.

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
where first the sinogram is created (if raw data is not explicitly used)
and then the raw data is stored (if selected). For GATE data, trues,
randoms and scatter are stored as well if selected. TOF data will have
different filenames from non-TOF data, though raw data does not
currently support TOF data.

Forming sinograms
-----------------

The corresponding m-file is
https://github.com/villekf/OMEGA/blob/master/source/form_sinograms.m[``form_sinograms.m``].
Currently, when data is loaded from GATE or list-mode data the sinograms
are created through separate MEX-file or OCT-file.
https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCII.cpp[``createSinogramASCII.cpp``]
is for the old C-API,
https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCIICPP.cpp[``createSinogramASCIICPP.cpp``]
is for the C++-API and
https://github.com/villekf/OMEGA/blob/master/source/createSinogramASCIIOct.cpp[``createSinogramASCIIOct.cpp``]
is for Octave.

.. _execution-2:

Execution
~~~~~~~~~

Sinograms can be formed from saved raw data, during data load (no need
to load the raw data separately) and also by simply modifying the
corrections applied to the sinogram (e.g. no actual new sinogram is
created). When sinograms are formed, a raw uncorrected sinogram is
always created and saved regardless of the corrections applied. This is
saved as ``raw_SinM``.

As mentioned above, the sinograms can be either created from the raw
data afterwards or during the data load itself. The latter method is
faster and more memory efficient. However, it can be useful to create a
sinogram of different size later from the same data. In this case, if
the data load takes a long time, it is probably beneficial to create a
new sinogram from the raw data. This, however, only works if raw data
was initially saved (``options.store_raw_data = true``).

*form_sinograms.m:*

When creating sinogram from raw data the first step is the formation of
an “initial Michelogram”. This is an intermediate step between the raw
data format and the Michelogram/sinogram format. The raw data is divided
into vectors that contain the future Michelogram bins. This is performed
in
https://github.com/villekf/OMEGA/blob/master/source/initial_michelogram.m[``initial_michelogram.m``].

Next step is the formation of the Michelograms by selecting the data
points that are within the predetermined orthogonal distance from the
center of the field-of-view. These are saved as unsigned 16-bit integers
and performed for all the selected data types (trues, prompts, delays,
etc.).

After this, the next step performs the axial compression, though using
span of 1 (no axial compression) is also possible. However, span of 1 is
only supported with prompts.

*MEX/OCT:*

When the sinograms are created with the MEX/OCT-file, a separate
function computes the sinogram indices based on each ring number (axial
position) and ring position (transaxial position).

*Corrections:*

The last step, corrections, is applied whether the sinogram was created
from raw data or during data load. However, most corrections are not
applied if ``options.corrections_during_reconstruction = false``, with
the exception of sinogram gap filling. Corrections are handled in the
following order: Randoms (variance reduction, then smoothing) -> Scatter
without normalization (variance reduction, then smoothing) ->
normalization correction -> Scatter when using normalized scatter
(variance reduction, then smoothing) -> global correction factor ->
Sinogram gap filling. If any of the corrections are set as ``false``,
then that step is omitted. Only prompts go through corrections. Scatter
can be applied only with normalization separately applied to it or
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
subtraction by setting ``options.subtract_scatter = true``, or
alternatively by multiplication. In the latter case the scatter data is
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
``[options.machine$$_$$name '$$_$$' options.name '$$_$$sinograms_combined_static$$_$$' num2str(options.Ndist) 'x' num2str(options.Nang) 'x' num2str(options.TotSinos) '$$_$$span' num2str(options.span) '.mat']``
for static data and
``[options.machine$$_$$name '$$_$$' options.name '$$_$$sinograms$$_$$combined$$_$$' num2str(options.partitions) 'timepoints$$_$$for$$_$$total$$_$$of$$_$$ ' num2str(options.tot$$_$$time) 's$$_$$' num2str(options.Ndist) 'x' num2str(options.Nang) 'x' num2str(options.TotSinos) '$$_$$span' num2str(options.span) '.mat']``
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

This section applies only to Inveon, mCT and Vision.

*Inveon*

For Inveon two different attenuation correction types are available. The
first is based on the blank and transmission scans while the other is
CT-based. Both are controlled by
https://github.com/villekf/OMEGA/blob/master/source/attenuation_correction_factors.m[attenuation_correction_factors.m].
For the blank and transmission case the .atn-file provided by the Inveon
Acquisition workplace is needed. This is reconstructed into an
attenuation image by the aforementioned function. All the reconstruction
parameters have been pre-set. Implementation 4 with PSF is always used
for the reconstruction. In the CT-case the umap-file contains ready-made
attenuation images that are simply loaded and rotated. It is assumed
that the bed is always at the lower part of the image. For the .atn-case
the attenuation values are also scaled with
https://github.com/villekf/OMEGA/blob/master/source/attenuation122_to_511.m[attenuation122_to_511.m].

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
https://github.com/villekf/OMEGA/blob/master/source/create_atten_matrix_CT.m[create_atten_matrix_CT.m]
and
https://github.com/villekf/OMEGA/blob/master/source/attenuationCT_to_511.m[attenuationCT_to_511.m].
The CT images are first scaled to 511 keV by using trilinear
interpolation.

Normalization correction
------------------------

Normalization coefficients are computed by
https://github.com/villekf/OMEGA/blob/master/source/normalization_coefficients.m[normalization_coefficients.m].

Image reconstruction
--------------------

The image reconstruction phase has been divided into four separate types
that are referred as implementations. Along with these four
implementations, each implementation has two different modes of working,
one with a precomputation phase and one without. When the precomputation
option is selected, a separate phase needs to be completed before the
image reconstruction which determines the valid LORs, i.e. LORs that
intersect the FOV (see above). This phase determines the indices of
those LORs that intersect the FOV and also determines the number of
voxels each of these valid LORs traverse (required for implementation
1). While sinogram data may not have any non-valid LORs, raw data often
has significant amount of them. As such, the precomputation phase should
increase the speed of the reconstruction phase as non-valid LORs are
never investigated. This should make even cases with no non-valid LORs
slightly faster due to lack of LOR validation, but the effect is greater
with raw data. However, due to ﬂoating point rounding eﬀects the
precomputation phase needs to be different when computing either
implementation 1 or 4 (CPU) or 2 or 3 (OpenCL) as the ﬁrst two are
computed in double precision (64-bit) while the last two are in single
precision (32-bit).

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
enough. Using the orthogonal (ODRT) or volume-based (VRT) ray tracers
even more emphasizes this as the system matrix grows even larger, making
even subset-based reconstruction very memory intensive.

As previously mentioned, two different versions of each implementation
is available. For this case the one without a precomputation phase is
the only non-parallel version due to the need to dynamically allocate
memory. The C++ code saves the row, column and non-zero indices for the
sparse matrix which is constructed in MATLAB. This version also includes
a pure MATLAB version (i.e. no C++ code) that can be optionally used,
but both of these versions are very slow. ODRT or VRT are not supported
as using them would be infeasible. The development of OMEGA has been an
iterative process with this non-parallel case being the very ﬁrst to be
developed. While this case is no longer recommended to be used, it is
included for feature parity.

The other case, with precomputation phase, is computed in parallel with
OpenMP. The precomputation phase is needed in order to allocate correct
amount of memory for the sparse matrix. In this case, the sparse matrix
is directly created and ﬁlled in the C++ MEX-ﬁle. MATLAB sparse matrices
are in compressed sparse column (CSC) format, but PET data is handled
row by row (i.e. each measurement) basis, making it more suitable for
compressed sparse row (CSR) format. However, this can be solved by
simply considering the sparse system matrix to be transposed, as a
transposed CSC matrix is a CSR matrix. As such, the output is actually
the transposed system matrix. This case also supports ODRT and VRT. The
precomputed phase was developed after the case without precomputation,
initially without OpenMP support. In both cases, the reconstruction
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
utilizes OpenCL and the open-source
https://arrayfire.com/download/[ArrayFire] library. Unlike
implementation 1, in this case the system matrix is never explicitly
computed, but rather the computations of the forward and backward
projections are done entirely matrix free. Both precomputed and
non-precomputed cases are available, but this time the differences
between these are smaller as there is no need to preallocate memory
based on a priori data. However, the precomputed version should still be
faster as before. In implementation 2, both the forward and backward
projections are computed in an OpenCL kernel that also computes the
system matrix elements using the selected projector (both SRT and ODRT
are supported). This kernel outputs two vectors, one containing the
sensitivity image and the other

*Δ* = (*A\ T* *p*) / (*Af* + *r* + *s*),

where *A* is the system matrix, *p* the measurements, *f* the current
estimate, *r* randoms and *s* scatter.

The vector Δ contains the necessary elements for all selected algorithms
and as such has a size of N × N\ :sub:`algorithms`, where N is the total
number of voxels and N\ :sub:`algorithms` the number of selected
algorithms. Both of these vectors are then used to compute the ﬁnal
estimates that are calculated by using ArrayFire functions. All
operations occur on the selected device and only the ﬁnal result from
each iteration is transferred to the host (if
``options.save_iter = true``, otherwise only the last iteration).
Implementation 2 supports all algorithms and priors. Implementation 2
was developed after implementation 1 had been completed. Furthermore, a
CUDA formulation of implementation 2 exists in v1.1 and has the same
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

The forward and backward projections work like in implementation 2, but
this time only for either OSEM or MLEM. This is due to that it allows
the use of multiple devices at once, which is also the biggest
difference between implementations 2 and 3. These devices can be either
CPUs and/or GPUs, though currently all devices have to be from the same
vendor. This allows heterogeneous computing with both CPU and GPU or
multiple GPUs, as long as they are from the same vendor. When using
different devices, more work (i.e. more LORs in this case) can be
assigned to the more powerful device. Currently any devices with memory
of 2 GB or less are ignored in order to prevent out of memory issues.
All operations are computed in single precision.

Implementation 3 was developed after implementation 2 as a separate
project to enable multi-device support and additionally to provide
OpenCL reconstruction without the need for third-party libraries.

Implementation 4
~~~~~~~~~~~~~~~~

Implementation 4 is a combination of implementations 1 and 3, meaning
that it is a pure CPU method that uses OpenMP for the parallellization,
as in implementation 1, but is implemented in matrix-free way as the
OpenCL methods. The matrix-free formulation itself does not essentially
differ from the OpenCL, except using C++ OpenMP code.

As with the OpenCL methods, the sensitivity image and Δ are computed,
but unlike the OpenCL methods, in implementation 4 these are output into
MATLAB/Octave where the actual reconstruction algorithms are used. Due
to this, implementation 4 supports more algorithms than 3, but less than
1. Supported ML methods include MLEM, OSEM, RAMLA and ROSEM, MAP-methods
OSL, BSREM and ROSEM-MAP along with all priors, though only one
algorithm and prior can be used at a time. All operations are computed
in double precision.

Implementation 4 was developed after the other implementations
(excluding CUDA in implementation 2) as a fallback method for
matrix-free computation without the need for OpenCL. It was also
developed for CPUs that lack OpenCL support and to provide numerically
more accurate matrix-free formulation.

Matrix-free formulation
~~~~~~~~~~~~~~~~~~~~~~~

The matrix-free forward and backprojection are implemented similarly
regardless of the used projector or reconstruction algorithm. Since in
PET the system matrix depicts the probability that an event originating
from voxel *j* is detected on LOR *i*, the ﬁrst goal is compute the
total distance that a LOR (or a TOR) traverses in the image domain. The
computations are performed by computing several LORs at the same time in
parallel. In the ﬁrst phase, the line intersection (or orthogonal
distance) is computed for each voxel along the LOR (TOR) as well as the
corresponding voxel index. The intersection lengths are summed together
as well as

*Ξ\ i* = *Σ\ l L\ il f\ l*

where

*a\ il* = *L\ il* / *Σ\ l\ L\ il*

where *L\ il* is the intersection length and *a\ il* the probability.

In implementation 4 the intersection lengths and voxel indices are then
saved in temporary variables. In case attenuation is included, then
*Σ\ l\ μ\ l\ L\ l*, where *μ* is the attenuation coefficient, is
computed as well.

After the ﬁrst phase, the inverse of *Σ\ l\ L\ il* is computed. If
attenuation is included, this inverse value is multiplied with
exp(*Σ\ l\ μ\ l\ L\ l*). With normalization enabled there is further
multiplication with the normalization coefficient. The resulting value
is then used to compute *a\ il* values. Randoms and/or scatter is then
added to *Ξ\ i* if either has been selected. The ﬁnal value is then used
to divide the current number of counts (*p\ i*)

*Θ\ i* = *p\ i* / (*Σ\ l\ a\ il* + *r\ i* + *s\ i*).

In the last step, the sensitivity image and the backprojection are
computed. Sensitivity image, however, is only computed during the very
ﬁrst iteration unless not enough memory is available for storage in
which case it will be computed on-the-ﬂy. When using implementation 4,
the intersection lengths and voxel indices are loaded from memory. In
OpenCL methods, however, both values are computed again, due to the high
memory costs of saving the variables in all the threads as well as the
slowness of the global memory in GPUs. Both the sensitivity image and
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
default, five (5) trapezoidal integration points are used. Each original
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
