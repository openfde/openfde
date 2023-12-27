OpenFDE是一个基于AOSP、Waydroid、LineageOS打造的可运行andorid 应用的linux桌面环境。

编译环境搭建
1.1 准备一台X86主机，内存要求16G，最少512G固态硬盘，1T更好。（硬盘对编译的速度影响非常大）安装ubuntu22.04 下载链接：https://ubuntu.com/download/server


1.2 下载repo脚本文件
  android使用repo脚本管理源码树，方便大家用一个命令就可以下载几百个仓库。
  curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo -o repo
  chmod +x repo
  sudo mv repo /usr/local/bin/repo

  export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo'
  echo export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo' >> ~/.bashrc

1.3 安装编译依赖库
sudo apt install git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip libncurses5



二. 源代码下载
  mkdir fde && cd fde
  repo init -u https://gitee.com/openfde/manifests -b 1.0.5 --git-lfs
  repo sync -j24

三. 源码编译
    source build/envsetup.sh
    lunch 46 
    make -j24