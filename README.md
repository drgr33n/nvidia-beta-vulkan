Nvidia Beta Vulkan
==================

[![ci](https://github.com/drgr33n/nvidia-beta-vulkan/actions/workflows/ci.yml/badge.svg)](https://github.com/drgr33n/nvidia-beta-vulkan/actions/workflows/ci.yml)

This is just a mash of both the Nvidia Beta DKMS and Nvidia Vulkan projects from AUR. I've mashed them together as Nvidia has just released v470 of their drivers as beta. This brings a bunch of much awaited features including fixing xwayland. 

Credits to both authors of the original projects!

* jcstryker <public at jasonstryker dot com>
* Daniel Bermond <dbermond@archlinux.org>

NOTE
----

This will fail if you use makepkg install the AUR using ``makepkg -si``. This is because this PKGBUILD will build the standard and DKMS packages and they conflict. You can install them individually by using ``pacman -U`` for example.

```
pacman -U \
lib32-nvidia-vulkan-utils-470.42.01-2-x86_64.pkg.tar.zst \
nvidia-vulkan-470.42.01-2-x86_64.pkg.tar.zst \
nvidia-vulkan-utils-470.42.01-2-x86_64.pkg.tar.zst \
lib32-opencl-nvidia-vulkan-470.42.01-2-x86_64.pkg.tar.zst \
opencl-nvidia-vulkan-470.42.01-2-x86_64.pkg.tar.zst
```

To install the standard packages and 

```
pacman -U \
lib32-nvidia-vulkan-utils-470.42.01-2-x86_64.pkg.tar.zst \
nvidia-vulkan-dkms-470.42.01-2-x86_64.pkg.tar.zst \
nvidia-vulkan-utils-470.42.01-2-x86_64.pkg.tar.zst \
lib32-opencl-nvidia-vulkan-470.42.01-2-x86_64.pkg.tar.zst \
opencl-nvidia-vulkan-470.42.01-2-x86_64.pkg.tar.zst
```

For DKMS

Enjoy!
