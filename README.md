# Archlinux 安装记录
## 格式化硬盘
#### 最好格式化为msdos,否则os-prober不能正确检测到Windows引导
```bash
mkfs.ext4 /dev/nvme0n1p2
mkfs.msdos /dev/nvme0n1p1
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
```
#### 编辑`/etc/default/grub`文件，修改为`GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 nowatchdog"`，去掉 `GRUB_DISABLE_OS_PROBER=false`前面的 `#` 允许OS探测
```bash
nano /etc/default/grub
```
#### 生成 GRUB 配置文件
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
## 完成安装
```bash
exit
umount -R  /mnt
reboot  
```
## 安装yay助手和添加Archlinuxcn源
```bash
sudo nano /etc/pacman.conf
wget https://arch.moichi.cn/res/yay-bin.pkg.tar.zst
sudo pacman -U yay-bin.pkg.tar.zst
sudo nano /etc/pacman.conf
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
## 安装wayfire
```bash
git clone https://aur.archlinux.org/wayfire-git.git
```
#### 修改 PKGBUILD `provides=("${pkgname%-git}" 'wlroots-git' 'wf-config-git' 'wlroots' 'wf-config' )`
```bash
makepkg -si
```
## 安装wayfire扩展
```bash
yay -S wcm-git wayfire-plugins-extra-git
```
## yay安装常用软件
```bash
yay -Sy obs-studio wlrobs-hg microsoft-edge-stable-bin mpv greetd nerd-fonts-hack fuse #fuse是Joplin必备
```
## EDGE浏览器中文设置
```bash
sudo nano /opt/microsoft/msedge/microsoft-edge
export LANGUAGE=ZH-CN.UTF-8
```
## 安装ZSH和Powerlevel10k zsh-autosuggestions zsh-syntax-highlighting
```bash
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
## 安装GRUB主题
```bash
git clone https://github.com/vinceliuice/grub2-themes.git
```
## 重建Archlinux引导
#### 格式化 EFI 分区，挂载分区
```bash
mkfs.msdos /dev/nvme0n1p1
mount /dev/nvme0n1p2 /mnt
mount /dev/nvme0n1p1 /mnt/boot
```
#### 生成 /boot 目录下 `initramfs-linux-fallback.img` ` initramfs-linux.img` `vmlinuz-linux` `amd-ucode.img`文件
```bash
pacman -S linux amd-ucode
```
#### 生成引导
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/ --bootloader-id=Archlinux
grub-mkconfig -o /boot/grub/grub.cfg
```
#### 若遇到 GRUB 启动前 `error：file'/grub/locale/C.gmo' not found`错误，在`/etc/default/grub`中添加`LANG=C`
