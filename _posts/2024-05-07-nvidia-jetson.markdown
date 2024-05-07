---
layout: post
title:  "How to install SLE-15-SP6 on NVIDIA Jetson platform (Jetson AGX Orin/IGX Orin)"
date:   2024-05-07
categories: nvidia
---
This covers the installation of updated Kernel, out-of-tree nvidia kernel modules package, how to get GNOME desktop running and installation/run of glmark2 benchmark. Also it describes how to get some CUDA and TensorRT samples running.

### SP6

Download [SLE-15-SP6 (Arm) installation image][image]. This you can put on a regular USB stick or on an SD card using `dd` command. Go into BIOS and change `SOC Display Hand-Off Mode` settings, i.e. `Device Manager -> NVIDIA Configuration -> Boot Configuration -> SOC Display Hand-Off Mode`, to `Never`.

Boot from the USB stick/SD card, that you wrote above and install SP6. You need to install via serial console, since the monitor won’t get any signal without the out-of-tree nvidia kernel modules, which are installed later in the process.

Make sure you select the following modules during installation:

* Basesystem
* Containers
* Desktop Applications
* Development Tools
* Python 3
* Server Applications

Select `SLES with GNOME` for installation.

### Kernel + KMP drivers

Continue installation with serial console.

Now update kernel and install our KMP (kernel module package) for all nvidia kernel modules.

We plan to make the KMP available as a driver kit via the SolidDriver Program. I’ve opened  SUSE bugzilla ticket [#1222604][boo1222604] for this. For now please install an updated kernel and the KMP after checking the [build status][buildstatus] (rebuilding can take a few hours!) from our open buildservice:

{% highlight shell %}
$ sudo zypper ar https://download.opensuse.org/repositories/home:/sndirsch:/sidecar/SLE_15_SP6/ home:sndirsch:sidecar
$ sudo zypper ref
# flavor either default or 64kb (check with uname -r command)
$ sudo zypper in -f -r home:sndirsch:sidecar kernel-<flavor>  nvidia-open-driver-G06-signed-sidecar-kmp-<flavor>
{% endhighlight %}

Reboot with the updated kernel.

{% highlight shell %}
$ sudo reboot
{% endhighlight %}

In Mokmanager (`Perform MOK management`) select `Continue boot`. Although Secureboot is enabled by default in BIOS it seems it hasn’t been implemented yet (BIOS from 04/04/2024). Select first entry `SLES 15-SP6` for booting.

### Userspace/Desktop

Unfortunately installing the userspace is a non-trivial task.

#### Internal SUSE package

@sndirsch worked on a minimal userspace package internally available for SUSE employees. Check this out [here][userspace-package].

#### Manual installation for regular customers

Download Jetpack 6 [Driver Package (BSP)][driver-pkg-bsp] from this [location]
[jetpack6-website]. Extract `jetson_linux_r36.3.0_aarch64.tbz2`.

{% highlight shell %}
$ tar xf jetson_linux_r36.3.0_aarch64.tbz2
{% endhighlight %}

Then you need to convert debian packages from this content into tarballs.

{% highlight shell %}
$ pushd Linux_for_Tegra
$ sudo ln -snf bzip2 /usr/bin/lbzip2
$ ./nv_tools/scripts/nv_repackager.sh -o ./nv_tegra/l4t_tar_packages --convert-all
$ popd
{% endhighlight %}

Details about the script mentioned above are described in chapter `4. Installing the Jet Pack 6 Userspace` of the PDF [Jetson-Linux - 3rd Party Distro Porting Guide - 20240201.pdf][porting-guide-pdf]. Unfortunately this PDF is not officially available.

From the generated tarballs you only need these:

{% highlight shell %}
nvidia-l4t-3d-core_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-camera_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-core_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-cuda_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-firmware_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-gbm_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-multimedia-utils_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-multimedia_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-nvfancontrol_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-nvml_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-nvpmodel_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-nvsci_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-pva_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-tools_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-vulkan-sc-sdk_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-wayland_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-x11_36.3.0-20240404104251_arm64.tbz2
nvidia-l4t-nvml_36.3.0-20240404104251_arm64.tbz2
{% endhighlight %}

And from this tarball `nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2` you only need these files:

{% highlight shell %}
etc/asound.conf.tegra-ape
etc/asound.conf.tegra-hda-jetson-agx
etc/asound.conf.tegra-hda-jetson-xnx
etc/nvidia-container-runtime/host-files-for-container.d/devices.csv
etc/nvidia-container-runtime/host-files-for-container.d/drivers.csv
etc/nvsciipc.cfg
etc/sysctl.d/60-nvsciipc.conf
etc/systemd/nv_nvsciipc_init.sh
etc/systemd/nvpower.sh
etc/systemd/nv.sh
etc/systemd/system.conf.d/watchdog.conf
etc/systemd/system/multi-user.target.wants/nv_nvsciipc_init.service
etc/systemd/system/multi-user.target.wants/nvpower.service
etc/systemd/system/multi-user.target.wants/nv.service
etc/systemd/system/nv_nvsciipc_init.service
etc/systemd/system/nvpower.service
etc/systemd/system/nv.service
etc/udev/rules.d/99-tegra-devices.rules
usr/share/alsa/cards/tegra-ape.conf
usr/share/alsa/cards/tegra-hda.conf
usr/share/alsa/init/postinit/00-tegra.conf
usr/share/alsa/init/postinit/01-tegra-rt565x.conf
usr/share/alsa/init/postinit/02-tegra-rt5640.conf
{% endhighlight %}

So first let’s repackage `nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2`:

{% highlight shell %}
$ pushd Linux_for_Tegra/nv_tegra/l4t_tar_packages/
$ cat > nvidia-l4t-init.txt << EOF
etc/asound.conf.tegra-ape
etc/asound.conf.tegra-hda-jetson-agx
etc/asound.conf.tegra-hda-jetson-xnx
etc/nvidia-container-runtime/host-files-for-container.d/devices.csv
etc/nvidia-container-runtime/host-files-for-container.d/drivers.csv
etc/nvsciipc.cfg
etc/sysctl.d/60-nvsciipc.conf
etc/systemd/nv_nvsciipc_init.sh
etc/systemd/nvpower.sh
etc/systemd/nv.sh
etc/systemd/system.conf.d/watchdog.conf
etc/systemd/system/multi-user.target.wants/nv_nvsciipc_init.service
etc/systemd/system/multi-user.target.wants/nvpower.service
etc/systemd/system/multi-user.target.wants/nv.service
etc/systemd/system/nv_nvsciipc_init.service
etc/systemd/system/nvpower.service
etc/systemd/system/nv.service
etc/udev/rules.d/99-tegra-devices.rules
usr/share/alsa/cards/tegra-ape.conf
usr/share/alsa/cards/tegra-hda.conf
usr/share/alsa/init/postinit/00-tegra.conf
usr/share/alsa/init/postinit/01-tegra-rt565x.conf
usr/share/alsa/init/postinit/02-tegra-rt5640.conf
EOF
$ tar xf nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2
$ rm nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2
$ tar cjf nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2 $(cat nvidia-l4t-init.txt)
$ popd
{% endhighlight %}

Then extract the generated tarballs to your system.

{% highlight shell %}
$ pushd Linux_for_Tegra/nv_tegra/l4t_tar_packages
$ for i in \
nvidia-l4t-core_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-3d-core_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-cuda_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-firmware_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-gbm_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-multimedia-utils_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-multimedia_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-nvfancontrol_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-nvpmodel_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-tools_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-x11_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-nvsci_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-pva_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-wayland_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-camera_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-vulkan-sc-sdk_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-nvml_36.3.0-20240404104251_arm64.tbz2 \
nvidia-l4t-init_36.3.0-20240404104251_arm64.tbz2; do
  sudo tar xjf $i -C /
done
$ popd
{% endhighlight %}

Then you still need to move

{% highlight shell %}
/usr/lib/xorg/modules/drivers/nvidia_drv.so
/usr/lib/xorg/modules/extensions/libglxserver_nvidia.so
{% endhighlight %}

to

{% highlight shell %}
/usr/lib64/xorg/modules/drivers/nvidia_drv.so
/usr/lib64/xorg/modules/extensions/libglxserver_nvidia.so
{% endhighlight %}

and add `/usr/lib/aarch64-linux-gnu` to `/etc/ld.so.conf.d/nvidia-tegra.conf`.


{% highlight shell %}
$ sudo mv /usr/lib/xorg/modules/drivers/nvidia_drv.so \
          /usr/lib64/xorg/modules/drivers/
$ sudo mv /usr/lib/xorg/modules/extensions/libglxserver_nvidia.so \
          /usr/lib64/xorg/modules/extensions/
$ sudo rm -rf /usr/lib/xorg
$ sudo echo /usr/lib/aarch64-linux-gnu >> /etc/ld.so.conf.d/nvidia-tegra.conf
{% endhighlight %}

Run ldconfig 

{% highlight shell %}
$ sudo ldconfig
{% endhighlight %}

#### Also needed when using the SUSE internal package

A regular user needs to be added to the group `video` to be able to log in to the GNOME desktop as regular user. This can be achieved by using YaST, usermod or editing `/etc/group` manually.

#### Reboot the machine

{% highlight shell %}
$ sudo reboot
{% endhighlight %}

### Basic testing

First basic testing will be running `nvidia-smi`. 

{% highlight shell %}
$ sudo nvidia-smi
{% endhighlight %}

Graphical desktop (GNOME) should work as well. Unfortunately Linux console is not available. Use either a serial console or a ssh connection if you don’t want to use the graphical desktop or need remote access to the system.

### glmark2

Install phoronix-test-suite

{% highlight shell %}
$ sudo zypper ar https://cdn.opensuse.org/distribution/leap/15.6/repo/oss/ repo-oss
$ sudo zypper ref
$ sudo zypper in phoronix-test-suite
{% endhighlight %}

Run phoronix-test-suite

{% highlight shell %}
$ sudo zypper in gcc gcc-c++
$ phoronix-test-suite benchmark glmark2
{% endhighlight %}

### CUDA/Tensorflow

#### Containers

NVIDIA provides containers available for Jetson that include SDKs such as CUDA. More details [here][container]. These containers are Ubuntu based, but can be used from SLE as well. You need to install the NVIDIA container runtime for this. Detailed information [here][container-toolkit].


##### 1. Install podman and nvidia-container-runtime

{% highlight shell %}
$ sudo zypper install podman
$ sudo zypper ar https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo
$ sudo zypper modifyrepo --enable nvidia-container-toolkit-experimental
$ sudo zypper --gpg-auto-import-keys install -y nvidia-container-toolkit
$ sudo nvidia-ctk cdi generate --mode=csv --output=/var/run/cdi/nvidia.yaml
$ sudo nvidia-ctk cdi list
{% endhighlight %}

##### 2. Download the CUDA samples

{% highlight shell %}
$ sudo zypper install git
$ cd
$ git clone https://github.com/NVIDIA/cuda-samples.git
$ cd cuda-samples
$ git checkout v12.4
{% endhighlight %}

##### 3. Start X

{% highlight shell %}
$ sudo rcxdm stop
$ sudo Xorg -retro &> /tmp/log &
$ export DISPLAY=:0
$ xterm &
{% endhighlight %}

Monitor should now show a Moiree pattern with an unframed xterm on it. Otherwise check /tmp/log.

##### 4. Download and run the JetPack6 container

{% highlight shell %}
$ sudo podman run --rm -it -e DISPLAY --net=host --device nvidia.com/gpu=all --group-add keep-groups --security-opt label=disable -v $HOME/cuda-samples:/cuda-samples nvcr.io/nvidia/l4t-jetpack:r36.2.0 /bin/bash
{% endhighlight %}

#### CUDA

##### 5. Build and run the samples in the container

{% highlight shell %}
$ cd /cuda-samples
$ make -j$(nproc)
$ pushd ./Samples/5_Domain_Specific/nbody
$ make
$ popd
$ ./bin/aarch64/linux/release/deviceQuery
$ ./bin/aarch64/linux/release/nbody
{% endhighlight %}

#### Tensorrt
##### 6. Build and run Tensorrt in the container

This is both with the GPU and DLA (deep-learning accelerator).

{% highlight shell %}
$ cd /usr/src/tensorrt/samples/
$ make -j$(nproc)
$ cd ..
$ ./bin/sample_algorithm_selector
$ ./bin/sample_onnx_mnist
$ ./bin/sample_onnx_mnist --useDLACore=0
$ ./bin/sample_onnx_mnist --useDLACore=1
{% endhighlight %}

### Misc

#### Performance

You can improve the performance by giving the clock a boost. For best performance you can run `jetson_clocks` to set the device to max clock settings

{% highlight shell %}
$ sudo jetson_clocks --show
$ sudo jetson_clocks
$ sudo jetson_clocks --show
{% endhighlight %}

The 1st and 3rd command just prints the clock settings.

### Known issues

In some cases the machine locks up. In that case you see an error message like this

{% highlight shell %}
[...]
[ 3511.532244][    C0] watchdog: BUG: soft lockup - CPU#0 stuck for 26s! [X:2935]
[ 3564.031881][    C0] BUG: workqueue lockup - pool cpus=0 node=0 flags=0x0 nice=0 stuck for 76s!
[...]
{% endhighlight %}

Unfortunately in that case you need to hard reboot the machine. This issue is tracked by NVIDIA and will hopefully be fixed soon.

[image]: https://www.suse.com/download/sles/
[boo1222604]: https://bugzilla.suse.com/show_bug.cgi?id=1222604
[buildstatus]: https://build.opensuse.org/project/monitor/home:sndirsch:sidecar
[userspace-package]: https://build.suse.de/project/show/home:sndirsch:sidecar
[jetpack6-website]: https://developer.nvidia.com/embedded/jetson-linux-r363
[driver-pkg-bsp]: https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v3.0/release/jetson_linux_r36.3.0_aarch64.tbz2
[porting-guide-pdf]: https://drive.google.com/file/d/1_P7-hRNmCIVxK-rChw6kiIrSLYn-zBOt/view?usp=drive_link
[container]: https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-jetpack
[container-toolkit]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
