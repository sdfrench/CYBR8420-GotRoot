# Designing for Software Security Engineering

## Level-0 Diagram
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/L0_Bitwarden.png)

## Level-1 DFDs and Identified Threats
![](https://github.com/caseyschmitz/CYBR8420-GotRoot/blob/master/Images/Bitwarden_DFD.pdf)


## Threats Requiring Further Investigation

### Threat 2

### Threat 3


### Threat 7

### Threat 20

### Threat 22

### Threat 26
The Bitwarden CLI client makes singular read/write attempts to a json file on the local file system that, despite not appearing to have graceful error-handling, should not consume resources in an excessive manner. The client will simply fail to read/write this file and fail to operate as intended.

### Threat 31
The Bitwarden CLI client does not perform any monitoring between the Secrets Module and the Bitwarden Server to prevent excessive disk/CPU consumption in the event that the Bitwarden Server is unavailable. However, the client makes singular requests to the Bitwarden Server that, upon failure, do not prompt additional requests to services on the Bitwarden Server.

### Threat 48
The Bitwarden CLI client does not perform any monitoring between the user and the Secrets Module to prevent excessive disk/CPU consumption in the event that the Secrets Module is unavailable.
