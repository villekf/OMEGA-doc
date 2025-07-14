Loading GATE data
=================

GATE 9 and earlier PET data
---------------------------

When using GATE data (ASCII or ROOT, Python supports only ROOT) the only requirements are that any of the options that you have selected are actually present and that volume IDs have been stored. For example, if you wish to obtain the original 
source image you need to select all the source coordinates in the coincidence mask and trues require both event IDs as well as scatter numbers. It is also recommended to select the time stamp for the first single. More information on the necessary 
variables in the coincidence mask below.

Data produced through clusters, e.g. files with name1.dat/root, name2.dat/root, etc. do not need to be combined. Neither is there any need to adjust the time slice, i.e. it can be the same as the total duration as time step/slices can be adjusted freely 
as long as the time tag is included in the coincidence mask. Having a different time slice will cause all the data to be combined in the case of static reconstruction. ALL files present in the folder specified by ``options.fpath`` will be loaded and included. 
Different simulations should thus be in different folders.

OMEGA has been tested mainly with cylindricalPET system, but it does also work with CPET, ECAT and scanner (OPET is untested, but should also work). This, however, applies only to ASCII data and in the case of scanner, the number of levels should be five as 
in cylindricalPET. ROOT data, currently, only supports cylindricalPET and, most likely, the similar OPET. CPET, on the other hand, cannot be used without modifications as it uses continuous detectors. The only way to reconstruct CPET data is to use it in 
list-mode mode, see :doc:`customcoordinates` for details. This means that you need to extract the absorption coordinates (e.g. ``[coincidences, ~, ~, ~, ~, x, y, z] = load_data(options);``) and then input those coordinates to the options struct (e.g. ``options.x = x; options.y = y; options.z = z;``). 
This will enable a list-mode reconstruction. The reconstructed results may, however, be more unreliable than with other PET systems. Corrections are not available on CPET systems.

Coincidences are loaded from files including string "Coincidences" (ASCII) or from tree named Coincidences (ROOT). Delays are loaded similarly ("delay" and delay tree).

If you wish to easily view the GATE data in matrix form, ASCII is the recommended format. If you wish to use ROOT format, you can control the data saved with the coincidence mask just as with ASCII or binary 
(unsupported) data (undocumented in GATE), this also means that the coincidence mask of ASCII affects ROOT data as well.

Coincidence mask
^^^^^^^^^^^^^^^^

As mentioned above, the base requirement is that volume IDs are present for both singles.

In order to have any control on the time properties/dynamics, the first single should have its time stamp included. The time stamp is recommended in all cases, however. No tests have so far been performed in cases where no time stamps were selected. 
Time stamp for the second single is not necessary at the moment.

In order to store the original/true decay image, XYZ position of the source in world referential has to be selected for both singles.

Locations of the photon absorption points in the detectors can be obtained if XYZ position in the world referential is included for both singles.

Trues and scatter require event IDs as well as number of Compton and Rayleigh scatter events for both singles. Randoms require only the event IDs. For more information on storing trues, scatter and randoms see here.

Extracting GATE scatter, randoms and trues data from GATE 9 data
----------------------------------------------------------------

OMEGA allows the import of GATE scatter, randoms and trues data into MATLAB, Octave or Python either in the raw data format, as a sinogram or as a "true" image depicting the number of counts emanating from each coordinate (this is converted into same pixel 
resolution as the reconstructed image). All three components (trues, scatter and randoms) are stored separately along with the actual coincidence (prompts) data. The import can be done either by using ``PET_main_gateExample.m``, ``PET_main_gateExampleSimple.m``, or any 
scanner specific main-file. ``PET_main_gateExampleSimple.m`` supports only OSEM reconstruction. If you need only the data import, ``PET_main_gateExampleSimple.m`` is recommended for better readability. Note that Python supports ROOT only! Similar examples exist for Python.

Randoms are supported in two formats (ASCII, and ROOT, ASCII only in MATLAB/Octave). ASCII and ROOT support Compton scattering in the detector and phantom, Rayleigh scattering in the phantom and Rayleigh scattering in the detector. 
You can select any one of these in the main-files (``options.scatter_components``). For example ``options.scatter_components = [1 1 0 0];`` in MATLAB and ``options.scatter_components = np.array([True, True, False, False])`` in Python, stores the Compton
interactions separately in the scatter component. Both detector and phantom are included together.

.. note::

	Using ROOT data, as mentioned in readme, will cause MATLAB R2018b and EARLIER to crash during GUI activity. This can be prevented by using MATLAB in the -nojvm mode (i.e. matlab -nojvm), which means without any GUIs. It is recommended to use this 
	only for data extraction (set options.only_sinos = true and run PET_main_gateExampleSimple.m or other main-file with GATE support). This issue is not present on Octave or MATLAB R2019a and up. Python is not affected.

Extracting the trues, randoms and/or scatter has no effect on the actual coincidences (prompts). I.e. they will also be extracted same regardless if any of the trues, randoms or scatter is extracted.

All coincidences that are from different events (i.e. not from the same annihilation) will be considered as randoms. All coincidences that come from the same event but have scattered in at least one of the four possibilities are considered scatter. 
For trues, it is possible to control on which coincidences are considered trues. Randoms and Compton scattered events in the phantom are ALWAYS excluded from trues, but the other three are excluded ONLY if they are selected in ``options.scatter_components``. 
E.g. if ``options.scatter_components = [1 1 0 0]`` then Rayleigh scattered events are included in trues, but not in scattered events.

For scattered events, the scattering in the phantom takes precedence. For example, if an event has Compton scattered in the phantom and in the detector, it is included ONLY in the Compton scattered events in the phantom. 
Compton scattering also takes precedence over Rayleigh scattering. The order is thus Compton scattering in the phantom → Compton scattering in the detector → Rayleigh scattering in the phantom → Rayleigh scattering in the detector.

For scattered events, it is also possible to select only multiply scattered events. For example, only Compton scattered events in the phantom that have scatted twice or more can be included in the scatter data.

Usage
^^^^^

First block (SCANNER PROPERTIES) needs to be filled with the parameters corresponding to the scanner in question. Components computed from earlier elements (e.g. ``det_per_ring``) do not need to be filled (only ``PET_main_gateExample.m``).

The second block (titled "GATE SPECIFIC SETTINGS") allows the user to specify which elements to extract by setting the appropriate options-value to true (or 1).

Setting ``options.obtain_trues = true`` causes automatic extraction of trues. You can also (optionally) choose to reconstruct the trues instead of the actual coincidences. This is done by setting ``options.reconstruct_trues = true``. 
``options.scatter_components`` is used to control the events included in trues (see below). As mentioned above, randoms and Compton scattered events in the phantom are always excluded from trues.

``options.store_scatter = true`` allows the storing of scatter. However, in order to store scatter at least one element in the next vector needs to be 1. ``options.scatter_components`` stores the different scatter components as mentioned above. 
The first one is Compton scattering in the phantom, second Compton scattering in the detector, this Rayleigh scattering in the phantom and fourth Rayleigh scattering in the detector. 
E.g. setting ``options.scatter_components = [1 0 1 0]`` stores only the Compton and Rayleigh scattering in the phantom, while scatter in the detectors will be ignored and not included in the scatter data, it will be, however, included in trues if 
trues are stored. As with trues data, you can optionally choose to reconstruct the scatter data by setting ``options.reconstruct_scatter = true``. Only one of trues, scatter or prompt coincidences can be reconstructed at the same time.

Randoms can be obtained by putting ``options.store_randoms = true``. The randoms obtained like this will not be used for randoms correction if it is selected. Both the actual randoms and delayed coincidences (if selected in GATE) can be extracted 
at the same time and in separate variables.

The "true"(ground truth) image can be optionally stored as well by putting ``options.source = true``. This will create a separate mat-file named machine_name 'Ideal_image_coordinates' name '_ASCII.mat', where machine_name is the name of the 
scanner you’ve specified and name the name of the examination you’ve specified. The last elements of C contains the trues (e.g. ``C{end}``), RA contains randoms and SC scatter. Randoms and scatter are stored as singles in the true images.

Only ONE of the below output data can be used at a time.

If you intent to form sinograms as well, the SINOGRAM PROPERTIES block also needs to be filled with correct values.

Using ASCII data
^^^^^^^^^^^^^^^^

MATLAB/Octave only!

In order to extract scatter, randoms and/or trues from ASCII data you need to set ``options.use_ASCII = true`` in the ASCII DATA FORMAT SETTINGS block. Additionally you need to copy-paste the ASCII coincidence mask used in your macro. E.g. 
if ``/gate/output/ascii/setCoincidenceMask 0 1 0 1 1 1 1 0 0 0 0 1 1 1 1 1 0 0 0 1 0 1 1 1 1 0 0 0 0 1 1 1 1 1 0 0`` then ``options.coincidence_mask = [0 1 0 1 1 1 1 0 0 0 0 1 1 1 1 1 0 0 0 1 0 1 1 1 1 0 0 0 0 1 1 1 1 1 0 0];``.

If you are extracting trues, then ALL the scatter components need to be selected in the GATE coincidence mask before running the simulation.

The location of the ASCII .dat files is specified by ``options.fpath`` in MISC PROPERTIES. Alternatively, the current working directory in MATLAB can be used as well.
	
Using ROOT data
^^^^^^^^^^^^^^^

Simply set ``options.use_root = true``. The location of the ROOT .root files is specified by ``options.fpath`` in MISC PROPERTIES. Alternatively, the current working directory in MATLAB can be used as well.

You need to run ``install_mex`` or ``compile.py`` before ROOT support is available. If thisroot.sh/csh has been sourced, ROOT should be found automatically on Linux. Otherwise you can input the ROOT path with ``install_mex(0, [], [], [], '/PATH/TO/ROOT')`` and 
``compile.py -R /path/to/ROOT``.

.. note::

	Using ROOT data, as mentioned in readme, will cause MATLAB R2018b and EARLIER to crash during GUI activity. This can be prevented by using MATLAB in the -nojvm mode (i.e. matlab -nojvm), which means without any GUIs. It is recommended to use 
	this only for data extraction (set ``options.only_sinos = true`` and run PET_main_gateExampleSimple.m). This issue is not present on Octave or MATLAB R2019a and up. 

Loading and saving data
^^^^^^^^^^^^^^^^^^^^^^^

Sinograms are automatically created during data load regardless of the type of data used. Raw data is stored if options.store_raw_data = true. These are also automatically saved into a mat-file in the current working directory. If you are using TOF 
data, all the trues, scatter and randoms sinograms will be TOF as well.


Reconstruction
^^^^^^^^^^^^^^

If you wish to reconstruct any data, run the next section (Reconstructions). The selected data (trues, scatter, coincidences [default]) will be automatically selected.

If you want to reconstruct e.g. trues (without any scattered coincidences) + Compton scatter in phantom, you should load the saved sinogram/raw data and sum the trues and Compton scatter together (i.e. if ``options.scatter_components = [1 0 0 0]`` then 
``SinScatter`` contains only the Compton scatter in phantom and you can perform them reconstructions with the following code ``options.SinM = SinTrues + SinScatter``). If ``options.SinM`` already exists, it will not be loaded from the saved mat-files. That way you 
can input any data combinations, but unless you want exclusively trues or scatter, they need to be performed manually now. Note that in this example case you need to obtain 
the scatter data and trues data separately since the trues will include the other scattered components if the scatter components is ``options.scatter_components = [1 0 0 0]``, i.e. trues should be obtained with ``options.scatter_components = [1 1 1 1]``.

Currently the user also has to individually extract each scatter component (i.e. you can't extract Compton scatter in phantom or in detector simultaneously in separate variables/data files, but rather need to extract each component on its own and rename 
the output data accordingly).


GATE 10 PET data
----------------

With GATE 10, it is possible to combine GATE simulations and OMEGA reconstructions in the same Python script. See https://github.com/villekf/OMEGA/tree/master/source/Python/GATE_OMEGA_reconstruction.py for an example.