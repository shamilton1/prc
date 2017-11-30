# prc
The xyz core is configured using an AXI4-Lite interface. Rx and Tx core share a common address map and register definitions where possible. Exceptions are highlighted.

The register map is shown below: 

| Address Offset | Description | R/W |
|-----|-----|-----|
| 0x0000 | Version | R |

