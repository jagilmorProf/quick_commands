# 🖥️ System Diagnostics & Hardware Discovery Cheat Sheet, Sourced from ChatGPT 5.1

A quick-reference guide for inspecting CPU, RAM, GPU, storage, network interfaces, temperatures, and OS information across Linux and Windows systems.

---

## 📌 Hardware Lookup Table

### Component → Commands

| Component | Linux (Terminal) | Windows (PowerShell) |
|----------|------------------|------------------------|
| **CPU / Basic System Info** | `lscpu` <br> `cat /proc/cpuinfo \| less` <br> `uname -a` | `Get-CimInstance Win32_Processor \| Select Name,NumberOfCores,NumberOfLogicalProcessors` <br> `Get-ComputerInfo \| Select CsName,OsName,OsVersion` <br> `systeminfo` |
| **Motherboard / BIOS** | `sudo dmidecode -t baseboard` <br> `sudo dmidecode -t bios` | `Get-CimInstance Win32_BaseBaseboard \| Select Manufacturer,Product,SerialNumber` <br> `Get-CimInstance Win32_BIOS \| Select SMBIOSBIOSVersion,Manufacturer` |
| **RAM (Size / Slots)** | `free -h` <br> `sudo dmidecode -t memory` | `Get-CimInstance Win32_PhysicalMemory \| Select BankLabel,Capacity,Speed,Manufacturer` <br> `Get-ComputerInfo \| Select CsTotalPhysicalMemory` |
| **GPU** | `lspci \| grep -i vga` <br> `sudo lshw -C display` | `Get-CimInstance Win32_VideoController \| Select Name,AdapterRAM,DriverVersion` |
| **Storage Devices** | `lsblk -o NAME,SIZE,TYPE,MOUNTPOINT` <br> `df -h` <br> `sudo smartctl -a /dev/sdX` | `Get-Disk` <br> `Get-PhysicalDisk` <br> `Get-Volume` |
| **Partitions / Filesystems** | `lsblk -f` <br> `df -Th` | `Get-Partition` <br> `Get-Volume \| Select DriveLetter,FileSystemLabel,FileSystem,SizeRemaining,Size` |
| **Network Interfaces** | `ip a` <br> `nmcli device status` <br> `sudo ethtool eth0` | `Get-NetAdapter` <br> `Get-NetIPAddress` <br> `Test-NetConnection 8.8.8.8` |
| **PCI Devices** | `lspci` <br> `lspci -nnk` | `Get-PnpDevice -PresentOnly \| Sort-Object Class \| Select Class,FriendlyName` |
| **USB Devices** | `lsusb` | `Get-PnpDevice -Class USB` |
| **OS / Distro Info** | `cat /etc/os-release` <br> `hostnamectl` | `Get-ComputerInfo \| Select OsName,OsVersion,OsArchitecture` |
| **Temperature / Sensors** | `sudo sensors` | `Get-Counter -ListSet "Thermal Zone Information"` |
| **Full System Summary** | `sudo lshw -short` <br> `inxi -Fxxxz` | `Get-ComputerInfo` <br> `msinfo32` |

---

## 🔧 Essential Linux Service Commands

## **SSH Setup**
```bash
sudo apt install openssh-server -y
sudo systemctl enable ssh
sudo systemctl start ssh
sudo systemctl status ssh
```
## **UFW Firewall Basics**
```bash
sudo apt install ufw -y
sudo ufw allow ssh
sudo ufw allow 22/tcp
sudo ufw enable
sudo ufw status verbose
```
## **Linux Full Hardware Report**
```
#!/bin/bash
# system_report.sh

OUT="system_report.txt"

{
echo "==== CPU ===="
lscpu
cat /proc/cpuinfo | head

echo -e "\n==== RAM ===="
free -h
sudo dmidecode -t memory

echo -e "\n==== GPU ===="
lspci | grep -i vga
sudo lshw -C display 2>/dev/null

echo -e "\n==== STORAGE ===="
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT
df -h

echo -e "\n==== NETWORK ===="
ip a
nmcli device status 2>/dev/null

echo -e "\n==== PCI ===="
lspci

echo -e "\n==== OS INFO ===="
cat /etc/os-release
uname -a

} > "$OUT"

echo "Report saved to $OUT"
read -p 'Press Enter to exit...'
```
## **Install OpenSSH Server**
```
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```
## **Start the SSH Service**
```
Start-Service sshd
```
## **Enable SSH to Start on Boot**
```
Set-Service -Name sshd -StartupType 'Automatic'
```
## **Allow inbound SSH on port 22**
```
New-NetFirewallRule -Name "SSH Inbound" -DisplayName "SSH Inbound" -Enabled True -Direction Inbound -Protocol TCP -LocalPort 22 -Action Allow
```
## **Allow outbound SSH on port 22**
```
New-NetFirewallRule -Name "SSH Outbound" -DisplayName "SSH Outbound" -Enabled True -Direction Outbound -Protocol TCP -RemotePort 22 -Action Allow
```
## **Verify SSH is Installed & Running**
```
Get-Service sshd
```
## 🧪 Windows PowerShell Hardware Report Script
```# system_report.ps1

$Out = "system_report.txt"

"==== CPU ====" | Out-File $Out
Get-CimInstance Win32_Processor | Out-File $Out -Append

"`n==== RAM ====" | Out-File $Out -Append
Get-CimInstance Win32_PhysicalMemory | Out-File $Out -Append

"`n==== GPU ====" | Out-File $Out -Append
Get-CimInstance Win32_VideoController | Out-File $Out -Append

"`n==== STORAGE ====" | Out-File $Out -Append
Get-Disk | Out-File $Out -Append
Get-Volume | Out-File $Out -Append

"`n==== NETWORK ====" | Out-File $Out -Append
Get-NetAdapter | Out-File $Out -Append
Get-NetIPAddress | Out-File $Out -Append

"`n==== OS INFO ====" | Out-File $Out -Append
Get-ComputerInfo | Out-File $Out -Append

Write-Host "Report saved to $Out"
Read-Host "Press Enter to exit..."
```
