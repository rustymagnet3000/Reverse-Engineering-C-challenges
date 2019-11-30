### Find binaries on VM
```
cd /opt/protostar/bin
./stack0
```
### Get Static Analysis tool radare2
```
$ git clone https://github.com/radare/radare2.git
$ cd radare2
$ ./sys/install.sh
```
### Host machine
Check your Guest is NOT Listening on a Port

`sudo lsof -iTCP -sTCP:LISTEN -n -P`
### Extract pre-built binaries
`scp -r user@192.168.0.76:/opt/protostar/bin/ ~/Security/app_binaries`
### VM in Bridged mode
`Bridged mode` - Your guest machine gets an IP Address on the same subnet as your host.
I have a Mac on which I had installed VirtualBox.
`ssh user@192.....`

You can use NAT but you need to setup a `Port Forwarding` rule.  

`Network -> Adapter 1(by default have attached to as NAT) -> Advanced -> Port Forwarding` and add a new entry with the following settings:
```
Host Port: 1111, Guest Port: 22, leave the host IP and guest IP blank
```
### Find VM I.P. address
`arp -a` won't help identify your Guest I.P. address.

`ip -4 addr | grep -oP '(?<=inet\s)\d+(\.\d+){3}'`

### Hotkeys
```
up arrow)
Ctrl+R: remember
```
### References
```
https://stackabuse.com/how-to-fix-warning-remote-host-identification-has-changed-on-mac-and-linux/
https://en.wikibooks.org/wiki/X86_Assembly/X86_Architecture
```
