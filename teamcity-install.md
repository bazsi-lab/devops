
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

- this VM responsible for the Artifacts, also act als Artifactory Server
- store and serve images, binaries etc. 

**VM03 - Runtime Host**

- this VM run containers, which are managed via systemd
- pull images/artifacts from vm02



# Content

