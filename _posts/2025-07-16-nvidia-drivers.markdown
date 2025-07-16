---
layout: post
title:  "Installation of NVIDIA drivers on openSUSE and SLE"
date:   2025-07-16
categories: nvidia
---

This blogpost covers only installation of G06 drivers, i.e. drivers for GPUs >= Maxwell, i.e.

* [Maxwell, Pascal, Volta][pci_ids-proprietary] (proprietary kernel driver)
* [Turing and higher][pci_ids-open] (open kernel driver)

Check with `inxi -aG` on openSUSE Leap/Tumbleweed if you have such a GPU. Use `hwinfo --gfxcard` on SLE. Use [G04/G05][legacy] legacy drivers (both proprietary driver) for older
NVIDIA GPUs.

There are two different ways to install NVIDIA drivers. Either use `GFX Repository` or use `CUDA Repository`.

### GFX Repository

First add the repository if it has not been added yet. On openSUSE Leap/Tumbleweed and SLE 15 Desktop and SLE 15 Workstation Extension it is been added by default. So check first, if it has already been added.

{% highlight shell %}
# openSUSE Leap/Tumbleweed
zypper repos -u | grep https://download.nvidia.com/opensuse/
# SLE
zypper repos -u | grep https://download.nvidia.com/suse
{% endhighlight %}

If the output is empty add the repository now:

{% highlight shell %}
# Leap 15.6
zypper addrepo https://download.nvidia.com/opensuse/leap/15.6/  nvidia
# Leap 16.0 (Beta)
zypper addrepo https://download.nvidia.com/opensuse/leap/16.0/  nvidia
# Tumbleweed
zypper addrepo https://download.nvidia.com/opensuse/tumbleweed/  nvidia
# SLE15-SP6
zypper addrepo https://download.nvidia.com/suse/sle15sp6/  nvidia
# SLE15-SP7
zypper addrepo https://download.nvidia.com/suse/sle15sp7/  nvidia
# SLE16 (Beta)
zypper addrepo https://download.nvidia.com/suse/sle16/  nvidia
{% endhighlight %}

With the following command the appropriate driver (proprietary or open kernel driver) will be installed depending on the GPU on your system. In addition the CUDA and Desktop drivers are installed according to the software packages which are currently installed (Desktop driver trigger: libglvnd package). 

{% highlight shell %}
zypper inr
{% endhighlight %}

The following graphics explains the installation and package dependancies.

![gfx-repo](/assets/2025-07-16-gfx-repo.svg)

### CUDA Repository

Add the repository if it hasn't been added yet. On SLE15 it might have already been added as a module. So check first:

{% highlight shell %}
# openSUSE Leap/Tumbleweed
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/opensuse15
# SLE
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/sles15
{% endhighlight %}

If the output is empty add the repository now:

{% highlight shell %}
# Leap 15.6/16.0(Beta)/Tumbleweed
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx/SLE16(Beta) (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda
# SLE15-SPx/SLE16(Beta) (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/sbsa/  cuda
{% endhighlight %}

#### Use Open prebuilt/secureboot-signed kernel driver (GPU >= Turing)

{% highlight shell %}
# Install open prebuilt/secureboot-signed Kernel driver
zypper in nvidia-open-driver-G06-signed-cuda-kmp-default

# Make sure userspace CUDA/Desktop drivers will be in sync with just installed open prebuilt/secureboot-signed Kernel driver
version=$(rpm -qa --queryformat '%{VERSION}\n' nvidia-open-driver-G06-signed-cuda-kmp-default | cut -d "_" -f1 | sort -u | tail -n 1)

# Install CUDA drivers
zypper in nvidia-compute-utils-G06 == ${version} 
# Install Desktop drivers
zypper in nvidia-video-G06 == ${version}
{% endhighlight %}

#### Use Open DKMS Kernel driver on GPUs >= Turing (latest driver available)

{% highlight shell %}
# Install latest Open DKMS Kernel driver 
zypper in nvidia-open-driver-G06

# Install CUDA drivers
zypper in nvidia-compute-utils-G06

# Install Desktop drivers
zypper in nvidia-video-G06
{% endhighlight %}


#### Use Proprietary DKMS Kernel driver on Maxwell >= GPU < Turing

{% highlight shell %}
# Install proprietary DKMS Kernel driver
zypper in nvidia-driver-G06

# Install CUDA drivers
zypper in nvidia-compute-utils-G06

# Install Desktop drivers
zypper in nvidia-video-G06
{% endhighlight %}

### Installation of CUDA

In case you used `GFX Repository` for installing NVIDIA drivers before, first add the CUDA repository as outlined above in `CUDA Repository` chapter.

The following commands will install CUDA itself. It describes a regular and minimal installation. In addition it makes it easy to do first tests with CUDA. Depending on which Kernel driver is being used it may be needed to install different CUDA versions.

{% highlight shell %}
# Kernel driver being installed via GFX Repo
# Regular installation
zypper in cuda-toolkit-12-8
# Minimal installation
zypper in cuda-libraries-12-8

# Kernel driver being installed via CUDA Repo
# Regular installation
zypper in cuda-toolkit-12-9
# Minimal installation
zypper in cuda-libraries-12-9

# Unfortunately the following package is not available for aarch64,
# but there are CUDA samples available on GitHub, which can be
# compiled from source: https://github.com/nvidia/cuda-samples
zypper in cuda-demo-suite-12-8
{% endhighlight %}

Let’s have a first test for using libcuda (only available on x86_64).

{% highlight shell %}
# Kernel driver being installed via GFX Repo
/usr/local/cuda-12.8/extras/demo_suite/deviceQuery

# Kernel driver being installed via CUDA Repo:
/usr/local/cuda-12.9/extras/demo_suite/deviceQuery
{% endhighlight %}

### Which one to choose for NVIDIA driver installation: GFX or CUDA Repository?
Good question! Not so easy to answer. If you rely on support from NVIDIA (especially when using SLE), for Compute Usage we strongly recommend to use the `CUDA Repository` for NVIDIA driver installation. Even if you use NVIDIA Desktop drivers as well. 

For others - usually running openSUSE Leap/Tumbleweed - it's fine to use `GFX Repository` for NVIDIA driver installation and adding `CUDA Repository` for adding CUDA packages.

[pci_ids-proprietary]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-driver-G06/files/pci_ids-570.169?expand=1
[pci_ids-open]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-open-driver-G06-signed/files/pci_ids-supported?expand=1
[legacy]:https://en.opensuse.org/SDB:NVIDIA_drivers
