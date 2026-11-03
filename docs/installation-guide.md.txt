# Splunk Enterprise Installation Guide

## 1. Objective

Deploy Splunk Enterprise on an Ubuntu Server virtual machine to establish the centralized SIEM platform for a home SOC lab.

## 2. Lab Environment

| Component | Configuration |
|---|---|
| Hypervisor | Oracle VirtualBox |
| Operating System | Ubuntu Server 26.04.1 LTS |
| Architecture | AMD64 |
| SIEM Platform | Splunk Enterprise 10.6.1.1 |
| Virtual Disk | 40 GB |
| Root Filesystem | Approximately 38 GB |
| Available Memory at verification | Approximately 3.8 GiB |
| Network Adapter 1 | NAT |
| Network Adapter 2 | Host-only |

## 3. Create the Ubuntu Virtual Machine

Created a virtual machine in Oracle VirtualBox with the following initial configuration:

- VM Name: `SOC-Splunk-Ubuntu`
- Operating System: Ubuntu Server
- Virtual Disk: 40 GB, dynamically allocated
- Network Adapter 1: NAT
- Network Adapter 2: Host-only Adapter

Ubuntu was installed using the official Ubuntu Server installation image.

A dedicated Linux account was configured for server administration.

## 4. Configure VirtualBox Networking

Two network adapters were configured.

### Adapter 1 — NAT

Purpose:
- Provide outbound internet connectivity.
- Allow Ubuntu to download system updates and packages.

### Adapter 2 — Host-only

Purpose:
- Establish private communication between the Windows host and Ubuntu VM.
- Allow secure SSH and SCP file transfers within the lab.

The host-only interface received an IPv4 address through DHCP.

Example interface configuration:

| Interface | Purpose |
|---|---|
| `enp0s3` | NAT |
| `enp0s8` | Host-only |

Useful verification commands:

```bash
ip -br addr
ip route
networkctl status enp0s8
```

## 5. Expand the Ubuntu Root Filesystem

The initial Ubuntu installation used approximately 19 GB for its root logical volume, while additional capacity remained available in the LVM volume group.

Inspected the storage configuration:

```bash
sudo pvs
sudo vgs
sudo lvs
```

Expanded the root logical volume:

```bash
sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv
```

Verified the result:

```bash
df -h /
```

The root filesystem increased to approximately 38 GB.

## 6. Update Ubuntu

Refreshed package repositories:

```bash
sudo apt update
```

Installed available updates:

```bash
sudo apt upgrade -y
```

Installed basic utilities:

```bash
sudo apt install -y wget curl net-tools
```

## 7. Download and Transfer Splunk Enterprise

Downloaded the Splunk Enterprise Debian package for Linux AMD64 from the official Splunk website.

Installer:

```text
splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb
```

Transferred the package from the Windows host to Ubuntu using SCP over the Host-only network.

Example:

```powershell
scp "D:\SOC-LAB\ISO\Splunk\splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb" socadmin@UBUNTU_HOST_ONLY_IP:~/splunk-install/
```

Verified the transferred package:

```bash
ls -lh ~/splunk-install/
```

## 8. Install Splunk Enterprise

Navigated to the installer directory:

```bash
cd ~/splunk-install
```

Installed Splunk using APT:

```bash
sudo apt install ./splunk-10.6.1.1-b2ae2962645c-linux-amd64.deb
```

Verified the installed version:

```bash
sudo /opt/splunk/bin/splunk version
```

Expected version:

```text
Splunk 10.6.1.1
```

## 9. Configure a Dedicated Splunk Service Account

Running Splunk as root generated a deprecation warning.

Checked the dedicated service account:

```bash
id splunk
```

Corrected ownership of the Splunk installation directory:

```bash
sudo chown -R splunk:splunk /opt/splunk
```

Started Splunk using the non-root service account:

```bash
sudo -u splunk /opt/splunk/bin/splunk start --accept-license
```

Configured the Splunk administrator account during first startup.

**Security note:** The Splunk administrator credentials are stored privately and are not included in this repository.

## 10. Verify Splunk Services

Checked Splunk status:

```bash
sudo -u splunk /opt/splunk/bin/splunk status
```

Checked listening ports:

```bash
sudo ss -lntp | grep -E ':8000|:8089'
```

Verified the Splunk version:

```bash
sudo -u splunk /opt/splunk/bin/splunk version
```

### Important Ports

| Port | Service | Purpose |
|---|---|---|
| TCP 8000 | Splunk Web | Browser-based administration and searches |
| TCP 8089 | Splunk Management Interface | Management API and administrative communication |

## 11. Access Splunk Web

Accessed the Splunk Web interface from the Windows host using:

```text
http://UBUNTU_HOST_ONLY_IP:8000
```

Logged in using the Splunk administrator account created during initial startup.

## 12. Implementation Results

The following deployment checks were completed:

- Ubuntu Server deployed successfully.
- NAT and Host-only networking configured.
- Root filesystem expanded using LVM.
- Splunk Enterprise 10.6.1.1 installed.
- Dedicated non-root Splunk service account configured.
- Splunk daemon confirmed running.
- TCP ports 8000 and 8089 confirmed listening.
- Splunk Web successfully accessed from the Windows host.

**Milestone Status: Completed**


