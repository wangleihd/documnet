# dice server

## 完整流程
```
1 (笔记本电脑)创建部署合约的账户: 
deployer                   = 0x1111111b142e4b85b89aa55a8e4897ac719e1077  以太坊地址 
deployersk                 = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0  以太坊私钥

2 (笔记本电脑)创建2个合约的最高权限账户:
owner1                     = 0x1111111b142e4b85b89aa55a8e4897ac719e1077
owner2                     = 0x99999988f193218ac3474ff650c90495167d90dd
ownersk1                   = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0
ownersk2                   = f798cf97315b1a763d7ec8a6fd45ca0e6332fea6f2c11e47694fa70c7531a215

2.1 (笔记本电脑)创建荷官账户, 用来给用户开奖, 需要给荷官预先充值1以太. 
croupieraddr               = 0x888888c3b072b711a8a06d0fb5fef3ff60aa76d3
croupiersk                 = 190e169334cf50115bc6cbe56515bf33637696b21a97899f9c733300cbf03d05

3 (笔记本电脑)创建一个只能提款的用户, 这里我们简单的设置为owner1
withdrawer                 = 0x1111111b142e4b85b89aa55a8e4897ac719e1077

4 (笔记本电脑)创建一个用于签名"票"的账户. 防止用户伪造"票". (票: 用户的每次下注都需要先申请票)
secretsigneraddr           = 0xdddddd008b35b8fc366454b481c86454ca11352c
secretsignersk             = 098d1831329e66726a839b42fedbcc3d8203f9cd5f9b7bbde7a908d97a8cffe5

5 (笔记本电脑)编辑配置文件 config.txt, 如何编辑下面讲解. 另外, config.txt里面有最高权限的私钥, 所以不能放到远程服务, 以免被黑客盗取

6 (笔记本电脑) $ diceserver deploy  部署合约到以太坊区块链

7 (远程服务器) 创建目录 
/data/dice 存放启动脚本 run.sh
/data/dice/static 存放网站, 下面有网站的结构


run.sh:
nohup diceserver listen --listenaddr 0.0.0.0:10030 \
    --confaddr 0.0.0.0:10031 --filecache \
    --adminaddr 127.0.0.1:10032 \
    --gethaddr <你的parity IP地址>:8546 \
    --autorefund \
    --autorecharge \
    --startBlock -1 --logdir "/data/dice" &

8 (远程服务器) $ run.sh

9 (笔记本电脑) 激活服务, 激活的时候会将config.txt中的一些内容写入diceserver listen的内存中. 比如荷官的私钥. 这样做是为了减少被黑客获取私钥的可能性.
    $ diceserver activatesrv --address 远程服务器的域名:10031

10 (笔记本电脑) 访问 远程服务器的域名:10030 打开游戏主页. 也可以用 nginx代理 10030 端口.
```


## 配置文件 config.txt
```
authrefundinterval         = 10m
chainid                    = main
contractAddr               = 0x55Fbc4b35809410Aef9518b1A30911ABFAF46c71
contractinitamountEther    = 0.1
croupieraddr               = 0x888888c3b072b711a8a06d0fb5fef3ff60aa76d3
croupiersk                 = 190e169334cf50115bc6cbe56515bf33637696b21a97899f9c733300cbf03d05
croupiergaspricedelteGwei  = 1.1
db_address                 = 127.0.0.1
db_dbname                  = dice
db_param                   = autocommit=true
db_password                = n7fADs858vgz
db_port                    = 3306
db_username                = worker
deployer                   = 0x1111111b142e4b85b89aa55a8e4897ac719e1077
deployersk                 = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0
gasLimit                   = 5000000
gaspriceGwei               = 9
gethaddr                   = https://mainnet.infura.io/v3/4183b1eae4974865aa94145db4d36c17
http_client_sk             = 0e078580b7b95e134438777495dfec03449dedbfacd0dc7a8e8b4bd34a49626b
isdev                      = false
owner1                     = 0x1111111b142e4b85b89aa55a8e4897ac719e1077
owner2                     = 0x99999988f193218ac3474ff650c90495167d90dd
ownersk1                   = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0
ownersk2                   = f798cf97315b1a763d7ec8a6fd45ca0e6332fea6f2c11e47694fa70c7531a215
placebet_diceserveraddr    = bbb.aaaa.com
safedepths                 = 1,6,12
secretsigneraddr           = 0xdddddd008b35b8fc366454b481c86454ca11352c
secretsignersk             = 098d1831329e66726a839b42fedbcc3d8203f9cd5f9b7bbde7a908d97a8cffe5
setMaxProfitEther          = 10
withdrawer                 = 0x1111111b142e4b85b89aa55a8e4897ac719e1077


说明:
authrefundinterval         = 10m 每隔10分钟, 检查一次是否有开奖失败需要退款的用户. 如果有, 则自动退款
chainid                    = main  代表主链
contractAddr               = 0x55Fbc4b35809410Aef9518b1A30911ABFAF46c71  合约的地址, 部署程序会自动更新这个值
contractinitamountEther    = 0.1  部署合约的时候, 顺便给合约打的资金
croupieraddr               = 0x888888c3b072b711a8a06d0fb5fef3ff60aa76d3 荷官的地址
croupiersk                 = 190e169334cf50115bc6cbe56515bf33637696b21a97899f9c733300cbf03d05 荷官的秘钥
croupiergaspricedelteGwei  = 1.1 开奖的时候, 在推荐的gasprice基础上增加1.1Gwei
db_address                 = xxx.xxx.xxx.xxx 数据库地址
db_dbname                  = xx 数据库库名字
db_param                   = autocommit=true 
db_password                = xx 数据库密码
db_port                    = 3306 数据库端口
db_username                = xx 数据库用户名字
deployer                   = 0x1111111b142e4b85b89aa55a8e4897ac719e1077 部署合约的账户
deployersk                 = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0 部署合约的账户的私钥
gasLimit                   = 5000000 部署合约使用的gaslimit
gaspriceGwei               = 9 部署合约使用的gasprice
gethaddr                   = https://mainnet.infura.io/v3/4183b1eae4974865aa94145db4d36c17 部署合约使用的以太坊节点
http_client_sk             = 0e078580b7b95e134438777495dfec03449dedbfacd0dc7a8e8b4bd34a49626b 激活服务的时候, 使用此秘钥进行签名. 
isdev                      = false 是否是测试环境
owner1                     = 0x1111111b142e4b85b89aa55a8e4897ac719e1077  最高权限所有者
owner2                     = 0x99999988f193218ac3474ff650c90495167d90dd  最高权限所有者
ownersk1                   = 43e9c017ec0bd501216d42292a73106502c13beb1dbc4f8272e522b0d36655a0  最高权限所有者
ownersk2                   = f798cf97315b1a763d7ec8a6fd45ca0e6332fea6f2c11e47694fa70c7531a215  最高权限所有者
placebet_diceserveraddr    = bbb.aaaa.com:10030 测试下注的时候, 通过这个地址取"票"
safedepths                 = 1,6,12 对区块链进行 3种深度的扫描. 
secretsigneraddr           = 0xdddddd008b35b8fc366454b481c86454ca11352c 创建"票"的时候, 在"票"上使用这个账户来签名. 用户下注的时候, 合约会根据这个账户来验证是否是伪造的"票"
secretsignersk             = 098d1831329e66726a839b42fedbcc3d8203f9cd5f9b7bbde7a908d97a8cffe5
setMaxProfitEther          = 10 每一笔下注最多能获得的奖励是10以太
withdrawer                 = 0x1111111b142e4b85b89aa55a8e4897ac719e1077 提款用户的地址

```


##部署服务
```
将web编译后, 放到目录 /data/dice/static:
# tree static/
static/
├── app.85b7a3d7.js
├── css
│   ├── app.30bd640f.css
│   ├── chunk-45a4577d.ee8e806e.css
│   ├── chunk-4e8090e0.012dfed2.css
│   ├── chunk-cf8d2924.5efb5e75.css
│   ├── chunk-dc5c3f50.7f971a89.css
│   └── Question.1020ed23.css
├── favicon.ico
├── fonts
│   ├── iconfont.5baf54d5.woff
│   ├── iconfont.cdfeac96.ttf
│   └── iconfont.d73bfa86.eot
├── img
│   ├── bg.e5964f8d.png
│   ├── closed.d93515a7.png
│   ├── game-coinflip.46de741f.png
│   ├── game-dice2.6306b2ae.png
│   ├── game-dice.897688fe.png
│   ├── game-etheroll.5c63e015.png
│   ├── iconfont.8106a75e.svg
│   ├── qrcode-weixin.63602b20.png
│   ├── sprite-flag.5d191bb8.png
│   └── waiting.353471ab.png
├── index.html
├── js
│   ├── chunk-45a4577d.917d15a9.js
│   ├── chunk-4e8090e0.41f35785.js
│   ├── chunk-cf8d2924.8ef3e54c.js
│   ├── chunk-dc5c3f50.63898d01.js
│   ├── chunk-vendors.ddce00c6.js
│   ├── NotFound.c9124350.js
│   └── Question.e5426e15.js
├── logo-coinflip.png
├── logo-dice2.png
├── logo-dice.png
└── logo-etheroll.png


创建目录/data/dice, 进入其中.
```

##运行服务
```
nohup diceserver listen --listenaddr 0.0.0.0:10030 \
    --confaddr 0.0.0.0:10031 --filecache \
    --adminaddr 127.0.0.1:10032 \
    --gethaddr <你的parity IP地址>:8546 \
    --autorefund \
    --autorecharge \
    --startBlock -1 --logdir "/data/dice" &

参数说明:
--listenaddr 荷官服务的监听地址端口. 给前端页面提供api调用
/api/ticket  页面需要先读取此接口, 获取commit字段, 根据此字段下注
/api/contract 页面获取合约的abi以及合约的地址
/api/history  页面获取交易历史信息
/api/jackpotlog 页面获取大奖历史信息


 --confaddr 用于等待激活, 等到注入config.txt中的内容.
 
 --filecache 是游戏页面缓存到diceserver listen的内存中
 
 --adminaddr 用于在登录到远程服务器上查询信息.
 
 --autorefund 开启自动退款功能
 
 --autorecharge 开启自动给荷官充值功能
 
 --logdir "/data/dice" 日志路径
 
 --startBlock -1 从最新的块开始扫描. 如果服务器宕机. 可以从宕机的块开始扫描. 建议部署2台 diceserver listen 形成高可用. 
```

## 命令

### 一级命令 diceserver
```
$ diceserver -h
NAME:
   diceserver - fck-dice-server

USAGE:
   diceserver [global options] command [command options] [arguments...]
   
VERSION:
   v1.1.0 2019-03-05 11:58:40 +0800 @fe75421
build by go version go1.12 linux/amd64 at 2019-03-05 12:01:28 +0800
   
AUTHOR(S):
   x <x@x.com> 
   
COMMANDS:
     listen       监听等待配置注入. 注入后才能工作
     activatesrv  上传配置文件到listen的服务, 然后激活其
     admin        管理命令工具集合
     deploy       部署
     place        押注
     sk2addr      秘钥得到地址
     stop         停止合约
     kill         停止合约
     status       查看详细的合约信息
     betinfo      押注的详细信息
     refund       退款
     profit       setMaxProfit
     jackpot      increaseJackpot
     withdraw     提款
     deposit      充值
     signer       修改signer
     owner        修改 owner
     withdrawer   修改 withdrawer
     keystore     keystore工具
     version      

GLOBAL OPTIONS:
   --help, -h     show help
   --version, -v  print the version
```

### 二级命令 listen
```
$ diceserver listen -h
NAME:
   diceserver listen - 监听等待配置注入. 注入后才能工作

USAGE:
   diceserver listen [command options] [arguments...]

OPTIONS:
   --listenaddr value  站点服务的地址 (default: "0.0.0.0:8188")
   --confaddr value    配置与激活的地址 (default: "0.0.0.0:8189")
   --adminaddr value   管理端口 (default: "127.0.0.1:8190")
   --filecache         是否开启文件的缓存
   --gethaddr value    以太坊客户端RPC (default: "192.168.1.3:8552")
   --accint value      访问区块链的间隔 (default: 1s)
   --startBlock value  从此块开始扫描日志, -1为从最新的块开始扫描 (default: -1)
   --logdir value      指定日志的目录, 不指定则输出到标准输出
   --autorefund        是否开启自动退款
   --autorecharge      是否开启自动给荷官充值的功能
```

### 二级命令 admin
```
$ diceserver admin -h
NAME:
   diceserver admin - 管理命令工具集合

USAGE:
   diceserver admin command [command options] [arguments...]

COMMANDS:
     checkcommit   检查指定的多个commit, 查看是否没有开奖, 扫描t_reveal表, 退款要求t_settle.placebet_tx_hash 不为空
     controlflood  开启关闭防刷功能
     settlegas     开奖使用的gas

OPTIONS:
   --help, -h  show help
```


