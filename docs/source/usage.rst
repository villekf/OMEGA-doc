Usage
=====

.. contents:: Table of Contents

Installation
------------

See :doc:`installation` for install help.

General Information
-------------------

Most OMEGA features are built around m-file scripts referred to as "main-files". In these main files, a struct variable, called options, is filled and contains all the parameters that are needed for any of the operations selected. 
PET, CT and SPECT have quite different main-file compositions due to different properties of these imaging modalities. CT, for example, has much less adjustable parameters due to lack of corrections. Reconstruction parameters, however, 
are unchanged between different modalities. It is not necessary to use these main-files and not every variable present in the main-files is necessary. 

Examples for MATLAB/Octave are contained in the main-files folders. For Python these are in /path/to/OMEGA/source/Python. 

Examples
--------

PET data
^^^^^^^^

Several PET examples are available. For GATE data, there are three examples for MATLAB/Octave and Python. ``PET_main_gateExample.m`` is a generic GATE example, ``PET_main_gateTOFExample.m`` is a TOF example, 
while ``PET_main_gateExampleSimple.m`` is a simplified version such that the number of adjustable parameters have been greatly reduced. For Python there is ``gate_PET.py`` and ``gate_PET_TOF.py`` and ``gate_PET_simpleExample.py``.

A more "generic" PET example, using Siemens Inveon PET data, is available in ``PET_main_genericExample.m`` and ``PET_main_genericExample.py``. These showcase the generic use case for a cylindrical PET system.

For list-mode data, there is ``Inveon_PET_main_listmode_example``. Note that for list-mode data, there are no restrictions on the geometry of the scanner.

In case you want to reconstruct PET data from a non-cylindrical scanner, this is also possible. For that you'll need to manually input the detector coordinates and the measurement data. An example of such a case is 
shown in ``custom_detector_coordinates_example``. In general, you'll need 6 coordinates for one measurement, with the first three being the x/y/z-coordinates of the first detector, and the next three the x/y/z-coordinates 
of the second detector. In Python, the data has to be Fourier ordered, or in vector format, such that the 6 coordinates are contiguously stored.

The above example files don't contain all the adjustable parameters. For a complete list of adjustable parameters, see ``main_PET_full``.

If you want to use the OMEGA forward and backward projedtion operators to develop, for example, your own reconstruction algorithms, you can use the ``custom_algorithms_example`` files. 
For MATLAB there is only one example, but Python has two, one for OpenCL using Arrayfire and one for CUDA using PyTorch. They also use different scanners, as the MATLAB one uses the Inveon scanner, while
the Python one uses the GATE example scanner. However, you can use any scanner you wish or also simply the detector coordinates.

Example MAT-files created from GATE data can be found from: https://doi.org/10.5281/zenodo.3522199

These files are based on the above GATE-example. Raw ASCII data obtained from the GATE macros mentioned above (not normalization) can be found from: https://doi.org/10.5281/zenodo.3526859

Example Inveon data is available from: https://doi.org/10.5281/zenodo.3528056 or https://doi.org/10.5281/zenodo.4646897. The ``PET_main_genericExample.m/py`` or ``Inveon_PET_main_listmode_example`` 
files can be used automatically for this data.

CT data
^^^^^^^

For CT, in general there are three different way to perform the reconstructions. One is largely an automatic version where the source/detector coordinates are computed by OMEGA. You can input offset values for the source and 
detector coordinates as well as for the FOV origin, but the coordinates themselves are computed by OMEGA. Second is a less automatic version where you can input the source coordinates and the coordinates for the center of the
detector panel, for each projection. In both cases you can input optional rotation of the detector panel or the direction vectors for each projection. In both cases, the projection angles are required. Third is the least automatic
where you can input all source/detector coordinates for each measurement, not just each projection, but for all measurements. This is, however, inefficient method and recommended only when other methods are not feasible. In general, 
you'll need 6 coordinates for one measurement, with the first three being the x/y/z-coordinates of the source, and the next three the x/y/z-coordinates 
of a single detector pixel. In Python, the data has to be Fourier ordered, or in vector format, such that the 6 coordinates are contiguously stored.

Several CT examples are available. For a rather generic case, see ``CT_main_generalExample`` which uses TIFF projection images as the input. This example automatically computes the source/detector coordinates 
and thus is applicable mainly to "typical" CBCT cases.

For GATE data, use gate_CT_main.m. An example of µCT (using either https://doi.org/10.5281/zenodo.4279613 or https://doi.org/10.5281/zenodo.4279549) is provided with the walnut_CT_main.m. 
A 2D (sinogram) example is shown in walnut2D_CT_main.m (uses https://doi.org/10.5281/zenodo.1254206). Lastly, an example script using preclinical Inveon CT is in Inveon_CT_main.m (uses https://doi.org/10.5281/zenodo.4646835). 
In all cases, the examples include both how to do built-in reconstruction or how to use the forward/backward projection class for your own custom algorithms.

SPECT data
^^^^^^^^^^

A couple of SPECT examples are available. 

Contact
-------

Currently it is recommended to ask questions in GitHub `discussions <https://github.com/villekf/OMEGA/discussions>`_.

However, if you prefer using e-mail for contact, use the following address:

.. image:: https://github.com/villekf/OMEGA/blob/master/doc/contact.png