---
layout: post
title:  "Installation of NVIDIA drivers on openSUSE and SLE"
date:   2025-07-16
categories: nvidia
---

This blogpost covers only installation of `G06` drivers, i.e. drivers for GPUs >= `Maxwell`, i.e.

* [Maxwell, Pascal, Volta][pci_ids-proprietary] (`Proprietary` Kernel driver)
* [Turing and higher][pci_ids-open] (`Open` Kernel driver)

Check with `inxi -aG` on `openSUSE Leap/Tumbleweed` if you have such a GPU. Use `hwinfo --gfxcard` on `SLE`. Use [G04/G05][legacy] legacy drivers (both are `Proprietary` drivers) for older `NVIDIA` GPUs.

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

With the following command the appropriate driver (`Proprietary` or `Open` Kernel driver) will be installed depending on the GPU on your system. In addition the `CUDA` and `Desktop` drivers are installed according to the software packages which are currently installed (`Desktop` driver trigger: `libglvnd` package). 

{% highlight shell %}
zypper inr
{% endhighlight %}

#### Installation of `Open` driver on SLE15-SP6, Leap 15.6 and Tumbleweed

Unfortunately in our `SLE15-SP6`, `Leap 15.6` and `Tumbleweed` repositories we still have driver packages for older `Proprietary` driver (version 550), which are still registered for `Turing+` GPUs. The reason is that at that time the `Open` driver wasn't considered stable yet for the desktop. Therefore, if you own a `Turing+` GPU (check above) and would like to use the `Open` driver (which is recommended!) please use the following command instead of the above.

{% highlight shell %}
zypper in nvidia-open-driver-G06-signed-kmp-meta
{% endhighlight %}

Otherwise you will end up with a `Proprietary` driver release 550 initially, which then will be updated later to the current version of the `Proprietary` driver, but not replaced by the open driver automatically.

#### Understanding package dependancies

The following graphics explains the installation and package dependancies. Zoom in for better reading.

![gfx-repo](/assets/2025-07-16-gfx-repo.svg)

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
# Leap 15.6/16.0(Beta)/Tumbleweed
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/opensuse15/x86_64/  cuda
# SLE15-SPx/SLE16(Beta) (x86_64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/x86_64/  cuda
# SLE15-SPx/SLE16(Beta) (aarch64)
zypper addrepo https://developer.download.nvidia.com/compute/cuda/repos/sles15/sbsa/  cuda
{% endhighlight %}

#### Use `Open` prebuilt/secureboot-signed Kernel driver (GPU >= `Turing`)

In case you have a `Turing` or later GPU it is strongly recommended to use our `prebuilt` and `secureboot-signed` Kernel driver. Unfortunately this is often not the latest driver, which is availabe, since this driver needs to go through our official `QA` and `Maintenance` process before it can be released through our product update channels, but things are much easier to handle for the user.

{% highlight shell %}
# Install open prebuilt/secureboot-signed Kernel driver
zypper in nvidia-open-driver-G06-signed-cuda-kmp-default

# Make sure userspace CUDA/Desktop drivers will be in sync with just installed open prebuilt/secureboot-signed Kernel driver
version=$(rpm -qa --queryformat '%{VERSION}\n' nvidia-open-driver-G06-signed-cuda-kmp-default | cut -d "_" -f1 | sort -u | tail -n 1)

# Install CUDA drivers
zypper in nvidia-compute-utils-G06 == ${version} nvidia-persistenced == ${version}
# Install Desktop drivers
zypper in nvidia-video-G06 == ${version}
{% endhighlight %}

#### Use `Open` DKMS Kernel driver on GPUs >= `Turing` (latest driver available)

If you really need the latest `Open` driver (also for `Turing` and later), use `NVIDIA`'s `Open` DKMS Kernel driver. This will build this driver on demand for the appropriate Kernel during the boot process.

{% highlight shell %}
# Install latest Open DKMS Kernel driver 
zypper in nvidia-open-driver-G06

# Install CUDA drivers
zypper in nvidia-compute-utils-G06

# Install Desktop drivers
zypper in nvidia-video-G06
{% endhighlight %}

On `Secure Boot` systems you still need to import the `certificate`, so you can later `enroll` it right after reboot in the `MOK-Manager` by using your `root` password.

{% highlight shell %}
mokutil --import /var/lib/dkms/mok.pub --root-pw
{% endhighlight %}

Otherwise your freshly built kernel modules can't be loaded by your kernel later.

#### Use `Proprietary` DKMS Kernel driver on `Maxwell` <= GPU < `Turing`

For `Maxwell`, `Pascal` and `Volta` you need to use the `Proprietary` DKMS Kernel driver.

{% highlight shell %}
# Install proprietary DKMS Kernel driver
zypper in nvidia-driver-G06

# Install CUDA drivers
zypper in nvidia-compute-utils-G06

# Install Desktop drivers
zypper in nvidia-video-G06
{% endhighlight %}

### Installation of CUDA

In case you used `GFX Repository` for installing `NVIDIA` drivers before, first add the `CUDA Repository` as outlined above in `CUDA Repository` chapter.

The following commands will install `CUDA` packages themselves. It describes a regular and minimal installation. In addition it makes it easy to do first tests with `CUDA`. Depending on which Kernel driver is being used it may be needed to install different `CUDA` versions.

{% highlight shell %}
# Kernel driver being installed via GFX Repo
cuda_version=13-0
# Kernel driver being installed via CUDA Repo
cuda_version=13-0

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
# only if you have Turing and higher, i.e. use Open Kernel driver
zypper addlock nvidia-driver-G06-kmp-default
# unless you plan to use the DKMS Open driver package from CUDA repository
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

[pci_ids-proprietary]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-driver-G06/files/pci_ids-supported?expand=1
[pci_ids-open]: https://build.opensuse.org/projects/X11:Drivers:Video:Redesign/packages/nvidia-open-driver-G06-signed/files/pci_ids-supported?expand=1
[legacy]:https://en.opensuse.org/SDB:NVIDIA_drivers
