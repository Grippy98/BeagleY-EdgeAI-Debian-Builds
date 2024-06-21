# BeagleY-EdgeAI-Debian-Builds
A repo to store temp builds of debian packages

Raw Notes:

June 20 2024 - Prod Board

beagle@BeagleY:~$ uname -a
Linux BeagleY 6.1.80-ti-arm64-r57 #1bookworm SMP PREEMPT_DYNAMIC Wed Jun 19 00:28:40 UTC 2024 aarch64 GNU/Linux - clean FS

For speed, I connect over Ethernet and boot BeagleY from NVMe PCIe drive. 

NOTE - Currently RPROC FW Broken

**NEW VERSION**

Get this private repo https://github.com/junechul/vision-apps-build - copy to ~/

`docker pull arm64v8/debian:12.5`

`mkdir -p ~/bin`
`PATH="${HOME}/bin:${PATH}"`
`curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo`
`chmod a+rx ~/bin/repo`
`sudo update-alternatives --install /usr/bin/python python /usr/bin/python3 1`

```
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
```

`ARCH=arm64 BASE_IMAGE=debian:12.5 ./init_setup.sh`

chmod +x *.sh

`ARCH=arm64 BASE_IMAGE=debian:12.5 ./init_setup.sh`

`ARCH=arm64 BASE_IMAGE=debian:12.5 ./docker_build.sh`

Docker build on target finished in 5 mins and 35 secs!!!

`ARCH=arm64 BASE_IMAGE=debian:12.5 ./docker_run.sh`

`SOC=j722s GCC_LINUX_ARM_ROOT=/usr CROSS_COMPILE_LINARO= LINUX_SYSROOT_ARM=/ LINUX_FS_PATH=/ TREAT_WARNINGS_AS_ERROR=0 make yocto_build`

`SOC=j722s PKG_DIST=debian12.5 make deb_package`

Total build time up to this point a little over 15 mins. Not bad. 

copy out of the docker container with 
`docker cp <containerId>:/file/path/within/container /host/path/target`

sudo dpkg -i libti-vision-apps.deb etc after you pick it up

Temporarily going to host builds here - 

[Untitled Database](https://www.notion.so/c6631e92480d416fb160d415011dc562?pvs=21)

TODO - Now need to pull in repo examples, test data, etc and run. 

Get edgeai-tiovx-apps

```
cd edgeai-tiovx-apps
export SOC=j722s
mkdir build
cd build
cmake ..
make -j2
```

FAIl- no edgeai_tiovx_img_proc

https://github.com/TexasInstruments/edgeai-tiovx-modules - 2 YEARS OUT OF DATE WTF

```
/opt# cd /opt/edgeai-tiovx-modules
/opt/edgeai-tiovx-modules# mkdir build
/opt/edgeai-tiovx-modules# cd build
/opt/edgeai-tiovx-modules/build# cmake ..
/opt/edgeai-tiovx-modules/build# make -j2
```

https://git.ti.com/cgit/edgeai/edgeai-tiovx-modules 

sudo git checkout remotes/origin/main

export SOC=j722s

https://git.ti.com/cgit/edgeai/edgeai-tiovx-kernels edgeai-tiovx-kernel dependency

edgeai-apps-utils dependency….

https://git.ti.com/cgit/edgeai/edgeai-apps-utils 

Build and install all of those backwards, now let’s get:

https://github.com/TexasInstruments/edgeai-tiovx-apps

Fails on drm_fourcc.h… 

ln -s /usr/include/libdrm/drm_fourcc.h /usr/include/drm_fourcc.h

ln -s /usr/include/libdrm/drm.h /usr/include/drm.h

ln -s /usr/include/libdrm/drm_mode.h /usr/include/drm_mode.h

Ok now make install and it all does it’s thing

`./bin/Release/edgeai-tiovx-apps-main configs/linux/object_detection.yaml`

Fails because video-+1280_768.h264 does not exist… so gotta get test data now

edgeAI test data https://software-dl.ti.com/jacinto7/esd/edgeai-test-data/09_02_00/edgeai-test-data.tar.gz


# OTHER NOTES


**Putting all this in a folder ~/EdgeAI** 

**FW Building:**

**TI Vision Apps:**

🛹 = On BeagleY

🖥️ = X86_64 Ubuntu Machine

💢 = ARM Server/Machine with a lot of RAM

🖥️ “Download and Install PSDK RTOS on x86 PC - Installer only available through mySecure”

Found - https://software-dl.ti.com/jacinto7/esd/processor-sdk-rtos-jacinto7/09_02_00_05/exports/docs/vision_apps/docs/user_guide/ENVIRONMENT_SETUP.html

Found - https://software-dl.ti.com/jacinto7/esd/processor-sdk-rtos-jacinto7/09_02_00_05/exports/docs/vision_apps/docs/user_guide/BUILD_INSTRUCTIONS.html 

 wget https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-R9W8pVWsOt/09.02.00.05/ti-processor-sdk-linux-adas-j722s-evm-09_02_00_05-Linux-x86-Install.bin - 2.21GB

Run through GUI installer… takes a while

cd ti-processor-sdk-linux-adas-j722s-evm-09_02_00_05/ 

chmod [setup.sh](http://setup.sh) to be executable…. 

run setup.sh…

**Fail - “Unsupported host Linux. Only Ubuntu 22.04 LTS is supported”** … on Ubuntu 23.04….

Modify /bin/setup-host-check.sh to stop only checking for Jammy… ridiculous requirement.

**Fail** - E: Package 'tftpd' has no installation candidate

Install manually and remove from bin/setup-package-install.sh 

**Fail** - Script tries to install Python yamllint from Pip not Apt….

Replace with python3-jsonschema, python3-pyelftools, python3-yaml in  bin/setup-package-install.sh  and remove “python3 dependencies” list, just make empty.

**Skip** TFTP, NFS, UBoot setup in script.

🖥️  OOPS INSTALLED PSDK LINUX NOT PSDK RTOS>>>>> FFS

wget https://dr-download.ti.com/software-development/software-development-kit-sdk/MD-1bSfTnVt5d/09.02.00.05/ti-processor-sdk-rtos-j722s-evm-09_02_00_05.tar.gz  - 1.57GB

tar -xvf ti-processor-sdk-rtos…

 cd ti-processor-sdk-rtos-j722s-evm-09_02_00_05/ 

export PSDKL_PATH=~/ti-processor-sdk-linux-adas-j722s-evm-09_02_00_05

export PSDKR_PATH=~/ti-processor-sdk-rtos-j722s-evm-09_02_00_05

export SOC=j722s

**HAVING TO DO THIS - Oh my….**

cp ${PSDKL_PATH}/board-support/prebuilt-images/boot-adas-${SOC}-evm.tar.gz ${PSDKR_PATH}/
cp ${PSDKL_PATH}/filesystem/tisdk-adas-image-${SOC}-evm.tar.xz ${PSDKR_PATH}/

**FAIL** - cp: cannot stat '/home/grippy/ti-processor-sdk-linux-adas-j722s-evm-09_02_00_05/board-support/prebuilt-images/boot-adas-evm.tar.gz': No such file or direc

FIX cp boot-adas-j722s-evm.tar.gz ${PSDKR_PATH}/

cp tisdk-adas-image-j722s-evm.tar.xz ${PSDKR_PATH}/ 

cd ${PSDKR_PATH}

./sdk_builder/scripts/setup_psdk_rtos.sh 

**Ridiculous Note** - “Make sure to call the script from ${PSDKR_PATH} as shown above. DO NOT "cd" into sdk_builder/scripts and call the script”

**FAIL** - PIP Fails to install because not using Venv and managed by external environment

At least this fails nicely… but requires meson, jsonschema, pycryptodomex

Fix - apt install meson python3-jsonschema

Install pycryptodome manuall

`sudo apt-get install build-essential libgmp3-dev python3-dev`

**I GIVE UP -** pip install pycryptodomex --break-system-packages

Edit  sdk_builder/build_flags.mak 

BUILD_EMULATION_MODE=no
BUILD_TARGET_MODE=yes
BUILD_LINUX_MPU=yes
PROFILE=release

cd sdk_builder
make vision_apps -jN
make tidl_rt -jN

Get -  https://git.ti.com/cgit/processor-sdk/vision_apps/ 

“warning: remote HEAD refers to nonexistent ref, unable to checkout”

**Fix** - git checkout remotes/origin/main

OK SCREW IT LETS DO SOMETHING CRAZY—

tar -xvf tisdk-adas-image-j722s-evm.tar.x 

sudo cp -r -n bin/ /bin/

sudo cp -r -n etc/ /etc/

sudo cp -r -n lib/ /lib/

sudo cp -r -n media/ /media/

sudo cp -r -n run/ /run/

sudo cp -r -n srv/ /srv/

sudo cp -r -n usr/ /usr/

Skipping /boot for now… I don’t trust it 

sudo cp -r -n dev/ /dev/

No Proc…

sudo cp -r -n sbin/ /sbin/

Sys fails with permissions… expected

sudo cp -r -n var/ /var/

Aaaand expectedly this killed my filesystem and I lost root so had to start from scratch 
