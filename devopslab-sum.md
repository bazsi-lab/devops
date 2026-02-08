
author:             saba
version:            v1.0
date:               Januar 25th, 2026

# Teamcity Lab

This lab based on a Rocky Linux 10.1 Linux VM, virtualized with KVM/Qemu on Ubuntu Host.

psa@psa-Precision-7720:~$ neofetch
            .-/+oossssoo+/-.               psa@psa-Precision-7720 
        `:+ssssssssssssssssss+:`           ---------------------- 
      -+ssssssssssssssssssyyssss+-         OS: Ubuntu 24.04.3 LTS x86_64 
    .ossssssssssssssssssdMMMNysssso.       Host: Precision 7720 
   /ssssssssssshdmmNNmmyNMMMMhssssss/      Kernel: 6.8.0-94-generic 
  +ssssssssshmydMMMMMMMNddddyssssssss+     Uptime: 20 hours, 32 mins 
 /sssssssshNMMMyhhyyyyhmNMMMNhssssssss/    Packages: 4107 (dpkg), 49 (flatpak), 
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Shell: bash 5.2.21 
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   Resolution: 3840x2160 
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   DE: GNOME 46.0 
ossyNMMMNyMMhsssssssssssssshmmmhssssssso   WM: Mutter 
+sssshhhyNMMNyssssssssssssyNMMMysssssss+   WM Theme: WhiteSur-Light 
.ssssssssdMMMNhsssssssssshNMMMdssssssss.   Theme: WhiteSur-Light [GTK2/3] 
 /sssssssshNMMMyhhyyyyhdNMMMNhssssssss/    Icons: Yaru [GTK2/3] 
  +sssssssssdmydMMMMMMMMddddyssssssss+     Terminal: gnome-terminal 
   /ssssssssssshdmNNNNmyNMMMMhssssss/      CPU: Intel i7-7820HQ (8) @ 3.900GHz 
    .ossssssssssssssssssdMMMNysssso.       GPU: Intel HD Graphics 630 
      -+sssssssssssssssssyyyssss+-         GPU: NVIDIA Quadro P3000 Mobile 
        `:+ssssssssssssssssss+:`           Memory: 17113MiB / 31941MiB 
            .-/+oossssoo+/-.
                                                                   

The VM has:
    - 8 GB RAM
    - 4 Proccesor

## Project Structure

This project contain 3 VMs, each simulate one or more simple task. The main goal is to build and operate a complete CI/CD pipes, complet automated. This lab give the opurtinity to install, operate and troubleshoot each aspect of lifecycle from the pipelines. To make everything realistic as possible, rhel linux downstream, Rocky Linux is used. 
I have the following structure:

**VM01 - Teamcity Server**

Responsibilities:
- Pull source code from GitHub
- Execute CI Pipelines
- Build container Images
- Run test and validations
- Publish build outputs (artifacts and images)

Core Software:
- Rocky Linux 10.1
- TeamCity Server with 2 Agents (on-premise)
- Podman
- systemd
- build tooling (JDK, Maven/Gradle, Node, etc)

Summary:
- Receives Github webhook / polling trigger
- Checks out source code
- Executes build steps:
    - Compile
    - Test
    - Container image build (Podman)
- Produces immutable artifacts
- Pushes artifacts/images to VM-02

Not responsible for:

- run long-living application containers
- store artifacts permanently
- host databases


**VM02 - Artifactory Server**

Responsibiliteis:
- Store build artifacts
- Serve container images / packages
- Maintain artifact metadata
- Provide controlled access to build outputs
- Persist CI state across rebuilds

Core Software
- JFrog Artifactory
- PostgreSQL (Artifactory DB)
- Podman
- systemd

Summary
- Receives artifacts from TeamCity
- Stores binaries, container layers, metadata
- Acts as the single source of truth for build outputs
- Serves artifacts to runtime systems (VM-03)
- Keeps CI stateless by externalizing storage

Later - Storage separation
- Artifact binaries → large volume
- Database data → separate volume
- Backups possible without CI downtime



**VM03 - Runtime Host**

Responsibilities
- Pull approved artifacts/images
- Run application containers
- Manage lifecycle via systemd
- Restart, monitor, and isolate services

Core Software
- Rocky Linux 10.1
- Podman
- systemd
- Application containers
- Optional monitoring agents

Summary
- Pulls released artifacts from VM-02
- Starts containers via systemd units

Ensures:
- Auto-start on boot
- Restart on failure
- Resource isolation
- Acts as the production-like execution environment

Critical role
- systemd + container integration
- Runtime vs build separation
- Operational troubleshooting
- Real Linux service management

## Lab summary

1. Developer pushes code → GitHub
2. TeamCity (VM-01) is triggered
3. Code is built & tested
4. Container image is created
5. Artifact is published → Artifactory (VM-02)
6. Runtime host (VM-03) pulls artifact
7. systemd starts container
8. Application runs persistently

| VM           | CI | Artifact | DB | Runtime |
| ------------ | -- | -------- | -- | ------- |
| VM-01 (CI)   | ✅  | ❌        | ❌  | ❌       |
| VM-02 (Repo) | ❌  | ✅        | ✅  | ❌       |
| VM-03 (Run)  | ❌  | ❌        | ❌  | ✅       |



