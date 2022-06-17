# Microchip SBI ECALL Extension

The RISC-V Supervisor Binary Interface (SBI) has the ability to support vendor specific extensions to allow supervisor
mode to perform custom actions in higher privilege modes. These vendor specific extension IDs (EIDs) should to be declared within the #0x09000000 - #0x09FFFFFF range.

The range assigned per each vendor is defined by the vendorid, which relies on JEDEC. For more information please refer to the RISC-V Supervisor Binary Interface [specification](https://github.com/riscv-non-isa/riscv-sbi-doc/blob/master/riscv-sbi.adoc) and RISC-V Priviliged Architectures [specification](https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf).

All Microchip specific SBI function IDs should be defined within Microchip's EID  (#0x09000029).

Maintaining a list of Microchip SBI function IDs (FIDs) important for allowing RISC-V support to play nicely across all BUs using RISC-V.

The list below contains a list of existing Function IDs registered within Microchip's EID region:

| Name                 | FID | Description                                                  | Supported Platform(s) |
| -------------------- | ---- | ------------------------------------------------------------ | ----------------------|
| SBI_EXT_IHC_CTX_INIT | 0x00 | Initialize  a communication channel within the Inter-Hart Communication (IHC) subsystem | MPFS (G5SOC)|
| SBI_EXT_IHC_SEND     | 0x01 | Send a message to an AMP context using the Mi-V Inter-Hart Communication (IHC) subsystem | MPFS (G5SOC) |
| SBI_EXT_IHC_RECEIVE  | 0x02 | Receive a message from an AMP context using the Mi-V Inter-Hart Communication (IHC) subsystem | MPFS (G5SOC) |
| SBI_EXT_HSS_REBOOT   | 0x10 | Perform a reboot command                                     | MPFS (G5SOC) |

## Registering new Microchip Function IDs (FIDs)

To register a new function ID (FID) to use with OpenSBI, please send a pull request to this repository to add a new entry to the table above.

The table entry should specify:

- FID name
- FID address
- FID description
- Platform(s) that support the FID
