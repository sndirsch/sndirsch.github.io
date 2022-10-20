---
layout: post
title:  "Packages needed for Vulkan development on openSUSE"
date:   2022-10-20
categories: vulkan
---

Recently I had a first look into `Vulkan` development. So I started by reading
a [Vulkan Tutorial][tutorial]. It's rather detailed and actually it takes a
long time before you see your first shaded triangle (about 900 lines of
code!). The [Vulkan Tutorial][tutorial] has some software requirements on
Linux, which are explained in detail in the [Development environment for
Linux][requirements]. In order to make things easier for openSUSE users here
is the package list you need to have installed. Just install them via
`zypper`.

Since the tutorial is using C++ ...

{% highlight shell %}
# if you don't have the C++ compiler installed yet
zypper in gcc-c++
{% endhighlight %}

Vulkan packages

{% highlight shell %}
zypper in vulkan-tools vulkan-devel vulkan-validationlayers libvulkan_intel libvulkan_radeon
{% endhighlight %}

Shader Compiler `glsc` for generating `SPIR-V` binaries

{% highlight shell %}
zypper in shaderc
{% endhighlight %}

`GLM` library needed for linear algebra operations (not included by Vulkan, but also popular on OpenGL)

{% highlight shell %}
zypper in glm-devel
{% endhighlight %}

`GLFW` library for window handling, etc. used by the Tutorial (Vulkan is platform-agnostic!)

{% highlight shell %}
zypper in libglfw-devel
{% endhighlight %}

Other needed packages since mentioned in the sample Makefile of the Tutorial

{% highlight shell %}
zypper in libXi-devel libXxf86vm-devel
{% endhighlight %}

![Shaded Triangle](/assets/2022-10-20-vulkan-packages-on-opensuse.jpg)

And now have fun with the [Vulkan Tutorial][tutorial] ! :-)

[tutorial]: https://vulkan-tutorial.com/
[requirements]: https://vulkan-tutorial.com/Development_environment#page_Linux
