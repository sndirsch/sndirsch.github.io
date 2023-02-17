---
layout: post
title:  "NVIDIA Open GPU kernel modules: openSUSE/SLE packages available"
date:   2022-06-07
categories: nvidia
---
On May 19, 2022 NVIDIA made a [release][nvidia-release] of their [Open GPU kernel modules][opengpu-github] for their newer GPU platforms (Turing and newer) with Risc-V system processor. Meanwhile we have openSUSE/SLE packages for simple testing available in the [X11:Drivers:Video:Redesign][x11-drivers-video-redesign] project of our [openSUSE Build Service][obs]. If you want to give these a try you need to install [nvidia-open-driver-G06-signed][kmp] and [kernel-firmware-nvidia-gsp-G06][firmware] packages.

## Installation

Installation instructions for Leap 15.4:

{% highlight shell %}
# if you have not added this repository yet
zypper addrepo -p 90 https://download.opensuse.org/repositories/X11:/Drivers:/Video:/Redesign/openSUSE_Leap_15.4/   X11:Drivers:Video:Redesign
# will install needed packages
zypper in nvidia-open-driver-G06-signed-kmp-default kernel-firmware-nvidia-gsp-G06
{% endhighlight %}

With that you can do a very simple test.

{% highlight shell %}
LD_LIBRARY_PATH=/usr/lib/kernel-firmware-nvidia-gsp-G06 \
/usr/lib/kernel-firmware-nvidia-gsp-G06/nvidia-smi --query
{% endhighlight %}

But unless you have access to one of these Turing or Ampere architecture GPUs (check with `inxi -aG`; use `hwinfo --gfxcard` on SLE):

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
options nvidia NVreg_OpenRmEnableUnsupportedGpus=1
{% endhighlight %}

Find a list of Turing/Ampere GPUs, where you need this option [here][pci_ids-unsupported].

# Secure Boot

Unfortunately the prebuilt kernel modules are not signed yet with the Secureboot Key of openSUSE/SLE, so on such systems you'll need to do this step manually. Which is not a trivial task, I know. :-(

## Display Drivers

`nvidia-video-G06`, `nvidia-gl-G06` and `nvidia-compute-G06` packages are
available via NVIDIA's [openSUSE][opensuse]/[SLE][sle] repositories, which
then can be used together with NVIDIA's Open GPU kernel modules above.

Installing Display Drivers on Leap 15.4

{% highlight shell %}
# if you have not added this repository yet
zypper addrepo -p 99 https://download.nvidia.com/opensuse/leap/15.4/  nvidia
# install all required packages
zypper in nvidia-video-G06 nvidia-gl-G06 nvidia-compute-G06
{% endhighlight %}

## CUDA

With that - after installing `nvidia-compute-G06` (contains libcuda) - you can experiment with CUDA. Install [CUDA stack][cuda-stack] from NVIDIA's webserver.

Installing CUDA on Leap 15.x

{% highlight shell %}
# if you have not added this repository yet
zypper addrepo -p 100 https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# will install needed CUDA packages
zypper in cuda-11-8
{% endhighlight %}

Let's have a first test for using libcuda.

{% highlight shell %}
/usr/local/cuda-11.8/extras/demo_suite/deviceQuery
{% endhighlight %}

## CUDA Minimal Installation

# Repository and Package Dependancies as Overview

![CUDA: Repository and Package Dependancies](/assets/2022-06-07-cuda-repos.svg)

# Explained in Detail

These `CUDA Packages` and `Proprietary:X11:Drivers` repositories on the picture right above are hosted on the `NVIDIA` server, whereas the `X11:Drivers:Video:Redesign` repository on the same picture is hosted on our `openSUSE Build Service`.

What happens is that package `cuda` requires package `cuda-runtime` (both on `CUDA packages` repo), which again requires `cuda-drivers`. The last one is provided by our `nvidia-compute-utils-G06` package on `Proprietary:X11:Drivers` repository. It has higher priority than the `cuda-drivers` meta package from `CUDA Packages` repository, which would require in addition the display driver packages `nvidia-video-G06` and `nvidia-gl-G06` with all kind of dependancies we would like to avoid for a `CUDA Minimal Installation`. Our `nvidia-compute-utils-G06` package on `Proprietary:X11:Drivers` requires `nvidia-compute-G06` (same repository), which again requires `nvidia-open-driver-G06-signed-kmp` package on `openSUSE Build Service` or `nvidia-driver-G06-kmp` package on `Proprietary:X11:Drivers` repository. But the former has a higher priority than the latter because of the repository priorities. Last but not least `kernel-firmware-nvidia-gsp-G06` package is required by `nvidia-open-driver-G06-signed-kmp`.

Example for installation on openSUSE Leap 15.4:

![Minimal CUDA: Zypper Install](/assets/2022-06-07-cuda-zypper-install-output.jpg)

## Feedback

If you have questions, comments and any kind of feedback regarding this topic, don't hesitate to contact me via email. Thanks!

[nvidia-release]: https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/
[opengpu-github]: https://github.com/NVIDIA/open-gpu-kernel-modules
[x11-drivers-video-redesign]: https://build.opensuse.org/project/monitor/X11:Drivers:Video:Redesign
[obs]: https://build.opensuse.org/
[kmp]: https://build.opensuse.org/package/show/X11:Drivers:Video:Redesign/nvidia-open-driver-G06-signed
[firmware]: https://build.opensuse.org/package/show/X11:Drivers:Video:Redesign/kernel-firmware-nvidia-gsp-G06
[pci_ids-unsupported]: https://build.opensuse.org/package/view_file/X11:Drivers:Video:Redesign/nvidia-open-driver-G06-signed/pci_ids-unsupported
[opensuse]: https://download.nvidia.com/opensuse
[sle]: https://download.nvidia.com/suse
[cuda-stack]: https://developer.download.nvidia.com/compute/cuda/repos/
