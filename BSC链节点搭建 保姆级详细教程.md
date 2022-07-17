# BSC链节点搭建 保姆级详细教程

### 一，服务器配置要求
 - [官方建议配置](https://docs.binance.org/smart-chain/developer/fullnode.html#fullnode)
```
系统：Mac & Linux
CPU：16核
内存：64 GB 内存
带宽：50M以上
硬盘：大于2T固态SSD可用空间数据盘
```

 - 本次搭建使用的服务器配置
```
系统：Debian 11.1 (base)
CPU：16核心32线程
内存：128 GB DDR4 ECC
带宽：1 GBit/s
硬盘：2 个 3.84 TB NVMe SSD 数据中心版 (2 个 1.92 TB NVMe SSD 数据中心版 已经足够)
区域：HEL1 (Helsinki)
```
----


 - BSC官方文档：[https://docs.binance.org/smart-chain/developer/fullnode.html](https://docs.binance.org/smart-chain/developer/fullnode.html)
 - BSC快照github：[https://github.com/binance-chain/bsc-snapshots](https://github.com/binance-chain/bsc-snapshots)
 - BSC github地址：[https://github.com/binance-chain/bsc/releases](https://github.com/binance-chain/bsc/releases)
 

>  【注】以下命令 基于Debian，但是由于部分命令我并未操作（如：安装go） 故 自行替换下命令  安装命令    Debian：apt-get install ***    
> Centos： yum install *** -y 
> 如：安装 wget
>  Debian： apt-get install wget    
>  Centos： yum install wget -y

 
----

### 二，服务器环境配置
- **2.1 更新Debian/Centos软件包**

```bash
Debian: sudo apt update && sudo apt dist-upgrade
Centos: yum -y upgrade
```
- **2.2 安装wget  git  vim  unzip**

```bash
Debian:
apt-get install wget
apt-get install git
apt-get install vim
apt-get install unzip
apt-get install liblz4-tool    # 解压用
apt-get install screen   # 安装linux下的窗口管理器工具screen
```

- **2.3 Centos安装最新版Golang（安装 Go 主要是为了去编译 go-ethereum 源码，此步骤非必要，可以直接下载官方编译后的文件（文章下方有详细步骤））**
  
2.3.1 最新版本
https://golang.google.cn/dl/
2.3.2 下载安装
```bash
cd /usr/local/src	
wget https://golang.google.cn/dl/go1.17.7.linux-amd64.tar.gz
tar -zxvf go1.17.7.linux-amd64.tar.gz -C /usr/local/
```
2.3.3 增加配置文件
```bash
vim /etc/profile
```
【tips】
此时是vim的命令模式 我们需要在最后加入下面的代码 所以 vim的命令模式–>输入模式 的方法是按i键
即 按i键  把光标移动到最后 换行  输入以下内容
```bash
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=/opt/go
export PATH=$PATH:$GOPATH/BIN
```

然后我们需要冲从 输入模式–>命令模式
按 ESC  键
接下来保存内容并退出
需要冲当前的命令模式–>指令行模式
按:键  
再输入wq 回车即可
![vim修改内容保存退出](https://img-blog.csdnimg.cn/2b6dd377fed0412a9259731fb71a0a9d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAb2tleXplcm8=,size_20,color_FFFFFF,t_70,g_se,x_16)（截图仅供参考，并非我实操过程的截图）

更新环境变量
```bash
source /etc/profile
```

查看版本

```bash
go version
```
返回以下 代表没问题

```bash
[root@localhost ~]# go version
go version go1.17.7 linux/amd64
```


## 三、节点安装部署
在根目录创建jiedian文件夹用来存放节点程序，并在同时在jiedian里边创建一个kuaizhao文件夹，下载的快照数据

> 1.务必使用固态硬盘并且可使用空间大于2T
> 2.如果固态不够使用，可以把快照压缩包下载到机械硬盘里边，解压的时候解压到固态硬盘

创建文件夹
```bash
 cd /		#进入根目录
 mkdir -p jiedian/kuaizhao		#创建jiedian及kuaizhao文件夹
 cd /jiedian		#进入jiedian文件夹
 ```
- **安装BSC版本的geth**
 - [ ] 安装go的需要执行接下来的命令
```bash
 git clone https://github.com/binance-chain/bsc
 cd bsc/
 make geth     #亲测使用yum install golang -y 此处报错
```
 配置路径
```bash
export PATH=$PATH:/jiedian/bsc/build/bin
```


  - [ ] 下载预构建二进制文件后按照以下说明进行操作
  
从[发布页面](https://github.com/bnb-chain/bsc/releases/latest)下载预构建二进制文件（如不想安装go 可以下载官方编译后的二进制文件）

```bash
wget   $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest |grep browser_ |grep geth_linux |cut -d\" -f4)
```
  设置可执行权限及重命名为geth
```bash
chmod +x geth_linux mv geth_linux geth
```

>修改 `/etc/bashrc`，在文件末尾添加了如下路径，目的是将 /jiedian/geth 加入环境变量，方便使用
```bash
export PATH=/jiedian/:$PATH
```
更新环境变量
```bash
source /etc/profile
```



> 使用 `geth version` 确认安装正确

- 配置创世区块

```bash
wget https://github.com/bnb-chain/bsc/releases/download/v1.1.11/mainnet.zip
unzip mainnet.zip
geth --datadir node init genesis.json
```
- 下载BSC快照
创建一个用来下载快照的screen窗口

```bash
screen -S xiazai
```

> 注意1：使用screen窗口期间可以退出或者关闭命令行对话框。
> 注意2：退出当前窗口时用ctrl+ad（顺序按a和d字母即可），绝对不要用exit或ctrl+d退出会话。
> 注意3：退出会话后，可以用screen -x xiazai重新连接到会话。这样可以保持在shell下运行，网络中断不会影响。

- 开始下载快照

```c
cd /jiedian/kuaizhao		#进入kuaizhao文件夹下载快照
wget -O geth.tar.lz4 "最新下载地址"
# 例：
# 选择的 Asia Endpoint 的下载地址
# 如果硬盘大小不到2T 如1.92T 解压的时候 会硬盘内存不足 可以选择一边下载一边解压
# wget -q -O - <snapshot URL> | tar -I lz4 -xvf -  一边下载 一边解压 同时删除 -q 更方便查看进度
如:
# wget -O - "https://tf-dex-prod-public-snapshot-site1.s3-accelerate.amazonaws.com/geth-20220715.tar.lz4?AWSAccessKeyId=AKIAYINE6SBQPUZDDRRO&Signature=f9fLGGym3%2FQCOgu6cv%2BjOZDHZKg%3D&Expires=1660542966" | tar -I lz4 -xvf -
```
这里要下载很久，所以带宽要尽可能高。
在这里获取最新全节点快照地址
BSC快照github：[https://github.com/binance-chain/bsc-snapshots](https://github.com/binance-chain/bsc-snapshots)


> 如不是一边下载一边解压下载完成后解压 并移动 chaindata 和 triecache 到./jiedian/bsc/node/geth/ 文件夹下(二进制执行文件方式的目录./jiedian/node/geth/)
```bash
tar -I lz4 -xvf geth.tar.lz4 -C /jiedian/kuaizhao
```
替换数据

```bash
备份原有数据
mv ${BSC_DataDir}/geth/chaindata ${BSC_DataDir}/geth/chaindata_backup
mv ${BSC_DataDir}/geth/triecache ${BSC_DataDir}/geth/triecache_backup  #如果有这个文件夹  第一次搭建 没这个文件夹 如果报错提示不存在 就不管他了

mv server/data-seed/geth/chaindata ${BSC_DataDir}/geth/chaindata
mv server/data-seed/geth/triecache ${BSC_DataDir}/geth/triecache

例：我的BSC_DataDir 是 ./jiedian/node/geth/  
mv /jiedian/node/geth/chaindata /jiedian/node/geth/chaindata_backup
mv /jiedian/node/geth/triecache /jiedian/node/geth/triecache_backup

mv /jiedian/kuaizhao/server/data-seed/geth/chaindata /jiedian/node/geth/chaindata
mv /jiedian/kuaizhao/server/data-seed/geth/triecache /jiedian/node/geth/triecache
```

> 【注】我测试不备份 一直没从快照同步 ，问题应该是 直接执行 mv /jiedian/kuaizhao/server/data-seed/geth/chaindata /jiedian/node/geth/chaindata 命令，实际快照移动到了 /jiedian/kuaizhao/server/data-seed/geth/chaindata/chaindata ，这样就出问题了 ，如果 mv /jiedian/node/geth/chaindata /jiedian/node/geth执行这个命令 ，会提示文件夹已经存在 ， 移动失败   ，所以可以把原有的chaindata文件夹改名称 chaindata_backup。这样执行 mv /jiedian/kuaizhao/server/data-seed/geth/chaindata /jiedian/node/geth/chaindata 命令后，快照文件在 /jiedian/kuaizhao/server/data-seed/geth/chaindata 中。

移动完毕以后退出screen的xiazai窗口，并创建bsc窗口并开始运行节点。

退出当前窗口时用ctrl+ad（顺序按a和d字母即可），绝对不要用exit或ctrl+d退出会话

```bash
ctrl+ad		#退出xiazai窗口
screen -S bscnode   #新建一个 节点运行窗口
```

启动之前可以先自行配置一下节点的配置文件

修改BSC主网配置文件

```bash
vim /jiedian/bsc/config.toml
```

参数说明：

> `TrieTimeout`：这意味着geth将不会将状态持久化到数据库中，直到达到这个时间阈值，如果节点已经被强制关闭，它将从最后一个状态开始同步，这可能需要很长时间,可设置为：TrieTimeout
> = 200000000000 注意：当TrieTimeout值设置的越大，系统崩溃后，节点恢复的时间越长
> `HTTPHost`: HTTP-RPC服务连接白名单，此参数的值默认为 “localhost”，仅允许本地可访问，如果需要外网访问节点请设置为：“0.0.0.0”
> `HTTPVirtualHosts`：HTTP-RPC服务监听接口,此参数的值默认为[“localhost”],可设置为：HTTPVirtualHosts = ["*"]
> `HTTPPort`：http协议rpc端口
> `WSPort`：websocket协议rpc端口
> `WSHost`：websocket服务连接白名单，此参数的值默认为 “localhost”，仅允许本地可访问，可设置为：“0.0.0.0”
> `WSOrigins`：websocket服务监听接口,可设置为：WSOrigins = ["*"]

三、启动BSC智能全节点
screen -S bsc	#创建bsc节点启动窗口

```bash
cd /jiedian

geth --config ./config.toml --datadir ./node --cache 65536 --diffsync --rpc.allow-unprotected-txs --snapshot=true --txlookuplimit 0 
```
根据经验的参数配置：

- cache 设置为了 65536 MB，即 64 GB
- config.toml 中 maxPeers 设置为了 200 （默认 30）

然后按ctrl+ad回到主会话即可

参数说明：

> `–config`：指定BSC节点配置文件
> `–datadir`：指定BSC节点数据库和密钥存储库的数据目录(默认即可)
> `–cache`：设置最大分配给内部缓存的内存，默认:1024（设置越大，每次同步的数据越多，消耗的内存也越大）
> `–rpc.allow-unprotected-txs`：允许通过RPC提交不受保护的（非 EIP155 签名）交易
>` –txlookuplimit 0 `: 禁用删除事务索引
> `–diffsync`：启用差异同步协议来帮助节点更快地同步

四、节点状态监听

```scala
geth attach http://127.0.0.1:8545
```

#这里的端口如果修改配置文件了，就填写配置文件的端口即可

> eth.syncing	#查看当前区块情况

说明：

> currentBlock: 14290861, #当前同步到区块高度 
> highestBlock: 14297354, #主网当前高度
> knownStates:297473485, 
> pulledStates: 297473485, 
> startingBlock: 14270385


> net.peerCount	#查看当前连接节点数量，结果为false为同步完成
> eth.blockNumber #当前同步到区块高度

退出请按 ctrl+d 回到主会话。

- 停止节点
打开bsc窗口

```scala
 screen -x bsc
```


然后按 ctrl+c 即可

五、注意事项
bsc链主网浏览器浏览器：https://bscscan.com/

我这个设备配置同步到最高区块用了大概20分钟左右就追到了最高区块。
