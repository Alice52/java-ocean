## CentOS

### common command

```shell
# 1. reboot
shutdown -r now
# 2. 查看版本
cat /etc/redhat-release
lsb_release -a
# 3. 更新 yum 源
yum makecache fast
# 4. start dcoker
systemctl start docker
```

---

## replace yum source

```shell
# 1. 备份原来的yum源
sudo cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
# 2. 设置aliyun的yum源
sudo wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 3. 添加EPEL源
sudo wget -P /etc/yum.repos.d/ http://mirrors.aliyun.com/repo/epel-7.repo
# 4. 清理缓存并生成新的缓存
sudo yum clean all
sudo yum makecache
```

## install software

### install docker

```shell
# Install required packages
yum install -y yum-utils device-mapper-persistent-data lvm2
# 配置阿里云加速
sudo yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 查看
cat etc/yum.repos.d/docker-ce.repo
# install docker
sudo yum install docker-ce
# start dcoker
systemctl start docker
# 配置镜像加速
docker vim /etc/docker/daemon.json
########################################
# {
#   "registry-mirrors": ["https://wfjvo9ge.mirror.aliyuncs.com"]
# }
########################################
docker sudo systemctl daemon-reload
docker sudo systemctl restart docker


# docker 开机自启动容器
docker update --restart=always 镜像ID
# docker log 查看
sudo docker logs -f -t --tail 行数 容器名
```

## remove software

### romove docker

```shell
systemctl stop docker
yum -y remove docker-ce
rm -rf /var/lib/docker
```

## install tool

- tree

  ```shell
  # 1. install
  yum -y install tree
  # 2. use
  man tree
  tree -L 2
  ```

- zsh
  ```shell
  # 1. install zsh
  echo $SHELL
  yum -y install zsh
  cat /etc/shells
  chsh -s /bin/zsh
  # 2. install oh-my-zsh
  wget https://github.com/robbyrussell/oh-my-zsh/raw/master/tools/install.sh -O - | sh
  ```

---

## question

- 1. docker 安装映射出来的 log 文件; 但是 没有 log 对比之前的非 docker 安装
