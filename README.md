add the following to /etc/apt/sources.list

I use nano:

nano -w /etc/apt/sources.list

paste in

#deb http://deb.debian.org/debian/ bookworm-backports contrib main non-free non-free-firmware
deb-src http://ftp.us.debian.org/debian bookworm main contrib non-free non-free-firmware

keep the bookworm-backports commented out while you setup the build evironment for qemu because you don't want to get in a dependency loop with held back packages, etc.
after building, you can enable the bookworm-backports repo.
This will also update several installed packages.


When you're done with the build - you will want to update the mesa-va-drivers



apt update
apt install aptitude ##use aptitude to solve any dependency issues that arise like librados-dev
apt build-dep qemu virglrenderer

apt install git build-essential

#### install the remaining packages needed for qemu build

apt install libacl1-dev libattr1-dev libglusterfs-dev libpci-dev libproxmox-backup-qemu0-dev libsdl1.2-dev libsystemd-dev python3-venv quilt xfslibs-dev lintian

cd /usr/src

git clone https://gitlab.freedesktop.org/virgl/virglrenderer.git

cd virglrenderer

meson build -Dvenus=true -Dvideo=True -Dprefix=/usr

If you have an amd gpu, you can add -Ddrm-renderers=amdgpu-experimental to the end of the meson build line

cd build

ninja install


Now you need to get the proxmox qemu source

cd /usr/src

git clone https://github.com/proxmox/pve-qemu

cd pve-qemu

git submodule update --init --recursive

cd qemu

meson subprojects download
###########################################


Patch the source from inside the pve-qemu/qemu directory

Patch file can be found here:

https://raw.githubusercontent.com/m...471b5bcab41842a5fe6ea9b05a09cb78f/virgl.patch


Just wget it to the qemu directory.

root@enyo:/usr/src/pve-qemu/qemu# patch < virgl.patch
can't find file to patch at input line 5
Perhaps you should have used the -p or --strip option?
The text leading up to this was:
--------------------------
|diff --git a/hw/display/virtio-gpu-virgl.c b/hw/display/virtio-gpu-virgl.c
|index 145a0b3879..2fee481219 100644
|--- a/hw/display/virtio-gpu-virgl.c
|+++ b/hw/display/virtio-gpu-virgl.c
--------------------------
File to patch: hw/display/virtio-gpu-virgl.c #type this in the prompt
patching file hw/display/virtio-gpu-virgl.c
patching file meson.build
root@enyo:/usr/src/pve-qemu/qemu#

Now you will cd .. and run make deb

cd ..
make deb

dpkg -i pve-qemu-kvm_9.2.0-3_amd64.deb
note: your file name may be newer than the one above.

