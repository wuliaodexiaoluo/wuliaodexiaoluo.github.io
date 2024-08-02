---
title: WSL 安装和 Systemd 配置
---

<link rel="stylesheet" href="https://sytle-cdn.luolingxue.us.kg/heimu-css/heimusty.css">

# WSL 安装和 Systemd 配置
本来今天是想用虚拟机配置一个 Git 方便我从任何地方同步我的代码

但是那漫长的安装过程属实是劝退了我

然后想起来许久没用的 WSL

## WSL 介绍（了解的可以跳过看下一步）
Windows Subsystem Linux，微软官方的定名是 **适用于 Linux 的 Windows 子系统** <span class="heimu" title="你知道的太多了">（微软式翻译）</span>

### WSL1 实现原理

WSL 1  基于软件层将 Linux 指令和 API 翻译成 Windows 指令和 API，来模拟 Linux 执行

#### 这种做法的优点

可以脱离虚拟化技术支持，没有 VBS 拉性能

#### 不过缺点也有

性能极差，没有 Systemctl，不方便配置 SSH ~(虽然低版本 WSL 2 也没有)~

### WSL2 实现原理

WSL 2 的实现依赖于硬件虚拟化~(本质上是个轻量级虚拟机)~

#### 这种做法的优点

提供了相对完整的 Linux 内核，和独立的文件系统，提高了运行效率

#### 缺点

要求硬件虚拟化，低版本还是用不了 Systemctl

# 安装部分

## 安装（已安装请跳过看升级和配置部分）

### 启动 VBS
按下 Win + X 键，选择 Windows Powershell （管理员）或命令提示符（管理员）
运行命令
```powershell
bcdedit /set hypervisorlaunchtype auto
```
### 安装 WSL 
打开控制面板，选择 **程序**

![84e4efe086a17b5537df997ac2cc947a.png](https://s2.loli.net/2024/07/16/8GvFlrKtJZdOuXo.png)

点击 **下面的启用或关闭 Windows 功能**
这里我们勾选两个功能
**适用于 Linux 的 Windows 子系统** 和 **虚拟机平台**
点击确定，等待配置完毕后重新启动

![84e4efe086a17b5537df997ac2cc947a.png](https://s2.loli.net/2024/07/16/8GvFlrKtJZdOuXo.png)

命令安装（自动重启，不想重启可以把 restart 改成 norestart）
```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /featurename:VirtualMachinePlatform /all /restart
```
## 检查版本
运行
```powershell
wsl --version
```
如果没有显示版本或报错或者版本低于如下显示版本
```text
WSL 版本： 2.2.4.0
内核版本： 5.15.153.1-2
WSLg 版本： 1.0.61
MSRDC 版本： 1.2.5326
Direct3D 版本： 1.611.1-81528511
DXCore 版本： 10.0.26091.1-240325-1447.ge-release
Windows 版本： 10.0.19045.4651
```
运行
```powershell
wsl --update
```
### 设置默认版本
```powershell
wsl --set-default-version 2
```
如果要给单独的发行版设置 WSL2 版本

```powershell
wsl --set-version 2 <填写发行版名称>
```
也可以切回去，把 2 改成 1 就行，但是非常耗时并可能导致**文件损坏**，切之**请务必三思**确认**是否有必要**

## 安装发行版

```powershell
wsl --list --online
```
列出可在线安装的发行版
安装（安装包地址可能有墙）
```powershell
wsl --install <发行版名称>
```
如果显示灾难性故障，可用管理员权限再运行一遍，或者检查下哪里漏步了

也可通过 Microsoft Stroe 安装

## 更换发行版安装位置
默认是安装在 C 盘，建议迁出避免 C 盘爆炸
### 导出
```powershell
wsl --export <发行版名称> <导出的文件位置>
```
压缩文件管理器打开，如果打不开就把后缀改成 vhdx
### 注销发行版
```powershell
wsl --unregister <发行版名称>
```
### 导入发行版
```powershell
wsl --import <指定发行版名称> <安装目录> <导出的发行版文件位置> --version2
```
vhdx 需要额外加入 --vhd 选项

### 删除文件（可选）
使用你最习惯的办法删掉文件
# 配置 Systemd
这一步非常简单(WSL 的 Vim 有点问题)
```bash
nano /ect/wsl.conf
```
然后输入
```conf
[boot]
systemd=true
```
保存，然后 powershell 运行命令
```powershell
wsl --shutdown
```
关闭发行版系统，然后重新打开

运行 systemctl 就不会报错了，service 也可用了
