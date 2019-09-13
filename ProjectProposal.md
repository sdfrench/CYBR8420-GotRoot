# Project Proposal
## Team GotRoot

  * [Project Repo](https://github.com/caseyschmitz/CYBR8420-GotRoot)
  * [Project Tasks](https://github.com/caseyschmitz/CYBR8420-GotRoot/projects)

## BitWarden
Bitwarden is a free and open-source password management service that stores sensitive information such as website credentials in an encrypted vault. The Bitwarden platform offers a variety of client applications including a web interface, desktop applications, browser extensions, mobile apps, and a CLI (Command Line Interface). It is written primarily in TypeScript and C#. 
	
No matter what device you use, Bitwarden is compatible. It offers native apps for Windows, macOS, Linux, Android, and iOS. Its browser extension supports the expected Chrome, Edge, Firefox, Opera, and Safari, as well as the less-common Vivaldi, Brave, and Tor Browser. It is distributed under the licenses GPLv3 and AGPLv3.
	
Bitwarden is created by 8bit Solutions LLC with its first release on August 10, 2016. The project has 29 contributors.

The app is very popular in its domain. It is ranked 4 stars and appears for the most part of reviews among the top 10 of the best password manager in 2019.

References

https://www.digitaltrends.com/computing/best-password-managers/ 

https://www.cnet.com/news/the-best-password-managers-of-2019/ 

https://en.wikipedia.org/wiki/Bitwarden

### Software Links
---
1. Website
  * [bitwarden.com](https://bitwarden.com)
2. Repo
  * [github.com](https://github.com/bitwarden)
3. Licenses
  * [Server - AGPLv3](https://github.com/bitwarden/server/blob/master/LICENSE.txt)
  * [CLI Client -GPLv3](https://github.com/bitwarden/cli/blob/master/LICENSE.txt)
4. Community
  * [Developer Chat](https://gitter.im/bitwarden/Lobby)
  * [Online Forum](https://community.bitwarden.com/)
5. Contribution Agreement
  * [Terms of Service](https://community.bitwarden.com/tos)
6. Contribution Procedures
  * [Security Issues Disclosure Policy](https://github.com/bitwarden/server/blob/master/SECURITY.md)
  * [Bug Bounty Scopes](https://hackerone.com/bitwarden)
   
### Motivation
---
Today’s consumers not only expect their password manager to be secure, be easy to use, work online, while supporting multiple types of devices. To work online, modern password managers are typically comprised of two major components, the client, which typically supports multiple hardware devices, and the server, which is 'online' (typically in the cloud) and is where the encrypted password vault is stored. This of course presents many more security challenges over the traditional password managers which stores its password vault locally.  In addition, popular password managers often offer features that not only make them easy to use but also help protect against unsafe practices. For exmaple, most password managers offer password generation which allow the user to quickly generate and access strong passwords for each site. 
                   
When you consider how many login credentials a typical user has to keep track of these days, it is easy to see why a good password manager is so important to have. This software is one of the more popular password managers on the market today, so it can inferred it has a good balance between ease of use, features, and security. However, unlike most of the others, it is completely open source. This transpaency is welcomed by savvy consumers who want to know exactly how their sensitive data is stored and protected both on the client and in the cloud. In addition if a consumer is not comfortable storing their sensitive data on the cloud, this software allows them to easily stand up a server stack on their own online host.

We chose this software due to its features, popularity and the fact it is open source, but in large part for its reputation of being secure. Even though this application appears to be well written, and has been subject to third party review, we welcome the challenge of reviewing this application using the perspectives taught in this course. 
 
### Operational Environment
---
Bitwarden is for anyone who wants to have secure online access to their passwords from almost any type of device. We feel that the overwhleming majority of the  users that use this software are going to be individual consumers, however do to the fact the server stack can be easily deployed on an online server, there may be some commercial interest as well.

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
#### [Bitwarden Security Assessment Report in November 8th, 2018](https://cdn.bitwarden.net/misc/Bitwarden%20Security%20Assessment%20Report%20-%20v2.pdf)

* BWN-01-001 – Browser extension autofill only checks top-level website address
When viewing a website, the Bitwarden browser extension provides autofill functions based on the URL shown in the browser’s address bar. It is possible for a website to include additional webpages inside of it by implementing embedded iframes. Bitwarden does not check the URL of these embedded iframes and assumes that they rightfully belong to the “top-level” website.

* BWN-01-006 – Desktop RCE and web vault XSS via login URI when “launched”
Bitwarden allows users to associate a login item with URIs. In the event a scheme/protocol is present on a URI, the Bitwarden desktop application allows users to quickly “launch” these URIs, which opens the associated resource. No additional checks are performed on these URIs to determine if they are of a malicious nature.

* BWN-01-007 – Weak master passwords are allowed
A user’s master password derives the master encryption key which is used to unlock all other data in a user’s Bitwarden vault. Bitwarden allows users to choose any master password. The only restriction in place for a master password that it must be at least 8 characters in length. Due to this lax policy, users can still choose very weak passwords such as “12345678” and “iloveyou”.

* BWN-01-008 – Malicious API server could steal organization encryption keys
While on-boarding new users in an organization, the “confirmation” process performs an exchange of the organization’s encryption key from the organization admin to the new organization user. The organization admin user asks the server for the new user’s public key, which is then used to encrypt the organization key before being transmitted back to the server for storage. If the server is malicious, it can provide the organization admin with a public key that it owns rather than the key owned by the newly on-boarded user, leading to the organization key being leaked to a malicious party.

* BWN-01-010 – Changing the master password does not change encryption keys
Multiple keys are involved with a user’s Bitwarden account:
	1. A “public key” and “private key” is used for the purposes of sharing protected information with other Bitwarden users (via organizations).
	2. An “encryption key” and “mac key” is used to encrypt all data in a Bitwarden user’s vault. These keys also protect the user’s private key (from #1 above).
	3. A “master key” is derived from a Bitwarden user’s master password. The master key is used to protect and unlock the encryption key and mac key (from #2 above).
During a password change operation, only the master key is changed which results in re-encrypting the encryption key and mac key. Since the encryption key and mac key do not change, no other data in the user’s vault is re-encrypted and decrypting existing and new data uses the same encryption key.

#### [Vulnerability report from August 16th, 2017 to October 18th, 2018](https://hackerone.com/bitwarden/scope_versions?change=22716)
| Type		      |	Identifier	         |	Max. Severity   |
|---------------------|--------------------------|----------------------|
| Domain  	      |	bitwarden.com.           |	Critical	|
| Domain  	      |	api.bitwarden.com        |	Critical	|
| Domain  	      |	identity.bitwarden.com   |	Critical	|
| Domain  	      |	bitwarden.com.           |	Critical |
| Domain  	      |	api.bitwarden.com        |	Critical |
| Domain  	      |	identity.bitwarden.com   |	Critical |
| iOS: App Store      | com.8bit.bitwarden	 |	Critical |
| Android: Play Store |	com.x8bit.bitwarden	 |	Critical |
| Domain	      | vault.bitwarden.com	 |	Critical |
| Source code	      |	https://github.com/bitwarden |	Critical |
| Other		      |	https://chrome.google.com/webstore/detail/bitwarden-free-password-m/nngceckbapebfimnlniiiahkandclblb?hl=en) |		 Critical |
| Other		      |	https://addons.mozilla.org/en-US/firefox/addon/bitwarden-password-manager/ | Critical |
| Other		      | https://www.microsoft.com/store/p/bitwarden-free-password-manager/9p6kxl0svnnl | Critical |
| Other |	https://addons.opera.com/extensions/details/bitwarden-free-password-manager/ | Critical | 
| Domain | help.bitwarden.com | Critical |
| Executable | https://github.com/bitwarden/desktop/releases/latest | Critical |
| Executable | https://github.com/bitwarden/cli/releases/latest | Critical |
| Other | https://safari-extensions.apple.com/details/?id=com.bitwarden.safari-LTZ2PFU5D6 | Critical |

