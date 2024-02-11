## Jenkins

#### [link tham khỏa](https://www.youtube.com/watch?v=d2-HXYKjfbc&list=PLf_Ekmm515S00fMH-uzqxGDOHCFA7V4Wy&index=2)

#### [link tham khỏa github](https://github.com/nhtua/bookshell/tree/master)

### B1: thêm user 'jenkins' ở máy host (tránh chạy bằng user 'root')

- Clone command từ git repository và chạy:

```command
git clone https://github.com/nhtua/bookshell.git
```

```command
cd bookshell/vps/
```

```command
sudo ./add-docker-user.sh jenkins
```

### B2: sửa cấu hình mặc định của docker

```vim
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target docker.socket firewalld.service containerd.service time-set.target
Wants=network-online.target containerd.service
Requires=docker.socket

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock -H tcp://192.168.1.33 --containerd=/run/containerd/containerd.sock
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutStartSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process
OOMScoreAdjust=-500

[Install]
WantedBy=multi-user.target
```

ps:

- cần sửa dòng: 'ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock'
  thành 'ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock -H tcp://192.168.1.33 --containerd=/run/containerd/containerd.sock'
  <br><br>
  <i>chú thích: 192.168.1.33 là ip của máy host</i>

### B3: chạy jenkins với docker

```command
mkdir -p /home/jenkins/data
```

```command
cd /home/jenkins
```

```command
docker run -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):$(which docker) -v `pwd`/data:/var/jenkins_home  -p 8080:8080  --user 1000:999 --name jenkins-server -d jenkins/jenkins:lts
```

### Các plugin cần cài đặt thêm để có thể áp dụng với github

```text
GitHub Integration

Docker

Docker Pipeline
```

## Forward port url của jenkins local để sử dụng wedhook sử dụng: (https://smee.io/)

- B1: truy cập [https://smee.io/](https://smee.io/) để lấy url (bấm Start a new channel)

- B2: chạy lệnh sau trên server có jenkins: để cài đặt 'smee'

```command
npm install --global smee-client
```

- B3: sau khi cài đặt lệnh 'smee' chạy lệnh sau để có thể thực hiện forward

```command
smee --url [link lấy ở B1] --path /github-webhook/ --port 8080
```

- B4: thực hiện thay đổi thông tin 'Webhooks' trong project github
