# Code Analysis for Software Security Engineering

## Code Review Strategy

Bitwarden is an open-source online password management service, and has developed a good reputation for being secure over the years. We knew when we chose this application for our project, it was going to be a big undertaking to find any significant security bugs given the maturity of the application, it is completely open source, and has already been [audited](https://cdn.bitwarden.net/misc/Bitwarden%20Security%20Assessment%20Report%20-%20v2.pdf) by third party security company. 

Bitwarden is primarily written in Microsoft's TypeScript language and has been ported to multiple platforms and Operating Systems to include most mobile devices and desktops/laptops used today. Due to time constraints we chose to focus on the command line application. When we started to do analysis a couple team members naturally gravitated towards manual code review while the other half towards the automated testing. 

As expected, we initially had trouble finding any programming errors. The static scans were not returning anything significant and we were not able to produce any segmentation faults with argument passing. All input validation appeared to handle any malformed arguments gracefully. Not getting anywhere with that we instead started to focus on logical errors, in particular in regards to it cryptographic implementations and best practices as outlined by NIST and OWASP.   


## Manual Code Review

### Background
Bitwarden is an open source, muti-platform, online password management service accessable via almost any personal device. Though Bitwarden stores sensitive data in the cloud, it has a proven reputation for security. The vault is only accessible via a one way hash derived from the user login information. Each item within the vault is encoded using the master hash and iterations of PBKDF2 defined by the user. The encrypted vault is then synced with the server via RSA public key infrastructure. The majority of Bitwarden's security features are client side, and this is where we have focused our analysis. To avoid most OS/GUI specific issues, manual code review was performed on Bitwarden's Command Line Interface (CLI) executable.

Because we wanted to have access to any data sent over TLS with no or minimal modifications data payloads, we choose to implement a MITM style transparent TLS proxy. This was accomplished using VMWare and two Linux Mint VMs, one for the  client and the other to act as a gateway for the first VM. The gateway VM also used a combination of iptables NAT rules and sslsplit to facilitate the transparent proxy. The openssl binary was used to create self signed cert/key pair which in turn was used by sslsplit to forge certificates for any TLS sites accessed by client. Because the bitwarden command line client has its  certificate authority (CA) trust store embedded in the binary with no arguments to modify, we had to replace one of the CA certs with our self signed cert.  This was accomplished using ghex hex editor, dd and cat to locate, carve out and replace the cert respectively. Some padding were needed to ensure that the new cert was the exact same size as the one it replaced ensuring no alignment issues for instructions or data in binary.  The modified binary was appropriately named, so there was no confusion as to which one we were using. The commands below show the error with the native CA store vs the modified one. As it turns out very few commands required the modified version,  as the client uses a local copy of the vault which is synced after a successful login is made. 


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


### MCR1: Password Strength ([CWE 521](https://cwe.mitre.org/data/definitions/521.html))
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html)


#### Findings

Bitwarden uses a "Master Password" that is chosen by the end user and what is ultimately used to protect the end user's sensitive data. Although the master password cannot be created and changed with command line client, it is worth mentioning that the minimum password length is 8 characters. A password shorter than 8 is considered weak to be weak according to [NIST SP800-63b](https://pages.nist.gov/800-63-3/sp800-63b.html). Bitwarden does have a Master Password strength validation tool however it is only suggestive and will allow a user to choose a weak password. Bitwarden complies with the minimum requirements for password length, however we would still like to see some enforcement of password complexity as well as checks to see if matches commonly used or previously breached password lists.


### MCR2: Password Storage ([CWE 916](https://cwe.mitre.org/data/definitions/916.html))
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [NIST](https://pages.nist.gov/800-63-3/sp800-63b.html)

#### Findings

Bitwarden uses a one way [hash](https://help.bitwarden.com/article/password-salt-hash/) to prevent someone from reverse engineering Master Password. At this time PBKDF2 is the only key derivation function used however it does appear that the developers are anticipating allowing the user to choose other key derivation functions as can be seen with makePreloginKey() function below. According to [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) Argon2 should be the first choice. Since Bitwarden has control of both client and server applications, it is entirely feasible for Bitwarden to implement Argon2.

Bitwarden allows the end user to choose the number of iterations for the key derivation function. This value defaults to 100,000 iterations and has a hard coded minimum of 5000 iterations which is enforced in the code as shown in makeKey() function below. According to [NIST SP800-63b](https://pages.nist.gov/800-63-3/sp800-63b.html) (which was published in 2017) the minimum number of iterations should be 10000. The purpose of the iterations is to ensure the computation time is significant enough to protect a strong password against attack. We recommend the minimum should be set to at least the NIST standards, however that may not enough with today's hardware capabilities. Testing should be done to ensure the minimum is adequate for today's computing capabilities and should take precedence over supporting antiquated hardware. If multiple KDF functions are implemented, the minimum iterations should be tested individually and set respectively.

Bitwarden uses a salt when computing the one way hash of the Master Password.  As can be seen in makePreloginKey() and then makeKey() functions below, the end user email is used as the salt. This does not seem  to follow [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) best practices and purposes listed below. An email would be considered cryptographically-strong random data and may not meet length requirements. 

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
    Store the salt and the password hash.


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


### MCR3: Session Management([CWE 256](https://cwe.mitre.org/data/definitions/256.html))
- [OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Session_Management_Cheat_Sheet.html)

#### Findings

Bitwarden command line client uses API calls to server to validate email and password hash. If that is successful, the vault data is then sent back to client as JSON, which is in turn is stored in a local [file](https://help.bitwarden.com/article/where-is-data-stored-computer/) on the device. All communication between client and server is done using a TLS connection. (On Linux the vault is stored in user's home directory at ~/.config/Bitwarden CLI/data.json.) While some metadata is stored as clear text in vault file (as noted [here](https://help.bitwarden.com/article/what-information-is-encrypted/)), all sensitive data is encrypted. Once the device has a local copy of the vault, it no longer needs to make any calls to Internet to decrypt items in the vault. Although it does not make calls to Internet, the Bitwarden command line program does not run in background and therefore uses a session ID to preserve auth status between executions. As can be seen below, the bw binary offers the users two methods to pass session IDs to program. The first method sets the session ID to an environment variable. Once the variable is set in the user's environment, the bw command can be run to display sensitive data as clear text without providing any other credentials. This of course means that a privileged user on that device was able to view environment variables, they could easily highjack session and use it to decrypt any data in the users unlocked vault. The second method allows the user to pass the session as a command line argument. With this method, any user could list the process table with command line args and potentially hijack the session. They would also need read access to local unlocked vault file as well which may prevent non-privileged users from carrying out a successful attack.

   
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
  

## Automated Code Review

### ACR1: SonarQube (Static Analysis)
- [Results on SonarCloud](https://sonarcloud.io/organizations/cybr8420gotroot/projects)

Sonar's SonarScanner performs automated static code analysis by searching a project's codebase for Bugs, Vulnerabilities, Code Smells, and Security Hotspots. The rules applied in scans are selected based on the type of file being scanned - the rules are documented in Sonar's [rule library](https://rules.sonarsource.com/). Rules are tagged and enriched with related CVEs, CWEs, and examples of code that would trigger a finding.

Sonar describes the finding types as follows:
- *Bug*: A coding error that will break your code and needs to be fixed immediately.
- *Vulnerabilities*: Code that can be exploited by hackers.
- *Security Hotspots*: Security-sensitive code that requires manual review to assess whether or not a vulnerability exists.
- *Code Smells*: Code that is confusing and difficult to maintain.

Scans were performed against components of Bitwarden's codebase by cloning repositories of interest, running SonarScanner locally against each, and uploading results to SonarCloud for reporting.

#### Findings

##### [Bitwarden JavaScript Libaray](https://github.com/bitwarden/jslib) ([results](https://sonarcloud.io/dashboard?id=bitwarden-jslib))
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/arc1_jslib.png)

###### Weak Cryptography

As expected, most of the findings in these common libraries were Security Hotspots related to weak cryptography. Implementations of hashing in Bitwarden's main cryptography services rely on the default *crypto* JavaScript library, which OWASP [does not recommend](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html) for the secure storage of data. Despite using a less-secure encryption library, these implementations do use salts and support strong legacy encryption algorithms like sha512 and sha256.

[Example](https://sonarcloud.io/code?id=bitwarden-jslib&line=29&selected=bitwarden-jslib%3Asrc%2Fservices%2FnodeCryptoFunction.service.ts):
```
hash(value: string | ArrayBuffer, algorithm: 'sha1' | 'sha256' | 'sha512' | 'md5'): Promise<ArrayBuffer> {
        const nodeValue = this.toNodeValue(value);
        const hash = crypto.createHash(algorithm);
        hash.update(nodeValue);
        return Promise.resolve(this.toArrayBuffer(hash.digest()));
    }
```

###### Denial of Service

Scan results also show usage of regular expressions that are potentially vulnerable to Regular Expression Denial of Service ([ReDoS](https://www.owasp.org/index.php/Regular_expression_Denial_of_Service_-_ReDoS)). Manual inspections of these regular expressions reveal that they are not vulnerable to ReDoS attacks as they do not contain grouping with repitition.

[Example](https://sonarcloud.io/code?id=bitwarden-jslib&line=259&selected=bitwarden-jslib%3Asrc%2Fmisc%2Futils.ts):
```
if (/(android|bb\d+|meego).+mobile|avantgo|bada\/|blackberry|blazer|compal|elaine|fennec|hiptop|iemobile|ip(hone|od)|iris|kindle|lge |maemo|midp|mmp|mobile.+firefox|netfront|opera m(ob|in)i|palm( os)?|phone|p(ixi|re)\/|plucker|pocket|psp|series(4|6)0|symbian|treo|up\.(browser|link)|vodafone|wap|windows ce|xda|xiino/i.test(a) || /1207|6310|6590|3gso|4thp|50[1-6]i|770s|802s|a wa|abac|ac(er|oo|s\-)|ai(ko|rn)|al(av|ca|co)|amoi|an(ex|ny|yw)|aptu|ar(ch|go)|as(te|us)|attw|au(di|\-m|r |s )|avan|be(ck|ll|nq)|bi(lb|rd)|bl(ac|az)|br(e|v)w|bumb|bw\-(n|u)|c55\/|capi|ccwa|cdm\-|cell|chtm|cldc|cmd\-|co(mp|nd)|craw|da(it|ll|ng)|dbte|dc\-s|devi|dica|dmob|do(c|p)o|ds(12|\-d)|el(49|ai)|em(l2|ul)|er(ic|k0)|esl8|ez([4-7]0|os|wa|ze)|fetc|fly(\-|_)|g1 u|g560|gene|gf\-5|g\-mo|go(\.w|od)|gr(ad|un)|haie|hcit|hd\-(m|p|t)|hei\-|hi(pt|ta)|hp( i|ip)|hs\-c|ht(c(\-| |_|a|g|p|s|t)|tp)|hu(aw|tc)|i\-(20|go|ma)|i230|iac( |\-|\/)|ibro|idea|ig01|ikom|im1k|inno|ipaq|iris|ja(t|v)a|jbro|jemu|jigs|kddi|keji|kgt( |\/)|klon|kpt |kwc\-|kyo(c|k)|le(no|xi)|lg( g|\/(k|l|u)|50|54|\-[a-w])|libw|lynx|m1\-w|m3ga|m50\/|ma(te|ui|xo)|mc(01|21|ca)|m\-cr|me(rc|ri)|mi(o8|oa|ts)|mmef|mo(01|02|bi|de|do|t(\-| |o|v)|zz)|mt(50|p1|v )|mwbp|mywa|n10[0-2]|n20[2-3]|n30(0|2)|n50(0|2|5)|n7(0(0|1)|10)|ne((c|m)\-|on|tf|wf|wg|wt)|nok(6|i)|nzph|o2im|op(ti|wv)|oran|owg1|p800|pan(a|d|t)|pdxg|pg(13|\-([1-8]|c))|phil|pire|pl(ay|uc)|pn\-2|po(ck|rt|se)|prox|psio|pt\-g|qa\-a|qc(07|12|21|32|60|\-[2-7]|i\-)|qtek|r380|r600|raks|rim9|ro(ve|zo)|s55\/|sa(ge|ma|mm|ms|ny|va)|sc(01|h\-|oo|p\-)|sdk\/|se(c(\-|0|1)|47|mc|nd|ri)|sgh\-|shar|sie(\-|m)|sk\-0|sl(45|id)|sm(al|ar|b3|it|t5)|so(ft|ny)|sp(01|h\-|v\-|v )|sy(01|mb)|t2(18|50)|t6(00|10|18)|ta(gt|lk)|tcl\-|tdg\-|tel(i|m)|tim\-|t\-mo|to(pl|sh)|ts(70|m\-|m3|m5)|tx\-9|up(\.b|g1|si)|utst|v400|v750|veri|vi(rg|te)|vk(40|5[0-3]|\-v)|vm40|voda|vulc|vx(52|53|60|61|70|80|81|83|85|98)|w3c(\-| )|webc|whit|wi(g |nc|nw)|wmlb|wonu|x700|yas\-|your|zeto|zte\-/i.test(a.substr(0, 4))) {
	mobile = true;
}
```

##### [Bitwarden Command-Line Interface](https://github.com/bitwarden/cli) ([results](https://sonarcloud.io/dashboard?id=bitwarden-cli))
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/arc1_cli.png)

###### Command Injection

The only notable results from a scan against the Command-Line Interface relate to command injection. The module that receives input from the command-line performs rigorous input sanitization prior to executing any code using user input as an argument.

Example:
```

```


## Summary of Findings

Overall Bitwarden appears to be a well written application and its developers appear to follow good coding practices. For the manual code review, we found the OWASP cheat sheets to be a valuable reference as to what we should be looking at. Initially we were unable to find any obvious coding issues and begin to set up a lab to do some live analysis. This lead us to several implementation issues we feel warranted some attention. The first was related to enforcement of password complexity, the next two, minimum iterations and salt values, did not appear to be in alignment for best practices as reported by OWASP and/or NIST, and the last two were related to run time management of passwords and session ids in a Linux environments. 

