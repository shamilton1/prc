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
| 30:*W*| Reserved | Ignored on read. 0 on write. |
| *W*-1:0 | Trigger | The trigger identifier. The value written to this register is a positive integer that directly specifies the row in the trigger register bank that holds the identifier of the Reconfigurable Module to be loaded by this trigger. Writing this while a trigger is pending overwrites the pending trigger. |
|     |      |• 000 = Shutdown
|     |      |• 010 = Restart with Status
|     |      |• 011 = Proceed
|     |      |• 100 = User Control
|     |      |All other values are reserved.
