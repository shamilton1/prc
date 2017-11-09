# DNN

## IMEM Regions

While any valid range of IMEM addresses can be provided to the XDNN Kernel when issuing Download, Upload, Conv2D, or Pool operations, it is recommended to adhere to the region map (as shown below)for maximum performance. By always using the associated reserved address ranges for the Download and Upload commands, the XDNN Kernel takes advantage of parallelization in downloading the next image into the reserved download region even while the current image is being processed in the Image Processing Region. Similarly, the Image upload of a processed image can occur in parallel with the processing of the next image. This scheme requires the XDNN Kernel to be given a hint of when the reserved Image Download region can be freed up for the next image download (as this can vary for different network architectures). This hint bit is provided in the CNN Command Register bit 11 “Free Reserved Download Region”, which is programmed during script memory initialization.

## Image Memory

The input image volume must be downloaded into the XDNN internal Image Memory before performing the Conv2D or MaxPool/AveragePool operations. The Conv2D and MaxPool/AveragePool processors always read the image volume from Image Memory and write the resulting image volume back into Image Memory.
The XDNN Image Memory's data width will match the DSP array width in bytes rounded up to the next power of 2. For GoogLeNet v1, a DSP width of 14 pixels at 2 bytes per pixel corresponds to a total DSP width of 28 bytes. Thus, the IMEM data width would be 32 bytes wide.

## DSP Dystolic Array

The DSP Systolic Array performs most of workload associated with the Conv2D operation. The depth of the array is fixed at 32, while the width (determined by parameter C_IMAGE_OP_PIXELS) can be selected to fit a specific neural network architecture. For example, for the GoogLeNet v1 architecture, a DSP array width of 14 or 28 pixels can be selected to efficiently process images that are a factor or multiple of the DSP width.

## Image Volumne Size

When programming the Image Memory Base Address registers, you can avoid conflicts in the memory space by calculating and checking the image volume sizes as it is stored in the XDNN IMEM. The image size in the XDNN IMEM can be used directly as the address offset from a given Base Address to find out the image volume's occupied memory address range.
An image volume stored in the XDNN Image Memory will use more bytes than the natural size of the image volume. This is because:

  - The image width (in bytes) must be rounded up to the closest power of two to be aligned with the IMEM width.
  - The image width (in bytes) will, at minimum, be equal to the IMEM width due to XDNN's data packing performance optimization.
  - The image depth must be even so an odd depth must be rounded up to the next even number.

The dimensional translation of an image volume to how it is stored in the XDNN IMEM is summarized as follows:

![]:(images/dbb.PNG)







