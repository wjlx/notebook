# git 配置

## 1. git 颜色设置

```md
git config --global color.diff auto
git config --global color.status auto
git config --global color.branch auto
```

## 2. git status 中文显示

```md
git config --global core.quotepath false
```

## 3. 免密pull/push

1. 先cd到根目录，执行git config --global credential.helper store命令

```md
[root@iZ25mi9h7ayZ ~]# git config --global credential.helper store
```

2. 执行之后会在.gitconfig文件中多加红色字体项

```md
[user]
        name = XX
        email = xxxx@xxxx.com
[credential]
        helper = store
```

3. 之后cd到项目目录，执行git pull命令，会提示输入账号密码。输完这一次以后就不再需要，并且会在根目录生成一个.git-credentials文件
4. 之后pull/push代码都不再需要输入账号密码了
