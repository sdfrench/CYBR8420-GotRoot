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

   * Brute Force - One possible way to carry this out is to use a brute force attack on *Mater Password*. This is mitigated by ensuring chosen *Master Password* meets minimum complexity requirements and is considered a strong password which will protect against brute force attacks given a resonable amount of time. 
   
   * Another way an attacker may attempt to reveal *Master Password* is by gaining access to it in clear text form in memory or storage.  
      * This is mitigated by ensuring once the *Master Password* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memeory is deallocated or released.
      * The *Master Password* should never be stored in clear text at rest at in storage any time.

* Information Leakage - The secret manager should reveal any information about secret, such as length, identical to another secret, etc.... that would weaken the strength of the encrypted data
	* No metadata on secret or even hashes of the Master Password should ever be stored at rest in storage or in vault.
	
* Network Evesdropping - The secret manager revealing clear text or information of password
	* The software shold avoid sending the *Master Password* in transit over the network at any time


#### Alignment of Security Requirements

Bitwarden does have a *Master Password* strength validation tool however it is only suggestive, therefore an end user will be allowed to use a weak *Master Password*.

Bitwarden claims to remove *Maaster Password* from memory when it is no longer needed in its clear text form. 

Bitwarden claims that *Master Password* never leaves client application and is never stored at rest or sent over network.


#### UML Diagram

![alt text](Images/Use%20Cases-Master%20Password.png)

### 2. Manage Encryption Key 

#### Use Cases

* Derive Key Encryption Key - *Key Encryption Key* is then derived from *Master Password* when it is created using a strong encryption key derivation algorithm. As noted above, end user can update *Master Password* at any time, but that may not automatically result in the derivation of a new *Key Encryption Key*. End user must be given the option to derive a new encryption key with new master password if they feel encryption key may have been at risk of compromise.

#### Misuse Cases / Security Requirements

* Reveal Vault Keys - The *Key Encryptoin Key* is used to protect the private *vault key/s* which is in turn used to unlock the vault. 

   * Brute Force - Given access to encrypted vault keys, an attacker may attempt to brute force *Master Password* and derive  *Key Encryption Key* in an attempt to reveal vault keys. Protection against this assumes a strong *Master Password* was chosen but furthe mitigates by ensuring a strong key derivation algorithm is used and is implented properly with use of good salt, seed and sufficent amount of iteratoins which ensures time to derive KEK is significant.   

I
#### Alignment of Security Requirements

Bitwarden the PBKDF2 SHA-256 algorithm for password derived encryption key algorithm. It uses open libraries and meet current industry standards for its implementation. (mention iteratoins, seed, salt)

#### UML Diagram

![alt text](Images/Use%20Cases-Key%20Encryption%20Key.png)

### 3. Manage Secrets

#### Use Cases
	Store Secret -> encrypt secret, sync vault
	Retrieve Secret -> decrypt secret
	View Secret |

#### Misuse Cases / Security Requirements
	Steal Secret
		Network Eves-dropping
			Encrypt secret in-transit- encrypt secret client side, encrypt communication
		reveal Clear text - scrub memory (before de-allocation) immediately after use, i.e auth 
			or locked, not always possible due to libs, etc..
			- never store at rest in clear - on disk, vault
			- only encyrpt/decrypt one secret (and its metadata) at a time
	
	Leak Information
		Encrypt all secret metadata
		Ensure encrypted blobs do not match or reveal like data
		
	Corrupt Secret	
		- only encrypt/decrypt one secret at a time
		- Do no re-use old encryption key after changing

n the event the encryption key is changed, the software should be prevent clients from using old encryption key to protect against data corruption. 


#### Alignment of Security Requirements


Bitwarden encrypts all sensitive data client side and only connects to online service over a TLS socket.

Bitwarden will try to log out any clients that still are connected to server however it strongly recommended that the user log out and in of any clients when changing encryption key. Failure to do so may result in data corruption.

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
