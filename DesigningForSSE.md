# Designing for Software Security Engineering

## Level-0 Diagram
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/L0_Bitwarden.png)

## [Level-1 DFDs and Identified Threats](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/Bitwarden_DFD.pdf)
Raw htm available [here](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/DFD_Bitwarden.htm)

## Threats Requiring Further Investigation


### Threat 2: External Entity Human User Potentially Denies Receiving Data

Local vault is encrypted and stored within a JSON file, along with 3 lines metadata. This metadata stores the applications ID hash and a timestamp for the most recent sync with the server. The client does not keep an audit log of failed attempts with the Bitwarden server.

### Threat 7: Potential Data Repudiation by the Authentication Module 
The Bitwarden CLI client does not maintain a local log file. That is maintained by the Bitwarden server. The local vault is purged of data when the client logs out. Only the application ID hash and a default timestamp remain.

### Threat 20: Spoofing of the Bitwarden Server External Destination Entity
The Bitwarden client uses TLS encryption and encrypts all sensitive data before sending it to server. Theoretically preventing information disclosure in the event that the Bitwarden server was spoofed. However, if the client attempts to sync any updates made to vault data or encryption keys, it may result in lost data or possible data corruption.

### Threat 26: Potential Excessive Resource Consumption for the Secrets Module or Local File System
The Bitwarden CLI client makes singular read/write attempts to a json file on the local file system that, despite not appearing to have graceful error-handling, should not consume resources in an excessive manner. The client will simply fail to read/write this file and fail to operate as intended.

### Threat 31: Data Flow HTTPS is Potentially Interrupted
The Bitwarden CLI client does not perform monitoring between the Secrets Module and the Bitwarden Server to prevent excessive disk/CPU consumption in the event that the Bitwarden Server is unavailable. However, the client makes singular requests to the Bitwarden Server that, upon failure, do not prompt additional requests to services on the Bitwarden Server.

### Threat 48: Data Flow Secrets Requests (CRUD) Is Potentially Interrupted
The Bitwarden CLI client does not perform any monitoring between the user and the Secrets Module to prevent excessive disk/CPU consumption in the event that the Secrets Module is unavailable.

