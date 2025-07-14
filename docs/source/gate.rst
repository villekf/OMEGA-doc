Loading GATE data
=================

GATE 9 and earlier
------------------

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

GATE 10
-------

With GATE 10, it is possible to combine GATE simulations and OMEGA reconstructions in the same Python script. See https://github.com/villekf/OMEGA/tree/master/source/Python/GATE_OMEGA_reconstruction.py for an example.