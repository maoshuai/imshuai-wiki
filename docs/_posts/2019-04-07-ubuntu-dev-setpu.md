---
title: Ubuntu开发机配置
tags: Ubuntu Linux
---


拿到了一台新笔记本，决定将其定制为Linux环境的主力开发机，下面记录一下主要配置和安装。后续会持更新和补充。

<!--more-->

# 规划目录

在`~`目录下创建一个目录`~/g`，算是工作目录。然后创建一个dev_tools放置一些开发工具。

```shell
mkdir -p ~/g/dev_tools
```

创建github本地目录，用于clone自己github仓库
```shell
mkdir -p ~/g/maoshuai.github
```

# 配置profile文件

在~下创建单独的profile文件，比如.ms_profile：

```shell
touch ~/.ms_profile
```

然后在shell的profile文件里加载：

```shell
# self-defined profile
if [ -f ~/.ms_profile ];then
    . ~/.ms_profile
fi
```

这样，可以讲自己定义的一些配置写在.ms_profile中，方便管理。

# 安装开发工具

## 安装vim
ubuntu默认自带的并不是完整版的vim，而是vim-lite，功能有缩水，所以需手工安装完整版vim

```shell
sudo apt install vim -y
```

## 安装git

```shell
sudo apt install git -y
```
## 安装Oracle JDK
官网下载JDK的tar.gz文件，解压到目录dev_tools目录下。然后通过注册到alternatives

```shell
sudo update-alternatives --install /usr/bin/java java /home/maoshuai/g/it/dev_tools/jdk1.8.0_201/bin/java 1
```
执行java -version确认已配置成功。

然后在.ms_profile中配置JAVA_HOME：

```shell
# config JAVA_HOME
JAVA_HOME=/home/maoshuai/g/it/dev_tools/jdk1.8.0_201
```

## 安装maven

官网下载maven的tar.gz文件，解压到dev_tools目录下，然后配置到.ms_profile中：

```shell
# config maven home
M2_HOME=~/g/it/dev_tools/apache-maven-3.6.0/
PATH=$PATH:$M2_HOME/bin
```

## 安装eclipse

官网直接下载eclipse的tar.gz文件，解压到dev_tools目录下。为了方便后续使用，需要将eclipse添加到launcher中。具体方法如下：
在目录~/.local/share/applications添加文件eclipse.desktop，内容如下（注意替换Icon和Exec的路径为实际安装路径）：

```
[Desktop Entry]
Type=Application
Name=Eclipse
Comment=Eclipse Integrated Development Environment
Icon=** something like /opt/eclipse/icon.xpm **
Exec= ** something like /opt/eclipse/eclipse **
Terminal=false
Categories=Development;IDE;Java;
StartupWMClass=Eclipse
```

参考：<https://askubuntu.com/questions/80013/how-to-pin-eclipse-to-the-unity-launcher>

## 安装 Intellij Idea

官网下载.tar.gz文件，解压到/dev_tools目录。

## 安装sublime

可以直接通过Ubuntu Software应用市场安装。

## 安装docker

参考docker官网的步骤即可：<https://docs.docker.com/install/linux/docker-ce/ubuntu/>

配置非root用户使用（即添加到docker用户组）：

```shell
sudo usermod -aG docker <userName>
```
重启机器。

## 安装net-tools

安装net-tools后，可以使用netstat、ifconfig等命令：

```shell
sudo apt install net-tools -y
```

## 安装typora

参考官网的安装命令。

## 安装Chrome

官网下载deb文件安装。

## 安装Dropbox

官网下载deb文件安装。

## 安装Virtualbox

官网下载deb文件安装。

## 安装Jekyll

首先安装ruby环境：

```shell
sudo apt-get install ruby-full build-essential zlib1g-dev
```

添加如下配置到.ms_profile，使得gems的安装目录放在~/gems下

```shell
# config ruby 
# Install Ruby Gems to ~/gems
export GEM_HOME="$HOME/gems"
export PATH="$HOME/gems/bin:$PATH"
```

然后安装jekyll

```shell
gem install jekyll bundler
```
参考：<https://jekyllrb.com/docs/installation/>




# 配置偏好

## 中文和输入法

首先打开Language Support，点击Install/Remove Languages，将简体中文添加进来。然后注销重新登录。

重新登录后，打开settings，进入Region & Language，点击Input Sources添加Cinese输入法。

参考：

<https://www.pinyinjoe.com/linux/ubuntu-18-gnome-chinese-setup.htm>

<https://www.pinyinjoe.com/linux/ubuntu-18-gnome-chinese-setup.htm>

## 配置termial

### 修改光标为竖线
Edit->Preference->Text->Cursor->Cursor shape，选择I-Beam

## 配置命令行为vi风格

在.ms_profile里增加如下内容：

```shell
set -o vi
```

## 关闭Ubuntu动画效果

ubuntu的gnome桌面动画太拖沓，尤其是打开菜单，图标一个一个的划过，等待时间太长。通过安装gnome-tweak-tool可以关闭动画效果。

```shell
sudo apt install gnome-tweak-tool -y
```

打开gnome-tweak-tool，点击Apperance->Animations，切换为关闭。

## dock 点击图标最小化窗口

```shell
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```

## 使用windows风格的任务栏，合并top bar和任务栏
安装如下插件，注销重新登录后打开gnome-tweak-tool，打开extension也签根据需要调整。
```
sudo apt install \
gnome-tweak-tool \
gnome-shell-extension-dash-to-panel \ # （变成传统样式的关键扩展，第一底部面板全集成）
gnome-shell-extensions \常用扩展合集）
gnome-shell-extension-top-icons-plus #（托盘图标显示扩展）
```

# 其他

## 挂载windows分区

如果电脑上有windows分区，为了共享文件，可以将其挂在到ubuntu下。

首先查看windows分区的设备名：

```shell
sudo fdisk -l 
```

输出类似如下：

```
evice             Start       End   Sectors   Size Type
/dev/nvme0n1p1      2048    206847    204800   100M EFI System
/dev/nvme0n1p2    206848    468991    262144   128M Microsoft reserved
/dev/nvme0n1p3    468992 250035659 249566668   119G Microsoft basic data
/dev/nvme0n1p4 250036224 500117503 250081280 119.3G Linux filesystem
```

根据名称和分区大小等信息，确定Windows分区，比如上图是/dev/nvme0n1p3 

### 临时挂载：

创建一个挂载目录，一般可以在/mnt下：

```shell

sudo mkdir -p /mnt/win
```

然后挂载windows设备（-o rw是可写）：

```shell
sudo mount -t ntfs -o rw /dev/nvme0n1p3 /mnt/win/
```

### 启动后自动挂载

上面的方式重启后挂载点自动消失，适合临时使用windows分区的数据，也可以配置启动后自动挂载。

## 外接显示器

### 选择投屏方式

外接显示器，可通过Settings->Devices->Displays选择Dispaly Mode

### HDIMI外接显示器音频

HDMI外接电视或带音频的显示器，可以通过Settings->Sound里的Output选择声音输出设备到显示器

参考： <https://itsfoss.com/how-to-fix-no-sound-through-hdmi-in-external-monitor-in-ubuntu/>



# 待续

- [ ] 配置vim
- [ ] 安装zsh
- [ ] 安装python
- [ ] 安装emacs
- [ ] 配置常用字体
- [ ] 安装Wine
- [ ] ubuntu hibernate
- [ ] 安装wps

