yay -S nvidia-390xx-dkms nvidia-390xx-utils lib32-nvidia-390xx-utils

sudo nano /etc/mkinitcpio.conf

# MODULES
# The following modules are loaded before any boot hooks are
# run.  Advanced users may wish to specify all system modules
# in this array.  For instance:
#     MODULES=(usbhid xhci_hcd)
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)
#MODULES=()

sudo mkinitcpio -P

sudo nano /etc/default/grub

GRUB_CMDLINE_LINUX_DEFAULT="... nomodeset nvidia-drm.fbdev=1 ..."

sudo nano /etc/pacman.d/hooks/nvidia.hook

[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia-390xx-dkms
Target=linux

[Action]
Description=Update Nvidia module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux) exit 0; esac; done; /usr/bin/mkinitcpio -P'


sudo nano /etc/X11/xorg.conf

# custom nvidia section by BeanGreen247
# forces X11 to run on nvidia gpu
# make sure to have only the discrete gpu enbled in bios of laptop
Section "Files"
    ModulePath "/usr/lib64/nvidia/xorg"
    ModulePath "/usr/lib64/xorg/modules"
EndSection

Section "Device"
    Identifier     "Nvidia Card"
    Driver         "nvidia"
    VendorName     "NVIDIA Corporation"
    Option "AllowEmptyInitialConfiguration" "true"
    Option "DRM" "true"
EndSection

Section "ServerFlags"
    Option "IgnoreABI" "1"
EndSection

[bean@thiccpad-t430 ~]$ lsmod | grep nvidia
nvidia_drm             65536  0
nvidia_uvm           2076672  0
nvidia_modeset       1343488  5 nvidia_drm
nvidia              19791872  200 nvidia_uvm,nvidia_modeset
video                  81920  2 nvidia,thinkpad_acpi
ipmi_msghandler        94208  2 ipmi_devintf,nvidia


