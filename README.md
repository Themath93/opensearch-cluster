# OpenSearch 멀티 클러스터 Local의 docker 환경에서 구성해보기
ver 1.0

## Cluster 구성
#### OpenSearch Cluster
- 총 4개의 Node
  - cluster_manager node -- 1 개
  - data node -- 2 개
  - coordinator node -- 1 개
#### Opendashboard 
- 추가예정

## 설치 환경
- OS : ubuntu:20.04
- OpenSearch : 2.9.0
- Opendashboard : 추가예정

## Requirments
- Docker
- docker-compose
- Elasticvue
  -  Chrome 확장 앱 Download [Link]

### Quick Start

1. opensearch image 생성
   - Windows
    ```shell
    docker build ./x64 -t mydockername/opensearch:1.0 
    ```
   - Mac m1
    ```shell
    docker build ./arm64 -t mydockername/opensearch:1.0 
    ```
2. image 생성확인
    ```shell
    $ docker image ls
    REPOSITORY                TAG       IMAGE ID       CREATED          SIZE
    mydockername/opensearch   1.0       757d4401b416   27 minutes ago   3.24GB
    ``` 
3. docker-compose 로 container group build
  ```shell
  docker-compose up -d
  ```

4. container group 확인
    ```shell
    % docker container ls
    CONTAINER ID   IMAGE                         COMMAND                  CREATED          STATUS          PORTS                                                                                            NAMES
    7100b95bce03   mydockername/opensearch:1.0   "/bin/bash -c ' echo…"   26 minutes ago   Up 26 minutes   0.0.0.0:9202->9200/tcp, 0.0.0.0:9252->9250/tcp, 0.0.0.0:9302->9300/tcp, 0.0.0.0:9602->9600/tcp   datanode2
    365703d9f41b   mydockername/opensearch:1.0   "/bin/bash -c ' echo…"   26 minutes ago   Up 26 minutes   0.0.0.0:9203->9200/tcp, 0.0.0.0:9253->9250/tcp, 0.0.0.0:9303->9300/tcp, 0.0.0.0:9603->9600/tcp   coordinator
    fb68429f7501   mydockername/opensearch:1.0   "/bin/bash -c ' echo…"   26 minutes ago   Up 26 minutes   0.0.0.0:9201->9200/tcp, 0.0.0.0:9251->9250/tcp, 0.0.0.0:9301->9300/tcp, 0.0.0.0:9601->9600/tcp   datanode1
    42065f0a0ee0   mydockername/opensearch:1.0   "/bin/bash -c ' echo…"   26 minutes ago   Up 26 minutes   0.0.0.0:9200->9200/tcp, 0.0.0.0:9250->9250/tcp, 0.0.0.0:9300->9300/tcp, 0.0.0.0:9600->9600/tcp   manager_node
    ```
    ![스크린샷 2023-08-02 오전 10 44 02](https://github.com/Themath93/opensearch-cluster/assets/108844287/a8679f69-33b1-4f00-b0ef-66a4f76f546c)

5. elasticvue 로 노드 상태확인
  - ADD CLUSTER : http://localhost:9203 --> Coordinator 를 관찰 
    ![스크린샷 2023-08-02 오전 10 46 19](https://github.com/Themath93/opensearch-cluster/assets/108844287/b8c03c85-d2aa-4ebc-bae1-bc5985de2bfe)
    - manager_node 가 Fail 시 투표로 datanode 2 개중 1개가 선출과정을 Coordinator port를 바라봐야 놓치지 않는다.
  - HOME > NODE > GRID
    ![스크린샷 2023-08-02 오전 10 47 42](https://github.com/Themath93/opensearch-cluster/assets/108844287/f8020dec-19c6-405a-8d0b-daa60bb6a2f7)


[Link]: https://chrome.google.com/webstore/detail/elasticvue/hkedbapjpblbodpgbajblpnlpenaebaa
