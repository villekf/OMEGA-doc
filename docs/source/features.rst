Features
========

(Almost) Full list of features for OMEGA v2.0

MATLAB/GNU Octave/Python
------------------------

Any data
^^^^^^^^
* Supports any data that uses ray-tracing
 * Only the source and detector coordinates need to be input
 * Source and/or detector can be inside the FOV
* Supported projectors include:
 * Improved Siddon's ray tracer
  * Also multi-ray version available
 * Interpolation-based ray tracer
 * Volume of intersection ray tracer
 * Supports hybrid projectors
 * All projectors have 100% GPU support
* Use custom algorithms by using the built-in projectors
 * A class object can be created which can be used to compute the forward and/or backward projections
 * Available in MATLAB, GNU Octave and Python
* Wide range of algorithms supported:
 * MLEM, OSEM, RAMLA, BSREM, MBSREM, PKMA, LSQR, CGLS, FISTA, (A/E)COSEM, ROSEM, DRAMA, PDHG, CV, FDK/FBP, PDDY, SPS, RBI and OSL
  * PDHG supports L1, L2 and Kullback-Leibler optimization
  * FDK/FBP supports several different windowing methods: Hamming, Hann, Blackman, Nuttal, Gaussian, Shepp-Logan, cosine, Parzen (de la Vallée Poussin) or none (ramp)
* Wide range of regularization techniques/priors:
 * Quadratic prior, Huber, MRP, Weighted mean, TV, NLM, RDP, APLS, (proximal) TGV, proximal TV and hyperbolic prior
 * Several different non-local variations
 * TV, NLM and APLS support anatomic/prior image weighting
* Supports time-varying dynamic data
 * Reconstruct dynamic data with static algorithms
* OpenCL and CUDA support (single precision only)
* Point spread function blurring
 * Optional deblurring available
* Save the last iteration or specific iterations
* Supports subsets
 * Several different ways to select subsets
 * Non-PET/Non-CT/Non-SPECT data or list-mode PET data supports three subset selection methods
  * Divide the data into N segments
  * Take every Nth measurement
  * Randomly sample the measurement data
* Seven image-based preconditioners
 * Diagonal preconditioner
 * EM-preconditioner
 * IEM-preconditioner
 * Momentum-based preconditioner
 * Gradient-based preconditioner
 * Filtering-based preconditioner
 * Curvature-based preconditioner
* Two measurement-based preconditioners
 * Diagonal preconditioner
 * Filtering-based preconditioner
* Both filtering-based preconditioners support the same windowing functions as FDK/FBP
* Filtering-based preconditioner can optionally be used for N iterations/subiterations only
* Supports positivity enforcement for non-Poisson algorithms
* Supports manual initial values
* Allows the storage and output of the intermediate forward projections
* Insert scatter and/or randoms correction data into the reconstruction with supported algorithms (Poisson-based algorithms)
* Allows input of object offsets
 * If the object is not centered on the origin
* Use 2D masks to limit forward projection and/or backprojection
 * 2D mask in measurement space can be used to ignore certain measurements (values that are set at 0 are ignored)
 * Similarly in backprojection the 2D mask can be used to specify the voxels to reconstruct (likewise values that are 0 are not reconstructed)
* Supports multi-resolution reconstruction
 * Extended FOV can have reduced resolution
 * Resolution can be manually set
 * Can be set only for axial, only for transaxial or for both directions
 * Should work with all non-SPECT data
* Allows the use of extended FOV without multi-resolution as well
 * Priors/regularization computed only in the main volume
 * Automatic cropping of the image

PET features
^^^^^^^^^^^^

* Optimized for PET
* Load GATE ROOT data for cylindrical/ECAT PET systems
 * Automatically convert the PET data into sinograms
 * Export trues, prompts, randoms and scatter sinograms
  * Rayleigh or Compton scatter in the detector and/or phantom can be separately selected
 * Form and reconstruct dynamic sinograms
 * Obtain a ground truth image from the GATE ROOT data
* Supports orthogonal distance-based ray tracer
* All projectors automatically use probabilities rather than the length of the line of intersection
* Automatically compute detector/source coordinates for cylindrical PET data (both GATE and non-GATE data)
* Several other subset selection methods
 * Use every Nth column sinogram bin
 * Use every Nth sinogram row
 * Use every Nth sinogram column
 * Use every Nth sinogram
 * Randomly sample the sinograms
 * Select the sinograms based on prime factors
* Supports attenuation correction during reconstruction, either image-based or sinogram-based
* Supports normalization correction during reconstruction
* Supports any manual sinogram-based correction
* Supports time-of-flight (TOF) data
* Supports formation of TOF sinograms from GATE data
* Supports list-mode data
* Supports pseudo detectors/rings or ring gaps
* Supports easy inclusion of GATE attenuation maps as the attenuation correction images

CT data
^^^^^^^
* Optimized for (CB)CT data
* Automatically load Planmeca CBCT data
* Automatically load image-based projections (e.g. tiff-images)
* Load GATE CT projections images
* Automatically compute source/detector coordinates for CBCT systems
 * Allows input of source and/or detector offsets
 * Supports multi-bed (step-and-shoot) data
* Supports GPU-optimized projectors
 * Voxel-based backprojector as well as the previously mentioned forward projectors
 * Branchless distance-driven projector, both for forward and backward projections
  * Allows subtraction of the DC-component
 * Supports hybrid projectors
* Supports projection image extrapolation
 * Automatically extrapolate and weight projections to fix out-of-FOV artifacts
* Supports offset correction
 * Offset weights can be automatically computed
 * Each projection has their own weight
* Several other subset selection methods
 * Use every Nth column of the projection image
 * Use every Nth projection image row
 * Use every Nth projection image column
 * Use every Nth projection image
 * Randomly sample the projection images
 * Select the projection images based on prime factors
* Most of the Poisson-based algorithms are supported with transmission-based (i.e. Lambert-Beer law) data as well
 * These include PKMA, MBSREM, RAMLA, ROSEM, OSEM, MLEM and BSREM

SPECT data
^^^^^^^^^^
* Optimized for parallel hole SPECT data
* Load GATE SPECT projections images
* Automatically compute detector response function for hexagonal or round holes
* Supports rotation-based projector
* Supports same subset selection methods as CT

MATLAB/GNU Octave only
----------------------

* Load GATE ASCII and LMF (LMF support has bee deprecated) data for cylindrical/ECAT PET systems
* Load Inveon PET list-mode data 
* Load Siemens Biograph mCT and Vision list-mode data
 * Supports both binned 32-bit list-mode data as well as 64-bit
 * Supports also .ptd-files
* Automatically convert any of the above PET data into sinograms
* Obtain a ground truth image from GATE ASCII or LMF data (LMF support has bee deprecated)
* Several different "implementations" available that perform the computations either on the CPU or the GPU
 * Implementation 1 forms a sparse system matrix that is used in computations
  * Double precision only
  * System matrix can be extracted
  * System matrix can be created for only a subset of data
  * Supports all features except hyperbolic prior
 * Implementation 2 uses OpenCL or CUDA for the reconstructions
  * Supports all features
  * Single precision only
 * Implementation 3 uses OpenCL for the reconstructions
  * Supports only MLEM/OSEM
  * Single precision only
 * Implementation 4 is a parallel matrix-free CPU implementation
  * Uses OpenMP
  * Supports all features except hyperbolic prior
  * Single (default) or double precision
 * Implementation 5 is similar to implementation 4, except that forward and backward projections are performed using OpenCL
  * All other computations are done in MATLAB/GNU Octave
  * Supports all features except hyperbolic prior
  * Single precision only
* Supports custom algorithms with the use of OpenCL or CPU
 * A class object needs to be created first
 * Forward and/or backward projections are transferred to host (CPU) first if using OpenCL
 * Simply using ``y = A * x`` computes the forward projection when A is the class object
 * Similarly, ``x = A' * y`` computes the backprojection
 * Supports the system matrix approach, OpenCL or OpenMP (CPU)
 * For SPECT, only OpenMP version is available
* Visualization function that does not require any toolboxes
* Supports arc correction for PET (MATLAB only)
* Supports randoms/scatter smoothing
* Supports randoms variance reduction (PET only)
* Supports computation of the normalization coefficients from a normalization measurement (PET only)
 * Component-based
* Supports increasing the sampling (i.e. interpolation) of PET sinograms
* Supports sinogram gap filling
* Supports scaling of CT-based attenuation coefficient to 511 keV attenuation coefficients
* Supports pre-correcting the sinogram
* Allows to automatically crop voxelized phantoms/sources for MC simulations
* Individual functions to load MetaImage or Interfile data
* Few additional priors
 * FMH and L-filter

Python only
-----------

* Supports custom algorithms with the use of OpenCL or CUDA
 * All computations can be performed on the GPU without the need to transfer the data to host first
 * ``y = A * x`` computes the forward projection
 * ``x = A.T() * y`` computes the backprojection
 * Allows the seamless use of Arrayfire arrays for easy implementation of custom, user-made, algorithms without the need for GPU coding knowledge
 * Same code can be used for either OpenCL or CUDA