title: Ubuntu前端开发环境搭建
author: wecash FE team
date: 2016-07-28 05:55:10
tags:
---
# Ubuntu前端开发环境搭建

# chrome

  [下载Chrome](https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb)

    sudo apt install libappindicator1 libindicator7
    sudo dpkg -i google-chrome-stable_current_amd64.deb

  Chrome所需插件

  [安装React](https://github.com/facebook/react-devtools/releases)

  [安装Vue](https://github.com/vuejs/vue-devtools)

  [安装Postman](http://www.getpostman.com/)

# input method

    sudo add-apt-repository ppa:fcitx-team/nightly
    sudo apt-get update
    sudo apt-get install fcitx fcitx-config-gtk fcitx-googlepinyin
    img-config // set fcitx to default

# git

    sudo apt install liberror-perl git-man git
    ssh-keygen // 一直回车
    cat ~/.ssh/id_rsa.pub  // 将该信息添加到 github或者gitlab的ssh配置中
    git config --global core.editor vim
    git config --global user.name name
    git config --global user.email name.xxx.com

>安装git commit message 格式化工具 [commitizen](https://github.com/commitizen/cz-cli) (需要先安装nodejs)

    npm install -g commitizen  // 安装后通过 git cz提交信息
    commitizen init cz-conventional-changelog --save --save-exact // 在本项目中安装选项式commit message生成工具

# nodejs

    curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
    sudo apt install nodejs

# visual studio code

  [下载Visual Studio Code](https://code.visualstudio.com/download)

    sudo dpkg -i vscode-amd64.deb

  打开VSCode后，使用 ctrl + p 快捷键打开命令输入面板，输入以下命令

    ext install vim
    ext install Theme-MaterialKit
    ext install vue
    ext install jsx

# shadowsocks

    sudo add-apt-repository ppa:hzwhuang/ss-qt5
    sudo apt update
    sudo apt install shadowsocks-qt5

  输入以下命令打开shaowsocks配置界面

    sudo ss-qt5

  shadowsocks配置如下：

  ![shaodowsocks 配置](/img/switchyomega.png)
  
  System Settings  ->  Network  ->  Network proxy 配置如下：

  ![本地网络配置](/img/localnetworkconfig.png)

  chrome通过[SwitchyOmega](https://github.com/FelisCatus/SwitchyOmega/releases)配置

  ![chrome配置](/img/switchyomega.png)

# email

  Thunderbird 配置如下(*只针对腾讯企业邮箱*)：

  ![第一步](/img/email1.png)
  ![第二步](/img/email2.png)

# places

  删除places的那些目录

    sudo vim ~/.config/user-dirs.dirs
    
  删除不需要的项，然后执行以下命令，重启系统

    echo "enabled=false" > ~/.config/user-dirs.conf

# java JDK

    sudo add-apt-repository ppa:webupd8team/java
    sudo apt upate
    sudo apt install oracle-java8-installe

# charles (需要先安装 java JDK)

    // install the GPG public key for the repository 
    wget -q -O - https://www.charlesproxy.com/packages/apt/PublicKey | sudo apt-key add -
    // add repository
    sudo add-apt-repository 'deb https://www.charlesproxy.com/packages/apt charles-proxy main'
    sudo apt update
    sudo apt install charles-proxy

  破解charles：上网找个破解文件 覆盖 /usr/lib/charles-proxy 的 charles.jar文件 （支持正版）

# zsh

    sudo apt install zsh

# tree

    sudo apt install tree