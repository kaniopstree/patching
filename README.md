# Patching

# Jenkins-Based Linux Package Patching Pipeline

## Overview

This project implements a parameterized Jenkins Pipeline to automate Linux package patching on remote Ubuntu servers using SSH authentication.

The pipeline allows an operator to specify the target server, package name, and optionally the package version. It validates connectivity, checks the currently installed package version, determines whether the requested version is available in the configured repository, installs or upgrades the package, and verifies the installation.

The solution was developed as a Proof of Concept (POC) and is designed to demonstrate an automated patching workflow that can be extended for enterprise environments.

---

# Features

* Parameterized Jenkins Pipeline
* SSH Key-Based Authentication
* Remote Package Version Validation
* Current Version Verification
* Latest Package Upgrade
* Specific Version Installation (when available)
* Post-Installation Verification
* Jenkins Build Logs for Audit
* Reusable Pipeline for Multiple Servers

---

# Pipeline Parameters

| Parameter       | Description                                                    |
| --------------- | -------------------------------------------------------------- |
| TARGET_HOST     | Target server IP or hostname                                   |
| TARGET_PORT     | SSH Port                                                       |
| SSH_USER        | Remote Linux user                                              |
| PACKAGE_NAME    | Linux package to patch                                         |
| PACKAGE_VERSION | Optional. Leave empty to install the latest repository version |

---

# Architecture

```
                Jenkins

                    │
                    │ SSH
                    ▼

          Remote Ubuntu Server

                    │

     Check Installed Package Version

                    │

     Check Repository Package Version

                    │

        Install / Upgrade Package

                    │

          Verify Installed Version

                    │

              Jenkins Report
```

---

# Pipeline Workflow

## Stage 1 – Connectivity Check

* Verify SSH connectivity.
* Validate authentication.
* Ensure target server is reachable.

Commands executed:

```
hostname
whoami
```

---

## Stage 2 – Package Validation

Check whether the requested package exists in the configured package repository.

Example:

```
apt-cache policy nginx
```

---

## Stage 3 – Current Version

Retrieve the currently installed package version.

Example:

```
dpkg -l | grep nginx
```

---

## Stage 4 – Package Installation / Upgrade

### Latest Version

If PACKAGE_VERSION is empty:

```
apt update
apt install --only-upgrade
```

### Specific Version

If PACKAGE_VERSION is provided:

```
apt install package=version
```

If the requested version is unavailable in the configured repository, the pipeline can be extended to use an external package repository (for example, the official NGINX repository or an internal Nexus/Artifactory repository).

---

## Stage 5 – Verification

Verify the installed package version.

Examples:

```
apt-cache policy nginx

dpkg -l | grep nginx
```

---

## Stage 6 – Patch Summary

The pipeline reports:

* Target Host
* Package Name
* Requested Version
* Installed Version
* Patch Status

---

# Repository Strategy

Current implementation uses the Ubuntu package repository.

Future enhancement:

```
Ubuntu Repository
        │
        ├── Version Available
        │         │
        │         ▼
        │    Install Package
        │
        └── Version Not Available
                  │
                  ▼
       Configure Vendor Repository
                  │
                  ▼
           Download Required Version
                  │
                  ▼
             Install Package
                  │
                  ▼
              Verify Installation
```

This approach enables installation of package versions not present in the default Ubuntu repository.

---

# Security

The pipeline uses:

* SSH Username with Private Key Authentication
* Jenkins Credentials Store
* Passwordless sudo on the target server (POC)

No passwords are stored inside the Jenkins Pipeline.

---

# Prerequisites

* Jenkins
* SSH Agent Plugin
* Ubuntu Target Server
* OpenSSH Server
* Passwordless sudo for the Jenkins user
* SSH Private Key configured in Jenkins Credentials

---

# Example Execution

Parameters:

```
TARGET_HOST=192.168.1.100
TARGET_PORT=22
SSH_USER=ubuntu
PACKAGE_NAME=nginx
PACKAGE_VERSION=
```

Result:

```
Current Version
↓

Package Upgrade

↓

Verification

↓

Patch Summary
```

---

# Current Limitations

* Supports Ubuntu package management using APT.
* Specific package versions must exist in the configured repository.
* External repository integration is planned for packages unavailable in the default repository.
* Multi-server patching is not included in this POC.

---

# Future Enhancements

* Multi-server patching
* Ansible integration
* Dynamic repository selection
* Vendor repository auto-configuration
* Package dependency validation
* Rollback support
* Slack / Email notifications
* Patch reporting
* Maintenance window integration
* Approval workflow before production deployment

---

# Technologies Used

* Jenkins
* Groovy Pipeline
* Ubuntu 24.04
* OpenSSH
* Linux APT Package Manager
* Docker (POC Environment)

---

# Author

Kanimozhi Viswanathan
