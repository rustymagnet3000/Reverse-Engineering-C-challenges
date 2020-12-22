# Exploring vulnerable C APIs
## Overview
This repo is a collection of notes from `crackme` challenges.

I used three tools:

- a debugger ( `gdb` with the `gef` extension )
- a command line disassembler ( `radare2` )
- A visual disassembler ( `Ghidra` )

Don't run these challenges on your normal computer.  Either setup a throwaway `Virtual Machine` or, better still, a `Docker Image`.

### Exploitable APIs
The Internet is full of excellent references to vulnerabilities that can be exploited with `C functions`.  In 2020, C-related vulnerabilities were most of the [top 25 issues](https://cwe.mitre.org/).  Most of the  APIs are well known:

- `gets`      -> get user input
- `strcpy`    -> copy a string
- `printf`    -> print formatted strings
- `sprintf`   -> Composes a string with the same text that would be printed
- `scanf`     -> gets user input and then composes a string with the given format

You use these APIs to trigger a `Buffer Overflow ` or `Format String Vulnerability`.  A great background paper on these types of vulnerabilities can be found at [Stanford](https://crypto.stanford.edu/cs155old/cs155-spring08/papers/formatstring-1.2.pdf):

`Code scanning tools` or `Development Environments` would alert on an insecure function being used.  Or - like on `macOS` - the compiler / linker would switch the function for a stricter alternative.  But the `crackme` challenges typical ensured all safety controls were off.

### Finding Reverse Engineering C challenges
Most offer a `Virtual Machine` so you play without worry.
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

## Docker - setup a challenge machine 4 commands
On MacOS, with `Docker Desktop` installed you could easily find a pre-canned machine. Easy to setup and run. Already have reversing tools pre-installed.
##### Find image you like
https://hub.docker.com/r/duckll/ctf-box/
##### pull docker image
`docker pull duckll/ctf-box`
##### Run
`docker run -idt --name ctf duckll/ctf-box`
##### Start the container
`docker start ctf`
##### Start the container with a bash terminal ( cut and paste allowed )
`docker container exec -it ctf bash`
### Extend Container with a 32-bit compiler
##### Update container
`apt-get update`
##### Install multi-architecture compiler
`apt install gcc-4.8-multilib`
##### Compile for 32 bit machine
`gcc-4.8 -m32 -o format-two-32b format_two.c`
##### Check file format
`file format-two-32b`     # format-two-32b: ELF 32-bit LSB executable
### Extend Container to compile ARM code
##### Install cross-compiler
`apt-get install gcc-arm-linux-gnueabi`
##### Extras
`apt-get install libc6-armel-cross libc6-dev-armel-cross binutils-arm-linux-gnueabi libncurses5-dev`
##### Required, generic header files
`sudo apt-get install gcc-arm-linux-gnueabihf g++-arm-linux-gnueabihf`
##### Compile for arm
`arm-linux-gnueabi-gcc -o arm_f2 format_two.c`



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
