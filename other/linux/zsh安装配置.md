# zsh安装配置

[TOC]

## 一、安装

### 1. 安装git

```md
    sudo yum install git -y
```

###　2. 安装zsh

```md
   sudo yum install zsh -y
   chsh -s /bin/zsh     # 设置默认shell
```

### 3. 安装oh-my-zsh

```md
    # via curl
    sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

    # via wget
    sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

### 4. 安装autojump

```md
    yum -y install epel-release  # 安装epel, 默认仓库没有autojump包
    yum repolist  # 刷新仓库
    sudo yum install autojump autojump-zsh -y
```

### 5. 安装zsh-syntax-highlighting

```md
    git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

    plugins=( [plugins...] zsh-syntax-highlighting)
```

### 6. 安装autosuggestions

```md
    git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

    plugins=(zsh-autosuggestions)
```

### 6. 安装fzf

```md
    git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
    ~/.fzf/install
```

## 二、配置

### 1. 终端显示ip

```md
IP=$(ip addr | grep -w "inet" | grep "192.168.21" | awk '{ print $2; }' | sed 's/\/.*$//')
PS1='${ret_status} %{$fg[cyan]%}[%n@$IP %~] $ %{$reset_color%} $(git_prompt_info)'
```
