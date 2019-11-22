# Progress of install Pytorch V1.0.0 on Raspberry Pie(last edit: 22/Nov/2019)
This description is upgrade version of [this manual](https://blog.openmined.org/federated-learning-of-a-rnn-on-raspberry-pis/)

## git issue what We upload on pytorch repository
https://github.com/pytorch/pytorch/issues/26455

## Environment
* model: Raspberry Pi 3 Model B V1.2
* PyTorch version: N/A
* Is debug build: N/A
* CUDA used to build PyTorch: N/A
* OS: Raspbian GNU/Linux 10 (buster)
* GCC version: (Raspbian 8.3.0-6+rpi1) 8.3.0
* CMake version: version 3.13.4
* Python version: 3.7
* Is CUDA available: N/A
* CUDA runtime version: Could not collect
* GPU models and configuration: Could not collect
* Nvidia driver version: Could not collect
* cuDNN version: Could not collect
* Versions of relevant libraries:
* [pip3] numpy==1.16.2
* [conda] Could not collect

## progress

### 0. Update pip3.
```
pip3 install --upgrade pip
```
After upgrade, you must reboot raspberry
```
(check version)
pip3 -V
```
Also, when you try apt update or apt-get update you need to set raspberry pi's time correctly
```
ex)
sudo date -s '9 Jul 2019 22:44'
sudo apt-get update
```

### 1. Create a SWAP file of 2 GBs. 
This will be necessary during the upcoming installation process of PyTorch, is it is rather memory-hungry. If you do not create a SWAP, you are likely going to run out of memory during the installation process - as it happened to me. Ensure you have sufficient available secondary memory on your Rasbperry's SD card with df -h before running the following commands:

```
sudo touch /swap1
sudo dd if=/dev/zero of=/swap1 bs=1024 count=2097152
sudo mkswap /swap1
sudo nano /etc/fstab
```

This will open up your fstab file. If you see something like,
```
/swap0 swap swap
```
That means you already had a swap,  so replace /swap0 swap swap with the following line:
```
/swap1 swap swap
```
If the line/swap0 swap swap is not present, then add the following line to your fstab file.
```
/swap1 swap swap
```
In order for your Raspberry PI to recognize the new SWAP space, reboot your Raspberry PI:
```
sudo reboot -h now
```

### 2. Install the Pytorch's packages' dependencies: 
We will be needing a few more packages before actually installing Pytorch, so run the following commands to install Pytorch's dependencies:

```
sudo apt install libopenblas-dev libblas-dev m4 cmake cython python3-dev python3-yaml python3-setuptools
```

### 3. Get Pytorch 1.0.0 from the official GIT repository: 
At the time of writing, Pytorch was not available from the official Python "pip" package manager for ARM architectures, so we will need to compile it from source. Let's get it the 1.0.0 version from the official GIT repository.
```
mkdir pytorch_install && cd pytorch_install
git clone --recursive https://github.com/pytorch/pytorch --branch=v1.0.1
(-> in my case I need to add sudo)
sudo git clone --recursive https://github.com/pytorch/pytorch --branch=v1.0.1)
cd pytorch
```

### 4. How to resolve __atomic_xxx_8 error: 
If you build pytorch without this progress 
```
undefined reference to __atomic_fetch_add_8' /usr/bin/ld: /home/pi/pytorch/build/lib/libcaffe2.so: undefined reference to __atomic_load_8'
```
this error will evoke.
```
sudo apt-get install libatomics-ops-dev
Change CMAKE_CXX_FLAGS variable in CMakeLists.txt file (in the main directory). i.e. add line set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -latomic")

cd pytorch
git submodule update --remote third_party/protobuf
```

### 5.0 Set up environment variables for compiling Pytorch: 
Let's run these last few commands in a shell before starting the compilation process, or add the environment variables to the. bashrc file in your home directory. Bear in mind we will need these variables just for the Pytorch's installation process. The NO_CUDA flag will make sure that the compiler doesn’t look for cuda files, as the Raspberry PI is not equipped with a GPU by default.
```
export NO_CUDA=1
export NO_DISTRIBUTED=1
export NO_MKLDNN=1 
export NO_NNPACK=1
export NO_QNNPACK=1
```
### 5.1 Build caffe2: 
If you build pytorch without building caffe2 
there will be Failed to run 'bash tools/build_pytorch_libs.sh caffe2'
So you need to build caffe2 before build pytorch.

```
cd pytorch

git submodule update –-init

sudo apt-get install python-yaml

sudo pip install typing

USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 ./scripts/build_raspbian.sh

(If there is permission error, then)
sudo –E USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 ./scripts/build_raspbian.sh

-> building takes 3 hours in my case
```

### 5.2 Compile Pytorch: 
Now start building and cross your fingers, hoping no errors arise. This process may be quite lengthy: in my case, I let it run overnight and manually installed a few missing packages that could not be automatically downloaded and compiled by the pytorch installer.

```
USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 BUILD_TEST=0 python3 setup.py build

(If there is permission error, then)
sudo –E USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 BUILD_TEST=0 python3 setup.py build
(-> ‘BUILD_TEST=0’ makes building more faster) 

-> This building takes about 1day in my case
```

TROUBLESHOOTING: If you encounter the following error:
Failed to run 'bash tools/build_pytorch_libs.sh --use-cuda --use-nnpack --use mkldnn --use qnnpack caffe2'
Then that means you haven’t set the environment variables properly. Set the environment variables correctly, then retry.
If the compilation process stops halfway because of an error, your progresses is not lost! It will resume compiling at the point where it stopped.

### 6. Install Pytorch: 
The installation should be much quicker than the compilation process (it took about 5 minutes on my Raspberry PI). To install Pytorch, just run:

```
USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 BUILD_TEST=0 python3 setup.py install

(If there is permission error, then)
sudo -E USE_MKLDNN=0 USE_QNNPACK=0 USE_NNPACK=0 USE_DISTRIBUTED=0 BUILD_TEST=0 python3 setup.py install
```

If the installation completed successfully, let’s try importing Pytorch in python3. Do not run the following commands while you’re in Pytorch's installation directory, as that’s likely to yield import errors. To test your installation:

```
cd 
python3
import torch
print(torch.__version__)
```

## Another comments
*	You can change the version of python but that makes pip error easier. So changing python version is not recommended (because of that, I reinstall OS about 4 times...)
*	If you need to use downgraded python version (like 3.5) you can use old version raspbian here (http://downloads.raspberrypi.org/raspbian/images/) but I recommend recent version
*	Using sudo drop or only keep some current environment variables. If you can, using sudo is not recommended. But if you must need to use ‘sudo’, you can try sudo –E to force sudo to preserve environment variables
*	If you need to install v1.1.0, you can follow this tutorial
https://nmilosev.svbtle.com/compling-arm-stuff-without-an-arm-board-build-pytorch-for-the-raspberry-pi
*	Sometimes when you use pip install command, SSL error can be evoked. you can resolve this problem with making sites which pip use trusted site by adding --trusted-host ‘site url’
Whenever I meet SSL error, I used this command and it works almost for ‘pip install’ 
‘--trusted-host pypi.org --trusted-host files.pythonhosted.org --trusted-host pypi.python.org --trusted-host piwheels.org’

## References
* [tutorial of how to install pysyft on raspberry pie (include pytorch)](https://blog.openmined.org/federated-learning-of-a-rnn-on-raspberry-pis/)

* [Build Caffe2](https://caffe2.ai/docs/getting-started.html?platform=raspbian&configuration=compile)
* how to resolve error message
undefined reference to __atomic_fetch_add_8' /usr/bin/ld: /home/pi/pytorch/build/lib/libcaffe2.so: undefined reference to __atomic_load_8' -> [Issues in pytorch repository](https://github.com/pytorch/pytorch/issues/22898)

* how to resolve ‘undefined reference to `__atomic_fetch_add_8' after install -> [Issues in pytorch repository](https://github.com/pytorch/pytorch/issues/22564)
