Choosing the optimal implementation
===================================

This page explains the details on the different implementations and their different advantages.

In short, use implementation 2. This is the only one available when using Python as well. For details, see below. If you cannot use implementation 2, use implementation 5 if you have a discrete GPU. If not, use implementation 4.

Implementation 1 is a CPU-only implementation. It computes and stores the actual (sparse) system matrix for the selected geometry. It supports only projector types 1-3 and their hybrid versions. In general,
this implementation is not recommended. For custom algorithms, it can sometimes, however, be beneficial to obtain the system matrix for a 2D case. In such cases, implementation 1 can be useful, and fast. However,
in short implementation 1 is NOT recommended. Implementation 1 is the only implementation that uses double-precision (64-bit floats) by default and also cannot use single precision values.

Implementation 2 is OpenCL-, CUDA-, or CPU-based implementation. Default is OpenCL, but CUDA can be selected by setting ``options.useCUDA = True`` in Python or ``options.use_CUDA = true`` in MATLAB/Octave. 
CPU can be enabled similarly with ``options.useCPU = True`` in Python or ``options.use_CPU = true`` in MATLAB/Octave. As mentioned above, implementation 2 is highly recommended as it has the most amount of features
and if you have a discrete GPU, it is also the fastest implementation. In implementation 2, all forward and backward projection operations are computed on-the-fly. This makes it much less memory intensive as implementation 1.
Implementation 2 uses Arrayfire to compute many of the algorithms and as such it is required. This is problematic in Windows environments if you are using MinGW compiler (such as when using Octave) as you need to manually compile Arrayfire. 

Implementation 3 is pure OpenCL-based implementation. This means that it simply uses OpenCL and not any external library. The downside is that only MLEM/OSEM are supported. As such implementation 3 is not recommended. 
Most likely it will also be deprecated in the future. However, it can be useful for GPU-based reconstruction of PET data if Arrayfire installation is not possible.

Implementation 4 is purely CPU-based implementation. It uses OpenMP for multithreaded reconstruction. As with implementations 2 and 3, it computes the system matrix on-the-fly. It also supports most of the features, but lacks
implementation 2 specific features such as branchless distance-driven projector. Generally not recommended, but is viable method for PET and SPECT reconstruction or for generic data that is not very high dimensional. For CT data,
implementation 4 is not recommended. Can also be used for custom algorithm computations. Implementation 4 uses single-precision by default, but can also use double-precision if you set ``options.useSingles = false``.

Implementation 5 is pure OpenCL-based implementation. The difference to implementation 3 is that only the forward and backward projections are computed on the OpenCL device (e.g. GPU). The rest of the computations are performed
on MATLAB/Octave. Supports branchless distance-driven projector and other GPU-based projectors. It thus supports more features than implementation 3 or 4. However, implementation 5 is only recommended if implementation 2 cannot
be used. For custom reconstructions, implementation 5 is the recommended method though it is used if implementation 2 or 3 is selected. 