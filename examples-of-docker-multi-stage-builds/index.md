# Examples of Docker Multi-stage Builds






## Purpose
前陣子，在把專案製作成 docker image 的過程，想到幾個問題 :
1. 製作出的 image 理所當然要包含專案系統的功能。但是有需要包含 testing 的部分嗎 ?
2. 不需要包含 testing 程式碼的話，要怎樣才能控制 docker build 的流程 ?
3. 先不論程式碼，有辦法維護一個 Dockerfile 就同時符合 testing/production/... 多個需求嗎?

搜尋相關資訊後，看起來解法就是 **"docker multi-stage builds"** (Docker 17.05 later)  
以往同樣的程式要製作成不同用途 image (for testing, for production)，通常需要多個 Dockerfiles，透過 multi-stage builds 可以做到 :
1. 控制建構流程，達到一個 Dockerfile 可產生不同目的的 image *&leftarrow;我的主要需求*
2. 可以只複製某個 stage 的中間結果 &rightarrow; 不需要該 stage 的環境 &rightarrow; 節省 image 大小

## Examples

以下 2 個範例，內容差不多

### Example 1
```dockerfile
FROM alpine:3.12 AS base
WORKDIR /root/
RUN touch fileInit

FROM base AS testing
RUN touch fileTesting

FROM base AS production
RUN touch fileProduct
```
以上面這個 Dockerfile 而言，有 3 個 stages

* stage 0
    ```dockerfile
    FROM alpine:3.12 AS base
    WORKDIR /root/
    RUN touch fileInit
    ```
    * **`FROM [image] AS [name]`**
        * `FROM` 搭配 `AS` 可以命名該 stage
    * 這裡把 `alpine:3.12` 命名為 `base`
    * 建立一個 fileInit 檔案

* stage 1
    ```dockerfile
    FROM base AS testing
    RUN touch fileTesting
    ```
    * 把 `base` 當作 parent image，並將此 stage 命名為 `testing`
    * 建立一個 fileTesting 檔案
* stage 2
    ```dockerfile
    FROM base AS production
    RUN touch fileProduct
    ```
    * 把 `base` 當作 parent image，並將此 stage 命名為 `production`
    * 建立一個 fileProduct 檔案
    * 因為是把 `base` 當作 parent image，代表流程是 `base` &rightarrow; `production`，中間不包含 `testing`，代表製作的 image 不包含 `testing` 的資訊。  
      
        **&rightarrow; "fileTesting 不存在"**

* Build image - EX1
    ```bash
    # Production
    docker build -t multi1 --target production -f Dockerfile .
    docker build -t multi1 -f Dockerfile .

    # Test
    docker build -t multi1 --target testing -f Dockerfile .
    ```
    * 透過 `--target` 指定流程走到哪個 stage 結束
    * 若 `--target production`，代表從頭 (`base`) 走到尾 (`production`)
        * 詳細去看 log 的話，`testing` 也會被執行，只是以 [Example 1](#Example-1) 而言， image 不會包含 `testing` 的資訊
    * 若 `--target testing`，代表從 `base` 到 `testing`
    * 把 `touch file` 改成自己需要的指令 (python main.py | python -m pytest | ... )，就能完成一個多目的的 Dockerfile :
        1. 確保各用途的 image 都有一樣的環境
        2. stage `testing` 才把測試用的程式碼與測試資料放到 image 中，產生 `production` image 時，就不會包含測試的資料與程式碼

### Example 2
[Example 1](#example-1) 透過 `--target` 控制流程，[Example 2](#example-2) 則是透過 `--build-arg` 來達到目的。  
個人認為 [Example 2](#example-2) 較有彈性。專案可能越來越複雜，testing image跟 production image 不見得在最後才有差異。這種情況下，`--target` 不見得夠用

```dockerfile
ARG BUILD_TYPE='production'

FROM alpine:3.12 AS base
WORKDIR /root/
RUN touch fileInit

FROM golang:1.14.6 AS go-build
WORKDIR /go
COPY hello.go .
RUN go build -o hello .

FROM base AS testing
RUN touch fileTesting

FROM base AS production
RUN touch fileProduct

FROM ${BUILD_TYPE} as after

FROM after
RUN touch fileFinal
COPY --from=go-build /go/hello .
CMD [ "./hello" ]
```

* \-\-build-arg

    ![build flow](/img/docker_multistage_build_ex2.png)

    先不考慮 stage `go-build` 以及 `COPY` 指令的話  
    就如前面所說，是透過 `--build-arg` (`BUILD_TYPE`) 來控制建構流程，順序如下
    1. base
    2. testing | production       # 根據 BUILD_TYPE
    3. after


* COPY (stage 1 & stage 5)
    ```dockerfile
    FROM golang:1.14.6 AS go-build
    WORKDIR /go
    COPY hello.go .
    RUN go build -o hello .

    FROM after
    RUN touch fileFinal
    COPY --from=go-build /go/hello .
    CMD [ "./hello" ]
    ```
    * **`COPY --from=[stage name] ...`**
        * 從指定的 stage A 複製特定檔案到該 stage B
    * (line: 8) 複製 stage `go-build` 產生的可執行檔 `hello` 到 stage `after` 中
    * stage `after` 需要執行 `hello`，因此對使用者而言，stage `go-build` 唯一需要保留的就是`hello`。透過 multi-stage build，最終產生的 image 不會包含 stage `go-build` 的 go 環境
        * 減少 image 大小

* Build image - EX2
    ```bash
    # Production
    docker build -t multi2 --build-arg BUILD_TYPE=production -f Dockerfile .
    docker build -t multi2 -f Dockerfile .

    # Test
    docker build -t multi2 --build-arg BUILD_TYPE=testing -f Dockerfile .
    ```
    * 類似 [Build image - EX1](#build-image---ex1)，只是改由 `--build-arg` 來指定流程

## Conclusion
基本上，知道下面兩個指令的作用
1. `FROM [image] AS [name]` 
2. `COPY --from=[stage name] ...`

搭配 docker build 的兩個參數
1. `--target`
2. `--build-arg`

就能看懂用 multi-stage 的 Dockerfile，
剩下就是從簡單的範例改出自己需要的 Dockerfile ~

## Reference
* https://docs.docker.com/develop/develop-images/multistage-build/
* https://www.docker.com/blog/advanced-dockerfiles-faster-builds-and-smaller-images-using-buildkit-and-multistage-builds/
* https://kkc.github.io/2018/04/28/docker-note/
* https://jiepeng.me/2018/06/09/use-docker-multiple-stage-builds
* https://tachingchen.com/tw/blog/docker-multi-stage-builds/
