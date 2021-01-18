# Installation of Deeplabcut to Ubuntu 18.04

The following information **may contain unnecessary processes**. Also, this information is current as of January 2021, and **we do not know if it will remain valid in the future**.

## 0. Prepare the necessary files

- Put the following files in your `Downloads` (or whatever you prefer) folder. It is not mentioned below, but you have to run `cd Downloads/` to run the files.

- Driver: `NVIDIA-Linux-x86_64-450.66.run`

  - Search from https://www.nvidia.com/download/Find.aspx

  - Firefox on Ubuntu can't access the NVIDIA download page for some reason, so download it from another PC and put it in the `Downloads/` folder.

  - If the driver is too old for the GPU, you will get the following error.

    - ```
      An error occurred while performing the step: "Building kernel modules".
      ```

    - https://forums.developer.nvidia.com/t/i-cant-install-nvidia-driver-an-error-occurred-while-performing-the-step-building-kernel-modules/72239

- CUDA: `cuda_10.0.130_410.48_linux.run`
  
  - Download from https://developer.nvidia.com/cuda-10.0-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=runfilelocal
- cudnn: `cudnn-10.0-linux-x64-v7.4.2.24.tgz`
  
  - Registration to NVIDIA Developer website is needed: https://developer.nvidia.com/rdp/cudnn-archive
- Anaconda: `Anaconda3-2019.07-Linux-x86_64.sh`
  - Download from https://repo.anaconda.com/archive/
  - The second version from the end, based entirely on Python 3.7. 2019.10 could not be installed for some reason.
- `cuda_10.0.130_410.48_linux.run` includes a driver file (410.48), and if you run it as it is, drivers may conflict with each other. You should disassemble it beforehand.

```sh
sudo ./cuda_10.0.130_410.48_linux.run --extract=/home/USERNAME/asdf
```

- Then, three run files of `cuda`, `cuda-samples`, and `NVIDIA` driver are stored in `/home/USERNAME/asdf/`, get only `cuda` file out of them.
- For `NVIDIA-Linux-x86_64-450.66.run`, provide the following execute permissions just in case (the others were not necessary).

```sh
chmod +x NVIDIA-Linux-x86_64-450.66.run
```



## 1. Installation of the NVIDIA driver

### 1-1. Installation of libraries

- `build-essential` is necessary.
- `gcc7.5` will be installed. The build configuration of Tensorflow 1.13.1 shows that gcc is 4.8, but this would not have any effect since we will use the prebuilt tensorflow binary.

```sh
# gcc installation----
sudo apt-get install build-essential
gcc --version
# Installation of i386 compatible libraries----
sudo dpkg --add-architecture i386
sudo apt install libc6:i386
```

### 1-2. Stop nouveau driver

- If you do not do this, you will get the following error and will not be able to install the driver.

```
ERROR: Unable to load the kernel module 'nvidia.ko'. This happens most frequently when this kernel module was built against the wrong or improperly configured kernel sources, with a version of gcc that differs from the one used to build the target kernel, or if another driver, such as nouveau, is present and prevents the NVIDIA kernel module from obtaining ownership of the NVIDIA GPU(s), or no NVIDIA GPU installed in this system is supported by this NVIDIA Linux graphics driver release.
```

- Code that worked：
  - http://tleyden.github.io/blog/2014/10/25/cuda-6-dot-5-on-aws-gpu-instance-running-ubuntu-14-dot-04/

```sh
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
## Add below code to blacklist-nouveau.conf
blacklist nouveau
blacklist lbm-nouveau
options nouveau modeset=0
alias nouveau off
alias lbm-nouveau off
## Exit editing
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
sudo update-initramfs -u # sudo is necessary
sudo reboot
## Verify that the driver is not running----
lsmod | grep -i nouveau # If nothing comes back, it's OK
```

- The nano editor can be quit with `ctrl+x`,  `y`, and `Enter`.
- Below is the code that did not worked：

```sh
sudo bash -c "echo blacklist nouveau > /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo bash -c "echo options nouveau modeset=0 >> /etc/modprobe.d/blacklist-nvidia-nouveau.conf"
sudo update-initramfs -u
sudo reboot
```

## 1-3. Driver installation

```sh
sudo init 3
sudo service gdm3 stop
sudo bash NVIDIA-Linux-x86_64-450.66.run
```

- Stop the GUI with `sudo init 3`. You will be asked to enter your login ID and password, follow the instructions.

- I set all the options to Yes for the time being (when the driver is installed for the first time).

- You can return to the GUI with `sudo init 5` or `sudo service gdm restart`.

- By using command`nvidia-smi`, you can check driver installation. Note that the upper right corner says `CUDA: 11.0 `which means "Supported MAX versions of CUDA" and is not the CUDA actually used. Check the CUDA version with `nvcc -V`.

  - https://github.com/DeepLabCut/DeepLabCut/issues/1056#issuecomment-754135754
  
  - > the nvidia-smi can actually show the highest possible CUDA given the driver, it's a bit annoying in that way! So, you likely have cuda 10 installed, you can check in the terminal with: `nvcc -V`



## 2. Installation of CUDA 10.0

### 2-1. Installation

```sh
sudo init 3
sudo service gdm3 stop
sudo bash cuda-linux.10.0.130-24817639.run # generated by extracting the original CUDA file
sudo reboot
```

- At this stage, you cannot use `nvcc -V`command. The terminal prompt to run `sudo apt install nvidia-cuda-toolkit`. But don't do it!

### 2-2. Add path

```sh
echo -e "\n## CUDA and cuDNN paths"  >> ~/.bashrc
echo 'export PATH=/usr/local/cuda-9.0/bin:${PATH}' >> ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-9.0/lib64:${LD_LIBRARY_PATH}' >> ~/.bashrc
source ~/.bashrc
```

- Then, `nvcc -V`  will work.



## 3. Installation of cudnn

- This process may be unnecessary.

```sh
# Extract archive
tar -zxvf cudnn-10.0-linux-x64-v7.4.2.24.tgz
# Move the extracted contents to the CUDA directory
sudo cp -P cuda/lib64/* /usr/local/cuda-10.0/lib64/
sudo cp cuda/include/* /usr/local/cuda-10.0/include/
# Provide read permissions to all users
sudo chmod a+r /usr/local/cuda-10.0/include/cudnn.h
```



## 4. Installation of Anaconda

### 4-1. Installation

```sh
sudo bash Anaconda3-2019.07-Linux-x86_64.sh
```

### 4-2. Add path

```sh
sudo nano ~/.bashrc
## Add below code to .bashrc
export PATH=~/anaconda3/bin:$PATH
## Exit editing .bashrc
source ~/.bashrc
```



## 5. Create a virtual environment

```sh
conda env create -f DLC-GPU.yaml
```



## 6. Limit GPU usage

- When I run `deeplabcut.train_network()` without preperation, I get the following error.

```
Failed to get convolution algorithm. This is probably because cuDNN failed to initialize, so try looking to see if a warning log message was printed above.
```

- I thought it might be because the version of cudnn is different (7.6) than recommended (7.4). However, cudnn=7.4 can not be installed via `conda install`. It seems that I should limit the GPU usage.  In fact, if I run the command `nvidia-smi` when  `deeplabcut.train_network()` doesn't work, the usage of GPU is almost 100%.
- So, run the following code


```python
import tensorflow as tf
from tensorflow.keras.backend import set_session
config = tf.ConfigProto()
config.gpu_options.allow_growth = True  
config.gpu_options.visible_device_list = "0"
config.log_device_placement = True  
sess = tf.Session(config=config)
set_session(sess)  # set this TensorFlow session as the default session for Keras
```

- Next, you can check the GPU by 

```python
from tensorflow.python.client import device_lib
device_lib.list_local_devices()
```

- and run  `deeplabcut.train_network()`.

- In some cases (≃50%?), you may still get a convolution error because you can't limit the GPU usage properly. In that case, you have to reboot and try again, and then **pray**.