# Instructions for setting up a Ubuntu 16.04 ec2 instance with caffe

## Note: Be sure to kick this instance off with at least 30GB Storage in the main partition!

First check that there's space available. Run `lsblk` and ensure main partition has >30GB for Caffe, and enough for all the training data.

## Update the package lists

    sudo apt-get update

## Installing general dependencies

    sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler

    sudo apt-get install --no-install-recommends libboost-all-dev

## Installing CUDA

    mkdir downloads
    cd downloads
    wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.5-18_amd64.deb
    sudo dpkg -i cuda-repo-ubuntu1404_7.5-18_amd64.deb
    
    sudo apt-get upgrade -y

If prompted, select "Keep the package managers version"

    sudo apt-get install -y opencl-headers build-essential protobuf-compiler \
        libprotoc-dev libboost-all-dev libleveldb-dev hdf5-tools libhdf5-serial-dev \
        libopencv-core-dev  libopencv-highgui-dev libsnappy-dev \
        libatlas-base-dev cmake libstdc++6-4.8-dbg libgoogle-glog0v5 libgoogle-glog-dev \
        libgflags-dev liblmdb-dev git python-pip gfortran
    
    sudo apt-get install libprotobuf-dev libleveldb-dev libsnappy-dev libopencv-dev libhdf5-serial-dev protobuf-compiler
    sudo apt-get clean
    sudo apt-get install -y linux-image-extra-4.4.0-22-generic linux-headers-4.4.0-22-generic linux-image-4.4.0-22-generic
    sudo apt-get update
    sudo apt-get install -y cuda
    sudo apt-get clean

### Add CUDA to ~/.bashrc

    echo 'export PATH=$PATH:/usr/local/cuda-7.5/bin' >> ~/.bashrc 
    echo 'export LD_LIBRARY_PATH=/usr/local/cuda-7.5/lib64' >> ~/.bashrc 

    source ~/.bashrc

## Installing cuDNN

    wget https://s3-eu-west-1.amazonaws.com/poc-ocr-caffe/cudnn-7.0-linux-x64-v4.0-prod.tgz
    tar -zxf cudnn-7.0-linux-x64-v4.0-prod.tgz
    cd cuda
    sudo cp lib64/* /usr/local/cuda/lib64/
    sudo cp include/cudnn.h /usr/local/cuda/include/

## Installing Caffe

    cd caffe
    sudo apt-get install libgflags-dev libgoogle-glog-dev liblmdb-dev
    cp Makefile.config.example Makefile.config
    sed -i '/^# USE_CUDNN := 1/s/^# //' Makefile.config
    sed -i '/^# WITH_PYTHON_LAYER := 1/s/^# //' Makefile.config
    sed -i '/^PYTHON_INCLUDE/a    /usr/local/lib/python2.7/dist-packages/numpy/core/include/ \\' Makefile.config

And set up hdf5

    cd /usr/lib/x86_64-linux-gnu
    sudo ln -s libhdf5_serial.so.10.1.0 libhdf5.so
    sudo ln -s libhdf5_serial_hl.so.10.0.2 libhdf5_hl.so
    
    cd ~/caffe
    vim Makefile.config
    /INCLUDE_DIRS
    dd
    dd

Add the following in instead:

    i
    INCLUDE_DIRS := $(PYTHON_INCLUDE) /usr/local/include /usr/include/hdf5/serial/
    LIBRARY_DIRS := $(PYTHON_LIB) /usr/local/lib /usr/lib /usr/lib/x86_64-linux-gnu/hdf5/serial/
    
Esc, then shift+z, then shift+z to leave

## Build Caffe

    make all -j8
    make test -j8
    echo 'export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64' >> ~/.bashrc
    source ~/.bashrc
    make runtest

