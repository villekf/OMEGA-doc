High-dimensional computing
==========================

In case of high-dimensional cases, there are ways to circumvent running out of GPU memory. Note that all of these methods require sufficient CPU RAM and only help with limited GPU memory.

High-dimensional measurement data
---------------------------------

This is especially relevant for PET TOF data, but can be used with any other data too. By default, all measurement data is transfered to the GPU before the reconstruction. However, it is possible to limit only the current subset to the GPU. This is
achieved by setting ``options.loadTOF`` to false. Although the name suggests TOF-only, it works with any measurement data. However, this only works if subsets are used. This also works with list-mode/custom coordinate data, where the coordinates are
also split according to the subset. The measurement data still needs to fit into the host RAM. Note that you can further reduce memory usage by using unsigned 16-bit integer input data whenever applicable. There are no restrictions when using this mode, 
it can only lead to slower reconstructions if there is sufficient memory on the GPU and the data is transfered on-the-fly.

High-dimensional CT data data
-----------------------------

This is specifically designed for CT data and especially for µCT. This is similar to the above, but the image is also reconstructed in subsets. Like above, this depends on the number of subsets as well. Unlike above, which supports any reconstruction
algorithm, this method only supports FDK, PDHG, and its variants, and PKMA. Note that FDK still computes only a single backprojection despite using subsets. The rest divide the reconstruction into the subsets as normal. This feature is enabled by 
setting ``options.largeDim`` to true. The setting of ``loadTOF`` is irrelevant if ``largeDim`` is true. For regularization, only non-local methods, RDP, GGMRF, gradient-based TV, and hyperbolic prior are supported. Diagonal and EM image-based 
preconditioners are supported as well as filtering-based measurement-based preconditioner. With sufficient number of subsets, any type of data can be reconstructed with any type of GPU. However, the input measurement data and output reconstruction has to
fit to the computer RAM. For PDHG, the intermediate variables, such as dual estimate, need to fit too. 