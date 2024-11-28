Useful links
============

Here is a collection of links to various potentially useful resources such as software or datasets. These are, mainly, open-source. This list will be, hopefully, updated over time.

Simulation software
-------------------

* Geant4: https://geant4.web.cern.ch/
 * Particle physics simulator

* GATE: http://www.opengatecollaboration.org/
 * Geant4-based simulation toolkit for PET, SPECT, CT, optimal imaging (bioluminescence and fluorescence) and radiotherapy
 * Not recommended for CBCT simulations
 
* SIMIND: https://simind.blogg.lu.se/
 * Monte Carlo simulation toolkit for SPECT
 * Easier to use than GATE but lacks more advanced features
 
* MCGPU: https://github.com/DIDSR/MCGPU
 * GPU(CUDA)-based Monte Carlo simulation toolkit for CT imaging
 * Mammography and PET simulators also available
 
* GGEMS: https://github.com/GGEMS/ggems
 * Geant4-based GPU(OpenCL)-based Monte Carlo simulation toolkit for CT imaging
 * Can be unreliable with higher-dimensional scanners
 
* DukeSim: https://cvit.duke.edu/resource/dukesim-v1-2/
 * GPU(CUDA)-based simulator for CT
 * Not open-source!
 * Not Monte Carlo!
 
* CTSim: http://ctsim.org/
 * CT simulation toolkit
 * Doesn't seem to be maintained anymore
 
* ValoMC: https://inverselight.github.io/ValoMC/
 * Open source Monte Carlo code for simulation the passage of visible and near infrared range photons through a medium
 * MATLAB only
 
* MRiLab: https://leoliuf.github.io/MRiLab/
 * MRI simulator
 
* Fastcat: https://github.com/jerichooconnell/fastcat
 * Analytical CBCT simulator for Python
 
* CTlab: https://github.com/MIPT-Oulu/CTlab
 * CTlab is virtually implemented medical imaging device, which can be widely used in computed tomography training for all professionals who use radiation in their work
 
Reconstruction software
-----------------------

* OMEGA: https://github.com/villekf/OMEGA
 * This software, in case someone ends up here through some other means
 * Open-source multi-dimensional tomographic reconstruction software

* HELMET, High-dimensional Kalman filter toolbox: https://github.com/villekf/HELMET
 * My own Kalman filter toolbox for MATLAB for linear dynamic problems, especially higher-dimensional ones
 
* STIR, Software for Tomographic Image Reconstruction: https://stir.sourceforge.net/
 * C++-based reconstruction software for PET and SPECT
 
* TIGRE, Tomographic Iterative GPU-based Reconstruction Toolbox: https://github.com/CERN/TIGRE/
 * MATLAB and Python based GPU (CUDA) capable reconstruction software for CT imaging
 
* CASToR, Customizable and Advanced Software for Tomographic Reconstruction: https://castor-project.org/
 * C++-based reconstruction software for PET, SPECT and CT
 
* PyTomography: https://github.com/qurit/PyTomography
 * Python-based tomography reconstruction toolkit for PET and SPECT
 
* ASTRA: https://astra-toolbox.com/
 * MATLAB and Python toolbox of high-performance GPU primitives for 2D and 3D tomography
 
* TIRIUS: https://sourceforge.net/projects/tirius/
 * Tomography reconstruction toolkit
 * Doesn't seem to be maintained anymore
 
* J-PET Analysis Framework: https://github.com/JPETTomography/j-pet-framework
 * Reconstruction and analysis toolkit for PET
 
* QSPECT: http://www.qspect-project.com/index_e.html
 * SPECT reconstruction toolkit
 * Doesn't seem to be maintained anymore
 
* MIRT, Michigan Image Reconstruction Toolkit: https://github.com/JeffFessler/mirt
 * Tomographic image reconstruction toolkit, especially for medical imaging (emission, transmission, MRI)
 * Julia version: https://github.com/JeffFessler/MIRT.jl
 
* NiftyRec: https://github.com/TomographyLab/NiftyRec
 * GPU(CUDA)-based image reconstruction toolkit for tomographic imaging
 
* MR-Hub: https://ismrm.github.io/mrhub/
 * Collection of various open-source MRI software, including reconstruction software

* RTK: https://github.com/RTKConsortium/RTK
 * The Reconstruction Toolkit
 
* ODL: https://github.com/odlgroup/odl
 * Operator Discretization Library for Python
 
* LEAP: https://github.com/LLNL/LEAP
 * LivermorE AI Projector for Computed Tomography
 
* HelTomo: https://github.com/Diagonalizable/HelTomo
 * CT reconstruction toolkit for MATLAB based on ASTRA and Spot Linear-Operator toolboxes
 
* OOEIT: https://github.com/PetriKuusela/OOEIT
 * EIT reconstruction and simulation software
 
* ToMoBAR: https://github.com/dkazanc/ToMoBAR
 * TOmographic MOdel-BAsed Reconstruction (ToMoBAR) software 
 
Data analysis software
----------------------

* AEDES: https://github.com/mjnissi/aedes
 * ROI analysis tool for MRI images
 
* Algotom: https://github.com/algotom/algotom
 * Data processing algorithms for tomography
 
* ROOT: https://root.cern/
 * CERN's data analysis software for particle physics
 
* CARIMAS: https://carimas.fi/
 * Data analysis tool for PET images
 * Commercial software
 
* MR-Hub: https://ismrm.github.io/mrhub/
 * Collection of various open-source MRI software, including data analysis software
 
* ImageJ: https://imagej.net/ij/
 * Potentially useful visualization and analysis tool for medical images
 
* AMIDE: https://amide.sourceforge.net/
 * A bit similar to ImageJ, i.e. a visualization and analysis tool for medical images
 
* Insight Toolkit: https://itk.org/
 * Image analysis toolkit for e.g. segmentation and registration
 
* (X)MedCon: https://xmedcon.sourceforge.io/
 * Medical image conversion tool
 
* 3D Slicer: https://www.slicer.org/
 * 3D Slicer is a free, open source software for visualization, processing, segmentation, registration, and analysis of medical, biomedical, and other 3D images and meshes; and planning and navigating image-guided procedures.
 
Programming and languages
-------------------------

* Julia language: https://julialang.org/
 * Modern, Python- and MATLAB-like, language (open-source)
 
* Flux.jl: https://github.com/FluxML/Flux.jl
 * Julia's machine learning library
 
* ZLUDA: https://github.com/vosen/ZLUDA
 * Run CUDA applications on AMD GPUs
 * An alternative fork, based on earlier work: https://github.com/lshqqytiger/ZLUDA
 
* AMD HIP: https://github.com/ROCm/HIP
 * AMD's version of CUDA
 * HIP code can run on both AMD and Nvidia hardware
 
* Intel OneAPI: https://www.intel.com/content/www/us/en/developer/tools/oneapi/overview.html
 * SYCL-based API for parallel architectures, such as GPUs
 
* ArrayFire: https://github.com/arrayfire/arrayfire
 * General-purpose tensor library for parallel architectures
 * Supports CPU, OpenCL, CUDA and OneAPI
 
* EasyCL: https://github.com/hughperkins/EasyCL
 * Can make running OpenCL kernels easier
 
* Kokkos: https://github.com/kokkos/kokkos
 * Implements a programming model in C++ for writing applications targeting all major HPC platforms. Supports CUDA, HIP, SYCL, HPX, OpenMP and C++.
 
Deep learning based denoisers
-----------------------------

* ADL: https://github.com/mogvision/ADL
 * Adversarial Distortion Learning for Denoising and Distortion Removal
 
* Low-dose CT denoiser: https://github.com/Ryosaeba8/Medical-Imaging-LOW-DOSE-CT-DENOISING
 * Implementation of Low Dose CT Image Denoising Using a Generative Adversarial Network with Wasserstein Distance and Perceptual Loss
 
* CoreDiff: https://github.com/qgao21/CoreDiff
 * Contextual Error-Modulated Generalized Diffusion Model for Low-Dose CT Denoising and Generalization
 
* DU-GAN: https://github.com/Hzzone/DU-GAN
 * Generative Adversarial Networks with Dual-Domain U-Net Based Discriminators for Low-Dose CT Denoising
 
Datasets
--------

* Finnish Inverse Problems Society datasets: https://zenodo.org/communities/fips/
 * Datasets for, for example, CBCT, electrical impedance tomography and PET
 
* fastMRI dataset: https://fastmri.med.nyu.edu/

* Low dose CT grand challenge dataset: https://www.aapm.org/GrandChallenge/LowDoseCT/

* Stanford University datasets: https://aimi.stanford.edu/shared-datasets

* FAME datasets: https://fameflagship.fi/category/output/data/
 * Datasets for, for example, fMRI and EIT