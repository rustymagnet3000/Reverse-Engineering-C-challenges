## Setup
- [x] Get VirtualBox: https://www.virtualbox.org/wiki/Downloads
- [x] Get Ubuntu image: https://ubuntu.com/download/
- [x] Specify `minimal` Ubuntu installation
- [x] Set `Network` in the Ubuntu profile to `Bridged` (so you will ssh using the `IP address`)

### VM Hard Disk size
- [x] Set allocated hard disk size to 12-15GB. It defaulted to 10GB, and I ran out of space
- [x] I moved to a `Fixed size` Hard Disk, over `Dynamically Allocated` as it was quicker
- [x] If you make an error, you can resize your `VBoxManage modifyhd /phoenixUbuntu.vdi --resize 15000`
- [x] After changing disk disk size, you need to reallocate space by loading the Guest and using `gparted`.

### VM Ubuntu check space
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
That freed up unused packages in: `/var/cache/apt/archives`. It gave me enough space to Boot Ubuntu and then use `gparted` which freed up a lot of space.

### Ubuntu Guest
- [x] Check O/S version: `lsb_release -d`
- [x] Get IP address: `ip a`
- [x] Check SSH turned on: `sudo systemctl status ssh`
- [x] Install SSH: `sudo apt install openssh-server`
- [x] Unblock firewall: `sudo ufw allow ssh`
- [x] Get ARM emulator `apt-get install qemu`
- [x] Download the QCOW2 image: https://exploit.education/downloads ``(AMD64 (also i486))``

### Host
```
ssh user@192.168.0.78
$ cd exploit-education-phoenix-arm64/
$ ./boot-exploit-education-arm64.sh

// Prep for GDB.  Phoenix is set to use gef instead of vanilla gdb. Use this command to silence ASCII related errors.

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
### Extract binaries for Analysis
```
// From Host to Guest
scp -r user@192.168.0.78:/opt/phoenix/arm ~/app_binaries

// From Guest to Host
scp ~/foobar.txt hostUser@192.168.0.37:~
scp -r /opt/phoenix/arm hostUser@192.168.0.37:~/
```
### Tips
`Bridged mode` - Your guest machine gets an IP Address on the same subnet as your host. But if you have locked down wifi, you may not be given an IP address if it detects you are bridged.

```
arp -a    // won't help identify your Guest I.P. address.
ip -4 addr | grep -oP '(?<=inet\s)\d+(\.\d+){3}'    //   On the Guest
VBoxManage guestproperty enumerate <name of VM>     // On the Host
VBoxManage guestproperty get <name of VM> "/VirtualBox/GuestInfo/Net/0/V4/IP" // on the Host

// SSH into the Guest VM with `NAT` after setup a `Port Forwarding` rule:

ssh -p 1111 user@localhost

Network -> Adapter 1(by default have attached to as NAT) -> Advanced -> Port Forwarding // add a new entry with the following settings

Host Port: 1111, Guest Port: 22, leave the host IP and guest IP blank

sudo lsof -iTCP -sTCP:LISTEN -n -P    // Check the `Port Forwarding` is setup correctly

// Gparted Command Line:
sudo parted
print                                                            
Model: ATA VBOX HARDDISK (scsi)
Disk /dev/sda: 13.6GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  ext4         boot

resizepart                                                 

Partition number? 1                                                       

End?  [10.7GB]? 13000
```

### References
```
http://www.iet.unipi.it/p.perazzo/teaching/cybsec/LAB.01.Phoenix_setup.pdf
https://blog.lamarranet.com/index.php/exploit-education-phoenix-setup/
https://stackabuse.com/how-to-fix-warning-remote-host-identification-has-changed-on-mac-and-linux/
https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture
https://www.tecmint.com/parted-command-to-create-resize-rescue-linux-disk-partitions/
```
