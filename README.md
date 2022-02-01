# Proxmox-qemu-audio
Rebuiild proxmox qemu with audio spport ( jack, alsa etc)


Add proxmox development repo to soruces
```
nano /etc/apt/sources.list
```

```
# proxmox development repo
deb http://download.proxmox.com/debian/devel/ bullseye main
```
Then 

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

Add jack ( or alternatives ) to build rules
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
control + x and save.


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

list files in current directory and make a note of the pve-qemu-kvm_ version number

`
ls
`

install your custom compiled pve-qemu ( where #.#.#.# is the version number previosly noted)

```
apt install ./pve-qemu-kvm_#.#.#.#.deb
```


Add the folowing to vm config args line
```
nano /etc/pve/qemu-server/VMID.conf
```
```
-audiodev id=jack,driver=jack -device ich9-intel-hda -device hda-duplex,audiodev=jack
```


## The folowing is to configure Jack server.  i beleave using pipwwire as a jack server would be better for most people, however as of now i have been unable to get this to work .  please let me know via issues if you know how and i can update this guide.


Create a service to start jackd automaticaly on host boot.

```
nano /etc/systemd/system/jack.service
```

and enter

```
[Unit]
Description=JACK server
After=sound.target local-fs.target

[Service]
Type=notify
user=root
#ExecStart=/root/startjack.sh 
#ExecStart=nohup jackd -d alsa -d hw:0 -r48000 -p256 -n2 &
ExecStart=/bin/jackd -d alsa -d hw:0 -r48000 -p256 -n2
LimitRTPRIO=95
LimitRTTIME=infinity
LimitMEMLOCK=infinity
Environment="JACK_NO_AUDIO_RESERVATION=1"
[Install]
WantedBy=default.target
```
Customise -d hw:0,  -r48000 ( sample rate) -p256 ( buffer size) and -n2 ( number of buffers?)  as needed.Then control + x and save.

then

```
systemctl daemon-reload
systemctl enable jack.service
systemctl start jack.service
systemctl status jack.service
```
youu should see that Jack server is active with no errors.

Restart any VM's using a jack audiodev. 

you can now use x11 fowarding to access qjackctl 

```
ssh -X root@**host name or ip o fproxmox host**
```
Click graph to set up audio routing. 
Once you are happy with your routing,  click patch bay > new > yes  then save.  
Then click setup > options,  tick "Activate patchbay persistence and slect your previouly saved patch file.



## references
pipewire instal instructions (reverse engineerd the unlinking)
https://wiki.debian.org/PipeWire

gnif's jack qemu instructions:
https://forum.level1techs.com/t/qemu-native-jack-audio-support/156494

debian services wiki
https://wiki.debian.org/systemd/Services
