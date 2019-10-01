# Requirements for Software Security Engineering

## Backstory

NOTE: Scope is open for discussion 

Bitwarden is an open-source online password management service. This software has earned a good reputation for security as all sensitive information is encrypted client side before it is sent to online vault and is recoverable by only using the master password supplied by the end user. In other words, if the end user forgets their master password, the items encrypted in the vault will not be recoverable. This software supports a multitude of client applications and devices that all can be used to sync to the online vault. Although the software is free, some features  are only available with premium accounts and/or family/commercial plans. Due to price, security and features like 2 factor authentication., data sharing, etc.. this software has appeal to both the individual user as well as commercial businesses as well. 

Since the majority of our focus will be client side since that is where the majority of the security features are implemented. As mentioned above, this software supports several platforms and we will be primarily focused on the command line client but make look at desktop and browser extensions if we feel there may be something unique to those clients worth pursuing. Mobile devices will be outside the scope of this evaluation however. 


## Essential Data Flows: 

### 1. Manage Master Password 

#### Use Cases

* Create/Update Master Password - End user must choose a *Master Password* upon initial creation of vault. End user can update the *Master Password* at any time during lifetime of vault.

* Sumbit to unlock vault - The *Master Password* is what is used to unlock the vault going forward and is only known by end user.

#### Misuse Cases / Security Requirements

* Reveal Password - If the software was to Reveal the *Master Password* it would allow an attacker to unlock vault and steal its secrets. 

   * Brute Force - One way to reveal the *Master Password* carry is to carry out a brute force attack against it. This is mitigated by ensuring chosen *Master Password* meets minimum complexity requirements and is considered strong which will protect against brute force attacks given a resonable amount of time. 
   
   * Another way an attacker may attempt to reveal *Master Password* is by gaining access to it in clear text form in memory or storage.  
      * This is mitigated by ensuring once the *Master Password* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated.
      * The *Master Password* should never be stored in clear text at rest in storage at any time.

* Information Leakage - The software should not reveal any information about secret, which infers that no *Master Password*  data, to include one way hashes of it, should ever be stored at rest in storage or in vault.

* Network Evesdropping - The secret manager revealing clear text or information of password
	* The software shold avoid sending the *Master Password* in transit over the network at any time

#### Alignment of Security Requirements

* Bitwarden does have a *Master Password* strength validation tool however it is only suggestive, therefore an end user will be allowed to use a weak *Master Password*.

* Bitwarden claims to remove *Maaster Password* from memory when it is no longer needed in its clear text form. 

* Bitwarden claims that *Master Password* never leaves client application and is never stored at rest or sent over network.

#### UML Diagram

![alt text](Images/Use%20Cases-Master%20Password.png)


### 2. Manage Encryption Key 

#### Use Cases

* Derive Key Encryption Key - *Key Encryption Key* is derived from *Master Password* when it is created using a strong encryption key derivation algorithm. As noted above, end user can update *Master Password* at any time, but that may not automatically result in the derivation of a new *Key Encryption Key*. However, end user must be given the option to derive a new encryption key with new master password if they feel encryption key may have been at risk of compromise.

#### Misuse Cases / Security Requirements

* Reveal Vault Keys - The *Key Encryption Key* is used to protect the private *Vault Keys* which are in turn used to decrypt secrets stored in the vault. 

   * Brute Force - Given access to encrypted vault keys, an attacker may attempt to brute force *Master Password* and derive  *Key Encryption Key* in an attempt to reveal vault keys. Protection against this assumes a strong *Master Password* was chosen but further mitigates by ensuring a strong key derivation algorithm is used and implemented properly. A good implmentation ensure use of good salt, seed and sufficent amount of iterations which in turn ensures the function takes a significant amount of time to derive the key.
   
* Reveal Key Encyrption Key - The software must take extra precautions to protect the KEK when it  
   
#### Alignment of Security Requirements

* Bitwarden uses the PBKDF2 SHA-256 algorithm for password encryption key derivatoin algorithm. It uses open libraries and meets current industry standards for its implementation. (mention iteratoins, seed, salt)

* Bitwarden gives end user the option to derivce a new *Key Encryption Key* when a new *Mastor PAssword* is chosen.

#### UML Diagram

![alt text](Images/Use%20Cases-Key%20Encryption%20Key.png)

### 3. Manage Secrets

#### Use Cases

* Store Secret - End user has a secret that they want to encrypt and store in vault. A secret is considered to be any data and any corresponding metadata that an end user deems as sensitive and therefore wants to keep it private. This includes data such as passwords but can inlcude any type of sensitive data, that falles within the size limits, and attributes defined by the software. 

* Retrieve Secret - End user has a secret that they need to decrypt from vault and view in clear text.

#### Misuse Cases / Security Requirements

* Reveal Secret - The attacker is able to reveal the secret the end user wants to keep private.
		
   * Network Evesdropping - If the attacker has direct access to network commonucatoins made by the software, they may be able to reveal secret using packet payload data. This can be mitigated by ensuring all data sent over netowrk communications is encrypted and/or data is sent over secure TLS channel.

   * Another way an attacker may attempt to reveal *Master Password* is by gaining access to it in clear text form in memory or storage.  
      * This is mitigated by ensuring once the *Master Password* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated.
      * Secret date should never be stored in clear text when at rest. To limit unecesary wexposure the software should encrypt each secret individually prferreabley with unique encryptoin keys for each. 

* Information Leakage - The software should not reveal any information about secret even when in ecnrypted form. For example, an attacker should not be able to compare encypted blobs and infer the two secrets are identical. 

* Currupt Data - Softwware should proctect against either intentional or unintentional data curruptoin. If encryptoins key for secret should ever be changed the software must ensure that the old keys are not unintentional used to encrypt and updates going forward. Also only encrypting/decypting one secret at time helps to limit exposre should data be currupted.
	
#### Alignment of Security Requirements

* Bitwarden encrypts all sensitive data client side and only connects to online service over a TLS socket.

* Bitwarden will try to log out any clients that still are connected to server however it strongly recommended that the user log out and in of any clients when changing encryption key. Failure to do so may result in data corruption.

#### UML Diagram
![alt text](Images/Use%20Cases-Secrets.png)

### 4. Manage Passwords (Subset of Secrets) 

#### Use Cases
Password Generation

#### Misuse Cases / Security Requirements
Attack Goals: (In addition to those defined in secrets)
	Steal Password
		Brute force Attack
			Generate Password - ensure password strength/complexit meets min. requirements
		Replay Attack
			Prevent Reuse of passwords across multiple accounts	Share Secret - assign user to share secret with

n the event the encryption key is changed, the software should be prevent clients from using old encryption key to protect against data corruption. 


#### Alignment of Security Requirements


#### UML Diagram
![alt text](Images/Use%20Cases-Passwords.png)

### 5. Manage Sharing 

#### Use Cases
  Share Secret - assign user, assign perms, remove user
	Access Shared secret

#### Misuse Cases / Security Requirements
	Steal Secret 
		bypass share - secret is encrypted with user pub key
	Change Secret (Integrity)
		Unauthorized change - Assign RO access (How is this enforced)
	Access Secret
		Secret is stored in each users local vault
		Encrypted with their public key (asymmetric encryption)

#### Alignment of Security Requirements


#### UML Diagram
![alt text](Images/Use%20Cases-Sharing.png)
