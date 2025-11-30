# VirtualBox

# 1. linux安装增强驱动

- 手动卸载安装包
- 在虚拟机启动界面点击“设置-》安装增强”
- 在虚拟机中手动挂载`sudo mount /dev/sr0 /mnt`
- 安装增强```sudo sh VBoxLinuxAdditions.run

# 2. 解决系统安装后启动刚进入SHELL问题

SHELL环境中

```
\\> FS0:
\\> mkdir \\EFI\\boot
\\> cp \\EFI\\grub\\grubx64.efi \\EFI\\boot\\bootx64.efi

```