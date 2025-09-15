Image reconstruction
====================

.. contents:: Table of Contents

This page outlines details on the two different ways to achieve image reconstruction: the built-in algorithms and the forward/backward projection operators. This page also outlines some details on how to obtain the (sparse) system matrix itself.

Whichever case is used, the geometry specifications, projector settings, and subsets are defined the same way, as well as any special properties like TOF.

Built-in reconstruction
-----------------------

The built-in reconstruction handles the entire reconstruction process in one process using the selected algorithm (see :doc:`algorithms`). The necessary selections are the number of iterations with ``options.Niter`` and the algorithm.
Optionally, subsets and/or regularization and/or preconditioners can be used if the algorithm supports them. You can enforce the positivity of most algorithms with ``options.enforcePositivity`` set to true.

At minimum, you only need to select a single algorithm. This will then use one iteration, with no subsets, by default. By default, projector type 1 is used except for CT-type data (i.e. when ``options.CT = true``) in which case projector type 4 is used.

The number of subsets is selected with ``options.subsets`` and projector with ``options.projector_type``.

The reconstruction is performed in ``reconstructions_main`` for PET, ``reconstructions_mainCT`` for CT, and ``reconstructions_mainSPECT`` for SPECT.

Using the forward and/or backward projection operators
------------------------------------------------------

Alternatively, the user can manually compute any and all algorithms and simply call the OMEGA forward (Ax) and/or backward (A^Ty) projection operators. In such cases, the user is responsible for the number of iterations and any and all algorithm-
specific settings. Note that by default emission tomography is assumed! You can switch to CT-style (intersection length, not probability) with ``options.CT`` as true. This should be applied even if you don't use CT data, but are not using emission
tomography either. Note that when using subsets handled by OMEGA with the forward and/or backward projection operators, you need to manually make sure that the data is ordered into the subsets correctly. OMEGA can provide the indices, but the user has
to perform the actual sorting operation using the indices. The examples include several subset cases. Note that when using subsets, you need to input the current subset number with ``A.subset = currentSubset`` before you compute the forward and/or backward 
projection, with zero-based numbering in Python and one-based in MATLAB/Octave. Below is an example of subset use in Python:

.. code-block:: python

	for it in range(A.Niter):
		for k in range(A.subsets):
			# This is necessary when using subsets
			A.subset = k
			fp = A * f
			bp = A.T() * fp

In MATLAB/Octave, you should first specify all the parameters in the ``options`` struct as with any other method. An important selection in MATLAB/Octave is the implementation. Selecting implementation 2, 3 or 5 uses OpenCL for the forward and/or 
backward projection operators (e.g. ``options.implementation = 2``) which is the recommended method. Alternatively, it is possible to use CPU also with implementation 4, or create the system matrix with implementation 1 (see below for the matrix creation). 
Regardless of the implementation, you should create the necessary class object after inputting the desired options into the ``options`` (or whatever you name it) struct, with e.g. ``A = projectorClass(options);``. For PET data, with corrections such as 
normalization, you should also call ``A = initCorrections(A);``. If you use subsets, and let OMEGA handle the subset selection, you need to reorder the input data: ``input = single(raw_SinM(A.index));``. This also applies to any possible corrections or 
forward projection masks that operate in the measurement space, such as ``A.param.normalization = A.param.normalization(A.index);``. This means that ``A.index`` contains the indices for each subset and behaves a bit differently depending on the subset type. 
For 1-7, you can simply use the above methods, but for types 8-11, you should only let the index-variable to operate on the third dimension. See the CT-examples for a concrete example on how to do this. Compute the forward projection simply with ``y = A * f;``, 
where ``f`` is the image volume in vector format. Backprojection is computed with ``x = A' * y``. If you make modifications to the geometry, such as change the projector, you should make the change in the ``options`` struct and recreate the class 
object with ``A = projectorClass(options);``. If the subset selections are not modified, it is not necessary to repeat those steps. If you modify the subset selections, you need to reload all the input data and reorder them again.

For Python, the process is similar, but you don't need to separately create any class object. The options you input are already input to the class object. However, you need to add the projector and initialize it before calling either the forward or 
backward projections: ``options.addProjector()`` followed by ``options.initProj()``. The reconstructions can be coupled with either PyOpenCL (default), Arrayfire, CuPy or PyTorch data. For Arrayfire, set ``options.useAF = True``. For CuPy, 
use ``options.useCUDA = True`` and ``options.useCuPy = True``. For PyTorch, use ``options.useCUDA = True`` and ``options.useTorch = True``. Call forward projection with ``y = options * f``, where ``f`` is either PyOpenCL buffer, Arrayfire array, 
CuPy array or PyTorch tensor. When using CuPy or PyTorch it is important to remember that OMEGA is column-major, while both are by default row-major! For CuPy, you can use Fortran-ordering as with NumPy, but with PyTorch you need to be extra careful. 
Note that ``options`` can also be called as something else too (such as ``A``). Backprojection is similarly with ``x = options.T() * y``. The input variables should be vectors. If you want to make modifications to the setup that is dependent of the 
``options`` variable (or whatever its name is), you need to rerun the ``options.addProjector()`` and ``options.initProj()`` steps.

Standalone regularization
-------------------------

For Python, it is also possible to manually call some of the regularization methods in a similar way (using PyOpenCL, Arrayfire, CuPy or PyTorch input data) as the forward and/or backward projection operators. 
These are RDP, non-local methods, and gradient-based TV. These are located in omegatomo/util/priors.py. 

For example

.. code-block:: python

	from omegatomo.util.priors import RDP
	from omegatomo.util.priors import NLReg
	from omegatomo.util.priors import TV
	
imports the functions. For details, see ``help(RDP)`` for RDP and similarly for the others. These can be seamlessly combined with the forward and/or backward projection operators. Note that you can use these completely separately too without any need
to use the forward/backward projection operators or creating the class object. Simply make sure the inputs are correct and correctly formatted.

Creating and storing the system matrix
--------------------------------------

This is, first of all, MATLAB/Octave only feature. Second, it supports only projector type 1. Third, this is only double precision currently. The process is otherwise identical to above, but instead of computing Ax you can create the matrix
itself with ``B = formMatrix(A);``. This creates the whole (sparse) system matrix. A subset, if you've selected subsets, can be computed with ``B = formMatrix(A, subsetNumber)``. Note, however, that this is the TRANSPOSE of the matrix! 
I.e. forward projection is computed with ``B' * f`` and backward projection with ``B * y``. Alternatively, you can also transpose the matrix.

The reason why the matrix is the transpose is for efficiency reasons. Also, before the matrix formation, a prestep is performed which determines the number of voxels traversed per ray and if some of the rays do not intersect with the FOV.

.. note::

	When forming the system matrix, the source and detector (or detector-detector) positions HAVE to be outside the FOV.