# Microchip SBI ECALL Extension

The RISC-V Supervisor Binary Interface (SBI) has the ability to support vendor
specific extensions to allow supervisor mode to perform custom actions in
higher privilege modes. These vendor specific extension IDs (EIDs) should be
declared within the #0x09000000 - #0x09FFFFFF range.

The range assigned per each vendor is defined by the vendorid, which relies on
JEDEC. For more information please refer to the [RISC-V Supervisor Binary
Interface specification][1] and to the [RISC-V Privileged Architecture
specification][2].

All Microchip specific SBI function IDs should be defined within Microchip's
EID (#0x09000029).

Maintaining a list of Microchip SBI function IDs (FIDs) is important for
allowing RISC-V support to play nicely across all BUs using RISC-V. The list
below contains a list of existing Function IDs registered within Microchip's
EID region:

| Name                          | FID  | Supported Platform(s) |
| ----------------------------- | ---- | ----------------------|
| SBI_EXT_IHC_CTX_INIT          | 0x00 | MPFS (G5SOC)          |
| SBI_EXT_IHC_SEND              | 0x01 | MPFS (G5SOC)          |
| SBI_EXT_IHC_RECEIVE           | 0x02 | MPFS (G5SOC)          |
| SBI_EXT_RPROC_STATE           | 0x03 | MPFS (G5SOC)          |
| SBI_EXT_RPROC_START           | 0x04 | MPFS (G5SOC)          |
| SBI_EXT_RPROC_STOP            | 0x05 | MPFS (G5SOC)          |
| SBI_EXT_HSS_REBOOT            | 0x10 | MPFS (G5SOC)          |
| SBI_EXT_CRYPTO_INIT           | 0x11 | MPFS (G5SOC)          |
| SBI_EXT_CRYPTO_SERVICES_PROBE | 0x12 | MPFS (G5SOC)          |
| SBI_EXT_CRYPTO_SERVICES       | 0x13 | MPFS (G5SOC)          |
| SBI_EXT_IPC_PROBE             | 0x100| MPFS (G5SOC)          |
| SBI_EXT_IPC_CH_INIT           | 0x101| MPFS (G5SOC)          |
| SBI_EXT_IPC_SEND              | 0x102| MPFS (G5SOC)          |
| SBI_EXT_IPC_RECEIVE           | 0x103| MPFS (G5SOC)          |
| SBI_EXT_IPC_STATUS            | 0x104| MPFS (G5SOC)          |

## Registering new Microchip Function IDs (FIDs)

To register a new function ID (FID) to use with OpenSBI, please send a pull
request to this repository to add a new entry to the table above. The table
entry should specify:

- FID name
- FID address
- FID description
- Platform(s) that support the FID


## Function ID Descriptions

Per the specifications, SBI functions must return a pair of values in `a0` and
`a1`, with `a0` returning an error code. This is analogous to returning the C
structure:

```c
struct sbiret {
    long error;
    long value;
};
```

Please refer to the [RISC-V Supervisor Binary Interface specification][1] for
more information, including a [list of standard error codes][3].
The descriptions in this document are accompanied by a C pseudocode function
prototypes, where the arguments represent values passed in `a0` and `a1`
respectively.

[1]: https://github.com/riscv-non-isa/riscv-sbi-doc/blob/master/riscv-sbi.adoc
[2]: https://github.com/riscv/riscv-isa-manual/releases/download/Priv-v1.12/riscv-privileged-20211203.pdf
[3]: https://github.com/riscv-non-isa/riscv-sbi-doc/blob/master/riscv-sbi.adoc#table_standard_sbi_errors

### SBI_EXT_IHC_CTX_INIT (FID 0x0)

```c
struct sbiret ihc_context_init(unsigned long context_id);
```

Initialise a communication channel within the Inter-Hart Communication (IHC)
subsystem for the context denoted by `context_id`.

### SBI_EXT_IHC_SEND (FID 0x01)

```c
struct sbiret ihc_send_to_remote_context(unsigned long context_id, unsigned long message_addr);
```

Send a message, located at `message_addr` in memory, to the AMP context denoted
by `context_id` using the Mi-V Inter-Hart Communication (IHC) subsystem.

### SBI_EXT_IHC_RECEIVE (FID 0x02)

```c
struct sbiret ihc_receive_from_remote_context(unsigned long context_id, unsigned long message_addr);
```

Receive a message from the AMP context denoted by `context_id` using the Mi-V
Inter-Hart Communication (IHC) subsystem and store it at `message_addr` in
memory.

### SBI_EXT_RPROC_STATE (FID 0x03)

```c
struct sbiret get_remote_context_state(unsigned long context_id);
```

Check if the AMP context denoted by `context_id` was started by the HSS
(early boot) or by another application (late boot), for example using the
remoteproc framework in Linux.

### SBI_EXT_RPROC_START (FID 0x04)

```c
struct sbiret start_remote_context(unsigned long context_id, unsigned long payload_addr);
```

Load & start the remote AMP context denoted by `context_id`'s firmware, that
has been loaded to `payload_addr` in memory.

### SBI_EXT_RPROC_STOP (FID 0x05)

```c
struct sbiret stop_remote_context(unsigned long context_id);
```

Stop the firmware running in the remote AMP context denoted by `context_id` by
setting the harts associated to that context hart(s) in `wfi`.

### SBI_EXT_HSS_REBOOT (FID 0x10)

```c
struct sbiret hss_reboot(void);
```

Reboot the HSS. This is particularly useful on harts that skip OpenSBI, as the
HSM extension's eponymous state machine will not have correctly tracked the
state of the hart correctly.

### SBI_EXT_CRYPTO_INIT (FID 0x11)

```c
struct sbiret crypto_init(unsigned int toggle);
```

Initialise the cryptographic hardware engine and enable or disable the crypto clock based on the `toggle`.

### SBI_EXT_CRYPTO_SERVICES_PROBE (FID 0x12)

```c
struct sbiret crypto_services_probe(unsigned long crypto_addr);
```
Send a crypto service probe request, to get the supported crypto services from the firmware and stored at `crypto_addr` in memory.

The layout of `crypto_addr` in memory.

```c
struct mchp_crypto_info {
	unsigned int   services;
	unsigned long  cipher_algo;
	unsigned int   aead_algo;
	unsigned int   hash_algo;
	unsigned long  mac_algo;
	unsigned long  akcipher_algo;
	unsigned int   nrbg_algo;
};
```
#### Services:

|bit-31:8|bit-7|bit-6|bit-5|bit-4|bit-3|bit-2|bit-1|bit-0|
|:-------|:----|:----|:----|:----|:----|:----|:----|:----|
|RESERVED|NRBG |ECDSA|DSA  |RSA  |MAC  |HASH |AEAD |AES  |


#### Symmetric cipher(AES) algorithms(cipher_algo):

|bit-63:5|bit-4|bit-3|bit-2|bit-1|bit-0|
|:-------|:----|:----|:----|:----|:----|
|RESERVED| CTR | CFB | OFB | CBC | ECB |


#### AEAD algorithms(aead_algo):

|bit-31:2|bit-1|bit-0|
|:-------|:----|:----|
|RESERVED|CCM  |GCM  |

#### HASH algorithms(hash_algo):

|bit-31:7|bit-6     |bit-5     |bit-4 |bit-3 |bit-2 |bit-1 |bit-0|
|:-------|:---------|:---------|:-----|:-----|:-----|:-----|:----|
|RESERVED|SHA512_256|SHA512_224|SHA512|SHA384|SHA256|SHA224|SHA1 |

#### MAC algorithms(mac_algo):

|bit-9:7 |bit-6     |bit-5     |bit-4 |bit-3 |bit-2 |bit-1 |bit-0|
|:------ |:---------|:---------|:-----|:-----|:-----|:-----|:----|
|RESERVED|SHA512_256|SHA512_224|SHA512|SHA384|SHA256|SHA224|SHA1 |

|bit-63:14|bit-13 |bit-12    |bit-11    |bit-10    |
|:--------|:------|:---------|:---------|:---------|
|RESERVED |AESGMAC|AESCMAC256|AESCMAC192|AESCMAC128|

#### Asymmetric cipher algorithms(akcipher_algo):

|bit-63:5|bit-4|bit-3 |bit-2|bit-1|bit-0|
|:-------|:----|:-----|:----|:----|:----|
|RESERVED| ECDH| ECDSA| DH  | RSA | DSA |

#### Random Number Generation(nrbg_algo):

|bit-31:2|bit-1|bit-0|
|:-------|:----|:----|
|RESERVED|DRBG |NRBG |

### SBI_EXT_CRYPTO_SERVICES (FID 0x013)

```c
struct sbiret crypto_services(unsigned int service, unsigned long crypto_addr);
```
Send a crypto service request, the `service` such as AES, AEAD, HASH, MAC, DSA, RSA, ECDSA, NRBG and related data located at `crypto_addr` in memory to the cryptographic hardware engine for encryption/decryption, hash, signature/verification.

Example: The AES layout of `crypto_addr` in memory.

```c
struct mchp_crypto_aes_req {
	unsigned long source_phys_addr;
	unsigned long initialisation_vector_phys_addr;
	unsigned long symmetric_key_phys_addr;
	unsigned long destination_phys_addr;
	unsigned long size_of_cipher_data;
};
```

### SBI_EXT_IPC_PROBE (FID 0x100)

```c
struct sbiret ipc_probe(unsigned long message_addr);
```

Send a probe request to get the following information about the IPC:

- `hw_type`: specifies the IPC implementation available in the hardware
- `num_channels`: specifies the number of IPC channels available in the hardware

The request returns a struct located in `message_addr` which contains the following information:

```c
struct mchp_ipc_probe {
    enum ipc_hw hw_type;
    u8 num_channels;
};
```

where `ipc_hw` is defined as shown below:

```c
enum ipc_hw {
    MIV_IHC,
    RESERVED1,
    RESERVED2,
};
```

### SBI_EXT_IPC_CH_INIT (FID 0x101)

```c
struct sbiret ipc_ch_init(unsigned long channel_id, unsigned long message_addr);
```

Initialise a communication channel within the Inter-Processor Communication (IPC) for the channel
denoted by `channel_id`.

The request returns a struct in `message_addr` containing the maximum message size in bytes of the
corresponding channel:

```c
struct mchp_ipc_init {
    u16 max_msg_size;
};
```

### SBI_EXT_IPC_SEND (FID 0x102)

```c
struct sbiret ipc_send(unsigned long channel_id, unsigned long message_addr);
```

Send a message, located at `message_addr` address to the channel denoted
by `channel_id` using the Inter-Processor Communication (IPC).

The layout of `message_addr` in memory is:

```c
struct mchp_ipc_sbi_msg {
    u64 buf_addr;
    u16 size;
    u8 irq_type;
};
```

- `buf_addr`: specifies the physical address of the buffer where the data will be copied.
- `size`: Indicates the maximum size (in bytes) of the data that can be stored in the buffer pointed to by `buf_addr`.
- `irq_type`: Not applicable for sending messages. This field is only used when receiving messages (refer to SBI_EXT_IPC_RECEIVE for details).

### SBI_EXT_IPC_RECEIVE (FID 0x103)

```c
struct sbiret ipc_receive(unsigned long channel_id, unsigned long message_addr);
```

Receive a message from the associated processor through a channel id denoted
by `channel_id` using the Inter-Processor Communication (IPC).

The request should contain a struct located in the address denoted by `message_addr` and should
contain the following information:

```c
struct mchp_ipc_sbi_msg {
    u64 buf_addr;
    u16 size;
    u8 irq_type;
};
```

- `buf_addr`: physical address where the received data should be copied
- `size`: Indicates the maximum size (in bytes) of the data that can be stored in the buffer pointed to by `buf_addr`.
- `irq_type`: A mask representing the types of interrupts that triggered the reception of this message.

The irq_type mask values are defined as follows:

```c
enum ipc_irq_type {
    IPC_OPS_NOT_SUPPORTED    = 1,
    IPC_MP_IRQ               = 2,
    IPC_MC_IRQ               = 4,
};
```

where `IPC_MP_IRQ` and `IPC_MC_IRQ` represent "message present" and "message clear" interrupts, respectively.

The `IPC_OPS_NOT_SUPPORTED` mask value indicates that interrupt aggregation is not supported in the
IPC implementation used.

### SBI_EXT_IPC_STATUS (FID 0x104)

```c
struct sbiret ipc_get_status(unsigned long message_addr);
```

Note: This function ID is currently implemented for G5 SoC using the MIV_IHC IP. However,
it can be extended by the user to other platforms if needed.

#### MPFS (G5SOC) Implementation

The request returns a struct in `message_addr` containing the message present and message clear
interrupt status for all channels associated with a cluster:

```c
struct mchp_ipc_status {
    u32 status;
    u8 cluster;
};
```

The `status` field is a 32-bit value that indicates the interrupt status for all channels associated
to a cluster. The bits are organized in alternating format, where each pair of bits represents
the status of the message present and message clear interrupts for each cluster/hart
(from hart 0 to hart 5). Each cluster can have up to 5 fixed channels associated.

| Bit   | Name                  |
|-------|-----------------------|
| 32:12 | RESERVED              |
| 11    | MSG_CLEARED_ACK_HART5 |
| 10    | MSG_CLEARED_MP_HART5  |
| 9     | MSG_CLEARED_ACK_HART4 |
| 8     | MSG_CLEARED_MP_HART4  |
| 7     | MSG_CLEARED_ACK_HART3 |
| 6     | MSG_CLEARED_MP_HART3  |
| 5     | MSG_CLEARED_ACK_HART2 |
| 4     | MSG_CLEARED_MP_HART2  |
| 3     | MSG_CLEARED_ACK_HART1 |
| 2     | MSG_CLEARED_MP_HART1  |
| 1     | MSG_CLEARED_ACK_HART0 |
| 0     | MSG_CLEARED_MP_HART0  |

The `cluster` variable specifies the cluster instance that originated the interrupt.
