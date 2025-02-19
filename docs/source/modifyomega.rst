Modifying OMEGA source code
===========================


Adding built-in algorithms
--------------------------

The process of adding new built-in algorithms into OMEGA is actually quite straightforward. In most cases, you only need to do some C++ coding in order to enable GPU support for both CUDA and OpenCL. This means
that most of the time you don't need to do actual GPU coding to implement algorithms on the GPU. The following points explain the general process of adding new algorithms. The list assumes that the algorithm is a
reconstruction algorithm and not a regularizer, but the below can also be used to implement regularizers, many of the steps are simply omitted. Note that the below instructions generally proceed backwards, starting from the results
obtained after backprojection.

1. Add a suitable boolean variable to the ``structs.h file`` in the ``RecMethods_`` struct. Generally the organization is that first is non-regularized algorithms, next priors, then regularized algorithms and lastly miscellanious ones. Note that "regularized algorithm" refers to an algorithm that supports regularization, mainly gradient-based but it can also be proximal TV or TGV.
2. Next modify ``subiterStep.h`` and add the new algorithm to the end. While you can directly add all the required functions here, it is recommended to implement separate function for the algorithm. These are stored in ``algorithms.h`` and will be handled later
3. This step applies only if the algorithm supports regularization, or if the added algorithm is a regularizer. The ``applyPrior`` function in ``priors.h`` handles regularization. In general, the regularization is ADDED to the backprojection. There is a possibility to save the regularization itself and some algorithms already use that. That way you can input the regularization to variable other than the backprojection (or subtract it). If you are adding a new regularizer, you should add it to this function. If the gradient of the prior is ADDED to the backprojection in the algorithm, you don't need to do any modifications here!
4. Add the algorithm-specific operations to ``algorithms.h`` as a new function. Note that these need to be operations that are computed AFTER the backprojection. The backprojection is saved in ``std::vector`` ``vec.rhs_os``. Each ``std::vector`` element is one multi-resolution volume. If no multi-resolution reconstruction is used, then the vector only contains one element. Same applies to the current image estimate saved in ``vec.im_os``. The image estimate thus should be saved in ``vec.im_os``!
5. If you need to use helper variables specific to the algorithm, you can input them into ``AF_im_vectors_`` struct in ``structs.h``
6. Built-in preconditioning can be applied with ``applyImagePreconditioning``
7. Since we are proceeding backwards, next step is adding any computations before backprojection. This is applied in ``computeForwardProject.h`` (function ``computeForwardStep``). ``outputFP`` contains the forward projection. In ``computeForwardStep`` this is named as ``input`` while ``y`` is the measurement data! Note that you should save the result into ``input``.
8. Measurement-based preconditioners can be applied with ``applyMeasPreconditioning``. This should be done inside ``computeForwardStep``
9. Some algorithms require specific initialization phases. This can be simply preallocating memory for algorithm-specific variables or some initial computations. This can be done at each iteration or only at the first one. These operations can be done in ``initializationStep``-function. The "``if (curIter == 0) {``" line specifies section for operations that are done only at the first iteration (such as preallocation)
10. Next you'll need to copy necessary variables from either MATLAB/Python or from both. For MATLAB/Octave, you'll need to modify ``mfunctions.h`` and for Python ``libHeader.h``. The latter is also used in dynamic library settings. At minimum, you'll need to load the boolean variable for the algorithm, i.e. if it's true, the algorithm is used
11. The last step includes adding the name of the algorithm in ``recNames`` for MATLAB/Octave or the boolean in ``proj.py`` in Python. For MATLAB/Octave, general functionality should be guaranteed as long as ``varMAP`` (supports regularization) or ``OS`` (doesn't support regularization), ``varProj1``, and ``varPriorFull`` are modified (i.e. your one is added). Other functionality can be enabled with the other variables. Note that these are used merely in error checking, i.e. if you want to use preconditioners, you'll get an error when trying to use one if the algorithm is not added to the supported list. For Python, you'll only need to add an initial value for the algorithm. You'll also need to make sure that the boolean variable is correctly transfered to the C++-library. This means adding a new variable both to the list at the beginning of ``proj.py`` (``class projectorClass``) as well as to the list at the end (``class parameters``). The first is used for Python-specific tasks, while the latter is for the library. Lastly, the transfer of the boolean to the C++-library is done in ``recomain.py`` (``transferData``). For "``class parameters``" the order should be identical with ``libHeader.h``!
12. Depending on the algorithm, you might need to do some additional adjustments. If some variables need to be precomputed, you'll need to take care of this yourself. One possible location is the ``prepass_phase.m`` file for MATLAB/Octave and ``prepassPhase`` in ``prepass.py`` for Python.
13. In order to use your algorithm, you'll need to enable it with ``options.yourAlgorithm = true`` (or ``True`` in Python).


Debugging
---------

While increasing the verbosity level (``options.verbose``) to 2 or 3 can help with debugging, you can also enable a lot additional output variables by modifying ``structs.h`` and setting ``DEBUG true``, and recompiling
the code. This can help with debugging as you can see the values of many of the input variables.