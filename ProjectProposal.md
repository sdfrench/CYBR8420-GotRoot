# Project Proposal
## Team GotRoot ([Github Repo](https://github.com/caseyschmitz/CYBR8420-GotRoot))

## BitWarden
**Project Links**
1. [Website](https://help.bitwarden.com/)
2. [Repo](https://github.com/bitwarden)

### Motivation
---

### Operational Environment
---

### Security Requirements
---
As an online password manager/vault the software must:
  * Ensure all sensitve data (passwords and corresponding metadata) is encrypted when at rest or in storage.
  * Ensure all sensitve data is encrypted before it leaves the device to online storage.
  * Ensure clear text data is only kept in memory and removed when no longer needed.
  * Ensure encrypted data can only be unencrypted with one encryption key.
  * Only use algorithms/libraries for enccyption and key generation that are proven to be secure by current industry standards.
  * Implement a sound key management solution that protects and manages keys thoughout their life cycle, from generation to revocation.
  * Solution should backup encrypted data.
  * Source should be audited.

### Security Features
---
The software has implemented the following security features (taken from bitwarden.com):
  * Data is never stored in its decrypted form on the remote Bitwarden servers or on your local device.
  * Data is never sent to the Bitwarden cloud servers without first being encrypted on your local device.
  * Data is only decrypted in memory, as needed. 
  * Data is fully encrypted and/or hashed before leaving your local device. Bitwarden servers only store encrypted and hashed data, therefore no one, including the Bitwarden team, can ever see, read, or reverse engineer the encrypted data to gain access to the real data in clear text.
  * Data is encyrpted using AES 256 bit encryption which is used by the US government and other government agencies around the world for protecting top secret data.
  * PBKDF2 SHA-256 is used to derive the encryption key from your master password. This key is then salted and hashed and passes through a minimum of 200,001 iterations before it stored online.
  * Only invokes crypto from popular and reputable crypto libraries that are written and maintained by cryptography experts. 
  * By default data is stored on Azure cloud. Copies of encypted data are stored on the local device as well.
  * All source code can by audited by the public on github.com and has been audited by third-party.

### History
---
