# Requirements for Software Security Engineering

## Background

Bitwarden, an open-source online password management service, has earned a good reputation for security. The client's sensative information is encrypted prior to tramsmission to Bitwarden's online vault. Once stored, the encrypted data is only accessable via the client's *Master Password*. The *Master Password* can be changed but not reset. If the user forgets their password, the encrypted items are unrecoverable. Bitwarden supports a multitude of client applications and devices that can be used to sync with the online vault. Although the software is free, some features  are only available with premium accounts and/or family/commercial plans. Due to price, security, and features, such as 2 factor authentication and data sharing, Bitwarden appeals to both the individual user and commercial entities. 

Since the majority of our focus will be client side that is where the majority of the security features are implemented. As mentioned above, this software supports several platforms and we will be primarily focused on the command line client but make look at desktop and browser extensions if we feel there may be something unique to those clients worth pursuing. Mobile devices will be outside the scope of this evaluation however. 


## Essential Data Flows: 


### 1. Manage Master Password

#### Backstory

This data flow concerns the management of the *Master Password.* The *Master Password* is the password used to ultimately "unlock" a user's vault giving them access to the *Secrets.*

#### Use Cases

* Create/Update Master Password - End user must choose a *Master Password* upon initial creation of vault. End user can update the *Master Password* at any time after during the lifetime of vault.

* Submit to unlock vault - The *Master Password* is submitted to application to unlock the vault going forward and is only known by end user.

#### Misuse Cases / Security Requirements

* Reveal Password - Revealing the *Master Password* would allow an attacker to unlock vault and reveal its *Secrets* to an unauthorized user. 

   * Brute Force - One way to reveal the *Master Password* is to carry out a brute force attack against it. This is mitigated by ensuring chosen *Master Password* meets minimum complexity requirements and therefore is considered strong. This will protect against brute force attacks from discovering *Master Password* in a reasonable amount of time. 
   
   * Obtain Clear Text - Another way an attacker may attempt to reveal *Master Password* is by gaining access to it in clear text form in either memory or storage. Mitigation strategies are as follows:
      * Sanitize Memory - Ensure that once the *Master Secret* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated if applicable.
      * The *Master Password* should never be stored in clear text at rest in storage at any time.

* Information Leakage - The software should not reveal any information about *Secret.* For example, hashes or encrypted blobs of *Master Password* must not Leak any information about it, such as its length.

* Network Eavesdrop - The secret manager revealing clear text or information of password
	* The software should avoid sending the *Master Password* over the network in clear text at any time

#### Alignment of Security Requirements

* Bitwarden does have a *Master Password* strength validation tool however it is only suggestive, therefore an end user will be allowed to choose a weak *Master Password*.

* Bitwarden [states](https://help.bitwarden.com/article/forgot-master-password/) *Master Password* is securely encrypted and cannot be reverse engineered by Bitwarden team. 

* Btiwarden [states](https://help.bitwarden.com/article/password-salt-hash/) claims to use one way hashes with proper salt to prevent someone from reverse engineering *Master Password.*

* Bitwarden [states](https://help.bitwarden.com/article/can-bitwarden-see-my-passwords/) that *Master Password* clear text never leaves client side application.


#### UML Diagram

![alt text](Images/Use%20Cases-Master%20Password.png)


### 2. Manage Key Encryption Key

#### Backstory

This data flow concerns the management of the *Key Encryption Key.*  The *Key Encryption Key* is derived from *Master Password* and is used to encrypt end user's private key which in turn is used decrypt vault keys.  

#### Use Cases

* Derive Key Encryption Key - *Key Encryption Key* is derived from *Master Password* when it is created using a strong encryption key derivation algorithm. As noted above, end user can update *Master Password* at any time, but that may not automatically result in the derivation of a new *Key Encryption Key*. However, end user must be given the option to derive a new encryption key in the event *Master Password* or vault data has been compromised.

#### Misuse Cases / Security Requirements

* Reveal Vault Keys - The *Key Encryption Key* is used to protect the private *Vault Keys* which are in turn used to decrypt secrets stored in the vault. 

   * Brute Force - Given access to encrypted vault keys, an attacker may attempt to brute force *Master Password* and derive  *Key Encryption Key* in an attempt to reveal vault keys. Protection against this assumes a strong *Master Password* was chosen but further mitigates by ensuring a strong key derivation algorithm is used and implemented properly. A good implementation ensure use of good salt, seed and sufficient amount of iterations which in turn ensures the function takes a significant amount of time to derive the key.
   
* Reveal Key Encryption Key - An attacker is able to reveal the *Key Encryption Key* in clear text form. The Key Encryption Key must never be stored at rest in clear text form.

  * Sanitize Memory - Ensure that once the *Master Secret* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated if applicable.
	
   
#### Alignment of Security Requirements

* Bitwarden [states](https://help.bitwarden.com/article/what-encryption-is-used/) that it uses the PBKDF2 SHA-256 algorithm for its password encryption key derivation algorithm. It uses open libraries and meets current industry standards for its implementation. (Use of sufficient iterations, seed, and salt)

* Bitwarden [states](https://help.bitwarden.com/article/change-your-master-password/) that the end user has the option to derive a new *Key Encryption Key* when a new *Master Password* is chosen.

#### UML Diagram

![alt text](Images/Use%20Cases-Key%20Encryption%20Key.png)

### 3. Manage Secrets

#### Backstory

This data flow concerns the management of *Secrets.*  A *Secret* is considered to be any data, and its corresponding metadata, that an end user deems as sensitive and therefore wants to keep private. This includes data such as passwords, but can include any type of sensitive data, as long as it falls within the limitations of the *Secrets Manager* application.

#### Use Cases

* Store Secret - End user has a *Secret* that they want to encrypt and store in vault. 

* Retrieve Secret - End user has a *Secret* that they need to decrypt from vault and view in clear text.

#### Misuse Cases / Security Requirements

* Reveal Secret - The attacker is able to reveal the *Secret* the end user wants to keep private.
		
   * Network Eavesdrop - If the attacker has direct access to network communications made by the software, they may be able to reveal secret using packet payload data. This can be mitigated by ensuring all data sent over network communications is encrypted and/or data is sent over secure TLS channel.

   * Obtain Clear Text - Another way an attacker may attempt to reveal a *Secret* is by gaining access to it in clear text form in memory or storage. Mitgations as follows:
      * Sanitize Memory - Ensure that once a *Secret* is no longer needed by application, the memory space where it is located is promptly scrubbed. This must happen before said memory space is deallocated if applicable.
      * Encrypt Data at Rest - Secret data should never be stored in clear text when at rest. To limit unnecessary exposure the software should encrypt each *Secret* individually and preferably with unique encryption keys for each. 

* Information Leakage - The software should not reveal any information about *Secret* even when in encrypted form. For example, an attacker should not be able to compare encrypted blobs and infer the two *Secrets* are identical. 

* Corrupt Data - Software should protect against intentional (attacker) or unintentional (end user) data corruption. For instance if encryption keys should ever be changed, the software must ensure that the old keys are not unintentionally used to encrypt and updates going forward. Also only encrypting/decrypting one *Secret* at a time helps to limit exposure should data be corrupted.
	
#### Alignment of Security Requirements

* Bitwarden [states](https://help.bitwarden.com/article/how-is-data-securely-transmitted-and-stored/) the application encrypts all sensitive data client side and only connects to online service over a TLS socket.

* Bitwarden [states](https://help.bitwarden.com/article/change-your-master-password/) the application will try to log out any clients that are still connected to server. However it is strongly recommended that the user log out and in of any clients when changing encryption key. Failure to do so may result in data corruption.

#### UML Diagram
![alt text](Images/Use%20Cases-Secrets.png)

### 4. Manage Passwords 

#### Backstory

This data flow concerns the management of *Password.*  When we use *Password* here we are referring to *Passwords* that are use for authenicating to third party sites or applications. The terms Secrets Managers and Password Managers are often used interchangebly although most modern managers are designed store Secrets outside of just credentials. In gerneral *Password* management can be really consdierd an extension to Secrets management. Managing *Passwords* is still a very important data flow to secret managers as it is an effective way to ensure the end user does not engage in unsafe practices such as sharing and/or using weak *Passwords*.

#### Use Cases

* Password Generation - End user has a *Password* randomly created for them that also meets minimum complexity requirements set by the end user.

#### Misuse Cases / Security Requirements

* Reveal Password - An attacker is able to reveal the *Password* the end user wants to keep private.

   * Brute force Attack - Attacker is able to reveal end users password using brute force techniques in a relatively short period of time. This mitigate by ensuing a strong password is used by meeting minimum complexity requirements. 
		
   * Replay Attack - An attacker is able to obtain credentials that were publically disclosed from a prior breace. If the end user shares the same credentials that were disclosed across multiple sites, the attacker may able to gain access to another site or application unrelated to original breach. This is easily mitigated by not sharing or daisy chaining credentials. 
 
#### Alignment of Security Requirements

* Bitwarden does have a *Password* generation tool that will generate a random password based on end user's minimum requirements, however the complexity of the password generated is completely dependent on those minimum requirements and may allow what is considered to be a weak password to be generated. 

#### UML Diagram
![alt text](Images/Use%20Cases-Passwords.png)

### 5. Manage Sharing 

#### Backstory

This data flow concerns the management of the sharing of *Secrets.*  In this case, a *Secret* owner can share a *Secret* or a collection of *secrets* with another user. In addition to that the *Secret* owner and assign Read-Only (RO) or Read-Write (RW) access to the shared *Secret* or collection. 

#### Use Cases
  
* Share Secret - Allow *Secret* owner to share a *Secret/s* with another user and assign RO or RW permissions for the share. Owner of object can change permissions or revoke share at any time. 

* Access Shared Secret - This is referring to accessing a shared *Secret* that was shared to them. The share may have RW or RO privileges granted to them by the *Secret* owner.

#### Misuse Cases / Security Requirements

* Reveal Secret - The attacker is able to reveal the *Secret* the owner wants to keep private.
		
   * Unauthorized Access - Attacker is able to gain access to shared secret that was not authorized by *Secret* owner. This can be mitigated by using asymmetric encryption where the *Secret* is encrypted with an authorized user's public key. Only the authorized user is able to decrypt the secret using their private key. 
   
* Corrupt Data - The attacker is able to alter the *Secret* rendering it useless to the owner. 

   * Unauthorized Change - Attacker is able to alter *Secret* without proper authorization.  This is mitigated by ensuring the software will not allow changes to a secret unless they have RW authorization. The software must ensure the implmentation for RO/RW flag is not subject to tampering. 

#### Alignment of Security Requirements

Bitwarden uses [collections](https://help.bitwarden.com/article/collections/) for managing shares to groups of *secrets*. Each indivual user to the share can be assigned RO/RW permissions as appropiate. Users with RW acccess to a collection can add, edit, or delete any Secret item contained within it.

#### UML Diagram

![alt text](Images/Use%20Cases-Sharing.png)

## Security Related Configuration and Installation Issues

When using the standard online version, Bitwarden is unique from most apps in the fact it does not have an in depth configuration or installation. Having said that, Bitwarden does have the option to host the complete server stack on your own system verses using their cloud based version as discussed [here](https://help.bitwarden.com/article/install-on-premise/). Unfortunately, time did not permit an in depth look of the self hosted option at this point. Bitwarden clients do not have much in the way of configuration and what they do have is synced from the online service. Bitwarden also has several client types (Desktop apps, browser extensions, command line clients, et..) and variations for popular Operating Systems (Windows, Mac, Linux), but for the sake of this document we looked solely at using the standard online portal with the command line client for Windows. Aside from some small differences, it appears that the clients all behave similarly in regards to configuration. Although the configuration and installation for Bitwarden is admittedly sparse, we still found some items of interest.

We started with the online portal which is where we set up our free account for testing. When creating a new account, you will be asked to create a Master Password which will be used to unlock your personal vault from that point forward. As mentioned previously when choosing a new master password, Bitwarden will display a strength meter to let the end user know how strong the password is as it is typed. Aside from a minimum requirement of 8 characters, Bitwarden will allow a user to choose a weak password after a pop up warning is acknowledged. The only items needed to create the account and vault are an email address, your name, your chosen password and an optional password hint. The whole process can be completed in under a minute at which time you can then start adding or importing secrets into your vault. I created a contrived secret in the vault for testing below. 

Once the account is set up, the end user then can start installing one or more of the client options availible to them. Again, we choose to install the portable command line executable for Windows. The installation required nothing more than extracting a zip file and executing the only file, bw.exe.

The first task attempted with client was to authenticate to vault which was pretty straight forward and provided the following output.

    C:\Users\unouser\bw-windows-1.7.4>bw login -obfuscated-@unomaha.edu
    ? Master password: [hidden]
    You are logged in!

    To unlock your vault, set your session key to the `BW_SESSION` environment variable. ex:
    $ export BW_SESSION="K8rabmxkjcU0mvlWEccneq5mdONw+41vOOKlpDY3fWQvIk1e++iwEZfboReSkHdoUTVsE8FTDejSn4kIzYiwgQ=="
    > $env:BW_SESSION="K8rabmxkjcU0mvlWEccneq5mdONw+41vOOKlpDY3fWQvIk1e++iwEZfboReSkHdoUTVsE8FTDejSn4kIzYiwgQ=="

    You can also pass the session key to any command with the `--session` option. ex:
    $ bw list items --session K8rabmxkjcU0mvlWEccneq5mdONw+41vOOKlpDY3fWQvIk1e++iwEZfboReSkHdoUTVsE8FTDejSn4kIzYiwgQ==

As can be seen the client was recommending setting the session key as an environment variable. If one left the command shell unattended or an attacker was able gain access to the current session, they could easily execute commands with providing any other credentials as shown below. The session did appear to time out after a few minutes of inactivity however.

    C:\Users\unouser\bw-windows-1.7.4>set BW_SESSION="K8rabmxkjcU0mvlWEccneq5mdONw+41vOOKlpDY3fWQvIk1e++iwEZfboReSkHdoUTVsE8FTDejSn4kIzYiwgQ=="

    C:\Users\unouser\bw-windows-1.7.4>echo %BW_SESSION%
    "K8rabmxkjcU0mvlWEccneq5mdONw+41vOOKlpDY3fWQvIk1e++iwEZfboReSkHdoUTVsE8FTDejSn4kIzYiwgQ=="

    C:\Users\unouser\bw-windows-1.7.4>cls

    C:\Users\unouser\bw-windows-1.7.4>bw list items
    [{"object":"item","id":"9aa91736-92da-4a0c-a234-aadb000c4100","organizationId":null,"folderId":null,"type":1,"name":"test","notes":null,"favorite":false,"login":{"username":"User","password":"VeryC0mplexP4ss","totp":null,"passwordRevisionDate":null},"collectionIds":[],"revisionDate":"2019-10-03T00:44:36.906Z"}]
   
Immediately after authenticating with master password it was noticed that the vault data was synced to client device as documented on Bitwarden's site [here](https://help.bitwarden.com/article/where-is-data-stored-computer/)

    C:\Users\unouser\AppData\Roaming\Bitwarden CLI>dir  %appdata%\"bitwarden CLI"\
     Volume in drive C has no label.
     Volume Serial Number is E8EA-01AE

     Directory of C:\Users\unouser\AppData\Roaming\bitwarden CLI

    10/02/2019  07:40 PM    <DIR>          .
    10/02/2019  07:40 PM    <DIR>          ..
    10/02/2019  08:48 PM             5,252 data.json
                   1 File(s)          5,252 bytes
                   2 Dir(s)  73,770,647,552 bytes free

This file appears to be a local copy of the vault data to include encrypted data as well as some clear text metadata as discussed [here](https://help.bitwarden.com/article/what-information-is-encrypted/). A couple things of interest are the fact that the user name (which is actually the user's email address) appears as clear text in one field but encrypted in another, and there is a field that defines the number of Iterations the Key Derivation Function should use when deriving a new Key Encryption Key. (Only relevant lines shown below, and some data obfuscated)

    ...
    "userEmail": "--obfuscated--@unomaha.edu",
    ...
    "kdfIterations": 100000,
    ...
      "login": {
        "username": "2.eyhmOV---yeW6YQ9ChYw==|ugxNKr6-----mXNcYIiXOYww==|nPH+KB8mV9FCLg5dcXw----Nic13ebjwcuJGAMRW18=",
        "password": "2.rIfX3K---vHIa3h+u6p3Q==|Df5AG4G-----fp8Kcng2w==|Ajro1X8NfswplISRSi3----YrIKQkw2KCUlb7E6e3s=",
        "passwordRevisionDate": null,
        "totp": null
      }
    ...
    
This file is synced with server (who is authoritative) at login but appears to conduct read operations from the cached vault data. We are wondering if "kdfIterations" can be overwritten locally and if the client can be tricked into using an artificially low iteration count making the master password susceptible to a brut force attack as documented with another online security manager [here](https://palant.de/2018/07/09/is-your-lastpass-data-really-safe-in-the-encrypted-online-vault/). Bitwarden has [documented](https://help.bitwarden.com/article/change-your-master-password/) cache mismatch issues so its sounds possible but we will need to conduct testing to vet that out.
  


Command Line Interface (CLI) download file: bw-windows-1.7.4.zip containing the standalone executable: bw.exe. 

	C:\Users\unouser\Documents>bw
	Usage: bw [options] [command]

	Options:

	--pretty                                   Format output. JSON is tabbed with two spaces.
	--raw                                      Return raw output instead of a descriptive message.
	--response                                 Return a JSON formatted version of response output.
	--quiet                                    Don't return anything to stdout.
	--session <session>                        Pass session key instead of reading from env.
	-v, --version                              output the version number
	-h, --help                                 output usage information

	Commands:

	login [options] [email] [password]         Log into a user account.
	logout                                     Log out of the current user account.
	lock                                       Lock the vault and destroy active session keys.
	unlock [password]                          Unlock the vault and return a new session key.
	sync [options]                             Pull the latest vault data from server.
	list [options] <object>                    List an array of objects from the vault.
	get [options] <object> <id>                Get an object from the vault.
	create [options] <object> [encodedJson]    Create an object in the vault.
	edit <object> <id> [encodedJson]           Edit an object from the vault.
	delete [options] <object> <id>             Delete an object from the vault.
	share <id> <organizationId> [encodedJson]  Share an item to an organization.
	import [options] [format] [input]          Import vault data from a file.
	export [options] [password]                Export vault data to a CSV or JSON file.
	generate [options]                         Generate a password/passphrase.
	encode                                     Base 64 encode stdin.
	config <setting> <value>                   Configure CLI settings.
	update                                     Check for updates.

  	Examples:

	bw login
	bw lock
	bw unlock myPassword321
	bw list --help
	bw list items --search google
	bw get item 99ee88d2-6046-4ea7-92c2-acac464b1412
	bw get password google.com
	echo '{"name":"My Folder"}' | bw encode
	bw create folder eyJuYW1lIjoiTXkgRm9sZGVyIn0K
	bw edit folder c7c7b60b-9c61-40f2-8ccd-36c49595ed72 eyJuYW1lIjoiTXkgRm9sZGVyMiJ9Cg==
	bw delete item 99ee88d2-6046-4ea7-92c2-acac464b1412
	bw generate -lusn --length 18 

### [Project Task Assignments and Collaborations](https://github.com/caseyschmitz/CYBR8420-GotRoot/projects/3)

