---
layout: post
title:  "Installation of NVIDIA drivers on openSUSE and SLE (G07)"
date:   2026-02-22
categories: nvidia
---

### Important

This blogpost explains how to install new `G07 NVIDIA` drivers. It is temporarily available as long as the content of the current [blogpost][g06-doc] for installation of `G06 NVIDIA` drivers is still needed.

We're currently in the release process of `G07 NVIDIA` driver packages.

At that moment this blogpost can be used for the following `openSUSE` and `SLE` products:

* `Leap 15.6` / `SLE 15 SP6`
* `SLE 15 SP7`
* `Leap 16.0` / `SLE 16`
* `openSUSE Tumbleweed`

For the following `openSUSE` and `SLE` products you still need to use the current [blogpost][g06-doc]  for installation of `G06 NVIDIA` drivers:

* (empty list)

Both lists above are updated when driver packages are becoming available for the appropriate products.

### Let's begin

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

![gfx-repo](/assets/2026-02-22-gfx-repo.svg)

Once `in-sync` becomes `latest` driver version, i.e the `nvidia-open-driver-G07-kmp-<flavor>` of `latest` driver has been released for your product and `nvidia-open-driver-G07-kmp-meta` has been updated accordingly all remaining userspace driver packages (`nvidia-video-G07`, `nvidia-compute-utils-G07` and dependancies) get updated to `latest` driver version.

### CUDA Repository

Add the repository if it hasn't been added yet. On `SLE15/SLE16` it might have already been added as a`Module`. So check first:

{% highlight shell %}
# openSUSE Leap 15.6
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/opensuse15
# SLE15
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/sles15
# SLE16/Leap 16.x/Tumbleweed
zypper repos -u | grep https://developer.download.nvidia.com/compute/cuda/repos/suse16
{% endhighlight %}

Verify that the repository is enabled. If the output is empty add the repository now:

{% highlight shell %}
# Leap 15.6
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda
# SLE15-SPx (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/sbsa/  cuda
# SLE16/SLE16.1(Beta)/Leap 16.0/16.1(Beta)/Tumbleweed (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/suse16/x86_64/  cuda
# SLE16/SLE16.1(Beta)/Leap 16.0/16.1(Beta)/Tumbleweed (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/suse16/sbsa/  cuda
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
cuda_version=13-2
# Kernel driver being installed via CUDA Repo
cuda_version=13-2

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

#### Mistakenly selected/installed packages

It may happen that some old or not recommended driver packages get mistakenly auto-selected for installation or even have already been mistakenly installed. These are:

##### On Leap 15.6/SLE15-SP6, Leap 16.0/SLE16 and Tumbleweed

* nvidia-open-driver-G06-signed-kmp-meta 580.xxx.yy (GFX repo)

##### On Leap 15.6/SLE15-SP6 and Tumbleweed

* nvidia-driver-G06-kmp-default 550.xxx.yy/570.xxx.yy (GFX repo)
* nvidia-open-driver-G07 595.xx.yy (CUDA repo)

##### On Leap 15.6/SLE15-SP6/SLE15-SP7

* nvidia-open-driver-G06 580.xxx.yy (CUDA repo)
* nvidia-open-driver-G06-kmp-default  570.xxx.yy (CUDA repo)
* nvidia-open-driver-G06-signed-kmp-default  570.xxx.yy (SLE15-SP6/SLE15-SP7)

##### On SLE15-SP7/Leap 16.1/SLE16.1

* nvidia-open-driver-G07  595.xx.yy (CUDA repo)

##### On SLE15-SP7

* nvidia-gfxG05-kmp-default 535.xxx.yy (CUDA repo)
* nvidia-open-gfxG05-kmp-default 535.xxx.yy (CUDA repo)

##### On Tumbleweed

* nvidia-gfxG05-kmp-default 470.xxx.yy (GFX repo)

In order to avoid mistakenly installing them add package locks for them with zypper.

{% highlight shell %}
# Leap 15.6/SLE15-SP6
zypper addlock \
  nvidia-open-driver-G06-signed-kmp-meta \
  nvidia-driver-G06-kmp-default \
  nvidia-open-driver-G07 \
  nvidia-open-driver-G06 \
  nvidia-open-driver-G06-kmp-default \
  nvidia-open-driver-G06-signed-kmp-default
# SLE15-SP7
zypper addlock \
  nvidia-gfxG05-kmp-default \
  nvidia-open-gfxG05-kmp-default \
  nvidia-open-driver-G06 \
  nvidia-open-driver-G06-kmp-default \
  nvidia-open-driver-G06-signed-kmp-default \
  nvidia-open-driver-G07
# Leap 16.0/SLE16
zypper addlock nvidia-open-driver-G06-signed-kmp-meta
# Leap 16.1/SLE16.1
zypper addlock nvidia-open-driver-G07
# Tumbleweed
zypper addlock \
  nvidia-open-driver-G06-signed-kmp-meta \
  nvidia-driver-G06-kmp-default \
  nvidia-open-driver-G07 \
  nvidia-gfxG05-kmp-default
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

[g06-doc]: https://sndirsch.github.io/nvidia/2025/07/16/nvidia-drivers.html
[pci_ids-open]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-open-driver-G07-signed/files/pci_ids-supported?expand=1
[legacy]:https://en.opensuse.org/SDB:NVIDIA_drivers
