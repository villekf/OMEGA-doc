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

Several PET examples are available. For GATE data, there are three examples for MATLAB/Octave and two for Python. ``PET_main_gateExample.m`` is a generic GATE example while ``PET_main_gateExampleSimple.m`` is a simplified
version such that the number of adjustable parameters have been greatly reduced. For Python there is ``gate_PET.py`` and ``gate_PET_TOF.py``.

A more "generic" PET example, using Siemens Inveon PET data, is available in ``PET_main_genericExample.m`` and ``PET_main_genericExample.py``.

GATE users should use the gate_main.m file to reconstruct GATE PET data. For other PET data, the file you should start with is main_PET.m. For computing the forward and/or backward projections use forward_backward_projections_example.m. For custom (gradient-based) priors, use custom_prior_test_main.m. A more simplified main-file for GATE data (simple OSEM reconstruction) is available in gate_main_simple.m. Inveon users should use Inveon_PET_main.m while Biograph mCT data can be used with Biograph_mCT_main.m and Biograph Vision with Biograph_Vision_main.m.

A GATE example with GATE macros is available in exampleGATE-folder. Simply run the GATE macros as a GATE simulation (GATE material database needs to be in the same folder) and then run the gate_main_example-file to reconstruct the data. By default, ASCII data is used in the reconstruction. This example is based on both benchPET and the cylindrical PET example found from https://opengate.readthedocs.io/en/latest/defining_a_system_scanner_ct_pet_spect_optical.html#cylindricalpet

exampleGATE also contains macros for normalization.

When using GATE data, all the output files of the specified format will be read in the specified folder. E.g. if you select ASCII data, all .dat-files with Coincidences in the file name will be loaded from the specified folder, with LMF all .ccs files and with ROOT all .root files.

Example MAT-files for non-GATE situation (created from GATE data) can be found from: 
.. image:: https://zenodo.org/badge/DOI/10.5281/zenodo.3522199.svg
   Image at https://zenodo.org/badge/DOI/10.5281/zenodo.3522199.svg
   :target: https://doi.org/10.5281/zenodo.3522199

These files are based on the above GATE-example. Raw data obtained from the GATE macros mentioned above (not normalization) can be found from: 
.. image:: https://zenodo.org/badge/DOI/10.5281/zenodo.3526859.svg
   Image at https://zenodo.org/badge/DOI/10.5281/zenodo.3526859.svg
   :target: https://doi.org/10.5281/zenodo.3526859

Example Inveon data is available from: 
.. image:: https://zenodo.org/badge/DOI/10.5281/zenodo.3528056.svg
   Image at https://zenodo.org/badge/DOI/10.5281/zenodo.3528056.svg
   :target: https://doi.org/10.5281/zenodo.3528056 or 
   .. image:: https://zenodo.org/badge/DOI/10.5281/zenodo.4646897.svg
   Image at https://zenodo.org/badge/DOI/10.5281/zenodo.4646897.svg
   :target: https://doi.org/10.5281/zenodo.4646897. The PET_main_genericExample.m/py file can be used automatically for this data.

CT data
^^^^^^^

Several different example scripts for CT data are provided, for different kind of CT-scanners. These include preclinical CT and µCT scanners as well as GATE simulated (imageCT) data. When using CT data, the direction of rotation should be from top to bottom or from bottom to top. If you have data that is rotated from left to right/right to left, you should transpose (permute) it and adjust the xSize and ySize variables according to the final transposed dimensions. Currently also only cone/fan beam CT scanners with flat panel detectors are inherently supported. You can use also other type of scanners, but then you have to input the detector/source coordinates yourself (see e.g. Custom detector coordinates and/or list mode reconstruction_).

For GATE data, use gate_CT_main.m. An example of µCT (using either .. image::
https://zenodo.org/badge/DOI/10.5281/zenodo.4279613.svg
Image at https://zenodo.org/badge/DOI/10.5281/zenodo.4279613.svg
:target: https://doi.org/10.5281/zenodo.4279613 or .. image::
https://zenodo.org/badge/DOI/10.5281/zenodo.4279549.svg
Image at https://zenodo.org/badge/DOI/10.5281/zenodo.4279549.svg
:target: https://doi.org/10.5281/zenodo.4279549) is provided with the walnut_CT_main.m. A 2D (sinogram) example is shown in walnut2D_CT_main.m (uses .. image::
https://zenodo.org/badge/DOI/10.5281/zenodo.1254206.svg
Image at https://zenodo.org/badge/DOI/10.5281/zenodo.1254206.svg
:target: https://doi.org/10.5281/zenodo.1254206). Lastly, an example script using preclinical Inveon CT is in Inveon_CT_main.m (uses .. image::
https://zenodo.org/badge/DOI/10.5281/zenodo.4646835.svg
Image at https://zenodo.org/badge/DOI/10.5281/zenodo.4646835.svg
:target: https://doi.org/10.5281/zenodo.4646835). In all cases, the examples include both how to do built-in reconstruction or how to use the forward/backward projection class for your own custom algorithms.
Help

For a short PET tutorial in image reconstruction in OMEGA see Tutorial. For CT imaging, see CT tutorial.

For help on using the individual main-files or the various functions see Function help_.

If you want to extract GATE PET scatter, randoms and/or trues data to MATLAB see Extracting GATE scatter, randoms and trues data_.

For recommendations and things to watch out, see Useful information_.

Contact
-------

Currently it is recommended to ask questions in GitHub discussions_.

However, if you prefer using e-mail for contact, use the following address:

.. image:: https://github.com/villekf/OMEGA/blob/master/doc/contact.png