# Explore vulnerable C APIs
## Overview
This repo is a collection of notes from `crackme` challenges using vulnerable C APIs or disassemblers.  

I used three tools for these challenges:

- radare2 (disassembler)
- gdb (debugger)
- Ghidra (tools developed by the NSA)

Don't run these challenges on your normal computer.  Either setup a throwaway `Virtual Machine` or `Docker Image`.  Instructions below.

Give some thought to the instruction set you want to understand.  I used `ARM (32 bit)` binaries, where I could.

### Reverse Engineering C challenges
The Internet is full of excellent references to vulnerabilities in C.  In 2020, C-related vulnerabilities were most of the [top 25 issues](https://cwe.mitre.org/).  Most of the vulnerable APIs are well known:

- `gets`
- `strcpy`
- `printf`
- `sprintf`
- `scanf`
- `access`

You can use these insecure APIs to complete online `Memory Corruption` challenges.  Most offer a `Virtual Machine` so you play without worry.
```
https://crackmes.one/
https://exploit.education/
https://ropemporium.com/
http://smashthestack.org/
https://www.hackthebox.eu/
https://www.virtualhackinglabs.com/
https://www.vulnhub.com/
https://azeria-labs.com/part-3-stack-overflow-challenges/
https://www.fuzzysecurity.com/tutorials/expDev/1.html
https://samsclass.info/
```
### Write-ups
```
https://www.lucas-bader.com/ (exploit.education Phoenix)
https://blog.lamarranet.com/index.php/ ( exploit.education Phoenix and ROP Emporium )
```
### References
```
https://www.coengoedegebure.com/buffer-overflow-attacks-explained/
https://security.web.cern.ch/security/recommendations/en/codetools/c.shtml
https://wiki2.org/en/Buffer_overflow_protection#Clang/LLVM
https://cwe.mitre.org/data/definitions/120.html
https://stackoverflow.com/questions/35659152/stack-based-buffer-overflow-challenge-in-c-using-scanf-with-limited-input
```


## Setup a Virtual Machine for challenges ( with ARM instructions, on MacOS )
- [x] Get VirtualBox: https://www.virtualbox.org/wiki/Downloads
- [x] Get Ubuntu image: https://ubuntu.com/download/
- [x] Specify `minimal` Ubuntu installation
- [x] Set `Network` mode to `NAT`. Then you can `ssh` into the `Guest VM` without an `IP Address`.  

  You will need to set a `Port Forwarding` rule:

```
ssh -p 1111 user@localhost
```
To set the Port Forwarding rule inside of VirtualBox:
```
Network -> Adapter 1(by default have attached to as NAT) -> Advanced -> Port Forwarding
Host Port: 1111, Guest Port: 22, leave the host IP and guest IP blank
```
Then confirm it is working on the host machine:
```
sudo lsof -iTCP -sTCP:LISTEN -n -P    // Check 1111 `Port Forwarding` is running
```

I would not recommend running the Linux guest in `Bridged mode`.  Although the guest machine get an IP Address on the same subnet - with `Bridged mode` - your host could hit roadblocks:

  - On locked down wifi, your Guest may not be given an IP address.
  - If you want to connect in Airplane mode, Bridge mode does not work.
  - You have to find your IP address.

### VM Hard Disk size
- [x] Set allocated hard disk size to `12-15GB`. It defaulted to `10GB`, and I ran out of space.
- [x] I moved to a `Fixed size` Hard Disk, over `Dynamically Allocated` as it was quicker.
- [x] If you made an error, you could resize with `VBoxManage modifyhd /phoenixUbuntu.vdi --resize 15000`.  After changing disk size, you needed to reallocate space by loading the Guest VM and using `gparted`.

### Reclaim space
On the Virtual Machine, type this:
```
df -H
Filesystem      Size  Used Avail Use% Mounted on
udev            2.6G     0  2.6G   0% /dev
tmpfs           507M  1.1M  506M   1% /run
/dev/sda1        11G   10G  605M   100% /

sudo apt-get autoclean
sudo apt-get autoremove
sudo apt-get clean
```
That freed up unused packages in: `/var/cache/apt/archives`. It gave me  space to Boot Ubuntu and use `gparted`.

### Ubuntu Guest
- [x] Check O/S version: `lsb_release -d`
- [x] Get IP address: `ip a`
- [x] Check SSH turned on: `sudo systemctl status ssh`
- [x] Install SSH: `sudo apt install openssh-server`
- [x] Unblock firewall: `sudo ufw allow ssh`
- [x] Get ARM emulator `apt-get install qemu`
- [x] Download the `QCOW2` image: https://exploit.education/downloads `(AMD64 (also i486))`

### Prepare the Debugger
Use `gef` instead of vanilla `gdb`. The following command silences `ASCII` related `gef` errors.
```
sudo sed -i 's/\\u27a4 />/g' /etc/gdb/gef.py
```
### Run ARM binary
```
// login to Phoenix
user: user
password: user
/opt/phoenix/arm
./stack_zero
```
### Get Static Analysis tool
```
$ git clone https://github.com/radare/radare2.git
$ cd radare2
$ ./sys/install.sh
```
### File exchange
```
// From Host to Guest
scp -P 1111 /Users/../file user@localhost:/home/user/NewFile

// From Guest to Host
scp -r user@192.168.0.78:/opt/phoenix/arm ~/app_binaries

```
### References
```
http://www.iet.unipi.it/p.perazzo/teaching/cybsec/LAB.01.Phoenix_setup.pdf
https://blog.lamarranet.com/index.php/exploit-education-phoenix-setup/
https://stackabuse.com/how-to-fix-warning-remote-host-identification-has-changed-on-mac-and-linux/
https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture
https://www.tecmint.com/parted-command-to-create-resize-rescue-linux-disk-partitions/
```
