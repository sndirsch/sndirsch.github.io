---
layout: post
title:  "NVIDIA Open GPU kernel modules: openSUSE/SLE packages available"
date:   2022-06-07
categories: nvidia
---
On May 19, 2022 NVIDIA made a [release][nvidia-release] of their [Open GPU kernel modules][opengpu-github] for their newer GPU platforms (Turing and newer) with Risc-V system processor. Meanwhile we have packages available in our currently supported openSUSE/SLE distributions. If you want to use these you need to install `nvidia-open-driver-G06-signed` package.

## Installation

Installation instructions since Leap 15.6/SLE15-SP6 and Tumbleweed:

{% highlight shell %}
# will install needed packages
zypper in nvidia-open-driver-G06-signed-kmp-default
{% endhighlight %}

Find supported Turing/Ampere/Hopper/Ada/Blackwell GPUs [here][pci_ids-supported]. Check with `inxi -aG`. Use `hwinfo --gfxcard` on SLE.

## Display Drivers

`nvidia-video-G06`, `nvidia-gl-G06` and `nvidia-compute-utils-G06` packages are
available via NVIDIA's [openSUSE][opensuse]/[SLE][sle] repositories, which
then can be used together with NVIDIA's Open GPU kernel modules above.

Installing Display Drivers on Leap 15.6/Tumbleweed/SLE15-SPx

{% highlight shell %}
# if you have not added this repository yet
# Leap 15.6
zypper addrepo https://download.nvidia.com/opensuse/leap/15.6/  nvidia
# Tumbleweed
zypper addrepo https://download.nvidia.com/opensuse/tumbleweed/  nvidia
# SLE15-SP6
zypper addrepo https://download.nvidia.com/suse/sle15sp6/  nvidia
# SLE15-SP7 (Beta)
zypper addrepo https://download.nvidia.com/suse/sle15sp7/  nvidia

# install all required packages
version=$(rpm -qa --queryformat '%{VERSION}\n' nvidia-open-driver-G06-signed-kmp-default | cut -d "_" -f1 | sort -u | tail -n 1)
zypper in nvidia-video-G06 == ${version} nvidia-compute-utils-G06 == ${version}
{% endhighlight %}

## CUDA

With that - after installing `nvidia-compute-utils-G06` (which requires `nvidia-compute-G06`, which contains libcuda) - you can experiment with CUDA. Install [CUDA stack][cuda-stack] from NVIDIA's webserver.

Installing CUDA on Leap 15.6/Tumbleweed/SLE15-SPx

{% highlight shell %}
# if you have not added this repository yet
# Leap 15.6/Tumbleweed
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda
# SLE15-SPx (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/sbsa/  cuda

# will install needed CUDA packages
zypper in cuda-toolkit-12-8

# Unfortunately the following package is not available for aarch64,
# but there are CUDA samples available on GitHub, which can be
# compiled from source: https://github.com/nvidia/cuda-samples
zypper in cuda-demo-suite-12-8
{% endhighlight %}

Let's have a first test for using libcuda (only available on x86_64).

{% highlight shell %}
/usr/local/cuda-12.8/extras/demo_suite/deviceQuery
{% endhighlight %}

## CUDA Minimal Installation

Users, who don't need a graphical desktop, can omit the installation of the display driver packages above and perform a `CUDA Minimal Installation` instead.

{% highlight shell %}
version=$(rpm -qa --queryformat '%{VERSION}\n' nvidia-open-driver-G06-signed-kmp-default | cut -d "_" -f1 | sort -u | tail -n 1)
zypper in nvidia-compute-utils-G06 == ${version} cuda-libraries-12-8
{% endhighlight %}

## Feedback

If you have questions, comments and any kind of feedback regarding this topic, don't hesitate to contact me via email. Thanks!

[nvidia-release]: https://developer.nvidia.com/blog/nvidia-releases-open-source-gpu-kernel-modules/
[opengpu-github]: https://github.com/NVIDIA/open-gpu-kernel-modules
[pci_ids-supported]: https://build.opensuse.org/package/view_file/X11:Drivers:Video:Redesign/nvidia-open-driver-G06-signed/pci_ids-supported
[opensuse]: https://download.nvidia.com/opensuse
[sle]: https://download.nvidia.com/suse
[cuda-stack]: https://developer.download.nvidia.com/compute/cuda/repos/
