---
layout: post
title: 在 WSL2 上安装 Archlinux 并运行 Emacs
categories: [Emacs, WSL2, Arch]
---
最近使用 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 安装 Emacs 的人越来越多，并且在 [Using Emacs on Windows with WSL2](https://emacsredux.com/blog/2020/09/23/using-emacs-on-windows-with-wsl2/) 这篇文章中了解到，Emacs 通过 [X410](https://x410.dev/) 可以方便地在 Windows 10 上运行 WSL2 的 GUI Apps 。于是萌生了安装 WSL2 体验 Emacs 的想法。  

### 在 WSL2 中安装 ArchLinux
#### 1. 启用WSL
用管理员打开 PowerShell 并输入
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```
#### 2. 升级为WSL2的必要条件
* 对于x64的系统要求win10版本为1903 或者更高
* win + R 输入 winver查看版本

#### 3. 启用虚拟平台
 用管理员打开 PowerShell 并输入
 ```powershell
 dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
 ```
#### 4. 下载Linux内核升级包
下载地址：https://wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi
下载完成后双击安装
#### 5. 将WSL2设置为默认版本
用管理员打开 PowerShell 输入
```powershell
wsl --set-default-version 2
```
到这里WSL就安装好了，下面安装ArchLinux

#### 6. 安装LxRunOffline
通过 Scoop 安装更加方便。也可以直接下载：https://github.com/DDoSolitary/LxRunOffline/releases 解压后将LxRunOffline.exe放入任意一个path文件夹下。
```powershell
scoop install LxRunOffline
```
#### 7. 下载Archlinux
下载地址：https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/latest/
找到 `archlinux-bootstrap-2020.10.01-x86_64.tar.gz`， 注意是 `tar.gz` 文件
#### 8. 安装archlinux到WSL

```powershell
LxRunOffline i -n ArchLinux -f C:\Users\Aqua\Downloads\archlinux-bootstrap-2021.03.01-x86_64.tar.gz -d C:\Users\Aqua\Linux -r root.x86_64
```
设置使用 WSL2:

```powershell
wsl --set-version ArchLinux 2
```

#### 9. 进入系统
```powershell
wsl -d ArchLinux
```
在Arch中执行
```bash
cd /etc/
explorer.exe .
```
注意后面的点，执行这条命令后会用windows的文件管理器打开/etc目录，然后找到pacman.conf，在这个文件最后加入
```
[archlinuxcn]
Server = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```
然后进入下一级目录pacman.d,编辑里面的mirrolist文件，将China的源注释去掉（选择部分即可）
然后回到Arch，执行
```bash
pacman -Syy
pacman-key --init
pacman-key --populate
pacman -S archlinuxcn-keyring
pacman -S base base-devel vim git wget
```

然后别忘了给当前的root设置密码
```bash
passwd
```
新建一个普通用户
```bash
useradd -m -G wheel -s /bin/bash arch
passwd arch
```

将文件`/etc/sudoers`中的`wheel ALL=(ALL) ALL`那一行前面的注释去掉
```bash
vim /etc/sudoers
```
查看当前用户id
```bash
id -u arch
```
#### 10. 设置使用普通用户登录Archlinux
紧接上一步，退出Arch
```bash
exit
```
在 PowerShell 中执行
```powershell
lxrunoffline su -n ArchLinux -v arch
```
到这里 ArchLinux 在 WSL 上的安装就结束了。

### 安装 Emacs
推荐直接安装 GccEmacs:
```bash
sudo pacman -S emacs-native-comp-git
```
由于 WSL2 是使用虚拟机的方式，每次重启系统都会变更主机 IP, 需要通过下面的方法获取 IP.
将下面的命令加入`.bashrc`:
```
export DISPLAY=$(cat /etc/resolv.conf | grep nameserver | awk '{print $2; exit;}'):0.0
```
然后执行以下命令应用环境：
 ```
 source ~/.bashrc
 ```
### 启动 Emacs
首先在 Windows 10 中启动 X410, 并在右下角的 X410 图标中打开`Allow Public Access` 选项。在弹出的防火墙对话框中点击允许。
然后在 WSL 终端中执行`emacs &`, 即可打开 GUI 的 Emacs 。

### 存在的问题
* 在高分辨率的情况下，显示模糊。
可以通过设置 Emacs 的大号字体解决。
* Emacs 中无法使用 `C-x` 按键。

### 后记
Mircosoft 以后将直接支持 Gui Apps, 不需要通过 Xserver 实现， 参考[这个文章](https://devblogs.microsoft.com/commandline/whats-new-in-the-windows-subsystem-for-linux-september-2020/#gui-apps).
