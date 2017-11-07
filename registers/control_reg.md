# Control Register Format

| Bits | Meaning | Details |
| ---- | ---- | ---- |
| 31:16 | HALFWORD FIELD | A 16-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 15:8 | BYTE FIELD | An 8-bit field containing extra information for the selected command. See the command descriptions for more information. |
| 7:0 | CMD | The following commands are defined |
|     |      |• 000 = Shutdown
|     |      |• 010 = Restart with Status
|     |      |• 011 = Proceed
|     |      |• 100 = User Control
|     |      |All other values are reserved.
