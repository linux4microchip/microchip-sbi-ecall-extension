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
