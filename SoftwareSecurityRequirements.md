# Requirements for Software Security Engineering

## Backstory

Bitwarden, an open-source online password management service, has earned a good reputation for security. The client's sensative information is encrypted prior to tramsmission to Bitwarden's online vault. Once stored, the encrypted data is only accessable via the client's *Master Password*. The *Master Password* can be changed but not reset. If the user forgets their password, the encrypted items are unrecoverable. Bitwarden supports a multitude of client applications and devices that can be used to sync with the online vault. Although the software is free, some features  are only available with premium accounts and/or family/commercial plans. Due to price, security, and features, such as 2 factor authentication and data sharing, Bitwarden appeals to both the individual user and commercial entities. 

Since the majority of our focus will be client side that is where the majority of the security features are implemented. As mentioned above, this software supports several platforms and we will be primarily focused on the command line client but make look at desktop and browser extensions if we feel there may be something unique to those clients worth pursuing. Mobile devices will be outside the scope of this evaluation however. 


## Essential Data Flows: 

### 1. Manage Master Password 

#### Use Cases

* Create/Update Master Password - End user must choose a *Master Password* upon initial creation of vault. End user can update the *Master Password* at any time during lifetime of vault.

* Submit to unlock vault - The *Master Password* is what is used to unlock the vault going forward and is only known by end user.

#### Misuse Cases / Security Requirements

* Reveal Password - Revealing the *Master Password* would allow an attacker to unlock vault and steal its *Secrets*. 

   * Brute Force - One way to reveal the *Master Password* is to carry out a brute force attack against it. This is mitigated by ensuring chosen *Master Password* meets minimum complexity requirements and is considered strong. This will protect against brute force attacks discovering *Master Password* in a reasonable amount of time. 
   
   * Obtain Clear Text -Another way an attacker may attempt to reveal *Master Password* is by gaining access to it in clear text form in memory or storage.  
      * This is mitigated by ensuring once the *Master Password* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated.
      * The *Master Password* should never be stored in clear text at rest in storage at any time.

* Information Leakage - The software should not reveal any information about *Secret*, which infers that no *Master Password*  data, to include one way hashes of it, should ever be stored at rest in storage or in vault.

* Network Eavesdrop - The secret manager revealing clear text or information of password
	* The software should avoid sending the *Master Password* in transit over the network at any time

#### Alignment of Security Requirements

* Bitwarden does have a *Master Password* strength validation tool however it is only suggestive, therefore an end user will be allowed to use a weak *Master Password*.

* Bitwarden claims to remove *Master Password* from memory when it is no longer needed in its clear text form. 

* Bitwarden claims that *Master Password* never leaves client application and is never stored at rest or sent over network.

#### UML Diagram

![alt text](Images/Use%20Cases-Master%20Password.png)


### 2. Manage Key Encryption Key 

#### Use Cases

* Derive Key Encryption Key - *Key Encryption Key* is derived from *Master Password* when it is created using a strong encryption key derivation algorithm. As noted above, end user can update *Master Password* at any time, but that may not automatically result in the derivation of a new *Key Encryption Key*. However, end user must be given the option to derive a new encryption key with new *Master Password* if they feel encryption key may have been at risk of compromise.

#### Misuse Cases / Security Requirements

* Reveal Vault Keys - The *Key Encryption Key* is used to protect the private *Vault Keys* which are in turn used to decrypt secrets stored in the vault. 

   * Brute Force - Given access to encrypted vault keys, an attacker may attempt to brute force *Master Password* and derive  *Key Encryption Key* in an attempt to reveal vault keys. Protection against this assumes a strong *Master Password* was chosen but further mitigates by ensuring a strong key derivation algorithm is used and implemented properly. A good implementation ensure use of good salt, seed and sufficient amount of iterations which in turn ensures the function takes a significant amount of time to derive the key.
   
* Reveal Key Encryption Key - The software must take extra precautions to protect the KEK when it is revealed.
   
#### Alignment of Security Requirements

* Bitwarden uses the PBKDF2 SHA-256 algorithm for password encryption key derivation algorithm. It uses open libraries and meets current industry standards for its implementation. (mention iterations, seed, salt)

* Bitwarden gives end user the option to derive a new *Key Encryption Key* when a new *Master Password* is chosen.

#### UML Diagram

![alt text](Images/Use%20Cases-Key%20Encryption%20Key.png)

### 3. Manage Secrets

#### Use Cases

* Store Secret - End user has a *Secret* that they want to encrypt and store in vault. A *Secret* is considered to be any data and any corresponding metadata that an end user deems as sensitive and therefore wants to keep it private. This includes data such as passwords, but can include any type of sensitive data, that falls within the size limits, and attributes defined by the software. 

* Retrieve Secret - End user has a *Secret* that they need to decrypt from vault and view in clear text.

#### Misuse Cases / Security Requirements

* Reveal Secret - The attacker is able to reveal the *Secret* the end user wants to keep private.
		
   * Network Evesdrop- If the attacker has direct access to network communications made by the software, they may be able to reveal secret using packet payload data. This can be mitigated by ensuring all data sent over network communications is encrypted and/or data is sent over secure TLS channel.

   * Another way an attacker may attempt to reveal a *Secret* is by gaining access to it in clear text form in memory or storage.  
      * This is mitigated by ensuring once a *Secret* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated.
      * Secret data should never be stored in clear text when at rest. To limit unnecessary exposure the software should encrypt each *Secret* individually preferably with unique encryption keys for each. 

* Information Leakage - The software should not reveal any information about *Secret* even when in encrypted form. For example, an attacker should not be able to compare encrypted blobs and infer the two *Secrets* are identical. 

* Corrupt Data - Software should protect against intentional (attacker) or unintentional (end user) data corruption. For instance if encryption keys should ever be changed, the software must ensure that the old keys are not unintentionally used to encrypt and updates going forward. Also only encrypting/decrypting one *Secret* at a time helps to limit exposure should data be corrupted.
	
#### Alignment of Security Requirements

* Bitwarden encrypts all sensitive data client side and only connects to online service over a TLS socket.

* Bitwarden will try to log out any clients that are still connected to server. However it is strongly recommended that the user log out and in of any clients when changing encryption key. Failure to do so may result in data corruption.

#### UML Diagram
![alt text](Images/Use%20Cases-Secrets.png)

### 4. Manage Passwords (Subset of Secrets) 

#### Use Cases

* Password Generation - When we use *Password* here we are referring to *Passwords* that are for third party sites or applications. Most Security Managers are often referred to as Password Managers even though they are not designed solely for managing *Passwords.*  Having said that, this is still a very important use case for modern secret managers as good password generation is an effective way to ensure the end user does not engage in unsage practices such as sharing passwords and/or using weak passwords.  (why we dont discuss form fill techniques)

#### Misuse Cases / Security Requirements

* Reveal Password -  An attacker is able to reveal the *Password* the end user wants to keep private.

   * Brute force Attack - Attacker is able to reveal end users password using brute force techniques in a relatively short period of time. This mitigate by ensuing a strong password is used by meeting minimum complexity requirements. 
		
   * Replay Attack - An attacker is able to obtain credentials that were disclose as the result of a breach, but because the end user shares the same credentials across multiple sites, the attacker is able to gain access to another site or application unrelated to original breach. This is easily mitigated by not sharing or daisy chaining credentials. 
 
#### Alignment of Security Requirements

* Bitwarden does have a *Password* generation tool that will generate a random password based on end user's minimum requirements, however the complexity of the password generated is completely dependent on those minimum requirements and may allow what is considered to be a weak password to be generated. 

#### UML Diagram
![alt text](Images/Use%20Cases-Passwords.png)

### 5. Manage Sharing 

#### Use Cases
  
* Share Secret - Allow end to share a *Secret* with another user and assign RO or RW permissions for the object and user pair. Owner of object can change permissions or revoke share at any time. *Secret* can only be managed and deleted by *Secret* owner. 

* Access Shared Secret - This is referring to accessing a shared *Secret* when the end user was not the *Secrets* owner. End user may have RW or RO privileges granted to them by *Secret* object owner.

#### Misuse Cases / Security Requirements

* Reveal Secret - The attacker is able to reveal the *Secret* the owner wants to keep private.
		
   * Unauthorized Access - Attacker is able to gain access to shared secret that was not authorized by *Secret* owner. This can be mitigated by using asymmetric encryption where the *Secret* is encrypted with an authorized user's public key. Only the authorized user is able to decrypt the secret using their private key. 
   
* Corrupt Data - The attacker is able to alter the *Secret* rendering it useless to the owner. 

   * Unauthorized Change - Attacker is able to alter *Secret* without proper authorization.  This is mitigated by ensuring the software will not allow changes to a secret unless they have RW authorization. The software must ensure the RO/RW flag for each shared *Secret* is not subject to tampering. 

#### Alignment of Security Requirements



#### UML Diagram
![alt text](Images/Use%20Cases-Sharing.png)
