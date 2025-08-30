### **manylinux 链接安装Python**

```bash
rm -rf /usr/bin/python
ln -s /opt/_internal/cpython-3.10.16/bin/pip3 /usr/bin/pip3
ln -s /opt/_internal/cpython-3.10.16/bin/pip3 /usr/bin/pip
ln -s /opt/_internal/cpython-3.10.16/bin/python3.10 /usr/bin/python3
ln -s /opt/_internal/cpython-3.10.16/bin/python3.10 /usr/bin/python
ln -s /opt/_internal/cpython-3.10.16/bin/python3.10 /usr/bin/python3.10

检查是否都是3.10
vi /usr/bin/yum  第一行python增加2
vi /usr/libexec/urlgrabber-ext-down  第一行python增加2
```



### 安装CANN

```bash
安装依赖
yum install -y unzip zlib-devel libffi-devel openssl-devel pciutils net-tools sqlite-devel lapack-devel gcc-gfortran python3-devel

pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple attrs numpy==1.24.0 decorator sympy cffi pyyaml pathlib2 psutil protobuf scipy requests absl-py wheel typing_extensions (备用源： https://pypi.mirrors.ustc.edu.cn/simple)

// 更换为arm 或者x86源
wget "https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.0.RC3/Ascend-cann-toolkit_8.0.RC3_linux-x86_64.run"

sh Ascend-cann-toolkit_8.0.RC3_linux-x86_64.run --full

source /usr/local/Ascend/ascend-toolkit/set_env.sh



# install CANN8.1
    wget https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.1.RC1/Ascend-cann-toolkit_8.1.RC1_linux-aarch64.run && \
    chmod +x Ascend-cann-toolkit_8.1.RC1_linux-aarch64.run && \
    ./Ascend-cann-toolkit_8.1.RC1_linux-aarch64.run --full -q && \
    wget https://ascend-repo.obs.cn-east-2.myhuaweicloud.com/CANN/CANN%208.1.RC1/Ascend-cann-kernels-910b_8.1.RC1_linux-aarch64.run && \
    chmod +x Ascend-cann-kernels-910_8.1.RC1_linux-aarch64.run && \
    ./Ascend-cann-kernels-910_8.1.RC1_linux-aarch64.run --install -q && \
    source ~/Ascend/ascend-toolkit/set_env.sh
```

### arm架构更换yum源 

```bash
curl http://mirrors.aliyun.com/repo/Centos-altarch-7.repo -O /etc/yum.repos.d/CentOS-Base.repo

# 配置CentOS-SCLo-scl.repo国内地址
vim /etc/yum.repos.d/CentOS-SCLo-scl.repo

[centos-sclo-sclo]
name=CentOS-7 - SCLo sclo
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/sclo/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo


# 配置CentOS-SCLo-scl-rh.repo
vim /etc/yum.repos.d/CentOS-SCLo-scl-rh.repo

[centos-sclo-rh]
name=CentOS-7 - SCLo rh
baseurl=https://mirrors.aliyun.com/centos/7/sclo/x86_64/rh/
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo


# 刷新缓存
yum repolist && yum clean all && yum makecache
```





