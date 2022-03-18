# Tested Configuration

Server | Platform | OS | Keepalived Version
------ | -------- | -- | ------------------
Primary | Raspberry Pi | Ubuntu 20.04 | 2.2.7
Backup | Virtualized | Debian 11 | 2.1.5
Backup | Raspberry Pi | Raspberry Pi OS (Bullseye) | 2.2.7

!!! bug "Known issue"
    An issue arose when attempting to track the FTL process on an older keepalived version. The above versions have been tested to work successfully.

--8<-- "includes/abbreviations.md"
