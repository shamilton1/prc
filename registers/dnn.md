# DNN

## IMEM Regions

While any valid range of IMEM addresses can be provided to the XDNN Kernel when issuing Download, Upload, Conv2D, or Pool operations, it is recommended to adhere to the region map (as shown below) for maximum performance. By always using the associated reserved address ranges for the Download and Upload commands, the XDNN Kernel takes advantage of parallelization in downloading the next image into the reserved download region even while the current image is being processed in the Image Processing Region. Similarly, the Image upload of a processed image can occur in parallel with the processing of the next image. This scheme requires the XDNN Kernel to be given a hint of when the reserved Image Download region can be freed up for the next image download (as this can vary for different network architectures). This hint bit is provided in the CNN Command Register bit 11 “Free Reserved Download Region”, which is programmed during script memory initialization.

This API call retrieves the image data from the XDNN image memory and performs a max or average pooling on the image volume. The result is written back to XDNN image memory at the specified output image memory address. The associated registers are programmed into the script memory so a Script Memory Push command is required as the last CSR write.
For the last average-pool layer before classification in the GoogLeNet v1 network, the fully-connected mode bit must be set in the command-options register field to indicate this average pool will map a 7x7 channel down to a 1x1 channel.

## Image Memory

The input image volume must be downloaded into the XDNN internal Image Memory before performing the Conv2D or MaxPool/AveragePool operations. The Conv2D and MaxPool/AveragePool processors always read the image volume from Image Memory and write the resulting image volume back into Image Memory.
The XDNN Image Memory's data width will match the DSP array width in bytes rounded up to the next power of 2. For GoogLeNet v1, a DSP width of 14 pixels at 2 bytes per pixel corresponds to a total DSP width of 28 bytes. Thus, the IMEM data width would be 32 bytes wide.

## DSP Dystolic Array

The DSP Systolic Array performs most of the workload associated with the **Conv2D** operation. The width of the array is fixed at 32 pixels, while the width (determined by parameter **C_IMAGE_OP_PIXELS**) can be selected to fit a specific neural network architecture. For example, for the GoogLeNet v1 architecture, a DSP array width of 14 or 28 pixels can be selected to efficiently process images that are a factor or multiple of the DSP width.

This API call retrieves the image data from the XDNN image memory and performs a max or average pooling on the image volume. The result is written back to XDNN image memory at the specified output image memory address. The associated registers are programmed into the script memory so a Script Memory Push command is required as the last CSR write.
For the last average-pool layer before classification in the GoogLeNet v1 network, the fully-connected mode bit must be set in the command-options register field to indicate this average pool will map a 7x7 channel down to a 1x1 channel.

## Image Volume Size

When programming the Image Memory (IMEM) Base Address registers, you can avoid conflicts in the memory space by calculating and checking the image volume sizes as it is stored in the XDNN IMEM. The image size in the XDNN IMEM can be used directly as the address offset from a given Base Address to find out the image volume's occupied memory address range.
An image volume stored in the XDNN Image Memory will use more bytes than the natural size of the image volume. This is because:

  - The image width (in bytes) must be rounded up to the closest power of two to be aligned with the IMEM width.
  - The image width (in bytes) will, at minimum, be equal to the IMEM width due to XDNN's data packing performance optimization.
  - The image depth must be even so an odd depth must be rounded up to the next even number.

The dimensional translation of an image volume to how it is stored in the XDNN IMEM is summarized as follows:

![](/images/eq1.PNG)

## NonBlocking Mode

The CORDIC core supports a *Non-Blocking* mode. This is intended to facilitate the migration from previous, non-AXI
versions of this core. The term “Non-Blocking” is used to indicate that a lack of data on one
input channel does not cause incoming data on the other channel to buffer. Also, back
pressure from the output is not possible because in NonBlocking mode, the output channel
has no **tready** signal. The full flow control of AXI4-Stream is not always required.
Blocking or NonBlocking behavior is selected using the flow_control parameter or GUI field.
Selection of Blocking or NonBlocking applies to the whole core, not to individual channels. Channels still have the non-optional tvalid signal, which is analogous to the New Data (ND) signal on many cores prior to the adoption of AXI4-Stream. Without the
facility to block dataflow, the internal implementation is much simplified, so fewer
resources are required for this mode. This mode is recommended for users migrating their
design to this version from a pre-AXI version with minimal change.
When all of the present input channels receive an active tvalid (and tready, if present, is
asserted), an operation is validated and the output tvalid (suitably delayed by the latency
of the core) is asserted to qualify the result. This is to allow minimal migration from
previous versions. If one channel receives tvalid and the other does not, an operation
does not occur, even if tready is present and asserted. Unlike Blocking mode (which is fully
AXI4-Stream compliant) valid transactions on an individual channel can be ignored in
NonBlocking mode. For performance, ARESETn is registered internally, which delays its
action by a single clock cycle meaning the core is still reset but does not accept input
in the cycle following the de-assertion of ARESETn. tvalid is also inactive on the output
channel for this cycle.

## Blocking Mode

The term ‘Blocking,’ in this context means that each channel with tready buffers data for use. The full flow
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

Before Right-Shift, filter vectors are loaded from DDR to XDNN internal memory on the fly while the XDNN engine is performing the Conv2D function. The XDNN engine requires the filter vectors for all convolutional network layers to be preloaded into the FPGA DDR in a specific format before starting execution. This format is described in this section.
This explanantion uses shorthand symbols as described in the following table:

| Symbol | Description | Dependencies|API|
|:------|:-----------------------|--------|-----|
|D<sub>1X</sub> | Output Volume depth: <ul><li> Unpadded</li><li> Padding dependent</li><li>N/A</li>|Optional|xfDNN_InitOnce|
|D<sub>IX</sub> | Input Volume depth |HW_Core_Num: XDNN|xfDNN_Download|
|D<sub>oX</sub> | Input Volume depth padded to even number |HW_ImgMem_WordCnt|xfDNN_InitOnceUpload|
|D<sub>sX</sub> | Output Volume depth padded to next multiple of 32 |HW_ImgMem_WordCnt|xfDNN_Conv2D|
|D<sub>HX</sub> | Kernel Window height |HW_ImgMem_WordCnt|xfDNN_Pool|
|D<sub>FX</sub> | Conventional 3D Filter matrix for a single layer |ComplVec[31:0] Set|xfDNN_InitOnce|
|D<sub>LX</sub> | Conventional 3D Filter matrix for a single layer, padded as described below |ComplVec[31:0]|xfDNN_InitOnce|

When stored in memory, this matrix is typically stored in row-major format, with the right-most dimension changing fastest.

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

Scaling, Shifting and Bias values provided can be applied as a different values per-output channel, or the same value across all output channels (termed "Global" in the XDNN CSR description). This value is assigned in the **Command Options** field. If the **Per-Output channel** option is selected, these values must be loaded into the FPGA DDR in advance of kernel execution, and a DDR offset from the filter base address must be provided through the XDNN CSR interface.

The stages following the DSP array output for a single element are shown below:

![](/images/scaling.PNG)

### Pre-Scale Right-Shift

This stage takes the 48-bit accumulated output from the DSP as input, and right-shifts the input by the amount defined in the 6-bit CSR field **Pre-Scale DSP Right-Shifting**. The resulting lower 16 bits are preserved and output to the next stage as described below.

**Note:** If the right-shifted value is not equal to the 16-bit signed value, the 16-bit output is capped to either the min. or max. of the 16-bit signed value.

### Scaling

This stage takes the 16-bit output from the Pre-Scale right-shift stage (higlighted) as its input. It multiplies the input by the amount assigned in the **8-bit CSR field DSP Scale Multipliucation value**. The resulting 24-bit value is output to the next stage (**Post-Scale Right-Shift**).

### Post-Scale Right-Shift

This stage takes the 24-bit output from the scaling stage and right shifts the input by the amount assigned in the 4-bit **CSR field Post-Scale DSP Right-Shifting**. The resulting lower 16 bits are preserved and output to the next stage.

### Bias

This stage takes the 16-bit output from the post-scale right-shift stage as input and adds a 16-bit signed bias value designated in the 16-bit CSR field Output Image Bias Value. The resulting 16-bit value is output to the next stage. If the resulting sum is greater or less than what a 16-bit signed value can represent, the 16-bit output will be capped to either the max or min 16-bit signed value.


## Running GoogleNet v1

Initialize XDNN script memory to describe the GoogLeNet network architecture. See Script Memory Initialization.

- Load filter and image data into DDR (Host >FPGA). See GoogLeNet v1 API.

- Submit new job(s) into the XDNN command-queue. See Submitting an Image Job.

- Interrupt/Poll completion registers for job(s) completion. See Checking Completion Status.

- Read output image data from DDR (FPGA >Host). See Transfer Output Images: FPGA > Host DDR.

## Script Memory Initialization

The script memory must be programmed with the attributes of each layer in the GoogLeNet architecture. This example API composed of a set of pseudo high-level programming language function calls. The important consideration in this example is the sequence of CSRs that must be written into the XDNN engine for correct operation.

## GoogleNet v1 Initialization

Using the API commands defined above, you can begin to issue the sequence of function calls that will program the GoogLeNet network into the XDNN script memory. The table below shows the sequence of commands.

The image memory addresses given (offsets: 0x60, 0x6c) apply only to the 14x32 DSP array XDNN engine. The 28x32 DSP array XDNN engine addresses needs to be calculated differently.

## XDNN Image Submissions

### Polling Based

Using the polling based method, the interrupt status register is polled first to determine which of the eight Image Completion Vector Registers has an index bit set. The following table shows an example API which details each of the registers read during this routine.

|Address&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|Register Name|Comments|
|--------|-----|-----|
|0x0008|Interrupt Status Register|Poll this register first to determine which Image Completion Vector register to read from.|
|0x30-0x4c|Image Completion Vector Register|Optional|

This API call is the download operation that retrieves the image data from DDR and stores it into the XDNN image memory. The corresponding set of registers is programmed into the script memory so a **Script Memory Push** command is required as the last CSR write.

### Interrupt Based

Using the Interrupt based method, the XDNN engine outputs eight interrupt wires which correspoind to the eight Image Completion Vector registers. When software detects the interrupt, it can optionally set the interrupt mask registerbit which masks the interrupt wire(s), and then proceed to read the appropriate Image Completion Vector Register. It can then clear the interrupt mask register bit(s) to re-enable interrupts.

The following table shows an example API detailing the relevant registers read during this routine.

|Address&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|Register Name|Comment|
|-----|-----|-----|
|0x000c|Interrupt Mask Register| Masks the interrupt wire outputs of the XDNN engine|
|0x30-0x4c|Image Completion Vector Register 0-7|Optional|


## APIs

This section describes the five fundamental API function calls to be defined. Each function lists the corresponding XDNN CSRs that will be programmed by the function's arguments (args). Although not shown explicitly, it is expected that each arg passed into each API function will map directly to each of the listed registers' bit fields.

Refer to Register Space for further details of how to program each register.

**Note:** All registers listed under each API call must be programmed by args, unless the comments state “Optional.”

     xfDNN_Pool
This API call retrieves the image data from the XDNN image memory and performs a max or average pooling on the image volume. The result is written back to XDNN image memory at the specified output image memory address. The associated registers are programmed into the script memory so a Script Memory Push command is required as the last CSR write.
For the last average-pool layer before classification in the GoogLeNet v1 network, the fully-connected mode bit must be set in the command-options register field to indicate this average pool will map a 7x7 channel down to a 1x1 channel.

    xfDNN_Conv2D
This API call retrieves the filter data from DDR at the specified Base+Offset Address, retrieves the image data from the XDNN image memory, and performs the convolution. The result is written back to XDNN image memory at the specified output image memory address. The associated registers are programmed into the script memory so a `Script Memory Push` command is required as the last CSR write

    xfDNN_Upload
This API call is the upload operation that retrieves the image from the XDNN image memory and uploads it to the DDR. A `Script Memory Push` command is required as the last CSR write.

    xfDNN_Download

This API call is the download operation that retrieves the image data from DDR and stores it into the XDNN image memory. The associated registers are programmed into the script memory so a `Script Memory Push` command is required as the last CSR write.

    xfDNN_InitOnce
This API call initializes the set of XDNN CSRs that require one-time programming. These registers are not programmed into the script memory, so they do not require a `Script Memory Push` command at the end. The associated registers are composed of DDR Base Addresses for Image and Filter data and AXI interface performance setting.
