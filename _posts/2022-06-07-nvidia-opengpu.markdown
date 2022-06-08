---
layout: post
title:  "NVIDIA Open GPU kernel modules: (open)SUSE packages available"
date:   2022-06-07
categories: nvidia
---
On May 19, 2022 NVIDIA made a [release][nvidia-release] of their [Open GPU kernel modules][opengpu-github] for their newer GPU platforms (Turing and newer) with Risc-V system processor. Meanwhile we have (open)SUSE packages for simple testing available in the [X11:Drivers:Video][x11-drivers-video] project of our [openSUSE Build Service][obs]. If you want to give these a try you need to install [ nvidia-open-gfxG06 ][kmp] and [kernel-firmware-nvidia-gsp][firmware] packages.

Installation instructions for openSUSE 15.4:

{% highlight shell %}
# if you have not added this repository yet
zypper addrepo https://download.opensuse.org/repositories/X11:/Drivers:/Video/openSUSE_Leap_15.4/   X11:Drivers:Video
# will install needed packages
zypper in nvidia-open-gfxG06-kmp-default kernel-firmware-nvidia-gsp
{% endhighlight %}

With that you can do a very simple test.

{% highlight shell %}
nvidia-smi --query
{% endhighlight %}

But unless you have access to one of these Turing or Ampere architecture GPUs (check with `inxi -aG`):

| NVIDIA GPU product | Device PCI ID * |
|--------------------|-----------------|
| Tesla T10 | 1E37 10DE 1370 |
| NVIDIA T4G                           | 1EB4 10DE 157D |
| Tesla T4                             | 1EB8           |
| NVIDIA T4 32GB                       | 1EB9           |
| NVIDIA A100-PG509-200                | 20B0 10DE 1450 |
| NVIDIA A100-SXM4-40GB                | 20B0           |
| NVIDIA A100-PCIE-40GB                | 20B1 10DE 145F |
| NVIDIA A100-SXM4-80GB                | 20B2 10DE 1463 |
| NVIDIA A100-SXM4-80GB                | 20B2 10DE 147F |
| NVIDIA A100-SXM4-80GB                | 20B2 10DE 1484 |
| NVIDIA PG506-242                     | 20B3 10DE 14A7 |
| NVIDIA PG506-243                     | 20B3 10DE 14A8 |
| NVIDIA A100-PCIE-80GB                | 20B5 10DE 1533 |
| NVIDIA PG506-230                     | 20B6 10DE 1491 |
| NVIDIA PG506-232                     | 20B6 10DE 1492 |
| NVIDIA A30                           | 20B7 10DE 1532 |
| NVIDIA A100-PG506-207                | 20F0 10DE 1583 |
| NVIDIA A100-PCIE-40GB                | 20F1 10DE 145F |
| NVIDIA A100-PG506-217                | 20F2 10DE 1584 |
| NVIDIA A40                           | 2235 10DE 145A |
| NVIDIA A10                           | 2236           |
| NVIDIA A10G                          | 2237           |
| NVIDIA A16                           | 25B6 10DE 14A9 |
| NVIDIA A2                            | 25B6 10DE 157E |

you'll need to remove the `#` from the options line in `/usr/lib/modprobe.d/50-nvidia-default.conf` or `/etc/modprobe.d/50-nvidia-default.conf` respectively before.

{% highlight shell %}
### Enable support on *all* Turing/Ampere GPUs: Alpha Quality!
#options nvidia NVreg_OpenRmEnableUnsupportedGpus=1
{% endhighlight %}

Unfortunately the prebuilt kernel modules are not signed yet with (open)SUSE's Secureboot Key, so on such systems you'll need to do this step manually. Which is not a trivial task, I know. :-(

I'm planning to have `x11-video-nvidiaG06`, `nvidia-glG06` and `nvidia-computeG06` packages available via NVIDIA's (open)SUSE repositories in week 24 (next week), which then can be used together with NVIDIA's Open GPU kernel modules above.

If you have questions, comments and any kind of feedback regarding this topic, don't hesitate to contact me via email. Thanks!

[nvidia-release]: https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/
[opengpu-github]: https://github.com/NVIDIA/open-gpu-kernel-modules
[x11-drivers-video]: https://build.opensuse.org/project/monitor/X11:Drivers:Video
[obs]: https://build.opensuse.org/
[kmp]: https://build.opensuse.org/package/show/X11:Drivers:Video/nvidia-open-gfxG06
[firmware]: https://build.opensuse.org/package/show/X11:Drivers:Video/kernel-firmware-nvidia-gsp

