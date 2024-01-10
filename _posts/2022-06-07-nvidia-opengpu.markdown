---
layout: post
title:  "NVIDIA Open GPU kernel modules: openSUSE/SLE packages available"
date:   2022-06-07
categories: nvidia
---
On May 19, 2022 NVIDIA made a [release][nvidia-release] of their [Open GPU kernel modules][opengpu-github] for their newer GPU platforms (Turing and newer) with Risc-V system processor. Meanwhile we have packages available in our currently supported openSUSE/SLE distributions. If you want to use these you need to install `nvidia-open-driver-G06-signed` and `kernel-firmware-nvidia-gspx-G06` packages.

## Installation

Installation instructions since Leap 15.4/SLE15-SP4 and Tumbleweed:

{% highlight shell %}
# will install needed packages
zypper in nvidia-open-driver-G06-signed-kmp-default kernel-firmware-nvidia-gspx-G06
{% endhighlight %}

Find supported Turing/Ampere/Hopper GPUs [here][pci_ids-unsupported]. Check with `inxi -aG`. Use `hwinfo --gfxcard` on SLE.

## Display Drivers

`nvidia-video-G06`, `nvidia-gl-G06` and `nvidia-compute-G06` packages are
available via NVIDIA's [openSUSE][opensuse]/[SLE][sle] repositories, which
then can be used together with NVIDIA's Open GPU kernel modules above.

Installing Display Drivers on Leap 15.x/Tumbleweed/SLE15-SPx

{% highlight shell %}
# if you have not added this repository yet
# Leap 15.4
zypper addrepo -p 99 https://download.nvidia.com/opensuse/leap/15.4/  nvidia
# Leap 15.5
zypper addrepo -p 99 https://download.nvidia.com/opensuse/leap/15.5/  nvidia
# Leap 15.6
zypper addrepo -p 99 https://download.nvidia.com/opensuse/leap/15.6/  nvidia
# Tumbleweed
https://download.nvidia.com/opensuse/tumbleweed/
# SLE15-SP4
zypper addrepo -p 99 https://download.nvidia.com/suse/sle15sp4/  nvidia
# SLE15-SP5
zypper addrepo -p 99 https://download.nvidia.com/suse/sle15sp5/  nvidia
# SLE15-SP6
zypper addrepo -p 99 https://download.nvidia.com/suse/sle15sp6/  nvidia

# install all required packages
zypper in nvidia-video-G06 nvidia-gl-G06 nvidia-compute-G06
{% endhighlight %}

## CUDA

With that - after installing `nvidia-compute-G06` (contains libcuda) - you can experiment with CUDA. Install [CUDA stack][cuda-stack] from NVIDIA's webserver.

Installing CUDA on Leap 15.x/Tumbleweed/SLE15-SPx

{% highlight shell %}
# if you have not added this repository yet
# Leap 15.x/Tumbleweed
zypper addrepo -p 100 https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx
zypper addrepo -p 100 https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda

# will install needed CUDA packages
zypper in cuda-12-3
{% endhighlight %}

Let's have a first test for using libcuda.

{% highlight shell %}
/usr/local/cuda-12.3/extras/demo_suite/deviceQuery
{% endhighlight %}

## CUDA Minimal Installation

# Repository and Package Dependancies as Overview

![CUDA: Repository and Package Dependancies](/assets/2022-06-07-cuda-repos.svg)

# Explained in Detail

These `CUDA Packages` and `Proprietary:X11:Drivers` repositories on the picture right above are hosted on the `NVIDIA` server.

What happens is that package `cuda` requires package `cuda-runtime` (both in `CUDA packages` repo), which again requires `cuda-drivers`. The last one is provided by our `nvidia-compute-utils-G06` package in `Proprietary:X11:Drivers` repository. It has higher priority than the `cuda-drivers` meta package from `CUDA Packages` repository, which would require in addition the display driver packages `nvidia-video-G06` and `nvidia-gl-G06` with all kind of dependancies we would like to avoid for a `CUDA Minimal Installation`. Our `nvidia-compute-utils-G06` package in `Proprietary:X11:Drivers` requires `nvidia-compute-G06` (same repository), which again requires `nvidia-open-driver-G06-signed-kmp` package included in our openSUSE/SLE distributions or `nvidia-driver-G06-kmp` package in `Proprietary:X11:Drivers` repository. Last but not least `kernel-firmware-nvidia-gspx-G06` package is required by `nvidia-open-driver-G06-signed-kmp`.

Example for installation on openSUSE Leap 15.4:

![Minimal CUDA: Zypper Install](/assets/2022-06-07-cuda-zypper-install-output.jpg)

## Feedback

If you have questions, comments and any kind of feedback regarding this topic, don't hesitate to contact me via email. Thanks!

[nvidia-release]: https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/
[opengpu-github]: https://github.com/NVIDIA/open-gpu-kernel-modules
[pci_ids-unsupported]: https://build.opensuse.org/package/view_file/X11:Drivers:Video:Redesign/nvidia-open-driver-G06-signed/pci_ids-unsupported
[opensuse]: https://download.nvidia.com/opensuse
[sle]: https://download.nvidia.com/suse
[cuda-stack]: https://developer.download.nvidia.com/compute/cuda/repos/
