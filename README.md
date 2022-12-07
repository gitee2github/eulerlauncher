<img src="./logos/logo-slogan.png" width="70%" height="70%" />

----

**OmniVirt**是由openEuler社区技术运营团队及基础设施团队孵化的开发者工具集，通过对主流桌面操作系统中的虚拟化技术(LXD、HyperV、Virtualization framework)等技术进行有机整合，使用openEuler社区官方发布的虚拟机、容器镜像，为开发者在Windows、MacOS、Linux上提供统一的开发资源(虚拟机、容器)发放和管理体验，提升主流桌面操作系统上openEuler开发环境使用的便利性和稳定性，有效提升开发者体验。

## 背景 & 简介

主流桌面操作系统上提供相关开发资源(虚拟机、容器等)的便利性和稳定性是影响openEuler开发者体验的重要因素，尤其是对于对开发资源受限的个人及高校开发者openEuler开发体验影响更为明显。当前常见的虚拟机管理平台有诸多局限性，如VirtualBox需要下载体积庞大的ISO镜像，同时需要进行操作系统安装等相关操作，WSL无法提供真实的openEuler内核，绝大多数虚拟机管理软件目前对Apple Sillicon芯片支持尚不完善且众多软件需要付费等，这些都极大的降低了开发者的工作效率。

**OmniVirt**支持在Windows、MacOS(规划中)及Linux(规划中)等主流桌面操作系统上提供方便、易用、统一体验的开发者工具集，硬件架构支持x86_64及Aarch64(包含Apple Sillicon系列芯片)；并支持各平台对应的虚拟化硬件加速能力，为开发者提供高性能的开发资源。**OmniVirt**支持使用openEuler社区发布的虚拟机、容器(规划中)镜像、openEuler社区提供的Daily Build镜像以及其他符合要求的自定义镜像，为开发者提供多种选择。

## OmniVirt使用指导
**OmniVirt**当前采用二进制发布的形式，使用时请下载对应架构的软件包。

### Windows下的安装与运行
**OmniVirt**当前支持Windows11/10，前往[OmniVirt最新版下载][1]下载Windows版软件包并解压到期望的位置。
右键点击 `config-env.bat` 并选择**以管理员身份运行**，该脚本将进行环境变量相关的配置，将当前目录添加到系统环境变量 `path`中，如果使用者掌握如何配置环境变量，或配置脚本出现问题，也可以进行手动配置，将当前脚本所在目录及 `qemu-img` 子目录添加至系统环境变量 `path` 中。

**OmniVirt**在Windows上运行需要对接 `Hyper-V` 虚拟化后端，`Hyper-V` 是 Microsoft 的硬件虚拟化产品，可以为Windows上的虚拟机提供更为出色的性能。在运行**OmniVirt**前，请先检查你的系统是否开启了 `Hyper-V`，具体检查及开启方法请参考[Hyper-V开启指导][2]或其他网络资源。

**OmniVirt**解压后包含以下几个部分：

- omnivirtd.exe：OmniVirt的主进程，是运行在后台的守护进程，负责与各类虚拟化后端交互，管理虚拟机、容器以及镜像的生命周期，omnivirtd.exe是运行在后台的守护进程。
- onivirt.exe：OmniVirt的CLI客户端，用户通过该客户端与omnivirtd守护进程交互，对虚拟机、镜像等进行相关操作。
- omnivirt-win.conf：OmniVirt配置文件，需与omnivirtd.exe放置于同一目录下，参考下面配置进行相应配置：

```Conf
[default]
# 配置日志文件的存储目录
log_dir = D:\omnivirt-workdir\logs
# 配置日志等级是否开启Debug
debug = True
# 配置OmniVirt的工作目录
work_dir = D:\omnivirt-workdir
# 配置OmniVirt的镜像目录，镜像目录为对工作目录的相对目录
image_dir = images
# 配置OmniVirt的虚拟机文件目录，虚拟机文件目录为对工作目录的相对目录
instance_dir = instances
```

配置完成后请右键点击omnivirtd.exe，选择以管理员身份运行，点击后omnivird.exe将以守护进程的形式在后台运行。

打开 `PowerShell` 或 `Terminal` ，准备进行对应的操作。

### Windows下退出omnivirtd后台进程

当omnivirtd.exe运行后，会在操作系统右下角托盘区域生成omnivirtd托盘图标：

<img src="./etc/images/tray-icon.png" width="10%" height="10%"/>
鼠标右键点击托盘图标，并选择 `Exit OmniVirt` 即可退出omnivirtd后台进程。

### 镜像操作

1. 获取可用镜像列表：
```PowerShell
omnivirt.exe images

+-----------+----------+--------------+
|   Images  | Location |    Status    |
+-----------+----------+--------------+
| 22.03-LTS |  Remote  | Downloadable |
|   21.09   |  Remote  | Downloadable |
| 2203-load |  Local   |    Ready     |
+-----------+----------+--------------+
```

**OmniVirt**镜像有两种位置属性：1）远端镜像 2）本地镜像，只有处于本地且状态为 `Ready` 的镜像可以直接用来创建虚拟机，位于远端的镜像需要下载后才能够使用；你也可以加载已经预先下载好的本地镜像到**OmniVirt**中，具体操作方法可以参考接下来的操作指导。

2. 下载远端镜像

```PowerShell
omnivirt.exe download-image 22.03-LTS

Downloading: 22.03-LTS, this might take a while, please check image status with "images" command.
```

镜像下载请求是一个异步请求，具体的下载动作将在后台完成，具体耗时与你的网络情况相关，整体的镜像下载流程包括下载、解压缩、格式转换等相关子流程，在下载过程中可以通过 `image` 命令随时查看下载进展与镜像状态：

```PowerShell
omnivirt.exe images

+-----------+----------+--------------+
|   Images  | Location |    Status    |
+-----------+----------+--------------+
| 22.03-LTS |  Remote  | Downloadable |
|   21.09   |  Remote  | Downloadable |
| 22.03-LTS |  Local   | Downloading  |
+-----------+----------+--------------+
```


当镜像状态转变为 `Ready` 时，表示镜像下载完成，处于 `Ready` 状态的镜像可被用来创建虚拟机：

```PowerShell
omnivirt.exe images

+-----------+----------+--------------+
|   Images  | Location |    Status    |
+-----------+----------+--------------+
| 22.03-LTS |  Remote  | Downloadable |
|   21.09   |  Remote  | Downloadable |
| 22.03-LTS |  Local   |    Ready     |
+-----------+----------+--------------+
```

3. 加载本地镜像

用户也可以加载自定义镜像或预先下载到本地的镜像到OmniVirt中用于创建自定义虚拟机：

```PowerShell
omnivirt.exe load-image --path {image_file_path} IMAGE_NAME
```

当前支持加载的镜像格式有 `xxx.qcow2.xz`，`xxx.qcow2`

例如：

```PowerShell
omnivirt.exe load-image --path D:\openEuler-22.03-LTS-x86_64.qcow2.xz 2203-load

Loading: 2203-load, this might take a while, please check image status with "images" command.
```

将位于 `D:\` 目录下的 `openEuler-22.03-LTS-x86_64.qcow2.xz` 加载到OmniVirt系统中，并命名为 `2203-load`，与下载命令一样，加载命令也是一个异步命令，用户需要用镜像列表命令查询镜像状态直到显示为 `Ready`, 但相对于直接下载镜像，加载镜像的速度会快很多：

```PowerShell
omnivirt.exe images

+-----------+----------+--------------+
|   Images  | Location |    Status    |
+-----------+----------+--------------+
| 22.03-LTS |  Remote  | Downloadable |
|   21.09   |  Remote  | Downloadable |
| 2203-load |  Local   |   Loading    |
+-----------+----------+--------------+

omnivirt images

+-----------+----------+--------------+
|   Images  | Location |    Status    |
+-----------+----------+--------------+
| 22.03-LTS |  Remote  | Downloadable |
|   21.09   |  Remote  | Downloadable |
| 2203-load |  Local   |     Ready    |
+-----------+----------+--------------+
```

4. 删除镜像：

通过下面的命令将镜像从OmniVirt系统中删除：

```PowerShell
omnivirt.exe delete-image 2203-load

Image: 2203-load has been successfully deleted.
```

### 虚拟机操作

1. 获取虚拟机列表：

```Powershell
omnivirt.exe list

+----------+-----------+---------+---------------+
|   Name   |   Image   |  State  |       IP      |
+----------+-----------+---------+---------------+
|   test1  | 2203-load | Running | 172.22.57.220 |
+----------+-----------+---------+---------------+
|   test2  | 2203-load | Running |      N/A      |
+----------+-----------+---------+---------------+
```

若虚拟机IP地址显示为 `N/A` ，若这台虚拟机的状态为 `Running` 则表示这台虚拟机为新创建的虚拟机，网络还未配置完成，网络配置过程大概需要若干秒，请稍后重新尝试获取相关虚拟机信息。

2. 登录虚拟机：

若虚拟机已成功分配到IP地址，可以直接使用 `SSH` 命令进行登录：

```PowerShell
ssh root@{instance_ip}
```
若使用的是openEuler社区提供的官方镜像，则默认用户为 `root` 默认密码为 `openEuler12#$`

3. 创建虚拟机

```PowerShell
omnivirt.exe launch --image {image_name} {instance_name}
```

通过 `--image` 指定镜像，同时指定虚拟机名称。

4. 删除虚拟机
```PowerShell
omnivirt.exe delete-instance {instance_name}
```
根据虚拟机名称删除指定的虚拟机。

[1]: https://gitee.com/openeuler/omnivirt/releases
[2]: https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v