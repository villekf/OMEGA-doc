Selecting the optimal projector
===============================

| This page outlines which projector is more optimal for specific data types. You can find references to the projector types below:
| Type 1: http://cit.fer.hr/index.php/CIT/article/view/2969
| Type 2: https://doi.org/10.1118/1.3501884
| Type 3: https://doi.org/10.1088/0031-9155/59/3/561
| Type 4: https://doi.org/10.1088/0031-9155/56/13/004
| Type 5: https://doi.org/10.1109/TCI.2017.2675705
| Type 6: https://doi.org/10.1109/23.552756

PET data
--------

For PET data, projector_type = 3 is the most optimal one, though also slowest to compute. This is a volume of intersection based projector, where the voxels are approximated as spheres and the tube of intersection as a cylinder.
The user can determine the size of the sphere with ``options.voxel_radius``, where value of 1 means that the sphere is just large enough to fully contain the cubic voxel. Larger values have larger spheres, while smaller
lead to smaller spheres. The radius of the tube can be set with ``options.tube_radius``.

An alternative to above, is projector_type = 2, which is an orthogonal distance-based ray-tracer. This is practically equal in speed with the above. It computes the orthogonal distance from the voxel where the ray is currently to the nearby voxels. If the distance is lower than
the threshold value then the distance is included, otherwise it is omitted. You can adjust this distance with ``options.tube_width_xy`` in the XY-plane, i.e. 2D, and with ``options.tube_width_z`` in 3D. If ``options.tube_width_z`` is
set, then ``options.tube_width_xy`` is ignored. ``options.tube_width_z`` always operates in all three dimensions.

The fastest methods are projector_type = 1 or projector_type = 4. The first one computes the exact line of intersection length in each voxel using an improved version of the Siddon's algorithm. The latter is an interpolation-based
projector, similar to Joseph's method, i.e. it linearly interpolates values after a pre-determined step-size. This step-size can be adjusted with ``options.dL`` and is the relative size of one voxel. I.e. ``options.dL = 1``
would use the length of one voxel as the interpolation length. Multi-ray version is also available though recommended only for projector_type = 1. Number of transaxial rays can be adjusted with ``options.n_rays_transaxial`` and 
axial with ``options.n_rays_axial``. While projector_types 1 and 4 are the fastest, they produce lower quality images than projector types 2 and 3. However, you can offset this somewhat by using PSF blurring. This can be enabled
by setting ``options.use_psf`` to true. The FWHM can be adjusted with ``options.FWHM`` and must include value for all three dimensions. Note that projector type 4 is OpenCL/CUDA only! This means that only implementations 2, 3 and 5 
support it.

Note that you can also use hybrid methods if you wish, such as ``projector_type = 13``, but these are not recommended with PET data. The first value corresponds to the forward projection (1 in this case) and the second to the
backprojection (3 in this case).

CT data
-------

All optimal CT projectors are OpenCL/CUDA only! This means that only implementations 2, 3 and 5 support them and the CPU version of implementation 2 does not!

For CT data, generally the most optimal choice is the branchless distance-driven (BDD) projector, i.e. projector_type = 5. However, this method is slow and can be practically useless in certain cases. It has reduced usefulness when
using regularization. BDD is the branchless version of the distance-driven projector. In general, the DD methods compute the area of the intersection. For forward projection, we project rays from the source to the four corners of
a detector pixel. The area inside these four points in the image volume is then computed for each slice. Backprojection works similarly, but we project the lines to the corners of each voxel and then compute the area on the detector.
In general, the backprojection tends to have greater beneficial effect when compared with the other projectors than the forward projection. See below for hybrid projectors.

Projector type 4 is method that should work in most cases well. It is ray-based interpolation-based projector in the forward projection and voxel-based projector in the backprojection. The forward projection is identical with the
PET version, i.e. it linearly interpolates values after a pre-determined step-size. This step-size can be adjusted with ``options.dL`` and is the relative size of one voxel. I.e. ``options.dL = 1``
would use the length of one voxel as the interpolation length. The backprojection projects rays from the source through each voxel to the detector and then interpolates this value on the detector using nearest neighbor interpolation. 
This projector is not exactly adjoint, but the difference is generally less than 1%. Using longer interpolation length leads to faster computation, but too large values lead to loss of accurace. Generally, value of 0.5 or 1 is
a good choice.

Projector types 1, 2 and 3 are not recommended for CT data, but they work with implementation 4, i.e. they support CPU reconstruction outside of OpenCL. Projector type 1 also works with implementation 1, if you need the actual 
system matrix.

Hybrid projectors are recommended. For example ``projector_type = 14`` is an alternative method where the forward projection uses an improved version of the Siddon's ray-tracer. This means that the exact intersection length
is computed. Alternatively, the BDD can be combined with either, e.g. ``projector_type = 45``. Note that generally it is not recommended to use hybrid projectors where BDD is the forward projector, such as ``projector_type = 54``.
A good combination of quality and speed is to use ``projector_type = 45``, or ``projector_type = 15``. 

SPECT data
----------

Only projector type 6 is recommended at this point. This is a rotation-based projector, meaning it rotates the images to simulate parallel beam reconstruction. Detector response can be included as well to enable PSF blurring.

Other data
----------

Projector type 1 is recommended. It is the most robust and flexible method and should work in all voxel-based ray-tracing cases. Projector type 3 might also work, but this depends on the data. See PET data above for details 
on projector type 3.

If your data is similar to that of CT data, then using CT projectors should be fine. In such a case, see CT data above.