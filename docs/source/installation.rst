Installation
============

.. contents:: Table of Contents
   :depth: 15
   :local:

As a general rule, if you want to use `built-in algorithms <https://omega-doc.readthedocs.io/en/latest/reconstruction.html#built-in-reconstruction>`_, you MUST also install ArrayFire and either OpenCL or CUDA!

The general steps to install OMEGA are as follows:

MATLAB/Octave
-------------

* Install a C++-compiler if one is not already installed
* (Optional) Install OpenCL, ArrayFire, ROOT, drivers/runtimes, OpenMP and/or CUDA
* Obtain OMEGA and add the necessary folders to the MATLAB/Octave path
* Run ``install_mex`` in MATLAB or Octave

Python
------

* Install with ``pip install omegatomo``
* (Optional) Install OpenCL, ArrayFire, ROOT, drivers/runtimes, OpenMP and/or CUDA

Windows
-------

This section specifies the necessary installation information when using OMEGA on Windows.

MATLAB
^^^^^^

Ignore this section if you intend to only use Python or Octave.

Simply download and install MATLAB for Windows. Any version from 2009b and up should work, but the latest version is always recommended. Some features require 2016b so it is recommended to use at least that. If you are using an older version, 
however, you should use 64-bit version as 32-bit version is not supported.

Image processing toolbox and parallel computing toolbox are used for some specific features. However, (slower) fallback alternatives are used if these toolboxes are not found in most cases. The only function that requires image processing toolbox 
is the anisotropic diffusion MRP prior when using either implementation 1 or 4. Due to this, it does not work in Octave. A warning is produced if AD-MRP prior is used without the toolbox. Parallel computing toolbox is used, if available, when using 
arc correction.

Octave
^^^^^^

Ignore this section if you intend to only use Python or MATLAB.

Simply download the `binary installer for Octave <https://www.gnu.org/software/octave/download.html#ms-windows>`_ and install the software (Windows-64, though the large data version should work as well). io, image, and statistics packages are 
required for some features, but are easy to install with the following commands in the Octave user interface: 

``pkg install -forge io``

``pkg install -forge image``

``pkg install -forge statistics``

after which you need to load the statistics and image packages

``pkg load statistics``

``pkg load image``

Anisotropic diffusion MRP prior when using either implementation 1 or 4 does not work on Octave. A warning is produced if AD-MRP prior is used.

Python
^^^^^^

Ignore this section if you intend to only use Octave or MATLAB.

You need to have Python installed. Any version from 3.6 and up should work, though most likely earlier versions work also. Note that Python 3.12 and newer haven't been tested! You can install Python either manually or from the Microsoft Store.

There are two ways you can install OMEGA itself for Python: through pip or manually.

pip install
***********

If using pip, simply use ``pip install omegatomo``. Note that examples are not included in this case and prebuilt library files are included. 

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``pyopencl`` and/or ``cupy`` packages installed. If you want to interface OMEGA with PyTorch, you'll need ``torch`` installed too or if you simply want to use PyTorch instead
of CuPy.

Manual install
**************

You'll need to add ``C:\path\to\OMEGA\source\Python`` to PYTHONPATH (you can do this easily in Spyder in Tools --> PYTHONPATH manager). The only required package is NumPy (``numpy``, version 1.20 or higher). ``scikit-image`` is required if you use  
binning and in some cases when using extended FOV.
``pymatreader`` is required in order to load mat-files, this is mainly for precomputed data, such as example data used by OMEGA examples. ``SimpleITK`` is 
required to load MetaImage-files, this is mainly for PET such as GATE attenuation images. ``arrayfire`` is highly recommended, as it allows to display device info. All packages can be installed through ``pip`` or ``conda``, e.g. ``pip install arrayfire``.

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``arrayfire`` and ``pyopencl`` or ``cupy`` and potentially also ``torch``. If you want to interface OMEGA with PyTorch, you'll need ``torch`` installed too or if you simply want 
to use PyTorch instead of CuPy.

Note that with ``pymatreader``, you can load measurement data from mat-files, which is useful when running the examples as many of them utilize the precomputed mat-files. MATLAB or Octave is NOT required.
The benefit of using ``pymatreader`` instead of SciPy is that ``pymatreader`` supports both v7 and v7.3 mat-files. SciPy only supports v7 mat-files.

Other notes
***********

If you want to load ROOT data, you'll need to make sure that PyROOT is in PYTHONPATH.

If you want to compute your own algorithms with OpenCL using Arrayfire, take into account this issue: https://github.com/arrayfire/arrayfire-python/issues/265 and this as well if you use CUDA device: https://github.com/arrayfire/arrayfire-python/issues/267

C++ Compiler
^^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

Compiled binaries are provided, but they might not work on all systems/configurations. In case they do not work, you'll need to compile the code yourself.

You need a C++11 compatible compiler to compile the necessary MEX-files on MATLAB/Octave or dll-files for Python. For a list of supported compilers for each version of MATLAB, see https://www.mathworks.com/support/requirements/previous-releases.html. 
On Octave the built-in compiler is sufficient (implementation 2 is not supported, unless you build it manually yourself, see Arrayfire section below).

On MATLAB and Python, however, `Visual Studio <https://visualstudio.microsoft.com/downloads/>`_ is highly recommended. Depending on your MATLAB version, you might need an `older version <https://visualstudio.microsoft.com/vs/older-downloads/>`_. 
To configure the C++ MEX-compiler you can use the command ``mex -setup C++`` which will show the currently selected compiler, list all available compilers and show the necessary commands to switch the compiler. 

On Python, any version of Visual Studio should suffice.

If you use Visual studio, OMEGA only requires "Desktop development with C++". No additional features need to be installed. Alternatively, you can try installing VS Build Tools, for example: https://aka.ms/vs/17/release/vs_BuildTools.exe. Note that the use
of the build tools hasn't been tested.

MATLAB allows the use of `MinGW++ <https://se.mathworks.com/matlabcentral/fileexchange/52848-matlab-support-for-mingw-w64-c-c-compiler>`_, but when using this compiler ArrayFire (implementation 2) will not work unless ArrayFire is built from the source with 
MinGW. See the ArrayFire section below for details.

See OMEGA section below on how to compile the code in all cases.

OpenCL/CUDA
^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

If you have a Nvidia GPU, then `CUDA toolkit <https://developer.nvidia.com/cuda-downloads>`_ is recommended. Both Development and Runtime libraries are required, especially if CUDA support is desired.

If you have Intel GPU/CPU, you should use the `Intel OpenCL SDK <https://software.intel.com/content/www/us/en/develop/tools/opencl-sdk.html>`_. When using a CPU you might need to install the `runtimes <https://software.intel.com/content/www/us/en/develop/articles/opencl-drivers.html>`_ as well. 

If you are using AMD GPUs, it is recommended to download and install `OCL-SDK <https://github.com/GPUOpen-LibrariesAndSDKs/OCL-SDK/releases>`_. Additionally, you should install official `drivers <https://www.amd.com/en/support>`_.

For AMD CPUs you should first install the `OCL-SDK <https://github.com/GPUOpen-LibrariesAndSDKs/OCL-SDK/releases>`_ and then either install the Intel CPU runtimes from above or install `POCL <https://github.com/pocl/pocl/releases>`_.

Note that if you are using Octave and want to use implementation 2, there are some special requirements for OpenCL location. See the ArrayFire section below for a link to the guide page.

ArrayFire
^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

*These instructions are for MATLAB and Python ONLY (when using Visual Studio):*

On Windows simply download the Windows binary from https://arrayfire.com/download/ and install it. For more help on installing ArrayFire on Windows see http://arrayfire.org/docs/installing.htm#Windows. Make sure you add Arrayfire to PATH!

*These instructions are for Octave and for MATLAB when using Mingw-w64:*

You'll need to build ArrayFire manually in order to get it to work. Furthermore, only OpenCL is supported. For details see https://github.com/villekf/OMEGA/wiki/Building-ArrayFire-with-Mingw-on-Windows.

Make sure you add ArrayFire to PATH! The current user is fine if no other user uses OMEGA on the same computer.

OMEGA
^^^^^

Python pip install
******************

The pip install doesn't include examples so you need to manually download those: https://github.com/villekf/OMEGA/tree/master/source/Python

With the pip version, you shouldn't need to compile anything. However, if you run into issues with the precompiled files, you can manually compile everything too. For this you need to first ``from omegatomo.util.compile import compileOMEGA`` and then run
``compileOMEGA()``. You can insert ArrayFire path with ``compileOMEGA('-A /path/to/arrayfire')`` and ROOT path with ``compileOMEGA('-R /path/to/ROOT')``. This can be done in an interactive session and you can add both paths in the same run.

Other cases
***********

Download either a release version from `releases <https://github.com/villekf/OMEGA/releases>`_, clone the current master with e.g. `GitHub desktop <https://desktop.github.com/>`_ or download an archive of the 
`master-branch <https://github.com/villekf/OMEGA/archive/master.zip>`_. If you downloaded either a release or master branch archive, you need to extract the contents to the folder of your choosing. 

MATLAB/Octave
+++++++++++++

Alternatively, if you are using MATLAB, you can download the mltbx package (`OMEGA.-.Open-source.MATLAB.emission.tomography.software.mltbx`) from the `releases <https://github.com/villekf/OMEGA/releases>`_ and simply run 
it in which case all the necessary folders will be automatically added to the MATLAB path.

Unless the MATLAB package was used, you need to add the source and mat-files folders to the MATLAB/Octave path (biograph-folder should be added if you intend to use mCT or Vision list-mode data files). 
In MATLAB you can do this by simply right clicking the folders and selecting "Add to path -> Selected folders" by selecting the OMEGA folder itself and selecting "Add to path -> Selected folders and subfolders". 
Alternatively, if you are using for example Octave, you can add the paths with ``addpath('C:\path\to\OMEGA\source')`` and ``addpath('C:\path\to\OMEGA\mat-files')`` or simply with ``addpath(genpath('C:\path\to\OMEGA\'))``. 
On MATLAB you can also add these folders to the list of folders in "Set path".

To build all the necessary mex-files, simply run ``install_mex``.

In case you have trouble compiling the mex-files, you can also try using the precompiled files on the `releases <https://github.com/villekf/OMEGA/releases>`_ page.

Python
++++++

Below part can be ignored if you only use the custom reconstruction in Python or if you use only MATLAB/Octave. Also, this only applies when you are not using the pip version.

For Python, it is highly recommended to use Visual Studio as the C++ compiler! User either Windows command prompt or Powershell. In the command prompt/shell, navigate to ``C:\path\to\OMEGA\source\Python`` and then run ``python3 compile.py`` 
or ``python compile.py``. or ``py compile.py``. If ArrayFire was installed somewhere other than Program files, you'll need to specify its location with ``python3 compile.py -A C:\path\to\Arrayfire\v3``. For ROOT, similarly with 
``python3 compile.py -R C:\path\to\root``. By default, the script should find a Visual Studio install if you are using either 2022, 2019 or 2017 releases. But if the script fails to find Visual Studio, you can also do the compilation 
by using "x64 Native Tools Command Prompt for VS 2022" (or 2019 or any other Visual studio version) from the Windows start menu. The process is otherwise identical.

Linux
-----

This section specifies the necessary installation information when using OMEGA on Linux distributions.

MATLAB
^^^^^^

Ignore this section if you intend to only use Python.

Simply download and install MATLAB for Linux. Any version from 2009b and up should work, but the latest version is always recommended. Some features require 2016b so it is recommended to use at least that. If you are using an older version, 
however, you should use 64-bit version as 32-bit version is not supported.

Image processing toolbox and parallel computing toolbox are used for some specific features. However, (slower) fallback alternatives are used if these toolboxes are not found in most cases. The only function that requires image processing toolbox is 
the anisotropic diffusion MRP prior when using either implementation 1 or 4. Due to this, it does not work on Octave. A warning is produced if AD-MRP prior is used without the toolbox. Parallel computing toolbox is used, if available, when using arc correction.

Octave
^^^^^^

Ignore this section if you intend to only use Python or MATLAB.

There are several different ways to install Octave on Linux systems. For instructions on how to install Octave on variety of Linux distributions see the `Octave wiki <https://wiki.octave.org/Category:Installation>`_. You also need to install the Octave 
development files (e.g. ``liboctave-dev`` on Debian/Ubuntu). Alternatively, you can use `distribution independent <https://wiki.octave.org/Octave_for_GNU/Linux#Distribution_independent>`_ methods or just `build from source <https://wiki.octave.org/Building>`_.

io, image and statistics packages are required for some features, but are easy to install with the following commands in the Octave user interface: 

``pkg install -forge io``

``pkg install -forge image``

``pkg install -forge statistics``

after which you need to load the statistics and image packages

``pkg load statistics``

``pkg load image``

Anisotropic diffusion MRP prior when using either implementation 1 or 4 does not work on Octave. A warning is produced if AD-MRP prior is used.

Python
^^^^^^

Ignore this section if you intend to only use Octave or MATLAB.

You need to have Python installed. Any version from 3.6 and up should work, though most likely earlier versions work also. Note that Python 3.12 and newer haven't been tested! You should install Python using your the package manager of your distro,
e.g. ``sudo apt install python``, though often some version should be preinstalled.

There are two ways you can install OMEGA itself for Python: through pip or manually.

pip install
***********

If using pip, simply use ``pip install omegatomo``. Note that examples are not included in this case and prebuilt library files are included. 

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``pyopencl`` and/or ``cupy``. If you want to interface OMEGA with PyTorch, you'll need ``torch`` installed too or if you simply want to use PyTorch instead
of CuPy.

Manual install
**************

You'll need to add ``/path/to/OMEGA/source/Python`` to PYTHONPATH (you can do this easilly in Spyder in Tools --> PYTHONPATH manager). The only required package is NumPy (``numpy``, version 1.20 or higher). ``scikit-image`` is required if you use binning
and in some cases when using extended FOV.
``pymatreader`` is required in order to load mat-files, this is mainly for precomputed data, such as example data used by OMEGA examples. ``SimpleITK`` is 
required to load MetaImage-files, this is mainly for PET such as GATE attenuation images. ``arrayfire`` is highly recommended, as it allows to display device info. All packages can be installed through ``pip`` or ``conda``, e.g. ``pip install arrayfire``.

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``arrayfire`` and ``pyopencl`` or ``cupy`` and potentially also ``torch``. If you want to interface OMEGA with PyTorch, you'll need ``torch`` installed too or if you simply want 
to use PyTorch instead of CuPy.

Note that with ``pymatreader``, you can load measurement data from mat-files, which is useful when running the examples as many of them utilize the precomputed mat-files. MATLAB or Octave is NOT required.
The benefit of using ``pymatreader`` instead of SciPy is that ``pymatreader`` supports both v7 and v7.3 mat-files. SciPy only supports v7 mat-files.

Other notes
***********

If you want to load ROOT data, you'll need to make sure that PyROOT is in PYTHONPATH.

If you want to compute your own algorithms with OpenCL using Arrayfire, take into account this issue: https://github.com/arrayfire/arrayfire-python/issues/265 and this as well if you use CUDA device: https://github.com/arrayfire/arrayfire-python/issues/267

C++ Compiler
^^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python. If the precompiled binaries work, you won't need to build anything.

A C++ compiler should already be included, but gcc/g++ is recommended. Any version 4.7 or up should be sufficient. It is recommended to use the g++ version supported by your MATLAB version whenever possible, when using MATLAB, 
though newer versions should work almost all the time. Some combinations of MATLAB and g++, however, will lead to errors. See OMEGA section below for more details. List of supported compilers is available 
at https://www.mathworks.com/support/requirements/previous-releases.html.

Octave should be fine in all cases.

For Python, g++ is required. Version should not matter.

On Ubuntu, you can install g++ with e.g. ``sudo apt install build-essential``.

See OMEGA section below on how to compile the code.

OpenCL/CUDA
^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

If you are using any GPU on Linux, it should be sufficient to simply download the OpenCL libraries and headers

Debian/Ubuntu: ``sudo apt-get install ocl-icd-opencl-dev opencl-headers ocl-icd-libopencl1``

as well as the official drivers.

Alternatively, if you have a Nvidia GPU, then `CUDA toolkit <https://developer.nvidia.com/cuda-downloads>`_ can be used. Both Development and Runtime libraries are required, especially if CUDA support is desired.

AMD GPUs should work with only the drivers. If that doesn't work, you can try using `ROCm OpenCL runtimes <https://github.com/RadeonOpenCompute/ROCm-OpenCL-Runtime/tree/roc-3.3.0>`_.

If you have Intel GPU/CPU, you can use the `Intel OpenCL SDK <https://software.intel.com/content/www/us/en/develop/tools/opencl-sdk.html>`_. When using a CPU you might need to install the `runtimes <https://software.intel.com/content/www/us/en/develop/articles/opencl-drivers.html>`_ as well. The runtimes, however, might not anymore support your current OS version.

Alternatively, and especially when using AMD CPUs, `POCL <http://portablecl.org/docs/html/install.html>`_ is recommended (`download <http://portablecl.org/download.html>`_). Note that if you use the default installation path, you need to 
move `/usr/local/etc/OpenCL/vendors/pocl.icd` to `/etc/OpenCL/vendors/`.

A useful, but not necessary, program is `clinfo <https://github.com/Oblomov/clinfo>`_ that should be available as a package (e.g. ``sudo apt-get install clinfo``). clinfo displays all the available OpenCL platforms, 
the devices available and various other features. A short list of OpenCL platforms and devices can be obtained in OMEGA with the ``OpenCL_device_info()`` function in MATLAB/Otave or with ``deviceInfo()`` in Python 
(after ``from omegatomo.util.devinfo import deviceInfo``).

ArrayFire
^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

Simply download the Linux binary from `ArrayFire <https://arrayfire.com/download/>`_ and install it. For more help on installing ArrayFire on Linux see `here <http://arrayfire.org/docs/installing.htm#Linux>`_. Note, however, that, if you are using the 
official binary, and you want a simple install of OMEGA, you should install ArrayFire to the default location in ``/opt``. Secondly, if you are using MATLAB and/or Octave, you should rename, or simply delete if you are not using ArrayFire's graphic features (not used in OMEGA), all 
the ``libforge`` files in ``/opt/arrayfire/lib64`` to something else (e.g. ``libforge.so.old``). Alternatively, you can use a "`no-GL <http://arrayfire.s3.amazonaws.com/3.6.2/ArrayFire-no-gl-v3.6.2_Linux_x86_64.sh>`_" version, but it is an older 
version that should, nevertheless, work. Leaving the ``libforge.so`` files with their original names will most likely lead to crashes as of AF 3.9.0 and earlier (except the no-gl versions) when using MATLAB or Octave.

Alternatively, you can `build from source <https://github.com/arrayfire/arrayfire/wiki/Build-Instructions-for-Linux>`_. If you are building ArrayFire from source, it is recommended to disable Forge (set ``AF_BUILD_FORGE`` to ``OFF``), otherwise you might get 
unstable behavior.

Make sure you add ``/path/to/arrayfire/lib64`` (or ``/lib`` if you built from source) to ``LD_LIBRARY_PATH``! If you complete the instructions above and have sudo permission, you're fine. Otherwise, if you lack sudo permission you can add the library 
path with ``export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/path/to/arrayfire/lib64`` on Linux terminal. Note that if you want to avoid typing it every time you open a terminal, you need to add it to .bashrc, .profile or something similar.

OMEGA
^^^^^

Python pip install
******************

The pip install doesn't include examples so you need to manually download those: https://github.com/villekf/OMEGA/tree/master/source/Python

With the pip version, you shouldn't need to compile anything. However, if you run into issues with the precompiled files, you can manually compile everything too. For this you need to first ``from omegatomo.util.compile import compileOMEGA`` and then run
``compileOMEGA()``. You can insert ArrayFire path with ``compileOMEGA('-A /path/to/arrayfire')`` and ROOT path with ``compileOMEGA('-R /path/to/ROOT')``. This can be done in an interactive session and you can input both paths at the same time.

Other cases
***********

Download either a release version from `releases <https://github.com/villekf/OMEGA/releases>`_, clone the current master with e.g. `git clone https://github.com/villekf/OMEGA.git` or download an archive of the 
`master-branch <https://github.com/villekf/OMEGA/archive/master.zip>`_. If you downloaded either a release or master branch archive, you need to extract the contents to the folder of your choosing. 

MATLAB/Octave
+++++++++++++

Alternatively, if you are using MATLAB, you can download the mltbx package (``OMEGA.-.Open-source.MATLAB.emission.tomography.software.mltbx``) from the `releases <https://github.com/villekf/OMEGA/releases>`_ and simply run it in which case all the 
necessary folders will be automatically added to the MATLAB path.

Unless the MATLAB package was used, you need to add the source and mat-files folders to the MATLAB/Octave path (biograph-folder should be added if you intend to use mCT or Vision list-mode data files). In MATLAB you can do this by simply right clicking the 
folders and selecting "Add to path -> Selected folders" by selecting the OMEGA folder itself and selecting "Add to path -> Selected folders and subfolders". Alternatively, if you are using for example Octave, you can add the paths 
with ``addpath('/path/to/OMEGA/source')`` and ``addpath('/path/to/OMEGA/mat-files')`` or simply with ``addpath(genpath('/path/to/OMEGA/'))``. On MATLAB you can also add these folders to the list of folders in "Set path".

To build all the necessary mex-files, simply run ``install_mex``. If ArrayFire was installed in some non-standard folder, the compilation might not work unless you include the folder to ``install_mex``. This can be done with
``install_mex(0, [], [], '/path/to/Arrayfire')``. See ``help install_mex`` for more details.

In case you have trouble compiling the mex-files or the library-files, you can also try using the precompiled files on the `releases <https://github.com/villekf/OMEGA/releases>`_ page.

*MATLAB troubleshooting*

If you are using MATLAB R2017b or EARLIER, you will most likely encounter problems when running the mex-files. The same can also happen if you use the latest gcc/g++ with MATLAB 2020a or earlier. One alternative is to install the supported compiler 
of the MATLAB version in use (see `here <https://www.mathworks.com/support/requirements/previous-releases.html>`_) and then re-run ``install_mex`` (the supported compiler is used if available). Alternatively, you can try one of solutions 
presented `here <https://www.mathworks.com/matlabcentral/answers/329796-issue-with-libstdc-so-6>`_ or try the precompiled mex-files from `releases <https://github.com/villekf/OMEGA/releases>`_. In short there are mainly three possibilities:

1. Install the compiler that MATLAB supports. If you are using, for example, Ubuntu 20, you can install older g++ as outlined `here <https://askubuntu.com/questions/1229774/how-to-use-an-older-version-of-gcc>`_. Note that you need to install 
g++ (e.g. ``sudo apt install g++-6``). If you are using R2017b or earlier, see `here <https://askubuntu.com/questions/1036108/install-gcc-4-9-at-ubuntu-18-04>`_. Then simply re-run ``install_mex``.

2. Locate the system version of libstdc++.so.6 and create an alias in .bashrc for MATLAB to use this one, for example:
``alias matlab='LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libstdc++.so.6 /path/to/MATLAB/bin/matlab -desktop'`` and then run MATLAB in terminal with the command ``matlab``. Or simply run MATLAB with the same ``LD_PRELOAD``.

3. Rename the libstdc++.so.6 file that ships with MATLAB, located in ``/path/to/MATLAB/sys/os/glnxa64/``
e.g. ``sudo mv /path/to/MATLAB/sys/os/glnxa64/libstdc++.so.6 /path/to/MATLAB/sys/os/glnxa64/libstdc++.so.6.old``. 

*ROOT support*

When importing ROOT data, you might run into errors (the crashes with R2018b and earlier can be fixed by running MATLAB with ``matlab -nojvm``, however, errors can still occur after this). These occur if you are using ROOT 6.16 or later and are using 
MATLAB (Octave and Python are unaffected). R2020b (and probably newer ones later) is unaffected. These errors can be fixed by similar methods as above with two additional possibilities: 

1. Locate the ROOT version of libtbb.so.2 and create an alias in .bashrc for MATLAB to use this one, for example:
``alias matlab='LD_PRELOAD=/opt/root/lib/libtbb.so.2 /path/to/MATLAB/bin/matlab -desktop'`` and then run MATLAB in terminal with the command ``matlab``.. Or simply run MATLAB with the same ``LD_PRELOAD``.

2. Rename the libtbb.so.2 file that ships with MATLAB, located in ``/path/to/MATLAB/bin/glnxa64/``
e.g. ``sudo mv /path/to/MATLAB/bin/glnxa64/libtbb.so.2 /path/to/MATLAB/bin/glnxa64/libtbb.so.2.old``. This is not recommended if the system is used by other users who use the same MATLAB.

3. Install ROOT 6.14 or earlier.

4. Use Octave or Python for ROOT data import.

Python
++++++

In Python, add ``/path/to/OMEGA/source/Python`` to PYTHONPATH.

The below compilation is not required if you only use custom reconstruction in Python.

In Python, navigate to ``/path/to/OMEGA/source/Python`` in terminal and run ``python compile.py`` (or ``python3 compile.py``) to compile the library files. If ArrayFire was not installed in ``opt`` add the path with ``python compile.py -A /path/to/arrayfire``.
For ROOT, the path can be added with ``python compile.py -R /path/to/root``.

Mac
---

This section specifies the necessary installation information when using OMEGA on MacOS.

.. note::

   Mac build of OMEGA hasn't been tested so far. Compilation has been tested on MATLAB ONLY.

MATLAB
^^^^^^

Ignore this section if you intend to only use Python.

Simply download and install MATLAB for Mac. Any version from 2009b and up should work, but the latest version is always recommended. Some features require 2016b. If you are using an older version, however, you should use 64-bit version as 32-bit version is 
not supported.

Image processing toolbox and parallel computing toolbox are used for some specific features. However, (slower) fallback alternatives are used if these toolboxes are not found in most cases. The only function that requires image processing toolbox is the 
anisotropic diffusion MRP prior when using either implementation 1 or 4. Due to this, it does not work on Octave. A warning is produced if AD-MRP prior is used without the toolbox. Parallel computing toolbox is used, if available, when using arc correction.

Octave
^^^^^^

Ignore this section if you intend to only use Python or MATLAB.

To install Octave on Mac, see their `wiki <https://wiki.octave.org/Octave_for_macOS>`_ for instructions.

io, image and statistics packages are required for some features, but are easy to install with the following commands in the Octave user interface: 

``pkg install -forge io``

``pkg install -forge image``

``pkg install -forge statistics``

after which you need to load the statistics and image packages

``pkg load statistics``

``pkg load image``

Anisotropic diffusion MRP prior when using either implementation 1 or 4 does not work on Octave. A warning is produced if AD-MRP prior is used.

Python
^^^^^^

Ignore this section if you intend to only use Octave or MATLAB.

You need to have Python installed. Any version from 3.6 and up should work, though most likely earlier versions work also. Note that Python 3.12 hasn't been tested! You should install Python using your the package manager of your distro,
e.g. ``sudo apt install python``, though often some version should be preinstalled.

There are two ways you can install OMEGA itself for Python: through pip or manually.

pip install
***********

If using pip, simply use ``pip install omegatomo``. Note that examples are not included in this case and prebuilt library files are included. Note that the prebuilt binaries included with pip do NOT include Mac versions! You'll need to manually
compile the code! See OMEGA and Python pip install below for details!

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``pyopencl`` and/or ``cupy`` and ``torch`` packages installed.

Manual install
**************

You'll need to add ``/path/to/OMEGA/source/Python`` to PYTHONPATH. The only required package is NumPy (``numpy``, version 1.20 or higher). ``scikit-image`` is required if you use extended FOV or binning.
``pymatreader`` is required in order to load mat-files, this is mainly for precomputed data, such as example data used by OMEGA examples. ``SimpleITK`` is 
required to load MetaImage-files, this is mainly for PET such as GATE attenuation images. ``arrayfire`` is highly recommended, as it allows to display device info.

Furthermore, if you want to use the custom algorithm reconstruction, you'll need ``arrayfire`` and ``pyopencl`` or ``cupy`` and ``torch``. All packages can be installed through ``pip`` or ``conda``.

Note that with ``pymatreader``, you can load measurement data from mat-files, which is useful when running the examples as many of them utilize the precomputed mat-files. MATLAB and/or Octave is NOT required.
The benefit of using ``pymatreader`` instead of SciPy is that ``pymatreader`` supports both v7 and v7.3 mat-files. SciPy only supports v7 mat-files.

Other notes
***********

If you want to load ROOT data, you'll need to make sure that PyROOT is in PYTHONPATH.

If you want to compute your own algorithms with OpenCL using Arrayfire, take into account this issue: https://github.com/arrayfire/arrayfire-python/issues/265 and this as well if you use CUDA device: https://github.com/arrayfire/arrayfire-python/issues/267

C++ Compiler
^^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

You should install `Xcode <https://apps.apple.com/us/app/xcode/id497799835?mt=12>`_ from the app store. Furthermore, if you wish to use implementations 1 and/or 4 with OpenMP (parallel computing) support, you might need to install OpenMP. This is most 
easily achieved with Homebrew:

``brew install libomp``

On MATLAB, you do not need to do any changes. On Octave, you need to make sure that both the library and header (`omp.h`) can be found on path. This might also be the case on MATLAB if the header is installed in non-standard location. If OpenMP support 
could NOT be applied, you should see a warning message(s) of the like `...built WITHOUT OpenMP (parallel) support.` when compiling the code.

OpenCL/CUDA
^^^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python.

OpenCL should already be included with your Mac installation or then it is most likely not supported at all. If running OpenCL functions fails, make sure that ``/System/Library/Frameworks/OpenCL.framework`` is included 
in the library path.

CUDA is not supported in Mac.

ArrayFire
^^^^^^^^^

Ignore this section if you intend to only use custom reconstructions in Python with non-SPECT data.

Simply download the Mac binary from `ArrayFire <https://arrayfire.com/download/>`_ and install it. For more help on installing ArrayFire on Mac see `here <http://arrayfire.org/docs/installing.htm#macOS>`_.

Alternatively, you can `build from source <https://github.com/arrayfire/arrayfire/wiki/Build-Instructions-for-OSX>`_.

OMEGA
^^^^^

Python pip install
******************

The pip install doesn't include examples so you need to manually download those: https://github.com/villekf/OMEGA/tree/master/source/Python

For Mac, you need to manually compile everything. For this you need to first type in Python ``from omegatomo.util.compile import compileOMEGA`` and then run
``compileOMEGA()``. You can insert ArrayFire path with ``compileOMEGA('-A /path/to/arrayfire')`` and ROOT path with ``compileOMEGA('-R /path/to/ROOT')``. This can be done in an interactive session.

Other cases
***********

Download either a release version from `releases <https://github.com/villekf/OMEGA/releases>`_, clone the current master with e.g. `git clone https://github.com/villekf/OMEGA.git` or download an archive of the 
`master-branch <https://github.com/villekf/OMEGA/archive/master.zip>`_. If you downloaded either a release or master branch archive, you need to extract the contents to the folder of your choosing. 

MATLAB/Octave
+++++++++++++

Alternatively, if you are using MATLAB, you can download the mltbx package (``OMEGA.-.Open-source.MATLAB.emission.tomography.software.mltbx``) from the `releases <https://github.com/villekf/OMEGA/releases>`_ and simply run it in 
which case all the necessary folders will be automatically added to the MATLAB path.

Unless the MATLAB package was used, you need to add the source and mat-files folders to the MATLAB/Octave path (biograph-folder should be added if you intend to use mCT or Vision list-mode data files). In MATLAB you can 
do this by simply right clicking the folders and selecting "Add to path -> Selected folders" by selecting the OMEGA folder itself and selecting "Add to path -> Selected folders and subfolders". Alternatively, if you are 
using for example Octave, you can add the paths with ``addpath('/path/to/OMEGA/source')`` and ``addpath('/path/to/OMEGA/mat-files')`` or simply with ``addpath(genpath('/path/to/OMEGA/'))``. On MATLAB you can also add 
these folders to the list of folders in "Set path".

To build all the necessary mex-files in MATLAB or Octave, simply run ``install_mex``. If ArrayFire was installed in some non-standard folder, the compilation might not work unless you include the folder to ``install_mex``. This can be done with
``install_mex(0, [], [], '/path/to/Arrayfire')``. See ``help install_mex`` for more details.

Python
++++++

The below compilation is not required if you only use custom reconstruction in Python.

In Python, navigate to ``/path/to/OMEGA/source/Python`` in terminal and run ``python compile.py`` to compile the library files. If ArrayFire was not installed in ``opt`` add the path with ``python compile.py -A /path/to/arrayfire``.

Optional software
-----------------

This section describes the optional software that can be used in OMEGA, but which are not required for any of the core functions. Most of these are for MATLAB only.

If you wish to use NIfTI data or save data as NIfTI format, on MATLAB you'll need EITHER image processing toolbox OR `Tools for NIfTI and ANALYZE image <https://se.mathworks.com/matlabcentral/fileexchange/8797-tools-for-nifti-and-analyze-image>`_. 
For Octave, only Tools for NIfTI and ANALYZE image can be used, though it hasn't been tested. For Python you can try `NiBabel <https://nipy.org/nibabel/>`_.

For Analyze data, you'll need the above `Tools for NIfTI and ANALYZE image <https://se.mathworks.com/matlabcentral/fileexchange/8797-tools-for-nifti-and-analyze-image>`_ in all cases.

For DICOM data, you'll need image processing toolbox on MATLAB and `dicom package <https://octave.sourceforge.io/dicom/index.html>`_ on Octave (untested). In Python you can use ``pydicom`` package to load DICOM data.

For 3D volumetric visualization, there is built-in support for `vol3d <https://www.mathworks.com/matlabcentral/fileexchange/22940-vol3d-v2>`_ in `visualize_pet.m <https://github.com/villekf/OMEGA/blob/master/visualize_pet.m#L344>`_.

For random subset sampling (``subset_type = 3``), `Shuffle <https://www.mathworks.com/matlabcentral/fileexchange/27076-shuffle>`_ can speed up the process as it is both faster and more memory efficient than the built-in function. Note that you need to 
enable this by setting ``options.shuffle = true``. MATLAB only!