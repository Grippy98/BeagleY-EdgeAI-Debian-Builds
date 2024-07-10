# BeagleY-EdgeAI-Debian-Builds
A repo to store temp builds of debian packages for TI Vision Apps on BeagleY-AI.

Problem Statement - Default SDK relies on Yocto, FW Builder etc and some values are hard-coded for 8GB of RAM as of the time of writing. 

Latest run against - 
beagle@BeagleY:~$ uname -a
Linux BeagleY 6.1.80-ti-arm64-r57 #1bookworm SMP PREEMPT_DYNAMIC Wed Jun 19 00:28:40 UTC 2024 aarch64 GNU/Linux - clean FS

Goals - 
* Get Vision Apps stack working on BeagleY
* Attempt native compilation wherever possible
* Provide Debian binaries
* Provide documentation on running demos
* Drive feedback back to TI devs. 

Note - Vision-Apps-Debian package currently built from Private repo. Assume .deb package is functional. 
Docker build on target finished in 5 mins and 35 secs.  

**Build Notes**

TODO - Now need to pull in repo examples, test data, etc and run. 

Get edgeai-tiovx-apps

```
cd edgeai-tiovx-apps
export SOC=j722s
mkdir build
cd build
cmake ..
make -j4
```

FAIL- no edgeai_tiovx_img_proc

https://git.ti.com/cgit/edgeai/edgeai-tiovx-modules  - GITHUB VERSION OF SAME OUT OF DATE

```
/opt# cd /opt/edgeai-tiovx-modules
/opt/edgeai-tiovx-modules# mkdir build
/opt/edgeai-tiovx-modules# cd build
/opt/edgeai-tiovx-modules/build# cmake ..
/opt/edgeai-tiovx-modules/build# make -j4
```

sudo git checkout remotes/origin/main

export SOC=j722s

https://git.ti.com/cgit/edgeai/edgeai-tiovx-kernels edgeai-tiovx-kernel dependency

edgeai-apps-utils dependency….

https://git.ti.com/cgit/edgeai/edgeai-apps-utils 

Build and install all of those backwards, now let’s get:

https://github.com/TexasInstruments/edgeai-tiovx-apps

Fails on drm_fourcc.h… 

Create Symlinks

ln -s /usr/include/libdrm/drm_fourcc.h /usr/include/drm_fourcc.h

ln -s /usr/include/libdrm/drm.h /usr/include/drm.h

ln -s /usr/include/libdrm/drm_mode.h /usr/include/drm_mode.h

Ok now make install and it all does it’s thing

`./bin/Release/edgeai-tiovx-apps-main configs/linux/object_detection.yaml`

Fails because video-+1280_768.h264 does not exist… so gotta get test data now

edgeAI test data is behind TI firewall. Need to get public link TODO. 

TODO - Add notes about Robert's repo (which may be slightly behind this) and on Loading FW to C7s and testing.
