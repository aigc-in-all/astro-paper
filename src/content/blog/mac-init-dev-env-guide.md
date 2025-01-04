---
title: "Mac 初始化开发环境搭建指南"
pubDatetime: 2024-10-26T15:57:10.982Z
featured: false
draft: false
tags:
  - mac
  - env
description: Mac 初始化环境搭建，命令工具、常用软件等
---

## Table of contents

## 命令行工具安装

### 1. 安装 Xcode Command Tools

> 可以不用先安装 Xcode,只安装 Command Tools

```bash
xcode-select --install
```
验证是否安装成功:
```bash
gcc --version
```

### 2. 安装 Homebrew

> *Homebrew* 是 *macOS* 上最流行的包管理器。它可以让你轻松地安装、更新和管理各种软件和开发工具.以后的命令行工具(比如 `nvm` )尽量使用 `brew` 安装,这样更加的简单和标准化.

官网: https://brew.sh/zh-cn/

安装命令:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后,注意输出的提示,按提示设置环境变量,下面是示例:
```bash
echo >> /Users/heqingbao/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> /Users/heqingbao/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

> *.zprofile* 文件是 *zsh* 的启动文件之一，类似于 *bash* 的 *.bash_profile*。*zsh* 在登录时会自动读取这个文件.

### 3. 安装zsh

确认 zsh 是否已安装: (默认系统是有安装的)
```bash
zsh --version
```

如果没有,可以使用brew安装:
```bash
brew install zsh
```

将 zsh 设置为默认 shell：
```bash
chsh -s /bin/zsh
```

退出当前终端,再打开新终端,检查是否切换完成:
```bash
echo $SHELL
```
> 输出 `/bin/zsh` 就表示成功了

安装 Oh My Zsh（这是一个流行的 zsh 配置框架，提供了很多有用的功能和主题）：

> 官网: https://ohmyz.sh/

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

安装完成后，Oh My Zsh 会自动更新你的 `.zshrc` 文件,你可以编辑 .zshrc 文件来自定义 zsh.

常用的配置包括：

* 更改主题：找到 `ZSH_THEME` 行，修改为你喜欢的主题，如 `ZSH_THEME="robbyrussell"`
* 添加插件：找到 `plugins=` 行，添加你需要的插件，如 `plugins=(git docker kubectl)`

### 4. 安装 zsh-autosuggestions

> 官网：https://github.com/zsh-users/zsh-autosuggestions/tree/master

```bash
brew install zsh-autosuggestions
```
> 注意看输出，有说要怎么添加到环境变量

在 `.zshrc` 添加：
```
# zsh-autosuggestions
source /opt/homebrew/share/zsh-autosuggestions/zsh-autosuggestions.zsh
```

### 5. 安装 NVM

> 官网:https://github.com/nvm-sh/nvm

使用 Homebrew 安装 NVM：
```bash
brew install nvm
```

创建 NVM 的工作目录：
```bash
mkdir ~/.nvm
```

配置 NVM 环境变量。我们需要编辑 .zshrc 文件来添加 NVM 的配置。运行以下命令：
```bash
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/opt/homebrew/opt/nvm/etc/bash_completion.d/nvm"' >> ~/.zshrc
```

使更改生效，重新加载 .zshrc 文件：

```bash
source ~/.zshrc
```

验证 NVM 安装：
```bash
nvm --version
```

安装 Node.js。你可以安装最新的长期支持版（LTS）：
```bash
nvm install --lts
```

或者安装特定版本：
```bash
nvm install 14.17.0  # 例如安装 14.17.0 版本
```

设置默认的 Node.js 版本：
```bash
nvm alias default node  # 设置最新安装的版本为默认版本
```

验证 Node.js 安装：
```bash
node --version
npm --version
```

设置淘宝镜像源：
```bash
# 设置
npm config set registry https://registry.npmmirror.com
# 验证
npm config get registry
```

### 6. 安装 pnpm

使用 Homebrew 安装 pnpm：
```bash
brew install pnpm
```

安装完成后，你可以验证安装是否成功：
```bash
pnpm --version
```

### 7. 安装 Python

使用 Homebrew 安装 Python 3:
```bash
brew install python
```
> 这会安装最新版本的 Python 3。

验证安装:
```bash
python3 --version
```
> 这应该显示安装的 Python 3 版本。

安装 virtualenvwrapper:
```bash
brew install virtualenvwrapper
```
> 注意,Python 3.11 及以后的版本引入了一个新的安全特性，防止用户意外地修改系统 Python 环境,所以 `pip3 install virtualenvwrapper`会失败

配置 virtualenvwrapper。在 .zshrc 文件末尾添加:
```bash
source virtualenvwrapper.sh
```
> 在使用 brew 安装完后,命令行有提示怎么配置环境变量. 不需要配置 WORKON_HOME ,因为虚拟环境默认就在 `~/.virtualenvs`

重新加载 .zshrc 文件:
```bash
source ~/.zshrc
```

创建一个新的虚拟环境:
```bash
mkvirtualenv test_env
```
> 这会创建并激活名为 "test_env" 的虚拟环境。

验证虚拟环境:
```bash
which python
```
> 这应该显示虚拟环境中 Python 的路径。

测试虚拟环境:
```bash
python --version
```
> 这应该显示你安装的 Python 3 版本。

退出虚拟环境:
```bash
deactivate
```

再次进入虚拟环境:
```bash
workon test_env
```

#### 其它命令:

列出所有虚拟环境：
```bash
workon

或
lsvirtualenv
```

创建新的虚拟环境：
```bash
mkvirtualenv env_name

或 指定 python 版本
mkvirtualenv -p python3.8 env_name
```

切换到特定的虚拟环境：
```bash
workon env_name
```

退出当前虚拟环境：
```bash
deactivate
```

删除虚拟环境：
```bash
rmvirtualenv env_name
```

复制虚拟环境：
```bash
cpvirtualenv source_env target_env
```

显示当前虚拟环境的路径：
```bash
cdvirtualenv
```

进入虚拟环境的 site-packages 目录：
```bash
cdsitepackages
```

列出虚拟环境中安装的包：
```bash
lssitepackages
```

在所有虚拟环境中执行命令：
```bash
allvirtualenv command_to_run
```

为虚拟环境添加工作目录：
```bash
setvirtualenvproject
```

切换到虚拟环境的项目目录：
```bash
cdproject
```

显示虚拟环境的详细信息：
```bash
showvirtualenv env_name
```

## 常用软件

作为一个开发者，有几个常用的工具可能对你有帮助。我会列出一些，并简要解释它们的用途。你可以根据自己的需求来决定是否安装：

Git：版本控制系统（通常已经随 Xcode Command Line Tools 安装）
> 如果没有安装,可以使用 `brew install git` 安装

Visual Studio Code：代码编辑器
> 官网: https://code.visualstudio.com/

iTerm2：终端模拟器
> 官网: https://iterm2.com/

Sublime Merge: Git客户端
> 官网: https://www.sublimemerge.com/

Sublime Text: 文本编辑器
> 官网: https://www.sublimetext.com/

Snipaste: 截图工具
> 官网: https://www.snipaste.com/

## 其它

### 安装 Jetbrains Mono 字体

JetBrains Mono 是一款优秀的编程字体。在 Mac 上安装 JetBrains Mono 字体有几种方法，我会介绍最简单的两种：

方法 1：使用 Homebrew（推荐）
```bash
brew install --cask font-jetbrains-mono
```

方法 2：手动下载安装

1. 访问 JetBrains Mono 的官方网站：https://www.jetbrains.com/lp/mono/

2. 下载后解压,然后双击每个 .ttf 文件，或者选择所有 .ttf 文件并双击进行安装.
