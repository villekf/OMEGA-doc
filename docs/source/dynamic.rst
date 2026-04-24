Dynamic image reconstruction
============================

.. contents:: Table of Contents

Starting from version 2.2, it is now possible to perform "true" dynamic reconstruction. In previous versions, the dynamic reconstruction was done individually for each timestep, but now the dynamic reconstruction is done for all timesteps at the same time. 
This also enables the use of temporal regularization with two temporal regularization methods available: A simple temporal smoothness prior and temporal total variation (TV).

Currently input sinogram or projection data needs to have the same number of sinograms/projections per timestep. Alternatively, you can use the "listmode" format instead which allows an arbitrary number of measurements per timestep. See details for both below.

In all cases, the input data should be stored in cell matrices in MATLAB/Octave and in lists in Python. Each timestep should be in one cell element/list index.

.. note::

   Dynamic reconstruction has only been tested for PET and SPECT at the moment!

Built-in reconstruction with sinogram or projection data
--------------------------------------------------------

Generally, dynamic sinogram or projection data should use the same number of sinograms/projections per timestep. This includes any possible TOF bins. While the data could be stored in 4D or 5D (with TOF) format in this case, cell/list format is highly recommended.

If you need to use arbitrary number of sinograms/projections per timestep, see the below section.

Randoms/scatter correction is supported when using sinograms/projections, but not when using listmode data.

Built-in reconstruction with listmode data
------------------------------------------

Dynamic reconstruction now also supports "listmode" data. This means the coordinate-based and index-based reconstructions, as outlined in :doc:`customcoordinates`. For coordinate-based reconstruction, you need to input the coordinates for the source and detector as before, but 
this time in different cell/list indices. The number of indices should equal the number of timesteps. The number of measurements per timestep can be anything, as long it is nonzero. Functionality is otherwise identical, so see the previous link for help on using the coordinates themselves.
The measurement data should be stored in cell matrices/lists as well. That is, ``options.SinM`` should be a cell matrix in MATLAB/Octave and a list in Python.

Index-based reconstruction works similarly, and the indices should be stored in cell matrices/lists. The coordinates should be as before with no changes compared to the static case. If you need an arbitrary number of sinograms/projections per timestep, use the index-based reconstruction
instead. However, unlike with listmode-based index-based reconstruction, you should then fill ``options.SinM`` with the corresponding measurements so not just ones. However, in both cases ``options.SinM`` should be a cell matrix in MATLAB/Octave and a list in Python. So a separate cell/list
for each timestep. The order of the measurements should also be the same as with the index variables (``options.trIndex`` and ``options.axIndex``).

Randoms correction is not supported when loading dynamic data with OMEGA built-in functions, but you can manually input them as negative values as outlined in the above link. Officially, however, randoms/scatter correction is not supported yet. All other corrections work, but they
have to be identical for each timestep. If you use sinogram/projection data with arbitrary number of sinograms/projections and need to perform normalization correction, it is recommended to precorrect the data rather than perform the correction during the reconstruction.

Temporal regularization
-----------------------

Regardless of the above choices, temporal regularization can be used in all cases. Note that spatial regularization is separate from temporal and both or either can be on. For temporal regularization, currently only two methods are available. 
Temporal smoothness prior can be enabled with ``options.temporal_smoothness`` and temporal TV with ``options.temporalTV``. Both can be adjusted with ``options.beta_temporal``. Temporal TV also includes the smoothness parameter present in gradient-based
TV implementations, which can be adjusted with ``options.temporalTVsmoothing``.

The temporal smoothness prior regularizes by subtracting the previous and "next" timestep. TV works the same as spatial TV except that the regularization is done in the temporal dimension (and is 1D).

Using the forward and/or backward projection operators
------------------------------------------------------

Using dynamic data is possible with the included forward/backward projection operators, but in such cases the user needs to adjust everything themselves. Currently no temporal regularization methods exist that can be used in conjunction with the operators.

Corrections
-----------

Attenuation correction is supported with dynamic data. The attenuation data can be either static of dynamic. In the case of dynamic, make sure that the size of the attenuation image matches that of the final reconstructed image with all timesteps. 

Only static normalization correction is supported. Randoms are supported for sinogram/projection data, but not for listmode. 