# rEFInd

[![License](https://img.shields.io/badge/license-GPL%203.0-blue.svg)](LICENSE)
[![rEFInd](https://img.shields.io/badge/rEFInd-0.14.2-purple.svg)](http://www.rodsbooks.com/refind/)
[![Platform](https://img.shields.io/badge/platform-UEFI%20(x64)%20|%20Linux%20|%20macOS%20|%20Windows-lightgrey.svg)](http://www.rodsbooks.com/refind/)
[![GitHub stars](https://img.shields.io/github/stars/techysy/rEFInd?style=social)](https://github.com/techysy/rEFInd/stargazers)

一款美观且功能强大的 **UEFI 启动管理器**，预配置多系统引导：**Windows 10**、**macOS**（基于 OpenCore）、**Ubuntu** 等。

![rEFInd 截图](rEFInd.jpg)

## 项目结构

```
rEFInd/
├── EFI/                      # 配置好的 rEFInd，引导 OC 和 WIN，使用 minimal 主题
│   ├── refind_x64.efi        # 主程序
│   ├── refind.conf           # 配置文件
├── refind-bin-0.14.2/        # 官方最新版本，包含驱动和安装脚本
├── refind-bin-0.11.4/        # 旧版本，保留供参考
├── Themes/                   # 主题包
└── Screenshot/               # 截图展示
```

## 相关资源

| 项目 | 说明 |
|------|------|
| [rEFInd 官网](http://www.rodsbooks.com/refind/) | 官方文档和教程 |
| [官方安装教程](http://www.rodsbooks.com/refind/installing.html) | 各平台安装指南 |
| [SourceForge 下载](https://sourceforge.net/projects/refind/) | 官方源码和二进制 |
| [rEFInd Minimal](https://evanpurkhiser.com/rEFInd-minimal/) | 简洁主题 |
| [ursamajor-rEFInd](https://github.com/kgoettler/ursamajor-rEFInd) | 另一款主题 |
| [更多主题](https://github.com/topics/refind-theme) | GitHub 主题合集 |

## 安装指南

### Linux 自动安装

```bash
cd refind-bin-0.14.2
sudo ./refind-install
```

可选参数：

```bash
# 使用 Secure Boot
sudo ./refind-install --shim /boot/efi/EFI/fedora/shimx64.efi

# 使用自定义密钥
sudo ./refind-install --localkeys

# 安装到指定位置
sudo ./refind-install --root /mountpoint
```

### Linux 手动安装

```bash
# 1. 复制文件
sudo cp -r refind-bin-0.14.2/refind /boot/efi/EFI/
# 2. 清理多余架构
sudo rm /boot/efi/EFI/refind/refind_ia32.efi /boot/efi/EFI/refind/refind_aa64.efi
sudo rm -r /boot/efi/EFI/refind/drivers_ia32 /boot/efi/EFI/refind/drivers_aa64
# 3. 重命名配置
sudo mv /boot/efi/EFI/refind/refind.conf-sample /boot/efi/EFI/refind/refind.conf
# 4. 添加启动项
sudo efibootmgr -c -l \\\\EFI\\\\refind\\\\refind_x64.efi -L rEFInd
```

### macOS 安装

> macOS 10.11+ 需要先禁用 SIP

```bash
# 进入恢复模式 → 终端 → csrutil disable
# 重启后安装
cd refind-bin-0.14.2
./refind-install
```

手动安装：

```bash
sudo mkdir -p /Volumes/ESP
sudo mount -t msdos /dev/disk0s1 /Volumes/ESP
sudo cp -r refind-bin-0.14.2/refind /Volumes/ESP/EFI/
sudo bless --mount /Volumes/ESP --setBoot --file /Volumes/ESP/EFI/refind/refind_x64.efi --shortform
```

### Windows 安装

以管理员身份运行命令提示符：

```cmd
mountvol R: /S
xcopy /E refind-bin-0.14.2\refind R:\EFI\refind\
R:
cd EFI\refind
del refind_ia32.efi refind_aa64.efi
ren refind.conf-sample refind.conf
bcdedit /set "{bootmgr}" path \EFI\refind\refind_x64.efi
bcdedit /set "{bootmgr}" description "rEFInd Boot Manager"
```

或使用 [EasyUEFI](http://www.easyuefi.com/) 图形界面工具。

## 卸载

| 平台 | 命令 |
|:----|:------|
| Linux | `sudo rm -r /boot/efi/EFI/refind && efibootmgr -b X -B` |
| macOS | `sudo rm -r /Volumes/ESP/EFI/refind && sudo bless --mount /Volumes/Macintosh\ HD --setBoot` |
| Windows | EasyUEFI 删除启动项 + 删除 `EFI\refind\` 目录 |

## 配置示例 (`refind.conf`)

```conf
screensaver 30
timeout 2

# 屏蔽自动搜索
dont_scan_files \EFI\Microsoft\Boot\bootmgfw.efi
dont_scan_dirs /EFI/BOOT,/EFI/Microsoft,/EFI/OC,/EFI/ubuntu

# 手动添加启动项
menuentry Ubuntu {
    volume "Ubuntu 18.04.3 LTS"
    loader \EFI\ubuntu\grubx64.efi
    icon \EFI\refind\themes\rEFInd-minimal\icons\os_ubuntu.png
}

menuentry "Windows 10" {
    loader \EFI\Microsoft\Boot\bootmgfw.efi
    icon \EFI\refind\themes\rEFInd-minimal\icons\os_win.png
}

menuentry "macOS 10.15.2" {
    volume "macOS boot"
    loader \EFI\OC\OpenCore.efi
    icon \EFI\refind\themes\rEFInd-minimal\icons\os_mac.png
}

include themes\rEFInd-minimal\theme.conf
```

## License

GPL 3.0
