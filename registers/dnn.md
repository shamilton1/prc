# DNN

## IMEM Regions

While any valid range of IMEM addresses can be provided to the XDNN Kernel when issuing Download, Upload, Conv2D, or Pool operations, it is recommended to adhere to the region map (as shown below)for maximum performance. By always using the associated reserved address ranges for the Download and Upload commands, the XDNN Kernel takes advantage of parallelization in downloading the next image into the reserved download region even while the current image is being processed in the Image Processing Region. Similarly, the Image upload of a processed image can occur in parallel with the processing of the next image. This scheme requires the XDNN Kernel to be given a hint of when the reserved Image Download region can be freed up for the next image download (as this can vary for different network architectures). This hint bit is provided in the CNN Command Register bit 11 “Free Reserved Download Region”, which is programmed during script memory initialization.

## Image Memory

The input image volume must be downloaded into the XDNN internal Image Memory before performing the Conv2D or MaxPool/AveragePool operations. The Conv2D and MaxPool/AveragePool processors always read the image volume from Image Memory and write the resulting image volume back into Image Memory.
The XDNN Image Memory's data width will match the DSP array width in bytes rounded up to the next power of 2. For GoogLeNet v1, a DSP width of 14 pixels at 2 bytes per pixel corresponds to a total DSP width of 28 bytes. Thus, the IMEM data width would be 32 bytes wide.

## DSP Dystolic Array

The DSP Systolic Array performs most of workload associated with the Conv2D operation. The depth of the array is fixed at 32, while the width (determined by parameter C_IMAGE_OP_PIXELS) can be selected to fit a specific neural network architecture. For example, for the GoogLeNet v1 architecture, a DSP array width of 14 or 28 pixels can be selected to efficiently process images that are a factor or multiple of the DSP width.

## Image Volume Size

When programming the Image Memory Base Address registers, you can avoid conflicts in the memory space by calculating and checking the image volume sizes as it is stored in the XDNN IMEM. The image size in the XDNN IMEM can be used directly as the address offset from a given Base Address to find out the image volume's occupied memory address range.
An image volume stored in the XDNN Image Memory will use more bytes than the natural size of the image volume. This is because:

  - The image width (in bytes) must be rounded up to the closest power of two to be aligned with the IMEM width.
  - The image width (in bytes) will, at minimum, be equal to the IMEM width due to XDNN's data packing performance optimization.
  - The image depth must be even so an odd depth must be rounded up to the next even number.

The dimensional translation of an image volume to how it is stored in the XDNN IMEM is summarized as follows:

![](/images/eq1.PNG)

## NonBlocking Mode

The CORDIC core provides a mode intended to ease the migration from previous, non-AXI
versions of this core. The term “NonBlocking” is used to indicate that lack of data on one
input channel does not cause incoming data on the other channel to be buffered. Also, back
pressure from the output is not possible because in NonBlocking mode the output channel
does not have a tready signal. The full flow control of AXI4-Stream is not always required.
Blocking or NonBlocking behavior is selected using the flow_control parameter or GUI field.
The choice of Blocking or NonBlocking applies to the whole core, not each channel
individually. Channels still have the non-optional tvalid signal, which is analogous to the
New Data (ND) signal on many cores prior to the adoption of AXI4-Stream. Without the
facility to block dataflow, the internal implementation is much simplified, so fewer
resources are required for this mode. This mode is recommended for users migrating their
design to this version from a pre-AXI version with minimal change.
When all of the present input channels receive an active tvalid (and tready, if present, is
asserted), an operation is validated and the output tvalid (suitably delayed by the latency
of the core) is asserted to qualify the result. This is to allow a minimal migration from
previous versions. If one channel receives tvalid and the other does not, an operation
does not occur, even if tready is present and asserted. Unlike Blocking mode (which is fully
AXI4-Stream compliant) valid transactions on an individual channel can be ignored in
NonBlocking mode. For performance, ARESETn is registered internally, which delays its
action by one clock cycle. The effect is that the core is still reset and does not accept input
in the cycle following the deassertion of ARESETn. tvalid is also inactive on the output
channel for this cycle.

## Blocking Mode

The term ‘Blocking’ means that each channel with tready buffers data for use. The full flow
control of AXI4-Stream aids system design because the flow of data is self-regulating.
Blocking or NonBlocking behavior is selected using the flow_control parameter or GUI field.
Data loss is prevented by the presence of back pressure (tready), so that data is only
propagated when the downstream datapath is ready to process the data. The CORDIC core
has one or two input channels and one output channel. When all input channels have
validated data available, an operation occurs and the result becomes available on the
output. If the output is prevented from off-loading data because m_axis_dout_tready
is low, data accumulates in the output buffer internal to the core. When this output buffer
is nearly full the core stops further operations. This prevents the input buffers from
off-loading data for new operations so the input buffers fill as new data is input. When the
input buffers fill, their respective treadys (s_axis_cartesian_tready and
s_axis_phase_tready) are deasserted to prevent further input. This is the normal action
of back pressure. The two input channels are tied, as each must receive validated data
before an operation can proceed. As an additional blocking mechanism, one input channel
does not receive validated data while the other does. In this case, the validated data is
stored in the input buffer of the channel. After a few cycles of this scenario, the buffer of the
channel receiving data fills and tready for that channel is deasserted until the empty
channel receives some data.

## Filter Vectors

Filter vectors are loaded from DDR to XDNN internal memory on the fly while the XDNN engine is performing the Conv2D function.The XDNN engine requires the filter vectors for all convolutional network layers to be preloaded into the FPGA DDR in a specific format before starting execution. This format is described in this section.
This explanantion uses shorthand symbols as described in the following table:

| Symbol | Description |
|:------|:-----------------------|
|D<sub>1</sub> | Output volume depth: <ul><li> - unpadded</li><li> - padding dependent</li><li>N/A</li>|
|D<sub>I</sub> | Input volume depth |
|D<sub>o</sub> | Input volume depth padded to even number |
|D<sub>s</sub> | Output volume depth padded to next multiple of 32 |
|D<sub>H</sub> | Kernel window height |
|D<sub>F</sub> | Conventional 3D Filter matrix for a single layer |
|D<sub>L</sub> | Conventional 3D Filter matrix for a single layer, padded as described below |

When stored in memory, this matrix is typically storeed in row-major format, with the right-most dimension changing fastest.

For the XDNN engine, the following sequence of transformations on this conventional filter matrix must occur before being loaded into the FPGA DDR:
  - Extend or shrink all filter values to 16b width
  - Transform padded 3D Filter Matrix into a 1D array
  - Switch to an even number of channels
  - Transform to the next multiple of 32

The following table shows the difference between the natural size of an image volume compared to when it is stored in the XDNN Image Memory (IMEM size)

#### Image Volume size -v- DNN IMEM

| Width | Height | Depth | Pixel Bytes | Natural Size | IMEM Size |
|-----|-----|-----|-----|-----|-----|
|97|98|99|224|107|106|
|98|99|102|226|108|107|
|95|97|94|225|107|106|
|97|98|99|224|107|106|
|99|99|102|224|128|107|

# Scaling, Shifting, and Bias

This section describes the Scaling, Shifting and Bias stages following the DSP array output for a single element.

After a 16-bit convolutional operation in the XDNN engine, scaling, shifting and bias operations are performed. These operations keep the 48-bit accumulated value within an appropriate (user defined) range to prevent overflow/underflow in the following layers of the network.

Scaling, Shifting and Bias values provided can be applied as a different value per-output channel, or the same value across all output channels (termed "Global" in the XDNN CSR description). This option is designated in the **Command Options** field. If the "per-output channel" option is selected, these values must be loaded into the FPGA DDR in advance of kernel execution, and a DDR offset from the filter base address must be provided through the XDNN CSR interface.

The stages following the DSP array output for a single element are shown below:

![](/images/scaling.PNG)











