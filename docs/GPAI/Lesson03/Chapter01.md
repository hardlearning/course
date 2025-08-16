# PyTorch框架介绍与配置安装

课程里安装的版本：
- torch: 1.6.0
- torchvision: 0.7.0

## 安装CPU版：

```
pip install torch
pip install torchvision
```

## 安装GPU版：

版本对照：https://pytorch.org/get-started/previous-versions/

1. 安装CUDA

下载地址：https://developer.nvidia.com/cuda-toolkit-archive

```
# 查看CUDA支持版本
nvidia-smi
# 卸载已安装的驱动
sudo apt purge nvidia*
# 安装对应版本的驱动
sudo apt install nvidia-driver-515
sudo reboot
```

下载CUDA安装包：

```
chmod +x cuda_11.7.1_515.65.01_linux.run
sudo ./cuda_11.7.1_515.65.01_linux.run
# 测试是否安装成功
nvcc -V
```

安装后显示：

```
Driver:   Not Selected
Toolkit:  Installed in /usr/local/cuda-11.7/

Please make sure that
 -   PATH includes /usr/local/cuda-11.7/bin
 -   LD_LIBRARY_PATH includes /usr/local/cuda-11.7/lib64, or, add /usr/local/cuda-11.7/lib64 to /etc/ld.so.conf and run ldconfig as root

To uninstall the CUDA Toolkit, run cuda-uninstaller in /usr/local/cuda-11.7/bin
***WARNING: Incomplete installation! This installation did not install the CUDA Driver. A driver of version at least 515.00 is required for CUDA 11.7 functionality to work.
To install the driver using this installer, run the following command, replacing <CudaInstaller> with the name of this run file:
    sudo <CudaInstaller>.run --silent --driver

Logfile is /var/log/cuda-installer.log
```

配置环境变量：

```
export PATH=/usr/local/cuda-11.7/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.7/lib64:$LD_LIBRARY_PATH
```

2. 安装PyTorch

下载地址：https://download.pytorch.org/whl/torch_stable.html

下载whl文件并安装：

```python
pip install torch-1.13.1+cu117-cp310-cp310-linux_x86_64.whl
```

测试是否安装成功：

```
>>> import torch
>>> print(torch.cuda.is_available())
True
```