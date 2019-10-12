# Assurance Cases for Software Security Engineering

## Assurance Cases

### 1. The System Prevents Network Eavesdropping During Client-Server Communications

![alt text](Images/Assurance%20Cases%20-%20Eavesdropping.png)

**Evidence:**

E3.1 - Bitwarden uses a *Master Password* which is used to derive a *Key Encryption Key.*  Bitwarden states [here](https://help.bitwarden.com/article/what-encryption-is-used/) that it uses PBKDF2 SHA-256 as its derivation function. The *Key Encryption Key* (or Master Key) is used to protect the *Private Encryption Key* which is used to decrypt secret data. Bitwarden states that they run 100,001 iterations client side and another 100,000 on the server side. This appears to be in line with other secret managers however it may be borderline with today's computing power. This also assumes a strong *Master Password* was chosen. 

E4.1 - Bitwarden [states](https://help.bitwarden.com/article/what-encryption-is-used/) that they do not write any cryptographic code, and only use reputable cryptographic libraries that are maintained by experts.

E5.1 - Bitwarden has had their source code [cryptographically analyzed](https://help.bitwarden.com/article/is-bitwarden-audited/) by a reputable third-party security firm. 

E7.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs show that [Bitwarden Vault](https://vault.bitwarden.com/#/) is using an acceptable key size of 2048 bits (RSA). However, stronger keys of 3072 or 4096 bits are recommended.

E8.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs show that [Bitwarden Online Vault](https://vault.bitwarden.com/#/) is using Comodo, which is considered to be a reputable Certificate Authority. 

E9.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs show that [Bitwarden Online Vault](https://vault.bitwarden.com/#/) does allow connections with weak ciphers (as well as old versions of TLS). However, the server is configured to prefer stong ciphers. This was most likely done to support old devices. There should be no risk to a user who connects with a modern device that supports strong ciphers and the latest versions of TLS. 

E10.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs shows that [Bitwarden Online Vault](https://vault.bitwarden.com/#/) has no high risk vulnerabilities present that can be fixed with configuration changes.

**Alignment:** 7|7 ✔️

### 2. The System is acceptably secure against login attacks

![alt text](Images/Assurance%20Cases%20-%20Logins.png)

**Evidence:**

E3.1 - Confirmed through experimentation: Bitwarden enforces a minimum size of 8 characters when creating a *Master Password*. However, according to Bitwarden's *Master Password* strength meter, an 8 character is still considered weak even with a good variety of character types. Minimum password size should be adjusted to 9 characters.

E4.1 - Bitwarden has a *Master Password* strength meter that shows the strength of the chosen password as the user types. The strength meter seems to be well implemented. Unfortunately it is only suggestive, and will allow a user to select a weak password.

E5.1 - (See Case 2: E3.1 and E4.1)

E6.1 - (See Case 1: E3.1)

E9.1 - MFA options offered by Bitwarden include [authenticator app](https://help.bitwarden.com/article/setup-two-step-login-authenticator/), [email](https://help.bitwarden.com/article/setup-two-step-login-email/), [Duo Security](https://help.bitwarden.com/article/setup-two-step-login-duo/), [YubiKey](https://help.bitwarden.com/article/setup-two-step-login-yubikey/), and [FIDO U2F](https://help.bitwarden.com/article/setup-two-step-login-u2f/).

E10.1 - Duo Security does allow [SMS](https://guide.duo.com/prompt) to be used as an authentication prompt. SMS is [no longer recommended](https://pages.nist.gov/800-63-3/sp800-63b.html) by NIST as a Two-Factor Authentication mechanism, citing weaknesses against Social Engineering and Endpoint Compromise.

**Alignment:** 2|4 ⚠️

### 3. The System prevents unauthorized access to secret data

![alt text](Images/Assurance%20Cases%20-%20UnauthAccess.png)

**Evidence:**

E3.1 - Bitwarden [states](https://help.bitwarden.com/article/forgot-master-password/) that the corresponding *Master Password* is the only way to unlock a vault. In addition, the *Master Password* is never stored or sent in the clear to the online vault according to [BitWarden](https://help.bitwarden.com/article/can-bitwarden-see-my-passwords/). 

E4.1 - Bitwarden [states](https://help.bitwarden.com/article/how-is-data-securely-transmitted-and-stored/) that sensitive data is never stored in its decrypted form on the client or server.

E5.1 - Bitwarden [states](https://help.bitwarden.com/article/how-is-data-securely-transmitted-and-stored/) that data is only decrypted as needed. Bitwarden reloads if left idle for 10 seconds, thereby [sanitizing](https://community.bitwarden.com/t/any-thoughts-on-this-independent-security-study-apparently-all-the-major-pw-managers-are-insecure/4663/5) the memory.

E6.1 - Bitwarden [states](https://help.bitwarden.com/article/what-encryption-is-used/) that no data, including *Master Password*, is sent unencrypted to cloud servers.

E8.1 - (See Case 1: E3.1)

E9.1 - (See Case 3: E5.1) 

E10.1 - Bitwarden discloses [here](https://help.bitwarden.com/article/what-information-is-encrypted/) what data won't be encrypted. Any metadata related to a secret will be encrypted.


### 4. The System adequately limits clear text exposure of user's secret data.


![alt text](Images/Assurance%20Cases%20-%20ClearText.png)

**Evidence:**

E2.1 - Bitwarden clears any sensitive vault data, as well as encryption keys from memory whenever the application enters a locked state. We also use other techniques, such as reloading the process after 10 seconds of inactivity on the lock screen, to make sure any managed memory addresses which have not yet been garbage collected are also purged.

E3.1 - Bitwarden uses fully audited open-source end-to-end encryption (e2ee). Which is as good a guarantee that it is secure and private as it’s possible to get. The only way to decrypt your data is by using the correct master password, which is not recoverable should you forget it. So don’t. [New Fonctionality release](https://blog.bitwarden.com/autofill-improvements-come-to-ios-12-android-9-3072576c7411).

E5.1 - When your devices sync with the Bitwarden cloud servers, a copy of the encrypted data is downloaded and securely stored to your local device. Whenever you use the Bitwarden apps or extensions your data is decrypted only in memory as needed. Data is never stored in its decrypted form on the remote Bitwarden servers or on your local device.

E6.1 - (See E3.1 case 4)


E8.1 - In November 2018 a crowdfunded independent security audit by Cure53 found no major issues with the software. Some non-critical issues were discovered, the most important of which were patched immediately. We can only presume that developer Kyle has been working hard this last year to fix any additional issues raised by the audit. 
### 5. The System adequately ensures the availability of secret data

![alt text](Images/Assurance%20Cases%20-%20Availability.png)

**Evidence:**

E3.1 - Bitwarden processes and stores data in the [Microsoft Azure cloud](https://help.bitwarden.com/article/cloud-server-security/). Trust is placed in Microsoft's ability to ensure uptime, scalability, and security.

E4.1 - Cloud [backups](https://help.bitwarden.com/article/backup-on-premise/) are managed by Bitwarden. However, backup procedures are the user's responibility in a self-hosted environment. Bitwarden's Docker containers use volume mapping to store data in the *./bwdata* directory, which is necessary to backup and recover data. Bitwarden is configured to automatically perform nightly backups of the *mssql* container, which are kept in *./bwdata/mssql/backups* for thirty days.

E5.1 - Cloud [backups](https://help.bitwarden.com/article/backup-on-premise/) are managed by Bitwarden.

E6.1 - Bitwarden's server-side ability to [scale](https://help.bitwarden.com/article/backup-on-premise/) make it inherently resilient to attacks against resources. Additionally, Bitwarden implements [IP-based blacklisting](https://www.reddit.com/r/Bitwarden/comments/9ld211/no_account_lockout_on_many_invalid_attempts/e75xxu1/?context=8&depth=9) against clients that fail too many logins in a period of time.


## Bitwarden Case Alignment 


## [Project Task Assignments and Collaborations](https://github.com/caseyschmitz/CYBR8420-GotRoot/projects/2)


## Team Reflections
| Issue | Description | Mitigation |
|-------|-------------|------------|
| Communication | We identified communication to be a root issue in our success in working as a team. A primary contributor to this issue is the variety of platforms that we use to communicate. Between in-person, email, Slack, and Github, we find ourselves having both redundant and disjointed conversations. This ultimately leads to wasted time as we try to recover context from previous and ongoing communications. | We determined that we would benefit most from choosing a central channel through which we communicate. Moving forward, Slack may prove to be our best option, as it avoids the issue of multi-threadded conversations (email) and also has convenient integrations with Github. |
| Task Management | Tasks for each milestone, while documented in distinct Github Projects, have not been used in a way that clearly facilitate team responsibilities. We have found that this complicates our workflow and requires frequent status updates from team members regarding the tasks they are working on. Poor task management has contributed to a tendency to start work close to deadlines. | Github Issues have features that we can leverage to better organize the work that goes into each milestone. Beyond simply 'assigning' tasks, there should be discussion of the status of work contributing to particular tasks. Additionally, we should leverage the ability to reference pull-requests in Issues to give more purpose and clarification to what we are delivering for each task and when we are delivering it. |
| Availability | As in any working situation, we have found that our varying availabilities challenge our ability to work 'in sync' on certain tasks. This issue is compounded by inefficient communication and poor task management. | Availability problems can likely be solved by communicating more effectively. We can also benefit from having more face-to-face working sessions, when possible. |
| Tooling | As a team, we have had minor issues with the collaborative features offered by some software that we've utilized for this project. Early challenges with Github were clearly due to a lack of experience with the tool, but Lucidchart has posed some issues that we have yet to resolve. | With Lucidchart, specifically, we are attempting to work in a central document with multiple pages to ensure that all of our work is being done in a single location. We will continue with this strategy moving forward. |
