# noir-rsa

[![CI][ci-shield-rsa]][ci-url-rsa]
[![CI][ci-shield-dkim]][ci-url-dkim]
[![CI][ci-shield-bigint]][ci-url-biguint]
[![MIT License][license-shield]][license-url]

This library contains an implementation of a RSA signature verify for the Noir language. RSA is one of the most widely used digital signature schemes in Web2 applications, such as DKIM email verification, TLS, message encryption etc

The repo contains 2 crates and 1 example circuit:

- rsa-biguint - Fork of shuklaayush's [Noir BigInt](https://github.com/shuklaayush/noir-bigint/) with additional functionality
- rsa - RSA signature verification library
- dkim - DKIM email verification circuit

### RSA BigUint

Fork of [Noir BigInt (commit: cfd409b)](https://github.com/shuklaayush/noir-bigint/tree/cfd409b689c28a14baf25b2d8e3ea1c5bdbd0862) with the following updates:

- Updated constants for a max 2048 bit RSA. The BigInt lib only supports 5 limbs
- Added mulmod and powmod functions
- Use unconstrained division for powmod

### RSA

Currently, Noir RSA supports pkcs1v15, sha256, max RSA modulus of 2048 bits and a max exponent value of 17 bits. Typical RSA modulus sizes are 512, 1024 and 2048 bits. And typically, 65537 is used as the public exponent (which is <2^17).

### DKIM Verification

Verifies a email DKIM signature which signs an email header. The email header comprises of fields such as `From`, `To`, `Subject` and `bh` (hash of the email body encoded in base64). The circuit does the following:

1. Decodes the base64 body hash
2. Hash email body (in bytes) using sha256
3. Compares the hash of the email body with the decoded base64 body hash
4. Hash email header (in bytes) using sha256
5. Verifies the RSA signature matches the public key and the hash of the email header

## Installation

In your Nargo.toml file, add the following dependency:

```
[dependencies]
noir-rsa = { tag = "main", git = "https://github.com/SetProtocol/noir-rsa" }
```

## Usage

The package names of the respective crates are:

- RSA: `rsa`
- RSA BigUint: `biguint`
- DKIM example: `dkim`

Replace `<PACKAGE>` in the following commands with the name of the package you would like to run.

### Running tests

At project root, run:

```
nargo test --package <PACKAGE> --show-output
```

### Proving the DKIM example

At project root, run:

```
nargo prove --package dkim
```

NOTE: The `main` branch only allows RSA bits up to 1024. This is due to proving time being significantly slower if you increase the BigInt limbs to support 2048 bit RSA. Currently proving time for 1024 bits takes ~45 min for the DKIM example. If you need to use a 2048 bit RSA, we support it in the `richard/2048-rsa` [branch](https://github.com/SetProtocol/noir-rsa/tree/richard/2048-rsa)

## Benchmarks

### RSA signature verification

#### CLI

TODO

#### Browser

TODO

### DKIM Example

```
+---------+------------------------+--------------+----------------------+
| Package | Language               | ACIR Opcodes | Backend Circuit Size |
+---------+------------------------+--------------+----------------------+
| dkim    | PLONKCSat { width: 3 } | 1651288      | 3051532              |
+---------+------------------------+--------------+----------------------+
```

#### CLI

On M2 Macbook Air, using Nargo v0.21.0 paired with the default proving backend:

Compiling takes approximately 30 seconds:

```
% time nargo compile --package dkim
nargo compile --package dkim  27.59s user 1.59s system 99% cpu 29.373 total
```

Executing for witness takes approximately 2 mins:

```
% time nargo execute witness --package dkim
nargo execute witness --package dkim  106.11s user 1.05s system 100% cpu 1:47.14 total
```

Executing + proving (as `nargo prove` always re-executes for witness) takes approximately 5 mins:

```
% time nargo prove --package dkim
nargo prove --package dkim  944.88s user 64.86s system 343% cpu 4:54.32 total
```

NOTE: Running `nargo prove` the first time / before `nargo compile` would automatically include program compilation. Subsequent runs without program modifications would make use of the cached artifacts and provide more representative benchmarking results.

#### Browser

TODO

## Ref

- [Noir BigInt](https://github.com/shuklaayush/noir-bigint/)
- [Halo2 RSA](https://github.com/zkemail/halo2-rsa)
- [Circom RSA](https://github.com/zkp-application/circom-rsa-verify)
- [Noir RSA Test Generation Scripts](https://github.com/SetProtocol/noir_rsa_scripts)

## Disclaimer

This is **experimental software** and is provided on an "as is" and "as available" basis. We do **not give any warranties** and will **not be liable for any losses** incurred through any use of this code base.

[ci-shield-dkim]: https://img.shields.io/github/actions/workflow/status/SetProtocol/noir-rsa/test-dkim.yml?branch=main&label=test-dkim
[ci-shield-rsa]: https://img.shields.io/github/actions/workflow/status/SetProtocol/noir-rsa/test-rsa.yml?branch=main&label=test-rsa
[ci-shield-bigint]: https://img.shields.io/github/actions/workflow/status/SetProtocol/noir-rsa/test-rsa-biguint.yml?branch=main&label=test-rsa-biguint
[ci-url-dkim]: https://github.com/SetProtocol/noir-rsa/actions/workflows/test-dkim.yml
[ci-url-rsa]: https://github.com/SetProtocol/noir-rsa/actions/workflows/test-rsa.yml
[ci-url-biguint]: https://github.com/SetProtocol/noir-rsa/actions/workflows/test-rsa-biguint.yml
[license-shield]: https://img.shields.io/badge/License-MIT-green.svg
[license-url]: https://github.com/SetProtocol/noir-rsa/blob/main/LICENSE
