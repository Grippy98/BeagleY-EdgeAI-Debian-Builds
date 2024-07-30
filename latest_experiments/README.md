

## We Start with EdgeAI App Stack Repo. - Expects x86 build Env - Used Ubuntu 22.04 VM

Starting Repo: https://github.com/TexasInstruments/edgeai-app-stack

---

TI.com docs
https://software-dl.ti.com/jacinto7/esd/processor-sdk-linux-am67a/09_02_00/exports/edgeai-docs/common/sdk_components.html#edge-ai-application-stack 

9.1.1. edgeai-gst-apps
---
These are plug-and-play deep learning applications which support running open source runtime frameworks such as TFLite, ONNX and NeoAI-DLR with a variety of input and output configurations.

The latest source code with fixes can be pulled from [TI Edge AI GStreamer apps](https://github.com/TexasInstruments/edgeai-gst-apps)

9.1.2. edgeai-dl-inferer
---
This repo provides interface to TI OSRT library whose APIs can be used standalone or with an application like edgeai-gst-apps. It also provides the source of NV12 post processing library and utils which are used with some custom GStreamer plugins.

The latest source code with fixes can be pulled from [TI Edge AI DL Inferer](https://git.ti.com/cgit/edgeai/edgeai-dl-inferer)

9.1.3. edgeai-gst-plugins
---
This repo provides the source of custom GStreamer plugins which helps offload tasks to the hardware accelerators with the help of edgeai-tiovx-modules.

Source code and documentation [TI Edge AI GStreamer plugins](https://github.com/TexasInstruments/edgeai-gst-plugins)
---
9.1.4. edgeai-app-utils
---
This repo provides utility APIs for NV12 drawing and font rendering, reporting MPU and DDR performance, ARM Neon optimized kernels for color conversion, pre-processing and scaling.

Source code and documentation: [TI Edge AI Apps utils](https://git.ti.com/cgit/edgeai/edgeai-apps-utils)

9.1.5. edgeai-tiovx-modules
---
This repo provides OpenVx modules which help access underlying hardware accelerators in the SoC and serves as a bridge between GStreamer custom elements and underlying OpenVx custom kernels.

Source code and documentation: [TI Edge AI TIOVX modules](https://git.ti.com/cgit/edgeai/edgeai-tiovx-modules)

9.1.6. edgeai-tiovx-kernels
---
This repo provides OpenVx kernels which help accelerate color-convert, DL-pre-processing and DL-post-processing using ARMv8 NEON accelerator

Source code and documentation: [TI Edge AI TIOVX kernels](https://git.ti.com/cgit/edgeai/edgeai-tiovx-kernels)
9.1.7. edgeai-tiovx-apps
---
This repo provides a layer on top of OpenVX to easily create a OpenVX graph and connect them to v4l2 blocks to realize various complex usecases

Source code and documentation: [TI Edge AI TIOVX Apps](https://github.com/TexasInstruments/edgeai-tiovx-apps)



# Build notes:

#### On X86 Ubuntu 22.04 Desktop:

```
git clone https://github.com/TexasInstruments/edgeai-app-stack.git
cd edgeai-app-stack
git submodule init
git submodule update 
export SOC=j722s
chmod +x setup.sh
./setup.sh $SOC
```

Part of the setup is it downloads arm-gnu-toolchain-11.3.rel1 aarch64-none-linux-gnu for the compiler which is a bit old
It also downloads tisdk-adas-image-j722s-evm


Instructions Now say:
```
4. Connect the SD card that is flashed with EdgeAI WIC Image. By default TargetFS and Install Path is set as $(PWD)/targetfs
5. Modify Makefile to set TARGETFS, SOC, INSTALL_PATH etc..


Then make the fragments 
```

Now let's experiment a bit - WITHOUT making any changes:

```apt install make cmake meson ninja-build pkg-config```


EdgeAI-GST-APPS has broken separator

```
-- Build files have been written to: /home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build
make[1]: Entering directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make[2]: Entering directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make[3]: Entering directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
utils/CMakeFiles/edgeai_utils.dir/flags.make:10: *** missing separator.  Stop.
make[3]: Leaving directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make[2]: *** [CMakeFiles/Makefile2:151: utils/CMakeFiles/edgeai_utils.dir/all] Error 2
make[2]: *** Waiting for unfinished jobs....
make[3]: Entering directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
common/CMakeFiles/edgeai_common.dir/flags.make:10: *** missing separator.  Stop.
make[3]: Leaving directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make[2]: *** [CMakeFiles/Makefile2:177: common/CMakeFiles/edgeai_common.dir/all] Error 2
make[2]: Leaving directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make[1]: *** [Makefile:136: all] Error 2
make[1]: Leaving directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-gst-apps/apps_cpp/build'
make: *** [Makefile:183: gst_apps] Error 2
```

### EdgeAI GST APPS doesn't build right because of Cmake errors...

So I edit the makefile to remove the following:

``` 
########################### EDGEAI-TIOVX-MODULES ###############################

gst_apps: dl_inferer_install gst_plugins_install apps_utils_install
	@echo "Building Gst Apps"
	cd $(GST_APPS_PATH)/apps_cpp; \
	mkdir build; \
	cd build; \
	cmake -DCMAKE_TOOLCHAIN_FILE=../cmake/cross_compile_aarch64.cmake ..; \
	$(MAKE)

gst_apps_install: gst_apps
	@echo "Install Gst Apps"
	cd $(GST_APPS_PATH); \
	mkdir -p $(INSTALL_PATH)/opt/edgeai-gst-apps-pc; \
	cp -r $(GST_APPS_PATH)/* $(INSTALL_PATH)/opt/edgeai-gst-apps-pc/

gst_apps_clean:
	@echo "Clean Gst Apps"
	cd $(GST_APPS_PATH)/apps_cpp; \
	rm -rf build bin lib
```


```
Install the project...
-- Install configuration: ""
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/lib/libedgeai-tiovx-apps.so.0.1.0
-- Up-to-date: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/lib/libedgeai-tiovx-apps.so
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/kms_display_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/linux_aewb_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_aewb_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_capture_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_color_convert_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_delay_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_display_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_dl_color_convert_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_dl_post_proc_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_dl_pre_proc_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_dof_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_dof_viz_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_fakesink_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_ldc_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_mosaic_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_multi_scaler_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_obj_array_split_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_pyramid_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_sde_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_sde_viz_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_sensor_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_tee_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_tidl_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/tiovx_viss_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/v4l2_capture_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/v4l2_decode_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/include/edgeai-tiovx-apps/v4l2_encode_module.h
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/bin/edgeai-tiovx-apps-test
-- Installing: /home/grippy/Desktop/edgyAI/edgeai-app-stack/targetfs/usr/bin/edgeai-tiovx-apps-main
make[1]: Leaving directory '/home/grippy/Desktop/edgyAI/edgeai-app-stack/edgeai-tiovx-apps/build'
```

# On target BeagleY

Image - BeagleY-AI Debian 12.6 2024-07-11 XFCE 
https://files.beagle.cc/file/beagleboard-public-2021/images/beagley-ai-debian-12.6-xfce-arm64-2024-07-11-12gb.img.xz 

    Kernel: 6.1.83-ti-arm64-r63
    U-Boot: v2023.04-ti-09.02.00.009-BeagleY-AI-Production


### To Download EdgeAI-GST-APPS Models 
EDGEAI SDK VERSION NEEDS TO BE SET 
```
export EDGEAI_SDK_VERSION=09_02_00 
./download_models.sh --recommended
```
https://github.com/TexasInstruments/edgeai-gst-apps/blob/main/download_models.sh 





On BeagleY I created a folder called edgyAI and I just SFTPd ` ```targetfs/opt``` ```targetfs/lib``` from the build X86 desktop to the Beagle. 

```
sudo cp -r -n lib/* /lib
sudo cp -r -n opt/* /opt
```
### Get the firmware files and TI Vision Apps

```
git clone https://github.com/Grippy98/BeagleY-EdgeAI-Debian-Builds
git checkout dev
cd release
sudo dpkg -i libti-vision-apps-j722s_9.2.0-debian12.5.deb
cd ..
```

### Now load R5 & C7x firmware binaries
 
```
cd firmware
sudo cp -v ./vx_app_rtos_linux_c7x_1.out /usr/lib/firmware/j722s-c71_0-fw
sudo cp -v ./vx_app_rtos_linux_c7x_2.out /usr/lib/firmware/j722s-c71_1-fw
sudo cp -v ./vx_app_rtos_linux_mcu2_0.out /usr/lib/firmware/j722s-main-r5f0_0-fw
```

### Let's update our overlays (RCN-ee maintains a patched version of the DTSI for 4GB DDR)

```cd /opt/source/dtb-6.1-Beagle/ ; git pull ; make clean ; make -j4 ; sudo make install_arm64```

add  "fdtoverlays /overlays/k3-j722s-edgeai-apps.dtbo" to /boot/firmware/extlinux/extlinux.conf default boot entry

#### Reboot...


```sudo ./vx_app_arm_remote_log.out```  in release to test, and/or ```dmesg | grep "firmware"```

```
cd /opt/vision_apps/
sudo ource ./vision_apps_init.sh
```

### Now we get to edgeai-gst-apps

```
cd /opt
git clone https://github.com/TexasInstruments/edgeai-gst-apps
export SOC=j722s
export EDGEAI_SDK_VERSION=09_02_00
```


### If we want to get system Python3.12
```
wget -qO- https://pascalroeleven.nl/deb-pascalroeleven.gpg | sudo tee /etc/apt/keyrings/deb-pascalroeleven.gpg

cat <<EOF | sudo tee /etc/apt/sources.list.d/pascalroeleven.sources
Types: deb
URIs: http://deb.pascalroeleven.nl/python3.12
Suites: bookworm-backports
Components: main
Signed-By: /etc/apt/keyrings/deb-pascalroeleven.gpg
EOF

sudo apt update
sudo apt install python3.12 python3.12-venv python3.12-devls
```

Now need to update symilnks to have /usr/bin/python to point to /usr/bin/python3.12. This might work with Venv too but I haven't gotten that far yet. 

### If you want to install EdgeAI TIDL Tools via PIP: 

```pip install git+https://github.com/TexasInstruments/edgeai-tidl-tools.git#subdirectory=scripts```

### Where I'm stuck:

```
git clone https://git.ti.com/cgit/edgeai/edgeai-dl-inferer
sudo apt-get update && sudo apt-get install build-essential
sudo apt-get --reinstall install libc6 libc6-dev
```

Aaaan I get stuck at trying to build edgeai-dl-inferer...
