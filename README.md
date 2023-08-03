# OpenSearch multi node cluter in Docker
- Local의 docker 환경에서 구성해보기 + 기존 Custer에 datanode 추가로 Connnect 해보기
해당 구성환경은 Mac M2 ARM 환경에서 테스트 되었음을 미리 말씀드립니다.

## Cluster 구성
---
#### OpenSearch Cluster
- 총 4개의 Node
  - cluster_manager node -- 1 개
  - data node -- 2 개
  - coordinator node -- 1 개
#### Opendashboard 
- 추가예정
---

## 설치 환경
- OS : ubuntu:20.04
- OpenSearch : 2.9.0
- Opendashboard : 추가예정

## Requirements
- Docker
- docker-compose
- Elasticvue
  -  Chrome 확장 앱 Download [Link]
---
---

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

6. curl 로 Opensearch REST API 확인
  ```shell
  $ curl -XGET http://localhost:9200
  
  {
  "name" : "opensearch-cluster_manager",
  "cluster_name" : "opensearch-cluster",
  "cluster_uuid" : "5wPrtmB_RHSqvwp_UdyqfA",
  "version" : {
    "distribution" : "opensearch",
    "number" : "2.9.0",
    "build_type" : "tar",
    "build_hash" : "1164221ee2b8ba3560f0ff492309867beea28433",
    "build_date" : "2023-07-18T21:22:48.164885046Z",
    "build_snapshot" : false,
    "lucene_version" : "9.7.0",
    "minimum_wire_compatibility_version" : "7.10.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "The OpenSearch Project: https://opensearch.org/"
  }
  ```
---

### add nodes (docker network)
- Node를 추가하는 환경을 구성하려면 **[Quick Start 링크]** 를 수행하기전에 1. Docker Network Create 를 먼저 수행 후 진행을 권장한다.
기존 Cluster에 추가적으로 Node를 추가로 연결할 수 있다. 아래의
내용은 기존의 datanode 2개에서 3개로 늘리는 과정이다.

1. Docker Network Create
  - 같은 호스트 내에서 실행중인 컨테이너 간 연결할 수 있도록 돕는 논리적 네트워크이다.
    - 해당 docker-compose.yml 에서는 네트워크 이름을 **opensearch-cluster-network** 라고 설정이 되어있다.
    ```shell
    # opensearch-cluster-network 생성
    $ docker network create opensearch-cluster-network
    
    # docker network 리스트에 생성된 것을 확인
    % docker network ls
    NETWORK ID     NAME                         DRIVER    SCOPE
    ************   bridge                       bridge    local
    ************   host                         host      local
    ************   opensearch-cluster-network   bridge    local
        ...                  ...                 ...       ...
    ```
2. Docker Run
  - 기존에 docker-copose up 을 진행했다면 mydockername/opensearch:1.0 Image 가 생성되어있다.
    ```shell
    # docker 이미지 리스트 확인
    $ docker images
    
    REPOSITORY                TAG       IMAGE ID       CREATED        SIZE
    mydockername/opensearch   1.0       ************   37 hours ago   3.24GB
    ```
  - Docker run datanode3 생성 터미널에 입력
    ```shell
    docker run -it \
    --name datanode3 \
    --network opensearch-cluster-network \
    -h datanode3 \
    -u worker \
    -p 9204:9200 \
    -p 9304:9300 \
    -p 9254:9250 \
    -p 9604:9600 \
    mydockername/opensearch:1.0
    ```
    
    ```shell
    # docker container 접속확인
    worker@datanode3:~$
    ```
    실행 중인 컨테이너중 "datanode3" 있는지 확인
    ```shell
    $ docker ps -a | grep datanode3
    ```
    
3. Opensearch Start
    - Opensearch Config Setting
      ```shell
      $ vi $OPENSEARCH_HOME/config/opensearch.yml
      ```
      
      ```shell
      # $OPENSEARCH_HOME/config/opensearch.yml
      plugins.security.disabled: true
      cluster.name: opensearch-cluster
      node.name: opensearch-d3
      node.roles: [ data, ingest ]
      network.host: 0.0.0.0
      discovery.seed_hosts: [manager_node]
      ```

   - OpenSearch Start
     ```shell
     $ sudo systemctl start opensearch.service
     ```

   - Status Follow
     로그확인
     ```sheell
     $ tail -f $OPENSEARCH_HOME/logs/opensearch-cluster.log
     ```
     
4. Elasticvue Node 확인
   ADD Cluster > http://localhost:9203 > CONNECT > NODES
   ![스크린샷 2023-08-04 오전 12 00 52](https://github.com/Themath93/opensearch-cluster/assets/108844287/2ddd6e95-0791-4897-9227-81e8077ffa42)

     
      

[Quick Start 링크]:[###-Quick-Start]
[Link]: https://chrome.google.com/webstore/detail/elasticvue/hkedbapjpblbodpgbajblpnlpenaebaa
