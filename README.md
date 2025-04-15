# Hyper-V using powershell

### Hyper-V : Create our network
In order for us to create virtual machines all on the same network, I am going to create a virtual switch in Hyper-v
Open Powershell in administrator
```
# get our network adapter where all virtual machines will run on
# grab the name we want to use
Get-NetAdapter

Import-Module Hyper-V
$ethernet = Get-NetAdapter -Name "Ethernet 2"
New-VMSwitch -Name "virtual-network" -NetAdapterName $ethernet.Name -AllowManagementOS $true -Notes "shared virtual network interface"
```
### Hyper-V : Create our machines
We firstly need harddrives for every VM.
Let's create three:
```
mkdir c:\temp\vms\linux-0\
mkdir c:\temp\vms\linux-1\
mkdir c:\temp\vms\linux-2\

New-VHD -Path c:\temp\vms\linux-0\linux-0.vhdx -SizeBytes 20GB
New-VHD -Path c:\temp\vms\linux-1\linux-1.vhdx -SizeBytes 20GB
New-VHD -Path c:\temp\vms\linux-2\linux-2.vhdx -SizeBytes 20GB
```
```
New-VM `
-Name "linux-0" `
-Generation 1 `
-MemoryStartupBytes 2048MB `
-SwitchName "virtual-network" `
-VHDPath "c:\temp\vms\linux-0\linux-0.vhdx" `
-Path "c:\temp\vms\linux-0\"

New-VM `
-Name "linux-1" `
-Generation 1 `
-MemoryStartupBytes 2048MB `
-SwitchName "virtual-network" `
-VHDPath "c:\temp\vms\linux-1\linux-1.vhdx" `
-Path "c:\temp\vms\linux-1\"

New-VM `
-Name "linux-2" `
-Generation 1 `
-MemoryStartupBytes 2048MB `
-SwitchName "virtual-network" `
-VHDPath "c:\temp\vms\linux-2\linux-2.vhdx" `
-Path "c:\temp\vms\linux-2\"
```
Setup a DVD drive that holds the iso file for Ubuntu Server
```
Set-VMDvdDrive -VMName "linux-0" -ControllerNumber 1 -Path "C:\temp\ubuntu-20.04.3-live-server-amd64.iso"
Set-VMDvdDrive -VMName "linux-1" -ControllerNumber 1 -Path "C:\temp\ubuntu-20.04.3-live-server-amd64.iso"
Set-VMDvdDrive -VMName "linux-2" -ControllerNumber 1 -Path "C:\temp\ubuntu-20.04.3-live-server-amd64.iso"
```
Start our VM's
```
Start-VM -Name "linux-0"
Start-VM -Name "linux-1"
Start-VM -Name "linux-2"
```
### Hyper-V : Setup SSH for our machines
Now in this demo, because I need to copy rancher bootstrap commands to each VM, it would be easier to do so using SSH. So let's connect to each VM in Hyper-V and setup SSH.
This is because copy+paste does not work without Enhanced Session mode in Ubuntu Server.

Let's temporarily turn on SSH on each server:
```
sudo apt update
sudo apt install -y nano net-tools openssh-server
sudo systemctl enable ssh
sudo ufw allow ssh
sudo systemctl start ssh
```
Record the IP Address of each VM so we can SSH into it.
```
sudo ifconfig
# record eth0
linux-0 IP=192.168.0.16
linux-1 IP=192.168.0.17
linux-2 IP=192.168.0.18
```
In new powershell window, let's SSH to our VMs
```
ssh linux-0@192.168.0.16
ssh linux-1@192.168.0.17
ssh linux-2@192.168.0.18
```
