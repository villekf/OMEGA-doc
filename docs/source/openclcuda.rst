OpenCL vs. CUDA vs. CPU
=======================

This page gives some explanation on when it is recommended to use OpenCL, CUDA or CPU. As such, this emphasizes the use of implementation 2 or custom reconstruction in Python. For documentation on the different 
implementations, see :doc:`implementation`.

When using the built-in reconstructions, in general it is recommended to use OpenCL (in MATLAB/Octave this equals to implementation 2 without CUDA or CPU). This is most tested and thus the most reliable. However, if use CUDA-capable GPU, PET and generic data can see improved performance with CUDA. 
For CT and/or SPECT, there shouldn't be significant differences. CPU version should only be used if no discrete GPU is available. The CPU version is simply a fallback method and lacks support for many algorithms. CPU is also only recommended
for PET and SPECT data, or for generic data with small dimensions. For CT, CPU is only suitable for small dimension cases. In general, OpenCL is recommended.

When using your own algorithms on Python using the OMEGA forward and backward projector operators, the situation is a bit different. In general there isn't that much difference between the two, except in the way to interoperability.
OpenCL only supports interoperability with Arrayfire arrays, meaning you can input OpenCL Arrayfire arrays into the forward and/or backward projections. In CUDA, CuPy and PyTorch are supported, though the latter also uses CuPy internally.
If you wish to input a PyTorch tensor into OMEGA forward/backward projection operator, you either need to use CUDA where you can do this without any modifications (as long as the tensor is a CUDA device tensor). For OpenCL, you must move
the data through the host (NumPy) first. Alternatively in CUDA, you can also use CuPy arrays.

On MATLAB/Octave, you can compute your own algorithms also using CPU, unlike on Python. However, CUDA is not available at all on MATLAB/Octave when using custom algorithms. In MATLAB/Octave OpenCL is, again, recommended though. 
However, if you are reconstructing a 2D case, then implementation 1 (CPU) is highly recommended. Otherwise implementation 5 is recommended.

Custom algorithm development is recommended to be performed on Python, while built-in reconstructions are generally more mature on MATLAB/Octave.