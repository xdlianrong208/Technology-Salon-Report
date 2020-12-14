 
# IPFS (InterPlanetary File System)，星际文件系统

## IPFS简介

IPFS 是一种内容可寻址、版本化、点对点超媒体的分布式存储、传输协议。

1. 内容可寻址
    
    现有的网络体系中，内容是基于位置（IP）寻址的，就是在查找内容的时候，需要先找到内容所在的服务器（根据 IP），然后再在服务器上找对应的内容。如果服务器宕机或遭到攻击，将无法访问内容。而且，URL提供不同的文件，或者中途遭到了篡改，都无法保证内容的正确性。

    内容寻址，基于内容标识符(CIDS)查找文件，CIDs作为文件的数字指纹。用这种方式寻址文件解决了位置寻址的问题。当客户端需要一个文件时，他们向网络中的节点询问具有特定CID的文件，而不是向一个服务器询问URL。客户端下载文件后，便会自己对其进行指纹识别。

2. 分布式存储

    分布式存储系统，是将数据分散存储在多台独立的设备上。传统的网络存储系统采用集中的存储服务器存放所有数据，存储服务器成为系统性能的瓶颈，也是可靠性和安全性的焦点，不能满足大规模存储应用的需要。分布式网络存储系统采用可扩展的系统结构，利用多台存储服务器分担存储负荷，利用位置服务器定位存储信息，它不但提高了系统的可靠性、可用性和存取效率，还易于扩展。

## 工作原理

1.  分布式哈希表 (DHT)

    全网维护一个巨大的文件索引哈希表，哈希表的条目形如 <key, value>。Key为某种算法的哈希值，Value则是存储文件的IP地址。查询时，仅需要提供Key,就能从表中查询到存储节点的地址并返回给查询节点。此哈希表被分割成小块，按照一定的算法和规则分布到全网各个节点上。每个节点只需要维护一小块哈希表。节点在查询文件时，仅需要把查询报文路由到相应的节点即可。

    - Kademlia DHT
    - CORAL DSHT
    - S/KADEMLIA DHT

    Kademlia DHT，节点的ID与关键字是同样的值域，都使用SHA-1算法生成160位摘要。新来的网络节点在初次连接网络时，会被分配一个ID，每个节点自身维护一个路由表和DHT，路由表保存网络中一部分节点的连接信息，DHT用于保存文件信息；每个节点优先保存距离自己更近的节点信息，但一定确保距离在 [2^n, 2^(n+1)-1]的全部节点至少保存k个。

2. 块交换协议 (BitTorrent)

    - torrent: 服务器接受的元数据文件。记录下载数据的信息，如文件名、文件大小、文件的哈希值，以及Tracker的URL地址。
    - tracker: 互联网上负责协调BitTorrent客户端行动的服务器。
    - peer: 互联网上的另一台可以连接并传输数据的计算机。
    - seed: 有一个特定torrent完整拷贝的计算机。

    新文件的发行，需要从seed开始进行初次分享。首先，seed会生成一个torrent，包含文件名、大小、tracker的URL。tracker保存文件信息和seed的连接信息，而seed保存文件本身。通过torrent文件，peer会访问tracker，获取其他peer/seed的连接信息，并与peer/seed沟通，建立连接下载文件。

3. 版本控制 (Git)

---

## IPFS与区块链

### IPFS能为区块链带来的改变

—— 改善区块链的存储压力

在目前的区块链中，存储代价非常高。以以太坊为例，如果想要存储1M的数据，如果全网有一万个矿工在记账，那么全网的存储开销则为10G。而且，在以太坊上如果想要存储数据，所要付出的gas的价格也是一份昂贵的。采用IPFS技术存储链上数据，将永久可用的IPFS地址放入区块中，可以大大减轻存储的压力，提高存储效率。

### Filecoin

Filecoin作为IPFS的激励层，是基于区块链的分布式存储协议。基于IPFS的应用有着巨大的数据存储和节点数量需求，IPFS作为P2P网络，节点越多下载速度越快。没有激励机制，很难保证IPFS网络会长久的运行。Filecoin中有两种矿工，检索矿工和存储矿工。
- 检索矿工向网络提供数据检索服务，通过与用户数据下载进行交易获取用户支付的数据下载费用。实际上是在销售自己的网络带宽。
- 存储矿工通过抵押一部分代币向网络提供可出售的存储空间。在存储空间被用户购买后获取用户支付的交易费用。

---

## IPFS的安装部署（私有集群）



### 利用docker部署

1. 环境

    - golang
    - docker
    
    生成搭建私有网络所需的swarm.key密钥
    
    ```
    go get github.com/Kubuxu/go-ipfs-swarm-key-gen
    cd $GOPATH
    go build
    ./ipfs-swarm-key-gen > swarm.key
    ```

2. 创建挂载目录：

    ipfs 的 export 和 data 目录挂载到主机上，这里做下准备，每个不同是私链节点的挂载目录不一样。

    以node0为例

    /home/cjz/Docker/IPFS/node0/export

    /home/cjz/Docker/IPFS/node0/data


3. 在docker上部署

    在docker上搜索ipfs的镜像

    ```
    docker search ipfs
    ```

    拉取镜像
    ```
    docker pull ipfs/go-ipfs
    ```
    启动容器，并创建节点
    ```
    docker run --name ipfs-node-0 -v /home/cjz/Docker/IPFS/node0/export:/export -v /home/cjz/Docker/IPFS/node0/data:/data/ipfs -p 10000:4001 -p 11000:5001 -p 12000:8080 -d ipfs/go-ipfs:latest
    ```
    查看日志
    ```
    docker logs -f ipfs-node-0
    ```
    日志情况：
    ```
    Changing user to ipfs
    ipfs version 0.7.0
    generating ED25519 keypair...done
    peer identity: 12D3KooWKKsuNQpDteX6Xk5949kT67B5kTeZARpt4MBRbBseG3n4
    initializing IPFS node at /data/ipfs
    to get started, enter:

    	ipfs cat /ipfs/QmQPeNsJPyVWPFDVHb77w8G42Fvo15z4bG2X8D2GhfbSXc/readme

    Initializing daemon...
    go-ipfs version: 0.7.0-ea77213
    Repo version: 10
    System version: amd64/linux
    Golang version: go1.14.4
    Swarm is limited to private network of peers with the swarm key
    Swarm key fingerprint: d11e26ed1fe5b43f743eaf27f74d704a
    Swarm listening on /ip4/127.0.0.1/tcp/4001
    Swarm listening on /ip4/172.17.0.2/tcp/4001
    Swarm listening on /p2p-circuit
    Swarm announcing /ip4/127.0.0.1/tcp/4001
    Swarm announcing /ip4/172.17.0.2/tcp/4001
    API server listening on /ip4/0.0.0.0/tcp/5001
    WebUI: http://0.0.0.0:5001/webui
    Gateway (readonly) server listening on /ip4/0.0.0.0/tcp/8080
    Daemon is ready
    ```

    出现“Swarm is limited to private network of peers with the swarm key”，代表使用了私有密钥，属于私有节点。
    "Daemon is ready"表示ipfs已经启动。


    删除公有集群的引导节点

    ```
    docker exec ipfs-node-0 ipfs  bootstrap rm --all
    ```
    单个节点启动完成。

    另起一个节点，创建方式与单个节点的方式相同。注意端口号，文件路径需要更改。    

    ```
    docker run --name ipfs-node-1 -v /home/cjz/Docker/IPFS/node1/export:/export -v /home/cjz/Docker/IPFS/node1/data:/data/ipfs -p 10001:4001 -p 11001:5001 -p 12001:8080 -d ipfs/go-ipfs:latest
    ```

    查看节点，打开节点的shell，在node0中可以看到node1的identity。
    ```
    docker exec -it ipfs-node-0 /bin/sh
    / # ipfs swarm peers
    /ip4/172.17.0.3/tcp/4001/p2p/12D3KooWFAWPkrCxHdTnM4KeuVVUD1hA8uaxUYDdefNEE1aNwKvu
    ```
    至此，部署完毕。
    
    测试：node0发布文件，node1通过哈希值获取文件
    ```
    docker exec ipfs-node-0 ipfs add /data/ipfs/version
    ```
    log:
    ```
    QmUnKvaCFaE8DSHMAq2XeHfrz1wW62uVbb4dNtxQ5zXy6n version 3 B / 3 B  100.00%
    ```
    在node1上获取
    ```
    docker exec ipfs-node-1 ipfs cat QmUnKvaCFaE8DSHMAq2XeHfrz1wW62uVbb4dNtxQ5zXy6n
    ```
    log:
    ```
    10
    ```
    测试成功。


### 在ubuntu虚拟机中部署
1. 为各个虚拟机分配不同的IP地址（静态IP），参考博客： https://www.cnblogs.com/yyee/p/12899953.html

2. 在官网上，下载ipfs安装包。下载地址： https://dist.ipfs.io/go-ipfs/ 

    由于特殊原因，网站需要科学上网，也可以从我的网盘获取go-ipfs的安装包(version: v0.7.0)。网盘地址：https://wws.lanzous.com/i0GjXjamlbi

3. 解压安装包，在go-ipfs目录下运行如下指令
    ```
    sudo ./install.sh
    ```
4. 初始化及相关配置
    
    在根目录运行如下指令
    ```
    ipfs init
    ```
    执行完此条命令之后，ipfs会在本地的根目录下创建一个名为.ipfs的隐藏文件夹，运行如下指令，即可看到新创建的文件夹
    ```
    ls -f
    ```
5. 搭建私有节点

    搭建私有集群需要所有的节点使用同一个swarm.key私有密钥。
    生成私有密钥需要配置go语言环境，如果虚拟机里没有go语言环境，可以选择在有go语言的环境中，生成swarm.key，再复制到节点的目录中也是一样的。
    - 从github上获取生成私有密钥的代码，并运行生成私有密钥。
    ```
    go get -u http://github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen
    cd $GOPATH
    cd src/github.com/Kubuxu/go-ipfs-swarm-key-gen/ipfs-swarm-key-gen/
    go build
    ./ipfs-swarm-key-gen > ～/swarm.key 
    ```
    “>”后是生成目录，可以任意选择，这里直接生成到了根目录下，方便之后copy到别的虚拟机中，也可以直接生成到ipfs的目录下。

    - 将swarm.key移动到ipfs的目录下
    
    ```
    sudo mv swarm.key ~/.ipfs
    ```
    - 移除默认的公有集群引导节点
    ```
    ipfs bootstrap rm --all
    ```
    - 启动服务
    ```
    ipfs daemon
    ```
    日志中出现“Swarm is limited to private network of peers with the swarm key”，代表创建的是私有节点。

6. 搭建私有网络集群
       
    首先，需要保证节点的ipfs服务在运行状态，即“ipfs daemon”指令在运行中。

    然后，在要连接的目标节点中，通过如下指令，查询该节点的id。
    ```
    ipfs id
    ```
    最后，在私有节点中，手动添加其它节点的bootstrap信息。
    ```
    ipfs bootstrap add /ip4/192.168.23.101/tcp/4001/ipfs/12D3KooWG8oq9bC6WWsuUuvzUjNF6vPpjkoWmhTpQonJPfZbCRosB
    ```
    其中，/ip4/后面的ip地址是要连接的目标节点的ip地址，可以通过指令：ip address在目标节点中查看。
    
    /ipfs/后是目标节点的ipfs的id。

7. 验证与测试

    在节点中输入如下指令，查询当前连接的节点。
    ```
    ipfs swarm peers
    ```

    测试在一个节点创建文件并添加到ipfs网络，其它节点接收。

    - 创建文件
        ```
        echo helloworld > test.txt
        ```

    - 添加到ipfs网络
        ```
        ipfs add test.txt
        ```
        返回信息，其中包括以Qm开头的哈希值，复制哈希值。

    - 通过哈希值获取文件
    
        在另一个节点中，通过文件的哈希值，获取文件。

        ```
        ipfs cat <Qm开头的哈希值>
        ```

        运行过后，打印出helloworld，代表搭建成功。

8. 注意事项

    如果发现ipfs节点没有正常连接，首先检查各个节点的ipfs服务是否正在运行状态，如果运行状态正常，检查是否正确的设置了虚拟机的静态ip地址。

