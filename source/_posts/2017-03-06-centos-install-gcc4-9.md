---
title: CentOS安装GCC4.9
toc: true
date: 2017-03-06 16:22:10
description: CentOS默认安装的GCC版本较低时需要
categories:
- 锤子的锤
tags:
- CentOS
---
## 编译安装

Step 1: 下载gcc 4.9.1, 在 https://ftp.gnu.org/gnu/gcc/gcc-4.9.1/ 下载。

```
wget https://ftp.gnu.org/gnu/gcc/gcc-4.9.1/gcc-4.9.1.tar.gz
```
Step 2: 解压压缩包
```
tar xvzf gcc-4.9.1.tar.gz
```
Step 3: 下载gcc编译的依赖项
```
./contrib/download_prerequisites
```
如果此步骤因为网络原因没有下载成功，手动去下载./contrib/download_prerequisites中的五个文件，然后在那个五个文件放到contrib目录下，然后编辑download_prerequisites文件，将带wget的语句注释掉，保存，然后在运行一次。

Step 4: 运行configure
```
./configure --prefix=/usr/gcc --disable-multilib
```
Step 5: 编译
```
make
```
编译过程若遇到`make[3]: *** [s-attrtab] Killed`错误，是因为内存不够，增加虚拟内存可解决。
参考[SO Question](https://stackoverflow.com/questions/18389612/make-exits-with-error-2-when-trying-to-install-gcc-4-8-1)
```
SWAP=/tmp/swap
dd if=/dev/zero of=$SWAP bs=1M count=1024
mkswap $SWAP
sudo swapon $SWAP
```

Step 6: 安装
```
make install
```
Step 7: 根据需要设置软链接和环境变量

Step 8: 测试是否安装成功
```
gcc -v
```

## yum安装

```bash
yum install centos-release-scl-rh
yum install devtoolset-3-gcc devtoolset-3-gcc-c++
scl enable devtoolset-3 bash
update-alternatives --install /usr/bin/gcc-4.9 gcc-4.9 /opt/rh/devtoolset-3/root/usr/bin/gcc 10
update-alternatives --install /usr/bin/g++-4.9 g++-4.9 /opt/rh/devtoolset-3/root/usr/bin/g++ 10
```

