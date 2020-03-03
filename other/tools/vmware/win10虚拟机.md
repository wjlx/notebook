# 虚拟机安装问题

## 目录挂载

```markdown
查看用户组信息命令： id wjl
共享目录挂载命令： vmhgfs-fuse .host:/ /home/mnt -o uid=1000,umask=133
卸载目录： umount -f /home/mnt
```

## 系统挂载

修改/etc/fstab文件

```txt
.host:/github   /home/wjl/github fuse.vmhgfs-fuse allow_other,uid=1000,umask=133,defaults       0 0
```

## [umask 权限分配](https://docs.oracle.com/cd/E19683-01/817-3814/userconcept-95347/index.html)

| umask Octal Value | File Permissions | Directory Permissions |
|-------------------|------------------|-----------------------|
|                 0 | rw-              | rwx                   |
|                 1 | rw-              | rw-                   |
|                 2 | r--              | r-x                   |
|                 3 | r--              | r--                   |
|                 4 | -w-              | -wx                   |
|                 5 | -w-              | -w-                   |
|                 6 | --x              | --x                   |
|                 7 | --- (none)       | --- (none)    |

## 使用过程中遇到的问题

### 问题1

```markdown
VMware Workstation 与 Device/Credential Guard 不兼容。在禁用 Device/Credential Guard 后，可以运行 VMware Workstation。有关更多详细信息，请访问 http://www.vmware.com/go/turnoff_CG_DG
```

解决方案：

```markdown
控制面板 -> 程序和功能 -> 启用或关闭windows功能
去掉勾选： Virtual Machine Platform
```
