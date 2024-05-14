OpenCL vs. CUDA vs. CPU
=======================

This page gives some explanation on when it is recommended to use OpenCL, CUDA or CPU. As such, this emphasizes the use of implementation 2 or custom reconstruction in Python. For documentation on the different 
implementations, see :doc:`implementation`.

When using the built-in reconstructions, in general it is recommended to use OpenCL. This is most tested and thus the most reliable. However, if use CUDA-capable GPU, PET and generic data can see improved performance with CUDA. 
For CT and/or SPECT, there shouldn't be significant differences. CPU version should only be used if no discrete GPU is available. The CPU version is simply a fallback method and lacks support for many algorithms. CPU is also only recommended
for PET and SPECT data, or for generic data with small dimensions. For CT, CPU is only suitable for small dimension cases. In general, OpenCL is recommended.

When using your own algorithms on Python using the OMEGA forward and backward projector operators, it is again recommended to use OpenCL. This time, however, OpenCL not only is about twice faster than CUDA, but also supports more
features when using CT, or CT-like, data. For example, the branchless distance-driven projector is not available in CUDA. This is simply due to lack of support for ``cudaTextureObject_t`` in PyCUDA. This might change in the future, 
however. However, if you need interoperability with PyTorch, you either have to use CUDA, or transfer the data into NumPy arrays before using either, e.g. if you have PyTorch tensor, you need to convert it into a NumPy array, 
then convert it into an PyOpenCL array which you can input into OMEGA. Then convert the resulting PyOpenCL array back to NumPy array and then to PyTorch. Which one is eventually the fastest method depends on the usecase. For non-CUDA
GPUs OpenCL is currently the only supported method.

On MATLAB/Octave, you can compute your own algorithms also using CPU, unlike on Python. However, CUDA is not available at all on MATLAB/Octave. In MATLAB/Octave OpenCL is, again, recommended though. 