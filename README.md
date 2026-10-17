Updated NVIDIA 340.108 Linux Drivers
====================================

This repository provides scripts and patches for building the NVIDIA 340.108 drivers for
recent Linux (6.0+) kernel.

Most code patches were sourced from [If Not True Then
False](http://www.if-not-true-then-false.com). Additional modifications were made to allow
it to build with recent Linux kernels.

This is distributed as a patch only. The official driver installer will be downloaded from
the NVIDIA website if necessary.


Installation
------------

    git clone https://github.com/dkosmari/nvidia-340.108-updated.git
    cd nvidia-340.108-updated
    ./apply-patch.sh # this will download NVIDIA-Linux-x86_64-340.108.run

In order to install using DKMS (recommended):

    sudo make install

To just compile the module, without installing it:

    make


Post-Installation
-----------------

This will only install the kernel drivers. You still need to run the installer to ensure
the rest of the driver stack gets installed:

    sudo make post-install

Or manually:

    sudo ./NVIDIA-Linux-x86_64-340.108.run --no-kernel-module --no-x-check --no-check-for-alternate-installs

You might need to rerun this every time your Xorg package gets updated, since there's a
conflict with the GLX library.

Display / X Server Notes (CentOS Stream 10)
--------------------------------------------

CentOS Stream 10 does **not** ship a native Xorg server (`xorg-x11-server-Xorg`
was removed). GDM requires Wayland; Xwayland is available for legacy X11 apps.

### Firmware / GSP Warning

The legacy 340.108 driver does not include the GPU System Processor (GSP)
firmware required by the newer open-source `nvidia.ko`. The following NVRM
message in `dmesg` / `journalctl` is **expected and harmless** for the GeForce
8600 GT (G84):

    NVRM: The NVIDIA GPU ... is not supported ... because it does not include
    the required GPU System Processor (GSP).

The legacy DKMS module (`nvidia/340.108`) is what actually drives the card; the
open-source module simply refuses to probe it, which is correct behavior.

### Wayland Compositor (Weston)

Because GDM cannot fall back to a native Xorg session on CentOS Stream 10, a
lightweight Weston compositor is configured as a user-level systemd service:

    ~/.config/systemd/user/weston.service

It uses the software (pixman) renderer so no EGL/OpenGL support is required:

    ExecStart=/usr/bin/weston --backend=drm --renderer=pixman

Enable/start it with:

    loginctl enable-linger $USER
    systemctl --user enable --now weston.service

Weston will start automatically on login (and at boot if linger is enabled).
