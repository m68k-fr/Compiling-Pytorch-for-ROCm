# Compiling Pytorch for ROCm on Ubuntu22.04


### Install needed dependencies

* ROCm must be installed, (use the **rocminfo** command to check your version)  
* If ROCm is missing, please refer to this guide for instructions: [Auto1111-AMD-ROCm](https://github.com/m68k-fr/Auto1111-Shark-Ubuntu-AMD-Howto)


````
sudo apt install clang
sudo apt install cmake
sudo apt install libjpeg-dev python3-dev
````


### Setting pytorch project

This will clone the pytorch project on your home folder, create and assign it a venv, and tune it for a ROCm compilation.  

````
cd ~
git clone --recursive -b v2.0.1 https://github.com/pytorch/pytorch
cd pytorch
python -m venv venv
source venv/bin/activate
python -m pip install --upgrade pip wheel
pip install -r requirements.txt
pip install mkl mkl-include
make triton
# Tune the project for a ROCm compilation
python tools/amd_build/build_amd.py
````

### Building pytorch

Compile the the latest pytorch release, install it in your pytorch venv, and build a whl package.

````
# set CMAKE_PREFIX_PATH
python -c "import site; print(site.getsitepackages()[0])"
# Replace '/home/username/pytorch/venv/lib/python3.10/site-packages' by the path reported by the previous command.
export CMAKE_PREFIX_PATH=/home/username/pytorch/venv/lib/python3.10/site-packages
BUILD_TEST=0 USE_CUDA=0 USE_CUDNN=0 USE_ROCM=1 python setup.py install
BUILD_TEST=0 USE_CUDA=0 USE_CUDNN=0 USE_ROCM=1 python setup.py bdist_wheel
````
A whl pytorch package file should be available in the dist folder.


### Getting and building vision

Clone the torchvision latest release for your pytorch version, compile it and build a whl package.  
Please note, we're using the venv from the main pytorch project compiled in the previous step.

To identify the compatible vision release for your pytorch version, use this chart:  
https://github.com/pytorch/pytorch/wiki/PyTorch-Versions

````
cd ~/pytorch
source venv/bin/activate
cd ~
git clone --recursive b- v0.15.2 https://github.com/pytorch/vision
cd vision
BUILD_TEST=0 USE_CUDA=0 USE_CUDNN=0 USE_ROCM=1 python setup.py bdist_wheel
````

A whl torchvision package file should be available in the dist folder.


### Updating Auto1111 to use our compiled pytorch 

````
cd ~/stable-diffusion-webui
source venv/bin/activate
# Uninstall the old torch packages
pip uninstall torch torchvision
# Install your torch compiled packages
pip install ~/pytorch/dist/torch-your-torch-compiled-package-name.whl
pip install ~/vision/dist/torchvision-your-torchvision-compiled-package-name.whl
````


### Reverting Auto1111 to use official ROCm 5.4.2 torch packages
````
cd ~/stable-diffusion-webui
source venv/bin/activate
# Uninstall the old torch packages
pip uninstall torch torchvision
# Install torch official packages
pip install torch torchvision --index-url https://download.pytorch.org/whl/rocm5.4.2
````


