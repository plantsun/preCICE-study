# Environment

**From Ubuntu:22.04**

```shell
docker run -itd \
  --name ubuntu-test \
  --network host \
  --gpus all \
  --runtime=nvidia \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  -e DISPLAY=$DISPLAY \
  -e XDG_RUNTIME_DIR=/tmp/runtime-root \
  -e NVIDIA_DRIVER_CAPABILITIES=all \
  ubuntu:22.04
```

命令参数说明：

| 参数                | 说明                                                     |
| ------------------- | -------------------------------------------------------- |
| `--network host`    | [网络]共享主机网络栈                                     |
| `-v /tmp/.X11-unix` | [图形化界面]挂载X11套接字                                |
| `-e DISPLAY`        | [图形化界面]传递显示变量                                 |
| `-e XAUTHORITY`     | [图形化界面]客户端图形界面程序安全地连接到服务器（未用） |
| `--privileged`      | [systemd]授予完整权限以运行systemd                       |
| `-v /sys/fs/cgroup` | [systemd]systemd依赖的cgroup                             |
| `/sbin/init`        | [systemd]使用systemd作为初始化进程                       |

1. `mkdir -p /tmp/runtime-root && chmod 700 /tmp/runtime-root`

   `echo "export XDG_RUNTIME_DIR=/tmp/runtime-root" >> ~/.zshrc`

2. 换源

   ```shell
   sed -i 's|^|# |g' /etc/apt/sources.list \
   && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list \
   && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list \
   && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list \
   && echo "deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list
   
   && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list
   ```

2. 更新软件列表

   ```shell
   apt update && apt install ca-certificates -y && apt update && apt upgrade -y
   ```

3. 安装软件

   ```shell
   DEBIAN_FRONTEND=noninteractive apt install -y \
   tzdata \
   language-pack-zh-hans \
   x11-apps \
   git \
   vim \
   zsh \
   openssh-server \
   openssh-client \
   curl \
   wget \
   silversearcher-ag \
   fzf \
   tmux
   ```

4. 设置密码

   ```shell
   passwd
   ```

5. 配置ssh

   ```shell
   sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config \
   && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config \
   && service ssh start
   ```

   切换至ssh连接

7. 安装preCICE

   + 安装依赖项

     ```shell
     DEBIAN_FRONTEND=noninteractive apt install -y \
     build-essential \
     pkg-config \
     cmake
     ```

   + 安装OpenFOAM2406

     ```shell
     wget -q -O - https://dl.openfoam.com/add-debian-repo.sh | bash \
         && apt update \
         && apt install openfoam2406-default \
         && echo "source /usr/lib/openfoam/openfoam2406/etc/bashrc" >> ~/.zshrc \
         && source ~/.zshrc
     ```

   + 安装preCICE 3.1.2

     ```shell
     wget https://github.com/precice/precice/releases/download/v3.1.2/libprecice3_3.1.2_jammy.deb \
     && chmod 777 ./libprecice3_3.1.2_jammy.deb \
     && apt install ./libprecice3_3.1.2_jammy.deb -y \
     && rm -rf libprecice3_3.1.2_jammy.deb
     ```

   + 安装OpenFOAM-preCICE adapter 1.3.1

     ```shell
     wget https://github.com/precice/openfoam-adapter/archive/refs/tags/v1.3.1.tar.gz \
         && tar -xzf v1.3.1.tar.gz \
         && cd openfoam-adapter-1.3.1/ \
         && ./Allwmake \
         && cd \
         && rm -rf ./v1.3.1.tar.gz
     ```

   + Config visualization

      > 安装pip3，graphviz

      ```shell
      DEBIAN_FRONTEND=noninteractive apt install -y \
      python3-pip \
      graphviz
      ```

      > 安装pygobject

      ```
      DEBIAN_FRONTEND=noninteractive apt install -y \
      python3-gi \
      python3-gi-cairo \
      libgirepository1.0-dev \
      libcairo2-dev \
      gir1.2-gtk-4.0 \
      && pip3 install pycairo \
      PyGObject
      ```

      > 创建并运行hello.py，有图形化程序说明正常

      ```python
      import sys
      
      import gi
      
      gi.require_version("Gtk", "4.0")
      from gi.repository import GLib, Gtk
      
      
      class MyApplication(Gtk.Application):
          def __init__(self):
              super().__init__(application_id="com.example.MyGtkApplication")
              GLib.set_application_name('My Gtk Application')
      
          def do_activate(self):
              window = Gtk.ApplicationWindow(application=self, title="Hello World")
              window.present()
      
      
      app = MyApplication()
      exit_status = app.run(sys.argv)
      sys.exit(exit_status)
      ```

      > 安装precice-config-visualizer
      >
      > > 主机安装[NVIDIA Container Toolkit](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)
      
      ```shell
      DEBIAN_FRONTEND=noninteractive apt install -y \
      yaru-theme-icon \
      libgtk-3-dev \
      libcanberra-gtk-module \
      libcanberra-gtk3-module \
      paraview \
      && pip3 install precice-config-visualizer \
      precice-config-visualizer-gui
      ```

7. 配置zsh

   ```shell
   echo "185.199.109.133  raw.githubusercontent.com" >> /etc/hosts \
   && echo "140.82.121.3 github.com" >> /etc/hosts \
   && chsh -s /bin/zsh \
   && sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" --unattended \
   && sed -i '/^plugins=(git)$/c\plugins=(\n    zsh-syntax-highlighting\n    zsh-autosuggestions\n    git\n    extract\n)' ~/.zshrc \
   && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
   && git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
   && git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k \
   && sed -i 's/ZSH_THEME="robbyrussell"/ZSH_THEME="powerlevel10k\/powerlevel10k"/' ~/.zshrc
   ```

8. 配置tmux

   ```shell
   cd && git clone https://github.com/gpakosz/.tmux.git \
   && ln -s -f .tmux/.tmux.conf \
   && cp .tmux/.tmux.conf.local .
   ```





























> 安装Boost 1.71
>
> ```shell
> apt install -y \
> libopenmpi-dev \
> openmpi-bin \
> libboost-all-dev \
> libeigen3-dev \
> libxml2-dev \
> libvtk7-dev \
> libscotch-dev \
> libparmetis-dev \
> libptscotch-dev \
> libmetis-dev \
> libfftw3-dev \
> libhdf5-openmpi-dev \
> python3-dev \
> python3-pip \
> && pip3 config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
> && pip3 install numpy
> wget https://archives.boost.io/release/1.71.0/source/boost_1_71_0.tar.gz \
> && tar -xzf boost_1_71_0.tar.gz \
> && cd boost_1_71_0 \
> && ./bootstrap.sh --prefix=/usr/local \
> && ./b2 -j$(nproc) \
> && ./b2 install \
> && cd .. \
> && rm -rf boost_1_71_0 boost_1_71_0.tar.gz

---------------

**总结**

Dockerfile

```dockerfile
FROM ubuntu:22.04

RUN sed -i 's|^|# |g' /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse" >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse" >> /etc/apt/sources.list \
    && echo "deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse" >> /etc/apt/sources.list \
    && echo "deb http://security.ubuntu.com/ubuntu/ jammy-security main restricted universe multiverse" >> /etc/apt/sources.list \
    && apt update -y && apt install ca-certificates -y && apt update -y && apt upgrade -y \
    && DEBIAN_FRONTEND=noninteractive apt install -y \
    tzdata \
    git \
    vim \
    zsh \
    openssh-server \
    openssh-client \
    curl \
    silversearcher-ag \
    fzf \
    tmux \
    && sed -i 's/#Port 22/Port 2222/' /etc/ssh/sshd_config \
    && sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config
```
