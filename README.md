# Proxmox-qemu-audio
Rebuiild proxmox qemu with audio spport ( jack alsa )


Add proxmox development repo to soruces
```
nano /etc/apt/sources.list
```

```
# proxmox devekopment repo
deb http://download.proxmox.com/debian/devel/ bullseye main
```
 then 

```
apt update
```
clone pve qemu repository recursively and cd to the rood directory
```
 git clone --recursive git://git.proxmox.com/git/pve-qemu.git
 cd pve-qemu
```

get build dependancys

```
apt build-dep .
```
install jack and librarys

```
apt install jackd2 libjack-jackd2-dev
```

Add jack to build rules
```
nano /debian/rules
```
find
```
--audio-drv-list="alsa"
```

change to 
```
--audio-drv-list="alsa jack"
```

if pipewire is installed to replace jack, unlink before compilling qemu

```
rm /etc/ld.so.conf.d/pipewire-jack-*
rm /etc/pipewire/media-session.d/with-jack
ldconfig
```
make deb package

```
run make deb
```


add the folowing to vm config args line
```
nano /etc/pve/qemu-server/VMID.conf
```
```
-audiodev id=jack,driver=jack -device ich9-intel-hda -device hda-duplex,audiodev=jack
```

use x11 fowarding to access qjackctl

```
ssh -X username@hostorip
```
