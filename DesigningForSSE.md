# Designing for Software Security Engineering

## Level-0 Diagram
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/L0_Bitwarden.png)

## [Level-1 DFDs and Identified Threats](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/Bitwarden_DFD.pdf)
Raw htm available [here](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/DFD_Bitwarden.htm)

## Threats Requiring Further Investigation

### Threat 2: External Entity Human User Potentially Denies Receiving Data

Data with the local vault is encrypted a local JSON file, along with 2 lines metadata. The metadata saves the time of the most recent sync with the server, however the client does not keep an audit log of failed attempts to sync vault with Bitwarden server.

### Threat 7: Potential Data Repudiation by Auth Module 
The Bitwarden CLI client in its source code does not address any line about saving a login history in a log file. Indeed, after login out, all the data contain in the unique file on which it writes are erase and the file contains only 3 lignes not related to any login information.

### Threat 20: Spoofing of the Bitwarden Server External Destination Entity
The Bitwarder client uses TLS encryption and encrypts all sensitve data before sending to server so no sensitvie information would be theorecitally compromised. However if the client was attempting to sync any updates made to vault data or encyptions it may result in lost data or possbile data currptoin

### Threat 26: Potential Excessive Resource Consumption for Secrets Module or Local File System
The Bitwarden CLI client makes singular read/write attempts to a json file on the local file system that, despite not appearing to have graceful error-handling, should not consume resources in an excessive manner. The client will simply fail to read/write this file and fail to operate as intended.

### Threat 31: Data Flow HTTPS Is Potentially Interrupted
The Bitwarden CLI client does not perform any monitoring between the Secrets Module and the Bitwarden Server to prevent excessive disk/CPU consumption in the event that the Bitwarden Server is unavailable. However, the client makes singular requests to the Bitwarden Server that, upon failure, do not prompt additional requests to services on the Bitwarden Server.

### Threat 48: Data Flow Secrets Requests (CRUD) Is Potentially Interrupted
The Bitwarden CLI client does not perform any monitoring between the user and the Secrets Module to prevent excessive disk/CPU consumption in the event that the Secrets Module is unavailable.
