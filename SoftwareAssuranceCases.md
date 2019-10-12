# Assurance Cases for Software Security Engineering

## Assurance Cases

### 1. The System Prevents Network Eavesdropping During Client-Server Communications

![alt text](Images/Assurance%20Cases%20-%20Eavesdropping.png)

**Evidence:**

E3.1 - Bitwarden uses a *Master Password* which is used to derive a *Key Encryption Key.*  Bitwarden states [here](https://help.bitwarden.com/article/what-encryption-is-used/) that it uses PBKDF2 SHA-256 as its derivation function. The *Key Encryption Key* (or Master Key) in turn is what is used to protet the *Private Encryption Key* which is what is used to decrypt secret data. Bitwarden state they use 100,1001 iterations client side and an additional 100,000 server side which appears to be in line with other secret managers. This of course assumes a strong *Master Password* was chosen. 

E4.1 - Bitwarden [states](https://help.bitwarden.com/article/what-encryption-is-used/) that they do not write and cryptographic code and only use reputable cryptographic libraries that maintained by experts.

E5.1 - Bitwarden has had their source code [cryptographically analyzed](https://help.bitwarden.com/article/is-bitwarden-audited/) by a reputable third-party security firm. 

E7.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs shows the [Bitwarden Vault](https://vault.bitwarden.com/#/) is using an acceptable key size of 2048 bits (RSA) however stronger keys of 3072 or 4096 bits are recommended.

E8.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs shows the [Bitwarden Online Vault](https://vault.bitwarden.com/#/) is using Comodo whom is  considered to be a reputable Certificate Authority. 

E9.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs shows the [Bitwarden Online Vault](https://vault.bitwarden.com/#/) it appears that the server does allow connections with weak ciphers (as well as old versions of TLS). However the server is configured to prefer stong cihpers over weak ones. This was most likely done to support old devices. There should be no risk to a user who connects with a modern device that supports strong ciphers and the latest versions of TLS. 

E10.1 - At the time of this document, the [results](https://www.ssllabs.com/ssltest/analyze.html?d=vault.bitwarden.com&hideResults=on&latest) from SSL Labs shows the [Bitwarden Online Vault](https://vault.bitwarden.com/#/) that there are no high risk vulnerabilities present that can be fixed with configuration changes.


### 2. The System is acceptably secure against login attacks

![alt text](Images/Assurance%20Cases%20-%20Logins.png)

**Evidence:**


### 3. The System prevents unauthorized access to secret data

![alt text](Images/Assurance%20Cases%20-%20UnauthAccess.png)

**Evidence:**

### 4. The System adequately limits clear text exposure of user's secret data.


![alt text](Images/Assurance%20Cases%20-%20ClearText.png)

**Evidence:**

### 5. The System adequately ensures the availability of secret data

![alt text](Images/Assurance%20Cases%20-%20Availability.png)

**Evidence:**


## Bitwarden Case Alignment 


## [Project Task Assignments and Collaborations](https://github.com/caseyschmitz/CYBR8420-GotRoot/projects/2)


## Team Reflections
| Issue | Description | Mitigation |
|-------|-------------|------------|
| Communication | We identified communication to be a root issue in our success in working as a team. A primary contributor to this issue is the variety of platforms that we use to communicate. Between in-person, email, Slack, and Github, we find ourselves having both redundant and disjointed conversations. This ultimately leads to wasted time as we try to recover context from previous and ongoing communications. | We determined that we would benefit most from choosing a central channel through which we communicate. Moving forward, Slack may prove to be our best option, as it avoids the issue of multi-threadded conversations (email) and also has convenient integrations with Github. |
| Task Management | Tasks for each milestone, while documented in distinct Github Projects, have not been used in a way that clearly facilitate team responsibilities. We have found that this complicates our workflow and requires frequent status updates from team members regarding the tasks they are working on. Poor task management has contributed to a tendency to start work close to deadlines. | Github Issues have features that we can leverage to better organize the work that goes into each milestone. Beyond simply 'assigning' tasks, there should be discussion of the status of work contributing to particular tasks. Additionally, we should leverage the ability to reference pull-requests in Issues to give more purpose and clarification to what we are delivering for each task and when we are delivering it. |
| Availability | As in any working situation, we have found that our varying availabilities challenge our ability to work 'in sync' on certain tasks. This issue is compounded by inefficient communication and poor task management. | Availability problems can likely be solved by communicating more effectively. We can also benefit from having more face-to-face working sessions, when possible. |
| Tooling | As a team, we have had minor issues with the collaborative features offered by some software that we've utilized for this project. Early challenges with Github were clearly due to a lack of experience with the tool, but Lucidchart has posed some issues that we have yet to resolve. | With Lucidchart, specifically, we are attempting to work in a central document with multiple pages to ensure that all of our work is being done in a single location. We will continue with this strategy moving forward. |
