# Just only me
## 没啥用
```bash
systemctl stop reflector.service
timedatectl set-ntp true
timedatectl status
```
## 格式化硬盘
```bash
mkfs.ext4 /dev/nvme0n1p2
mkfs.vfat /dev/nvme0n1p1
mount /dev/nvme0n1p2 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```
## 修改源
```bash
nano /etc/pacman.d/mirrorlist
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch
```
## 安装基础包
```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware  #base-devel在AUR包的安装是必须的
```
## 安装软件包
```bash 
pacstrap /mnt dhcpcd iwd nano zsh openssh htop ncdu wget neofetch lolcat bat kitty lsd bpytop tar gzip ranger alsa-utils pulseaudio pulseaudio-alsa pavucontrol noto-fonts-emoji noto-fonts-extra wqy-zenhei fcitx5-im fcitx5-chinese-addons fcitx5-material-color waybar swaylock wofi brightnessctl qt5-wayland python-pillow ntp gping 
```
## 生成 fstab 文件，切换系统，设置时区时间
```bash
genfstab -U /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
arch-chroot /mnt
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```
## 设置 Locale 进行本地化，设置主机名和hosts
```bash
tee /etc/locale.gen <<-`EOF`
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
`EOF`

locale-gen
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf

tee /etc/hostname <<-`EOF`
Archlinux
`EOF`

tee /etc/hosts <<-`EOF`
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux
`EOF`
```
## 创建root密码和新用户
```bash
passwd root
useradd -m avrileader
passwd avrileader
```
## 安装AMD微码和GRUB引导
```bash
pacman -S amd-ucode grub efibootmgr os-prober
grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=Archlinux
nano /etc/default/grub
grub-mkconfig -o /boot/grub/grub.cfg
```
## 完成安装
```bash
exit
umount -R  /mnt
reboot  
```
## 安装yay助手
```bash
sudo nano /etc/pacman.conf
wget https://arch.moichi.cn/res/yay-bin.pkg.tar.zst
sudo pacman -U yay-bin.pkg.tar.zst
```
## 安装wayfire
```bash
git clone https://aur.archlinux.org/wayfire-git.git
```
修改 PKGBUILD `provides=("${pkgname%-git}" 'wlroots-git' 'wf-config-git' 'wlroots' 'wf-config' )`
```bash
makepkg -si
```
## 安装wayfire扩展
```bash
yay -S wcm-git wayfire-plugins-extra-git
```
## yay安装常用软件
```bash
yay -Sy obs-studio wlrobs-hg microsoft-edge-stable-bin mpv greetd 
```
## 安装ZSH和Powerlevel10k
```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
```
## 安装GRUB主题
```bash
git clone https://github.com/vinceliuice/grub2-themes.git
```
