## Control Register Format

| Bits | Meaning | Details |
| ----------- | ------------- | ----------------- |
| 31:16 | HALFWORD FIELD | A 16-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 15:8 | BYTE FIELD | An 8-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 7:0 | CMD | The following commands are defined: <ul><li>000 = Shutdown</li><li>010 = Restart with Statuses</li><li>000 = Proceed</li><li>100 = User Control</li></ul>All other values are reserved.|

## SW_Trigger Register

| Bits | Meaning | Details |
| ------------ | ----------- | ------------------ |
| 31 | Trigger Pending &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Ignored on Write. On read, returns: 1 if there is a software trigger pending. 0 if there is no software trigger pending |
| 30:*W* | Reserved | Ignored on read. 0 on write. |
| *W*-1:0 | Trigger | The trigger identifier. The value written to this register is a positive integer that directly specifies the row in the trigger register bank that holds the identifier of the Reconfigurable Module to be loaded by this trigger. Writing this while a trigger is pending overwrites the pending trigger. <ul><li>000 = Shutdown</li><li>010 = Restart with Status</li><li>011 = Proceed</li><li>100 = User Control</li></ul> All other values are reserved.|

## Bank 1: Trigger to Reconfigurable Module Registers

The Trigger to Reconfigurable Module registers contain the mapping between the Triggers
and the Reconfigurable Modules to load. There can be more triggers than Reconfigurable
Modules, allowing for the in-field addition of Reconfigurable Modules and/or easier
triggering of the same Reconfigurable Module from multiple sources.

#### X-Bits

Each register is 32 bits wide, but only the lower X bits are used, where X = *log <sub>2</sub>  (Number of triggers allocated for this virtual socket manager)*

Unused bits are ignored on writes, and return '0' on reads. The Trigger to *<sup>Reconfigurable</sup>*
Module registers can only be accessed when the Virtual Socket Manager is in the shutdown
state. If the Virtual Socket Manager is not in the shutdown state, reads return '0' and writes
are ignored.

#### UltraScale+

When managing UltraScale+ devices, all registers in this bank are
readable and writable when the Virtual Socket Manager is in the shutdown state.
Reconfigurable Modules for this type of device require two bitstreams, therefore all bitstream
identifiers are 0 or 1.

When managing 7 series or UltraScale+ devices, the *BS_ADDRESS* and
*BS_SIZE* registers in this bank are readable and writable when the Virtual Socket Manager is in the shutdown state. Reconfigurable Modules for this type of device only require one
bitstream, so all bitstream identifiers are 0. Writes to the BS_ID registers are ignored and
reads always return 0.

## Restart with no Status command

The **Restart with no Status** command instructs the Virtual Socket Manager to exit the
shutdown state. The Virtual Socket Manager's Empty/Full status, Reconfigurable Module
identifier, and error status remain as they were before the Virtual Socket Manager entered
the shutdown state.

This command should be used to restart a Virtual Socket Manager in shutdown if the
Virtual Socket has not been modified during shutdown.

This command can only be used if the Virtual Socket Manager is in the shutdown state.

The BYTE and HALFWORD fields of the control word are not used with this command.

## Restart with Status command

The **Restart with Status** command instructs the Virtual Socket Manager to exit the
shutdown state. The Virtual Socket Manager's Empty/Full status, and Reconfigurable
Module identifier are specified as part of the command.

*This command should be used to restart a shutdown Virtual Socket Manager if the Virtual
Socket has been modified during shutdown (that is, a Reconfigurable Module is loaded into
the Virtual Socket by something other than the Virtual Socket Manager).*

This command must only be used with a full status if the loaded Reconfigurable Module is
known to the Virtual Socket Manager. If a Reconfigurable Module is loaded that is unknown
to the Virtual Socket Manager, the Virtual Socket must be left in an empty state before the
*Virtual Socket Manager is restarted. An empty state means that either there is no
Reconfigurable Module in the Virtual Socket, or that the loaded Reconfigurable Module
does not need any shutdown steps.*

When the Virtual Socket is on an UltraScale device,
there is an additional requirement that the loaded Reconfigurable Module is unmasked and
does not need its clearing bitstream loaded.

## Shutdown command

The **Shutdown** command instructs the Virtual Socket Manager to enter the shutdown state
at the earliest safe opportunity. There can be a long delay (indeterminate) between the
request and when the Virtual Socket Manager enters the shutdown state. You cannot cancel
the Shutdown command after it has been sent.

```
• vsm_<name>_rm_shutdown_req

• vsm_<name>_rm_decouple

• vsm_<name>_rm_reset

• vsm_<name>_sw_shutdown_req

• vsm_<name>_sw_startup_req

```

![](/images/hyperbolic_sinh_cosh.PNG)


## Output Quantization Error

The Output Quantization Error can be split into two components; the Output Quantization
Error due to the Input Quantization (**OQEIQ**), and the Output Quantization Error due to
Internal Precision (**OQEIP**).

**OQEIQ** is due to the half LSB of quantization noise on the **X_IN,Y_IN** and **PHASE_IN** inputs.
In a vector rotation this input quantization noise results in OQEIQ of a half LSB on both the
**X_OUT** and **Y_OUT** outputs. In a vector translation this input quantization noise results in
OQEIQ of a half LSB on the **X_OUT** output; however, **OQEIQ** on the phase output is
dependent on the ratio **(Y_IN/ X_IN)**. Thus for small **X_IN** inputs the effect of input
quantization noise on **OQEIQ** is greatly magnified.

Due to the limited precision of internal calculations, in the CORDIC core the default
internal precision is set such that the accumulated OQEIP is less than 1/2 the OQEIQ. The
internal precision can be manually set to **(Input_Width + Output_Width +
log2(Output_Width)**). This reduces OQEIP to a half LSB (the phase is calculated to full
precision regardless of the magnitude input vector).

## Phase Signals
The **s_axis_phase_tdata** Phase operand is **PHASE_IN**. The **m_axis_dout_tdata** phase
output is called **PHASE_OUT**. The phase signals are always represented using a fixed-point
twos complement number with an integer width of 3 bits. As with the data signals the
integer width is fixed and any remaining bits are used for the fractional portion of the
number. The Phase Signals require an increased integer width to accommodate the
increased range of values they must represent when the Phase Format is set to Radians.

In 2Q8, or **Fix11_7**, format values, **+Pi** and **-Pi** are:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**“01100100100” => 011.00100100 => +3.14**
  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**“10011011100” => 100.11011100 => - 3.14**
  
When Phase Format is set to Scaled Radians PHASE_IN must be in the range:

## Q Numbers Format

An XQN format number is an **1+X+N** bit twos complement binary number; a sign bit
followed by X integer bits followed by an N bit mantissa (fraction). XQN format can be used
to express numbers in the range **(-2X)** to **(2X - 2(-N))**. An equivalent notation using the
System Generator Fix format, defined as Fix*word_length_fractional_length*, would be
**Fix(1+X+N)_N.**

A number using Q15 format is equivalent to a number using **Fix16_15** representation, and a
number in 1Q15 format is equivalent to a number using **Fix17_15** representation.

## Rotate, Translate, Sin, Cos and Atan Functional Configurations

For functional configurations **Rotate**, **Translate**, **Sin**, **Cos**, and **Atan**, it is possible to map
alternative Data Signal formats to the fixed integer width fractional number used by the
CORDIC core.
When the input and output width differ, care must be taken to re-interpret the CORDIC
output.

### Example 8a

The Vector Translation function determines the magnitude and phase angle of a given input
vector **(X_IN, Y_IN)**. The input and output width is set to 10 bits. The standard CORDIC data
representation is **Fix10_8**, the alternative format being mapped onto the input of the
CORDIC is **Fix10_1**.

### Example 8b
If the output width is less than the input width, the CORDIC reduces the fractional width of
the result. When the data output, **X_OUT**, is being re-interpreted to an alternative data
format, the value must be scaled appropriately.

The Vector Translation function determines the magnitude and phase angle of a given input
vector **(X_IN, Y_IN)**. The input and output width is set to 10 bits. The standard CORDIC data
representation is **Fix10_8**, the alternative format being mapped onto the input of the
CORDIC is **Fix10_1**.

# Square Root Functional Configuration

For the Square Root functional configuration it is also possible to map other data formats
onto the data format of the CORDIC but it might be necessary to re-interpret and scale the
output.
The expected output values for each input format are as follows:

- UFix8_7 format: sqrt(0.0625) = **0.25**

- UFix8_1 format: sqrt(4) = **2**

- UFix8_0 format: sqrt(8) = **2.8284**

* The CORDIC output is: **X_OUT value: “00100000”**

The scaling becomes a simple divide by 2, or right shift, of the input, **X_IN**, before applying
it to the square root function. Followed by scaling the output, **X_OUT**, by **2<sup>-M</sup>**.

When **N** is odd, the scaling factor is not an integer power of two. This introduces an
additional output scaling factor of . The example using UFix8_0 demonstrates this with a
scaling factor of 2<sup>-7/2</sup> = 2<sup>-3.5</sup>.

This shows that the CORDIC output value, **0.0010110**, maps to a decimal value of 22
in **UFix8_0** formatting. Applying the output scaling of 2<sup>-3</sup>, or 1/8, gives **2.75**. The loss in
accuracy is due to representing using only 8 bits. If the full accuracy result is used
and then re-interpreted to the alternative data format **(Fix<sup>8_0</sup>)** and then scaled, the correct
result is obtained.

A coarse rotation is performed to rotate the input sample from the full circle into the first
quadrant. (The coarse rotation stage is required as the CORDIC algorithm is only valid over
the first quadrant). An inverse coarse rotation stage rotates the output sample into the
correct quadrant.

The CORDIC algorithm introduces a scale factor to the amplitude of the result, and the
CORDIC core provides the option to automatically compensate for the CORDIC scale
factor.

The CORDIC algorithm can be used to solve several functions as described above. These
functions take different combinations of Cartesian and polar operands. The operands **X_IN**
and **Y_IN** are input using the **S_AXIS_CARTESIAN** channel and the **PHASE_IN** operand is
input using the **S_AXIS_PHASE** input.

**1QN Format Data: Example of a 1Q7 (or Fix9_7) Format Number**

![](/images/frac_table.PNG)

**2QN Format Phase: Example of a 2Q6 (or Fix9_6) Format Number**

![](/images/frac_table2.PNG)

# Linking

To use the C model the user executable must be linked against the correct libraries for the
target platform.

## Linux

The executable must be linked against the following shared object libraries:

- libgmp.so.11

- libIp_cordic_v6_0_bitacc_cmodel.so

Using GCC, linking is typically achieved by adding the following command line options:

    -L. -Wl,-rpath,. -lIp_cordic_v6_0_bitacc_cmodel

This assumes the shared object libraries are in the current directory. If this is not the case,
the -L. option should be changed to specify the library search path to use.

Using GCC, the provided example program run_bitacc_cmodel.c can be compiled and
linked using the following command:

    gcc -x c++ -I. -L. -lIp_cordic_v6_0_bitacc_cmodel -Wl,-rpath,. -o run_bitacc_cmodel run_bitacc_cmodel.c

## Windows

The executable must be linked against the following dynamic link libraries:

- libgmp.dll

- libIp_cordic_v6_0_bitacc_cmodel.dll

Depending on the compiler used, the following import libraries may also be required:

- libgmp.lib

- libIp_cordic_v6_0_bitacc_cmodel.lib

Using **Microsoft Visual Studio**, linking is typically achieved by adding the import libraries to
the Additional Dependencies entry under the Linker section of Project Properties.

# Dependent Libraries

The C model uses the MPIR library. This is governed by the GNU Lesser General Public
License. You can obtain source code for the MPIR library from www.xilinx.com/
guest_resources/gnu/. A pre-compiled MPIR library is provided with the C model, using the
following versions:

* MPIR 2.6.0
* MPIR 2.7.5

As MPIR is a compatible alternative to GMP. The GMP library can be used in place of MPIR.
It is possible to use GMP or MPIR libraries from other sources, for example, compiled from
source code.

GMP and MPIR in particular contain many low level optimizations for specific processors.
The libraries provided are compiled for a generic processor on each platform, using no
optimized processor-specific code. These libraries work on any processor, but run more
slowly than libraries compiled to use optimized processor-specific code. For the fastest
performance, compile libraries from source on the machine on which you run the
executables.

Source code and compilation scripts are provided for the version of MPIR that was used to
compile the provided libraries. Source code and compilation scripts for any version of the
libraries can be obtained from the GMP and MPIR []websites.

**Note**: If compiling MPIR using its configure script (for example, on Linux platforms), use the
`--enable-gmpcompat` option when running the configure script. This generates a **libgmp.so**
library and a **gmp.h** header file that provide full compatibility with the GMP library.




