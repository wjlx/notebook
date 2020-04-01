# gitlab使用

## gitlab

### 安装docker

```console
yum install docker-ce
```

### 拉取gitlab镜像

```console
docker pull gitlab/gitlab-ce
```

### 启动gitlab

编写shell脚本： `gitlab.sh`

```sh
docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 222:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

启动脚本： `./gitlab.sh`