## Control Register Format

| Bits | Meaning | Details |
| ---- | ---- | ---- |
| 31:16 |HALFWORD FIELD &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | A 16-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 15:8 | BYTE FIELD | An 8-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 7:0 | CMD | The following commands are defined |
|     |      |• 000 = Shutdown
|     |      |• 010 = Restart with Status
|     |      |• 011 = Proceed
|     |      |• 100 = User Control
|     |      |All other values are reserved.


## SW_Trigger Register

| Bits | Meaning | Details |
| ---- | ---- | ---- |
| 31 |Trigger Pending &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | Ignored on Write. On read, returns: 1 if there is a software trigger pending. 0 if there is no software trigger pending |
| 30:*W *| Reserved | Ignored on read. 0 on write. |
| *W*-1:0 | Trigger | The trigger identifier. The value written to this register is a positive integer that directly specifies the row in the trigger register bank that holds the identifier of the Reconfigurable Module to be loaded by this trigger. Writing this while a trigger is pending overwrites the pending trigger. |
|     |      |• 000 = Shutdown
|     |      |• 010 = Restart with Status
|     |      |• 011 = Proceed
|     |      |• 100 = User Control
|     |      |All other values are reserved.

## Bank 1: Trigger to Reconfigurable Module Registers

The Trigger to Reconfigurable Module registers contain the mapping between the Triggers
and the Reconfigurable Modules to load. There can be more triggers than Reconfigurable
Modules, allowing for the in-field addition of Reconfigurable Modules and/or easier
triggering of the same Reconfigurable Module from multiple sources.

#### X-Bits

Each register is 32 bits wide, but only the lower X bits are used, where X = *log <sub>2</sub> (Number of triggers allocated for this virtual socket manager)*

Unused bits are ignored on writes, and return 0 on reads. The Trigger to *<sup>Reconfigurable</sup>*
Module registers can only be accessed when the Virtual Socket Manager is in the shutdown
state. If the Virtual Socket Manager is not in the shutdown state, reads return 0 and writes
are ignored.

#### UltraScale+

When managing UltraScale+ devices, all registers in this bank are
readable and writable when the Virtual Socket Manager is in the shutdown state.
Reconfigurable Modules for this type of device require two bitstreams, so all bitstream
identifiers are 0 or 1.

When the device to be managed is a 7 series or UltraScale+ device, the *BS_ADDRESS* and
BS_SIZE registers in this bank are readable and writable when the Virtual Socket Manager is in the shutdown state. Reconfigurable Modules for this type of device only require one
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
Error due to the Input Quantization (OQEIQ), and the Output Quantization Error due to
Internal Precision (OQEIP).

OQEIQ is due to the half LSB of quantization noise on the X_IN,Y_IN and PHASE_IN inputs.
In a vector rotation this input quantization noise results in OQEIQ of a half LSB on both the
X_OUT and Y_OUT outputs. In a vector translation this input quantization noise results in
OQEIQ of a half LSB on the X_OUT output; however, OQEIQ on the phase output is
dependent on the ratio (Y_IN/ X_IN). Thus for small X_IN inputs the effect of input
quantization noise on OQEIQ is greatly magnified.

Due to the limited precision of internal calculations, in the CORDIC core the default
internal precision is set such that the accumulated OQEIP is less than 1/2 the OQEIQ. The
internal precision can be manually set to (Input_Width + Output_Width +
log2(Output_Width)). This reduces OQEIP to a half LSB (the phase is calculated to full
precision regardless of the magnitude input vector).

# Phase Signals
The s_axis_phase_tdata Phase operand is PHASE_IN. The m_axis_dout_tdata phase
output is called PHASE_OUT. The phase signals are always represented using a fixed-point
twos complement number with an integer width of 3 bits. As with the data signals the
integer width is fixed and any remaining bits are used for the fractional portion of the
number. The Phase Signals require an increased integer width to accommodate the
increased range of values they must represent when the Phase Format is set to Radians.

In 2Q8, or Fix11_7, format values, +Pi and -Pi are:

  “01100100100” => 011.00100100 => +3.14
  “10011011100” => 100.11011100 => - 3.14
  
When Phase Format is set to Scaled Radians PHASE_IN must be in the range:

