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

Each register is 32-bits wide, but only the lower X bits are used, where X = *log <sub>2</sub> (Number of triggers allocated for this virtual socket manager)*

Unused bits are ignored on writes, and return 0 on reads. The Trigger to <sup>Reconfigurable</sup>
Module registers can only be accessed when the Virtual Socket Manager is in the shutdown
state. If the Virtual Socket Manager is not in the shutdown state, reads return 0 and writes
are ignored.
