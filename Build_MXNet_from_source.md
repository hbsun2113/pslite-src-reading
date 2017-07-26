# HOW TO Build MXNet from source

## Pre1: Install OpenBLAS from source

For this part, I mainly referred to [OpenBlas#Installation-from-source](https://github.com/xianyi/OpenBLAS#installation-from-source). For me, "Normal Compile" is enough.

**NOTE:** To avoid using sudo, install in my own directory.

## Pre2: Install OpenCV from source

I mainly referred to this article: [Installation in Linux](http://docs.opencv.org/2.4/doc/tutorials/introduction/linux_install/linux_install.html).

Also, a pdf version for this doc is provided [HERE](appendix/Installation_in_Linux-OpenCV_2.4.13.pdf)

Some tricks to boost installation:

1. Use zip instead of git repo.


**NOTE:** Avoid using `sudo`, install in my own directory.

## Pre3: Configure Environment Variables

Add the following snippet into `.bashrc` or `.bash_profile` in the home directory.

```sh
# Wen Ke's Environment for OpenBLAS, OpenCV and 

export OPENCV_HOME=$HOME/opencv-3.2.0
export OPENBLAS_HOME=$HOME/openblas-0.2.19

export PATH=$OPENBLAS_HOME/bin:$OPENCV_HOME/bin:$PATH
export C_INCLUDE_PATH=$OPENBLAS_HOME/include:$OPENCV_HOME/include:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=$OPENBLAS_HOME/include:$OPENCV_HOME/include:$CPLUS_INCLUDE_PATH
export LD_LIBRARY_PATH=$OPENBLAS_HOME/lib:$OPENCV_HOME/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$OPENBLAS_HOME/lib:$OPENCV_HOME/lib:$LIBRARY_PATH
export PKG_CONFIG_PATH=$OPENCV_HOME/lib/pkgconfig:$PKG_CONFIG_PATH
```

## Step1: Build MXNet from source

Just follow the [official installation guide](http://mxnet.io/get_started/install.html).

There is also a [pdf format](appendix/Installing_MXNet_mxnet_documentation.pdf).

Major steps are:

1. Use `wget` to download and extract MXNet zip. Current stable release is [mxnet-0.10.0.zip](https://codeload.github.com/dmlc/mxnet/zip/v0.10.0). I don't recommend cloning the repo since it is too large.
2. Build MXNet.

```sh
cd mxnet
make -j $(nproc) USE_OPENCV=1 USE_BLAS=openblas
```

## Step2: Install MXNet python package from source

### 2.1 Deploy a `virtualenv` environment

```shell
cd ~
virtualenv --no-site-packages venv
vim .bash_profile
```

Append the following snippet to  `.bash_profile` and update my bash with it.

```shell
source $HOME/venv/bin/activate
```

After that, my command prompt begins with a `(venv)` prefix.

### 2.2 Install MXNet

```shell
cd <mxnet_home>/python/
pip install --upgrade pip                      # upgrade pip
pip install -e .                               # -e equals --editable
```

### 2.3 Validate MXNet Installation

```python
>>> import mxnet as mx
>>> a = mx.nd.ones((2, 3))
>>> b = a * 2 + 1
>>> b.asnumpy()
array([[ 3.,  3.,  3.],
       [ 3.,  3.,  3.]], dtype=float32)
```



## Run MXNet on Multiple CPU/GPU's

See [Run MXNet on Multiple CPU/GPUs with Data Parallelism](http://mxnet.io/how_to/multi_devices.html) for more details.

Remember to enable the following switches:

```
USE_DIST_KVSTORE=1
```

## Appendix A: If the host machine cannot access github files

Due to some well-known reason in China, the host machine usually fails to `wget` some dependencies from GitHub CDN.

For that, we may need to

1. Find out the URLs from the output of `make`
2. Download them via Thunder or sth. else.
3. Upload these offline packages to the corresponding directories.
4. Modify the makefile along with its dependencies.

I have encountered a timeout problem when `wget`-ing from `https://raw.githubusercontent.com/mli/deps/master/build/*.tar.gz`. To avoid such problem, I've modified `ps-lite/make/deps.mk` so that the makefile only download the packages if they don't exist. 

The code is as follows:

```shell
# protobuf
PROTOBUF = ${DEPS_PATH}/include/google/protobuf/message.h
${PROTOBUF}:
	$(eval FILE=protobuf-2.5.0.tar.gz)
	$(eval DIR=protobuf-2.5.0)
	rm -rf $(DIR)
	@if [ ! -f "$(FILE)" ]; then echo -e "\n\e[33mOffline package does not exist. Start downloading ...\e[0m\n" && $(WGET) $(URL)/$(FILE); fi
	tar --no-same-owner -zxf $(FILE)
	cd $(DIR) && export CFLAGS=-fPIC && export CXXFLAGS=-fPIC && ./configure -prefix=$(DEPS_PATH) && $(MAKE) && $(MAKE) install
	rm -rf $(FILE) $(DIR)
```

NOTE: In general, the **current working directory** of a makefile is the directory where `makefile` or `Makefile` resides, even though it is run by another makefile. So, we should upload our offline packages to `ps-lite/` directory.

