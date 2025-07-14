Examples
=====

.. contents:: Table of Contents

Installation
------------

See :doc:`installation` for install help.

General Information
-------------------

Most OMEGA features are built around m-file/py scripts referred to as "main-files" (which can be considered as examples). In these main files, a struct variable, called options, is filled and contains all the parameters that are needed for any of the operations selected. 
PET, CT and SPECT have quite different main-file compositions due to different properties of these imaging modalities. CT, for example, has much less adjustable parameters due to lack of corrections. Reconstruction parameters, however, 
are unchanged between different modalities. It is not necessary to use these main-files and not every variable present in the main-files is necessary. In the simplest form, only the source-detector coordinates, the size of the FOV
and the number of voxels are required.

Examples for MATLAB/Octave are contained in the main-files folder. For Python these are in ``/path/to/OMEGA/source/Python``. 

In general OMEGA uses units millimeter (mm) and seconds (s).

Currently the examples only include PET, CT and SPECT. However, as mentioned elsewhere, other type of data can also be used. The only requirement is that the data can be reconstructed by using ray-tracing methods in a voxel/pixel mesh.
For cases other than PET, CT or SPECT, I recommend reading :doc:`customcoordinates`.

PET data
--------

Several PET examples are available. For GATE data, there are three examples for MATLAB/Octave and Python. ``PET_main_gateExample.m`` is a generic GATE example, ``PET_main_gateTOFExample.m`` is a TOF example, 
while ``PET_main_gateExampleSimple.m`` is a simplified version such that the number of adjustable parameters have been greatly reduced. For Python there is ``gate_PET.py`` and ``gate_PET_TOF.py`` and ``gate_PET_simpleExample.py``.

A more "generic" PET example, using Siemens Inveon PET data, is available in ``PET_main_genericExample.m`` and ``PET_main_genericExample.py``. These showcase the generic use case for a cylindrical PET system.

For list-mode data, there is ``Inveon_PET_main_listmode_example``. Note that for list-mode data, there are no restrictions on the geometry of the scanner. The listmode example contains two different ways for the listmode
reconstruction. One uses the coordinates of each detector for each measurement and second uses an index-based reconstruction, where the transaxial and axial indices of each event are stored. These indices then correspond to
coordinates.

In case you want to reconstruct PET data from a non-cylindrical scanner, this is also possible. For that you'll need to manually either input the detector coordinates and the measurement data or transaxial and axial indices 
for each measurement that point to the coordinates. An example of coordinate-based case is 
shown in ``custom_detector_coordinates_example``. For coordinate based reconstruction, in general, you'll need 6 coordinates for one measurement, with the first three being the x/y/z-coordinates of the first detector, and the next three the x/y/z-coordinates 
of the second detector. In Python, the data has to be Fortran-ordered, or in vector format, such that the 6 coordinates are contiguously stored. For index-based, you'll need two transaxial indices and two axial indices, plus
the coordinates. Like with coordinates, the data needs to be Fortran-ordered in Python. The indices also need to be zero-based. The listmode example ``Inveon_PET_main_listmode_example`` shows both methods. Note that the data can be
in any format (sinogram, listmode, projection image, something else), as long as each measurement has the correct coordinates.

The above example files don't contain all the adjustable parameters. For a complete list of adjustable parameters, see ``main_PET_full``.

If you use large datasets, such as TOF data, you may want to limit the amount of measurements transfered to the GPU, when using GPU computing. This can be achieved by setting ``options.loadTOF = false``. In such a case
only the current subset is transfered to the GPU. This only works with subsets and will most likely slow down the computations. Default value is true, which means that all the measurement data is transfered to the GPU before
computations. Note that, despite the variable name, this can be applied to any data and not just TOF data.

Note that by default OMEGA assumes the FOV to be centered on the origin. You can, however, move the FOV location with ``options.oOffsetX/Y/Z`` variables (separate ones for each!). If you use your own detector coordinates, be
sure to take this into account.

If you want to use the OMEGA forward and backward projection operators to develop, for example, your own reconstruction algorithms, you can use the ``custom_algorithms_example`` files. 
For MATLAB/Octave there is only one example, but Python has two, one for OpenCL using Arrayfire and one for CUDA using PyTorch. They also use different scanners, as the MATLAB/Octave one uses the Inveon scanner, while
the Python one uses the GATE example scanner. However, you can use any scanner you wish or also simply the detector coordinates or indices as outlined above. For custom coordinates, see ``custom_coordinates_custom_algorithm_example``.

Example MAT-files created from GATE data can be found from: https://doi.org/10.5281/zenodo.12743218

Older raw ASCII data obtained from the GATE simulation (not normalization) can be found from: https://doi.org/10.5281/zenodo.3526859

Example preclinical Inveon data is available from: https://doi.org/10.5281/zenodo.3528056 or https://doi.org/10.5281/zenodo.4646897. The ``PET_main_genericExample.m/py`` or ``Inveon_PET_main_listmode_example`` 
files can be used automatically for this data.

CT data
--------

For CT, in general there are three different way to perform the reconstructions. One is largely an automatic version where the source/detector coordinates are computed by OMEGA. You can input offset values for the source and 
detector coordinates as well as for the FOV origin, but the coordinates themselves are computed by OMEGA. Second is a less automatic version where you can input the source coordinates and the coordinates for the center of the
detector panel, for each projection. In both cases you can input optional rotation of the detector panel or the direction vectors for each projection. In both cases, the projection angles are required. Third is the least automatic
where you can input all source/detector coordinates for each measurement, not just each projection, but for all measurements. This is, however, inefficient method and recommended only when other methods are not feasible. In general, 
you'll need 6 coordinates for one measurement, with the first three being the x/y/z-coordinates of the source, and the next three the x/y/z-coordinates 
of a single detector pixel. In Python, the data has to be Fourier ordered, or in vector format, such that the 6 coordinates are contiguously stored.

Several CT examples are available. For a rather generic case, see ``CT_main_generalExample`` which uses TIFF projection images as the input. This example automatically computes the source/detector coordinates 
and thus is applicable mainly to "typical" CBCT cases.

For a case using source coordinates and the center of the detector panel coordinates for each projection, see ``CBCT_main_generic`` files. These also highlight a case where the panel also rotates along its own axis (slightly).
Offset correction cases can also be used with this. Example data can be obtained from: https://doi.org/10.5281/zenodo.12722386

An example of µCT (using either https://doi.org/10.5281/zenodo.4279613 or https://doi.org/10.5281/zenodo.4279549) is provided with the ``walnut_CT_main`` though ``CT_main_generalExample`` works just as well. 
A 2D (sinogram) example is shown in ``CT2D_fanbeam_mainExample`` (uses https://doi.org/10.5281/zenodo.1254206). Lastly, an example script using preclinical Inveon CT is in ``Inveon_CT_main`` (uses https://doi.org/10.5281/zenodo.4646835). 

For high-dimensional µCT, you can use ``skyscan_CT_main_highDimExample`` or ``nikon_CT_main_highDimExample``. These are useful for datasets that are dozens of gigabytes large. They should also work straight for Skyscan or Nikon
µCT data. You can reconstruct such datasets at full resolution 
using a GPU even if the GPU does not have enough memory to hold all the data. Note that you will need a lot of physical RAM for these as the data is stored in the main memory, while only a subset of the data is stored in the GPU. The 
features are limited though as only FDK, PKMA and PDHG algorithms work. Regularization can be used, but it is highly suboptimal at the moment. Example SkyScan data can be obtained from: https://doi.org/10.5281/zenodo.12744181

For custom algorithms, see ``CT_main_generic_custom_algorithms_example`` or ``Planmeca_CT_main_generic_custom_algorithms``.

Note that in helical CT cases the curvature of the panel is NOT taken into account at the moment.

SPECT data
--------

There are examples included for Siemens Pro.specta and SIMIND data reconstruction. Reconstruction with other data requires the sinograms/projection data, the projection angles, radial distances between the panel center and FOV center, as well as the collimator geometry and detector intrinsic resolution.. Attenuation correction requires a 3D volume of linear attenuation coefficients, which should be aligned with the FOV of the reconstruction.

At the moment, only parallel hole collimators are supported, though pin-hole or coded aperture collimator might be possible with manual adjustment of detector coordinates (contact me if you are interested in trying out pin-hole or coded aperture reconstruction).

``SPECT_main_Siemens_Prospecta`` includes an example for two-head Siemens Pro.specta SPECT scanner (no data available at the moment). ``SPECT_main_simind_voxelbased`` contains a SIMIND-simulated test case with a link to the data.
There is also a ``SPECT_main`` example file, which loads Interfile SPECT data (no data available).

The Python version also includes examples for custom algorithm reconstructions. These are, however, based on the Siemens Pro.specta case and as such there is no open data available at the moment. For MATLAB/Octave custom reconstruction
might be possible with implementation 4 (CPU), but there are no examples at the moment. 

The SPECT examples are, in general, not as refined as the others mainly due to the lack of test data.

Contact
-------

Currently it is recommended to ask questions in GitHub `discussions <https://github.com/villekf/OMEGA/discussions>`_.

However, if you prefer using e-mail for contact, use the following address:

.. figure:: contact.png
   :scale: 100 %
   :alt: Contact e-mail