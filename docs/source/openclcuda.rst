OpenCL vs. CUDA vs. CPU
=======================

This page gives some explanation on when it is recommended to use OpenCL, CUDA or CPU. As such, this emphasizes the use of implementation 2 or custom reconstruction in Python. For documentation on the different 
implementations, see :doc:`implementation`.

When using the built-in reconstructions, in general it is recommended to use OpenCL (in MATLAB/Octave this equals to implementation 2 without CUDA or CPU). This is the most tested and thus the most reliable method. However, if use CUDA-capable GPU, PET and generic data can see improved performance with CUDA. 
For CT and/or SPECT, there shouldn't be significant differences. CPU version should only be used if no discrete GPU is available. The CPU version is simply a fallback method and lacks support for CT-specific projectors. CPU is also only recommended
for PET and SPECT data, or for generic data with small dimensions. For CT, CPU is only suitable for small dimensional cases. In general, OpenCL is recommended.

When using your own algorithms on Python using the OMEGA forward and backward projector operators, the situation is a bit different. In general there isn't that much difference between OpenCL or CUDA, except in the way to interoperability.
OpenCL only supports interoperability with Arrayfire arrays (and PyOpenCL arrays), meaning you can input OpenCL Arrayfire arrays (or PyOpenCL arrays) into the forward and/or backward projections. In CUDA, CuPy and PyTorch are supported, though the latter also uses CuPy internally.
If you wish to input a PyTorch tensor into OMEGA forward/backward projection operator, you need to use CUDA (and the tensor HAS to be a CUDA device tensor). For OpenCL, you must move
the data through the host (NumPy) first if you want to utilize PyTorch. For ROCm, you thus need to use OpenCL on OMEGA and cannot directly use PyTorch tensors at the moment. Alternatively in CUDA, you can also use CuPy arrays, and 
also PyCUDA GPUArrays, though CuPy is recommended.

On MATLAB/Octave, you can compute your own algorithms also using CPU, unlike on Python. However, CUDA is not available at all on MATLAB/Octave when using custom algorithms. In MATLAB/Octave OpenCL is, again, recommended though. 
However, if you are reconstructing a 2D case, then implementation 1 (CPU) is highly recommended. Otherwise implementation 5 is recommended.

Custom algorithm development is recommended to be performed on Python, while built-in reconstructions are generally more mature on MATLAB/Octave.