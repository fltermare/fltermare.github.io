# Docker - Local Registry


## Intro
上線環境常常連不到外網，更不用提去 Dockerhub 拉 image 來用。通常是在內網建個公司用的 registry server，一來是降低連線到外網的風險、二來是能檢驗公司內用的 image 是否合規。
</br>  
這篇的目的在於，**在本地建立個測試用的 registry server** ，方便我自己測試 pull image 的過程是否有問題，畢竟公司的 registry server 也不是想測就測的。
</br>  
然後，發現 docker doc 其實寫得很完整，所以節錄自己用到的部分，也記錄之後要去哪裡找資料。


## Deploy a local registry server
```bash
docker run -d -p 5000:5000 --restart=always --name registry registry:2
```

## Copy an image from Docker Hub to your registry
1. Pull the ubuntu:16.04 image from Docker Hub
    ```bash
    docker pull ubuntu:16.04
    ```
2. Tag the image as `localhost:5000/my-ubuntu`
    ```bash
    docker tag ubuntu:16.04 localhost:5000/my-ubuntu
    ```
3. Push the image to the local registry running at `localhost:5000`
    ```bash
    docker push localhost:5000/my-ubuntu
    ```
4. Pull the localhost:5000/my-ubuntu image from your local registry
    ```bash
    docker pull localhost:5000/my-ubuntu
    ```


## Stop a local registry
```bash
docker container stop registry
```

```bash
docker container stop registry && docker container rm -v registry
```


## Reference
* https://docs.docker.com/registry/deploying/

