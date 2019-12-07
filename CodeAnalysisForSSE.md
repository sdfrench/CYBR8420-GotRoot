# Code Analysis for Software Security Engineering

## Code Review Strategy


## Manual Code Review

### Background
Bitwarden, an open-source online password management service, that has several different client application for almost any personal device. Even though Bitwarden uses the cloud to store sensitive data, it has earned a good reputation for security as the data can only be encrypted using a master password chosen by end users and all sensitive information is encypted client side before it is sent to vault. Because the majority of the security features for Bitwarden are client side, that is where we focused our analysis. For the manual code review we choose the Linux version of the command line client.

Because we wanted to have access to any data sent over TLS with no or minimal modifications data payloads, we choose to implement a MITM style transparent TLS proxy. This was accomplished using VMWare and two Linux Mint VMs, one for the bw client and the other to act as a gateway for the first VM. The gateway VM also used a combination of iptables NAT rules and sslsplit to facilitate the transparent proxy. The openssl binary was used to create self signed cert/key pair which in turn was used by sslsplit to forge certificates for any TLS sites accessed by client. Because the bitwarden command line client has its CA trust store embedded in the binary with no arguments to modify, we had to replace one of the CA certs with our self signed cert.  This was accomplished using ghex hex editor, dd and cat to locate , carve out and replace the cert respectively. Some padding was needed to ensure that the new cert was the exact same size as the one it replaced ensuring no alignment issues for instructions or data in binary.  The modified binary was appropriately named so it there was no confusion as to which one we were using. The commands below show the error with the native CA store vs the modified one. As it turns out very few commands required the modified version,  as the client uses a local copy of the vault which is synced after a successful login is made. 


    $  bw login
    ? Email address: [someuser]@unomaha.edu
    ? Master password: [hidden]
    request to https://api.bitwarden.com/accounts/prelogin failed, reason: self signed certificate in certificate chain

    $ $ bw.mod login
    ? Email address: [someuser]h@unomaha.edu
    ? Master password: [hidden]
    You are logged in!

    To unlock your vault, set your session key to the `BW_SESSION` environment variable. ex:
    $ export BW_SESSION="lbSk2rQD0SR1sYFKJjt9EIFTCyNDKq7bFNt8uCfzlI+znBfPETrkNOWJECmM8+VPZxRDoF2PJY5XEmV6OeoEIg=="
    > $env:BW_SESSION="lbSk2rQD0SR1sYFKJjt9EIFTCyNDKq7bFNt8uCfzlI+znBfPETrkNOWJECmM8+VPZxRDoF2PJY5XEmV6OeoEIg=="

    You can also pass the session key to any command with the `--session` option. ex:
    $ bw list items --session lbSk2rQD0SR1sYFKJjt9EIFTCyNDKq7bFNt8uCfzlI+znBfPETrkNOWJECmM8+VPZxRDoF2PJY5XEmV6OeoEIg==


### MCR1: Password Strength ([CWE]())
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [Code Example]()

#### Findings

Bitwarden uses a "Master Password" that is chosen by the end user and what is ultimately used to protect the end user's sensitive data. Although the master password cannot be created and changed with command line client, it is worth mentioning that the minimum password length is 8 characters. A password shorter than 8 is considered weak to be weak according to [NIST SP800-63b](https://pages.nist.gov/800-63-3/sp800-63b.html). Bitwarden does have a Master Password strength validation tool however it is only suggestive and will allow a user to choose a weak password. Bitwarden complies with the minimum requirements for password length, however we would still like to see some enforcement of password complexity as well as checks to see if matches commonly used or previously breached password lists.


### MCR2: Password Storage ([CWE]())
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html)
- [Code Example]()

#### Findings

Bitwarden uses a one way [hash](https://help.bitwarden.com/article/password-salt-hash/) to prevent someone from reverse engineering Master Password. At this time PBKDF2 is the only key derivation function used however it does appear that the developers are anticipating allowing the user to choose other key derivation functions as can be seen with makePreloginKey() function below. According to [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) Argon2 should be the first choice. Since Bitwarden has control of both client and server applications, its entirely feasible for Bitwarden to implement Argon2.

Bitwarden allows the end user to choose the number of iterations for the key derivation function. This value defaults to 100,000 iterations and has a hard coded minimum of 5000 iterations which is enforced in the code as shown in makeKey() function below. According to [NIST SP800-63b](https://pages.nist.gov/800-63-3/sp800-63b.html) (which was published in 2017) the minimum number of iterations should be 10000. The purpose of the iterations is to ensure the computation time is significant enough to protect a strong password against attack. We recommend the minimum should be set to at least the NIST standards, however that may not enough with today's hardware capabilities. Testing should be done to ensure the minimum is adequate for today's computing capabilities and should take precedence over supporting antiquated hardware. If multiple KDF functions are implemented, the minimum iterations should be tested individually and set respectively.

Bitwarden uses a salt when computing the one way hash of the Master Password.  As can be seen in makePreloginKey() and then makeKey() functions below, the end users email is used as the salt. This does not seem  to follow [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) best practices and purposes listed below. An email would be considered cytptographically-strong random data and may not meet length requirements. 

It should also be noted here that the Bitwarden command line client had a couple commands that allow for Master Password to be passed as a argument vs an interactive prompting for password. (Notice "unlock" and "export" commands in --help output below.) This is a dangerous practice as process tree can be viewed with command line arguments exposed by any user on system. This may have been added for scripting but alternative means should be used instead, such a using a text file containing passsword. 

[auth.service.ts](https://github.com/bitwarden/jslib/blob/57e49207e9ad57c71576fc487a38513a4d0fe120/src/services/auth.service.ts) Source File:

    ...
    async makePreloginKey(masterPassword: string, email: string): Promise<SymmetricCryptoKey> {
        email = email.trim().toLowerCase();
        this.kdf = null;
        this.kdfIterations = null;
        try {
            const preloginResponse = await this.apiService.postPrelogin(new PreloginRequest(email));
            if (preloginResponse != null) {
                this.kdf = preloginResponse.kdf;
                this.kdfIterations = preloginResponse.kdfIterations;
            }
        } catch (e) {
            if (e == null || e.statusCode !== 404) {
                throw e;
            }
        }
        return this.cryptoService.makeKey(masterPassword, email, this.kdf, this.kdfIterations);
    }
    
    ...


[crypto.service.ts](https://github.com/bitwarden/jslib/blob/57e49207e9ad57c71576fc487a38513a4d0fe120/src/services/crypto.service.ts) Source File:

    ...
    async makeKey(password: string, salt: string, kdf: KdfType, kdfIterations: number):
        Promise<SymmetricCryptoKey> {
        let key: ArrayBuffer = null;
        if (kdf == null || kdf === KdfType.PBKDF2_SHA256) {
            if (kdfIterations == null) {
                kdfIterations = 5000;
            } else if (kdfIterations < 5000) {
                throw new Error('PBKDF2 iteration minimum is 5000.');
            }
            key = await this.cryptoFunctionService.pbkdf2(password, salt, 'sha256', kdfIterations);
        } else {
            throw new Error('Unknown Kdf.');
        }
        return new SymmetricCryptoKey(key);
    }
    
    ...

[OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) salt best practices:

    Generate a salt using a cryptographically secure function.
    The salt should be at least 16 characters long.
    Encode the salt into a safe character set such as hexadecimal or Base64.
    Combine the salt with the password.
       This can be done using simple concatenation, or a construct such as a HMAC.
    Hash the combined password and salt.
    Store the salt and the password hash


Bitwarden Command Line --help Output:
    $ bw --help
    Usage: bw [options] [command]

    Options:

      --pretty                                    Format output. JSON is tabbed with two spaces.
      --raw                                       Return raw output instead of a descriptive message.
      --response                                  Return a JSON formatted version of response output.
      --quiet                                     Don't return anything to stdout.
      --session <session>                         Pass session key instead of reading from env.
      -v, --version                               output the version number
      -h, --help                                  output usage information

    Commands:

      login [options] [email] [password]          Log into a user account.
      logout                                      Log out of the current user account.
      lock                                        Lock the vault and destroy active session keys.
      unlock [options] [password]                 Unlock the vault and return a new session key.
      sync [options]                              Pull the latest vault data from server.
      list [options] <object>                     List an array of objects from the vault.
      get [options] <object> <id>                 Get an object from the vault.
      create [options] <object> [encodedJson]     Create an object in the vault.
      edit [options] <object> <id> [encodedJson]  Edit an object from the vault.
      delete [options] <object> <id>              Delete an object from the vault.
      share <id> <organizationId> [encodedJson]   Share an item to an organization.
      confirm [options] <object> <id>             Confirm an object to the organization.
      import [options] [format] [input]           Import vault data from a file.
      export [options] [password]                 Export vault data to a CSV or JSON file.
      generate [options]                          Generate a password/passphrase.
      encode                                      Base 64 encode stdin.
      config <setting> <value>                    Configure CLI settings.
      update                                      Check for updates.

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
        bw config server https://bitwarden.example.com


### MCR3: Session Management([CWE]())
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)
- [Code Example]()

#### Findings

Bitwarden uses API calls to server to verify email and password hashes match. If that is successful, vault data is then sent back to client as JSON, which is in turn is stored in a local [file](https://help.bitwarden.com/article/where-is-data-stored-computer/) on the device. All communication between client and server is done using a TLS connection. (On Linux the vault is stored in user's home directory at ~/.config/Bitwarden CLI/data.json.) While some metadata is stored as clear text in vault file (as noted [here](https://help.bitwarden.com/article/what-information-is-encrypted/)), all sensitive data is encrypted. Once the device has a local copy of the vault, it no longer needs to make any calls to Internet to decrypt items in the vault. Although it does not make calls to Internet, the Bitwarden command line program does not run in background and therefore uses a session ID to preserve auth status between executions. As can be seen below, the bw binary offers the users two methods to pass session IDs to program. The first method sets the session ID to an environment variable. Once the variable is set in the user's environment, the bw command can be run to display sensite data as clear text without providing any other credentials. This of course means tht a privileged user on that device was able to view environment variables, they could easily highjack session and use it to decrypt any data in the users unlocked vault. The second method allows the user to pass the session as a command line argument. With this method, any user could list he process table with command line args and potentially hijack the session. They would also need read access to local unlocked vault file as well which may prevent non-privileged users from carrying out a successful attack.

   
    $ bw logout
    You have logged out.
    $ bw login
    ? Email address: [someuser]@unomaha.edu
    ? Master password: [hidden]
    You are logged in!

    To unlock your vault, set your session key to the `BW_SESSION` environment variable. ex:
    $ export BW_SESSION="Y7jkFuQU+PyXZzpvmx+9CqasR6M4oZHzzEZ42hfKa16ldqBVgZAGBNzM8PI3f8eNNBSMEUv6VKbeH5QidC7V9g=="
    > $env:BW_SESSION="Y7jkFuQU+PyXZzpvmx+9CqasR6M4oZHzzEZ42hfKa16ldqBVgZAGBNzM8PI3f8eNNBSMEUv6VKbeH5QidC7V9g=="

    You can also pass the session key to any command with the `--session` option. ex:
    $ bw list items --session Y7jkFuQU+PyXZzpvmx+9CqasR6M4oZHzzEZ42hfKa16ldqBVgZAGBNzM8PI3f8eNNBSMEUv6VKbeH5QidC7V9g==
  



### MCR1: Improper Input Validation ([CWE-20](https://cwe.mitre.org/data/definitions/20.html))
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [Code Example](https://rules.sonarsource.com/typescript/RSPEC-4829)

#### Findings
- #TODO

### MCR2: Improper Neutralization of Argument Delimiters in a Command ('Argument Injection') ([CWE-88](https://cwe.mitre.org/data/definitions/88.html))
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [Code Example](https://rules.sonarsource.com/typescript/RSPEC-4823)

#### Findings
- #TODO

## Automated Code Review

### ACR1: SonarQube (Static Analysis)
- [Results on SonarCloud](https://sonarcloud.io/organizations/cybr8420gotroot/projects)

#### Findings
- #TODO

### ACR2: (@scott)

#### Findings
- #TODO

## Summary of Findings
