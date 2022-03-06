### **Issue**

HD Video Playback in default browser Firefox is stuttering

### **Hardware**
```
Intel(R) Core(TM) i7-4790 CPU @ 3.60GHz
nVidia GP107 [GeForce GTX 1050]
```

### **Troubleshooting** 

A quick google of the problem shows this isnt uncommon, and seems to be related to Hardware Video Acceleration not being enabled by default in browsers.

Ubuntuhandbook.org has a step-by-step breakdown to enable this in Firefox, I'll be specifically following the VA-API steps as that seems to apply to most systems

1. The first step is to install the related package named `vainfo` so I'll install that via the terminal:

``` 
sudo apt install vainfo 
```

2. Running `vainfo` will output some information that could be important:

```
vainfo
```

And my output: 
```
$ vainfo
libva info: VA-API version 1.7.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/nouveau_drv_video.so
libva info: Found init function __vaDriverInit_1_7
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.7 (libva 2.6.0)
vainfo: Driver version: Mesa Gallium driver 21.2.6 for NV137
vainfo: Supported profile and entrypoints
      VAProfileNone                   :	VAEntrypointVideoProc
```
VAProfileNone doesnt seem right, so that led me down a path to see where others were having a similar issue and were also sharing `dmesg` outputs.

Running `dmseg` myself found a couple errors, most notably:
```
[    2.397408] nouveau 0000:01:00.0: bus: MMIO read of 00000000 FAULT at 122124 [ PRIVRING ]
```
Seems like a driver issue, likely just not compatible with the admittedly older GPU in my system. Instead of removing the GPU and using integrated Intel graphics, I'll try to install a compatible driver even if it is proprietary and not open source. 

3. Run `sudo ubuntu-drivers devices`:

Output:

 ```
$ sudo ubuntu-drivers devices
WARNING:root:_pkg_get_support nvidia-driver-390: package has invalid Support Legacyheader, cannot determine support level
== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00001C81sv00001458sd00003747bc03sc00i00
vendor   : NVIDIA Corporation
model    : GP107 [GeForce GTX 1050]
driver   : nvidia-driver-450-server - distro non-free
driver   : nvidia-driver-390 - distro non-free
driver   : nvidia-driver-418-server - distro non-free
driver   : nvidia-driver-470-server - distro non-free
driver   : nvidia-driver-470 - distro non-free
driver   : nvidia-driver-510 - distro non-free recommended
driver   : xserver-xorg-video-nouveau - distro free builtin
```
4. Let's install the recommended driver listed as "nvidia-driver-510 - distro non-free recommended," but first, let's remove the old driver to avoid any issues:

```
$ sudo apt update
$ sudo apt upgrade
$ sudo apt autoremove
$ sudo apt-get remove --purge nvidia*
$ sudo apt-get remove --purge "nvidia*"
```
Then:

```
sudo apt install nvidia-driver-510
sudo reboot
```
`dmesg` no longer shows the error message. However, `vainfo` now returns 

```
$ vainfo
libva info: VA-API version 1.7.0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/nvidia_drv_video.so
libva info: va_openDriver() returns -1
vaInitialize failed with error code -1 (unknown libva error),exit
```


So I am still not done yet.













### *******[WIP/future steps]** 
