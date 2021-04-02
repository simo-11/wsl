# Service notes

## Introduction

Notes on required services and logging for running wsl.
Tables using https://www.tablesgenerator.com/markdown_tables

## Related services

| Service  |Before starting bash   | After starting bash | After stoppig bash |
|---|---|---|---|
|LxssManager   | stopped  | running  |   |
|vmcompute |  stopped | running   |   |
|LxssManagerUser   | stopped  | stopped   |   |
|LxssManagerUser_[a-z0-9]{5}   | stopped  | stopped   |   |

## Related Network Drivers
Hyper-V Extensible Virtual Switch is not used in WSL nor Ethernet driver
### vEthernet (WSL)
IP e.g. 172.24.112.1/255.255.240.0
