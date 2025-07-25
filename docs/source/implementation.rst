Choosing the optimal implementation
===================================

This page explains the details on the different implementations and their different advantages. This does not apply to custom reconstructions in Python! Custom reconstructions in Python directly call the OpenCL or CUDA kernels.

In short, use implementation 2 with OpenCL as it is the most feature-rich and tested one. This is the only implementation available when using Python as well (CUDA and CPU are available too). For details, see below. If you cannot use implementation 2, use implementation 5 if you have a discrete GPU. If not, use implementation 4.

Implementation 1 is a CPU-only implementation. It computes and stores the actual (sparse) system matrix for the selected geometry. It supports only projector type 1. In general,
this implementation is not recommended. For custom algorithms, it can sometimes, however, be beneficial to obtain the system matrix for a 2D case. In such cases, implementation 1 can be useful, and fast. However,
in short implementation 1 is NOT recommended. Implementation 1 is the only implementation that uses double-precision (64-bit floats) by default and also cannot use single precision values. While most features should work with
implementation 1, this hasn't been extensively tested and even then only with PET data and 2D fan-beam CT data. Only the system matrix creation is considered reliable and thus built-in reconstructions are not recommended! See :doc:`reconstruction` for details.

Implementation 2 is OpenCL-, CUDA-, or CPU-based implementation. Default is OpenCL, but CUDA can be selected by setting ``options.useCUDA = True`` in Python or ``options.use_CUDA = true`` in MATLAB/Octave. 
CPU can be enabled similarly with ``options.useCPU = True`` in Python or ``options.use_CPU = true`` in MATLAB/Octave. Of the three, OpenCL is the recommended one as it is most extensively tested. CUDA should be a viable option
with CUDA capable GPUs. CPU should only be used for PET and SPECT and even then OpenCL is recommended. Note that you can also use CPUs as OpenCL devices if runtimes, or `POCL <https://portablecl.org/>`_, for the CPU are installed. As mentioned above, implementation 2 is highly recommended as it has the most amount of features
and if you have a discrete GPU, it is also the fastest implementation. In implementation 2, all forward and backward projection operations are computed on-the-fly. This makes it much less memory intensive when compared to implementation 1.
Implementation 2 uses Arrayfire to compute many of the algorithms and as such it is required. This can be problematic in Windows environments if you are using MinGW compiler (such as when using Octave) as you need to manually compile Arrayfire. 

Implementation 3 is pure OpenCL-based implementation. This means that it simply uses OpenCL and not any external library. The downside is that only MLEM/OSEM are supported. As such implementation 3 is not recommended. 
Most likely it will also be deprecated in the future. However, it can be useful for GPU-based reconstruction of PET data if Arrayfire installation is not possible.

Implementation 4 is purely CPU-based implementation. It uses OpenMP for multi-threaded reconstruction. As with implementations 2 and 3, it computes the system matrix on-the-fly. It also supports most of the features, but lacks
implementation 2 specific features such as branchless distance-driven projector. Generally not recommended, but is a viable method for PET and SPECT reconstruction or for generic data that is not very high dimensional. For CT data,
implementation 4 is not recommended. Can also be used for custom algorithm computations. Implementation 4 uses single-precision by default, but can also use double-precision if you set ``options.useSingles = false``. Hasn't been
as extensively tested as implementation 2 and some features may not work correctly. Most PET and SPECT features, such as OSEM reconstruction, work fine though.

Implementation 5 is pure OpenCL-based implementation. The difference to implementation 3 is that only the forward and backward projections are computed on the OpenCL device (e.g. GPU). The rest of the computations are performed
on MATLAB/Octave. Supports branchless distance-driven projector and other GPU-based projectors. It thus supports more features than implementation 3 or 4. However, implementation 5 is only recommended if implementation 2 cannot
be used. For custom reconstructions, implementation 5 is the recommended method though it is used even if implementation 2 or 3 is selected. Same things apply to implementation 5 as to 4, i.e. it is not as extensively tested 
and some features might not work correctly.