# satori-imager
The Imager part of the Satori-Suite

![logo2](https://github.com/satori-ng/Logos/blob/master/logos/light/logo2.png)

[![PyPI version](https://badge.fury.io/py/satori-imager.svg)](https://pypi.org/project/satori-imager) - `pip install satori-imager`


The *Satori Imager* is the tool that creates *Satori Images* from hosts.

### The entrypoint `satori-imager`

```sh
$ satori-imager 
usage: satori-imager [-h] [-e EXCLUDED_DIRS] [-l LOAD_EXTENSIONS] [-q]
                     [-t THREADS] [-r REMOTE]
                     entrypoints [entrypoints ...] image_file
satori-imager: error: the following arguments are required: entrypoints, image_file
```

Imaging the `/etc` directory of the local host in a *Satori Image* file named `etc_dir_simple.json.gz`:
```sh
$ satori-imager /etc etc_dir_simple   # filetype extension (".json.gz") extension is appended automatically
[+] Directory '/etc/audit' could not be listed
[+] Directory '/etc/grub.d' could not be listed
[+] Directory '/etc/firewalld' could not be listed
[+] Directory '/etc/dhcp' could not be listed
[+] Directory '/etc/sudoers.d' could not be listed
[+] Directory '/etc/libvirt' could not be listed
[+] Directory '/etc/sssd' could not be listed
[+] Directory '/etc/audisp' could not be listed
[+] Directory '/etc/cups/ssl' could not be listed
[+] Directory '/etc/lvm/archive' could not be listed
[+] Directory '/etc/lvm/cache' could not be listed
[+] Directory '/etc/lvm/backup' could not be listed
[+] Directory '/etc/openvpn/client' could not be listed
[+] Directory '/etc/openvpn/server' could not be listed
[+] Directory '/etc/polkit-1/localauthority' could not be listed
[+] Directory '/etc/polkit-1/rules.d' could not be listed
[+] Directory '/etc/vmware-tools/GuestProxyData/trusted' could not be listed
[+] Processed 2064 files
[+] Image Generated!
[!] Stored to file 'etc_dir_simple.json.gz'
```
*Satori Imager* fails to list several directories due to privileges:
```sh
$ ls -l /etc/ | grep sssd
drwx------.  3 root root     4096 Feb 17 20:51 sssd
```


#### And now it can inspected with `satori-file` (from [satori-core](https://github.com/satori-ng/satori-core)):
```json
$ satori-file etc_dir_simple.json.gz
{
 "metadata":{
  "uuid":"e4d131b3-2ace-4e0a-be74-528c892a910c",
  "system":{
   "type":"Linux",
   "platform":"Linux-4.15.3-300.fc27.x86_64-x86_64-with-fedora-27-Twenty_Seven",
   "hostname":"FkUkO43a",
   "machine":"x86_64",
   "release":"4.15.3-300.fc27.x86_64",
   "processor":"x86_64",
   "user":"user1"
[...]
},
  "data":{
    "filesystem":{
     "/":{
      "type":"D",
      "contents":{
       "etc":{
        "type":"D",
        "contents":{
         "gdbinit.d":{
          "type":"D",
          "contents":{},
          "stat":{
           "blksize":4096,
           "blocks":8,
           "dev":64768,
           "gid":0,
           "ino":1048304,
           "mode":16877,
           "nlink":2,
           "rdev":0,
           "size":4096,
           "uid":0
          },
 [...]
 ```
##### This is actually my dev machine

----

### Running extensions upon Imaging
The `-l` argument loads extensions (several can already be found in [satori-extensions repo](https://github.com/satori-ng/satori-extensions)).
Typical example is the need to *compute hashes* for each file in a folder:
```sh
$ satori-imager /etc etc_dir_simple.md5 -l satori-extensions/hash/md5.py
[!] Extension 'md5' loaded                
[...]
[+] [Errno 13] Permission denied: '/etc/shadow' . File '/etc/shadow' could not be opened.
[...]
[+] Processed 2064 files
[+] Image Generated!
[!] Stored to file 'etc_dir_simple.md5.json.gz'
```
The `md5.py` extension uses the `with_open` hook, and forces the *Satori Imager* to use `open()` to all files, in order to retrieve (and calculate the hash) of the contents.

But the `/etc/shadow` cannot be opened and read by non-root users, hence the `Errno 13` warning...

#### Check:
```sh
$ satori-file etc_dir_simple.json.gz | gron | grep md5
json.data.filesystem["/"].contents.etc.contents.DIR_COLORS.md5 = "9856a682b9e1ac6ecaac082d35f8858c";
json.data.filesystem["/"].contents.etc.contents.GREP_COLORS.md5 = "6dae2e66d0089d8479b14855d4203497";
json.data.filesystem["/"].contents.etc.contents.NetworkManager.contents.VPN.contents["nm-openconnect-service.name"].md5 = "d5077be47811e9be2a6efc7015d36d18";
json.data.filesystem["/"].contents.etc.contents.NetworkManager.contents.VPN.contents["nm-openvpn-service.name"].md5 = "0b93d73970088faff0b455bad04b78fa";
json.data.filesystem["/"].contents.etc.contents.NetworkManager.contents.VPN.contents["nm-pptp-service.name"].md5 = "56856ed0df336411d802ea364f4d8960";
json.data.filesystem["/"].contents.etc.contents.NetworkManager.contents.VPN.contents["nm-ssh-service.name"].md5 = "631a3c73c7e27a5a98b929aa616a0398";
json.data.filesystem["/"].contents.etc.contents.NetworkManager.contents.VPN.contents["nm-vpnc-service.name"].md5 = "d56f8a6ad32c94a1d28503a6950cefc3";
[...]
```
[`gron` can make JSON greppable](https://github.com/tomnomnom/gron) 
----

### Going remote!
If [`satori-remote`](https://github.com/satori-ng/satori-remote) package is available, the *Satori Imager* can accept the `-r` argument.
The `-r` (or `--remote`) accepts URIs. For `SSH/SFTP` protocol to the remote host 10.5.0.12, it will be:

```sh
$ satori-imager /etc etc_dir_10_5_0_12.md5 -l ../satori-extensions/hash/md5.py -r ssh://10.5.0.12
[+] Connecting to 'ssh://10.5.0.12'
Username: user1
Password:
[!] Connected to 10.5.0.12
[!] Extension 'md5' loaded
[...]
[+] Processed 2064 files
[+] Image Generated!
[!] Stored to file 'etc_dir_10_5_0_12.md5.json.gz'
```
And the `etc_dir_10_5_0_12.md5.json.gz` contains data about the `/etc` of the remote machine. No agents installed, no puppies harmed.
