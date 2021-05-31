# Environment Setup (Docker + Tensorflow + Mxnet + GPU)

## 前言
先前的經驗告訴我，建個 GPU 訓練環境是很麻煩的事  

最近想學習一下 Computer Vision，是時候再來弄個環境  
原先在猜，如果用 container 的方式來做說不定會比較簡單，也容易搬到別的地方部屬  

結果，還是有不少眉眉角角的地方要注意  
目前看來，應該有比較好 deploy 吧？


## Pre-installation
簡單說明安裝前的環境
* Ubuntu 18.04 LTS
* Docker version 19.03.11
* GPU (1070ti)

```
接下來，文章中會用 Host 代表所謂的 Localhost (本機)， Guest 代表 Container
我不確定這用法是否合適，若有更好的說法，請再指教 :)
```

可以看到，Host 還沒有裝 `nvidia-driver` 跟 `CUDA`  
實際上，Host 不用刻意去裝 CUDA。之後直接 pull 別人處理好的 image 就行。  
除非，你想在 Host 也有個可以使用 GPU training 的環境。

`Docker`的部份請參考 https://docs.docker.com/engine/install/ubuntu/  
除非像敝司的網路環境，什麼都要設定 Proxy，不然這官網這一頁就很夠了

## Installation
### Nvidia Driver
1. 先檢查是否有 `ubuntu-drivers-common` 這個工具 (我的環境是已安裝)  
    沒有的話，請先安裝

```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update
sudo apt install ubuntu-drivers-common
```
2. 用 `ubuntu-drivers-common` 檢查 Driver 狀態

```
ubuntu-drivers devices
```
```
== /sys/devices/pci0000:00/0000:00:03.1/0000:1c:00.0 ==
modalias : pci:v000010DEd00001B82sv000019DAsd00002445bc03sc00i00
vendor   : NVIDIA Corporation
model    : GP104 [GeForce GTX 1070 Ti]
driver   : nvidia-driver-415 - third-party free
driver   : nvidia-driver-418 - third-party free
driver   : nvidia-driver-390 - distro non-free
driver   : nvidia-driver-440 - third-party free recommended
driver   : nvidia-driver-435 - distro non-free
driver   : nvidia-driver-410 - third-party free
driver   : xserver-xorg-video-nouveau - distro free builtin
```
    實際上，以現在時間(2020-06-12)最新的版本是 `nvidia-driver-450`  
    但是根據清單，我選擇清單中出現的 `nvidia-driver-440`

    裝完後，重新開機
```
sudo reboot
```

### Nvidia Container Runtime
3. 接著安裝 `nvidia-container-runtime`

```
sudo apt install nvidia-container-runtime
```

### Build Docker Image
4. 以下是我的 Dockerfile (也可以 pull 後再修改)

    ```
    FROM tensorflow/tensorflow:latest-gpu-py3

    ENV LC_ALL C.UTF-8
    ENV LANG C.UTF-8
    ENV LANGUAGE C.UTF-8

    RUN apt-get update && apt-get install -y libsm6 libxext6 libxrender-dev 

    RUN useradd --create-home --shell /bin/bash user
    WORKDIR /home/user

    RUN python3 -m pip install -U pip setuptools
    RUN python3 -m pip install jupyterlab gluoncv opencv-python pandas numpy ensemble-boxes
    RUN python3 -m pip install mxnet-cu101
    ```

    **注意 base image (tensorflow/tensorflow:latest-gpu-py3) 中的 `CUDA` 版本**
    檢查 (tensorflow, CUDA) & (mxnet, CUDA) 版本對應是否正確  

    原則上，從 tensorflow 抓的 image， tensorflow 跟 CUDA 的版本是沒有問題的  
    但我還想使用 mxnet， 所以要去確認 image 的 CUDA 版本  

    run container，到 `/usr/local` 確認 CUDA 版本  
    我這裡是 cuda-10-1，所以安裝 mxnet-cu101  

    p.s. 不曉得為什麼，在 container 用 `nvidia-smi` 測試，顯示是 10.2

5. 在 Dockerfile 同一個資料夾下 build image

```
docker build -t <IMAGE NAME> .
```

### Verification
6. 借 tensorflow 跟 mxnet 官方文件的指令來測試，GPU環境是否一切安好
    * tensorflow

    ```
    docker run --gpus all -it --rm <IMAGE NAME> \
    python -c "import tensorflow as tf; print(tf.reduce_sum(tf.random.normal([1000, 1000])))"
    ```
    * mxnet
        * Launch the container
    
    ```
    docker run --gpus all -it --rm <IMAGE NAME> bash
    ```
        * Start the python terminal and test it

    ```
    python
    ```
    ```python
    >>> import mxnet as mx
    >>> a = mx.nd.ones((2, 3))
    >>> b = a * 2 + 1
    >>> b.asnumpy()
    array([[ 3.,  3.,  3.],
            [ 3.,  3.,  3.]], dtype=float32)
    ```
    沒有 error，就沒事啦

### Launch with jupyter lab
7. enjoy

```
docker run --gpus all -p 8888:8888 -u user:user -v $(pwd):/home/user <IMAGE NAME> \
jupyter lab --ip=0.0.0.0 --port=8888 --notebook-dir=/home/user
```


## References
* https://www.celantur.com/blog/run-cuda-in-docker-on-linux/
* https://www.tensorflow.org/install/docker#examples_using_gpu-enabled_images
* https://gitpress.io/@chchang/install-nvidia-driver-cuda-pgstrom-in-ubuntu-1804
