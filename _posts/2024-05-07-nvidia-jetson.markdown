---
layout: post
title:  "How to install SLE-15-SP6 on NVIDIA Jetson platform (Jetson AGX Orin/IGX Orin)"
date:   2024-05-07
categories: nvidia
---
This covers the installation of updated Kernel, out-of-tree nvidia kernel modules package, how to get GNOME desktop running and installation/run of glmark2 benchmark. Also it describes how to get some CUDA and TensorRT samples running.

### SP6

Download [SLE-15-SP6 (Arm) installation image][image]. This you can put on a regular USB stick or on an SD card using `dd` command. 

Boot from the USB stick/SD card, that you wrote above and install SP6. You can install via serial console or connect a monitor to the display port.

#### When using a connected monitor for installation

This needs for the installation a special setting in the Firmware of the machine.

{% highlight shell %}
--> UEFI Firmware Settings
 --> Device Manager
  --> NVIDIA Configuration
   --> Boot Configuration
    --> SOC Display Hand-Off Mode <Always>
{% endhighlight %}

This setting for `SOC Display Hand-Off Mode` will change automatically to `Never` later with the installation of the graphics driver.

Once grub starts you need to edit the grub entry `Installation`. Press `e` for doing this and add `console=tty0` to the `linux [...]` line.

{% highlight shell %}
[...]
linux /boot/aarch64/linux splash=silent console=tty0
[...]
{% endhighlight %}

Then press `F10` to continue to boot.

#### Installation

Unfortunately the product registration fails with

{% highlight shell %}
[...]
Error code: Curl error 60
Error message: SSL certificate problem: certificate is not yet valid.
{% endhighlight %}

The reason for this is that the machine is missing a battery-backed RTC (Real Time Clock) and therefore doesn't have the correct time set during installation.

When installing with a connected monitor you can workaround this issue. For doing this you can start in that `Registration` dialogue an xterm by pressing `Ctrl-Alt-Shift-x`. In this xterm run `date` to set the current date. It looks like this:

{% highlight shell %}
date -s '2025-01-23 21:00:00'
{% endhighlight %}

Then try again to register the product. When installing on a serial console you can register the product later after installation by running `yast2 registration`.

Make sure you select the following modules during installation:

* Basesystem (enough for just installing the kernel driver)
* Containers (needed for podman for CUDA libraries)
* Desktop Applications (needed for running a desktop)
* Development Tools (needed for git for CUDA samples)

Select `SLES with GNOME` for installation.

### Time configuration and Product registration

Afer installation first configure correct time using ntp.

{% highlight shell %}
yast2 ntp-client
{% endhighlight %}

Then on a serial consle finally register your product by running

{% highlight shell %}
yast2 registration
{% endhighlight %}

### Kernel + KMP drivers

Now Update kernel and install our KMP (kernel module package) for all nvidia kernel modules.

We plan to make the KMP available as a driver kit via the SolidDriver Program. For now please install an updated kernel and the KMP after checking the [build status][buildstatus] (type 'jetson' in Search... field; rebuilding can take a few hours!) from our open buildservice:

{% highlight shell %}
sudo zypper ar https://download.opensuse.org/repositories/X11:/XOrg/SLE_15_SP6/  jetson-kmp
sudo zypper ref
# flavor either default or 64kb (check with `uname -r` command)
sudo zypper up kernel-<flavor>
sudo zypper in -r jetson-kmp nvidia-jetson-36_4-kmp-<flavor>
{% endhighlight %}


### Userspace/Desktop

Unfortunately installing the userspace is a non-trivial task.

#### Installation

Download [Driver Package (BSP)][driver-pkg-bsp] from this [location][jetpack6-website]. Extract `Jetson_Linux_R36.4.0_aarch64.tbz2`.

{% highlight shell %}
tar xf Jetson_Linux_R36.4.0_aarch64.tbz2
{% endhighlight %}

Then you need to convert debian packages from this content into tarballs.

{% highlight shell %}
pushd Linux_for_Tegra
sed -i -e 's/lbzip2/bzip2/g' -e 's/-I zstd //g' nv_tools/scripts/nv_repackager.sh
./nv_tools/scripts/nv_repackager.sh -o ./nv_tegra/l4t_tar_packages --convert-all
popd
{% endhighlight %}

From the generated tarballs you only need these:

{% highlight shell %}
nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-camera_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-core_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-cuda_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-gbm_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-multimedia-utils_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-multimedia_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-nvfancontrol_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-nvml_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-nvpmodel_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-nvsci_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-pva_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-tools_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-vulkan-sc-sdk_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-wayland_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2
nvidia-l4t-nvml_36.4.0-20240912212859_arm64.tbz2
{% endhighlight %}

And from this tarball `nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2` you only need these files:

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

So first let’s repackage `nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2`:

{% highlight shell %}
pushd Linux_for_Tegra/nv_tegra/l4t_tar_packages/
cat > nvidia-l4t-init.txt << EOF
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
tar xf nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2
rm nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2
tar cjf nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2 $(cat nvidia-l4t-init.txt)
popd
{% endhighlight %}

On IGX Orin platform with dedicated graphics card (dGPU systems) you need to
get rid of some files due to conflicts with dGPU userspace drivers.

{% highlight shell %}
# repackage nvidia-l4t-x11_ package
tar tf nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2 | grep -v /usr/bin/nvidia-xconfig \
  > nvidia-l4t-x11_36.4.0-20240912212859.txt
tar xf  nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2
rm      nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2
tar cjf nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2 $(cat nvidia-l4t-x11_36.4.0-20240912212859.txt)

# repackage nvidia-l4t-3d-core_ package
tar tf nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2 | \
  grep -v \
       -e /etc/vulkan/icd.d/nvidia_icd.json \
       -e /usr/lib/xorg/modules/drivers/nvidia_drv.so \
       -e /usr/lib/xorg/modules/extensions/libglxserver_nvidia.so \
       -e /usr/share/glvnd/egl_vendor.d/10_nvidia.json \
       > nvidia-l4t-3d-core_36.4.0-20240912212859.txt
tar xf  nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2
rm      nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2
tar cjf nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2 $(cat nvidia-l4t-3d-core_36.4.0-20240912212859.txt)
{% endhighlight %}

Then extract the generated tarballs to your system.

{% highlight shell %}
pushd Linux_for_Tegra/nv_tegra/l4t_tar_packages
for i in \
nvidia-l4t-core_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-3d-core_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-cuda_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-gbm_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-multimedia-utils_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-multimedia_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-nvfancontrol_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-nvpmodel_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-tools_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-x11_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-nvsci_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-pva_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-wayland_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-camera_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-vulkan-sc-sdk_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-nvml_36.4.0-20240912212859_arm64.tbz2 \
nvidia-l4t-init_36.4.0-20240912212859_arm64.tbz2; do
  sudo tar xjf $i -C /
done
popd
{% endhighlight %}

On systems without dedicated graphics (internal GPU systems) card you still
need to move

{% highlight shell %}
/usr/lib/xorg/modules/drivers/nvidia_drv.so
/usr/lib/xorg/modules/extensions/libglxserver_nvidia.so
{% endhighlight %}

to

{% highlight shell %}
/usr/lib64/xorg/modules/drivers/nvidia_drv.so
/usr/lib64/xorg/modules/extensions/libglxserver_nvidia.so
{% endhighlight %}

So let's do this.

{% highlight shell %}
sudo mv /usr/lib/xorg/modules/drivers/nvidia_drv.so \
          /usr/lib64/xorg/modules/drivers/
sudo mv /usr/lib/xorg/modules/extensions/libglxserver_nvidia.so \
          /usr/lib64/xorg/modules/extensions/
sudo rm -rf /usr/lib/xorg
{% endhighlight %}

Then add `/usr/lib/aarch64-linux-gnu` and
`/usr/lib/aarch64-linux-gnu/tegra-egl` to
`/etc/ld.so.conf.d/nvidia-tegra.conf`.

{% highlight shell %}
echo /usr/lib/aarch64-linux-gnu | sudo tee -a /etc/ld.so.conf.d/nvidia-tegra.conf
echo /usr/lib/aarch64-linux-gnu/tegra-egl | sudo tee -a /etc/ld.so.conf.d/nvidia-tegra.conf
{% endhighlight %}

Run ldconfig 

{% highlight shell %}
sudo ldconfig
{% endhighlight %}

#### Video group for regular users

A regular user needs to be added to the group `video` to be able to log in to the GNOME desktop as regular user. This can be achieved by using YaST, usermod or editing `/etc/group` manually.

#### Reboot the machine with the previously updated kernel

{% highlight shell %}
sudo reboot
{% endhighlight %}

In Mokmanager (`Perform MOK management`) select `Continue boot`. Although Secureboot is enabled by default in BIOS it seems it hasn’t been implemented yet (BIOS from 04/04/2024). Select first entry `SLES 15-SP6` for booting.

### Basic testing

First basic testing will be running `nvidia-smi`. 

{% highlight shell %}
sudo nvidia-smi
{% endhighlight %}

Graphical desktop (GNOME) should work as well. Unfortunately Linux console is not available. Use either a serial console or a ssh connection if you don’t want to use the graphical desktop or need remote access to the system.

### glmark2

Install phoronix-test-suite

{% highlight shell %}
sudo zypper ar https://cdn.opensuse.org/distribution/leap/15.6/repo/oss/ repo-oss
sudo zypper ref
sudo zypper in phoronix-test-suite
{% endhighlight %}

Run phoronix-test-suite

{% highlight shell %}
sudo zypper in gcc gcc-c++
phoronix-test-suite benchmark glmark2
{% endhighlight %}

### CUDA/Tensorflow

#### Containers

NVIDIA provides containers available for Jetson that include SDKs such as CUDA. More details [here][container]. These containers are Ubuntu based, but can be used from SLE as well. You need to install the NVIDIA container runtime for this. Detailed information [here][container-toolkit].


##### 1. Install podman and nvidia-container-runtime

{% highlight shell %}
sudo zypper install podman
sudo zypper ar https://nvidia.github.io/libnvidia-container/stable/rpm/nvidia-container-toolkit.repo
sudo zypper modifyrepo --enable nvidia-container-toolkit-experimental
sudo zypper --gpg-auto-import-keys install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --mode=csv --output=/var/run/cdi/nvidia.yaml
sudo nvidia-ctk cdi list
{% endhighlight %}

##### 2. Download the CUDA samples

{% highlight shell %}
sudo zypper install git
cd
git clone https://github.com/NVIDIA/cuda-samples.git
cd cuda-samples
git checkout v12.4
{% endhighlight %}

##### 3. Start X

{% highlight shell %}
sudo rcxdm stop
sudo Xorg -retro &> /tmp/log &
export DISPLAY=:0
xterm &
{% endhighlight %}

Monitor should now show a Moiree pattern with an unframed xterm on it. Otherwise check /tmp/log.

##### 4. Download and run the JetPack6 container

{% highlight shell %}
sudo podman run --rm -it -e DISPLAY --net=host --device nvidia.com/gpu=all --group-add keep-groups --security-opt label=disable -v $HOME/cuda-samples:/cuda-samples nvcr.io/nvidia/l4t-jetpack:r36.2.0 /bin/bash
{% endhighlight %}

#### CUDA

##### 5. Build and run the samples in the container

{% highlight shell %}
cd /cuda-samples
make -j$(nproc)
pushd ./Samples/5_Domain_Specific/nbody
make
popd
./bin/aarch64/linux/release/deviceQuery
./bin/aarch64/linux/release/nbody
{% endhighlight %}

#### Tensorrt
##### 6. Build and run Tensorrt in the container

This is both with the GPU and DLA (deep-learning accelerator).

{% highlight shell %}
cd /usr/src/tensorrt/samples/
make -j$(nproc)
cd ..
./bin/sample_algorithm_selector
./bin/sample_onnx_mnist
./bin/sample_onnx_mnist --useDLACore=0
./bin/sample_onnx_mnist --useDLACore=1
{% endhighlight %}

### Misc

#### Performance

You can improve the performance by giving the clock a boost. For best performance you can run `jetson_clocks` to set the device to max clock settings

{% highlight shell %}
sudo jetson_clocks --show
sudo jetson_clocks
sudo jetson_clocks --show
{% endhighlight %}

The 1st and 3rd command just prints the clock settings.


#### MaxN Power

For maximum performance you also need to set MaxN Power. This can be done by running

{% highlight shell %}
sudo nvpmodel -m 0
{% endhighlight %}

Afterwards you need to reboot the system though.

{% highlight shell %}
sudo reboot
{% endhighlight %}

In order to check for the current value run

{% highlight shell %}
sudo nvpmodel -q
{% endhighlight %}

[image]: https://www.suse.com/download/sles/
[buildstatus]: https://build.opensuse.org/project/monitor/X11:XOrg
[jetpack6-website]: https://developer.nvidia.com/embedded/jetson-linux-r3640
[driver-pkg-bsp]: https://developer.nvidia.com/downloads/embedded/l4t/r36_release_v4.0/release/Jetson_Linux_R36.4.0_aarch64.tbz2
[container]: https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-jetpack
[container-toolkit]: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
