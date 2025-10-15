Using TOF data
==============

.. contents:: Table of Contents

TOF data can be reconstructed either in sinogram or list-mode format. This page outlines the variables needed to perform TOF reconstructions. 

There are two ways you can add TOF data: either using the built-in features or input your own TOF distances and standard deviation. For inputting your own, see "Inputing custom TOF values" below.

Note that in all cases, the measurement data has to contain TOF data! See "TOF in other PET data" below for details.

TOF properties
--------------

The following parameters can and need to be set to use TOF data:

``options.TOF_bins`` this variable determines both the number of TOF bins and also whether TOF data is used at all. Setting it to 1 will disable the use of TOF data, however, it will not convert TOF data into non-TOF data (for that see options.TOF_bins_used below). This value needs to correspond to the number of TOF bins you have (or want to have when using GATE data).

``options.TOF_width`` this is the width of each TOF time bin (in seconds). Currently, each bin needs to have identical lengths.

``options.TOF_offset`` this value specifies possible offset in the TOF data (in seconds). What this means is that if your TOF bins are not centered at zero (center of FOV) you can specify the offset here. Offset is applied to Biograph mCT and Vision data obtained from the scanner.

``options.TOF_noise_FWHM`` This parameter has two properties. The first one applies to any TOF data that is saved by OMEGA (GATE, Inveon/Biograph list-mode), the second only to GATE data. The first use of this parameter is in naming purposes as this value is included in the filename. If you use Biograph data it is recommended to set this to the actual time resolution of the scanner although it is not necessary. This variable is ignored if you manually load the measurement data. The second use of this variable is to add temporal noise to the GATE data with the specified full width at half maximum (FWHM, in seconds). For more information on creating GATE TOF data see the next section.

``options.TOF_FWHM`` the FWHM of the TOF time resolution used in image reconstruction (in seconds). This variable is used solely for image reconstruction purposes and as such can be different from the above options.TOF_noise_FWHM. This is the actual time resolution of the device that is used when computing the TOF weighted reconstruction.

``options.TOF_bins_used`` the number of TOF bins used. Currently, this value has to either be the same as options.TOF_bins or 1. In the latter case the TOF data set is converted (summed) into a non-TOF data set before the reconstruction. In the future, this will allow TOF mashing.

Enabling TOF
------------

TOF data can be enabled simply by adding more TOF bins than 1, i.e. ``options.TOF_bins`` > 1.

TOF in GATE data can either be included directly in GATE by setting the `temporal resolution module <https://opengate.readthedocs.io/en/latest/digitizer_and_detector_modeling.html#time-resolution>`_ or by simply adding the preferred temporal noise in OMEGA. In the first case, you will only be able to use the one temporal resolution that you set in GATE, but in the latter case you will be able to choose any temporal resolution and use the same simulated data to create different TOF data sets each with different temporal resolution.

If you are using TOF data with the temporal resolution module, you should set ``options.TOF_noise_FWHM = 0`` such that no additional noise is included. ``options.TOF_FWHM`` on the other hand should be the FWHM of the added temporal noise multiplied with sqrt(2).

When using GATE data, the actual temporal resolution will most likely differ from the one specified by ``options.TOF_noise_FWHM``. If you want to know the actual time resolution with the specified added noise, you should run a simulation with a point source. Alternatively, multiplying with sqrt(2) should be relatively accurate in most cases.

Inputing custom TOF values
--------------------------

Instead of using the above, you can also enable TOF data by simply inputting ``options.TOF_bins`` and ``options.TOFCenter`` values. ``options.TOFCenter`` are the distances of each bin from the center in millimeters thus the number of elements 
needs to be the same as the number of bins. In addition to that, you need to input the standard deviation (in millimeters, usually derived from the FWHM) of the TOF data into ``options.sigma_x``.

Internally both values are computed (in MATLAB/Octave) as:

.. code-block:: matlab

	c = 2.99792458e11; % speed of light in mm/s
	obj.param.sigma_x = (c*obj.param.TOF_FWHM/2) / (2 * sqrt(2 * log(2)));
	edges_user = linspace(-obj.param.TOF_width * obj.param.TOF_bins/2, obj.param.TOF_width * obj.param.TOF_bins / 2, obj.param.TOF_bins + 1);
	edges_user = edges_user(1:end-1) + obj.param.TOF_width/2; % the most probable value where annihilation occurred
	TOFCenter = zeros(size(edges_user));
	TOFCenter(1) = edges_user(ceil(length(edges_user)/2));
	TOFCenter(2:2:end) = edges_user(ceil(length(edges_user)/2) + 1:end);
	TOFCenter(3:2:end) = edges_user(ceil(length(edges_user)/2) - 1: -1 : 1);
	if isfield(obj.param, 'TOF_offset') && obj.param.TOF_offset > 0
		TOFCenter = TOFCenter + obj.param.TOF_offset;
	end
	TOFCenter = -TOFCenter * c / 2;
	

TOF in other PET data
---------------------

Any PET data with TOF can be used. TOF is assumed to be the fourth dimension of the sinogram matrix (potential time steps are assumed to be the fifth dimension) when using sinogram data. All values except for ``options.TOF_noise_FWHM`` need to be filled.

Biograph mCT and Vision allow for automatic extraction of TOF data. However, currently only the default number of bins (13 for mCT and 33 for Vision) are available. 

For details on using list-mode data with TOF, see :doc:`customcoordinates`.

TOF integration points
----------------------

By default, the trapezoidal integration uses 4 points. However, for small TOF FWHM values this might not be accurate enough for accurate reconstruction. For implementation 4 this can be modified by changing the value of ``TRAPZ_BINS`` in ``projector_functions.h``. For implementation 2 (OpenCL/CUDA), modify ``TRAPZ_BINS`` with the desired number of bins in ``general_opencl_functions.h``. Implementation 4 requires recompilation before the changes take effect (run ``install_mex`` again). Implementation 2 does not require anything else except re-running the reconstruction.
