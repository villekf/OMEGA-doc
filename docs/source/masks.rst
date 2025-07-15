Using mask images
=================

In OMEGA, it is currently possible to limit the measurements to be used or voxels to be reconstructed. This is possible with the use of mask images.

Masks in forward projection
---------------------------

A mask can be input to ``options.maskFP`` in unsigned 8-bit format (uint8 or uchar) where 1 means that the measurement is taken into account while 0 means that it is not.
This type of mask is utilized in the forward projection to skip certain measurement locations. The forward projection is thus not computed if the pixel is 0.
The mask should either be a 2D image which is then used for all measurement data, or a 3D stack of images where an individual image is used for each measurement slice. 
The measurement data can be, for example, projections or sinograms, where each image would correspond to a projection or sinogram. The masks can be used to remove dead pixels
from the projections, or sinograms, by setting those pixels to zero in the mask images. The mask images will only work as uint8 images, logical masks won't work!

Subset type 3 cannot be used with forward projection mask!

Note that if you use the forward and/or backward projection operators, you need to handle the subset division of the mask image when you are using subsets types 8-11. 
Subset types 0-7, excluding 3, should work correctly, but this is untested.

Masks in backprojection
-----------------------

These are similar to above, but operate in the backprojection, i.e. you can reconstruct only specific voxels. The mask is input to ``options.maskBP``  in unsigned 8-bit format 
(uint8 or uchar) where 1 means that the voxel is reconstructed while 0 means that it is not. The mask should either be a 2D image which is then used for all image volume slices, 
or a 3D stack of images where an individual image is used for each image volume slice. 

Unlike the forward projection mask, there are no restrictions in using the backprojection mask.

To create a mask that covers the FOV in a cylinder shape, you can use the following code

MATLAB/Octave:

.. code-block:: matlab

	[columnsInImage, rowsInImage] = meshgrid(1:options.Nx, 1:options.Ny);
	centerX = options.Nx/2;
	centerY = options.Ny/2;
	radius = options.Nx/2;
	options.maskBP = uint8((rowsInImage - centerY).^2 ...
		+ (columnsInImage - centerX).^2 <= radius.^2);
		

Python:

.. code-block:: python

	columns_in_image, rows_in_image = np.meshgrid(np.arange(1, options.Nx + 1), np.arange(1, options.Ny + 1))
	centerX = options.Nx / 2
	centerY = options.Ny / 2
	radius = options.Nx / 2
	options.maskBP = ((rows_in_image - centerY)**2 + (columns_in_image - centerX)**2 <= radius**2).astype(np.uint8)
	
Note that the above examples create a single 2D image, but you can create a 3D stack if you want more control over the reconstruction per slice.