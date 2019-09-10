# Project Proposal
## Team GotRoot ([Github Repo](https://github.com/caseyschmitz/CYBR8420-GotRoot))

## BitWarden
Bitwarden is a free and open-source password management service that stores sensitive information such as website credentials in an encrypted vault. The Bitwarden platform offers a variety of client applications including a web interface, desktop applications, browser extensions, mobile apps, and a CLI (Command Line Interface). It is written primarily in TypeScript and C#. 
	
No matter what device you use, Bitwarden is compatible. It offers native apps for Windows, macOS, Linux, Android, and iOS. Its browser extension supports the expected Chrome, Edge, Firefox, Opera, and Safari, as well as the less-common Vivaldi, Brave, and Tor Browser. It is distributed under the licenses GPLv3 and AGPLv3.
	
Bitwarden is created by 8bit Solutions LLC with its first release on August 10, 2016. The project has 29 contributors.

The app is very popular in its domain. It is ranked 4 stars and appears for the most part of reviews among the top 10 of the best password manager in 2019.

References

https://www.digitaltrends.com/computing/best-password-managers/ 

https://www.cnet.com/news/the-best-password-managers-of-2019/ 

https://en.wikipedia.org/wiki/Bitwarden


### Project Links
---
1. [Website](https://bitwarden.com)
2. [Repo](https://github.com/bitwarden)
3. Licenses
  * [Server](https://github.com/bitwarden/server/blob/master/LICENSE.txt)
  * [CLI](https://github.com/bitwarden/cli/blob/master/LICENSE.txt)
4. Community
  * [Developer Chat](https://gitter.im/bitwarden/Lobby)
  * [Online Forum](https://community.bitwarden.com/)
5. [Contribution Agreement](https://community.bitwarden.com/tos)

### Motivation
---
Bitwarden is one of the premier passwword managers on the marktet today. Not only do today's consumers expect their password manager to be secure, it must be easy to use, and work with multiple onlline devices. As a result this infers that the password manager is comprised of two components, the client which must be written for multiple hardware devices, but also will have a server presense usually on the cloud where the encrypted passwrod vault iskept. This of cousre presents many security challanges but as mentioned above, Bitwarden seems to be up to the ask. On top of that Bitwarden is no black box, as it is open source and completely transparent in its implementation. Dont want to store your vault on the cloud? Bitwarden has got you covered there too. Anyone can host the entire server stack on their own online host with several operating systems to choose from. 
 
### Operational Environment
---
Bitwarden is for anyone who wants to have access to their passwords from almost any online device. Most users are more than likely to be individual consumers, however do to the fact the server stack can be deployed to an online server, there may be some commercial interest as well.

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
