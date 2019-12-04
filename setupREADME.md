## Find binaries on VM
```
cd /opt/protostar/bin
./stack0
```
## Get Static Analysis tool
```
$ git clone https://github.com/radare/radare2.git
$ cd radare2
$ ./sys/install.sh
```
## Extract pre-built binaries
`scp -r user@192.168.0.76:/opt/protostar/bin/ ~/Security/app_binaries`
## Network
### SSH in Bridged mode
`Bridged mode` - Your guest machine gets an IP Address on the same subnet as your host.
I have a Mac on which I had installed VirtualBox.
`ssh user@192.....`

But if you have locked down wifi, you may not be given an IP address if it detects you are bridged.

##### Find I.P. address
```
arp -a    // won't help identify your Guest I.P. address.
ip -4 addr | grep -oP '(?<=inet\s)\d+(\.\d+){3}'    //   On the Guest
VBoxManage guestproperty enumerate <name of VM>     // On the Host
VBoxManage guestproperty get <name of VM> "/VirtualBox/GuestInfo/Net/0/V4/IP" // on the Host
```
### SSH in NAT Mode
You can SSH into the Guest VM with `NAT` after setup a `Port Forwarding` rule.  
```
ssh -p 1111 user@localhost

Network -> Adapter 1(by default have attached to as NAT) -> Advanced -> Port Forwarding // add a new entry with the following settings

Host Port: 1111, Guest Port: 22, leave the host IP and guest IP blank

sudo lsof -iTCP -sTCP:LISTEN -n -P    // Check the `Port Forwarding` is setup correctly
```
### References
```
https://blog.lamarranet.com/index.php/exploit-education-phoenix-setup/
https://stackabuse.com/how-to-fix-warning-remote-host-identification-has-changed-on-mac-and-linux/
https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture
https://download.virtualbox.org/virtualbox/
```
