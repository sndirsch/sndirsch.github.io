---
layout: post
title:  "Installation of NVIDIA drivers on openSUSE and SLE"
date:   2025-07-16
categories: nvidia
---

This blogpost covers only installation of `G07` drivers, i.e. drivers for GPUs >= `Turing`, i.e.

* [Turing and higher][pci_ids-open] (`Open` Kernel driver)

Check with `inxi -aG` on `openSUSE Leap/Tumbleweed` if you have such a GPU. Use `hwinfo --gfxcard` on `SLE`. Use [G04/G05/G06][legacy] legacy drivers (`Proprietary` drivers) for older `NVIDIA` GPUs.

There are two different ways to install `NVIDIA` drivers. Either use `GFX Repository` or use `CUDA Repository`.

### GFX Repository

First add the repository if it has not been added yet. On `openSUSE Leap/Tumbleweed` and `SLE 15 Desktop` and `SLE 15 Workstation Extension` it is being added by default. So check first, if it has already been added.

{% highlight shell %}
# openSUSE Leap/Tumbleweed
zypper repos -u | grep https://download.nvidia.com/opensuse/
# SLE
zypper repos -u | grep https://download.nvidia.com/suse
{% endhighlight %}

Verify that the repository is enabled. If the output was empty add the repository now:

{% highlight shell %}
# Leap 15.6
zypper addrepo https://download.nvidia.com/opensuse/leap/15.6/  nvidia
# Leap 16.0
zypper addrepo https://download.nvidia.com/opensuse/leap/16.0/  nvidia
# Leap 16.1 (Beta)
zypper addrepo https://download.nvidia.com/opensuse/leap/16.1/  nvidia
# Tumbleweed
zypper addrepo https://download.nvidia.com/opensuse/tumbleweed/  nvidia
# SLE15-SP6
zypper addrepo https://download.nvidia.com/suse/sle15sp6/  nvidia
# SLE15-SP7
zypper addrepo https://download.nvidia.com/suse/sle15sp7/  nvidia
# SLE16
zypper addrepo https://download.nvidia.com/suse/sle16/  nvidia
# SLE16.1 (Beta)
zypper addrepo https://download.nvidia.com/suse/sle16.1/  nvidia
{% endhighlight %}

With the following command the `Open` Kernel driver will be installed. In addition the `CUDA` and `Desktop` drivers are installed according to the software packages which are currently installed (`Desktop` driver trigger: `libglvnd` package). 

{% highlight shell %}
zypper in nvidia-open-driver-G07-signed-kmp-meta
{% endhighlight %}

#### Understanding package dependancies

The following graphics explains the installation and package dependancies. Zoom in for better reading.

![gfx-repo](/assets/2025-07-16-gfx-repo.svg)

Once `in-sync` becomes `latest` driver version, i.e the `nvidia-open-driver-G07-kmp-<flavor>` of `latest` driver has been released for your product and `nvidia-open-driver-G07-kmp-meta` has been updated accordingly all remaining userspace driver packages (`nvidia-video-G07`, `nvidia-compute-utils-G07` and dependancies) get updated to `latest` driver version.

### CUDA Repository

Add the repository if it hasn't been added yet. On `SLE15` it might have already been added as a`Module`. So check first:

{% highlight shell %}
# openSUSE Leap/Tumbleweed
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/opensuse15
# SLE
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/sles15
{% endhighlight %}

Verify that the repository is enabled. If the output is empty add the repository now:

{% highlight shell %}
# Leap 15.6/16.0/16.1(Beta)/Tumbleweed
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx/SLE16/SLE16.1(Beta) (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda
# SLE15-SPx/SLE16/SLE16.1(Beta) (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/sbsa/  cuda
{% endhighlight %}

#### Use `Open` prebuilt/secureboot-signed Kernel driver

It is strongly recommended to use our `prebuilt` and `secureboot-signed` Kernel driver. Unfortunately this is often not the latest driver, which is availabe, since this driver needs to go through our official `QA` and `Maintenance` process before it can be released through our product update channels, but things are much easier to handle for the user.

{% highlight shell %}
# Install open prebuilt/secureboot-signed Kernel driver
zypper in nvidia-open-driver-G07-signed-cuda-kmp-default

# Make sure userspace CUDA/Desktop drivers will be in sync with just installed open prebuilt/secureboot-signed Kernel driver
version=$(rpm -qa --queryformat '%{VERSION}\n' nvidia-open-driver-G07-signed-cuda-kmp-default | cut -d "_" -f1 | sort -u | tail -n 1)

# Install CUDA drivers
zypper in nvidia-compute-utils-G07 == ${version} nvidia-persistenced == ${version}
# Install Desktop drivers
zypper in nvidia-video-G07 == ${version}
{% endhighlight %}

#### Use `Open` DKMS Kernel driver (latest driver available)

If you really need the latest `Open` driver, use `NVIDIA`'s `Open` DKMS Kernel driver. This will build this driver on demand for the appropriate Kernel during the boot process.

{% highlight shell %}
# Install latest Open DKMS Kernel driver 
zypper in nvidia-open-driver-G07

# Install CUDA drivers
zypper in nvidia-compute-utils-G07

# Install Desktop drivers
zypper in nvidia-video-G07
{% endhighlight %}

On `Secure Boot` systems you still need to import the `certificate`, so you can later `enroll` it right after reboot in the `MOK-Manager` by using your `root` password.

{% highlight shell %}
mokutil --import /var/lib/dkms/mok.pub --root-pw
{% endhighlight %}

Otherwise your freshly built kernel modules can't be loaded by your kernel later.

### Installation of CUDA

In case you used `GFX Repository` for installing `NVIDIA` drivers before, first add the `CUDA Repository` as outlined above in `CUDA Repository` chapter.

The following commands will install `CUDA` packages themselves. It describes a regular and minimal installation. In addition it makes it easy to do first tests with `CUDA`. Depending on which Kernel driver is being used it may be needed to install different `CUDA` versions.

{% highlight shell %}
# Kernel driver being installed via GFX Repo
cuda_version=13-1
# Kernel driver being installed via CUDA Repo
cuda_version=13-1

# Regular installation
zypper in cuda-toolkit-${cuda_version}
# Minimal installation
zypper in cuda-libraries-${cuda_version}

# Unfortunately the following package is not available for aarch64,
# but there are CUDA samples available on GitHub, which can be
# compiled from source: https://github.com/nvidia/cuda-samples
zypper in cuda-demo-suite-12-9
{% endhighlight %}

Let’s have a first test for using `libcuda` (only available on x86_64).

{% highlight shell %}
/usr/local/cuda-12/extras/demo_suite/deviceQuery
{% endhighlight %}

### Which one to choose for NVIDIA driver installation: GFX or CUDA Repository?
Good question! Not so easy to answer. If you rely on support from `NVIDIA` (especially when using `SLE`), for `Compute` usage we strongly recommend to use the ``CUDA Repository`` for `NVIDIA` driver installation. Even if you use `NVIDIA` Desktop drivers as well. 

For others - usually running `openSUSE Leap/Tumbleweed` - it's fine to use `GFX Repository` for `NVIDIA` driver installation and adding `CUDA Repository` for installing `CUDA` packages.

### Migration from G06 to G07 Open drivers

Migration from G06 `Open` drivers to G07 is a manual step. Uninstall all NVIDIA driver packages first:

{% highlight shell %}
rpm -e $(rpm -qa | grep -e ^nvidia -e ^libnvidia | grep -v container)
{% endhighlight %}

Then install G07 `Open` drivers as described in the sections above.

### Known issues

#### CUDA Repository

Once you have added the `CUDA Repository` it may happen that some old or not recommended driver packages get mistakenly auto-selected for installation or even have already been mistakenly installed. These are:

* nvidia-gfxG05-kmp-default  535.x
* nvidia-open-gfxG05-kmp-default  535.x
* nvidia-open-driver-G06-kmp-default  570.x
* nvidia-driver-G06-kmp-default  570.x
* nvidia-open-driver-G06

In order to avoid mistakenly installing them add package locks for them with zypper.

{% highlight shell %}
zypper addlock nvidia-gfxG05-kmp-default
zypper addlock nvidia-open-gfxG05-kmp-default
zypper addlock nvidia-open-driver-G06-kmp-default
zypper addlock nvidia-driver-G06-kmp-default
zypper addlock nvidia-open-driver-G06
{% endhighlight %}

In case you see any of these packages already installed on your system, better read the Troubleshooting section below how to get rid of these and all other nvidia driver packages related to them. Afterwards add locks to them as described right above.

#### Tumbleweed

On Tumbleweed it may happen that some legacy driver packages get mistakenly auto-selected for installation or even have already been mistakenly installed. These are:

* nvidia-gfxG04-kmp-default
* nvidia-gfxG05-kmp-default

In order to avoid mistakenly installing them add package locks for them with zypper.

{% highlight shell %}
zypper addlock nvidia-gfxG04-kmp-default
zypper addlock nvidia-gfxG05-kmp-default
{% endhighlight %}

In case you see any of these packages already installed on your system, better read the Troubleshooting section below how to get rid of these and all other nvidia driver packages related to them. Afterwards add locks to them as described right above.

#### Leap 15.6

On Leap 15.6 when doing a `zypper dup` this may result in a proposal to dowgrade the driver packages to some older 570 version and switching to `-azure` kernel flavor at the same time. The culprit for this issue is currently unknown, but you can prevent it from happening by adding a package lock with zypper.

{% highlight shell %}
zypper addlock nvidia-open-driver-G06-signed-kmp-azure
{% endhighlight %}

### Troubleshooting

In case you got lost in a mess of nvidia driver packages for different driver versions the best way to figure out what the current state the system is in is to run:

{% highlight shell %}
rpm -qa | grep -e ^nvidia -e ^libnvidia | grep -v container | sort
{% endhighlight %}

Often then the best approach is to begin from scratch, i.e remove all the nvidia driver packages by running:

{% highlight shell %}
rpm -e $(rpm -qa | grep -e ^nvidia -e ^libnvidia | grep -v container)
{% endhighlight %}

Then follow (again) the instructions above for installing the driver using the GFX or CUDA Repository.

[pci_ids-open]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-open-driver-G07-signed/files/pci_ids-supported?expand=1
[legacy]:https://en.opensuse.org/SDB:NVIDIA_drivers
