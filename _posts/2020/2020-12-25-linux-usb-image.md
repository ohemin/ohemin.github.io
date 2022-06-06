---
categories: [Linux]
---

做物联网相关项目时，经常需要在众多同类主机上安装Linux系统，并配置上一系列软件，手动一台台安装又慢又容易出错，此时可采用USB设备安装方案，具体做法是使用量产已安装好Linux系统及软件的U盘，怼到主机上就可以运行了，先按照需要制作好Linux镜像，然后将镜像写入USB盘，最后将USB盘插上主机USB口设置为USB引导开机。

## Linux发行版选择

选定Linux发行版，在主机上进行系统安装和软件功能测试，确保系统功能兼容和软件可用性，完成测试后就可以选定发行版了。

下面以ArchLinux发行版为例

## 编写调度安装脚本

1. 在开发机上安装VMWare或Virtualbox虚拟机软件
2. 下载ArchLinux发行版iso镜像文件
3. 创建一个Linux虚拟机并设置为iso镜像启动
4. 准备1个8G以上U盘（或者USB移动硬盘）空盘，容量大小视要安装的系统及软件大小而定，16G以上容量更为稳妥
5. 将U盘接入宿主机，并设置虚拟机对USB设备的访问，VMWare 或 Virtualbox有不同的设置方式
6. 启动虚拟机从光驱镜像进入ArchLinux系统
7. 编写在USB设备上安装Linux系统的初始化脚本

```bash
#!/usr/bin/bash
#1.Network Test
timedatectl set-ntp true
dhcpcd

echo "ArchLinux to USB Flash Disk Auto Install"
echo "Ver:1.0"
echo "github.com/multichian/arch-usb-installer"

#2.Disk Partition
lsblk
read -t 5 -p "Select Your USB Disk (Default 'sdb'):" DISK
DISK=${DISK:-'sdb'}
read -t 5 -p "Storage Size(Default 200M):" STORAGE
STORAGE=${STORAGE:-200}
read -t 5 -p "EFI Size(Default 100M):" EFI
EFI=${EFI:-100}
OS=`expr 8000 - $STORAGE - $EFI`
echo "OS Size:" $OS
read -t 5 -p "Are you sure you choice is correct? Enter N to restart: " AX

if [[ ${AX} = N ]]; then
  exit
fi

fdisk /dev/${DISK} <<EOF
  d
  o
  n
  p


  +${EFI}M
  n
  p


  +${STORAGE}M
  n
  p


  +${OS}M
  wq

EOF
echo "Partition USB Disk Done"

#3.Wipe And Mount Disk
mkfs.vfat /dev/${DISK}1 -n EFI
mkfs.vfat /dev/${DISK}2 -n MCOS
mkfs.ext4 -O "^has_journal" /dev/${DISK}3
mount /dev/${DISK}3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/${DISK}1 /mnt/boot/efi
echo "Wipe And Mount USB Disk Done"


#4.Using China Mirrorlist
mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
#echo -e "https://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch\n"
echo "Server=https://mirrors.ustc.edu.cn/archlinux/\$repo/os/\$arch" >> /etc/pacman.d/mirrorlist
echo "Server=https://mirrors.aliyun.com/archlinux/\$repo/os/\$arch" >> /etc/pacman.d/mirrorlist
#echo -e "\n## China mirrors\nhttps://mirrors.ustc.edu.cn/archlinux/\$repo/os/\$arch\nhttps://mirrors.163.com/archlinux/\$repo/os/\$arch" >> mirrorlist
echo "Change China Source Done"

#5.Install System
#pacstrap /mnt base base-devel linux linux-firmware dhcpcd ntfs-3g dialog vim wireless_tools wpa_supplicant net-tools dosfstools openssh
pacstrap /mnt base base-devel linux linux-firmware dhcpcd vim net-tools sudo grub efibootmgr
echo "Install Package Done"

#6.FSTAB
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

#7.Enter New System
arch-chroot /mnt
```
{: file="ArchInstall.sh"}

执行完成该脚本后，USB设备上已经安装好Arch Linux基本系统，下一步需要做一些引导配置、用户密码及自动登录设置，因为这些需要在已经安装好的系统上执行，因此需要再编写一个脚本
```bash
#8.Locale and Lang
echo -e "en_US.UTF-8 UTF-8\nzh_CN.UTF-8 UTF-8\n" > /etc/locale.gen
locale-gen
echo -e "LANG=en_US.UTF-8\nLC_TYPE=en_US.UTF-8" > /etc/locale.conf
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc --localtime

#9.Machine info
echo "MCOS" > /etc/hostname
echo -e '127.0.0.1 localhost\n::1       localhost\n127.0.1.1 mcos.localdomain mcos' >> /etc/hosts
sed -i '/^HOOKS/cHOOKS=(base udev block keyboard autodetect modconf filesystems fsck)' /etc/mkinitcpio.conf
mkinitcpio -P
systemctl enable dhcpcd
systemctl enable sshd
echo "Create New initramfs Done"


#10.Install GRUB
echo "Install Grub"
lsblk
read -p "Select Your USB Disk:" DISK
grub-install --target=i386-pc /dev/${DISK}
grub-install --target=x86_64-efi --efi-directory=/boot/efi --removable --recheck
grub-mkconfig -o /boot/grub/grub.cfg
cp -v /usr/share/grub/{unicode.pf2,ascii.pf2} /boot/grub/
cp -v /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
echo "Install Grub Done"

#11.Setup Autologin
echo "Autologin"
mkdir /etc/systemd/system/getty@tty1.service.d/
cd /etc/systemd/system/getty@tty1.service.d/
echo -e "[Service]\nExecStart=\nExecStart=-/usr/bin/agetty --autologin miner --noclear %I 38400 linux" > autologin.conf

#12. Initialize User and password
echo "Initialize the Users"
echo "Set Root passwd:"
passwd
useradd -m -G wheel,audio,video,optical,storage miner
echo "Set miner passwd:"
passwd miner
read -p "Do you want VISUDO? Enter Y or N" VX
if [[ ${VX} = Y ]]; then
    visudo
fi
## %wheel ALL=(ALL) ALL
## $wheel ALL=(ALL) NOPASSWD: ALL
echo "User initialization is complete"

read -p "Do you want to poweroff? Enter Y to shutdown:" CX
if [[ ${CX} = Y ]]; then
    poweroff
else
    exit
fi
```
{: file="ArchInstall2.sh"}

到这一步可以USB启动系统并登录了，还有一此基于用户的配置和软件安装，这一步可以自主定制，比如安装业务软件什么的...
```bash
#13. Install Openbox
# 需要在登录用户下操作
sudo pacman -Sy xorg-server xorg-xinit xterm openbox 
# 安装字体
sudo pacman -S noto-fonts-cjk python python-pip
# 安装扩展字体
sudo pacman -S ttf-dejavu ttf-liberation wqy-zenhei ttf-arphic-ukai ttf-arphic-uming 
mkdir -p ~/.config/openbox
cp /etc/xdg/openbox/* ~/.config/openbox/
echo "xterm -hold -e cal" >> ~/.config/openbox/autostart
echo "exec openbox-session" > ~/.xinitrc

echo "#Auto startx"
echo "if [ -z \"\$DISPLAY\" ] && [ -n \"\$XDG_VTNR\" ] && [ \"\$XDG_VTNR\" -eq 1 ]; then"
echo "  exec startx"
echo "fi"

#14. Install environment
sudo pacman -S python python-pip
pip install psutil GPUtil tabulate
```
{: file="SoftInstall.sh"}

一切安装完毕后，将USB盘内容制成img磁盘镜像，Mac/Linux系统中可以用dd命令或者安装镜像制作软件
```bash
dd if=/dev/sdb1 of=./archlinux.img bs=10m
```

然后换上其它空U盘均可以用这个img文件写入，因为U盘写入速度较慢，可以几个USB口插满U盘然后多条命令并行写入，大大缩短制作时间

到这一步，可能有人会说：__"也可以不编写脚本啊，我只要在虚拟机里把系统制作好，再制成镜像就行。"__ 没错！制作脚本的目的主要是为了方便维护和分发，从脚本就可以很清楚知道在系统基础上做了哪些设置和封装，后续要更改集成的软件，或者升级之类只需要修改脚本就行，不需要一直虚拟机镜像和img文件，分发文件体积会大大缩小
