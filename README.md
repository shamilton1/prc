# prc
The abc core is configured using an AXI4-Lite interface. Rx and Tx core share a common address map and register definitions where possible. Exceptions are highlighted.

The register map is shown below: - Note etc.

| Address Offset Rx | Description | R/W |Address Offset Tx | Description| R/W|
|-----|-----|-----|-----|-----|-----|
| 0x0000 | Version | R | 0x0000 | Version | R |
| 0x0004 | Reset | R/W | 0x0004 | Reset | R/W |
| 0x824 | Lane 0 Link error count| R | 0x824 | Lane 0 Link error count| R |
| 0x825 | Lane 1 Link error count| R | 0x825 | Lane 1 Link error count| R |
| 0x826 | Lane 2 Link error count| R | 0x826 | Lane 2 Link error count| R |
| 0x827 | Lane 3 Link error count| R | 0x827 | Lane 3 Link error count| R |
| 0x828 | Lane 4 Link error count| R | 0x828 | Lane 4 Link error count| R |
| 0x829 | Lane 5 Link error count| R | 0x829 | Lane 5 Link error count| R |
| 0x830 | Lane 6 Link error count| R | 0x830 | Lane 6 Link error count| R |
| 0x831 | Lane 7 Link error count| R | 0x831 | Lane 7 Link error count| R |
| 0x832 | Lane 8 Link error count| R | 0x832 | Lane 8 Link error count| R |
| 0x924 | Lane 0 Link error count| R | 0x924 | Lane 0 Link error count| R |
| 0x925 | Lane 1 Link error count| R | 0x925 | Lane 1 Link error count| R |
| 0x926 | Lane 2 Link error count| R | 0x926 | Lane 2 Link error count| R |
| 0x927 | Lane 3 Link error count| R | 0x927 | Lane 3 Link error count| R |
| 0x928 | Lane 4 Link error count| R | 0x928 | Lane 4 Link error count| R |
| 0x929 | Lane 5 Link error count| R | 0x929 | Lane 5 Link error count| R |
| 0x930 | Lane 6 Link error count| R | 0x930 | Lane 6 Link error count| R |
| 0x931 | Lane 7 Link error count| R | 0x931 | Lane 7 Link error count| R |
| 0x932 | Lane 8 Link error count| R | 0x932 | Lane 8 Link error count| R |
